# Prometheus-kubeStaticMetrics-Grafana-Kubernetes
### Overview
- we will start working on a simple website project
- we want to automate any of the manual steps required for integrating the changes of multiple developers
- we will create a pipeline that will build and test the software we are creating
- we will try to automate the build and deployment of a simple website project build with JavaScript using the React framework
- Automation CLI tools we are using is GitLab CI
#### 📚 Resources
* [Website project (fork this)](https://github.com/mnforba/cicd-gitlab.git)
### Getting started - Build the project
- most software projects have a build step where code is compiled or prepared for production-use
- yarn is a Node.js package manager that helps with managing the project by running scripts and installing dependencies
- for a Node.js project, the `node_modules` folder contains all project dependencies
- project dependencies are installed using `yarn install` but are NOT stored in the Git repository
- it is really a bad idea to store project dependencies in the code repository
- image tags that contain `alpine` or `slim` are smaller in size as they use a lightweight Linux distribution

#### 📚 Resources
* [Official Node.js images on Dockerhub](https://hub.docker.com/_/node)
* [Check the current Node.js LTS version](https://nodejs.org/en/)
* [Yarn CLI documentation](https://yarnpkg.com/cli/)
* [Pipeline after this step](poc/pipeline-configs/2-03-.gitlab-ci.yml)

- Create a job to verify that the build folder contains a file named index.html
- Create another job that runs the project unit tests using the command yarn test
* [Pipeline after this step](poc/pipeline-configs/2-05-.gitlab-ci.yml)
### How do we integrate changes?
- we use Git to keep track of code changes
- we need to ensure we don't break the main pipeline

### Merge requests
- we need to ensure that the chances of breaking the main branch are reduced
- here are some project settings for working with Merge Requests that I used:
    * Go to Settings > Merge requests
    * Merge method > select *Fast-forward merge*
    * Squash commits when merging > select *Encourage*
    * Merge checks > check *Pipelines must succeed*
- protect the master by allowing changes only through a merge request: 
    * Settings > Repository > Branch main > Allowed to push - select *No one*
#### 📚 Resources

* [Pipeline after this step](poc/pipeline-configs/2-07-.gitlab-ci.yml)
### Code review
- merge requests are often used to review the work before merging it
- merge requests allow us to document why a specific change was made
- other people on the team can review the changes and share their feedback by commenting
- if you still need to make changes from the merge request, you can open the branch using the Web IDE

### Integration tests
- before we ship the final product, we try to test it to see if it works
- testing is done of various levels but high-level tests typically include integration and acceptance tests
- we use cURL to create an HTTP call to the website
#### 📚 Resources
* [Pipeline after this step](poc/pipeline-configs/2-09-.gitlab-ci.yml)

## How to structure a pipeline
- our current pipeline structure is just an example, not a rule
- there are a few principles to consider
- principle #1: Fail fast
    * we want to ensure that the most common reasons why a pipeline would fail are detected early
    * put jobs of the same size in the same stage
- principle #2: Job dependencies
    * you need to understand the dependencies between the jobs
    * example: you can't test what was not already built yet
    * if jobs have dependencies between them, they need to be in distinct stages
## Continuous Deployment with GitLab & AWS
### overview
- we will take our website project and deploy it to the AWS cloud. 
- we need an AWS account to continue
- the first AWS service that we will use is AWS S3 storage service
- the website is static and requires no computing power or a database
- we will use AWS S3 to store the public files and serve them over HTTP
- AWS S3 files (objects) are stored in buckets
- to interact with the AWS cloud services, we need to use AWS CLI
#### 📚 Resources
* [AWS CLI documentation](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/index.html)
* [AWS CLI on Dockerhub](https://hub.docker.com/r/amazon/aws-cli)
* [Pipeline after this step](poc/pipeline-configs/3-04-.gitlab-ci.yml)

### Uploading a file to S3
- to upload a file to S3, we will use the copy command `cp`
- `aws s3 cp` allows us to copy a file to and from AWS S3 
#### 📚 Resources
* [AWS CLI for S3 documentation](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/s3/index.html)
* [Pipeline after this step](poc/pipeline-configs/3-05-.gitlab-ci.yml)
### Masking & protecting variables
- to define a variable, go to *Settings > CI/CD > Variables > Add variable*
- we typically store passwords or other secrets
- a variable has a key and a value
- it is a good practice to write the key in uppercase and to separate any words with underscores
- flags:
    * Protect variable: if enabled, the variable is not available in branches, apart from the default branch (main), which is a protected branch
    * Mask variable: if enabled, the variable value is never displayed in clear text in job logs
#### 📚 Resources
* [Pipeline after this step](poc/pipeline-configs/3-06-.gitlab-ci.yml)
### Identity management with AWS IAM
- we don't want to use our username and password to use AWS services from the CLI 
- as we only need access to S3, it makes sense to work with an account with limited permissions
- IAM allows us to manage access to the AWS services
- we will create a new user with the following settings:
    * account type: programmatic access
    * permissions: attach existing policy: AmazonS3FullAccess
- the account details will be displayed only once
- go to *Settings > CI/CD > Variables > Add variable* and define the following unprotected variables:
    * AWS_ACCESS_KEY_ID
    * AWS_SECRET_ACCESS_KEY
    * AWS_DEFAULT_REGION
- AWS CLI will look for these variables and use them to authenticate
### Uploading multiple files to S3
- using cp to copy individual files can take a lot of space in the pipeline config
- some file names are generated during the build process, and we can't know them in advance
- we will use sync to align the content between the build folder in GitLab and the S3 bucket
#### 📚 Resources

* [AWS S3 sync command documentation](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/s3/sync.html)
* [Pipeline after this step](poc/pipeline-configs/3-08-.gitlab-ci.yml)
### Hosting a website on S3
- files in the S3 bucket are not publicly available
- to get the website to work, we need to configure the bucket
- from the bucket, click on *Properties* and enable Static website hosting
- from the bucket, click on the *Permissions* tab and disable *Block public access*
- from the bucket, click on the *Permissions* tab and set a bucket policy
#### 📚 Resources
* [S3 bucket policy example](poc/s3-bucket-policy-example.json)
### Controlling when jobs run

- we don’t want to deploy to production from every branch
- to decide which jobs to run, we can use `rules:` to set a condition
- `CI_COMMIT_REF_NAME` gives us the current branch that is running the pipeline
- `CI_DEFAULT_BRANCH` gives us the name of the default branch (typically main or master)

#### 📚 Resources
* [GitLab reference for the .gitlab-ci.yml file - rules:](https://poc.gitlab.com/ee/ci/yaml/#rules)
* [Predefined variables in GitLab](https://poc.gitlab.com/ee/ci/variables/predefined_variables.html)
* [Pipeline after this step](poc/pipeline-configs/3-10-.gitlab-ci.yml)

## Post-deployment testing

- we will use `cURL` to download the index.html file from the website
- with `grep`, we will check to see if the index.html file contains a specific string
#### 📚 Resources

* [Pipeline after this step](poc/pipeline-configs/3-11-.gitlab-ci.yml)

- create a staging environment and add it to the CI/CD pipeline
#### 📚 Resources

* [Pipeline after this step](poc/pipeline-configs/3-14-.gitlab-ci.yml)
### Environments

- every system where we deploy an application is an environment
- typical environments include test, staging & production
- GitLab offers support for environments
- we can define environments in *Deployments > Environments*
#### 📚 Resources
* [Predefined variables in GitLab](https://poc.gitlab.com/ee/ci/variables/predefined_variables.html)
* [Pipeline after this step](poc/pipeline-configs/3-15-.gitlab-ci.yml)

### Reusing configuration
- sometimes, multiple jobs may look almost the same, and we should try to avoid repeating ourselves
- GitLab allows us to inherit from an exiting job with the `extends:` keyword
- add a file named `version.html` which contains the current build number
- the current build number is given by a predefined GitLab CI variable named `CI_PIPELINE_IID`
#### 📚 Resources

* [Pipeline after this step](poc/pipeline-configs/3-16-.gitlab-ci.yml)
* [Predefined variables in GitLab](https://poc.gitlab.com/ee/ci/variables/predefined_variables.html)
* [Pipeline after this step](poc/pipeline-configs/3-18-.gitlab-ci.yml)
### Continuous Delivery pipeline

- adding `when: manual` allows us to manually trigger the production deployment
#### 📚 Resources

* [Pipeline after this step](poc/pipeline-configs/3-19-.gitlab-ci.yml)
## Deploying a dockerized application to AWS

- modern applications tend to be more complex, and most of them use Docker 
- we will build & deploy an application that runs in a Docker container
### Introduction to AWS Elastic Beanstalk

- AWS Elastic Beanstalk (AWS EB) is a service that allows us to deploy an application in the AWS cloud without having to worry about managing the virtual server(s) that runs it
### Creating a new AWS Elastic Beanstalk application

- when creating an EB app, choose the *Docker* platform and deploy the sample app
- AWS EB will use two AWS services to run the application:
    * EC2 (virtual servers)
    * S3 (storage)
- to deploy a new version, go to the environment in EB and upload the file in templates called `Dockerrun.aws.public.json`

#### 📚 Resources
* [Dockerrun.aws.public.json](templates/Dockerrun.aws.public.json)
### Creating the Dockerfile

- create a new file called `Dockerfile` in the root of the project
- add the following contents to it:
```
FROM nginx:1.20-alpine
COPY build /usr/share/nginx/html
```
#### 📚 Resources

* [Official build of Nginx on Dockerhub](https://hub.docker.com/_/nginx)
### Building the Docker image

- to build the Docker image, we will use the command `docker build`
- to build Docker images from a GitLab CI pipeline, we need to start the Docker Daemon as a service
#### 📚 Resources

* [docker build command reference](https://poc.docker.com/engine/reference/commandline/build/)
* [Docker in Docker (dind) on Dockerhub](https://hub.docker.com/_/docker)
* [Predefined variables in GitLab](https://poc.gitlab.com/ee/ci/variables/predefined_variables.html)
* [docker image ls](https://poc.docker.com/engine/reference/commandline/image_ls/)
* [Pipeline after this step](poc/pipeline-configs/4-05-.gitlab-ci.yml)

### Docker container registry
- to preserve a Docker image, we need to push it to a registry
- Dockerhub is a public registry with Docker images
- GitLab offers a private Docker registry which is ideal for private projects
#### 📚 Resources

* [docker login command reference](https://poc.docker.com/engine/reference/commandline/login/)
* [docker push command reference](https://poc.docker.com/engine/reference/commandline/push/)
* [Pipeline after this step](poc/pipeline-configs/4-06-.gitlab-ci.yml)

### Testing the container
- we want to test the container to see if the application running on it (web server) is working properly
- to start the container, we will use the `services:` keyword
#### 📚 Resources

* [Pipeline after this step](poc/pipeline-configs/4-07-.gitlab-ci.yml)
### Private registry authentication

- AWS EB requires authentication credentials to pull our Docker image
- GitLab allows us to create a Deploy Token that AWS can use to log to the registry
- to generate a Deploy Token, from your project, go to *Settings > Repository > Deploy tokens*.
- create a new Deploy Token with the following scopes:
    * read_repository
    * read_registry
#### 📚 Resources

* [Pipeline after this step](poc/pipeline-configs/4-08-.gitlab-ci.yml)
### Deploying to AWS Elastic Beanstalk

- a new deployment to AWS EB happens in two steps
- step 1: we create a new application version with `aws elasticbeanstalk create-application-version`
- step 2: we update the environment with the new application version with `aws elasticbeanstalk update-environment`
#### 📚 Resources
* [Pipeline after this step](poc/pipeline-configs/4-09-.gitlab-ci.yml)

### Post-deployment testing

- updating an EB environment does not happen instantly
- we will use `aws elasticbeanstalk wait` to know when the environment has been updated
#### 📚 Resources
* [Pipeline after this step](poc/pipeline-configs/4-10-.gitlab-ci.yml)

file:///C:/Users/nforb/Downloads/20220605_023334.jpg

