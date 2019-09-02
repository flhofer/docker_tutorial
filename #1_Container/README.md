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
ubuntu                  rolling             9f3d7c446553        1 week ago          70MB
ubuntu                  latest              a2a15febcdf3        1 week ago          64.2MB
```
Note the compact size of the images.

These images give us the grounds we need to run containers. If we run now one of the images in a container, we create a new process, user and mount space. As discussed in the introduction, this is an isolation concept close to full virtualization. We will see later on what the differences are.

Let's try our image

```sh
docker run --name cont1 -it ubuntu:latest 
```

This starts a new container instance with the name `cont1` and interactively (-it) connects to it. The container automatically exits once it has nothing to do anymore, i.e. if you start a container to run a program, it stops again as soon as the program terminates.

:information_source: -i = always attach to container,  -t create TTY, together -it. Alternatively, -d stands for detach, thus running in the background. We will see later what it means. More info in the [run reference][3]

If you type `exit`, you terminate the prompt return back to the console. You did not specify any command, but any container has a default execution or command entry. We can verify that by typing `docker inspect <image-name>`.
For `ubuntu:latest` this gives a JSON format output containing

```sh
            "Cmd": [
                "/bin/sh",
                "-c",
                "#(nop) ",
                "CMD [\"/bin/bash\"]"
            ],
 ```

This means, for this image is actually the same as 
```sh
docker run --name cont1 -it ubuntu:latest bash 
```
where you can put any command instead ( we can omit '/bin/' as it is specified in the environment variable path). The command could also be as simple as `echo "Hello world"`, just printing the line to the console and exiting.

:grey_question: Why do you think, you can't run the conainer again, even with the same parameters?

Without deleting any container, we can start our `cont1` again by typing `docker container start -i cont1`, and we are back in the bash. ( no need for -t, tty is already created )
Here now the curious thing, the actual difference to a virtualization.
In the bash, type `uname -a`. You should have a different output for each *version* of your systems. mine is

```sh
Linux e024b496d020 5.0.0-25-generic #26~18.04.1-Ubuntu SMP Thu Aug 1 13:51:02 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
```

:grey_question: If you viewed the introductory video, you should know why this happens. If not, what is your educated guess?

The container can be removed with `docker container rm <name>`. If you have a container running in the background, like with -d, it can be stopped or killed using `stop` or `kill` instead of `rm`.
Running containers can be listed with `docker ps` or `docker container ls`. To show also inactive containers, add the -a flag.
You can find more information on the docker 

[1]: <https://docs.docker.com/engine/reference/builder/> "Docker Builder reference"   
[2]: <https://www.linuxjournal.com/content/everything-you-need-know-about-linux-containers-part-i-linux-control-groups-and-process> "Linux Control groups and process"
[3]: <https://docs.docker.com/v17.09/engine/reference/run/> "Docker run reference"
