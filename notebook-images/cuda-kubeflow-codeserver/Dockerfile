# Sources:
# - notebook: https://github.com/kubeflow/kubeflow/blob/master/components/example-notebook-servers/base/Dockerfile
# - cuda: https://gitlab.com/nvidia/container-images/cuda/-/blob/master/dist/11.8.0/ubuntu2204/base/Dockerfile

### >>>>> 1: Base packages + CUDA <<<<<

FROM ubuntu:22.04 as BASE_CUDA11

# common environemnt variables
ENV NB_USER jovyan
ENV NB_UID 1000
ENV NB_PREFIX /
ENV HOME /home/$NB_USER
ENV SHELL /bin/bash

# args - software versions
ARG KUBECTL_ARCH="amd64"
ARG KUBECTL_VERSION=v1.21.0
ARG S6_ARCH="amd64"
 # renovate: datasource=github-tags depName=just-containers/s6-overlay versioning=loose
ARG S6_VERSION=v2.2.0.3

# set shell to bash
SHELL ["/bin/bash", "-c"]

# install - usefull linux packages
RUN export DEBIAN_FRONTEND=noninteractive \
 && apt-get -yq update \
 && apt-get -yq install --no-install-recommends \
      apt-transport-https bash bzip2 ca-certificates curl git gnupg gnupg2 locales lsb-release nano \
      software-properties-common tzdata unzip vim wget zip \
      htop sudo man procps net-tools jq libatomic1 netcat python3 python3-pip

# Install CUDA 11

ENV NVARCH x86_64
ENV NVIDIA_REQUIRE_CUDA "cuda>=11.8 brand=tesla,driver>=450,driver<451 brand=tesla,driver>=470,driver<471 brand=unknown,driver>=470,driver<471 brand=nvidia,driver>=470,driver<471 brand=nvidiartx,driver>=470,driver<471 brand=geforce,driver>=470,driver<471 brand=geforcertx,driver>=470,driver<471 brand=quadro,driver>=470,driver<471 brand=quadrortx,driver>=470,driver<471 brand=titan,driver>=470,driver<471 brand=titanrtx,driver>=470,driver<471 brand=tesla,driver>=510,driver<511 brand=unknown,driver>=510,driver<511 brand=nvidia,driver>=510,driver<511 brand=nvidiartx,driver>=510,driver<511 brand=geforce,driver>=510,driver<511 brand=geforcertx,driver>=510,driver<511 brand=quadro,driver>=510,driver<511 brand=quadrortx,driver>=510,driver<511 brand=titan,driver>=510,driver<511 brand=titanrtx,driver>=510,driver<511 brand=tesla,driver>=515,driver<516 brand=unknown,driver>=515,driver<516 brand=nvidia,driver>=515,driver<516 brand=nvidiartx,driver>=515,driver<516 brand=geforce,driver>=515,driver<516 brand=geforcertx,driver>=515,driver<516 brand=quadro,driver>=515,driver<516 brand=quadrortx,driver>=515,driver<516 brand=titan,driver>=515,driver<516 brand=titanrtx,driver>=515,driver<516"
ENV NV_CUDA_CUDART_VERSION 11.8.89-1
ENV NV_CUDA_COMPAT_PACKAGE cuda-compat-11-8
ENV CUDA_VERSION 11.8.0

# install - CUDA11
RUN curl -fsSLO https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/${NVARCH}/cuda-keyring_1.0-1_all.deb \
 && dpkg -i cuda-keyring_1.0-1_all.deb \
 && apt-get install -y --no-install-recommends \
      cuda-cudart-11-8=${NV_CUDA_CUDART_VERSION} \
      ${NV_CUDA_COMPAT_PACKAGE} \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

# Required for nvidia-docker v1
RUN echo "/usr/local/nvidia/lib" >> /etc/ld.so.conf.d/nvidia.conf \
 && echo "/usr/local/nvidia/lib64" >> /etc/ld.so.conf.d/nvidia.conf

ENV PATH /usr/local/nvidia/bin:/usr/local/cuda/bin:${PATH}
ENV LD_LIBRARY_PATH /usr/local/nvidia/lib:/usr/local/nvidia/lib64

# nvidia-container-runtime
ENV NVIDIA_VISIBLE_DEVICES all
ENV NVIDIA_DRIVER_CAPABILITIES compute,utility

# CUDA runtime

ENV NV_CUDA_LIB_VERSION 11.8.0-1
ENV NV_NVTX_VERSION 11.8.86-1
ENV NV_LIBCUSPARSE_VERSION 11.7.5.86-1
ENV NV_LIBNPP_VERSION 11.8.0.86-1
ENV NV_LIBNPP_PACKAGE libnpp-11-8=${NV_LIBNPP_VERSION}

ENV NV_LIBCUBLAS_VERSION 11.11.3.6-1
ENV NV_LIBCUBLAS_PACKAGE_NAME libcublas-11-8
ENV NV_LIBCUBLAS_PACKAGE ${NV_LIBCUBLAS_PACKAGE_NAME}=${NV_LIBCUBLAS_VERSION}

ENV NCCL_VERSION 2.15.5-1
ENV NV_LIBNCCL_PACKAGE_NAME libnccl2
ENV NV_LIBNCCL_PACKAGE_VERSION 2.15.5-1
ENV NV_LIBNCCL_PACKAGE ${NV_LIBNCCL_PACKAGE_NAME}=${NV_LIBNCCL_PACKAGE_VERSION}+cuda11.8

ENV NV_CUDNN_VERSION 8.6.0.163
ENV NV_CUDNN_PACKAGE_NAME "libcudnn8"
ENV NV_CUDNN_PACKAGE "libcudnn8=$NV_CUDNN_VERSION-1+cuda11.8"


