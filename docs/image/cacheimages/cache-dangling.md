Let's do a quick check on our docker system: 

```
docker images

REPOSITORY                    TAG       IMAGE ID       CREATED         SIZE
test                          latest    259e2e6b1ac1   7 minutes ago   101MB
<none>                        <none>    589c6427b137   7 minutes ago   101MB
ubuntu                        18.04     81bcf752ac3d   3 weeks ago     63.1MB
gcr.io/k8s-minikube/kicbase   v0.0.22   bcd131522525   5 weeks ago     1.09GB
```

we should wonder what are the images with a `<none>` tag, you can see when you list all the images present on your system

!!! tip
    Spolinig: these are dagnling images

One thing more: 

```
docker images -a 

REPOSITORY                    TAG       IMAGE ID       CREATED         SIZE
test                          latest    259e2e6b1ac1   7 minutes ago   101MB
<none>                        <none>    589c6427b137   7 minutes ago   101MB
<none>                        <none>    e63fa5024b8d   7 minutes ago   101MB
<none>                        <none>    24a0ae7fb9f2   7 minutes ago   101MB
<none>                        <none>    333abd901bf3   7 minutes ago   99.7MB
ubuntu                        18.04     81bcf752ac3d   3 weeks ago     63.1MB
gcr.io/k8s-minikube/kicbase   v0.0.22   bcd131522525   5 weeks ago     1.09GB
```

again: why there are these images with a `<none>` tag, you can see when you list all the images present on your system

!!! tip
    Spolinig: these are intermediate cached images

Let's clean-up all our docker environment and use our previusly developed Dockerfile to package the application

```bash
docker system prune
```

* Be carefull! this wil clean-up a lot: 
    * all stopped containers
    * all networks not used by at least one container
    * all dangling images
    * all dangling build cache

there are other way to select what to remove see here [REF]

!!! tip 
    * `docker rmi $(docker images -a --filter=dangling=true -q)`
    * `docker rm $(docker ps --filter=status=exited --filter=status=created -q)`

once we have cleaned our system we will get something like the following: 

<pre><code> 
<b>$ docker images</b>
REPOSITORY                    TAG       IMAGE ID       CREATED       SIZE
gcr.io/k8s-minikube/kicbase   v0.0.22   bcd131522525   5 weeks ago   1.09GB
ubuntu                        latest    7e0aa2d69a15   7 weeks ago   72.7MB


<b>$ docker images  -a</b>
REPOSITORY                    TAG       IMAGE ID       CREATED       SIZE
gcr.io/k8s-minikube/kicbase   v0.0.22   bcd131522525   5 weeks ago   1.09GB
ubuntu                        latest    7e0aa2d69a15   7 weeks ago   72.7MB
tutor5@tutorvm-5:~/myimage$ 
</code></pre>

this is time now to build our application. So let's start again with our Dockerfile:

```
FROM ubuntu:18.04
RUN apt-get update
RUN apt-get install -y figlet
ENTRYPOINT ["figlet", "-f", "script"]
CMD ["pippo"]
```

Now we can build it and we will get something like this: 
<pre><code>
<b>docker build -t testnone .</b>

Sending build context to Docker daemon  2.048kB
Step 1/5 : FROM ubuntu:18.04
18.04: Pulling from library/ubuntu
4bbfd2c87b75: Pull complete 
d2e110be24e1: Pull complete 
889a7173dcfe: Pull complete 
Digest: sha256:67b730ece0d34429b455c08124ffd444f021b81e06fa2d9cd0adaf0d0b875182
Status: Downloaded newer image for ubuntu:18.04
 ---> <b>81bcf752ac3d</b>
Step 2/5 : RUN apt-get update
 ---> Running in 80bb3a4b5a8c
... ( remove output ) 
Reading package lists...
Removing intermediate container 80bb3a4b5a8c
 ---> e095319ffe18
Step 3/5 : RUN apt-get install figlet
 ---> Running in 30b20940a069
.... ( remove output ) 
Removing intermediate container 30b20940a069
 ---> 9ad1cce70073
Step 4/5 : ENTRYPOINT ["figlet", "-f", "script"]
 ---> Running in 911a64ae9765
Removing intermediate container 911a64ae9765
 ---> cca054892aec
Step 5/5 : CMD ["pippo"]
 ---> Running in 67d507ed70c0
Removing intermediate container 67d507ed70c0
 ---> f0684153d22f
Successfully built <b>f0684153d22f</b>
Successfully tagged testnone:latest
</code></pre>

and let's check again our system: 

