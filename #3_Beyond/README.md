# 3 - Beyond 
EMSE Docker tutorial

The filder javaApp contains a simple HelloWorld application that you can use for the first example of this fifth POM. 
However you can use any console based java program. 

Step 1. Try to create the container file yourself, then, in case, check the solution.

Step 2. Once done, we try to set the container as a stand-alone application setting the default behaviour of a container.

Containers can also be used in different environments. Continuous integration (CI) makes big use of it.

Sample :

docker build -t gitlab.inf.unibz.it:4567/\<user-name\>/\<repo-name\>/\<container\> .
docker push gitlab.inf.unibz.it:4567/\<user-name\>/\<repo-name\>/\<container\>

You might need to login before pushing to the registry is possilbe.

These Docker images can then be used in the CI flow to build, test and deploy an application

https://gitlab.inf.unibz.it/help/ci/docker/using_docker_images.md

These are build according to a pipeline. More information about pipelines can be found here 

https://gitlab.inf.unibz.it/help/ci/pipelines.md

as an example, you could actualy use the plain Oracle image of the previous example and test and compile the code in CI.
The java code put in a gitlab repository can be build and tested automatically at every commit.

Get an idea of the CI configruation with the example in the folder

