The storage location of Docker images and containers depends on the operating system. The command `docker info` provides information about your Docker configuration, including the `Storage Driver` and the `Docker Root Dir`.

On Ubuntu, Docker stores images and containers files under `/var/lib/docker`:

```bash
total 52
drwxr-xr-x 39 root root 4096 Sep  4 07:24 ..
drwx------  2 root root 4096 Sep  4 07:24 runtimes
drwx-----x  2 root root 4096 Sep  4 07:24 volumes
drwx------  4 root root 4096 Sep  4 07:24 plugins
-rw-------  1 root root   36 Sep  4 07:24 engine-id
drwx------  3 root root 4096 Sep  4 07:24 image
drwxr-x---  3 root root 4096 Sep  4 07:24 network
drwx------  2 root root 4096 Sep  4 07:24 swarm
drwx--x--- 12 root root 4096 Sep  4 07:24 .
drwx--x--x  4 root root 4096 Sep  4 07:24 buildkit
drwx------  2 root root 4096 Sep  4 07:31 tmp
drwx--x---  6 root root 4096 Sep  4 07:31 overlay2
drwx--x---  3 root root 4096 Sep  4 07:31 containers
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
        "Id": "sha256:9c7a54a9a43cca047013b82af109fe963fde787f63f9e016fdc3384500c2823d",
        "RepoTags": [
            "hello-world:latest"
        ],
....
        "GraphDriver": {
            "Data": {
                "MergedDir": "/var/lib/docker/overlay2/04635df94f3219c431d9dfba92f4456f5e0247b3dcc0b94f55d331d4c3ced400/merged",
                "UpperDir": "/var/lib/docker/overlay2/04635df94f3219c431d9dfba92f4456f5e0247b3dcc0b94f55d331d4c3ced400/diff",
                "WorkDir": "/var/lib/docker/overlay2/04635df94f3219c431d9dfba92f4456f5e0247b3dcc0b94f55d331d4c3ced400/work"
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
sudo ls -latr /var/lib/docker/overlay2/04635df94f3219c431d9dfba92f4456f5e0247b3dcc0b94f55d331d4c3ced400/diff 
total 24
-rwxrwxr-x 1 root root 13256 May  4 17:36 hello
drwxr-xr-x 2 root root  4096 Sep  4 07:31 .
drwx--x--- 3 root root  4096 Sep  4 07:31 ..
``` 

Another interesting command is `docker history` that shows the hystory of an image. Try it!

```bash
docker history hello-world
```

Output:
```bash
IMAGE          CREATED        CREATED BY                                      SIZE      COMMENT
9c7a54a9a43c   4 months ago   /bin/sh -c #(nop)  CMD ["/hello"]               0B
<missing>      4 months ago   /bin/sh -c #(nop) COPY file:201f8f1849e89d53…   13.3kB
```

Again, you can see here that the image has only one layer (0B lines are neglected).
 
