This is a quick recap of things that you already saw in the Part-1. The following steps guide you toward the interactive image building process. 

###Getting an image and apply some changes

=== "Command"
```bash
docker run -it ubuntu:18.04
```

You will get something like the following output:

```
Unable to find image 'ubuntu:18.04' locally
18.04: Pulling from library/ubuntu
4bbfd2c87b75: Pull complete 
d2e110be24e1: Pull complete 
889a7173dcfe: Pull complete 
Digest: sha256:67b730ece0d34429b455c08124ffd444f021b81e06fa2d9cd0adaf0d0b875182
Status: Downloaded newer image for ubuntu:18.04
root@844ec9e540a5:/#
```

Run the command apt-get update to refresh the list of packages available to install.
For this simple exercise we will use [Figlet](https://howtoinstall.co/en/figlet){:target="_blank"}

Then run the command apt-get install figlet to install the program we are interested in.


=== "Command"
```bash
apt-get update && apt-get install figlet
```

You will get something like the following output:

```
root@844ec9e540a5:/# apt-get update && apt-get install figlet
Get:1 http://archive.ubuntu.com/ubuntu bionic InRelease [242 kB]
Get:2 http://archive.ubuntu.com/ubuntu bionic-updates InRelease [88.7 kB]
Get:3 http://security.ubuntu.com/ubuntu bionic-security InRelease [88.7 kB]         
Get:4 http://archive.ubuntu.com/ubuntu bionic-backports InRelease [74.6 kB]                   
Get:5 http://archive.ubuntu.com/ubuntu bionic/main amd64 Packages [1344 kB]                    
Get:6 http://archive.ubuntu.com/ubuntu bionic/restricted amd64 Packages [13.5 kB]                         
Get:7 http://archive.ubuntu.com/ubuntu bionic/multiverse amd64 Packages [186 kB]                          
Get:8 http://archive.ubuntu.com/ubuntu bionic/universe amd64 Packages [11.3 MB]                           
Get:9 http://archive.ubuntu.com/ubuntu bionic-updates/multiverse amd64 Packages [33.5 kB]                 
Get:10 http://archive.ubuntu.com/ubuntu bionic-updates/restricted amd64 Packages [481 kB]
Get:11 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 Packages [2619 kB]
Get:12 http://archive.ubuntu.com/ubuntu bionic-updates/universe amd64 Packages [2185 kB]
Get:13 http://archive.ubuntu.com/ubuntu bionic-backports/universe amd64 Packages [11.4 kB]
Get:14 http://archive.ubuntu.com/ubuntu bionic-backports/main amd64 Packages [11.3 kB]
Get:15 http://security.ubuntu.com/ubuntu bionic-security/universe amd64 Packages [1415 kB]
...
...

```

###Check the differences with respect to the original image

Once done type exit at the container prompt to leave the interactive session.
Now let's run docker diff to see the difference between the base image and our container.

=== "Command"
```bash
docker diff 844ec9e540a5
```

!!! tip
    Remember: you need to use your container ID. In order to get it you can always use `docker ps -a`

and the output you will get is something like

```
C /root
A /root/.bash_history
C /etc
A /etc/emacs
A /etc/emacs/site-start.d
A /etc/emacs/site-start.d/50figlet.el
C /etc/alternatives
A /etc/alternatives/figlet
C /usr
C /usr/share
A /usr/share/figlet
A /usr/share/figlet/646-ca.flc
A /usr/share/figlet/646-ca2.flc
A /usr/share/figlet/646-hu.flc
```

!!! tip
    Three different types of change are tracked by `docker diff `:
        -`A` A file or directory was added
        -`D` A file or directory was deleted
        -`C` A file or directory was changed

###Commit the changes and use your image

The last step is now to commit the changes, that way we will create a new layer with the changes we made before, and a new image using this new layer.

=== "Command"
```bash
docker commit 844ec9e540a5 interactivefiglet
```

Here we go!  we have build our first image interactively! Let's run it and test it

=== "Command"
```bash
docker run -ti interactivefiglet 
```

once you entered the container you can type  something like the following:

=== "Command"
```bash
figlet ciao ciao 
```
you will get your expected output: 

```
      _                    _             
  ___(_) __ _  ___     ___(_) __ _  ___  
 / __| |/ _` |/ _ \   / __| |/ _` |/ _ \ 
| (__| | (_| | (_) | | (__| | (_| | (_) |
 \___|_|\__,_|\___/   \___|_|\__,_|\___/ 
                                         
root@be9e5e08db21:/#
```

!!! tip
    in one line command you could have typed the following with the same result: `docker run interactivefiglet figlet ciao ciao`

This is ok for quick & dirty test and playground .. but what about if we need to be reproducible, automated ? 
To this end we need to learn the build process by writing a Dockerfile.

