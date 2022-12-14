# NerfStudio Installation

## Docker

1. Install Docker
2. Pull cuda.11.3 image

```
docker pull nvidia/cuda:11.3.0-base-ubuntu20.04
```

```
docker run --name nerf_docker --gpus all -p 3000:3000 -it cobanovgithub
```

```
apt-get update
apt-get install wget
```
```

ns-download-data --dataset=nerfstudio --capture=poster
```
```
ns-train nerfacto --viewer.websocket-port 3000 nerfstudio-data --data data/nerfstudio/poster --downscale-factor 4
```
## CONDA

Download the latest shell script
wget <https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh>

Make the miniconda installation script executable
chmod +x Miniconda3-latest-Linux-x86_64.sh

Run miniconda installation script
./Miniconda3-latest-Linux-x86_64.sh

Create and activate an conda environment
To create a conda environment, run conda create -n newenv

You can also create the environment from a file like environment.yml, you can use use the conda env create -f command: conda env create -f environment.yml. The environment name will be the directory name.

```
source ~/.bashrc
```

## Create environment

```
conda create --name nerfstudio -y python=3.8
conda activate nerfstudio
python -m pip install --upgrade pip
```

## TinyCudaNN

```
apt-get install build-essential git
source ~/.bashrc
```

pip install torch==1.12.1+cu113 torchvision==0.13.1+cu113 -f <https://download.pytorch.org/whl/torch_stable.html>
