Running Docker on the Mac OSX
-

Docker does not run natively on Mac OSX. Fortunately, the docker team has created
[Docker For Mac](https://docs.docker.com/docker-for-mac/) to get docker running
on a mac using the xhyve hypervisor.  It runs almost as if it were native. It does
not get hosed by connections to the VPN. It can mount all host directories as
volumes, and attaches ports exposed by containers to localhost.

If you had previously installed the Docker Toolbox, which uses boot2docker,
[this guide](https://docs.docker.com/docker-for-mac/docker-toolbox/) contains helpful
advice for migrating from Docker Toolbox to Docker for Mac, or allowing them to
coexist.

Once you have docker installed, you should be able to run the following from your Terminal application
```bash
$ docker run -ti --rm alpine echo "HELLO WORLD"
```

replace alpine with any of the official distros available on the [Docker Hub](https://registry.hub.docker.com/)
