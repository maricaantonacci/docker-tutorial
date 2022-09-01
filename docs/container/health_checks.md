Health checks (available since Docker 1.12) allow a container to expose its workload’s availability. This stands apart from whether the container is running. If your database goes down, your API server won’t be able to handle requests, even though its Docker container is still running.

When a health check command is specified, it tells Docker how to test the container to see if it's working. With no health check specified, Docker has no way of knowing whether or not the services running within your container are actually up or not.

Health checks can be configured in different ways:

- directly in the docker image (Dockerfile)
- when you launch your standalone container with `docker run` (we will cover this case below) or in your docker compose file

In all cases, the health check is configured as a command that the docker daemon will execute every 30 seconds (default interval that can be overriden). Docker uses the command’s exit code to determine your container’s healthiness:

* 0 – The container is healthy and working normally.
* 1 – The container is unhealthy; the workload may not be functioning.
* 2 – This status code is reserved by Docker and should not be used. 

Without health checks, a simple `docker ps` would report the container as available. Adding a health check extends the `docker ps` output to include the container’s true state.

## Health Check Configuration

There are a few options that we can use to customize our health check instruction:

- interval - DURATION (default: 30s)
- timeout - DURATION (default: 30s)
- start-period - DURATION (default: 0s)
- retries - DURATION (default: 3)

- **interval** (option: `--health-interval`, default: 30s) - specifies the time between the health check for the application container. it waits for the specified time from one check to another.

- **timeout** (option: `--health-timeout`, default: 30s) - specifies the time that the health check waits for a response to consider the status of the container. For example, if we define 30 seconds and our server doesn’t respond within 30 seconds, then it’s considered as failed.

- **start-period** (option: `--health-start-period`, default: 0s) - specifies the number of seconds the container needs to start; health check will wait for that time to start.

- **retries** (option: `--health-retries`, default: 3) - specifies the number of consecutive health check failures required to declare the container status as unhealthy. Health check will only try up to the specified retry number. If the server fails consecutively up to the specified times, it is then considered unhealthy.

## Example

