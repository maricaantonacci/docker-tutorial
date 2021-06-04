The storage location of Docker images and containers depends on the operating system. The command `docker info` provides information about your Docker configuration, including the `Storage Driver` and the `Docker Root Dir`.

On Ubuntu, Docker stores images and containers files under `/var/lib/docker`:

```bash
sudo ls /var/lib/docker/ -latr
total 60
drwxr-xr-x 40 root root  4096 May 27 13:27 ..
drwx------  3 root root  4096 May 27 13:27 image
drwx------  2 root root  4096 May 27 13:27 trust
drwxr-x---  3 root root  4096 May 27 13:27 network
drwx------  2 root root  4096 May 27 13:27 swarm
drwx--x--x  4 root root  4096 May 27 13:27 buildkit
drwx------  2 root root  4096 May 27 13:27 runtimes
drwx--x--x 13 root root  4096 May 27 13:27 .
drwx-----x  3 root root  4096 May 27 18:23 volumes
drwx------  4 root root  4096 Jun  2 15:50 plugins
drwx------  2 root root  4096 Jun  3 10:58 tmp
drwx-----x  6 root root 12288 Jun  3 10:58 overlay2
drwx-----x  3 root root  4096 Jun  3 10:58 containers
```

Docker images are stored in `/var/lib/docker/overlay2`.

Let's explore the content of our image `hello-world`:

???+ info inline end
     The command `docker inspect` returns low-level information on Docker objects (images, containers, networks, etc.).

     More info in the Docker official [doc](https://docs.docker.com/engine/reference/commandline/inspect/){:target=_blank}.

```bash
docker image inspect hello-world
```

```bash
[
    {
        "Id": "sha256:d1165f2212346b2bab48cb01c1e39ee8ad1be46b87873d9ca7a4e434980a7726",
        "RepoTags": [
            "hello-world:latest"
        ],
....
        "GraphDriver": {
            "Data": {
                "MergedDir": "/var/lib/docker/overlay2/f9457322a6bf658b38d3faf3bdd607f6873b89e3be2e9de222c9eb81a069f29d/merged",
                "UpperDir": "/var/lib/docker/overlay2/f9457322a6bf658b38d3faf3bdd607f6873b89e3be2e9de222c9eb81a069f29d/diff",
                "WorkDir": "/var/lib/docker/overlay2/f9457322a6bf658b38d3faf3bdd607f6873b89e3be2e9de222c9eb81a069f29d/work"
            },
            "Name": "overlay2"
        },
....
```

The `GraphDriver.Data` dictionary contains information about the layers of the image.

In general, the **LowerDir** contains the read-only layers of an image. The read-write layer that represents changes are part of the **UpperDir**.

The `hello-world` image is built starting from the base image [`scratch`](https://hub.docker.com/_/scratch){:target="_blank"} that is an explicitly empty image. 

The `hello-world` image therefore contains just one layer that adds the `hello` executable (statically linked) to the base empty image. 

Look at the content of the **UpperDir**:

```bash
sudo ls -latr /var/lib/docker/overlay2/f9457322a6bf658b38d3faf3bdd607f6873b89e3be2e9de222c9eb81a069f29d/diff
total 24
-rwxrwxr-x 1 root root 13336 Mar  5 23:25 hello
drwxr-xr-x 2 root root  4096 Jun  3 14:39 .
drwx-----x 3 root root  4096 Jun  3 14:39 ..
``` 

Another interesting command is `docker history` that shows the hystory of an image. Try it!

```bash
docker history hello-world
```

Output:
```bash
IMAGE          CREATED        CREATED BY                                      SIZE      COMMENT
d1165f221234   2 months ago   /bin/sh -c #(nop)  CMD ["/hello"]               0B
<missing>      2 months ago   /bin/sh -c #(nop) COPY file:7bf12aab75c3867a…   13.3kB
```

Again, you can see here that the image has only one layer (0B lines are neglected).
 
