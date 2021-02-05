# 2 - Composing your flavor

## Docker compose 

Docker compose is a command-line interface for Docker that can be used for a variety of goals.  Similar to Docker, it uses a configuration file that helps to automate a process. The file, or the *composition* is a set of information that describes how containers have to interact with the environment. However, one of the goals is often overlooked. 
Take for example, the following composition file. 

```yaml
version: '3'

services:
  python:
    build: .
    volumes:
      - .:/home/myapp
```

:grey_question: What does this composition file do? (you can use the [getting started][1] guide)

With this in mind, a thing of docker-compose as an extension of the Docker engine.
In the following, we will see how to actually compose a service.

## Create a compose file

One of the most straightforward uses of a composition is a web service. It needs a stable operating system, a web server, a database, and an interpreter for an eventual server-side language like PHP or python. 
A LAMP (Linux Apache MySql PHP) is such a combination. It was trendy for a long time and is still of wide use.
The tutorial [HERE][2] shows how to create such a service using Docker and Docker compose.

What you will encounter during this tutorial is that many options and flags remind a Dockerfile. Indeed, in many cases if specified in a compose file, these can replace or override the default values of an image.

## Resources

Resources management grow of importance the more a system is used to perform multiple tasks. On a server, one service mustn't threaten the health of the other. 
As you may already have guessed, docker-compose foresees parameters to manage these resources. 

:grey_question: Which are they? Try to set up a memory limit for the database. 

A thing not discussed previously is the flexibility of volumes. Beyond the traditional bind mounting of a folder, volumes can also be shared only among containers. They are created and labeled and then assigned to the instances requiring them.

:grey_question: In which cases such a shared volume might be useful?

## Swarm

An extension of a composition is a Swarm. Docker swarm is only one of a few ways to use multiple nodes to provide a particular service. The [Voting App][3] example of the tutorial shows how an application can use parallelized instances. With multiple nodes, the service can be spawned on many servers or virtual instances. If you didn't try it out yet, do it now.

:grey_question: So, when is this really useful? Who might be applying this technology?




[1]: <https://docs.docker.com/compose/gettingstarted/> "Docker compose, getting started"
[2]: <https://linuxconfig.org/how-to-create-a-docker-based-lamp-stack-using-docker-compose-on-ubuntu-18-04-bionic-beaver-linux>
[3]: <https://github.com/docker/labs/blob/master/beginner/chapters/votingapp.md> "Deploying to a Swarm"
