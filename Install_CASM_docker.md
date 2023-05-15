# Install CASM in a Docker container
- The purpose of this note is to install CASM on some new Linux distributions
- Docker needs root privileges, we need to install a rootless version
## Install Rootless version of Docker
```bash
curl -fsSL https://get.docker.com/rootless | sh
```
## Configure container using a dockerfile
- `casm_dev.docker`
```dockerfile
# syntax=docker/dockerfile:1
FROM ubuntu:20.04

ENV LANG=C.UTF-8 LC_ALL=C.UTF-8
ENV PATH /opt/conda/bin:$PATH

# Install dependencies and Miniconda
RUN apt-get update --fix-missing && \
    apt-get install -y wget bzip2 ca-certificates curl git && \
	apt-get install -y vim build-essential autoconf m4 bash-completion libtool pkg-config && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

RUN wget --quiet https://repo.anaconda.com/miniconda/Miniconda3-4.5.11-Linux-x86_64.sh -O ~/miniconda.sh && \
    /bin/bash ~/miniconda.sh -b -p /opt/conda && \
    rm ~/miniconda.sh && \
    /opt/conda/bin/conda clean -tipsy && \
    ln -s /opt/conda/etc/profile.d/conda.sh /etc/profile.d/conda.sh && \
    echo ". /opt/conda/etc/profile.d/conda.sh" >> ~/.bashrc && \
    echo "conda activate base" >> ~/.bashrc

ENV TINI_VERSION v0.16.1
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /usr/bin/tini
RUN chmod +x /usr/bin/tini

ENTRYPOINT [ "/usr/bin/tini", "--" ]
CMD [ "/bin/bash" ]

WORKDIR /app/

# Make RUN commands use `bash --login`:
SHELL ["/bin/bash", "--login", "-c"]

# Install CASM dependencies
COPY casm_dev.yaml ./
RUN conda env create -f casm_dev.yaml
RUN conda remove casm casm-python casm-cpp --force
# Initialize conda in bash config fiiles:
RUN conda init bash

# Activate the environment, and make sure it's activated:
RUN echo "conda activate casm_dev" > ~/.bashrc

# Get and build CASM
WORKDIR /app
RUN git clone git@github.com:caneparesearch/CASMcode.git:/app/CASMcode
WORKDIR /app/CASMcode
RUN bash build_install.sh
```
- `casm_dev.yaml` file is shown below
```yaml
name: casm_dev
channels:
  - prisms-center
  - conda-forge
  - bpuchala/label/dev
  - defaults
dependencies:
  - _libgcc_mutex=0.1=main
  - astroid=2.2.5=py36_0
  - binutils_impl_linux-64=2.31.1=h6176602_1
  - binutils_linux-64=2.31.1=h6176602_7
  - blas=1.0=mkl
  - bokeh=1.2.0=py36_0
  - bzip2=1.0.8=h7b6447c_0
  - ca-certificates=2019.5.15=1
  - casm=0.3.dev269+gd07b42=condagcc_0
  - casm-boost=1.66.0=condagcc_0
  - casm-cpp=0.3.dev269+gd07b42=condagcc_0
  - casm-python=0.3.dev269+gd07b42=0
  - certifi=2019.6.16=py36_1
  - deap=1.3.0=py36hb3f55d8_0
  - freetype=2.9.1=h8a8886c_1
  - future=0.17.1=py36_0
  - gcc_impl_linux-64=7.3.0=habb00fd_1
  - gcc_linux-64=7.3.0=h553295d_7
  - gfortran_impl_linux-64=7.3.0=hdf63c60_1
  - gfortran_linux-64=7.3.0=h553295d_7
  - gxx_impl_linux-64=7.3.0=hdf63c60_1
  - gxx_linux-64=7.3.0=h553295d_7
  - intel-openmp=2019.4=243
  - isort=4.3.21=py36_0
  - jinja2=2.10.1=py36_0
  - joblib=0.13.2=py36_0
  - jpeg=9b=h024ee3a_2
  - lazy-object-proxy=1.4.2=py36h7b6447c_0
  - libedit=3.1.20181209=hc058e9b_0
  - libffi=3.2.1=hd88cf55_4
  - libgcc-ng=9.1.0=hdf63c60_0
  - libgfortran-ng=7.3.0=hdf63c60_0
  - libpng=1.6.37=hbc83047_0
  - libstdcxx-ng=9.1.0=hdf63c60_0
  - libtiff=4.0.10=h2733197_2
  - markupsafe=1.1.1=py36h7b6447c_0
  - mccabe=0.6.1=py36_1
  - mkl=2019.4=243
  - mkl-service=2.0.2=py36h7b6447c_0
  - mkl_fft=1.0.12=py36ha843d7b_0
  - mkl_random=1.0.2=py36hd81dba3_0
  - mock=3.0.5=py36_0
  - ncurses=6.1=he6710b0_1
  - numpy=1.16.4=py36h7e9f1db_0
  - numpy-base=1.16.4=py36hde5b4d6_0
  - olefile=0.46=py36_0
  - openssl=1.1.1d=h7b6447c_1
  - packaging=19.0=py36_0
  - pandas=0.24.2=py36he6710b0_0
  - pillow=6.1.0=py36h34e0f95_0
  - prisms-jobs=4.0.2=py36_0
  - pylint=2.3.1=py36_0
  - pyparsing=2.4.0=py_0
  - python=3.6.8=h0371630_0
  - python-dateutil=2.8.0=py36_0
  - pytz=2019.1=py_0
  - pyyaml=5.1.1=py36h7b6447c_0
  - readline=7.0=h7b6447c_5
  - scikit-learn=0.21.2=py36hd81dba3_0
  - scipy=1.2.1=py36h7c811a0_0
  - setuptools=41.0.1=py36_0
  - sh=1.12.14=py36_0
  - six=1.12.0=py36_0
  - sqlite=3.28.0=h7b6447c_0
  - tk=8.6.8=hbc83047_0
  - tornado=6.0.3=py36h7b6447c_0
  - typed-ast=1.3.4=py36h7b6447c_0
  - wheel=0.33.4=py36_0
  - wrapt=1.11.2=py36h7b6447c_0
  - xz=5.2.4=h14c3975_4
  - yaml=0.1.7=had09818_2
  - zlib=1.2.11=h7b6447c_3
  - zstd=1.3.7=h0b5b093_0
```
## Build container
- Make sure that current directory has `casm_dev.docker` and conda environment file: `casm_dev.yaml`
```bash
docker build ./ -f casm_dev.docker -t casm_dev
```

# Below are deprecated for Docker Installation
## Create a container
- Use `docker` to create a container (`ubuntu 20.04` is installed here with a container named with `casm_dev_env`)
```bash
docker run -it -d --name casm_dev_env ubuntu:20.04
```
## Enter the container
```bash
docker exec -it casm_dev_env bash
```
## Install all requirements
- select Asia/Singapore for tzdata configuration
```
apt update
apt install vim build-essential autoconf m4 bash-completion zlib1g zlib1g-dev libtool pkg-config git
```
## Install Miniconda and CASM 
- Follow the same procedures above
- Use the `.yaml` file as shown above
```
git clone https://github.com/prisms-center/CASMcode.git

cd CASMcode

git checkout be1c21b

conda env create -f casm_dev.yaml

conda activate casm_dev

conda remove casm casm-python casm-cpp --force

bash build_install.sh
```