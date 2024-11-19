FROM nvidia/cuda:11.3.1-cudnn8-devel-ubuntu20.04

ENV TZ=Europe/Berlin
ARG DEBIAN_FRONTEND=noninteractive
ENV TORCH_CUDA_ARCH_LIST="8.0 8.6"

# System dependencies
RUN apt update && \
    apt -y install git curl bc libgl1-mesa-glx libglib2.0-0 htop build-essential libopenblas-dev python3-dev

# Install Miniconda
RUN cd /root && curl -O -J https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh && bash Miniconda3-latest-Linux-x86_64.sh -b -p $HOME/miniconda3
ENV PATH="${PATH}:/root/miniconda3/bin"
RUN conda init bash

# Copy requirements
COPY requirements.txt /Mask3D/requirements.txt

# Create conda environment
RUN conda create -n mask3d python=3.10
RUN echo "conda activate mask3d" >> $HOME/.bashrc
ENV CONDA_DEFAULT_ENV=mask3d
SHELL ["conda", "run", "-n", "mask3d", "/bin/bash", "-c"]

# Setup environment with some workarounds
RUN pip install pip==23.1
RUN pip install "cython<3.0.0" && pip install --no-build-isolation pyyaml==5.4.1
RUN pip install -r /Mask3D/requirements.txt
RUN pip install --no-build-isolation pycocotools==2.0.4

#######################################
# Install torch
RUN pip install torch==1.12.1+cu113 torchvision==0.13.1+cu113 --extra-index-url https://download.pytorch.org/whl/cu113
RUN pip install torch-scatter -f https://data.pyg.org/whl/torch-1.12.1+cu113.html
RUN pip install --no-deps torchmetrics==0.11.4
RUN pip install --no-deps pytorch-lightning==1.7.2

#######################################
# Install detectron2
RUN pip3 install 'git+https://github.com/facebookresearch/detectron2.git@710e7795d0eeadf9def0e7ef957eea13532e34cf' --no-deps

#######################################
# Install MinkowskiEngine
RUN git clone --recursive "https://github.com/NVIDIA/MinkowskiEngine" /MinkowskiEngine && \
    cd /MinkowskiEngine && \
    git checkout 02fc608bea4c0549b0a7b00ca1bf15dee4a0b228 && \
    python setup.py install --force_cuda --blas=openblas

#######################################
# Install Segmentator
RUN git clone https://github.com/ScanNet/ScanNet.git /ScanNet && \
    cd /ScanNet/Segmentator && \
    git checkout 3e5726500896748521a6ceb81271b0f5b2c0e7d2 && \
    make

#######################################
# Install PointNet2
RUN git clone https://github.com/cvg/Mask3D.git /tmp/Mask3D
RUN cp -R /tmp/Mask3D/third_party/pointnet2/ /pointnet2
RUN rm -rf /tmp/Mask3D
RUN cd /pointnet2 && \
    python setup.py install

#######################################
# Set working directory
WORKDIR /Mask3D