We will now add a health check to our `nginx` container: the check will be implemented using [`curl`](https://curl.se/docs/manpage.html) command `curl --fail http://localhost`:

```
docker run --name nginx -d --health-cmd='curl --fail http://localhost:80 || exit 1' nginx
``` 

The `curl` command makes a request to `localhost:80` and if the request returns the http code 200, it will return exit code 0; otherwise, it will return exit code 1. 

Look at the container status using `docker ps`. The container can have three states:

* starting – Initial status when the container is still starting
* healthy – If the command succeeds then the container is healthy
* unhealthy – If a single run of the  takes longer than the specified timeout then it is considered unhealthy. If a health check fails then the  will run retries number of times and will be declared unhealthy if the  still fails.

```
CONTAINER ID   IMAGE                    COMMAND                  CREATED         STATUS                            PORTS                                                           NAMES
ed1a8b6c451e   nginx                    "/docker-entrypoint.…"   9 seconds ago   Up 7 seconds (health: starting)   80/tcp                                                          nginx
```

Initially, it will take some time to start the health check and update, whether the application is healthy or not.
After the start period, if the application is healthy you will get:

```
CONTAINER ID   IMAGE                    COMMAND                  CREATED          STATUS                    PORTS                                                           NAMES
ed1a8b6c451e   nginx                    "/docker-entrypoint.…"   46 seconds ago   Up 44 seconds (healthy)   80/tcp                                                          nginx
```

You can have a look at the container log and see the requests made to check the health status of the application:

```
docker logs nginx
```
```
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2022/09/01 13:21:09 [notice] 1#1: using the "epoll" event method
2022/09/01 13:21:09 [notice] 1#1: nginx/1.23.1
2022/09/01 13:21:09 [notice] 1#1: built by gcc 10.2.1 20210110 (Debian 10.2.1-6)
2022/09/01 13:21:09 [notice] 1#1: OS: Linux 5.4.0-122-generic
2022/09/01 13:21:09 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2022/09/01 13:21:09 [notice] 1#1: start worker processes
2022/09/01 13:21:09 [notice] 1#1: start worker process 30
127.0.0.1 - - [01/Sep/2022:13:21:39 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.74.0" "-"
127.0.0.1 - - [01/Sep/2022:13:22:09 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.74.0" "-"
127.0.0.1 - - [01/Sep/2022:13:22:39 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.74.0" "-"
127.0.0.1 - - [01/Sep/2022:13:23:10 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.74.0" "-"
127.0.0.1 - - [01/Sep/2022:13:23:40 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.74.0" "-"
127.0.0.1 - - [01/Sep/2022:13:24:10 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.74.0" "-"
127.0.0.1 - - [01/Sep/2022:13:24:40 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.74.0" "-"
127.0.0.1 - - [01/Sep/2022:13:25:10 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.74.0" "-"
127.0.0.1 - - [01/Sep/2022:13:25:41 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.74.0" "-"
127.0.0.1 - - [01/Sep/2022:13:26:11 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.74.0" "-"
```

As you can see, the check runs every 30s (default) but you can change the interval with the option `--health-interval`.

Now let's remove the `index.html` file that nginx serves on http://localhost in order to simulate an application failure:

```
docker exec nginx sh -c 'mv /usr/share/nginx/html/index.html /usr/share/nginx/html/index.html.1'
```

Look again at the log:

```
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2022/09/01 13:21:09 [notice] 1#1: using the "epoll" event method
2022/09/01 13:21:09 [notice] 1#1: nginx/1.23.1
2022/09/01 13:21:09 [notice] 1#1: built by gcc 10.2.1 20210110 (Debian 10.2.1-6)
2022/09/01 13:21:09 [notice] 1#1: OS: Linux 5.4.0-122-generic
2022/09/01 13:21:09 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2022/09/01 13:21:09 [notice] 1#1: start worker processes
2022/09/01 13:21:09 [notice] 1#1: start worker process 30
127.0.0.1 - - [01/Sep/2022:13:21:39 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.74.0" "-"
127.0.0.1 - - [01/Sep/2022:13:22:09 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.74.0" "-"
127.0.0.1 - - [01/Sep/2022:13:22:39 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.74.0" "-"
127.0.0.1 - - [01/Sep/2022:13:23:10 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.74.0" "-"
127.0.0.1 - - [01/Sep/2022:13:23:40 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.74.0" "-"
127.0.0.1 - - [01/Sep/2022:13:24:10 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.74.0" "-"
127.0.0.1 - - [01/Sep/2022:13:24:40 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.74.0" "-"
127.0.0.1 - - [01/Sep/2022:13:25:10 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.74.0" "-"
127.0.0.1 - - [01/Sep/2022:13:25:41 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.74.0" "-"
127.0.0.1 - - [01/Sep/2022:13:26:11 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.74.0" "-"
127.0.0.1 - - [01/Sep/2022:13:26:41 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.74.0" "-"
127.0.0.1 - - [01/Sep/2022:13:27:11 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.74.0" "-"
127.0.0.1 - - [01/Sep/2022:13:27:41 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.74.0" "-"
127.0.0.1 - - [01/Sep/2022:13:28:11 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.74.0" "-"
127.0.0.1 - - [01/Sep/2022:13:28:42 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.74.0" "-"
127.0.0.1 - - [01/Sep/2022:13:29:12 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.74.0" "-"
127.0.0.1 - - [01/Sep/2022:13:29:42 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.74.0" "-"
127.0.0.1 - - [01/Sep/2022:13:30:12 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.74.0" "-"
127.0.0.1 - - [01/Sep/2022:13:30:42 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.74.0" "-"
2022/09/01 13:31:13 [error] 30#30: *20 directory index of "/usr/share/nginx/html/" is forbidden, client: 127.0.0.1, server: localhost, request: "GET / HTTP/1.1", host: "localhost"
127.0.0.1 - - [01/Sep/2022:13:31:13 +0000] "GET / HTTP/1.1" 403 153 "-" "curl/7.74.0" "-"
```

The `curl` command now returns the http error code 403 and therefore the check returns an exit code 1. Check the status of the container with `docker ps`:

```
CONTAINER ID   IMAGE                    COMMAND                  CREATED          STATUS                      PORTS                                                           NAMES
ed1a8b6c451e   nginx                    "/docker-entrypoint.…"   13 minutes ago   Up 13 minutes (unhealthy)   80/tcp                                                          nginx
```

The container is now flagged as unhealthy.

!!! question "Exercise"
    Restore the `index.html` file and verify the container status.

