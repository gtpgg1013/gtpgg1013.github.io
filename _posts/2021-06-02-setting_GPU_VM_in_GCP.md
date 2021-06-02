---
title: "setting gpu vm in GCP"
excerpt: "맨땅에서 시작하는 tensorflow server 구축하기"

# permalink: /categories/kubernetes/4/

categories:
  - develop
  - deep learning
  - framework
tags:
  - python
  - tensorflow
last_modified_at: 2021-06-02T00:13:00-05:00

comments: true
---

## 개요

- 말 그대로 맨땅에서부터 시작해보는 텐서플로 서버구축

## 1. 구축

- GPU Quota 할당받고, VM instance 생성
  - ubuntu 18.04 LTS (python 3.6)
- ssh 접속

## 2. 설치 커맨드

```bash
# 파이썬 버전 업데이트
sudo apt update
sudo apt-get update
sudo apt-get install python3.8
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.6 1
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.8 2
sudo update-alternatives --config python3
#select ver 3.8
python3 -V
-> 그런데 이렇게 쓰면 뻑남.. 그냥 3.6 쓸란다...
```

```bash
# pip3 install
sudo apt update
sudo apt install python3-pip
sudo pip3 install --upgrade pip
vi ~/.bashrc
-> alias pip=pip3 맨 밑 추가
source .bashrc
```

```bash
# tensorflow 설치 / venv
pip install tensorflow==2.4.1 # 텐플 버전도 안맞으면 GPU 사용이 안되더라...
sudo apt update
sudo apt install python3-dev python3-pip python3-venv
python3 -m venv --system-site-packages ./venv
source ./venv/bin/activate
pip install --upgrade pip
deactivate
```

```bash
# GPU setting
# Add NVIDIA package repositories
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/cuda-ubuntu1804.pin
sudo mv cuda-ubuntu1804.pin /etc/apt/preferences.d/cuda-repository-pin-600
sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/7fa2af80.pub
sudo add-apt-repository "deb https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/ /"
sudo apt-get update

wget http://developer.download.nvidia.com/compute/machine-learning/repos/ubuntu1804/x86_64/nvidia-machine-learning-repo-ubuntu1804_1.0.0-1_amd64.deb

sudo apt install ./nvidia-machine-learning-repo-ubuntu1804_1.0.0-1_amd64.deb
sudo apt-get update

wget https://developer.download.nvidia.com/compute/machine-learning/repos/ubuntu1804/x86_64/libnvinfer7_7.1.3-1+cuda11.0_amd64.deb
sudo apt install ./libnvinfer7_7.1.3-1+cuda11.0_amd64.deb
sudo apt-get update

# Install development and runtime libraries (~4GB)
sudo apt-get install --no-install-recommends \
    cuda-11-0 \
    libcudnn8=8.0.4.30-1+cuda11.0  \
    libcudnn8-dev=8.0.4.30-1+cuda11.0

# Reboot. Check that GPUs are visible using the command: nvidia-smi

# Install TensorRT. Requires that libcudnn8 is installed above.
sudo apt-get install -y --no-install-recommends libnvinfer7=7.1.3-1+cuda11.0 \
    libnvinfer-dev=7.1.3-1+cuda11.0 \
    libnvinfer-plugin7=7.1.3-1+cuda11.0

apt install nvidia-cuda-toolkit
```

```python
# GPU 사용 확인 코드
from tensorflow.python.client import device_lib
device_lib.list_local_devices()
```

```bash
# CUDA / CUDNN으로 해보려하다가 실패
export PATH=/usr/local/cuda-11.0/targets/x86_64-linux/lib${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda-11.0/targets/x86_64-linux/lib/:${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
sudo chmod a+r /usr/local/cuda-11.0/targets/x86_64-linux/lib/libcuda*

sudo ln -s /usr/local/cuda-11.0/targets/x86_64-linux/lib/libcusolver.so.11 /home/gtpgg1013_dev1/.local/lib/python3.6/site-packages/tensorflow/python/libcusolver.so.11
```

```bash
# jupyter
sudo apt update
pip install jupyter
# jupyter notebook 원격접속 설정
jupyter notebook --generate-config
ipython
```

```python
# ipython
from notebook.auth import passwd
passwd()
# password 설정
# 그리고 암호화된 비밀번호 저장
```

```bash
# 원격접속 설정 관련 config 수정
vi ~/.jupyter/jupyter_notebook_config.py
c.NotebookApp.ip = #접속할 ip 입력(서버 ip)
c.NotebookApp.open_browser = False  # 원격 실행이므로 브라우저 실행X
c.NotebookApp.password = #미리 복사해놓은 암호화된 비밀번호 붙여넣기'
c.NotebookApp.password_required = True # 비밀번호 요구
c.NotebookApp.port = 8888 # 8888 포트번호 열기

# 실행
jupyter notebook

http://34.64.199.10:8888/
```

## 2. 교훈

- 삽질을 열심히 했다...
- 내일은 그냥 이걸로 tf server를 띄워봐야겠다.
- nvidia docker로 훨씬 쉽게 띄우는 방법이 있다고 하긴 하던데.
