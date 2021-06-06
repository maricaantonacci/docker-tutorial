### Run a basic http server in a docker container

Let's deploy a simple web server using [`nginx`](https://www.nginx.com/){:target=_blank}.
First of all, let's search on Docker Hub for an already available image.

We can use the `search` command as follows:

```bash
docker search nginx
```

You will get something like the following output:

```bash
NAME                               DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
nginx                              Official build of Nginx.                        14963     [OK]
jwilder/nginx-proxy                Automated Nginx reverse proxy for docker con…   2033                 [OK]
richarvey/nginx-php-fpm            Container running Nginx + PHP-FPM capable of…   813                  [OK]
jc21/nginx-proxy-manager           Docker container for managing Nginx proxy ho…   197
linuxserver/nginx                  An Nginx container, brought to you by LinuxS…   148
tiangolo/nginx-rtmp                Docker image with Nginx using the nginx-rtmp…   127                  [OK]
jlesage/nginx-proxy-manager        Docker container for Nginx Proxy Manager        115                  [OK]
alfg/nginx-rtmp                    NGINX, nginx-rtmp-module and FFmpeg from sou…   99                   [OK]
bitnami/nginx                      Bitnami nginx Docker Image                      96                   [OK]
nginxdemos/hello                   NGINX webserver that serves a simple page co…   70                   [OK]
privatebin/nginx-fpm-alpine        PrivateBin running on an Nginx, php-fpm & Al…   53                   [OK]
nginx/nginx-ingress                NGINX and  NGINX Plus Ingress Controllers fo…   52
nginxinc/nginx-unprivileged        Unprivileged NGINX Dockerfiles                  36
staticfloat/nginx-certbot          Opinionated setup for automatic TLS certs lo…   23                   [OK]
schmunk42/nginx-redirect           A very simple container to redirect HTTP tra…   19                   [OK]
nginx/nginx-prometheus-exporter    NGINX Prometheus Exporter for NGINX and NGIN…   18
centos/nginx-112-centos7           Platform for running nginx 1.12 or building …   15
centos/nginx-18-centos7            Platform for running nginx 1.8 or building n…   13
bitwarden/nginx                    The Bitwarden nginx web server acting as a r…   11
flashspys/nginx-static             Super Lightweight Nginx Image                   10                   [OK]
mailu/nginx                        Mailu nginx frontend                            8                    [OK]
bitnami/nginx-ingress-controller   Bitnami Docker Image for NGINX Ingress Contr…   8                    [OK]
navidonskis/nginx-php5.6           Docker nginx + php5.6 on Ubuntu                 7                    [OK]
ansibleplaybookbundle/nginx-apb    An APB to deploy NGINX                          2                    [OK]
wodby/nginx                        Generic nginx                                   1                    [OK]
```
!!! tip
    The `docker search` command returns the following image information:

    - Repository names
    - Image descriptions
    - Stars - these measure the popularity of an image
    - Official - an image managed by the upstream developer (e.g., the fedora image managed by the Fedora team) 
    - Automated - an image built by the Docker Hub's Automated Build process

