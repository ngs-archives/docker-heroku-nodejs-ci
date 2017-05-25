docker-heroku-nodejs-ci
=======================

Node.js image for CircleCI 2.0 with Heroku CLI

[![](https://img.shields.io/docker/automated/atsnngs/heroku-nodejs-ci.svg)](https://hub.docker.com/r/atsnngs/heroku-nodejs-ci/)

Setup
-----

ref: https://circleci.com/docs/2.0/project-walkthrough/#deployment-

### .circleci/setup-heroku.sh

```sh
#!/bin/bash

set -eu

git remote add heroku https://git.heroku.com/$HEROKU_APP_NAME.git

cat > ~/.netrc << EOF
machine api.heroku.com
	login $HEROKU_LOGIN
	password $HEROKU_API_KEY
machine git.heroku.com
	login $HEROKU_LOGIN
	password $HEROKU_API_KEY
EOF

# Add heroku.com to the list of known hosts
ssh-keyscan -H heroku.com >> ~/.ssh/known_hosts
```

### .circleci/config.yml

```yaml
version: 2
jobs:
  build:
    working_directory: /app/user
    docker:
      - image: atsnngs/heroku-nodejs-ci:latest
    steps:
      - checkout
      - restore_cache:
          key: node_modules-{{ .Branch }}-{{ checksum "package.json" }}
      - run:
          command: npm install
      - save_cache:
          key: node_modules-{{ .Branch }}-{{ checksum "package.json" }}
          paths:
            - node_modules
      - run:
          command: npm test
      - run:
          command: |
            if [ "${CIRCLE_BRANCH}" == "master" ]; then
              ./.circleci/setup-heroku.sh
              git push heroku HEAD:master
            fi
```
