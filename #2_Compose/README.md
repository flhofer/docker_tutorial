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

:grey_question: What does this compostition file do? (you can use the [getting started]


## Create a compose file

One of the most straight forward uses of a composition is a web service. It needs a stable operating system, a web server, a database and an interpreter for an eventual server side language like PHP or python. 
A LAMP (Linux Apache MySql PHP) is such a combination. It was very popular for a long time and is still of wide use.
The tutoria [HERE][2] shows how to create such a service using Docker and Docker compose.

What you will encouter during this tutorial is that there are many options and flags that remind a Dockerfile. Indeed, in many cases if specified in a compose file, these can replace or override the default values of an image.




[1]: <https://docs.docker.com/compose/gettingstarted/> "Docker compose, getting started"
[2]: <https://linuxconfig.org/how-to-create-a-docker-based-lamp-stack-using-docker-compose-on-ubuntu-18-04-bionic-beaver-linux>

