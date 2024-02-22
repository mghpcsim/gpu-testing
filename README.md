# Benchmarks

This repo hosts benchmark scripts to benchmark GPUs using NVIDIA GPU-Accelerated Containers. 

## Overview

## Fresh AWS GPU instance

Create a new EC2 instance with a GPU using Rocky 9. 
This example uses a g4dn.4xlarge instance and ami-00876579da91da598.
The root volume is 50 GB (gp2) and this instance type comes with a dedicated fisk
of ~200 GB that will be used for datasets.


### Initial instance configuration
```
# update the system
sudo dnf  -y upgrade --refresh

# create a filesystem, mount at /scratch, and add to fstab
sudo mkfs.xfs /dev/nvme1n1 && sudo mkdir /scratch && sudo mount /dev/nvme1n1 /scratch && sudo chown $USER:$USER /scratch && echo "/dev/nvme1n1 /scratch xfs defaults 0 0" | sudo tee -a /etc/fstab

# add crb and epel repos
sudo dnf config-manager  --set-enabled crb
sudo dnf install -y    https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm     https://dl.fedoraproject.org/pub/epel/epel-next-release-latest-9.noarch.rpm

# install useful packages
sudo dnf -y install vim-enhanced wget git screen

# turn off selinux for next reboot
sudo perl -pi -e "s/SELINUX=enforcing/SELINUX=disabled/" /etc/selinux/config 

# reboot
sudo reboot
```

### Installing nvidia drivers
```
# add Nvidia repos
sudo dnf config-manager --add-repo http://developer.download.nvidia.com/compute/cuda/repos/rhel9/$(uname -i)/cuda-rhel9.repo

# install prereqs for dkms drivers
sudo dnf  -y install kernel-headers-$(uname -r) kernel-devel-$(uname -r) tar bzip2 make automake gcc gcc-c++ pciutils elfutils-libelf-devel libglvnd-opengl libglvnd-glx libglvnd-devel acpid pkgconfig dkms

# install latest version of the closed source driver
sudo dnf -y module install nvidia-driver:latest-dkms
# OR the latest version of the open source driver
sudo dnf module install nvidia-driver:open-dkms

# reboot the system 
sudo reboot
```

### Setup docker, containerd, and nvidia container toolkit
```
# add docker repo
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# install docker and containerd, add $USER to docker group and load new group
sudo dnf -y install docker-ce docker-ce-cli containerd.io
sudo usermod -aG docker $(whoami)
newgrp docker

# enable, start, and test docker
sudo systemctl enable docker --now
sudo systemctl status docker
docker run hello-world

# add nvidia container toolkit repo
curl -s -L https://nvidia.github.io/libnvidia-container/stable/rpm/nvidia-container-toolkit.repo |   sudo tee /etc/yum.repos.d/nvidia-container-toolkit.repo
sudo yum-config-manager --enable nvidia-container-toolkit-experimental
sudo dnf install -y nvidia-container-toolkit

# configure nvitida toolkit to use containerd runtime and restart containerd and docker
sudo nvidia-ctk runtime configure --runtime=containerd
sudo systemctl restart containerd docker
```

### Setup and run benchmarks
# Fetch benchmark tool for SSD_AMP
wget https://raw.githubusercontent.com/mghpcsim/gpu-testing/master/pytorch/Makefile

# Set the path to store code and data
export PATH_STORAGE=/scratch

# Clone repos and pull docker image
make install PATH_STORAGE=${PATH_STORAGE}

# Pepare data
make create_data PATH_STORAGE=${PATH_STORAGE}

# Run benchmark
make benchmark PATH_STORAGE=${PATH_STORAGE}


### g4dn.4xlarge results

#### closed GPU drivers
```
DLL 2024-02-07 22:45:30.189486 - () avg_img/sec : 78.47830359568704 images/s med_img/sec : 78.46226943054549 images/s min_img/sec : 78.20951905168042 images/s max_img/sec : 78.85041334209333 images/s
Done benchmarking. Total images: 19200  total time: 244.654     Average images/sec: 78.478      Median images/sec: 78.462
Training performance = 78.46227264404297 FPS
DLL 2024-02-07 22:45:30.192908 - (0,) time : 319.8903548717499
```

#### open GPU drivers
```
DLL 2024-02-07 23:29:49.648111 - () avg_img/sec : 77.4660948215365 images/s med_img/sec : 77.45894054556102 images/s min_img/sec : 77.1730385428987 images/s max_img/sec : 77.84095169162816 images/s
Done benchmarking. Total images: 19200  total time: 247.850     Average images/sec: 77.466      Median images/sec: 77.459
Training performance = 77.45893859863281 FPS
DLL 2024-02-07 23:29:49.651626 - (0,) time : 323.97230982780457
```
