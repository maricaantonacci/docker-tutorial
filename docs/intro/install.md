We will install docker engine on our Ubuntu Jammy 22.04 (LTS) Virtual Machine using the repository.

!!! info
    Full instructions are available at [https://docs.docker.com/engine/install/](https://docs.docker.com/engine/install/)

## Setup the repository

1. Update the apt package index and install packages to allow apt to use a repository over HTTPS:
   ``` bash
   sudo apt-get update
   
   sudo apt-get install \
       ca-certificates \
       curl \
       gnupg \
   ```

2. Add Docker’s official GPG key:
``` bash
 sudo mkdir -p /etc/apt/keyrings
 curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

3. Use the following command to set up the repository:
``` bash
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

## Install Docker Engine

Update the apt package index, and install the latest version of Docker Engine, containerd, and Docker Compose, or go to the next step to install a specific version:

```` bash
 sudo apt-get update
 sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
````


## Post-installation steps

The Docker daemon binds to a Unix socket instead of a TCP port. By default that Unix socket is owned by the user `root` and other users can only access it using `sudo`. The Docker daemon always runs as the `root` user.

If you don’t want to preface the `docker` command with `sudo`, add users to the Unix group `docker`. When the Docker daemon starts, it creates a Unix socket accessible by members of the `docker` group.

- Add your user to the docker group.

```
sudo usermod -aG docker $USER
```

- Log out and log back in so that your group membership is re-evaluated.

## Verify

=== "Command"
    ```
    docker --version
    ```
=== "Example output"
    ```
    Docker version 24.0.5, build ced0996
    ```