RUN apt-get update && apt-get install -y --no-install-recommends \
    cuda-libraries-11-8=${NV_CUDA_LIB_VERSION} \
    cuda-command-line-tools-11-8=${NV_CUDA_LIB_VERSION} \
    ${NV_LIBNPP_PACKAGE} \
    cuda-nvtx-11-8=${NV_NVTX_VERSION} \
    libcusparse-11-8=${NV_LIBCUSPARSE_VERSION} \
    ${NV_LIBCUBLAS_PACKAGE} \
    ${NV_LIBNCCL_PACKAGE} \
    ${NV_CUDNN_PACKAGE} \
    && rm -rf /var/lib/apt/lists/*

# Keep apt from auto upgrading the cublas and nccl packages. See https://gitlab.com/nvidia/container-images/cuda/-/issues/88
RUN apt-mark hold ${NV_LIBCUBLAS_PACKAGE_NAME} ${NV_LIBNCCL_PACKAGE_NAME} ${NV_CUDNN_PACKAGE}

# Add entrypoint items
COPY entrypoint.d/ /opt/nvidia/entrypoint.d/
COPY nvidia_entrypoint.sh /opt/nvidia/
ENV NVIDIA_PRODUCT_NAME="CUDA"
ENTRYPOINT ["/opt/nvidia/nvidia_entrypoint.sh"]


### >>>>> 2: Notebook <<<<<

FROM BASE_CUDA11 as NOTEBOOK_CUDA11

# install - s6 overlay
RUN export GNUPGHOME=/tmp/ \
 && curl -sL "https://github.com/just-containers/s6-overlay/releases/download/${S6_VERSION}/s6-overlay-${S6_ARCH}-installer" -o /tmp/s6-overlay-${S6_VERSION}-installer \
 && curl -sL "https://github.com/just-containers/s6-overlay/releases/download/${S6_VERSION}/s6-overlay-${S6_ARCH}-installer.sig" -o /tmp/s6-overlay-${S6_VERSION}-installer.sig \
 && gpg --keyserver keys.gnupg.net --keyserver pgp.surfnet.nl --recv-keys 6101B2783B2FD161 \
 && gpg -q --verify /tmp/s6-overlay-${S6_VERSION}-installer.sig /tmp/s6-overlay-${S6_VERSION}-installer \
 && chmod +x /tmp/s6-overlay-${S6_VERSION}-installer \
 && /tmp/s6-overlay-${S6_VERSION}-installer / \
 && rm /tmp/s6-overlay-${S6_VERSION}-installer.sig /tmp/s6-overlay-${S6_VERSION}-installer

# install - kubectl
RUN curl -sL "https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/${KUBECTL_ARCH}/kubectl" -o /usr/local/bin/kubectl \
 && curl -sL "https://dl.k8s.io/${KUBECTL_VERSION}/bin/linux/${KUBECTL_ARCH}/kubectl.sha256" -o /tmp/kubectl.sha256 \
 && echo "$(cat /tmp/kubectl.sha256) /usr/local/bin/kubectl" | sha256sum --check \
 && rm /tmp/kubectl.sha256 \
 && chmod +x /usr/local/bin/kubectl

# create user and set required ownership
RUN useradd -M -s /bin/bash -N -u ${NB_UID} ${NB_USER} \
 && mkdir -p ${HOME} \
 && chown -R ${NB_USER}:users ${HOME} \
 && chown -R ${NB_USER}:users /usr/local/bin \
 && chown -R ${NB_USER}:users /etc/s6

# set locale configs
RUN echo "en_US.UTF-8 UTF-8" > /etc/locale.gen \
 && locale-gen
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US.UTF-8
ENV LC_ALL en_US.UTF-8

USER $NB_UID

### >>>>> 2: Codeserver <<<<<

FROM NOTEBOOK_CUDA11
USER root

# args - software versions
ARG CODESERVER_PYTHON_VERSION=2022.21.13531008

# Install code-server
RUN CODE_RELEASE=$(curl -sX GET https://api.github.com/repos/coder/code-server/releases/latest \
      | awk '/tag_name/{print $4;exit}' FS='[""]' | sed 's|^v||'); \
    curl -fOL "https://github.com/coder/code-server/releases/download/v${CODE_RELEASE}/code-server_${CODE_RELEASE}_amd64.deb" && \
    dpkg -i ./code-server_${CODE_RELEASE}_amd64.deb && rm ./code-server_${CODE_RELEASE}_amd64.deb

# s6 - copy scripts
COPY --chown=jovyan:users s6/ /etc

USER $NB_UID

# install - requirements.txt
COPY --chown=jovyan:users requirements.txt /tmp
RUN python3 -m pip install -r /tmp/requirements.txt --quiet --no-cache-dir \
 && rm -f /tmp/requirements.txt \
 && chown -R ${NB_USER}:users ${CONDA_DIR} \
 && chown -R ${NB_USER}:users ${HOME}

# install - codeserver extensions
RUN curl -# -L -o /tmp/ms-python-release.vsix "https://github.com/microsoft/vscode-python/releases/download/${CODESERVER_PYTHON_VERSION}/ms-python-release.vsix" \
 && code-server --install-extension /tmp/ms-python-release.vsix \
 && code-server --list-extensions --show-versions

# s6 - 01-copy-tmp-home
USER root
RUN mkdir -p /tmp_home \
 && cp -r ${HOME} /tmp_home \
 && chown -R ${NB_USER}:users /tmp_home

USER $NB_UID
EXPOSE 8888
ENTRYPOINT ["/init"]
