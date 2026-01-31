# Dockerfiles

For Dockerfiles saving

## Install Docker (Ubuntu)

### 1. Set up Docker's APT repository

```bash
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
```

### 2. Install Docker packages

```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### 3. Verify installation

```bash
sudo docker run hello-world
```

For more details, see the [official Docker documentation](https://docs.docker.com/engine/install/ubuntu).

## Build Images

### pytorch-base

#### 1. Build the image

```bash
cd pytorch-base
sudo docker build -t tyttyttyt/pytorch-base:latest .
```

#### 2. Push to Docker Hub

```bash
sudo docker login
sudo docker push tyttyttyt/pytorch-base:latest
```
