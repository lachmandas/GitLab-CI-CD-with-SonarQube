stages:
  - build
  - sonar_scan
  - push_to_dockerhub
  - deploy

build_job:
  stage: build
  script:
    - docker build -t node-app:latest .

sonarqube-check:
  stage: sonar_scan
  image: 
    name: sonarsource/sonar-scanner-cli:latest
    entrypoint: [""]
  variables:
    SONAR_USER_HOME: "${CI_PROJECT_DIR}/.sonar"  # Defines the location of the analysis task cache
    GIT_DEPTH: "0"  # Tells git to fetch all the branches of the project, required by the analysis task
  cache:
    key: "${CI_JOB_NAME}"
    paths:
      - .sonar/cache
  script: 
    - sonar-scanner
  allow_failure: true
  only:
    - master

push_job:
  stage: push_to_dockerhub
  script:
    - docker login -u $DOCKERHUB_USER -p $DOCKERHUB_PASS
    - echo "hi 2"
    - docker image tag node-app:latest $DOCKERHUB_USER/node-app:latest
    - docker push $DOCKERHUB_USER/node-app:latest

deploy_job:
  stage: deploy
  script:
    - docker compose up -d
