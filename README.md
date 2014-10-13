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
12. [Lab 5: Run a More Complicated Container](#lab4)
13. [Unit 6: Image and Container Management](#unit6)
14. [Lab 6: Cleanup!](#unit7)

<a name='unit0'></a>
## What Is Docker?

<a name='lab0'></a>
## Lab 0 - Creating a personal Linux VM and Installing Docker

1. Using a web browser, go to *https://vm-manage.oit.duke.edu*
2. Login using your Duke NetId.
3. Create a new project for this class.
4. Select *Ubuntu 14 Basic* for the Server.

The vm-manage web page will tell you the name for your VM. The web site will also tell you the initial username and password. You should connect via ssh.

Example: `ssh bitnami@colab-sbx-87.oit.duke.edu`

5. Once logged in via ssh, enter the `passwd` command to set a unique password.

Example:

    passwd
    Changing password for bitnami.
    (current) UNIX password:
    Enter new UNIX password:
    Retype new UNIX password:

6. Install the Docker package: `sudo apt-get install docker.io`
7. Add your user to the Docker group: `sudo usermod -aG docker bitnami`
8. Logout and back in
9. Verify you are now in the Docker group:

Example:

    groups
    bitnami adm cdrom dip plugdev lpadmin sambashare docker
 
<a name='unit1'></a>
## Unit 1: Docker Images

At it's core, Docker creates **containers** from **images**.

**Containers** are the active, running parts of Docker that *do* something.

**Images** are pre-built environments and instructions that tell a container *what* to do.

We will come back to containers later.  For now, let's focus on images.

Images are based on layers, built up from the bottom, or *base image*.  The base image is a basic, stripped-down OS, like Ubuntu or CentOS.  Every change becomes another layer that is applied to the layer below it, and creates an *intermediate image*.

Example:


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

Example:

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

Example:

    docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE

There's not much to see there, is it?  That's because there are no images installed yet.

Base images are maintained by a few groups in the Docker community, and are stored in the **Docker Registry** - a registry of community-contributed images hosted at www.Docker.com.  Docker can pull all of these images with a simple command: `docker pull`

Example - download the CentOS base image

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

In order to build a Docker image, you need at least 1 file, called a **Dockerfile**.  A Dockerfile consists of lines of instructions that make up each of the layers in your final image.  Let's look at a simple Dockerfile:

    # This is a basic "Hello World" image, for 
    # Linux@Duke's Introduction to Docker: Do Your Own Docker class

    FROM centos:centos7
    MAINTAINER Chris Collins <collins.christopher@gmail.com>

    CMD [ "/bin/echo", "Hello, World" ]

That's it!  Pretty simple.   Let's break it down a little:

The first two lines, the ones beginning with "#" - are just comments.  They're not required, but it's nice to put some identifying information in there so you, or anyone you share your Dockerfile with, has an idea of what to expect.

The `FROM` line specifies which base image your image is built on.  It's generally a good idea to be specific here.

The `MAINTAINER` line specifies who created and maintains the image.  This should always be you or your group, if you're working with others on the image.  I like to add my email address, as well, in case anyone has any questions about the image.

The From and Maintainer lines are the only required lines in an image.

The last line, `CMD`, specifies the command to run immediately when a container is started from this image, unless you specify a different command.  In this case, the container started from this image will echo "Hello, World" and then exit.

To build the "helloworld" image, you need the Dockerfile.  You can write your own, or copy-and-paste it, but the easiest way is to clone it from a Github repository.  Obviously, you'll need **git** installed to do this (`apt-get install git`).

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

Alright, so what did we do here?  You should be familiar with Git already, so the git command `git clone https://github.com/LinuxAtDuke/Intro-To-Docker.git` cloned the contents of the LinuxAtDuke Intro-To-Docker repositoy to your local disk.

After inspecting the Dockerfile to make sure it's OK (remember the Note about Security, above?), we then ran `docker build -t helloworld .`.

The `docker build` command tells Docker we're building an image from a Dockerfile.  The `-t helloworld` command tells Docker to tag the resulting image with the name "helloworld" (Otherwise, it would only have the IMAGE ID - a random string of characters).  Finally, the trailing `.` (period) tells Docker to use the Dockerfile from inside the current directory.

<a name='lab2'></a>
## Lab 2 - Building the FSM Image

There are many, many other commands that can go into a Dockerfile.  Let's go download the source for an image written by someone else, and examine it.

1. Clone this GitHub Repo: https://github.com/DockerDemos/FullScreenMario.git
2. Inspect the Dockerfile.  Is there anything new in there?  Take a guess at what any new things are doing.
3. For the purposes of this class, I am confirming you can trust this Dockerfile (SECURITY!)
4. Build the image from this Dockerfile.
5. Look at your image list to confirm it's there.

<a name='unit3'></a>
## Unit 3: Running Containers from Images

Up to this point, we've only downloaded or created images, but so far, we haven't done anything with them.  Let's look at running a container from the images we've been working with.

The `docker run` command is used to run containers.  At it's most basic, you need only two flags to run a container: `docker run -d <image>`  In this case, you're telling the container to run in "detached" mode (in the background), and to run the image specified.  You will usually use more flags and probably a command.  Let's look at these:

Some basic "docker run" flags:

    -d		Detached mode: run the container in the backgroup (opposite of -i -t)
    -i		Interactive (usually used with -t)
    -t		TTY: Allocate a pseud-TTY (basically a terminal interface)
    -p		Publish Ports: -p <host port>:<container port>

These are probably the most common options.  Using the `-i` and `-t` flags (or `-it`) will attach you to a terminal inside the container.  Using the `-d` flag will run the container in the background, instead.

The `-p` flag lets you map ports from the container to your host, allowing you to access that port in the container from the outside, as you would with any other Linux host.

Example:

Let's run the "helloworld" container.

    docker run -it helloworld
    Hello, World

In this example, attached to the container, and it ran the command specified in it's Dockerfile (/bin/echo Hello, world).

Another important part of the `docker run` commmand is the command you tell the container to run.  In the "helloworld" image, there is a command specified in the Dockerfile.  However, we can override that command when we start the container.

Example:

    docker run -it helloworld /bin/echo Goodbye, world!
    Goodbye, world!

In the example above, we ran the command "/bin/echo Goodbye, world!" instead of the command in the Dockerfile.  If you specify a command, it **always** comes last, and after the image name.


<a name='lab3'></a>
## Lab 3 - Running the FSM container 

In this lab, you're going to run the Full Screen Mario container you built in the last lab.

1. Examine your images, and note what you called the Full Screen Mario image.
2. Run the Full Screen Mario image, with the following options:
    * Detached
    * Map your VMs port 8080 to the container's port 80
    * No command is necessary (Look at the Dockerfile - can you tell why?)
3. Open a browser and navigate to your VM, port 8080: http://\<YOUR VM NAME\>:8080

---

![Creative Commons CC0 1.0 License](http://i.creativecommons.org/p/zero/1.0/88x31.png)

To the extent possible under law, [Linux@Duke](https://github.com/LinuxAtDuke) has waived all copyright and related or neighboring rights to *Introduction To Docker*.  This work published from: United States.

