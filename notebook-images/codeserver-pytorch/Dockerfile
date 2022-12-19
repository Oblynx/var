# Source: https://github.com/works-on-my-machine/pytorch-code-server/blob/main/Dockerfile
# Source#2: https://github.com/linuxserver/docker-code-server/blob/master/Dockerfile

# Base image source: https://hub.docker.com/r/nvidia/cuda
# This base image includes DNN libs which will be necessary for pytorch
# Sticking to CUDA 11 (not 12) because it's less likely to cause compatibility issues
FROM nvidia/cuda:11.8.0-cudnn8-runtime-ubuntu22.04

# NVidia settings
ENV NVIDIA_VISIBLE_DEVICES all
ENV NVIDIA_DRIVER_CAPABILITIES compute,utility
ENV LD_LIBRARY_PATH /usr/local/nvidia/lib:/usr/local/nvidia/lib64

RUN apt-get update
RUN apt-get install -y python3 python3-pip

# Install dependencies
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
  curl ca-certificates dumb-init htop sudo git bzip2 libx11-6 locales \
  man nano git procps openssh-client vim lsb-release net-tools jq libatomic1 netcat \
  && pip install poetry

RUN sed -i "s/# en_US.UTF-8/en_US.UTF-8/" /etc/locale.gen \
  && locale-gen
ENV LANG=en_US.UTF-8

# Create project directory
RUN mkdir /projects

# Create a non-root user
RUN adduser --disabled-password --gecos '' --shell /bin/bash coder \
  && chown -R coder:coder /projects
RUN echo "coder ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/90-coder

# Install fixuid
ENV ARCH=amd64
RUN curl -fsSL "https://github.com/boxboat/fixuid/releases/download/v0.4.1/fixuid-0.4.1-linux-$ARCH.tar.gz" | tar -C /usr/local/bin -xzf - && \
  chown root:root /usr/local/bin/fixuid && \
  chmod 4755 /usr/local/bin/fixuid && \
  mkdir -p /etc/fixuid && \
  printf "user: coder\ngroup: coder\n" > /etc/fixuid/config.yml

# Install code-server
WORKDIR /tmp

RUN CODE_RELEASE=$(curl -sX GET https://api.github.com/repos/coder/code-server/releases/latest \
      | awk '/tag_name/{print $4;exit}' FS='[""]' | sed 's|^v||'); \
    curl -fOL "https://github.com/coder/code-server/releases/download/v${CODE_RELEASE}/code-server_${CODE_RELEASE}_amd64.deb" && \
    dpkg -i ./code-server_${CODE_RELEASE}_amd64.deb && rm ./code-server_${CODE_RELEASE}_amd64.deb

# Install Pytorch
RUN pip3 install --quiet --extra-index-url https://download.pytorch.org/whl/cu116 torch torchvision torchaudio

# Install other base python dependencies
COPY /requirements.txt .
RUN pip3 install --quiet -r requirements.txt

COPY /root /

# Switch to default user
USER coder
ENV USER=coder
ENV HOME=/home/coder
WORKDIR /projects

EXPOSE 8443
#ENTRYPOINT ["/usr/bin/entrypoint.sh", "--bind-addr", "0.0.0.0:8443", "--cert", "--disable-telemetry", "."]
