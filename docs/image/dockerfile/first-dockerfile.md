A Dockerfile is a build recipe for a Docker image. It contains a series of instructions telling Docker how an image is constructed.


### Our first Dockerfile
Our Dockerfile must be in a new, empty directory so first step is to create a directory to hold our Dockerfile.

=== "Command"
```bash
mkdir myimage
```
and now create a Dockerfile inside this directory.

=== "Command"
```bash
cd myimage
```
```bash
vim Dockerfile
```
(feel fre to choose any editor you like) and add the follwing content: 

```Dockerfile
FROM ubuntu:18.04
RUN apt-get update
RUN apt-get install -y figlet
```

!!! tip
    * `FROM` indicates the base image for our build.
    * Each RUN line will be executed by Docker during the build.
    * RUN commands must be non-interactive. (No input can be provided to Docker during the build.) this is why we add the -y flag to apt-get.

Every Dockerfile must start with the FROM instruction. The idea behind is that you need a starting point to build your image. You can start `FROM scratch`, scratch is an explicitly empty image on the Docker store that is used to build base images such as [Alpine](https://hub.docker.com/_/alpine) a lightweight linux distro that allows you to reduce the overall size of Docker images.

Save our file, then execute:
=== "Command"
```bash
docker build -t myfiglet .
```

!!! tip
    - `-t` indicates the tag to apply to the image.
    - `.` indicates the location of the build context.

The output you will get is something like the follwing 

```
Sending build context to Docker daemon  2.048kB
Step 1/3 : FROM ubuntu:18.04
 ---> 81bcf752ac3d
Step 2/3 : RUN apt-get update
 ---> Running in 9f07f31f5608
...(..cut RUN output..)...
Reading package lists...
Removing intermediate container 9f07f31f5608
 ---> e2ed94338e24
Step 3/3 : RUN apt-get install figlet
 ---> Running in 9548f425acac
Reading package lists...
...(..cut RUN output..)...
Removing intermediate container 9548f425acac
 ---> d9c8c229f154
Successfully built d9c8c229f154
Successfully tagged myfiglet:latest

```

Let's analyze the oputput: 

`Sending build context to Docker daemon 2.048 kB`

- The build context is the . directory given to docker build.
    - It is sent (as an archive) by the Docker client to the Docker daemon.
        - This allows to use a remote machine to build using local files.
    - Be careful (or patient) if that directory is big and your link is slow.
        - You can speed up the process with a .dockerignore file which tells docker to ignore specific files in the directory, ignore files that you won't need in the build context!


```
Step 2/3 : RUN apt-get update
 ---> Running in 9f07f31f5608
...(..cut RUN output..)...
Reading package lists...
Removing intermediate container 9f07f31f5608
 ---> e2ed94338e24
```

A container (9f07f31f5608) is created from the base image. 

- The RUN command is executed in this container.
- The container is committed into an image (e2ed94338e24).
- The build container (9f07f31f5608) is removed.
- The output of this step will be the base image for the next one.


!!! tip 
    - After each build step, Docker takes a snapshot of the resulting image and before executing a step, Docker checks if it has already built the same sequence.
    Docker uses the exact strings defined in your Dockerfile, so the following two are not the same!
    - RUN apt-get install figlet [cowsay](https://cran.r-project.org/web/packages/cowsay/vignettes/cowsay_tutorial.html)
    - RUN apt-get install cowsay figlet   

You can force a rebuild with docker build --no-cache ....

And to close the loop: (only) `RUN`, `COPY` and `ADD` instructions create layers to improve build performance. The main advantage of image layering lies in image caching.

### Running the image
The resulting image is not different from the one produced manually. :)


```bash
docker run -ti myfiglet  bash
```
and issue something like: 
```
root@4d7d8ec44135:/# figlet ciao ciao 
      _                    _             
  ___(_) __ _  ___     ___(_) __ _  ___  
 / __| |/ _` |/ _ \   / __| |/ _` |/ _ \ 
| (__| | (_| | (_) | | (__| | (_| | (_) |
 \___|_|\__,_|\___/   \___|_|\__,_|\___/ 
                                         
root@4d7d8ec44135:/# 
```