In alternative, you can make a similar search on the Docker Hub Web [site](https://hub.docker.com/){:target=_blank}:

![Search nginx image](images/docker_hub_nginx.png)

Let's download the official image using the `docker image pull` command:

```bash
docker image pull nginx
```

```bash
Using default tag: latest
latest: Pulling from library/nginx
69692152171a: Pull complete
30afc0b18f67: Pull complete
596b1d696923: Pull complete
febe5bd23e98: Pull complete
8283eee92e2f: Pull complete
351ad75a6cfa: Pull complete
Digest: sha256:6d75c99af15565a301e48297fa2d121e15d80ad526f8369c526324f0f7ccb750
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest
```

In order to list the images downloaded on your host, you can use the command:

```bash
docker image ls 
```
The output of this command provides useful information, including the size of the image:

```bash
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
nginx        latest    d1a364dc548d   10 days ago   133MB
ubuntu       latest    7e0aa2d69a15   5 weeks ago   72.7MB
```

Let's have a look at the image with the commands we have already seen in the previous section:

```bash
docker image inspect nginx
```

```bash linenums="1" hl_lines="29 37 61"
[
    {
        "Id": "sha256:d1a364dc548d5357f0da3268c888e1971bbdb957ee3f028fe7194f1d61c6fdee",
        "RepoTags": [
            "nginx:latest"
        ],
        "RepoDigests": [
            "nginx@sha256:6d75c99af15565a301e48297fa2d121e15d80ad526f8369c526324f0f7ccb750"
        ],
        "Parent": "",
        "Comment": "",
        "Created": "2021-05-25T15:43:43.382480482Z",
        "Container": "7b06b818c018bb8563a3d786d6b16971c6f470c3d4c5288d908a3851b8261086",
        "ContainerConfig": {
            "Hostname": "7b06b818c018",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "80/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "NGINX_VERSION=1.21.0",
                "NJS_VERSION=0.5.3",
                "PKG_RELEASE=1~buster"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "#(nop) ",
                "CMD [\"nginx\" \"-g\" \"daemon off;\"]"
            ],
            "Image": "sha256:697718de459ceac2204a10028cb4008e64513e26697c154309ae93d2f64baa57",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": [
                "/docker-entrypoint.sh"
            ],
            "OnBuild": null,
            "Labels": {
                "maintainer": "NGINX Docker Maintainers <docker-maint@nginx.com>"
            },
            "StopSignal": "SIGQUIT"
        },
        "DockerVersion": "19.03.12",
        "Author": "",
        "Config": {
            "Hostname": "",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "80/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "NGINX_VERSION=1.21.0",
                "NJS_VERSION=0.5.3",
                "PKG_RELEASE=1~buster"
            ],
            "Cmd": [
                "nginx",
                "-g",
                "daemon off;"
            ],
            "Image": "sha256:697718de459ceac2204a10028cb4008e64513e26697c154309ae93d2f64baa57",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": [
                "/docker-entrypoint.sh"
            ],
            "OnBuild": null,
            "Labels": {
                "maintainer": "NGINX Docker Maintainers <docker-maint@nginx.com>"
            },
            "StopSignal": "SIGQUIT"
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 133117876,
        "VirtualSize": 133117876,
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/fa16037866db03e19d99c1d3695d6951569e751ba938036bcc1e82e31dad41cb/diff:/var/lib/docker/overlay2/adf9f3208ebac0a1566f031e9404f218349446153d3124c46c96d0db7d1b6097/diff:/var/lib/docker/overlay2/a5121e4e639c611e78ca69f1f5d90d032249852a6927b12968ac7709945f41e4/diff:/var/lib/docker/overlay2/8bf60f425fa90fe22dd757fc45d388c568254b79ec6f5bf724ac6a3a7972af84/diff:/var/lib/docker/overlay2/d75a9d8d1cabc06f29c2d1ea6854083e8de20bec99df58efed06bcbc01a9e0f4/diff",
                "MergedDir": "/var/lib/docker/overlay2/c1e6aceded10a0b1ebf0c89ed6b382faea13f6a2564b6be59641c91be4da68b3/merged",
                "UpperDir": "/var/lib/docker/overlay2/c1e6aceded10a0b1ebf0c89ed6b382faea13f6a2564b6be59641c91be4da68b3/diff",
                "WorkDir": "/var/lib/docker/overlay2/c1e6aceded10a0b1ebf0c89ed6b382faea13f6a2564b6be59641c91be4da68b3/work"
            },
            "Name": "overlay2"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:02c055ef67f5904019f43a41ea5f099996d8e7633749b6e606c400526b2c4b33",
                "sha256:766fe2c3fc083fdb0e132c138118bc931e3cd1bf4a8bdf0e049afbf64bae5ee6",
                "sha256:83634f76e73296b28a0e90c640494970bdfc437749598e0e91e77eea9bdb6a4e",
                "sha256:134e19b2fac580eff84faabfd5067977b79e36c5981d51fd63e8ac752dbdf9ec",
                "sha256:5c865c78bc96874203b5aa48f1a089d1eabcbe1607edaa16aaa6dee27c985395",
                "sha256:075508cf8f04705d8dc648cfb9f044f5dff57c31ccf34bde32cd2874f402dfad"
            ]
        },
        "Metadata": {
            "LastTagTime": "0001-01-01T00:00:00Z"
        }
    }
]
```

We can see that the nginx version in our container will be `1.21.0`, the service will be listening on port `80` and the command that will be executed at the container start is `nginx -g daemon off;`.

This is a useful exercise, but in general you will find these information in the [description](https://hub.docker.com/_/nginx?tab=description&page=1&ordering=last_updated){:target=_blank} of the image on Docker hub.

### Creating a daemonized container
In addition to the interactive containers, we can create *longer-running* containers. 

Daemonized containers don't have the interactive session we've used in our previous example and are ideal for running applications and services. 
Most of the containers you're likely to run will probably be daemonized. 

Let's start a daemonized container now.

```bash
docker container run -d --name nginx nginx
```

!!! note 
    The `-d` flag tells Docker to detach the container to the background.
    The `--name` option allows to set a name for your container

Instead of being attached to a shell, the `docker run` command has instead returned a container ID and returned us to our command prompt. 

We can see our container running with:

```bash
docker container ps
```

```bash
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS     NAMES
2f03f1f2cd10   nginx     "/docker-entrypoint.…"   13 seconds ago   Up 11 seconds   80/tcp    nginx
```

### Getting the container log

What's happening inside our container?
We can use the `docker container logs` command to fetch the log of a container:

```bash
docker container logs nginx
```

```bash
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2021/06/06 14:52:38 [notice] 1#1: using the "epoll" event method
2021/06/06 14:52:38 [notice] 1#1: nginx/1.21.0
2021/06/06 14:52:38 [notice] 1#1: built by gcc 8.3.0 (Debian 8.3.0-6)
2021/06/06 14:52:38 [notice] 1#1: OS: Linux 5.4.0-60-generic
2021/06/06 14:52:38 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2021/06/06 14:52:38 [notice] 1#1: start worker processes
2021/06/06 14:52:38 [notice] 1#1: start worker process 32
2021/06/06 14:52:38 [notice] 1#1: start worker process 33
```

!!! tip
    We can also monitor the container's logs much like the `tail -f` binary operates using the `-f` flag.
    You can also tail a portion of the logs of a container by using the `--tail` option.
    Moreover you can also use the `-t` flag to prefix the log entries with timestamps.

### Inspecting the container processes

We can inspect the processes running inside our container using the `docker container top` command:

```bash
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                138792              138771              0                   14:52               ?                   00:00:00            nginx: master process nginx -g daemon off;
systemd+            138861              138792              0                   14:52               ?                   00:00:00            nginx: worker process
systemd+            138862              138792              0                   14:52               ?                   00:00:00            nginx: worker process
```

### Finding out more about our container

Let's use again the command `docker container inspect` to get more information about our container:

=== "command"

    ```bash
    docker container inspect nginx
    ```

=== "Output"

    ```bash
    [
        {
            "Id": "2f03f1f2cd100ec584349124a93f180fa743d1c3ab7bcc31c9d8156225d6f06a",
            "Created": "2021-06-06T14:52:36.827435939Z",
            "Path": "/docker-entrypoint.sh",
            "Args": [
                "nginx",
                "-g",
                "daemon off;"
            ],
            "State": {
                "Status": "running",
                "Running": true,
                "Paused": false,
                "Restarting": false,
                "OOMKilled": false,
                "Dead": false,
                "Pid": 138792,
                "ExitCode": 0,
                "Error": "",
                "StartedAt": "2021-06-06T14:52:38.171320981Z",
                "FinishedAt": "0001-01-01T00:00:00Z"
            },
            "Image": "sha256:d1a364dc548d5357f0da3268c888e1971bbdb957ee3f028fe7194f1d61c6fdee",
            "ResolvConfPath": "/var/lib/docker/containers/2f03f1f2cd100ec584349124a93f180fa743d1c3ab7bcc31c9d8156225d6f06a/resolv.conf",
            "HostnamePath": "/var/lib/docker/containers/2f03f1f2cd100ec584349124a93f180fa743d1c3ab7bcc31c9d8156225d6f06a/hostname",
            "HostsPath": "/var/lib/docker/containers/2f03f1f2cd100ec584349124a93f180fa743d1c3ab7bcc31c9d8156225d6f06a/hosts",
            "LogPath": "/var/lib/docker/containers/2f03f1f2cd100ec584349124a93f180fa743d1c3ab7bcc31c9d8156225d6f06a/2f03f1f2cd100ec584349124a93f180fa743d1c3ab7bcc31c9d8156225d6f06a-json.log",
            "Name": "/nginx",
            "RestartCount": 0,
            "Driver": "overlay2",
            "Platform": "linux",
            "MountLabel": "",
            "ProcessLabel": "",
            "AppArmorProfile": "docker-default",
            "ExecIDs": null,
            "HostConfig": {
                "Binds": null,
                "ContainerIDFile": "",
                "LogConfig": {
                    "Type": "json-file",
                    "Config": {
                        "max-file": "3",
                        "max-size": "100m"
                    }
                },
                "NetworkMode": "default",
                "PortBindings": {},
                "RestartPolicy": {
                    "Name": "no",
                    "MaximumRetryCount": 0
                },
                "AutoRemove": false,
                "VolumeDriver": "",
                "VolumesFrom": null,
                "CapAdd": null,
                "CapDrop": null,
                "CgroupnsMode": "host",
                "Dns": [],
                "DnsOptions": [],
                "DnsSearch": [],
                "ExtraHosts": null,
                "GroupAdd": null,
                "IpcMode": "private",
                "Cgroup": "",
                "Links": null,
                "OomScoreAdj": 0,
                "PidMode": "",
                "Privileged": false,
                "PublishAllPorts": false,
                "ReadonlyRootfs": false,
                "SecurityOpt": null,
                "UTSMode": "",
                "UsernsMode": "",
                "ShmSize": 67108864,
                "Runtime": "runc",
                "ConsoleSize": [
                    0,
                    0
                ],
                "Isolation": "",
                "CpuShares": 0,
                "Memory": 0,
                "NanoCpus": 0,
                "CgroupParent": "",
                "BlkioWeight": 0,
                "BlkioWeightDevice": [],
                "BlkioDeviceReadBps": null,
                "BlkioDeviceWriteBps": null,
                "BlkioDeviceReadIOps": null,
                "BlkioDeviceWriteIOps": null,
                "CpuPeriod": 0,
                "CpuQuota": 0,
                "CpuRealtimePeriod": 0,
                "CpuRealtimeRuntime": 0,
                "CpusetCpus": "",
                "CpusetMems": "",
                "Devices": [],
                "DeviceCgroupRules": null,
                "DeviceRequests": null,
                "KernelMemory": 0,
                "KernelMemoryTCP": 0,
                "MemoryReservation": 0,
                "MemorySwap": 0,
                "MemorySwappiness": null,
                "OomKillDisable": false,
                "PidsLimit": null,
                "Ulimits": null,
                "CpuCount": 0,
                "CpuPercent": 0,
                "IOMaximumIOps": 0,
                "IOMaximumBandwidth": 0,
                "MaskedPaths": [
                    "/proc/asound",
                    "/proc/acpi",
                    "/proc/kcore",
                    "/proc/keys",
                    "/proc/latency_stats",
                    "/proc/timer_list",
                    "/proc/timer_stats",
                    "/proc/sched_debug",
                    "/proc/scsi",
                    "/sys/firmware"
                ],
                "ReadonlyPaths": [
                    "/proc/bus",
                    "/proc/fs",
                    "/proc/irq",
                    "/proc/sys",
                    "/proc/sysrq-trigger"
                ]
            },
            "GraphDriver": {
                "Data": {
                    "LowerDir": "/var/lib/docker/overlay2/8b4c0276493d49acf7c85ad47ca5192896e9ea1ea2d2aa3f02d4f2d8eb47f58d-init/diff:/var/lib/docker/overlay2/c1e6aceded10a0b1ebf0c89ed6b382faea13f6a2564b6be59641c91be4da68b3/diff:/var/lib/docker/overlay2/fa16037866db03e19d99c1d3695d6951569e751ba938036bcc1e82e31dad41cb/diff:/var/lib/docker/overlay2/adf9f3208ebac0a1566f031e9404f218349446153d3124c46c96d0db7d1b6097/diff:/var/lib/docker/overlay2/a5121e4e639c611e78ca69f1f5d90d032249852a6927b12968ac7709945f41e4/diff:/var/lib/docker/overlay2/8bf60f425fa90fe22dd757fc45d388c568254b79ec6f5bf724ac6a3a7972af84/diff:/var/lib/docker/overlay2/d75a9d8d1cabc06f29c2d1ea6854083e8de20bec99df58efed06bcbc01a9e0f4/diff",
                    "MergedDir": "/var/lib/docker/overlay2/8b4c0276493d49acf7c85ad47ca5192896e9ea1ea2d2aa3f02d4f2d8eb47f58d/merged",
                    "UpperDir": "/var/lib/docker/overlay2/8b4c0276493d49acf7c85ad47ca5192896e9ea1ea2d2aa3f02d4f2d8eb47f58d/diff",
                    "WorkDir": "/var/lib/docker/overlay2/8b4c0276493d49acf7c85ad47ca5192896e9ea1ea2d2aa3f02d4f2d8eb47f58d/work"
                },
                "Name": "overlay2"
            },
            "Mounts": [],
            "Config": {
                "Hostname": "2f03f1f2cd10",
                "Domainname": "",
                "User": "",
                "AttachStdin": false,
                "AttachStdout": false,
                "AttachStderr": false,
                "ExposedPorts": {
                    "80/tcp": {}
                },
                "Tty": false,
                "OpenStdin": false,
                "StdinOnce": false,
                "Env": [
                    "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                    "NGINX_VERSION=1.21.0",
                    "NJS_VERSION=0.5.3",
                    "PKG_RELEASE=1~buster"
                ],
                "Cmd": [
                    "nginx",
                    "-g",
                    "daemon off;"
                ],
                "Image": "nginx",
                "Volumes": null,
                "WorkingDir": "",
                "Entrypoint": [
                    "/docker-entrypoint.sh"
                ],
                "OnBuild": null,
                "Labels": {
                    "maintainer": "NGINX Docker Maintainers <docker-maint@nginx.com>"
                },
                "StopSignal": "SIGQUIT"
            },
            "NetworkSettings": {
                "Bridge": "",
                "SandboxID": "a74ea121ebf1163153ed7f24fb22b4cc11af77f336cfdbee9252d7c80ed9be0f",
                "HairpinMode": false,
                "LinkLocalIPv6Address": "",
                "LinkLocalIPv6PrefixLen": 0,
                "Ports": {
                    "80/tcp": null
                },
                "SandboxKey": "/var/run/docker/netns/a74ea121ebf1",
                "SecondaryIPAddresses": null,
                "SecondaryIPv6Addresses": null,
                "EndpointID": "c3b64e072f7851e3bf55ba3685fa1fbdec6b9c59d18ae27e5e90a1be5728f54c",
                "Gateway": "172.17.0.1",
                "GlobalIPv6Address": "",
                "GlobalIPv6PrefixLen": 0,
                "IPAddress": "172.17.0.2",
                "IPPrefixLen": 16,
                "IPv6Gateway": "",
                "MacAddress": "02:42:ac:11:00:02",
                "Networks": {
                    "bridge": {
                        "IPAMConfig": null,
                        "Links": null,
                        "Aliases": null,
                        "NetworkID": "0aa786b7de5fbfcd3e18db78511ed544d450bb0ad198cab0244bc8b83d30b514",
                        "EndpointID": "c3b64e072f7851e3bf55ba3685fa1fbdec6b9c59d18ae27e5e90a1be5728f54c",
                        "Gateway": "172.17.0.1",
                        "IPAddress": "172.17.0.2",
                        "IPPrefixLen": 16,
                        "IPv6Gateway": "",
                        "GlobalIPv6Address": "",
                        "GlobalIPv6PrefixLen": 0,
                        "MacAddress": "02:42:ac:11:00:02",
                        "DriverOpts": null
                    }
                }
            }
        }
    ] 
    ```

We can also selectively query the inspect results hash using the `-f` or `--format` flag.

For example, let's retrieve the container network address:

```bash
docker container inspect -f '{{.NetworkSettings.IPAddress}}' nginx
```

```
172.17.0.2
```