<pre><code>
tutor5@tutorvm-5:~/myimage$ docker images
REPOSITORY                    TAG       IMAGE ID       CREATED              SIZE
testnone                      latest    <b>f0684153d22f</b>   About a minute ago   101MB
ubuntu                        18.04     <b>81bcf752ac3d</b>   3 weeks ago          63.1MB
gcr.io/k8s-minikube/kicbase   v0.0.22   bcd131522525   5 weeks ago          1.09GB
ubuntu                        latest    7e0aa2d69a15   7 weeks ago          72.7MB
</code></pre>

alright but then if I check further: 
 
<pre><code>
tutor5@tutorvm-5:~/myimage$ docker images -a 
REPOSITORY                    TAG       IMAGE ID       CREATED              SIZE
testnone                      latest    f0684153d22f   About a minute ago   101MB
&ltnone&gt                        &ltnone&gt    <b>cca054892aec</b>   About a minute ago   101MB
&ltnone&gt                        &ltnone&gt    <b>9ad1cce70073</b>   About a minute ago   101MB
&ltnone&gt                        &ltnone&gt    <b>e095319ffe18</b>   2 minutes ago        99.7MB
ubuntu                        18.04     81bcf752ac3d   3 weeks ago          63.1MB
gcr.io/k8s-minikube/kicbase   v0.0.22   bcd131522525   5 weeks ago          1.09GB
ubuntu                        latest    7e0aa2d69a15   7 weeks ago          72.7MB
</code></pre>

where those come from ? 
Those are the intermediate images genereated while building our image:

<pre><code>
Removing intermediate container 80bb3a4b5a8c
 ---> <b>e095319ffe18</b>
Step 3/5 : RUN apt-get install figlet
 ---> Running in 30b20940a069
.... ( remove output )
Removing intermediate container 30b20940a069
 ---> <b>9ad1cce70073</b>
Step 4/5 : ENTRYPOINT ["figlet", "-f", "script"]
 ---> Running in 911a64ae9765
Removing intermediate container 911a64ae9765
 ---> <b>cca054892aec</b>
</code></pre>

- a container is run
- changes corresponding to the instruction defined in the step are done inside of this container
- the container is committed into an image (that is an **intermediate** image) that will be used as the base image of the next step


Now the last step.. let's change our build starting from the previous Dockerfile and changing it: 

```Dockerfile
FROM ubuntu:18.04
RUN apt-get update
RUN apt-get install -y figlet
ENTRYPOINT ["figlet", "-f", "script"]
CMD ["ciccio"]  <----
```

build it as we did before: 
<pre><code>
<b>docker build -t testnone .</b>
Sending build context to Docker daemon  2.048kB
Step 1/5 : FROM ubuntu:18.04
 ---> 81bcf752ac3d
Step 2/5 : RUN apt-get update
 ---> Using cache
 ---> e095319ffe18
Step 3/5 : RUN apt-get install figlet
 ---> Using cache
 ---> 9ad1cce70073
Step 4/5 : ENTRYPOINT ["figlet", "-f", "script"]
 ---> Using cache
 ---> cca054892aec
Step 5/5 : CMD ["Ciccio"]
 ---> Running in 976759d6217c
Removing intermediate container 976759d6217c
 ---> 3624cff02928
Successfully built <b>3624cff02928</b>
Successfully tagged testnone:latest
</code></pre>

and now check again your images.. 

<pre><code>
<b>docker images</b>

REPOSITORY                    TAG       IMAGE ID       CREATED          SIZE
testnone                      latest    <b>3624cff02928</b>   4 seconds ago    101MB
&ltnone&gt                        &ltnone&gt    f0684153d22f   17 minutes ago   101MB
ubuntu                        18.04     81bcf752ac3d   3 weeks ago      63.1MB
gcr.io/k8s-minikube/kicbase   v0.0.22   bcd131522525   5 weeks ago      1.09GB
ubuntu                        latest    7e0aa2d69a15   7 weeks ago      72.7MB
</code></pre>

Now, this IMAGE ID **f0684153d22f** is not more linked to the image named `testnone`  cause the second build has set it on the newly created image (the one containing the changes we did in Dockerfile)

The previous image, now considered as dangling, is not referenced anymore. We can remove it, but we probably first need to make sure we have not used the same tag in the second build by mistake (that happens :) )

If needed, you can tag again the dandling image any other image ( we saw it previously ) 

Finally, in order to check ( and possibly remove ) the dangling images you can play with something like this: 

```bash
 docker images -a --filter=dangling=true 
```


