# 1 - Building images and creating containers

There are essentialy two ways to ways to create an image:

* Via an interactive prompt and commit
* Via a build script that details all the steps


Let's start with basics. Image: Ubuntu.

You need privileges to access control groups (CGroups), thus docker might need `sudo` or root access

```sh
docker pull ubuntu:latest
```

This loads an Ubuntu image from the default registry (hub.docker.com). The text after the colon defines the version, or *tag* of the image. `latest` always stands for the newest release.

:grey_question: Which other releases are there? Name some tags


 

The reference for `Dockerfile` can be found here 
https://docs.docker.com/engine/reference/builder/

