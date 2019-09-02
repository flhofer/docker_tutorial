# 1 - Building images and creating containers

There are essentialy two ways to ways to create an image:

* Via an interactive prompt and commit
* Via a build script that details all the steps, a [Dockerfile][1]


Let's start with basics. Image: Ubuntu.

You need privileges to access control groups ([CGroups][2]), thus docker might need `sudo` or root access

```sh
docker pull ubuntu:latest
```

This loads an Ubuntu image from the default registry (hub.docker.com). The text after the colon defines the version, or *tag* of the image. `latest` always stands for the newest release.

:grey_question: Which other releases are there? Name some tags

:information_source: Hint\: you might want to check the [hub]: hub.docker.com
 

References:
[1]: <https://docs.docker.com/engine/reference/builder/> "Docker Builder reference"
[2]: <https://www.linuxjournal.com/content/everything-you-need-know-about-linux-containers-part-i-linux-control-groups-and-process> "Linux Control groups and process"

