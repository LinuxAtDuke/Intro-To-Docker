Introduction to Docker: "Do Your Own Docker"
============================================

*Version 1.0 - 20141013*

**Instructor**

Chris Collins

**Table of Contents**

1. [What Is Docker?](#unit0)
2. [Lab 0: Creating a Personal Linux VM and Installing Docker](#lab0)
3. [Unit 1: Docker Images](#unit1)
4. [Lab 1: Download Some Images](#lab1)
5. [Unit 2: Building an Image from "Source"](#unit2)
6. [Lab 2: Building the FSM image](#lab2)
7. [Unit 3: Running Containers from Images](#unit3)
8. [Lab 3: Running the FSM container](#lab3)
9. [Unit 4: More Dockerfile Instructions ](#unit4)
10. [Lab 4: Examine a More Complicated Dockerfile](#lab4)
11. [Unit 5: More Docker Run Flags](#unit5)
12. [Lab 5: Run a More Complicated Container](#lab5)
13. [Unit 6: Image and Container Management](#unit6)
14. [Lab 6: Cleanup!](#lab6)
15. [Appendix 1: Running Docker on OSX]](#apx1)

<a name='unit0'></a>
## What Is Docker?

If you go to the Docker website, [https://docs.docker.com](https://docs.docker.com), you'll see a dozen paragraphs explaining what Docker can do and how different techie folks can use it.  It's worth a read, but when it comes down to it, Docker is really just **containers that run commands** and **images that make containers**.

Docker uses LXC (LinuX Containers) as a way to run multiple isolated Linux systems on a single host. This uses cgroups to isloate resources and namespaces to make sure each virtual system is as isolated as possible from the host and other containers.  LXC and cgroups are built into the Linux kernel, so you can do all of this without Docker, but Docker provides an nice, human-friendly interface for this technology.

You don't need to know all about the LXC or cgroups stuff to use Docker, but it's interesting stuff, if you're curious.  All you need to know for this class is that Docker containers are tiny, efficient virtual servers that run in their own bubble inside a host server.

*Further Reading:*

1. Docker: [https://www.docker.com](https://www.docker.com)
2. LXC: [https://en.wikipedia.org/wiki/LXC](https://en.wikipedia.org/wiki/LXC)
3. cgroups: [https://en.wikipedia.org/wiki/Cgroups](https://en.wikipedia.org/wiki/Cgroups)


<a name='lab0'></a>
## Lab 0: Creating a personal Linux VM and Installing Docker

1. Using a web browser, go to *https://vm-manage.oit.duke.edu*
2. Login using your Duke NetId.
3. Create a new project for this class.
4. Select *Ubuntu 14 Basic* for the Server.

The vm-manage web page will tell you the name for your VM. The web site will also tell you the initial username and password. You should connect via ssh.

*Example:* `ssh bitnami@colab-sbx-87.oit.duke.edu`

5. Once logged in via ssh, enter the `passwd` command to set a unique password.

*Example:*

    passwd
    Changing password for bitnami.
    (current) UNIX password:
    Enter new UNIX password:
    Retype new UNIX password:

6. Install the Docker package: `sudo apt-get install docker.io`
7. Add your user to the Docker group: `sudo usermod -aG docker bitnami`
8. Logout and back in
9. Verify you are now in the Docker group:

*Example:*

    groups
    bitnami adm cdrom dip plugdev lpadmin sambashare docker
 
<a name='unit1'></a>
## Unit 1: Docker Images

At it's core, Docker creates **containers** from **images**.

**Containers** are the active, running parts of Docker that *do* something.

**Images** are pre-built environments and instructions that tell a container *what* to do.

We will come back to containers later.  For now, let's focus on images.

Images are based on layers, built up from the bottom, or **base image**.  The base image is a basic, stripped-down OS, like Ubuntu or CentOS.  Every change becomes another layer that is applied to the layer below it, and creates an **intermediate image**.

*Example:*


                Base Image
                    |
                    |
             "apt-get update"               <--  Change
                    |
                    |
               511136ea3c5a                 <--  Intermediate Image
                    |
                    |
           "apt-get install apache2"        <-- Change
                    |
                    |
               f34473a3bf5b                 <--  Intermediate Image
                    |
                    |
     "touch /var/www/html/hello-world.htm"  <-- Change
                    |
                    |
               df3a29e82c0e  



Images can also be branched: You can apply a different set of changes to any layer, producing a new branch.

*Example:*

          Base Image
              |
              |
       "apt-get update"
              |
              |
         511136ea3c5a --------------------------- |
              |                                   |
              |                                   |
     "apt-get install apache2"    "apt-get install openssh-server" 	
              |                                   |
              |                                   |
         e793d90272b6                        b70ad18cfc2a


You can see what images you have on your system by using the `docker images` command:

*Example:*

    docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE

There's not much to see there, is it?  That's because there are no images installed yet.

Base images are maintained by a few groups in the Docker community, and are stored in the **Docker Registry** - a registry of community-contributed images hosted at www.Docker.com.  Docker can pull all of these images with a simple command: `docker pull`

*Example:* - download the CentOS base image

    docker pull centos
    68edf809afe7: Pulling dependent layers 
    87e5b6b3ccc1: Pulling dependent layers 
    87e5b6b3ccc1: Pulling metadata 
    87e5b6b3ccc1: Pulling fs layer 
    511136ea3c5a: Download complete 
    5b12ef8fd570: Download complete 
    87e5b6b3ccc1: Downloading   526 kB/78.11 MB 1m14s

    ...

    5b12ef8fd570: Download complete 

After you download the CentOS base image, you can examine your images again.

    docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
    centos              centos5             504a65221a38        11 days ago         467.1 MB
    centos              centos6             68edf809afe7        11 days ago         212.7 MB
    centos              centos7             87e5b6b3ccc1        11 days ago         224 MB
    centos              latest              87e5b6b3ccc1        11 days ago         224 MB

Notice that there are multiple images listed.  These are all part of the same download.  There are images for three different versions of CentOS - CentOS 5, CentOS 6 and CentOS 7, as well as "latest".

Notice that the "IMAGE ID" for "latest" is the same as "centos7".  The "latest" image is a **tag** that points to centos7.  This tag system allows you to specify `centos` or `centos:latest` when you use the image, and you'll always get the latest version.  Similarly, you can specify `centos:centos6`, etc, and you'll always get that specific version of the image.

### A Note About Security

The Docker repository hosts hundreds (maybe thousands) of images - very few of them are base images.  All of these images are developed by folks in the Docker community.  While the folks in the community make an effort to self-regulate each other, it is possible for an image hosted there to be written with some malicious intent.  Therefore,  you can trust the base images (ubuntu, centos, etc), and any official repositories (https://registry.hub.docker.com/), but should be careful to inspect other images you want to pull from the registry **before** you pull them.

<a name='lab1'></a>
## Lab 1: Download some images 

1. Download the "ubuntu" base image
2. Download the "wordpress" image, and official image
3. Examine your local images. Verify that you have the Ubuntu base image, and the WordPress image.
4. Look at the various WordPress images.  Are there any tags?  If so, what are they?

<a name='unit2'></a>
## Unit 2: Building an image from "source"

In addition to pulling an image from the registry, you can also build them from "source" (it's not the same as building a program from source code, but the idea is similar).

In order to build a Docker image, you need at least one file, called a **Dockerfile**.  A Dockerfile consists of lines of instructions that make up each of the layers in your final image.  Let's look at a simple Dockerfile:

    # This is a basic "Hello World" image, for 
    # Linux@Duke's Introduction to Docker: Do Your Own Docker class

    FROM centos:centos7
    MAINTAINER Chris Collins <collins.christopher@gmail.com>

    CMD [ "/bin/echo", "Hello, World" ]

That's it!  Pretty simple.   Let's break it down a little:

The first two lines, the ones beginning with "#" - are just comments.  They're not required, but it's nice to put some identifying information in there so you, or anyone you share your Dockerfile with, has an idea of what to expect.

The `FROM` instruction specifies which base image your image is built on.  It's generally a good idea to be specific here.

The `MAINTAINER` instruction specifies who created and maintains the image.  This should always be you or your group, if you're working with others on the image.  I like to add my email address as well, in case anyone has any questions about the image.  This is actually built into the image itself, too.

The From and Maintainer instruction are the only required lines in an image.

The last instruction, `CMD`, specifies the command to run immediately when a container is started from this image, unless you specify a different command.  In this case, the container started from this image will echo "Hello, World" and then exit.

To build the "helloworld" image, you need the Dockerfile.  You can write your own, or copy-and-paste it, but the easiest way is to clone it from a Github repository.  Obviously, you'll need **git** installed to do this (`sudo apt-get install git`).

    git clone https://github.com/LinuxAtDuke/Intro-To-Docker.git 
    cd Intro-To-Docker/helloworld
    cat Dockerfile

    # This is a basic "Hello World" image, for 
    # Linux@Duke's Introduction to Docker: Do Your Own Docker class

    FROM centos:centos7
    MAINTAINER Chris Collins <collins.christopher@gmail.com>

    CMD [ "/bin/echo", "Hello, World" ]

    docker build -t helloworld .

    Sending build context to Docker daemon  2.56 kB
    Sending build context to Docker daemon 
    Step 0 : FROM centos:centos7
     ---> 70214e5d0a90
    Step 1 : MAINTAINER Chris Collins <collins.christopher@gmail.com>
     ---> Using cache
     ---> dd140908e4b5
    Step 2 : CMD [ "/bin/echo", "Hello, World" ]
     ---> Running in ef9d4e02a356
     ---> c2d5fc2d6b0a
    Removing intermediate container ef9d4e02a356
    Successfully built c2d5fc2d6b0a

Alright, so what did we do here?  You should be familiar with git already, so the git command `git clone https://github.com/LinuxAtDuke/Intro-To-Docker.git` cloned the contents of the LinuxAtDuke Intro-To-Docker repositoy to your local disk.

After inspecting the Dockerfile to make sure it's OK (remember the *Note About Security*, above?), we then ran `docker build -t helloworld .`

The `docker build` command tells Docker we're building an image from a Dockerfile.  The `-t helloworld` command tells Docker to tag the resulting image with the name "helloworld" (Otherwise, it would only have the IMAGE ID - a random string of characters).  Finally, the trailing `.` (period) tells Docker to use the Dockerfile from inside the current directory.

<a name='lab2'></a>
## Lab 2: Building the FSM Image

There are many, many other commands that can go into a Dockerfile.  Let's go download the source for an image written by someone else, and examine it.

1. Clone this GitHub Repo: https://github.com/DockerDemos/FullScreenMario.git
2. Inspect the Dockerfile.  Is there anything new in there?  Take a guess at what any new things are doing.
3. For the purposes of this class, I am confirming you can trust this Dockerfile (SECURITY!)
4. Build the image from this Dockerfile.
5. Look at your image list to confirm it's there.

<a name='unit3'></a>
## Unit 3: Running Containers from Images

Up to this point, we've only downloaded or created images, but so far we haven't done anything with them.  Let's look at running a container from the images we've been working with.

The `docker run` command is used to run containers.  At it's most basic, you need only two flags to run a container: `docker run -d <image>`  In this case, you're telling the container to run in "detached" mode (in the background), and to run the image specified.  You will usually use more flags and probably a command.  Let's look at some of these:

*Some basic "docker run" flags:*

    -d		Detached mode: run the container in the backgroup (opposite of -i -t)
    -i		Interactive (usually used with -t)
    -t		TTY: Allocate a pseud-TTY (basically a terminal interface)
    -p		Publish Ports: -p <host port>:<container port>

These are probably the most common options.  Using the `-i` and `-t` flags (or `-it`) will attach you to a terminal inside the container.  Using the `-d` flag will run the container in the background, instead.

The `-p` flag lets you map ports from the container to your host, allowing you to access that port in the container from the outside, as you would with any other Linux host.

*Example:*

Let's run the "helloworld" container.

    docker run -it helloworld
    Hello, World

In this example we attached to the container, and it ran the command specified in it's Dockerfile (/bin/echo Hello, World).

Another important part of the `docker run` commmand is the command you tell the container to run.  In the "helloworld" image, there is a command specified in the Dockerfile.  However, we can override that command when we start the container.

*Example:*

    docker run -it helloworld /bin/echo Goodbye, world!
    Goodbye, world!

In the example above, we ran the command "/bin/echo Goodbye, world!" instead of the command in the Dockerfile.  If you specify a command, it *always* comes last, and after the image name.


<a name='lab3'></a>
## Lab 3: Running the FSM container 

In this lab, you're going to run a container from the Full Screen Mario image you built in the last lab.

1. Examine your images, and note what you called the Full Screen Mario image.
2. Run the Full Screen Mario image, with the following options:
    * Detached
    * Map your VM's port 8080 to the container's port 80
    * No command is necessary (Look at the Dockerfile - can you tell why?)
3. Open a browser and navigate to your VM, port 8080: http://*your_vm_name*:8080

<a name='unit4'></a>
## Unit 4: More Dockerfile Instructions 

Earlier, we learned about the FROM, MAINTAINER and CMD instructions, but there are more instructions available to use inside a Dockerfile.  Each instruction that is listed is saved as another layer, or intermediate image (remember those?) 

Here are a few more common instructions:

**ADD**

The ADD instruction will copy new files from a source and add them to the containers filesystem path.  The source is usually relative to the Dockerfile:

*Example:*

    ADD helloworld.txt  /srv/helloworld.txt

This copies a helloworld.txt file from the same directory as the Dockerfile into /srv/helloworld.txt inside the container.

The ADD instruction can also take a URL as the source:

*Example:*

    ADD https://upload.wikimedia.org/wikipedia/commons/e/ee/Grumpy_Cat_by_Gage_Skidmore.jpg /srv/grumpycat.jpg

This ADD instruction will copy the image of Grumpy Cat from Wikimedia and put it inside the container as /srv/grumpycat.jpg

The ADD instruction will also unpack an archive in a support format:

    ADD https://wordpress.org/latest.tar.gz /var/www/html

This ADD instruction will download and unpack the WordPress source code into /var/www/html inside the container.

**RUN**

The RUN instruction does just that:  It runs a command inside the container.

*Example:*

    RUN yum install -y git

This RUN instruction will rum `yum install -y git` inside the container, installing the git program.

**EXPOSE**

The EXPOSE instruction tells Docker that the container will listen on the specified port when it starts.

*Example:*

    EXPOSE 80
    EXPOSE 443

These two EXPOSE instructions tell Docker that the container will listen to the web ports (80 and 443) when it runs. 

**VOLUME**

The VOLUME instruction will create a mount point with the specified name and tell Docker that the volume may be mounted by the host, or other containers.

*Example:*

    VOLUME ["/var/www/html"]

This VOLUME instruction make the /var/www/html directory inside the contianer available to be mounted by the host, or other linked contianers.

<a name='lab4'></a>
## Lab 4: Examine a More Complicated Dockerfile

In this lab, you'll examine a Dockerfile that has a few of the commands used above, and then build the image.

1. You should have already cloned the Intro-To-Docker repo when you looked at the "helloworld" Dockerfile.  If you haven't:
  * `git clone git clone https://github.com/LinuxAtDuke/Intro-To-Docker.git`
2. Change directories to the "grumpycat" directory.
3. Examine the Dockerfile inside:
  * What does the ADD instruction do here?
  * What does the RUN instruction do here?
  * What does the EXPOSE instruction do here?
  * What does the VOLUME instruction do here?
  * What does the CMD instruction do here?
  * What do you think the container will do when you run it?
4. For the purposes of this class, I am confirming you can trust this Dockerfile (SECURITY!)
5. Build the grumpycat image from this Dockerfile.
6. Examine your image list to confirm the image is there.

<a name='unit5'></a>
## Unit 5: More Docker Run Flags

The `docker run` command has a lot of flags, and can get really complicated once you start naming and linking containers, mounting volumes back and fourth, etc. Here are a few more basic flags you will probably use regularly, though.

**--rm=true**

The `--rm=true` option will remove your container from the host when it stops running.  It is only available if you run your Docker container interactively (`-it`), and not in detached mode (`-d`).

**-v \<host volume\>:\<container volume\>**

The `-v` flag allows you to mount a volume from inside your container (that has been specified with the `VOLUME` instruction in the Dockerfile, remember) to a volume on your host.

*Example:*

`docker run -it -v /tmp/my-volume:/var/www/html centos touch /var/www/html/test.txt` will mount /tmp/my-volume on your host (creating it if it doesn't exist) into /var/www/html on the container.  The command (touch /var/www/html/test.txt) then creates a test.txt file, which you would be able to see in /tmp/my-volume.

**--name \<name\>**

The `--name` flag lets you assign a specific name to your container, rather than assigning a random one to it.  This helps when you want to link containers, or just to make them easier to identify.  You can only have one container with a specific name, though, even if it's no longer running.

*Example:*

`docker run -it --name MyAwesomeContainer centos /bin/echo Hello!`

**-P**

The `-P` flag is similar to the `-p` flag, but it publishes ALL ports on the container (that are specified by the EXPOSE instruction in the Dockerfile), and maps them to random ports on your host.

<a name='lab5'></a>
## Lab 5: Run a More Complicated Container 

1. Using the grumpycat image you built in the last lab, start a grumpycat container
  * Run it interactively with a pseudo-TTY (terminal)
  * Make sure it removes itself when it stops
  * Map port 8081 of your host VM to port 80 in the container
  * Mount ~/grumpy of your host VM as /var/www/html/grumpy
  * Name your new container "my_grumpy_cat"
  * Override the CMD line in the Dockerfile, and tell the contianer to run "/bin/bash" instead
2. What happend when you ran your contianer?
3. Inside the container, start the webserver: `/usr/sbin/httpd &`
4. Use a new command: CTRL+p CTRL+q (Control p, control q) to detach from your container, leaving it running.
5. Open a browser and navigate to your VM, port 8081: http://*your_vm_name*:8081
6. Copy Intro-To-Docker/grumpycat/doge.jpg into ~/grumpy
7. Open a browser and navigate to your VM, port 8081: http://*your_vm_name*:8081/doge

<a name='unit6'></a>
## Unit 6: Image and Container Management

This is where we get into some commands that will help you manage you images and containers, get more information about them, and remove them if you want.  As before, let's start with images.

You already know how to see your images, using the `docker images` command, and you know how to download new images from the Docker Registry with `docker pull`.  You also know how to build images with `docker build`.  Let's learn a few more:

**docker rmi \<name\>**

The `docker rmi` command removes (deletes) images.  You can specify the image by it's NAME, it's NAME:TAG, or it's IMAGE ID.

**docker tag \<image\> \<image\>:\<tagname\>**

We have talked about tagging before.  Tags are like symlinks, sort of.  You can tag an image with another name using `docker tag <image> <image>:<tagname>`.  This is usually used for version control, tagging "latest" to a particular build, and updating it when the build changes.  (Remember the base images, or WordPress?)

*Example:*

     docker images 
     REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
     fsm                 latest              06c25cda9581        3 hours ago         451.4 MB

     docker tag fsm fsm:2014
     docker images
     REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
     fsm                 latest              06c25cda9581        3 hours ago         451.4 MB
     fsm                 2014                06c25cda9581        3 hours ago         451.4 MB

**docker inspect \<image\> / docker inspect \<container\>**

The `docker inspect` command allows you to specify an image OR a container, and returns a JSON formatted list of attributes.  It's someone less userfriendly, but provides *a lot* of infomration about the image or container.  This is very useful  for troubleshooting (and is really useful if you ever get into using the Docker API).

*Example:*

    docker inspect 4c68cd64d2a 
    [{
    "Architecture": "amd64",
    "Author": "",
    "Comment": "",
    "Config": {
        "AttachStderr": false,
        "AttachStdin": false,
        "AttachStdout": false,
        "Cmd": [
            "apache2",
            "-DFOREGROUND"
        ],
        "CpuShares": 0,
        "Cpuset": "",
        "Domainname": "",
        "Entrypoint": [
            "/entrypoint.sh"
        ],
        "Env": [
            "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
     
    ...

Now we can move on to managing containers.  You already know how to run a container with `docker run`.

**docker ps**

The `docker ps` command shows a list of all the running containers on the host.  Here you can see the container ID, the image it was started from, the command it's running, when it was created, it's current status, any ports that are mapped, and any names that have been assigned.

*Example:*

    docker ps
    CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS                  NAMES
    c38e504f2716        grumpycat:latest    /usr/sbin/httpd -DFO   31 minutes ago      Up 31 minutes       0.0.0.0:8080->80/tcp   trusting_meitner    

**docker ps -a**

Adding the `-a` flag to `docker ps` will show you a list of *all* the containers on the host, including those that have stopped.

*Example:*

    docker ps -a
    CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS                      PORTS                  NAMES
    c38e504f2716        grumpycat:latest    /usr/sbin/httpd -DFO   34 minutes ago      Up 34 minutes               0.0.0.0:8080->80/tcp   trusting_meitner    
    6a3ccf4ca3a3        896447f597d6        /usr/sbin/httpd -DFO   35 minutes ago      Exited (0) 34 minutes ago                          naughty_torvalds    
    a2f4de40310e        63c840f565f3        /usr/sbin/httpd -DFO   43 minutes ago      Exited (0) 35 minutes ago                          happy_morse     

**docker stop \<container\>**

The `docker stop` command will stop a running container, gracefully sending a SIGTERM to the command that container is running.

**docker kill \<container\>**

The `docker kill` command is similar to `docker stop`, but it's more forceful, sending a SIGKILL to the command the container is running.  This should be avoided if possible - it can cause some strange issues with Docker.

**docker rm \<container\>**

The `docker rm` command is like the `docker rmi` command, and removes (deletes) a container.  This is good to do to containers you are no longer using, to clean up space.

**docker attach \<container\>**

The `docker attach` command will allow you to attach to a container that's already running, but not attached.  This is most useful when attaching to a container that has a TTY (terminal) allocated (such as when you run /bin/bash as the containers CMD).  It's less helpful when there's not TTY present.

**CTRL+p CTRL+q**

Typing `CTRL+p CTRL+q` while you are inside a running container (ie: one that you started interactively, or one that you've attached to) will detach you from that container, leaving it running.

**docker logs \<container\>**

The `docker logs` command will show you anything written to STDOUT by the command running inside your container.  Because of this, it's advantagous to send STDERR to STDOUT as well, if you're not already logging it somewhere that you can get to.

<a name='lab6'></a>
## Lab 6: Cleanup!

Using the commands you learned in Unit 6:

1. Show any running containers you have on your host
2. Stop all of the running containers
3. Show all the containers, including the stopped ones, on your host
4. Remove all the stopped containers
5. Run a new container, interactively, from the base image.  Make "/bin/bash" the command.
6. Detatch from the running container.
7. Inspect the running container.  What is it's IP Address?  What is it's Hostname?
8. Stop the running image, and remove it.
9. Show all of your images.
10. Tag your grumpycat image (or whatever you called it) with today's date
11. Remove all the Full Screen Mario images
12. Remove any images with the name \<none\>.  What do you think these are?

<a name='apx1'></a>
## Appendix 1: Running Docker on OSX

Right now, Docker runs primarily on Linux systems, but you can run Docker on Apple's OSX operating system using a virtual machine and [VirtualBox](https://www.virtualbox.org/).  Darin London [\(https://github.com/dmlond\)] has written a great guide to using Docker on OSX and graciously allowed it to be added to this class: [Running Docker on the Mac OSX](max_osx)

---

![Creative Commons CC0 1.0 License](http://i.creativecommons.org/p/zero/1.0/88x31.png)

To the extent possible under law, [Linux@Duke](https://github.com/LinuxAtDuke) has waived all copyright and related or neighboring rights to *Introduction To Docker*.  This work published from: United States.

