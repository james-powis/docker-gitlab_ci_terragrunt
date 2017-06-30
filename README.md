Leveraging the container as a Docker Gitlab-CI runner we can utilize Terragrunt Locking and Terraform Remote state to allow for CI executed ephemeral runners for executing terraform plan and run workflows.

I am happy to provide any assistance you may have questions about however please read the following

Reference:
https://docs.gitlab.com/ce/ci/docker/using_docker_images.html
https://github.com/gruntwork-io/terragrunt#keep-your-remote-state-configuration-dry
https://docs.gitlab.com/ce/ssh/README.html#deploy-keys

You will need to configure a deploy key which has permissions to each of the repositories your terraform modules are stored in. The private key should be stored in a gitlab variable for the terragrunt repo (the repo containing your terraform.tfvars files). In the `gitlab-ci.yaml` example below that variable is `TERRAGRUNT_KEY`. It is also recommended that any sensitive data be stored in gitlab variables as demonstrated with `TF_VAR_access_key` and `TF_VAR_secret_key`

```yaml
stages:
- plan
- deploy

variables:
  AWS_ACCESS_KEY_ID: $TF_VAR_access_key
  AWS_SECRET_ACCESS_KEY: $TF_VAR_secret_key
  AWS_DEFAULT_REGION: us-east-1

before_script:
  - eval $(ssh-agent -s)
  - ssh-add <(echo "$TERRAGRUNT_KEY")
  - mkdir -p /root/.ssh
  - echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config

plan:
  stage: plan
  image: powisj/gitlab_ci_terragrunt:0.9.8
  script:
    - terragrunt plan-all --terragrunt-non-interactive
  tags:
    - docker-runner

deploy:
  stage: deploy
  image: powisj/gitlab_ci_terragrunt:0.9.8
  script:
    - terragrunt plan-all --terragrunt-non-interactive
    - terragrunt apply-all --terragrunt-non-interactive
  tags:
    - docker-runner
  only:
    - /^v\d+/
  except:
    - /^v\d+.*-rc\d+/
```
