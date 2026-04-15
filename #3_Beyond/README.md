# 3 - Beyond 
EMSE Docker tutorial

## Using Docker as a service for coding

The `javaApp` folder contains a simple HelloWorld application that you can use for the first example of this fifth POM.
However, you can use any console-based Java program.

## Before you start (2 minutes)

Run these checks in `#3_Beyond/javaApp`:

```sh
docker build -t javaapp:demo .
docker run --rm javaapp:demo
```

If `docker build` fails, inspect the first error line and verify that `Dockerfile` and `HelloWorld.java` are in the current folder.

## Quick mental model

* Build stage: turn source code into an artifact (for example `.class` or `.jar`).
* Runtime stage: execute the built artifact in a container.
* CI pipeline: automated jobs (build/test/deploy) triggered by commits.
* Registry: stores container images produced by CI.
* Artifact: output produced by one job and reused by later jobs.

## Minimal command cheat sheet

* `docker build -t <name>:<tag> .`: build an image.
* `docker run --rm <name>:<tag>`: run and auto-remove container.
* `docker image ls`: verify image build result.
* `docker logs <container>`: inspect runtime output/errors.
* `docker tag <img> <registry/path:tag>`: prepare for push.
* `docker push <registry/path:tag>`: publish image to registry.

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

### Reading command output quickly

When you run `docker build`, focus on the first failing step (`#N [x/y] ...`) and its exact error line.
When you run containers, check `docker logs` for startup errors before changing code.
In CI, check which stage failed first (build vs test) and whether expected artifacts were produced.

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

## Beginner troubleshooting

* `javac` not found during build: verify base image and Dockerfile steps.
* Image builds but app does not start: confirm `CMD`/`ENTRYPOINT` and working directory.
* CI job fails on missing artifact: ensure artifact paths match produced filenames.
* Push fails with permission denied: verify `docker login` target registry and project rights.

## 10-minute practice

1. Build `javaApp` from project root: `docker build -t javaapp:demo './#3_Beyond/javaApp'`.
2. Run and verify output: `docker run --rm javaapp:demo`.
3. Change `HelloWorld.java` text and rebuild image.
4. Run again to confirm the change appears.
5. (Optional) Tag image as if for a registry: `docker tag javaapp:demo my-registry.example.com/demo/javaapp:demo`.

[1]: <https://docs.gitlab.com/ci/runners/> "Runner configuration" 
[2]: <https://docs.gitlab.com/ci/pipelines/> "Automation pipeline"
[3]: <https://docs.gitlab.com/ci/docker/using_docker_build/> "Docker build in CI"
[4]: <https://docs.gitlab.com/ci/docker/using_docker_images/> "Docker images in CI"
[5]: <https://docs.gitlab.com/ci/yaml/> "YAML CI - Gitlab"
