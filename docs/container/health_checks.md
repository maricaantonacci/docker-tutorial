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

where

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
* unhealthy – If a single run of the command takes longer than the specified timeout then it is considered unhealthy. If a health check fails, retries will be run for the requested number of times and the container status will be declared unhealthy if the check still fails.

```
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS                            PORTS     NAMES
75beb1db3d27   nginx     "/docker-entrypoint.…"   9 seconds ago   Up 3 seconds (health: starting)   80/tcp    nginx
```

Initially, it will take some time to start the health check and update, whether the application is healthy or not.
After the start period, if the application is healthy you will get:

```
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS                    PORTS     NAMES
75beb1db3d27   nginx     "/docker-entrypoint.…"   46 seconds ago   Up 44 seconds (healthy)   80/tcp    nginx
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
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2023/09/06 06:44:20 [notice] 1#1: using the "epoll" event method
2023/09/06 06:44:20 [notice] 1#1: nginx/1.25.2
2023/09/06 06:44:20 [notice] 1#1: built by gcc 12.2.0 (Debian 12.2.0-14)
2023/09/06 06:44:20 [notice] 1#1: OS: Linux 5.15.0-82-generic
2023/09/06 06:44:20 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2023/09/06 06:44:20 [notice] 1#1: start worker processes
2023/09/06 06:44:20 [notice] 1#1: start worker process 28
2023/09/06 06:44:20 [notice] 1#1: start worker process 29
127.0.0.1 - - [06/Sep/2023:06:44:49 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.88.1" "-"
127.0.0.1 - - [06/Sep/2023:06:45:20 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.88.1" "-"
127.0.0.1 - - [06/Sep/2023:06:45:51 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.88.1" "-"
127.0.0.1 - - [06/Sep/2023:06:46:22 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.88.1" "-"
127.0.0.1 - - [06/Sep/2023:06:46:53 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.88.1" "-"
127.0.0.1 - - [06/Sep/2023:06:47:25 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.88.1" "-"
127.0.0.1 - - [06/Sep/2023:06:47:55 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.88.1" "-"
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
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2023/09/06 06:44:20 [notice] 1#1: using the "epoll" event method
2023/09/06 06:44:20 [notice] 1#1: nginx/1.25.2
2023/09/06 06:44:20 [notice] 1#1: built by gcc 12.2.0 (Debian 12.2.0-14)
2023/09/06 06:44:20 [notice] 1#1: OS: Linux 5.15.0-82-generic
2023/09/06 06:44:20 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2023/09/06 06:44:20 [notice] 1#1: start worker processes
2023/09/06 06:44:20 [notice] 1#1: start worker process 28
2023/09/06 06:44:20 [notice] 1#1: start worker process 29
127.0.0.1 - - [06/Sep/2023:06:44:49 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.88.1" "-"
127.0.0.1 - - [06/Sep/2023:06:45:20 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.88.1" "-"
127.0.0.1 - - [06/Sep/2023:06:45:51 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.88.1" "-"
127.0.0.1 - - [06/Sep/2023:06:46:22 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.88.1" "-"
127.0.0.1 - - [06/Sep/2023:06:46:53 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.88.1" "-"
127.0.0.1 - - [06/Sep/2023:06:47:25 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.88.1" "-"
127.0.0.1 - - [06/Sep/2023:06:47:55 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.88.1" "-"
127.0.0.1 - - [06/Sep/2023:06:48:26 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.88.1" "-"
127.0.0.1 - - [06/Sep/2023:06:48:57 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.88.1" "-"
2023/09/06 06:49:27 [error] 29#29: *10 directory index of "/usr/share/nginx/html/" is forbidden, client: 127.0.0.1, server: localhost, request: "GET / HTTP/1.1", host: "localhost"
127.0.0.1 - - [06/Sep/2023:06:49:27 +0000] "GET / HTTP/1.1" 403 153 "-" "curl/7.88.1" "-"
2023/09/06 06:49:58 [error] 29#29: *11 directory index of "/usr/share/nginx/html/" is forbidden, client: 127.0.0.1, server: localhost, request: "GET / HTTP/1.1", host: "localhost"
127.0.0.1 - - [06/Sep/2023:06:49:58 +0000] "GET / HTTP/1.1" 403 153 "-" "curl/7.88.1" "-"
2023/09/06 06:50:29 [error] 29#29: *12 directory index of "/usr/share/nginx/html/" is forbidden, client: 127.0.0.1, server: localhost, request: "GET / HTTP/1.1", host: "localhost"
127.0.0.1 - - [06/Sep/2023:06:50:29 +0000] "GET / HTTP/1.1" 403 153 "-" "curl/7.88.1" "-"
```

The `curl` command now returns the http error code 403 and therefore the check returns an exit code 1. Check the status of the container with `docker ps`:

```
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS                     PORTS     NAMES
75beb1db3d27   nginx     "/docker-entrypoint.…"   6 minutes ago   Up 6 minutes (unhealthy)   80/tcp    nginx
```

The container is flagged as unhealthy after three failures of the check (you can change the number of retries with the option `--health-retries`).

!!! question "Exercise"
    Restore the `index.html` file and verify the container status.

## Example: create a MariaDB container configuring a custom health check to check whether the server is available.

1. Set the following environment variables with appropriate values for your MySQL configuration:
   ```
   export MYSQL_USER=...                  # MySQL user (replace with your desired value)
   export MYSQL_PASSWORD=...              # MySQL user password (replace with your desired value)
   export MYSQL_ROOT_PASSWORD=...         # MySQL root user password (replace with your desired value)
   export MYSQL_DATABASE=...              # MySQL database name (replace with your desired value)
   ```

2. Run the container with the custom health check:
   ```
   docker run -d --name db \
     -e MYSQL_DATABASE=${MYSQL_DATABASE} \
     -e MYSQL_USER=${MYSQL_USER} \
     -e MYSQL_PASSWORD=${MYSQL_PASSWORD} \
     -e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD} \
     --health-cmd='mysqladmin -p${MYSQL_ROOT_PASSWORD} ping -h localhost' \
     --health-interval=20s \
     --health-retries=3 \
     mariadb:latest
   ```

In this command:

* `--name db`: Specifies the name of the container as "db."
* `-e MYSQL_DATABASE=${MYSQL_DATABASE}`: Sets the MySQL database name as an environment variable.
* `-e MYSQL_USER=${MYSQL_USER}`: Sets the MySQL user as an environment variable.
* `-e MYSQL_PASSWORD=${MYSQL_PASSWORD}`: Sets the MySQL user's password as an environment variable.
* `-e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}`: Sets the MySQL root user's password as an environment variable.
* `--health-cmd`: Specifies the custom health check command, which uses the provided MySQL root password to check if the MariaDB server is responsive.
* `--health-interval=20s`: Sets the health check interval to 20 seconds.
* `--health-retries=3`: Specifies the number of retries before marking the container as "unhealthy."


## Lab challenge

**Goal**: create a PostgreSQL container and setup a health check to monitor its status

!!! info "Hints"
    * Search for PostgreSQL official docker image on [docker hub](https://hub.docker.com/)
    * Read the documentation and understand how to start your server
    * Specify a simple health check using the tool "[pg_isready](https://www.postgresql.org/docs/current/app-pg-isready.html)" to check if the server is alive 
