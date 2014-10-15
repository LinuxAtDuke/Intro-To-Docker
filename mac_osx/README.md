Running Docker on the Mac OSX
-

Docker does not run natively on Mac OSX.  To work with Docker on the Mac, you have to run a Virtual Machine Image of
Linux with the Docker host server.  To make this easier, boot2docker has been developed.  This is an almost transparent
installation that allows you to interact with and create docker images and containers from Terminal.

####Installing Boot2docker
Follow the following [instructions](https://docs.docker.com/installation/mac) to get Boot2docker installed on your system.
This installs the following items:
* a boot2docker virtual machine running in VirtualBox
* the docker commandline application which runs natively on the mac in your terminal, and interfaces with the $DOCKER_HOST over tcp
 (that is set up with the shellinit boot2docker command below)

Once you have boot2docker installed, you should be able to run the following from your Terminal application
```bash
$ boot2docker init
$ boot2docker up
$ $(boot2docker shellinit)
$ docker run -ti --rm centos:centos6 echo "HELLO WORLD"
```

replace centos:centos6 with any of the official distros available on the [Docker Hub](https://registry.hub.docker.com/)

###What do you mean 'Almost'
I qualified 'seemless' because the boot2docker virtual machine go between introduces some extra roadblocks that have to
be surmounted, compared to running on a linux host.

#####Networking
The DOCKER_HOST tcp layer must interact with the docker server hosted by the boot2docker-vm running in VirtualBox.  This
network layer is based on a host-only NAT network bridge that boot2docker sets up when it starts.  By default, this prevents you
from interacting a network hosted application container using localhost.  Try it:
```bash
$ docker run -d -p 80:8800 --name web nginx
$ curl http://localhost:8800
```

Instead, run the following to find out the IP of your NAT:
```bash
$ boot2docker ip 2> /dev/null
192.168.59.103
```

and use it instead of localhost, or just tie these together using
```bash
$ curl $(boot2docker ip 2> /dev/null):8800
```

####Mounting Host Directories as Volumes
This is a bit more technically challenging to fix, but not impossible.  The problem is that the default boot2docker-vm VirtualMachine image
installed with your boot2docker installation does not contain the VirtualBox Guest Additions.  This prevents you from mounting your directories
on the boot2docker-vm Virtual Machine image.  Because your directories cannot be mounted on the VM instance, they cannot then be
hosted on the docker container running on the VM instance, since they are using the host VM filesystem and not your mac filesystem.

Thus, when you run something like the following:
```bash
$ docker run -ti --rm -v /Users/foo_user/foo:/foo centos:centos6 /bin/bash
centos> ls /foo
```

foo is empty!

Thankfully, someone created both a version of the boot2docker image with the Guest Additions:
```bash
$ mv ~/.boot2docker/boot2docker.iso ~/.boot2docker/boot2docker.iso.bak
$ curl http://static.dockerfiles.io/boot2docker-v1.2.0-virtualbox-guest-additions-v4.3.14.iso > ~/.boot2docker/boot2docker.iso
```

And, for the future when a new version of boot2docker needs to be released, a [docker container application](https://gist.github.com/mattes/2d0ffd027cb16571895c)
that can install this on the default boot2docker image.  Once you do one of the above, you must then mount your /Users directory on the boot2docker-vm
```bash
$ boot2docker stop
$ VBoxManage sharedfolder add boot2docker-vm -name home -hostpath /Users
$ boot2docker up
```

Then you can mount volumes relative to /Users in your containers.

####Cisco Anyconnect VPN is not your friend
VPN providers can instruct the VPN client to route all network traffic through the VPN network.  This will prevent your docker
cli from interfacing with the Host Only NAT bridge.  Even a command as simple as:
```bash
$ docker images
```

will hang.  To get around this, you have to add port forwards to the boot2docker-vm image with at least the docker port 2375, and also
any other ports that you will need to forward (e.g. any port created by running a network service container app).
```bash
$ boot2docker stop
$ VBoxManage modifyvm "boot2docker-vm" --natpf1 "docker,tcp,127.0.0.1,2375,,2375"
$ VBoxManage modifyvm "boot2docker-vm" --natpf1 "rails,tcp,127.0.0.1,2375,,3000"
$ boot2docker up
$ export DOCKER_HOST='tcp://127.0.0.1:2375'
$ docker images
```

Note, this can cause a headache when you want to allow docker to provision the port for your
containers using the -P run flag.  You will need to determine the port assigned to docker, and,
stop boot2docker, forward that port, start it again.  It might be easier to explicitly set the forward
for the docker container using the -p run flag.

#####References
* (http://viget.com/extend/how-to-use-docker-on-os-x-the-missing-guide)
