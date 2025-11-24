# Jellyfin Docker Compose Lab Documentation

## Basic Installation and Setup Steps (Ubuntu)

### Install Docker Engine
Remove all existing docker packages, add Docker's official GPG key, and add the Docker Repository using the following commands (From the [Docker Docs](https://docs.docker.com/engine/install/ubuntu/)). As Docker requires super user permissions, it is advisable to add `alias docker="sudo docker"` to your shell's rc file.
```
# Remove old packages:
$ sudo apt remove $(dpkg --get-selections docker.io docker-compose docker-compose-v2 docker-doc podman-docker containerd runc | cut -f1)

# Add Docker's official GPG key:
$ sudo apt update
$ sudo apt install ca-certificates curl
$ sudo install -m 0755 -d /etc/apt/keyrings
$ sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
$ sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
$ sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

# Update apt
$ sudo apt update

# Install new Docker packages
$ sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
### Installation Troubleshooting:
If this installation were to go wrong, these steps should be taken in order:
- Remove all potential outdated packages.
```
$ sudo apt remove $(dpkg --get-selections docker.io docker-compose docker-compose-v2 docker-doc podman-docker containerd runc | cut -f1)
```
- Ensure the Docker Repo is installed and the corresponding key is enrolled. If the output does not match, repeat from the GPG enrollment step.
```
$ cat /etc/apt/sources.list /etc/apt/sources.list.d/* | grep docker
URIs: https://download.docker.com/linux/ubuntu
Signed-By: /etc/apt/keyrings/docker.asc
```
- Update apt again
```
$ sudo apt update
```
- Install the correct packages again
```
$ sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Set up the Project Directory
Create and configure the directory where you wish to keep the files for this project. Jellyfin requires config, cache, and media volumes:
```
$ mkdir -p ~/code/jellyfin
$ cd ~/code/jellyfin
$ mkdir config cache media
$ sudo chown -R 1000:1000 config cache
```
### Create the Docker Compose File
Create the `docker-compose.yml` file in the project root directory. A pared down example is below (This example assumes your UID and GID are 1000):
```
services:
  jellyfin:
    image: jellyfin/jellyfin
    container_name: jellyfin
    user: "1000:1000"
    ports:
      - 127.0.0.1:8096:8096
    volumes:
      - ./config:/config
      - ./cache:/cache
      - ./media:/media
    restart: 'unless-stopped'
```

### Start the Jellyfin Server
Start the container in detached mode and check for its successful start with:
```
$ docker compose up -d
$ docker ps
```

#### Access Jellyfin
You can now access Jellyfin over `http://localhost:8096`, where you will be prompted to configure the server name as well as credentials for an account.

#### Next Steps
After the Jellyfin server has been properly installed and set up, further documentation can be found at the [Jellyfin Docs](https://jellyfin.org/docs/)
