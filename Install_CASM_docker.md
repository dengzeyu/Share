# Install CASM in a Docker container
- The purpose of this note is to install CASM on some new Linux distributions where some old libraries are needed but don't exist
- Here we use `docker` to build the image because it is compatible with Vscode plugin for software development
- Docker needs root privileges, we need to install a rootless version
- `ubuntu 20.04` is used as base environment
## Install Rootless version of Docker
- Run below will install docker in `${HOME}/bin`
```bash
curl -fsSL https://get.docker.com/rootless | sh
```
- You also need to change `~/.bashrc` as suggested by the output of the above script, shown below
- `YOUR_USER_ID` can be obtained using `id` 
- `YOUR_USER_NAME` is your user name
```bash
export DOCKER_HOST=unix:///run/user/YOUR_USER_ID/docker.sock
export PATH=/home/YOUR_USER_NAME/bin:$PATH
```
- Then you need to `source ~/.bashrc`
## Configure container using a Dockerfile
- Now you can import `CASM 0.3.x` directly from my DockerHub channel `dengzeyu/casm_0.3`
- `CASM` source code is in `/app/CASMcode` and `Miniconda` is installed in `/opt/conda`
- The Dockerfile below will install `Ubuntu 20.04`, `Miniconda 3`, and `CASM`
## Configure CASM configuration for development
- You can continue your development of `CASM` based on the version on my DockerHub channel
- Prepare a dockerfile:`casm_dev.dockerfile`
```dockerfile
# syntax=docker/dockerfile:1
FROM dengzeyu/casm_0.3:latest 
SHELL ["/bin/bash", "--login", "-c"]

# Get and build CASM
WORKDIR /app
RUN  conda remove casm casm-python casm-cpp --force
RUN git clone https://github.com/caneparesearch/CASMcode.git /app/CASMcode
WORKDIR /app/CASMcode
ENV CASM_DIRTY_IS_OK=1
RUN conda activate casm_dev && bash build.sh
RUN make install
RUN conda activate casm_dev && pip install python/casm
WORKDIR /
```
- Build this image name as `casm_dev`
```bash
docker build ./ -f casm_dev.dockerfile -t casm_dev
```
## Development using VSCode
- Use `vscode-remote` and `ssh` to a machine, you need to configure the machine in `~/.ssh/config`
- Install `Docker` in the remote machine
- Start a container from an image
- Then right-click the container and use `Attach Visual Studio Code`  to enter the container
---
## Run container
- More details can be found here: https://docs.docker.com/engine/reference/commandline/run/
- `-d` is detachable mode (background)
- `-i` is interactive mode
- `-t` is related to pseudo-TTY
```bash
docker run -it -d casm_dev
```
## Run a command in container
- Here I'm running `casm monte --desc` command in a conda environment `casm_dev` inside a container `silly_tu`
```bash
docker exec -it silly_tu conda run -n casm_dev casm monte --desc
```
## Enter the container
```bash
docker exec -it casm_dev bash
```
## Save Docker image
- Useful for running docker image on an HPC cluster using `singularity` where `docker` is not available
```bash
docker save casm:latest | gzip > casm.tar.gz
```
## Publish to DockerHub
- Here is an example of commit and push to your docker hub
- You need to run `docker login` and enter credentials before commit and push
```bash
docker commit NAME_OF_YOUR_CONTAINER casm:latest
docker tag casm:latest dengzeyu/casm_0.3:latest
docker push dengzeyu/casm_0.3:latest
```
