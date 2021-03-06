# 1 - Building images and creating containers

If you watched the tutorial video, you know that there are two primary components for docker: images and containers. An image contains the software run in a container; esentially, every container needs some minimal image to run something.
There are essentially two ways to create an image:

* Via an interactive prompt and commit
* Via a build script that details all the steps, a [Dockerfile][1]

We will see in the following how to create them using both approaches.

## The image and its use

Let's start with the basics, image: Ubuntu. We will download a ready image for an Ubuntu environment and play a bit with it. 
Depending on your system settings, you may need additional privileges to access control groups ([CGroups][2]), a feature used in Docker.  Thus for all `docker` commands, you might need either a preceding `sudo` or root login (MacOs/Linux/Unix).
So, let's pull the image. Open the Docker console, enter a new folder or create one, and type:

```sh
docker pull ubuntu:latest
```

This loads an Ubuntu image from the default container registry (hub.docker.com). The text after the colon defines the version or *tag* of the image. `latest` stands for the newest release that is *safe* to use. If `:latest` is omitted, it is chosen by default.

:grey_question: Which other releases are there? Name some tags

:information_source: Hint\: you might want to check the [hub](https://hub.docker.com)
 
If you look up the hub's entries, you can see two main groups, official and user/community maintained. The name tells the whole story. 
We now have the `latest`, which seems not the actual latest, but more the *stable*. Indeed, it's the LTS (long term support) version, thus the safest for production code. For development and new features, you might want to use the *:rolling* version.

If you try the pull command again with another tag, the download will start again. Typing `docker images`, we now get the list of available images. (example with `:rolling`)

```sh
florian@helius:~/Public/Projects/docker_tutorial/#1_Container$ sudo docker images
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
ubuntu                  rolling             9f3d7c446553        1 week ago          70MB
ubuntu                  latest              a2a15febcdf3        1 week ago          64.2MB
```
Note the compact size of the images.
In alternative to tags, one can also use the *digest*. It is a unique identifier for the image and might be needed if no tag has been given to an image. The IMAGE_ID in the example represents the short notation of the UUID. Thus the same images can also be pulled using, e.g., `ubuntu@a2a15febcdf3` as image name, where `@` tells that we are looking for a digest UUID.

These images give us the grounds we need to run containers. If we run one of the images in a container, we create a new process, user, and mount space. As discussed in the introduction, this is an isolation concept close to full virtualization. We will see later on what the differences are.

Let's try our image

```sh
docker run --name cont1 -it ubuntu:latest 
```

This starts a new container instance with the name `cont1` and interactively (-it) connects to it. The container automatically exits once it has nothing to do anymore, i.e., if you start a container to run a program, it stops again as soon as the program terminates.

:information_source: -i = always attach to container,  -t create TTY, together -it. Alternatively, -d stands for detach, thus running in the background. We will see later what it means. More info in the [run reference][3]

If you type `exit`, you terminate the prompt return back to the console. You did not specify any command, but any container has a default execution or command entry. We can verify that by typing `docker inspect <image-name>`.
For `ubuntu:latest`, this gives a JSON format output containing the following.

```sh
            "Cmd": [
                "/bin/sh",
                "-c",
                "#(nop) ",
                "CMD [\"/bin/bash\"]"
            ],
 ```

This means for this image is actually the same as 
```sh
docker run --name cont1 -it ubuntu:latest bash 
```
You can put any command instead ( we can omit '/bin/' as specified in the environment variable path). The command could also be as simple as `echo "Hello world"`, just printing the line to the console and exiting. If no `--name <name>` parameter is given, docker chooses a random name for the container. As for images, containers are also distinguished with digest UUIDs.

:grey_question: Why do you think you can't rerun the container, even with the same parameters? (check at the end of the chapter for a hint)

### Isolation ??

Without deleting any container, we can start our `cont1` again by typing `docker container start -i cont1`, and we are back in the bash. ( here -i stands for interactive )
Here now the curious thing, the actual difference to virtualization.
In the bash, type `uname -a`. You should have a different output for each *version* of your systems. mine is

```sh
Linux e024b496d020 5.0.0-25-generic #26~18.04.1-Ubuntu SMP Thu Aug 1 13:51:02 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
```

:grey_question: If you viewed the introductory video, you should know why this happens. If not, what is your educated guess?

### Processes and runtime

`ps` shows you all the processes running. The separated process space isolates it from your host, so you will not see any of the other processes running on your computer. This is particularly useful if you want to separate and manage small services. Let's call it micromanagement.
Typing `ps`, you should see the following.

```sh
root@e024b496d020:/# ps 
  PID TTY          TIME CMD
    1 pts/0    00:00:00 bash
   34 pts/0    00:00:00 ps
```
The container's main process, the process with number 1, is bash, the command shell, our main/default command. Like said before, the container exits as soon as the process terminates. To demonstrate the effect, try this command while in the container: `for i in {1..10}; do echo "Hello $i"; sleep 1; done &`.
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
In fact, it returns a newly launched process number, 10 in my case, and executes it in the background. As long as `bash` is alive, the new process runs. If you rerun it but type `exit` before all lines are print, the result is different. Try to come up with an explanation. 

:information_source: If you use programs such as a web server or a database management system, those run daemons remain active until terminated by event or user.

:grey_question: What does this mean when you are dealing with containers in your future environment?


The container can be removed with `docker container rm <name>`. If you have a container running in the background, like with -d, it can be stopped or killed using `stop` or `kill` instead of `rm`.
Running containers can be listed with `docker ps` or `docker container ls`. To show also inactive containers, add the -a flag.
You can find more information on the docker website, [command reference docker stable][3].

### Summary

So far, we have seen that there are images and containers. Both are identified by names and by an id, UUID, long or short. Containers have an isolation concept that focuses on simplicity; thus, it is common practice to run a single service or program per container. They are ephemeral, which means that runtime information is not stored if containers are replaced with a newer image version.

This brings advantages and some disadvantages. 

:grey_question: what would you say are those? Any idea what the limits of these containers might be?

## Customizing the container

The container we created, `cont1`, is a running instance of the image `ubuntu:latest`, but it can also be changed to our needs. We can install software and alter the configuration to what suits best to us. For this example, we will use the small Python program in this folder. Download the `.py` file and put it into your working directory.

Using the interactive mode, we can enter the container and perform all the changes. This time, instead of starting and stopping the container every time, we just attach and detach it to our active console (terminal).
So, let's start the container again.

```sh
docker start -i cont1
```

We now install `Python` to run the little calculator demo program inside the container. Thus, type `apt-get update` to update the list of available software, then `apt-get install python`. It will prompt you if you want to install the packets.
Now, we have a running environment for python programs inside the container. If we need more libraries, we can also add software like `pip` to manage Python. However, we are fine for now.

The next step is to bring the app into the container. Instead of typing `exit`, we use `CTRL-p` `CTRL-q` keys if we need to return to the console. This will leave the container running, e.g., continue installing software, etc.
To copy files to and from the container, we can use `docker cp`, which works like the Unix cp. If the source or destination is a container, the path is preceded by `<containername>:`.
Let's copy the app. (verify the file is in your actual working directory)

```sh
docker cp calc.py cont1:/home
```

If we need to return to the container, we can use `docker attach cont1`, and we are back in the container.
Let's run our program:

```sh
cd /home
python calc.py
```

Done. What if we did a lot of work and want to make these changes permanent, like an image?

:grey_question: What are the advantages of having it as an image now?

The container, running or not, can be saved as an image by using the commit command. We can do that with

```sh
docker commit cont1 <imagename>
```
The image name is optional, but helps to organize things. Moreover, images are incremental. This means only changes to the previous image (here ubuntu:latest) are stored. If you update your container and then commit again with the same name, the image will have a new version on top of `ubuntu:latest`. This allows to incrementally update while saving space. Only changes to the original image are saved. 

You can observe these details by `inspecting the two images. You should get something like the following for the container image.

```sh
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:6cebf3abed5fac58d2e792ce8461454e92c245d5312c42118f02e231a73b317f",
                "sha256:f7eae43028b334123c3a1d778f7bdf9783bbe651c8b15371df0120fd13ec35c5",
                "sha256:7beb13bce073c21c9ee608acb13c7e851845245dc76ce81b418fdf580c45076b",
                "sha256:122be11ab4a29e554786b4a1ec4764dd55656b59d6228a0a3de78eaf5c1f226c",
                "sha256:a418b64d6e82c4635231fbb41da0788417c671cc631b221db5363be1aa4128af"
            ]
        },
