Whenever a running container wants to persist data, it actually put that data into the writable layer through storage driver.

Let's use our `nginx2` container created previously.

!!! tip
    If you don't have this container running you can recreate it with the following command:
    `docker container run -d -p 80:80 --name nginx2 nginx`

Let’s use the `docker exec` command to edit the welcome page and load it.

```bash
docker container exec -it nginx2 bash
```
You should now be inside your container. Run the following command to change the welcome page:

```bash
echo "I changed the content of this file inside the running container..." > /usr/share/nginx/html/index.html
```

You will be able to see these changes connecting to the port 80 of your host:

![](images/nginx_modified.png)

Let’s restart the container. What happens? We can still see the changes that we made. 

Now..what if we stop this container and start another one and load the page?

```bash
docker container run -d -p 8080:80 --name nginx3 nginx
```
!!! warning
    For this second container you need to specify a different host port, otherwise there will be a conflict and your container will not be started:
 
    `driver failed programming external connectivity on endpoint nginx3 (96fad8e096e1a124147049765f0d734e2a034712fb253711450c62a7158b1f21): Bind for 0.0.0.0:80 failed: port is already allocated.`


Connect to port 8080 of your host, you will see the default welcome page: there is no way that we could access the file that we have changed in another container.

![](images/nginx_default.png)

### Using docker volumes

Let's create our first docker volume:

```bash
docker volume create myvol
```

We can list the volumes with:

```bash
docker volume ls
```

```bash
DRIVER    VOLUME NAME
local     myvol
```

We can see the location of volumes in the docker area of the host file system with the inspect command:

```bash
docker volume inspect myvol
```

```bash
[
    {
        "CreatedAt": "2021-06-07T10:53:26Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/myvol/_data",
        "Name": "myvol",
        "Options": {},
        "Scope": "local"
    }
]
```

Let's re-create our `nginx` container with the following command that mounts the volume `myvol` in `/usr/share/nginx/html`:

Remove the old container
```bash
docker container rm -f nginx3
```

Recreate it with the volume mounted:
```bash
docker container run -d -p 8080:80 --name nginx3 --mount type=volume,source=myvol,destination=/usr/share/nginx/html nginx
```

!!! note
    - if we didn’t create the volume earlier docker will create it for us with the name given in source field of `--mount` parameter
    - volumes by default will not be deleted while we removing the container
    - if the container has got in `target` directory any files, this files will be copied into the volume

Enter the container and modify the welcome page:

```bash
docker exec -it nginx3 bash
```

Once inside the container, run the following command:

```bash
echo "I've changed the content of this file in the docker volume" > /usr/share/nginx/html/index.html
```

![](images/nginx_volume.png)

Let' stop and remove this container and create a new one with the same command:

```bash
docker rm -f nginx3
```
then
```bash
docker container run -d -p 8080:80 --name nginx3 --mount type=volume,source=myvol,destination=/usr/share/nginx/html nginx
```

If we load the page again we will still see the html file that we edited in the volume.

### Using bind mounts

!!! note
    - if we didn’t create a directory on docker host earlier docker will not create it for us with `--mount` parameter, auto-creating is available only in older `--volume`
    - bind mounts by default will not be deleted while removing the container
    - if the container has got in `target` directory any files, this files will **NOT** be copied into bind mount directory, bind directory will cover any files in a target container directory

Try the following command:

```bash
docker container run -d -p 8088:80 --name nginx4 --mount type=bind,source=/tmp/nginx,destination=/usr/share/nginx/html nginx
```

You will get an error since the path `/tmp/nginx` does not exist:
```
docker: Error response from daemon: invalid mount config for type "bind": bind source path does not exist: /tmp/nginx.
See 'docker run --help'.
```

Let's create the directory on the host:

```bash
mkdir /tmp/nginx
```

Now re-run the command for creating the container:

```bash
docker container run -d -p 8088:80 --name nginx4 --mount type=bind,source=/tmp/nginx,destination=/usr/share/nginx/html nginx
```

If you inspect the container you will see the bind-mount:

```bash
docker container inspect nginx4
```

```bash
...
            "Mounts": [
                {
                    "Type": "bind",
                    "Source": "/tmp/nginx",
                    "Target": "/usr/share/nginx/html"
                }
            ],
...
```

Connect to port `8088` on the host IP to see the result:

![](images/nginx_volume_2.png)

As you can see we get an error message from nginx as the bind-mount has overwritten the content of `/usr/share/nginx/html` (Remember that the behaviour with docker volumes is different, any file inside the container is copied in the volume)

Let's create the `index.html` file in the host path `/tmp/nginx`:

```bash
echo "I've changed the content of this file on the host" > /tmp/nginx/index.html
```

Then reload the page in the browser:

![](images/nginx_volume_3.png) 

!!! question
    As done before, try to remove the container and recreate it with the same bind-mount...what happens?

