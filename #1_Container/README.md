# 1 - Building images and creating containers

There are essentialy two ways to ways to create an image:

* Via an interactive prompt and commit
* Via a build script that details all the steps, a [Dockerfile][1]


Let's start with basics. Image: Ubuntu.

You need privileges to access control groups ([CGroups][2]), thus docker might need `sudo` or root access

```sh
docker pull ubuntu:latest
```

This loads an Ubuntu image from the default registry (hub.docker.com). The text after the colon defines the version, or *tag* of the image. `latest` stands for the newest release that is *safe* to use.

:grey_question: Which other releases are there? Name some tags

:information_source: Hint\: you might want to check the [hub](hub.docker.com)
 
If you lookup the entries of the hub, you can see two main groups, official and user/comunity maintained. The name tells the whole story. 
So we have now the `latest`, which seems not the actual latest, but more the *stable*. Indeed, its the actual LTS (long term support) version

If you try the pull command again with an other tag, the download will start again. Typing `docker images` we now get the list of available images.

```sh
florian@helius:~/Public/Projects/docker_tutorial/#1_Container$ sudo docker images
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
ubuntu                  rolling             9f3d7c446553        2 weeks ago         70MB
ubuntu                  latest              a2a15febcdf3        2 weeks ago         64.2MB
```
Note the compact size of the images.

These images give us the grounds we need to run containers. If we run now one of the images in a container, we create a new process, user and mount space. As discussed in the introduction, this is an isolation concept close to full virtualization. We will see later on what the differences are.

Let's try our image

```sh
docker run --name cont1 -ait ubuntu:latest 
```

This starts a new container instance and interactively (-ait) connects to it. The container automatically exits once it has nothing to do anymore, i.e. if you start a container to run a program, it stops again as soon as the program terminates.
If you type `exit`, you terminate the promopt return back to the console.
You did not specify any command, but any container has a default execution entry. We can verify that by typing `docker inspect <image-name>`.
For `ubuntu:latest` this gives a JSON format output containing

```sh
            "Cmd": [
                "/bin/sh",
                "-c",
                "#(nop) ",
                "CMD [\"/bin/bash\"]"
            ],
 ```

This means, it is actually the same as 
```sh
docker run --name cont1 -ait ubuntu:latest bash 
```
where you can put any command instead ( we can omit '/bin/' as it is specified in the environment variable path). The command could also be as simple as `echo "Hello world"`, just printing the line to the console and exiting.



References:   
[1]: <https://docs.docker.com/engine/reference/builder/> "Docker Builder reference"   
[2]: <https://www.linuxjournal.com/content/everything-you-need-know-about-linux-containers-part-i-linux-control-groups-and-process> "Linux Control groups and process"

