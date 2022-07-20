`CMD` and `ENTRYPOINT` are the commands that allow us to set the default command to run in a container.

### Adding CMD to our Dockerfile
As example: we want to se a nice hello message, and using a custom font in our docker container, for that reason we will execute:

`figlet -f script hello`

!!! tip
    - `-f script` tells figlet to use a fancy font.
    - `hello` is the message that we want it to display.

Let's modify our Dockerfile to support this default: 

```Dockerfile
FROM ubuntu
RUN apt-get update
RUN apt-get install -y figlet
CMD figlet -f script hello
```

and let's build it again : 

```bash
docker build -t myfiglet .
```

This time you will see the effect of the cache and the output is the following: 

```
Sending build context to Docker daemon  2.048kB
Step 1/4 : FROM ubuntu:18.04
 ---> 81bcf752ac3d
Step 2/4 : RUN apt-get update
 ---> Using cache
 ---> e2ed94338e24
Step 3/4 : RUN apt-get install figlet
 ---> Using cache
 ---> d9c8c229f154
Step 4/4 : CMD figlet -f script hello
 ---> Running in a9f4bc819ea7
Removing intermediate container a9f4bc819ea7
 ---> e14a6ebfbd5e
Successfully built e14a6ebfbd5e
Successfully tagged myfiglet:latest
```

Run it:

```bash
docker run -ti myfiglet 
```

The output will looks like the following: 

```
 _          _   _       
| |        | | | |      
| |     _  | | | |  __  
|/ \   |/  |/  |/  /  \_
|   |_/|__/|__/|__/\__/ 
                     
```

### Overriding CMD

If we want to get a shell into our container (instead of running figlet), we just have to specify a different program to run. If we aspecify `bash`, it will replace the value of `CMD`.

Try it:

```bash
docker run -it myfiglet bash
```

### Using ENTRYPOINT

And what about if We want to be able to specify a different message on the command line, while retaining figlet and some default parameters?
Example: we  would like to be able to do this:

```
docker run myfiglet Good Morning
```

We will use the `ENTRYPOINT` verb in Dockerfile

```Dockerfile
FROM ubuntu
RUN apt-get update
RUN apt-get install -y figlet
ENTRYPOINT ["figlet", "-f", "script"]
```
!!! tip
    - ENTRYPOINT defines a base command (and its parameters) for the container.
    - The command line arguments are appended to those parameters.
    - Like CMD, ENTRYPOINT can appear anywhere, and replaces the previous value.


When CMD or ENTRYPOINT use string syntax, they get wrapped in `sh -c` and it would run the following command in the figlet image:
```
sh -c "figlet -f script" salut
```
To avoid this wrapping, we can use JSON syntax.

Let's build and test:

```bash
docker build -t myfiglet .
```
and run: 
```
docker run  myfiglet pippo
                        
      o                 
   _       _    _   __  
 |/ \_|  |/ \_|/ \_/  \_
 |__/ |_/|__/ |__/ \__/ 
/|      /|   /|         
\|      \|   \|         
```

If we want to run a shell in our container, We cannot just do `docker run myfiglet bash` because that would just tell figlet to display the word "bash."

We use the --entrypoint parameter:

```
$ docker run -it --entrypoint bash myfiglet
root@6027e44e2955:/#
```

### Combine CMD and ENTRYPOINT
What if we want to define a default message for our container?

Then we will use ENTRYPOINT and CMD together.

ENTRYPOINT will define the base command for our container.

CMD will define the default parameter(s) for this command.

They both have to use JSON syntax.

ENTRYPOINT defines a base command (and its parameters) for the container.

If we don't specify extra command-line arguments when starting the container, the value of CMD is appended.

Otherwise, our extra command-line arguments are used instead of CMD.

```Dockerfile
FROM ubuntu:18.04
RUN apt-get update
RUN apt-get install -y figlet
ENTRYPOINT ["figlet", "-f", "script"]
CMD ["ciccio"]
```

!!! tip 
    Finally CMD and ENTRYPOINT recap
    - `docker run myimage` executes ENTRYPOINT + CMD
    - `docker run myimage args` executes ENTRYPOINT + args (overriding CMD)
    - `docker run --entrypoint prog myimage` executes prog (overriding both)
