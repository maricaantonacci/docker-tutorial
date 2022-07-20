Now that we have built our first images, we can publish them to the Docker Hub!

###Logging into our Docker Hub account

```bash
docker login
```

This requires an account on the Docker Hub ( which is free as well as storing the images it is free ). So if you have one you can try a thus you'll see a output  like this: 
```
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: spiga
Password: 
WARNING! Your password will be stored unencrypted in /home/tutor5/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

### Tagging an image to push it on the Hub

Docker images tags are like Git tags and branches, like bookmarks pointing at a specific image ID.

Tagging an image doesn't rename an image: it adds another tag. When pushing an image to a registry, the registry address is in the tag.

- `Example: registry.example.net:5000/image`

    - spiga/test is  index.docker.io/spiga/test
    - ubuntu is index.docker.io/library/ubuntu

### Let's tag our myfiglet image

Or please do it with any other image you like. Let's look for the image to tag: 

<pre><code>
$docker images 

REPOSITORY                    TAG               IMAGE ID       CREATED          SIZE
<b>myfiglet                      latest            cabcb70593be   15 seconds ago   103MB</b>
mycontainerizedapp            latest            b64faab26e8f   30 hours ago     147MB
python                        3.7-slim-buster   867339bd5033   2 weeks ago      113MB
ubuntu                        18.04             81bcf752ac3d   3 weeks ago      63.1MB
gcr.io/k8s-minikube/kicbase   v0.0.22           bcd131522525   5 weeks ago      1.09GB
ubuntu                        latest            7e0aa2d69a15   7 weeks ago      72.7MB
</code></pre>

Ok let's tag the `myfiglet`

```bash
docker tag myfiglet spiga/myfiglet
```

and check what's happening now: 
<pre><code>
docker images 
REPOSITORY                    TAG               IMAGE ID       CREATED              SIZE
myfiglet                      latest            cabcb70593be   About a minute ago   103MB
<b>spiga/myfiglet                latest            cabcb70593be   About a minute ago   103MB</b>
mycontainerizedapp            latest            b64faab26e8f   30 hours ago         147MB
python                        3.7-slim-buster   867339bd5033   2 weeks ago          113MB
ubuntu                        18.04             81bcf752ac3d   3 weeks ago          63.1MB
gcr.io/k8s-minikube/kicbase   v0.0.22           bcd131522525   5 weeks ago          1.09GB
ubuntu                        latest            7e0aa2d69a15   7 weeks ago          72.7MB
</code></pre>

Ok so we are ready to push it on Dockerhub 
```bash
docker push spiga/myfiglet 
```
and the output will look like something this one 

```
Using default tag: latest
The push refers to repository [docker.io/spiga/myfiglet]
f0f9028d0afb: Pushed 
3ee587de5e03: Pushed 
2f140462f3bc: Mounted from library/ubuntu 
63c99163f472: Mounted from library/ubuntu 
ccdbb80308cc: Mounted from library/ubuntu 
latest: digest: sha256:ec988085cd4d5efa6bdb5b26a3694eff2c6e26032521c4b6cb248b4efb0f5f51 size: 1365
```

Ok, very good! That's it!

Anybody can now docker run spiga/myfiglet **anywhere**.

### Quick check

Now if we remove the image on local filesystem and the we get it back from the registry ( DockerHub ) we can see how it works and we close the loop. 

!!! tip
    To remove the image : `docker rmi IMAGE ID`. To get the `IMAGE ID` just use `docker images`

<pre><code>
<b>docker pull spiga/myfiglet<b>

Using default tag: latest
latest: Pulling from spiga/myfiglet
345e3491a907: Already exists 
57671312ef6f: Already exists 
5e9250ddb7d0: Already exists 
d9dac9d5417f: Pull complete 
bdf8f857644c: Pull complete 
Digest: sha256:ec988085cd4d5efa6bdb5b26a3694eff2c6e26032521c4b6cb248b4efb0f5f51
Status: Downloaded newer image for spiga/myfiglet:latest
<b>docker.io/spiga/myfiglet:latest</b>
</code></pre>

