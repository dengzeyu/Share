#Install CASM 0.3.x
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
## Remove installed CASM
```bash
conda remove casm casm-python casm-cpp --force
```
## Compile and install CASM from source
```bash
bash build_install.sh
```
