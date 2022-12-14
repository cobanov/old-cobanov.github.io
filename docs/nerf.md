# NeRF Docker Setup

## Prerequisites

1. Docker
2. CUDA 11.3

## Install

1. Pull latest docker image

```bash
docker pull dromni/nerfstudio:<version_number>
```

2. Run docker image

```bash
docker run --gpus all -v /folder/of/your/data:/workspace/ -v /home/<YOUR_USER>/cache/:/home/user/.cache/ -p 7007:7007 --rm -it dromni/nerfstudio:0.1.13
```

Example for Windows:

```bash
docker run --gpus all -v D:\nerf:/workspace/ -v D:\nerf/:/home/user/.cache/ -p 7007:7007 --rm -it dromni/nerfstudio:0.1.13
```

```
-  Give the container access to nvidia GPU (required).
-  Mount a folder from the local machine into the container to be able to process them (required).
-  Mount cache folder to avoid re-downloading of models everytime (recommended).
-  Map port from local machine to docker container (required to access the web interface UI).
-  Remove container after it is closed (recommended).
-  Start container in interactive mode.
-  Docker image name
```

## Training First Model

### Don't forget to change the directory

```
cd /workspace
```

Be sure to do everything you do after this point in the `/workspace` directory.

### Start Training

```
ns-download-data nerfstudio --capture-name=poster
```

Because this code is often used too much, google drive can put a download restriction, read the error message and try to download the file in the link with your browser.

After downloading the file, you can copy the folder into the volume `D:\nerf\data\nerfstudio` you opened.

### Explore NeRF Studio

A web page similar to the link below will appear on your terminal screen, open this page and continue.

```
https://viewer.nerf.studio/versions/22-12-02-0/?websocket_url=ws://localhost:7007 
```

### Resume & Stop Training

You can stop the training with the `Ctrl + c` shortcut.

If you want to continue the training from where it left off, run the code below.

```
ns-train nerfacto --data data/nerfstudio/poster --trainer.load-dir {outputs/.../nerfstudio_models}
```

## Training on Custom Data

Put your files in the main directory of the volume you opened, you can also keep them in a folder in the main directory.

```
ns-process-data {video,images,polycam,insta360,record3d} --data {DATA_PATH} --output-dir {PROCESSED_DATA_DIR}
```

### Example

It points to the same place in the following two paths. one its location on the host and the other its location inside the container.

Raw File Path:

- `D:\nerf\raw_data\IMG_6070.MOV`
- `/workspace/raw_data/IMG_6070.MOV`

Target Folder Path:

- `D:\nerf\outputs\test_video`
- `/workspace/outputs`

```bash
ns-process-data video --data /workspace/raw_data/IMG_6070.MOV --output-dir /workspace/outputs
```

## Exporting Results


## KIRI Engine
