# Docker repo pull syncer

Useful Docker image that can be used in GitLab (Scheduled) Pipelines for pulling remote repositories.

Normally, GitLab Community Edition only offers you to mirror repositories via `push`. This Docker image with yaml file (see below), can be used to enable git `pull` mirror feature for free!

Docker image: `danger89/repo_mirror_pull`

GitLab Pipeline yaml file:

```yml
repo_pull_sync:
  image: danger89/repo_mirror_pull:latest
  rules:
    - if: '$CI_PIPELINE_SOURCE == "schedule"'
    - if: $REMOTE_URL
    - if: $REMOTE_BRANCH
    - if: $ACCESS_TOKEN
  before_script:
    - git config --global user.name "${GITLAB_USER_NAME}"
    - git config --global user.email "${GITLAB_USER_EMAIL}"
  script:
    - git checkout $CI_DEFAULT_BRANCH
    - git pull
    - git remote remove upstream || true
    - git remote add upstream $REMOTE_URL
    - git fetch upstream
    - git merge upstream/$REMOTE_BRANCH
    - git push "https://${GITLAB_USER_LOGIN}:${ACCESS_TOKEN}@${CI_REPOSITORY_URL#*@}" "HEAD:${CI_DEFAULT_BRANCH}"
```

* Create in GitLab a Project Access Token first in GitLab. Via: Settings->Access Tokens. Check 'api' as the scope.
* Create a new schedule, via: CI/CD->Schedules->New schedule. With the following 3 variables:
  * REMOTE_URL (example: `https://github.com/project/repo.git`)
  * REMOTE_BRANCH (example: `master`)
  * ACCESS_TOKEN: (see Access Token in the first step! Example: `gplat-234hcand9q289rba89dghqa892agbd89arg2854`, )
* Save pipeline schedule

