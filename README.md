# Local LLM with vLLM

## Installing CUDA drivers on WSL 

### Installing CUDA drivers

```shell
wget https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo apt-get update
sudo apt-get -y install cuda-toolkit-13-1
```


### Verifying CUDA drivers

Verify CUDA version on the right side.

```shell
nvidia-smi
```

### Install Docker

Download and install docker

```shell
curl https://get.docker.com | sh
sudo service docker start
```

Allow docker for non-root users as well

```shell
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
```


### Install CUDA Container Toolkit

Install the prerequisites for the instructions below
```shell
sudo apt-get update && sudo apt-get install -y --no-install-recommends \
   curl \
   gnupg2
```

Configure the production repository

```shell
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```

Update the packages list from the repository
```shell
sudo apt-get update

export NVIDIA_CONTAINER_TOOLKIT_VERSION=1.18.1-1
  sudo apt-get install -y \
      nvidia-container-toolkit=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
      nvidia-container-toolkit-base=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
      libnvidia-container-tools=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
      libnvidia-container1=${NVIDIA_CONTAINER_TOOLKIT_VERSION}
```

### Configure CUDA Container Toolkit

```shell
nvidia-ctk runtime configure --runtime=docker
sudo service docker restart
```

### Verify CUDA Container Toolkit

```shell
sudo docker run --rm --runtime=nvidia --gpus all ubuntu nvidia-smi
```

```shell
sudo docker run --gpus all nvcr.io/nvidia/k8s/cuda-sample:nbody nbody -gpu -benchmark
```

### Run vLLM 25.12
Check for compatibility with CUDA toolkit first. For instance vLLM 25.12 is compatible with CUDA toolkit 13.1.

```bash
docker run --gpus all -it --rm nvcr.io/nvidia/vllm:25.12-py3
```


### Reference Pages

#### CUDA drivers

- https://learn.microsoft.com/en-us/windows/ai/directml/gpu-cuda-in-wsl
- https://docs.nvidia.com/cuda/wsl-user-guide/index.html#getting-started-with-cuda-on-wsl
- https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=24.04&target_type=deb_network
- https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#meta-packages

#### NVIDIA Container Toolkit

- https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html#with-apt-ubuntu-debian
- https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html#configuration

#### vLLM

- https://docs.vllm.ai/en/stable/getting_started/installation/gpu/#pre-built-wheels
- https://docs.nvidia.com/deeplearning/frameworks/vllm-release-notes/rel-25-12.html

#### Docker
- https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository
- https://learn.microsoft.com/en-us/windows/wsl/tutorials/gpu-compute
- https://docs.docker.com/engine/install/linux-postinstall/

