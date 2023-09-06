If you read the output from our `hello world`, they even recommend what to try next.

```bash
docker container run -it ubuntu bash
```
Let's see what happens:

```bash
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
445a6a12be2b: Pull complete
Digest: sha256:aabed3296a3d45cede1dc866a24476c4d7e093aa806263c27ddaadbdce3c1054
Status: Downloaded newer image for ubuntu:latest
root@2c8a2962ca5c:/#
```

We are inside the docker container!

!!! tip
    You need to use the `-it` option whenever you want to run a container in interactive mode.
    - The `-i` or `--interactive` option connects you to the input stream of the container, so that you can send inputs to bash;
    - The `-t` or `--tty` option makes sure that you get some good formatting and a native terminal-like experience by allocating a pseudo-tty. 

### Playing with a running container 

This is a fully fledged Ubuntu host, and we can do anything we like in it. Let's explore it a bit, starting with asking for its hostname:

```bash
root@2c8a2962ca5c:/# hostname
2c8a2962ca5c
```

!!! tip
    A container's hostname defaults to be the container's ID in Docker. You can override the hostname using `--hostname`.

Let's have a look at the `/etc/hosts` file too.
```bash
root@2c8a2962ca5c:/# cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.2	2c8a2962ca5c
```
Docker has also added a host entry for our container with its IP address. Let's also check out its networking configuration.

```bash
root@2c8a2962ca5c:/# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
6: eth0@if7: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```
!!! warning "ip: command not found"
    Install the package `iproute2` that provides a collection of utilities for networking and traffic control.
    ```bash
       apt update && apt install -y iproute2
    ```

As we can see, we have the `lo` loopback interface and the `eth0@if7` network interface with an IP address of 172.17.0.2, just like any other host. 

We can also check its running processes:

```bash
root@2c8a2962ca5c:/# ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0   4624  3780 pts/0    Ss   07:43   0:00 bash
root         352  0.0  0.0   7060  1568 pts/0    R+   07:53   0:00 ps aux
```

Note that the process `bash` has PID 1. 

Now type `exit` or the `CTRL-d` key sequence...you'll return to the command prompt of your Ubuntu host. So what's happened to our container? 
The container only runs for as long as the command we specified, `/bin/bash`, is running. Once we exited the container, that command ended, and the container was stopped.

So the container still exists but it's stopped:
```bash
docker container ps -a
CONTAINER ID   IMAGE         COMMAND    CREATED          STATUS                      PORTS     NAMES
2c8a2962ca5c   ubuntu        "bash"     11 minutes ago   Exited (0) 13 seconds ago             magical_sanderson
3a3eb75d3da3   hello-world   "/hello"   22 minutes ago   Exited (0) 22 minutes ago             sharp_kepler
```
### Starting a stopped container
We can start again our stopped container with `docker container start <container-id or container-name>`:

```bash
docker container start 2c8a2962ca5c
2c8a2962ca5c
```
Our container will restart with the same options we had specified when we launched it with the `docker run` command.

### Attaching to a container

The `docker container attach` command allows you to attach your terminal to the running container. 

!!! tip
    The command that is executed when starting a container is specified using the ENTRYPOINT and/or CMD instruction in the Dockerfile.
    The `attach` command allows you to connect and interact with the container’s main process which has `PID 1`.
    Remember that **if you kill the main process the container will terminate**.

This is useful when you want to see what is written in the standard output in real-time, or to control the process interactively.

So running the `attach` command on our Ubuntu container will bring us back to our bash prompt:
```bash
docker container attach 2c8a2962ca5c
root@2c8a2962ca5c:/#
```

You can detach from a container and leave it running using the `CTRL-p CTRL-q` key sequence.

What happens if you type `exit`?

### Getting a shell to a container

The `docker exec` command allows you to run commands inside a running container.
The command can be run in background using the option `-d` or interactively using the option `-i`.

Try the following command on your Ubuntu container:

```bash 
docker exec -it 2c8a2962ca5c bash
root@2c8a2962ca5c:/#
```
Let's look at the processes inside the container:
```bash
root@2c8a2962ca5c:/# ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0   4624  3844 pts/0    Ss+  07:55   0:00 bash
root           9  0.0  0.0   4624  3788 pts/1    Ss   07:59   0:00 bash
root          17  0.0  0.0   7060  1572 pts/1    R+   07:59   0:00 ps aux
```
We can see that the `exec` command started a new shell session. 


!!! tip
    Usually the `exec` command is used to launch `bash` within the container and work with that. 
    The `attach` command primarily is used if you quickly want to see the output of the main process (`PID 1`) directly and/or want to kill it.
    

