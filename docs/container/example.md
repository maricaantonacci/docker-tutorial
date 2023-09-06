### Run a basic http server in a docker container

Let's deploy a simple web server using [`nginx`](https://www.nginx.com/){:target=_blank}.
First of all, let's search on Docker Hub for an already available image.

We can use the `search` command as follows:

```bash
docker search nginx
```

You will get something like the following output:

```bash
NAME                                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
nginx                                             Official build of Nginx.                        18966     [OK]
unit                                              Official build of NGINX Unit: Universal Web …   10        [OK]
nginxinc/nginx-unprivileged                       Unprivileged NGINX Dockerfiles                  118
nginx/nginx-ingress                               NGINX and  NGINX Plus Ingress Controllers fo…   76
nginx/nginx-prometheus-exporter                   NGINX Prometheus Exporter for NGINX and NGIN…   33
nginx/unit                                        NGINX Unit is a dynamic web and application …   64
nginxinc/nginx-s3-gateway                         Authenticating and caching gateway based on …   2
nginx/nginx-ingress-operator                      NGINX Ingress Operator for NGINX and NGINX P…   0
nginxinc/amplify-agent                            NGINX Amplify Agent docker repository           1
nginx/nginx-quic-qns                              NGINX QUIC interop                              1
nginxinc/ingress-demo                             Ingress Demo                                    4
nginxproxy/nginx-proxy                            Automated Nginx reverse proxy for docker con…   102
nginxproxy/acme-companion                         Automated ACME SSL certificate generation fo…   123
bitnami/nginx                                     Bitnami nginx Docker Image                      172                  [OK]
bitnami/nginx-ingress-controller                  Bitnami Docker Image for NGINX Ingress Contr…   29                   [OK]
ubuntu/nginx                                      Nginx, a high-performance reverse proxy & we…   98
nginxinc/nginmesh_proxy_debug                                                                     0
nginxproxy/docker-gen                             Generate files from docker container meta-da…   12
kasmweb/nginx                                     An Nginx image based off nginx:alpine and in…   6
nginxinc/mra-fakes3                                                                               0
rancher/nginx-ingress-controller                                                                  11
nginxinc/ngx-rust-tool                                                                            0
nginxinc/mra_python_base                                                                          0
nginxinc/nginmesh_proxy_init                                                                      0
rancher/nginx-ingress-controller-defaultbackend                                                   2
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
52d2b7f179e3: Pull complete
fd9f026c6310: Pull complete
055fa98b4363: Pull complete
96576293dd29: Pull complete
a7c4092be904: Pull complete
e3b6889c8954: Pull complete
da761d9a302b: Pull complete
Digest: sha256:104c7c5c54f2685f0f46f3be607ce60da7085da3eaa5ad22d3d9f01594295e9c
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest
```

In order to list the images downloaded on your host, you can use the command:

```bash
docker image ls 
```
The output of this command provides useful information, including the size of the image:

```bash
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
nginx         latest    eea7b3dcba7e   2 weeks ago    187MB
ubuntu        latest    c6b84b685f35   2 weeks ago    77.8MB
hello-world   latest    9c7a54a9a43c   4 months ago   13.3kB
```

Let's have a look at the image with the commands we have already seen in the previous section:

```bash
docker image inspect nginx
```

