# 3 - Beyond 
EMSE Docker tutorial

## Using Docker as a service for coding

The `javaApp` folder contains a simple HelloWorld application that you can use for the first example of this fifth POM.
However, you can use any console-based Java program.

Step 1. Try to create the container file yourself, then check the solution if needed.

We create a container that compiles our Java program inside this folder. Using a Dockerfile, we can compile and run the code in the container.

:grey_question: Is there another way to do that? How would you do it? 

Step 2. Once done, set the container as a stand-alone application by defining its default behavior.

Let's say that HelloWorld is our final application that accepts many parameters, and we use the container to provide the environment it needs to execute. To achieve this, we use the `ENTRYPOINT` command in the Dockerfile.
The difference between `CMD` and `ENTRYPOINT` is that `ENTRYPOINT` defines the standard executable but leaves room for parameters.

:grey_question: How does this differ from CMD, and when would you use it?



## Automating steps

Containers can also be used in different environments. In an environment such as GitLab, Continuous Integration (CI) uses runners to automate processes. Those runners then launch tasks of various kinds, easing the development and deployment process.

If a repository has to be automated, it usually contains a `yml` file configuring the steps. You can find an example of such a `.gitlab-ci.yml` file in the HelloWorld application folder. The example looks as follows.

```yaml
default:
  image: eclipse-temurin:21-jdk

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

For example, you could use the previous example image to test and compile the CI code. The Java code in a GitLab repository can be built and tested automatically at every commit.

Let's try to create a repository and automate the process. Once the files are uploaded, and CI enabled in the settings, you have to create a new [pipeline][2]. 

:grey_question: So, where do you see the limits of this approach?

Another useful feature is that you can use GitLab CI to build Docker images as well. If you have software that has to run in a container, the image can be built on every commit and deployed to the Docker registry (if configured).
Info about this process can be found [here][4].

Lastly, sometimes we want to use our own registry for containers. GitLab provides a registry for every project. Container images can be built and pushed to the registry using this schema:

Sample :
```sh
docker build -t gitlab.inf.unibz.it:4567/<user-name>/<repo-name>/<container> .
docker push gitlab.inf.unibz.it:4567/<user-name>/<repo-name>/<container>
```

You might need to log in (`docker login <registry>`) before pushing to the registry. These Docker images can then be used in the CI flow to build, test, and deploy an application in a custom or private manner. More details are in the [manual][3].

[1]: <https://docs.gitlab.com/ci/runners/> "Runner configuration" 
[2]: <https://docs.gitlab.com/ci/pipelines/> "Automation pipeline"
[3]: <https://docs.gitlab.com/ci/docker/using_docker_build/> "Docker build in CI"
[4]: <https://docs.gitlab.com/ci/docker/using_docker_images/> "Docker images in CI"
[5]: <https://docs.gitlab.com/ci/yaml/> "YAML CI - Gitlab"
