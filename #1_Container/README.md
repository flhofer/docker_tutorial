# 1 - Building images and creating containers

There are essentialy two ways to ways to create an image:

* Via an interactive prompt and commit
* Via a build script that details all the steps, a [Dockerfile][1]


## The imange and its use

Let's start with basics. Image: Ubuntu.

You need privileges to access control groups ([CGroups][2]), thus docker might need `sudo` or root access

```sh
docker pull ubuntu:latest
```

This loads an Ubuntu image from the default registry (hub.docker.com). The text after the colon defines the version, or *tag* of the image. `latest` stands for the newest release that is *safe* to use. If `:latest` is omitted, it is chosen by default.

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
In alternative to tags, one can also use the *digest*. It is a unique identifier for the image and might be needed if no tag has been given to an image. The IMAGE_ID in the example represents the short notation of the UUID. Thus the same imanges can also be pulled using `ubuntu@a2a15febcdf3` as image name, where `@` tells that we are looking for a digest UUID.

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
where you can put any command instead ( we can omit '/bin/' as it is specified in the environment variable path). The command could also be as simple as `echo "Hello world"`, just printing the line to the console and exiting. If no `--name <name>` parameter is given, docker choses a random name for the container. As for images, containers are also distinguished with digest UUIDs.

:grey_question: Why do you think, you can't run the conainer again, even with the same parameters? (check at the end of the chapter for a hint)

### Isolation ??

Without deleting any container, we can start our `cont1` again by typing `docker container start -i cont1`, and we are back in the bash. ( here -i stands for interactive )
Here now the curious thing, the actual difference to a virtualization.
In the bash, type `uname -a`. You should have a different output for each *version* of your systems. mine is

```sh
Linux e024b496d020 5.0.0-25-generic #26~18.04.1-Ubuntu SMP Thu Aug 1 13:51:02 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
```

:grey_question: If you viewed the introductory video, you should know why this happens. If not, what is your educated guess?

### Processes and runtime

`ps` shows you all the processes running. The separated process space makes isolates it from your host, so you will not see any of the other processes running on your computer. This is particularly useful if you want to separate and manage small services. Let's call it micro management.
Typing `ps` you should see the following

```sh
root@e024b496d020:/# ps 
  PID TTY          TIME CMD
    1 pts/0    00:00:00 bash
   34 pts/0    00:00:00 ps
```
The main process of the container, the process with number 1, is bash, the command shell, our main/default command. Like said before, the container exits as soon as the process terminates. To demonstrate the effect, try this command while in the container: `for i in {1..10}; do echo "Hello $i"; sleep 1; done &`.
It should list you a bunch of *Hello* lines, one every second and then end.

```sh
[1] 10
root@25d340668cd7:/# Hello 1
Hello 2
Hello 3
Hello 4
Hello 5
Hello 6
Hello 7
Hello 8
Hello 9
Hello 10
```
In fact, it returns a new launched process number, 10 in my case, and executes it in the background. As long as `bash` is alive, the new process runs. if you run it again, but type `exit` before all lines are print, the result is different. Try to come up with an explanation. 

:information_source: If you use programs such as a web server or a database management system, those run daemons which remain active until terminated by event or user.

:grey_question: What does thsis mean when you are dealing with containers in your future environment?


The container can be removed with `docker container rm <name>`. If you have a container running in the background, like with -d, it can be stopped or killed using `stop` or `kill` instead of `rm`.
Running containers can be listed with `docker ps` or `docker container ls`. To show also inactive containers, add the -a flag.
You can find more information on the docker website, [command reference docker stable][3].

### Summary

So far we have seen that there are images and containers. Both are identified by names and by a id, UUID, long or short. Containers have and isolation concept that focuses on simplicity, Thus it is common practice to run a single service or program per container. They are ephermeral, this means that runtime information is not stored if containers are replaced with a newer image version.

This brings advantages and some disadvantages. 

:grey_question: what would you say are those? Any idea what the limtis of these containers might be?

## Customizing the container

The container we created, `cont1`, is a running instance of the image `ubuntu:latest` but it can also be changed to our needs. We can install software and alter the configuation to what suites best to us.

Using the interactive mode, we can enter the container and perform all the changes. This time, instead of starting and stoping the container every time, we just attach and detach our console.
So, let's start the container again.

```sh
docker start -i cont1
```

We now install `Python` to run the little calculator demo program inside the container. Thus, type `apt-get update` to update the list of available software, then `apt-get install python`. It will propt you if you want to install the packets.
Now, we have a running environment for python programs inside the container. If we need more libraries, we can add also software like `pip` to manage Python. However, we are fine for now.

The next step is to bring the app into the container. Instead of typing `exit`, if we need to return to the console we use `CTRL-p` `CTRL-q` keys. This will leave the container running, e.g. continue installing software ecc.
To copy files to and from the container we can use `docker cp`, which works like the unix cp. If the source or destination is a container, the path is preceeded by `<containername>:`.
Let's copy the app.

```sh
docker cp calc.py cont1:/home
```

If then we need to return to the container, we can use `docker attach cont1` and we are back in the container.
Let's run our program:

```sh
cd /home
python calc.py
```

Done. Well, what if we did a lot of work and want to make this changes permanent, like as an image?

:grey_question: What are the advantages of having it as an image now?



## Using Dockerfiles




## Process resources



## Summary



[1]: <https://docs.docker.com/engine/reference/builder/> "Docker Builder reference"   
[2]: <https://www.linuxjournal.com/content/everything-you-need-know-about-linux-containers-part-i-linux-control-groups-and-process> "Linux Control groups and process"
[3]: <https://docs.docker.com/v17.09/engine/reference/run/> "Docker run reference"
