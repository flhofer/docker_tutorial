# 3 - Beyond 
EMSE Docker tutorial

## Using Docker as a service for coding

The folder javaApp contains a simple HelloWorld application that you can use for the first example of this fifth POM. 
However, you can use any console-based java program. 

Step 1. Try to create the container file yourself, then, in case, check the solution.

We create a container that compiles our Java program inside this folder. Using a Dockerfile, we can compile and run the code in the container.

:grey_question: Is there another way to do that? How would you do it? 

Step 2. Once done, we try to set the container as a stand-alone application setting the container's default behavior.

Let's say that HelloWorld is our great final application that accepts a lot of parameters etc., and we use the container to provide the environment it needs to execute. To achieve this, we use the `ENRTYPOINT` command in the Dockerfile
The difference between command and entry point is that the latter defines the standard executable but leaves room for parameters.

:grey_question: How does this differ from CMD, and when would you use it?



## Automating steps

Containers can also be used in different environments. In an environment such as GitLab, Continuous integration (CI) uses runners to automate processes. Those runners then launch task of various kinds, easing the development and deployment process.

If a repository has to be automated, it usually contains a `yml` file configuring the steps. You can find an example of such a `.gitlab-ci.yml` file in the HelloWorld application folder. The example looks as follows.

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

In this configuration, we have two stages for our automation pipeline, build and test. Each of the stages defines the execution and the expected artifacts. If this configuration is enabled, it starts the pipeline when a new commit is done. Details on how to compose this configuration can be found [here][5].

:information_source: CI, pipelines, and shared runners are not enabled by default. If there is no shared runner available, you can configure every computer as a runner. The GitLab of UniBz has a shared Docker runner. See [runners][1] for more info about runners.

For example, you could actually use the previous example's plain Oracle image and test and compile the CI code. The java code put in a GitLab repository can be built and tested automatically at every commit.

Let's try to create a repository and automate the process. Once the files are uploaded, and CI enabled in the settings, you have to create a new [pipeline][2]. 

:grey_question: So, where do you see the limits of this approach?

Another exciting feature is that you can use the CI of GitLab to build docker images as well. Thus, if you have a particular software that has to run in a container, the image is built on every commit and deployed to the docker registry (if configured).
Info about this process can be found [here][4].

Lastly, sometimes we want to use our own registry for containers. Gitlab foresees a registry for every project. The container images can be built and pushed to their registry following this schema:

Sample :
```sh
docker build -t gitlab.inf.unibz.it:4567/<user-name>/<repo-name>/<container> .
docker push gitlab.inf.unibz.it:4567/<user-name>/<repo-name>/<container>
```

You might need to login (`docker login <registry>`) before pushing to the registry is possible. These Docker images can then be used in the CI flow to build, test and deploy an application in a custom or private manner. More details in the [manual][3]. 

[1]: <https://docs.gitlab.com/ee/ci/runners/> "Runner configuration" 
[2]: <https://gitlab.inf.unibz.it/help/ci/pipelines.md> "Automation pipeline"
[3]: <https://gitlab.inf.unibz.it/help/ci/docker/using_docker_build.md> "Docker build in CI"
[4]: <https://gitlab.inf.unibz.it/help/ci/docker/using_docker_images.md> "Docker images in CI"
[5]: <https://gitlab.inf.unibz.it/help/ci/yaml/README.md> "YAML CI - Gitlab"

