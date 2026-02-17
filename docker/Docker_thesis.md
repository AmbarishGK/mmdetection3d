# MoRAL Docker Image - Complete Guide

## Image: `ambarishgk007/moral-bevfusion-thesis:v1.0`

---

## What's Inside This Image

| Component | Version | Status |
|-----------|---------|--------|
| Ubuntu | 22.04 | ✅ |
| CUDA | 11.8 | ✅ |
| PyTorch | 2.1.2+cu118 | ✅ |
| MMDetection3D | 1.4.0 | ✅ |
| BEVFusion | Compiled | ✅ bev_pool built |
| spconv | cu118 | ✅ |
| nuScenes devkit | 1.1.11 | ✅ |
| Open3D | 0.19.0 | ✅ |
| BEVFusion checkpoint | 5239b1af | ✅ |

---

## PART 1: Fix Visualization (Run INSIDE Docker)

### The Problem
The demo ran but output was empty because `--snapshot` flag was missing.

### The Fix
```bash
cd /workspace/mmdetection3d

python projects/BEVFusion/demo/multi_modality_demo.py \
  demo/data/nuscenes/n015-2018-07-24-11-22-45+0800__LIDAR_TOP__1532402927647951.pcd.bin \
  demo/data/nuscenes/ \
  demo/data/nuscenes/n015-2018-07-24-11-22-45+0800.pkl \
  projects/BEVFusion/configs/bevfusion_lidar-cam_voxel0075_second_secfpn_8xb4-cyclic-20e_nus-3d.py \
  checkpoints/bevfusion/bevfusion_lidar-cam_voxel0075_second_secfpn_8xb4-cyclic-20e_nus-3d-5239b1af.pth \
  --cam-type all \
  --score-thr 0.2 \
  --out-dir /workspace/moral/results/bevfusion_demo \
  --snapshot
```

### Expected Output Files
```
/workspace/moral/results/bevfusion_demo/
├── CAM_FRONT.png          ← Front camera with 3D boxes
├── CAM_FRONT_LEFT.png     ← Front-left camera
├── CAM_FRONT_RIGHT.png    ← Front-right camera
├── CAM_BACK.png           ← Rear camera
├── CAM_BACK_LEFT.png      ← Rear-left camera
├── CAM_BACK_RIGHT.png     ← Rear-right camera
└── pts_vis.png            ← Point cloud BEV visualization
```

### Copy Results to Host (Run on HOST)
```bash
docker cp moral_container:/workspace/moral/results/bevfusion_demo ~/moral_workspace/results/
```

---

## PART 2: Save Your Docker Image (Run on HOST)

### Step 1: Find your container
```bash
docker ps -a
# Look for: moral_container or feeb793b6299
```

### Step 2: Commit container to image
```bash
# This saves EVERYTHING: compiled ops, checkpoints, your scripts
docker commit moral_container ambarishgk007/moral-bevfusion-thesis:v1.0
```

This will take 2-5 minutes (image is ~10-15GB).

### Step 3: Push to Docker Hub
```bash
docker login
# Enter your Docker Hub credentials

docker push ambarishgk007/moral-bevfusion-thesis:v1.0

# Also tag as latest
docker tag ambarishgk007/moral-bevfusion-thesis:v1.0 \
           ambarishgk007/moral-bevfusion-thesis:latest
docker push ambarishgk007/moral-bevfusion-thesis:latest
```

### Step 4: Verify on Docker Hub
Visit: https://hub.docker.com/r/ambarishgk007/moral-bevfusion-thesis

---

## PART 3: Pull and Run on AWS

### AWS Instance Requirements
- Instance type: `g4dn.xlarge` or `g4dn.2xlarge` (NVIDIA T4)
- AMI: Deep Learning AMI (Ubuntu 22.04)
- Storage: 100GB minimum
- Cost: ~$0.50-1.00/hour

### Step 1: Launch AWS Instance
Use Deep Learning AMI which already has:
- Docker
- NVIDIA drivers
- CUDA

### Step 2: Install NVIDIA Container Toolkit
```bash
# Only needed if NOT using Deep Learning AMI
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
sudo systemctl restart docker
```

### Step 3: Pull Your Image
```bash
docker pull ambarishgk007/moral-bevfusion-thesis:latest
```

