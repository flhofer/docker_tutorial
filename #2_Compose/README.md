# 2 - Composing your flavor

## Docker compose 

Docker compose is a command line interface for docker that can be used for a variety of goals.  Similar to Docker, it uses a configuration file that helps automating a process. The file, or the *composition* is a set of information that describes how containers have to interact with the environment. However, one of the goals is often overlooked. 
Take for example the following compostion file. 

```yaml
version: '3'

services:
  python:
    build: .
    volumes:
      - .:/home/myapp
```

:grey_question: What does this compostition file do? (you can use the [getting started][1] guide)

With this in mind, thing of docker-compose as an extension of the Docker engine.
In the following we will see how to actually compose a service.

## Create a compose file

One of the most straight forward uses of a composition is a web service. It needs a stable operating system, a web server, a database and an interpreter for an eventual server side language like PHP or python. 
A LAMP (Linux Apache MySql PHP) is such a combination. It was very popular for a long time and is still of wide use.
The tutoria [HERE][2] shows how to create such a service using Docker and Docker compose.

What you will encouter during this tutorial is that there are many options and flags that remind a Dockerfile. Indeed, in many cases if specified in a compose file, these can replace or override the default values of an image.

## Resources

Resources management grow of importance the more a system is used to perform multiple tasks. On a server it is important that one service does not threaten the health of the other. 
As you may already have guessed, docker-compose forsees parameters to manage these resources. 

:grey_question: Which are they? Try to setup a memory limit for the database. 

A thing not discussed previously is the flexibility of volumes. Beyond the traditional mounting of a folder, volumes can also be shared only among containers. They are created and labeled, and then assigned to the instances requiring them.

:grey_question: In which cases such a shared volume might be useful?

## Swarm



[1]: <https://docs.docker.com/compose/gettingstarted/> "Docker compose, getting started"
[2]: <https://linuxconfig.org/how-to-create-a-docker-based-lamp-stack-using-docker-compose-on-ubuntu-18-04-bionic-beaver-linux>

