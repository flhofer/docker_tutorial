# 3 - Beyond 
EMSE Docker tutorial

## Using Docker as a service for coding

The folder javaApp contains a simple HelloWorld application that you can use for the first example of this fifth POM. 
However you can use any console based java program. 

Step 1. Try to create the container file yourself, then, in case, check the solution.

We create a container that compiles our Java program inside this folder. Using a Dockerfile we can compile and run the code in the container.

:grey_question: Is there another way to do that? How would you do it? 

Step 2. Once done, we try to set the container as a stand-alone application setting the default behaviour of a container.

Lets say that HelloWorld is our final great application that accepts a lot of parameters ecc and we use the container to provide the environment it needs to execute. To achieve this, we use the `ENRTYPOINT` command in the Dockerfile
The difference between command and entry point is that the latter defines the standard executable, but leaves room for parameters.

:grey_question: How does this differ from CMD, and when would you use it?



## Automating steps

Containers can also be used in different environments. In an environment such as GitLab, Continuous integration (CI) makes use of runners to automate processes. Those runners than launch task of variousc kinds, easing the development and deployment process.

If a repository has to be automated, it usually contains a `yml` file configuring the steps. You can find an example of such `.gitlab-ci.yml` file in the folder of the HelloWorld application. The example looks as follows.

```yaml
default:
  image: openjdk:7

stages:
  - build
  - test

compile: 
  stage: build
  script: javac HelloWorld.java
  artifacts:
    paths:
      - HelloWorld.class

testrun:
  stage: test
  script: java HelloWorld
  artifacts:
    paths:
      - HelloWorld.class
```

In this configuration we have two stages for our automation pipeline, build abd test. each of the stages the defines the executiona and the expected artifacts.
If this configuration is enabled, it starts the pipeline when a new commit is done.

:information_source: CI, pipelines and shared runners are not enabled by default. If there is no shared runner available, you can configure every computer as a runner. The GitLab of unibz has a shared Docker runner. 

Lets try to create a repository and automate the process. Once the files are uploaded and CI enabled in the settings, you have to create a new pipeline.

:grey_question: So, where do you see the limits of this approach?

Another interresting feature is that you can use the CI of GitLab to build docker images as well. Thus, if you have a particular software that has to run in a contaier, on every commit, the image is build and deployed to the docker registry (if configured).






Sample :
```sh
docker build -t gitlab.inf.unibz.it:4567/<user-name>/<repo-name>/<container> .
docker push gitlab.inf.unibz.it:4567/<user-name>/<repo-name>/<container>
```

You might need to login before pushing to the registry is possilbe.

These Docker images can then be used in the CI flow to build, test and deploy an application


These are build according to a pipeline. More information about pipelines can be found here 

https://gitlab.inf.unibz.it/help/ci/pipelines.md

as an example, you could actualy use the plain Oracle image of the previous example and test and compile the code in CI.
The java code put in a gitlab repository can be build and tested automatically at every commit.

Get an idea of the CI configruation with the example in the folder

https://gitlab.inf.unibz.it/help/ci/docker/using_docker_build.md
https://gitlab.inf.unibz.it/help/ci/docker/using_docker_images.md