### Step 4: Transfer nuScenes Data to AWS
```bash
# Option A: Use AWS S3 (recommended for large datasets)
aws s3 cp ~/Downloads/moral_project/mmdetection3d/data/ s3://your-bucket/nuscenes/ --recursive
# Then on AWS:
aws s3 cp s3://your-bucket/nuscenes/ ~/data/nuscenes/ --recursive

# Option B: Direct SCP (slower)
scp -r ~/Downloads/moral_project/mmdetection3d/data/ ubuntu@AWS_IP:~/data/
```

### Step 5: Run Container on AWS
```bash
# Create workspace
mkdir -p ~/moral_workspace ~/data/nuscenes

# Run container
docker run --gpus all -it \
  --name moral_aws \
  --shm-size=16gb \
  -v ~/moral_workspace:/workspace/moral \
  -v ~/data/nuscenes:/workspace/mmdetection3d/data/nuscenes \
  ambarishgk007/moral-bevfusion-thesis:latest
```

### Step 6: Verify on AWS
```bash
# Inside container on AWS
cd /workspace/mmdetection3d

# Test BEVFusion
python -c "from projects.BEVFusion.bevfusion import BEVFusion; print('Works')"

# Test CUDA
python -c "import torch; print(f'CUDA: {torch.cuda.is_available()}, GPU: {torch.cuda.get_device_name(0)}')"
```

---

## PART 4: What the Real Visualization Looks Like

When you run the demo correctly with `--snapshot`, you'll get:

### Camera Views (6 images)
- 3D bounding boxes projected onto each camera
- Color-coded by object class
- Shows: cars, pedestrians, trucks, barriers

### Point Cloud Visualization
- Colored point cloud from LiDAR
- 3D bounding boxes overlaid
- BEV perspective showing fused detection

### This is YOUR MoRAL output showing:
- LiDAR-enhanced BEV (your thesis contribution)
- Multi-sensor fusion working
- Real-time 3D object detection

---

## PART 5: Description for Thesis/Paper

### What These Visualizations Show:

**"Figure X: MoRAL BEVFusion Perception Output"**

> "The visualization demonstrates MoRAL's perception layer operating on nuScenes data. 
> The system processes synchronized LiDAR point clouds (Velodyne VLP-16 equivalent) 
> and 6-camera surround view images through BEVFusion, producing 3D bounding boxes 
> for all detected objects. The BEV representation encodes spatial geometry that 
> serves as grounded evidence for the LLM reasoning layer, enabling geometric 
> queries such as distance estimation and free-space identification that are 
> infeasible with camera-only approaches."

### Key Points to Highlight:
1. **Multi-sensor fusion**: LiDAR + 6 cameras
2. **BEV representation**: Top-down geometric view
3. **3D boxes**: Position, size, orientation
4. **LLM grounding**: These detections feed into LLM prompts

---

## PART 6: Complete Workflow Summary

```
Local Machine                    AWS Instance
─────────────────                ─────────────────
Docker container                 Pull same image
  │                                    │
  ├── BEVFusion inference              ├── Scale experiments
  ├── Visualization                    ├── Run full dataset
  ├── Feature extraction               ├── Training (optional)
  └── LLM integration                 └── Evaluation
```

---

## Troubleshooting

### If visualization still empty after --snapshot:
```bash
# Check if output dir was created
ls -la /workspace/moral/results/bevfusion_demo/

# If empty, run with verbose
python projects/BEVFusion/demo/multi_modality_demo.py \
  demo/data/nuscenes/n015-2018-07-24-11-22-45+0800__LIDAR_TOP__1532402927647951.pcd.bin \
  demo/data/nuscenes/ \
  demo/data/nuscenes/n015-2018-07-24-11-22-45+0800.pkl \
  projects/BEVFusion/configs/bevfusion_lidar-cam_voxel0075_second_secfpn_8xb4-cyclic-20e_nus-3d.py \
  checkpoints/bevfusion/bevfusion_lidar-cam_voxel0075_second_secfpn_8xb4-cyclic-20e_nus-3d-5239b1af.pth \
  --cam-type all \
  --score-thr 0.1 \
  --out-dir results/test \
  --snapshot 2>&1 | tee viz_log.txt

cat viz_log.txt
```

### If Docker push fails:
```bash
# Check image size first
docker images ambarishgk007/moral-bevfusion-thesis

# If >25GB, might timeout. Use incremental approach:
# 1. Start from base image
# 2. Only commit new layers
```

### If AWS GPU not detected:
```bash
# Check NVIDIA drivers
nvidia-smi

# Check Docker GPU access
docker run --gpus all nvidia/cuda:11.8.0-base-ubuntu22.04 nvidia-smi
```