```bash linenums="1" hl_lines="29 37 61"
[
    {
        "Id": "sha256:eea7b3dcba7ee47c0d16a60cc85d2b977d166be3960541991f3e6294d795ed24",
        "RepoTags": [
            "nginx:latest"
        ],
        "RepoDigests": [
            "nginx@sha256:104c7c5c54f2685f0f46f3be607ce60da7085da3eaa5ad22d3d9f01594295e9c"
        ],
        "Parent": "",
        "Comment": "",
        "Created": "2023-08-16T09:50:55.765544033Z",
        "Container": "50b019921f82064e1d8af7e2723929d4c5fafcfd6d8b03595711bd1e455dd3c4",
        "ContainerConfig": {
            "Hostname": "50b019921f82",
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
                "NGINX_VERSION=1.25.2",
                "NJS_VERSION=0.8.0",
                "PKG_RELEASE=1~bookworm"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "#(nop) ",
                "CMD [\"nginx\" \"-g\" \"daemon off;\"]"
            ],
            "Image": "sha256:d59ed5fe14c2a306f94488f41ddc8fb060312ee31997f5e077a4c4b29b19114e",
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
        "DockerVersion": "20.10.23",
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
                "NGINX_VERSION=1.25.2",
                "NJS_VERSION=0.8.0",
                "PKG_RELEASE=1~bookworm"
            ],
            "Cmd": [
                "nginx",
                "-g",
                "daemon off;"
            ],
            "Image": "sha256:d59ed5fe14c2a306f94488f41ddc8fb060312ee31997f5e077a4c4b29b19114e",
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
        "Size": 186639842,
        "VirtualSize": 186639842,
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/8072cde78daa76af66dabe39fb661b0faf407786de21649c26fc18a6e1faf6a2/diff:/var/lib/docker/overlay2/4478bca3aabcb4ad7375573863ae4470ccc6aa5dfdbf055d94f881ea364f2da7/diff:/var/lib/docker/overlay2/f982333ae41a3a42a3eb7046e74def02cd1c2e6ddbc5d8bb2966720911f3ce5e/diff:/var/lib/docker/overlay2/e15b9368ef0bf3c0c048c6ca2ae517d3136663880fae208addb20574c9032e64/diff:/var/lib/docker/overlay2/927f8fc9244550a2d1da3449dd58bbdb98bf3d294f094436ac62761790dfaab8/diff:/var/lib/docker/overlay2/edb78366fb60e8316a834b56050484af04954ca588288d8cece3b38f6783fd72/diff",
                "MergedDir": "/var/lib/docker/overlay2/511ad049a7c912383723eb26a7112997bf95c2b0033e9effa50fbb21dae974ac/merged",
                "UpperDir": "/var/lib/docker/overlay2/511ad049a7c912383723eb26a7112997bf95c2b0033e9effa50fbb21dae974ac/diff",
                "WorkDir": "/var/lib/docker/overlay2/511ad049a7c912383723eb26a7112997bf95c2b0033e9effa50fbb21dae974ac/work"
            },
            "Name": "overlay2"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:511780f88f80081112aea1bfdca6c800e1983e401b338e20b2c6e97f384e4299",
                "sha256:4713cb24eeff341d0c36343149beba247572a5ff65c2be5b5d9baafb345c7393",
                "sha256:d0a62f56ef413f60049bc87e43e60032b2a2ab8d931e15b86ee0286c85ae91a2",
                "sha256:8a7e12012e6f60450e6d2d777b2a2c2256d34a0ccd84d605f72cc5329a87c8b8",
                "sha256:e161c3f476b5199ab13856c7e190ed12a6562b7be059c7026ae9f594e1abbcaf",
                "sha256:6fb960878295b567d25900b590157b976d080340caeaa8bf8c46d38c01b4537d",
                "sha256:563c64030925e9016a2329d3a2b7d47b0c90931baf5d2d0aa926c4c8d94ab894"
            ]
        },
        "Metadata": {
            "LastTagTime": "0001-01-01T00:00:00Z"
        }
    }
]
```