```



## Using Dockerfiles

Sometimes these process tends to be laborious. Also, if you have to repeat it because of some minor changes, the manual and interactive approach gets too tedious. Thus, there is another way to create custom images: the Dockerfile. 

A Dockerfile contains all the steps needed to create the desired image and does not require any manual intervention (if done right).
Thus, let's try to create one for the previous python app example.

For this approach, we don't need to get an image. Previously we needed the `ubuntu:latest` image and had to download it manually. In this case, we just specify exactly this, and docker does the rest. In a new empty file called Dockerfile, add this line:

```sh
FROM ubuntu:latest
```
 
The next thing we did, is to install Python. The `RUN` command defines what to execute inside the container. It can also be multiple lines with an `\` to escape at the end of the previous line. Let's add the installation commands we used

```sh
RUN apt-get update && \ 
	apt-get install -y python
```

(-y so it doesn't prompt)

Now, create a directory, again with `RUN`, and copy the app's file with `COPY`. The command `WORKDIR` comes in handy to define a working directory and avoid specifying the path again all the time.

```sh
RUN mkdir /home/myapp
WORKDIR /home/myapp
COPY calc.py .
```

There is also an `ADD` command that is very similar to `COPY`, but it also features automatic extraction of archives, etc.
Lastly, we have to define our default behavior, what happens if somebody starts the container. We would like to run our calculator, so let's put it in with `CMD`.

```sh
CMD ["python", "calc.py"]
```

If somebody runs this image in a container, our calculating app is launched by default.
If we have to set an environment variable, let's say the executable search path `$PATH`, we can set that with `ENV`. 

The final step, build the image. Just run `docker build .` to compile the image. With the flag -t, you can specify an optional image name.
The result should be something like `Successfully built 3c7a3b8cbbd6`.
If you now run it as a new container, either with the name you specified (optional via -t) or the resulting UUID, you get the calculator running: `docker run -it 3c7a3b8cbbd6`.

See [Builder reference][1] for more options.

Optionally, now you can login to your docker hub account with `docker login` and then push the container image to the repository with `docker push <imagename>`. (you need an account for that..)

## Resources

Once we have the container running, we might require a set of resources. Some need connectivity or file access outside of the container. Ex. a web server needs to expose a port but also to access the page files. If the files are stored inside the container, it gets hard to change and update a web page.
These are resources that have to be set before running a container.

There are also other resources. A container uses memory and CPU, which are claimed from the host running the container. Through CGroups, we can also set and limit the number of resources assigned to the container.
These are resources that can be set with existing containers.

:grey_question: What are the risks if there is no limit to containers?


Returning to resource type 1, we can set these in the image file or at creation. A port for a web client can e.g., be set through the line `EXPOSE 80` and expose port 80. This sets the default port on the image side and makes it available on the configured network adapter. By default, Docker creates a virtual network with its own IP range.

The port is then accessible in this network. As an example, let's try a default apache2 web server image. 

```sh
docker run  httpd:latest
```
This downloads the latest image and runs it locally. If you check the [Dockerfile of httpd][7], you can see that it contains `EXPOSE 80`. This tells us that this port is now available in the internal network. If you run it, the output of the console shows you the assigned IP of the web-server. In my case, `172.17.0.2`. Putting this address in a Browser should give you the "It works" page.

Now, this runs a simple web server, and we have no connection to the outside world yet. The port is accessible only for us. If we want to map it to be reachable from the outside, we can attach the port to the host with the -p flag. The use it as `-p <host>:<container>`. Running the following makes the web-server also available at your computer's IP. 

```sh
docker run -p 80:80 httpd:latest
```

Now try typing `localhost` in a browser window.

If we want to access files on our computer or share them between containers, we need volumes. A use case could be the web pages of apache, which we want to change from outside, or the program `calc.py` as we don't want to copy it in the image and recompile all the code the time.

A mount point to a volume can be set, for example, in a docker file with `VOLUME`. On Unix/Linux systems, this creates a folder in the file system reflecting the container. If we create a volume `/myvolume`, you will have this resource in the container and in your host. Unfortunately, Windows uses virtualization, and this doesn't work right out of the box.

Another way is to use binding. Let's say we would like to use the file we have on disk for the calculator to reflect changes when we add new features. 

:information_source: In Windows, you have to enable shared drives first. See [this][6] short guide 

```sh
docker run --name cont2 -v "$PWD":/home/myapp -it 981dd3fd5830
```
$PWD in Unix environments is the current working directory. You can also use ".". For fol Windows, use forward slash `/` instead of `\` as it might make some issues.

Now, every time we start	`cont2`, the container accesses the actual file on disk. If you try to change it, you will see the program's changes if you restart it.
Volumes support different modes, including remote drives, etc. This direct connection (bind) is only one mode.
A volume might also just be mounted in one container and serve as closed external storage.
Like images and containers, volumes can be created and managed in the docker command line with the subcommand `volume`. Try it out. 
Additional configurations are described in the [volumes][4] reference.

:grey_question: Why should a container have a volume that is exclusive and not accessible? What might be the purpose?

Type 2 resources of a container can be changed at runtime. We can set them with `docker run` at creation or with `docker update` later on. 
For example let's try to set the CPU assigned to `cont2`.

```sh
docker update --cpu-quota 1000 --cpu-period 1000000 cont2
```

This sets the amount to 1\% of the total assigned CPU. Try it out a bit yourself. Change the resources and see how it reacts.

## Users and privileges

It is common practice to create users and limit access and privileges of containers. One thing is that a container by default has only one user, `root`. Thus, to restrict access, the finished image should use an unprivileged user that runs the software. For proper access management, commands like `ADD` and `COPY` foresee flags to change a file's ownership. More details can be found in the [builder refernece][1].

As we have seen previously, a container shares the kernel with the host (or Virtual machine in the case of Windows). To limit the container's host access, all containers have no advanced privileges by default. Thus, capabilities such as changing a task's priority or scheduling policy need particular privileges. These privileges are given with the `--cap-add=` parameter at container run. Example:

```sh
docker run --cap-add=SYS_NICE mycontainer
```

Adds the capability to change scheduling and priority of tasks. Also, default capabilities such as changing the ownership of access to files can be dropped with `--cap-drop=`. There is a `--privileged` flag that enables full access, but it's highly discouraged.
More details on capabilities and allowance can be found in the [run reference][3].

## Summary

In this tutorial, we saw basic interaction with containers and their images. We created and used containers, then we customized them and saved them for future use. 
Finally, we quickly saw how to change a container's resources.


[1]: <https://docs.docker.com/engine/reference/builder/> "Docker Builder reference"   
[2]: <https://www.linuxjournal.com/content/everything-you-need-know-about-linux-containers-part-i-linux-control-groups-and-process> "Linux Control groups and process"
[3]: <https://docs.docker.com/v17.09/engine/reference/run/> "Docker run reference"
[4]: <https://docs.docker.com/storage/> "Volumes and Storage"
[5]: <https://stackoverflow.com/questions/28302178/how-can-i-add-a-volume-to-an-existing-docker-container> "Adding a volume to an existing container"
[6]: <https://rominirani.com/docker-on-windows-mounting-host-directories-d96f3f056a2c> "Shared volumes on Windows"
[7]: <https://hub.docker.com/_/httpd> "Apache2 image on Hub-Docker"
