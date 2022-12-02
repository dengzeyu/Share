# Install CASM 0.3.x
- This is the easiest way to install CASM from source. We first install original CASM using Anaconda from their repository. This will install all dependencies, which is very hard to install manually. Then we uninstall the CASM from repository and reinstall our own version of CASM locally.
## Clone and install CASM 0.3.x
```bash
git clone https://github.com/prisms-center/CASMcode.git
git checkout be1c21b
```
## Create conda environment and install all packages
```bash
conda create -n casm_0.3 --override-channels -c bpuchala/label/dev -c prisms-center -c defaults -c conda-forge casm=0.3.dev269+gd07b42=condagcc_0 casm-boost=1.66.0=condagcc_0 casm-cpp=0.3.dev269+gd07b42=condagcc_0 casm-python=0.3.dev269+gd07b42=0 scikit-learn=0.21.2=py36hd81dba3_0 bokeh=1.2.0=py36_0 python=3.6.8=h0371630_0
```
## Enable CASM environment
```bash
conda activate casm_0.3
```
## Remove the installed CASM
```bash
conda remove casm casm-python casm-cpp --force
```
## Compile and install CASM from source
- Make sure your environment has GCC installed. I have gcc-7.4.0 and it works.
```bash
cd CASMcode
bash build_install.sh
```
