# Install CASM in a Docker container
- The purpose of this note is to install CASM on some new Linux distributions where some old libraries are needed but don't exist
- Here we use `docker` to build the image because it is compatible with Vscode plugin for software development
- Docker needs root privileges, we need to install a rootless version
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
## configure a image using a Dockerfile
- You can import `CASM 0.3.x` directly from my Docker Hub channel `dengzeyu/casm.0.3.x:latest`
- For the CASM with source code stored in `/app/CASMcode`, you can pull `dengzeyu/casm.0.3.x:latest-dev`
- `CASM` source code is in `/app/CASMcode` and `Miniconda` is installed in `/opt/conda`
- The Dockerfile below will install `Ubuntu 20.04`, `Miniconda 3`, and `CASM`
## build a CASM  image for development
- You can do development of `CASM`, e.g. changing the source code, based on the version on my Docker Hub channel
- Prepare a Dockerfile:`casm_dev.dockerfile`
```dockerfile
# syntax=docker/dockerfile:1
FROM dengzeyu/casm.0.3.x:latest 
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
RUN conda activate casm_dev && bash clean.sh
WORKDIR /
```
- Build this image name as `casm_dev`
```bash
docker build ./ -f casm_dev.dockerfile -t casm_dev
```
## Development using VS Code
- Use `vscode-remote` and `ssh` to a machine, you need to configure the machine in `~/.ssh/config`
- Install `docker` in the remote machine
- Start a container from an image
- Then right-click the container and use `Attach Visual Studio Code`  to enter the container
---
## Create a container
- More details can be found here: https://docs.docker.com/engine/reference/commandline/run/
- Every time you execute `docker run` you will create a new container
- `-d` run in detachable mode (background)
- `-i` run in interactive mode
- `-t` is related to pseudo-TTY
- `--name` is the name of this container
- `--storage-opt size=200G` limit the storage to 200 GB
- `-v` mount `/home/jerry`  outside the container to `/home/jerry` inside the container
- `dengzeyu/casm.0.3.x:latest` is the image that we want to run
- `--rm` automatically remove this container when it is stopped
```bash
docker run -itd -v /home/jerry:/home/jerry --storage-opt size=200G --name casm_dev_container dengzeyu/casm.0.3.x:latest
```
## Run a command in container outside of container
- Here I'm running `casm init` command in a conda environment `casm_dev` inside a container `casm_dev_container` in the current path outside of container: `$PWD`
```bash
docker exec -it -w $PWD casm_dev_container conda run -n casm_dev casm init
```
## Enter the container
```bash
docker exec -it casm_dev_container bash
```
## Save Docker image
- Useful for running docker image on an HPC cluster using `singularity` where `docker` is not available
```bash
docker save casm:latest | gzip > casm.tar.gz
```
## Publish to Docker Hub
- Here is an example of commit and push to your docker hub
- You need to run `docker login` and enter credentials before commit and push
```bash
docker commit NAME_OF_YOUR_CONTAINER casm.0.3.x:latest
docker tag casm.0.3.x:latest dengzeyu/casm.0.3.x:latest
docker push dengzeyu/casm.0.3.x:latest
```