We can see that the nginx version in our container will be `1.25.2`, the service will be listening on port `80` and the command that will be executed at the container start is `nginx -g daemon off;`.

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
docker container ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS     NAMES
43d8e7c9a160   nginx     "/docker-entrypoint.…"   17 seconds ago   Up 9 seconds    80/tcp    nginx
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
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2023/09/04 08:08:28 [notice] 1#1: using the "epoll" event method
2023/09/04 08:08:28 [notice] 1#1: nginx/1.25.2
2023/09/04 08:08:28 [notice] 1#1: built by gcc 12.2.0 (Debian 12.2.0-14)
2023/09/04 08:08:28 [notice] 1#1: OS: Linux 5.15.0-82-generic
2023/09/04 08:08:28 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2023/09/04 08:08:28 [notice] 1#1: start worker processes
2023/09/04 08:08:28 [notice] 1#1: start worker process 29
2023/09/04 08:08:28 [notice] 1#1: start worker process 30
```

!!! tip
    We can also monitor the container's logs much like the `tail -f` binary operates using the `-f` flag.
    You can also tail a portion of the logs of a container by using the `--tail` option.
    Moreover you can also use the `-t` flag to prefix the log entries with timestamps.

### Inspecting the container processes

We can inspect the processes running inside our container using the `docker container top` command:

```bash
docker container top nginx
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                4989                4969                0                   08:08               ?                   00:00:00            nginx: master process nginx -g daemon off;
systemd+            5035                4989                0                   08:08               ?                   00:00:00            nginx: worker process
systemd+            5036                4989                0                   08:08               ?                   00:00:00            nginx: worker process
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
            "Id": "43d8e7c9a160a792bb7b9e85fd589c2986504edc0a8f7698473648dd71ee8810",
            "Created": "2023-09-04T08:08:19.846675755Z",
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
                "Pid": 4989,
                "ExitCode": 0,
                "Error": "",
                "StartedAt": "2023-09-04T08:08:27.030292632Z",
                "FinishedAt": "0001-01-01T00:00:00Z"
            },
            "Image": "sha256:eea7b3dcba7ee47c0d16a60cc85d2b977d166be3960541991f3e6294d795ed24",
            "ResolvConfPath": "/var/lib/docker/containers/43d8e7c9a160a792bb7b9e85fd589c2986504edc0a8f7698473648dd71ee8810/resolv.conf",
            "HostnamePath": "/var/lib/docker/containers/43d8e7c9a160a792bb7b9e85fd589c2986504edc0a8f7698473648dd71ee8810/hostname",
            "HostsPath": "/var/lib/docker/containers/43d8e7c9a160a792bb7b9e85fd589c2986504edc0a8f7698473648dd71ee8810/hosts",
            "LogPath": "/var/lib/docker/containers/43d8e7c9a160a792bb7b9e85fd589c2986504edc0a8f7698473648dd71ee8810/43d8e7c9a160a792bb7b9e85fd589c2986504edc0a8f7698473648dd71ee8810-json.log",
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
                    "Config": {}
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
                "ConsoleSize": [
                    60,
                    281
                ],
                "CapAdd": null,
                "CapDrop": null,
                "CgroupnsMode": "private",
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
                "Isolation": "",
                "CpuShares": 0,
                "Memory": 0,
                "NanoCpus": 0,
                "CgroupParent": "",
                "BlkioWeight": 0,
                "BlkioWeightDevice": [],
                "BlkioDeviceReadBps": [],
                "BlkioDeviceWriteBps": [],
                "BlkioDeviceReadIOps": [],
                "BlkioDeviceWriteIOps": [],
                "CpuPeriod": 0,
                "CpuQuota": 0,
                "CpuRealtimePeriod": 0,
                "CpuRealtimeRuntime": 0,
                "CpusetCpus": "",
                "CpusetMems": "",
                "Devices": [],
                "DeviceCgroupRules": null,
                "DeviceRequests": null,
                "MemoryReservation": 0,
                "MemorySwap": 0,
                "MemorySwappiness": null,
                "OomKillDisable": null,
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
                    "LowerDir": "/var/lib/docker/overlay2/1485063bd2a8e353b620248832fdae8836b6050e9499d293d1b65c5608a0433f-init/diff:/var/lib/docker/overlay2/511ad049a7c912383723eb26a7112997bf95c2b0033e9effa50fbb21dae974ac/diff:/var/lib/docker/overlay2/8072cde78daa76af66dabe39fb661b0faf407786de21649c26fc18a6e1faf6a2/diff:/var/lib/docker/overlay2/4478bca3aabcb4ad7375573863ae4470ccc6aa5dfdbf055d94f881ea364f2da7/diff:/var/lib/docker/overlay2/f982333ae41a3a42a3eb7046e74def02cd1c2e6ddbc5d8bb2966720911f3ce5e/diff:/var/lib/docker/overlay2/e15b9368ef0bf3c0c048c6ca2ae517d3136663880fae208addb20574c9032e64/diff:/var/lib/docker/overlay2/927f8fc9244550a2d1da3449dd58bbdb98bf3d294f094436ac62761790dfaab8/diff:/var/lib/docker/overlay2/edb78366fb60e8316a834b56050484af04954ca588288d8cece3b38f6783fd72/diff",
                    "MergedDir": "/var/lib/docker/overlay2/1485063bd2a8e353b620248832fdae8836b6050e9499d293d1b65c5608a0433f/merged",
                    "UpperDir": "/var/lib/docker/overlay2/1485063bd2a8e353b620248832fdae8836b6050e9499d293d1b65c5608a0433f/diff",
                    "WorkDir": "/var/lib/docker/overlay2/1485063bd2a8e353b620248832fdae8836b6050e9499d293d1b65c5608a0433f/work"
                },
                "Name": "overlay2"
            },
            "Mounts": [],
            "Config": {
                "Hostname": "43d8e7c9a160",
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
                    "NGINX_VERSION=1.25.2",
                    "NJS_VERSION=0.8.0",
                    "PKG_RELEASE=1~bookworm"
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
                "SandboxID": "0f0dc6953649e1d30c725332e4071257dd750addd46a6c0aa0c6e77a96e31364",
                "HairpinMode": false,
                "LinkLocalIPv6Address": "",
                "LinkLocalIPv6PrefixLen": 0,
                "Ports": {
                    "80/tcp": null
                },
                "SandboxKey": "/var/run/docker/netns/0f0dc6953649",
                "SecondaryIPAddresses": null,
                "SecondaryIPv6Addresses": null,
                "EndpointID": "25e03f10335ea9356d88ad67ed999291d6c6e26c3f91a1fd8853772a3988604f",
                "Gateway": "172.17.0.1",
                "GlobalIPv6Address": "",
                "GlobalIPv6PrefixLen": 0,
                "IPAddress": "172.17.0.3",
                "IPPrefixLen": 16,
                "IPv6Gateway": "",
                "MacAddress": "02:42:ac:11:00:03",
                "Networks": {
                    "bridge": {
                        "IPAMConfig": null,
                        "Links": null,
                        "Aliases": null,
                        "NetworkID": "6e452f5b4a2bc0f04db8a0393f0346a1a6852379c7443adb20cbb5a05b3c52bf",
                        "EndpointID": "25e03f10335ea9356d88ad67ed999291d6c6e26c3f91a1fd8853772a3988604f",
                        "Gateway": "172.17.0.1",
                        "IPAddress": "172.17.0.3",
                        "IPPrefixLen": 16,
                        "IPv6Gateway": "",
                        "GlobalIPv6Address": "",
                        "GlobalIPv6PrefixLen": 0,
                        "MacAddress": "02:42:ac:11:00:03",
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



