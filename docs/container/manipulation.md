If you read the output from our `hello world`, they even recommend what to try next.

```bash
docker container run -it ubuntu bash
```
Let's see what happens:

```bash
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
345e3491a907: Pull complete
57671312ef6f: Pull complete
5e9250ddb7d0: Pull complete
Digest: sha256:adf73ca014822ad8237623d388cedf4d5346aa72c270c5acc01431cc93e18e2d
Status: Downloaded newer image for ubuntu:latest
```

We are inside the docker container!

!!! tip
    You need to use the `-it` option whenever you want to run a container in interactive mode.
    - The `-i` or `--interactive` option connects you to the input stream of the container, so that you can send inputs to bash;
    - The `-t` or `--tty` option makes sure that you get some good formatting and a native terminal-like experience by allocating a pseudo-tty. 

### Playing with a running container 

This is a fully fledged Ubuntu host, and we can do anything we like in it. Let's explore it a bit, starting with asking for its hostname:

```bash
root@a5a2c01df566:/# hostname
a5a2c01df566
```

We can see that our container's hostname is the container ID. Let's have a look at the `/etc/hosts` file too.
```bash
root@a5a2c01df566:/# cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.2	a5a2c01df566
```
Docker has also added a host entry for our container with its IP address. Let's also check out its networking configuration.

```bash
root@a5a2c01df566:/# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
85: eth0@if86: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```
!!! warning "ip: command not found"
    Install the package `iproute2` that provides a collection of utilities for networking and traffic control.
    ```bash
       apt update && apt install -y iproute2
    ```

As we can see, we have the `lo` loopback interface and the `eth0@if86` network interface with an IP address of 172.17.0.2, just like any other host. 

We can also check its running processes:

```bash
root@a5a2c01df566:/# ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0   4216  3588 pts/0    Ss   08:46   0:00 bash
root         601  0.0  0.0   5896  2852 pts/0    R+   10:17   0:00 ps aux
```

Note that the process `bash` has PID 1. 

Now type `exit` or the `CTRL-d` key sequence...you'll return to the command prompt of your Ubuntu host. So what's happened to our container? 
The container only runs for as long as the command we specified, `/bin/bash`, is running. Once we exited the container, that command ended, and the container was stopped.

So the container still exists but it's stopped:
```bash
docker container ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED       STATUS                     PORTS     NAMES
a5a2c01df566   ubuntu    "bash"    2 hours ago   Exited (0) 6 seconds ago             festive_cerf
```
### Starting a stopped container
We can start again our stopped container with `docker container start <container-id or container-name>`:

```bash
docker container start a5a2c01df566
a5a2c01df566
```
Our container will restart with the same options we had specified when we launched it with the `docker run` command.

### Attaching to a container

The `docker container attach` command allows you to attach your terminal to the running container. 

!!! tip:
    The command that is executed when starting a container is specified using the ENTRYPOINT and/or CMD instruction in the Dockerfile.
    The `attach` command allows you to connect and interact with the container’s main process which has `PID 1`.
    Remember that **if you kill the main process the container will terminate**.

This is useful when you want to see what is written in the standard output in real-time, or to control the process interactively.

So running the `attach` command on our Ubuntu container will bring us back to our bash prompt:
```bash
docker container attach a5a2c01df566
root@a5a2c01df566:/#
```

You can detach from a container and leave it running using the `CTRL-p CTRL-q` key sequence.

What happens if you type `exit`?

### Getting a shell to a container

The `docker exec` command allows you to run commands inside a running container.
The command can be run in background using the option `-d` or interactively using the option `-i`.

Try the following command on your Ubuntu container:

```bash 
docker exec -it a5a2c01df566 bash
root@a5a2c01df566:/#
```
Let's look at the processes inside the container:
```bash
root@a5a2c01df566:/# ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0   4108  3420 pts/0    Ss+  14:05   0:00 bash
root           9  0.6  0.0   4108  3468 pts/1    Ss   14:06   0:00 bash
root          17  0.0  0.0   5896  2848 pts/1    R+   14:06   0:00 ps aux
```
We can see that the `exec` command started a new shell session. 








!!! tip
    Usually the `exec` command is used to launch `bash` within the container and work with that. 
    The `attach` command primarily is used if you quickly want to see the output of the main process (`PID 1`) directly and/or want to kill it.
    



