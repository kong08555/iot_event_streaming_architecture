# Setting Docker
https://docs.docker.com/engine/install/ubuntu/

Run the following command to uninstall all conflicting packages:

```
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

1. Set up Docker's apt repository.
```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

2. Install the Docker packages.
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

3. Verify that the Docker Engine installation is successful by running the hello-world image.

```
sudo docker run hello-world
```

This command downloads a test image and runs it in a container. When the container runs, it prints a confirmation message and exits.



# Post installation

https://docs.docker.com/engine/install/linux-postinstall/


Manage Docker as a non-root user

To create the docker group and add your user:

Create the docker group.

```
sudo groupadd docker
```

Add your user to the docker group.

```
sudo usermod -aG docker $USER
```

Log out and log back in so that your group membership is re-evaluated.

You can also run the following command to activate the changes to groups:

```
newgrp docker
```

# Configure Docker to start on boot with systemd

Many modern Linux distributions use systemd to manage which services start when the system boots. On Debian and Ubuntu, the Docker service starts on boot by default. To automatically start Docker and containerd on boot for other Linux distributions using systemd, run the following commands:


```
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

To stop this behavior, use disable instead.

```
sudo systemctl disable docker.service
sudo systemctl disable containerd.service
```


# Docker, Starting from the ground

Stop all container
```
docker stop $(docker ps -aq)
```

Delete all containers
```
docker container rm -f $(docker container ls -aq)
```

Delete all volumes
```
docker volume rm -f $(docker volume ls -q)
```

Delete network
```
docker network rm -f $(docker network ls -q)
```

Delete images
```
docker image rm -f $(docker image ls -q)
```


# Control startup and shutdown order in Compose

https://docs.docker.com/compose/startup-order/