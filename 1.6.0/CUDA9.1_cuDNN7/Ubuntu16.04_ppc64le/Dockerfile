FROM nvidia/cuda-ppc64le:9.1-cudnn7-devel-ubuntu16.04
MAINTAINER Yasuhiro Nakajima <e35294@jp.ibm.com>

# install base
RUN apt-get update && DEBIAN_FRONTEND=noninteractive  apt-get install -y \
    wget curl git apt-transport-https apt-utils vim htop tree zip unzip \
    openssh-server iputils-ping net-tools language-pack-ja-base language-pack-ja fonts-ipafont-mincho build-essential openjdk-8-jdk && \
    apt-get clean

# install python3 module
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y \
    python python3 python3-pip python3-numpy python3-scipy python3-matplotlib python3-dev python3-wheel && \
    apt-get clean


# upgrade module
RUN DEBIAN_FRONTEND=noninteractive apt-get upgrade -y && \
    apt-get clean

#setup python3
RUN update-alternatives --install /usr/bin/python python /usr/bin/python2 10
RUN update-alternatives --install /usr/bin/python python /usr/bin/python3 11

#Build Bazel 0.10.0 for Tensorflow v1.6.0

RUN cd /tmp && \
    mkdir bazel && \
    cd bazel && \
    wget https://github.com/bazelbuild/bazel/releases/download/0.10.0/bazel-0.10.0-dist.zip && \
    unzip bazel-0.10.0-dist.zip && \
    chmod -R +w .&& \
    ./compile.sh 

ENV PATH=$PATH:/tmp/bazel/output/
ENV LD_LIBRARY_PATH=/usr/local/cuda/extras/CUPTI/lib64:$LD_LIBRARY_PATH


RUN cd /tmp && \
    git clone --recurse-submodules https://github.com/tensorflow/tensorflow && \
    cd tensorflow && \ 
    git checkout v1.6.0 && \
    export CC_OPT_FLAGS="-mcpu=power8 -mtune=power8" && \
    export GCC_HOST_COMPILER_PATH=/usr/bin/gcc && \
    export PYTHON_BIN_PATH=/usr/bin/python && \
    export USE_DEFAULT_PYTHON_LIB_PATH=1 && \
    export TF_NEED_GCP=1 && \
    export TF_NEED_HDFS=1 && \
    export TF_NEED_JEMALLOC=1 && \
    export TF_NEED_KAFKA=0 && \
    export TF_NEED_TENSORRT=0 && \
    export TF_ENABLE_XLA=0 && \
    export TF_NEED_OPENCL=0 && \
    export TF_NEED_CUDA=1 && \
    export TF_CUDA_VERSION=9.1 && \
    export CUDA_TOOLKIT_PATH=/usr/local/cuda-9.1 && \
    export TF_CUDA_COMPUTE_CAPABILITIES=6.0,7.0 && \
    export CUDNN_INSTALL_PATH=/usr/local/cuda-9.1 && \
    export TF_CUDNN_VERSION=7 && \
    export TF_NEED_MKL=0 && \
    export TF_NEED_VERBS=0 && \
    export TF_NEED_MPI=0 && \
    export TF_CUDA_CLANG=0 && \
    export TF_NEED_S3=1 && \
    export TF_NEED_OPENCL_SYCL=0 && \
    export TF_NEED_GDR=0 && \
    export TF_SET_ANDROID_WORKSPACE=0 && \
    ./configure && \
    touch /usr/include/stropts.h && \
    ln -s /usr/local/cuda/lib64/stubs/libcuda.so /usr/local/cuda/lib64/stubs/libcuda.so.1 && \
    LD_LIBRARY_PATH=/usr/local/cuda/lib64/stubs:${LD_LIBRARY_PATH} \
    tensorflow/tools/ci_build/builds/configured GPU \
    bazel build --copt="-mcpu=power8" --copt="-mtune=power8" --config=cuda //tensorflow/tools/pip_package:build_pip_package --verbose_failures --local_resources=32000,8,1.0 && \
    bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg && \
    pip3 install /tmp/tensorflow_pkg/tensorflow-1.6.0*

# Build nccl ver1

RUN cd / && \
    git clone https://github.com/NVIDIA/nccl.git && \
    cd /nccl  && \
    make CUDA_HOME=/usr/local/cuda-9.1 test 

ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/nccl/build/lib

# setup ssh
RUN mkdir /var/run/sshd
RUN echo root:$(echo "passw0rd") | chpasswd
RUN sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
RUN sed -i '/^AcceptEnv/s/^/#/' /etc/ssh/sshd_config
RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd
RUN echo "export PATH=/usr/local/nvidia/bin:/usr/local/cuda/bin:$PATH" >> /root/.bashrc && \
    echo "export LD_LIBRARY_PATH=/usr/local/nvidia/lib:/usr/local/nvidia/lib64:/nccl/build/lib" >> /root/.bashrc


# setup timezone and locale
ENV TZ=Asia/Tokyo
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
RUN echo "export LANG=ja_JP.UTF-8" >> /root/.bashrc
RUN echo "export LANGUAGE=ja_JP.UTF-8" >> /root/.bashrc
ENV LANGUAGE=ja_JP
ENV LANG=ja_JP.UTF-8

# import python modules
RUN pip3 install IPython==5.3
RUN pip3 install jupyter
RUN mkdir /root/.jupyter
RUN echo "c.NotebookApp.ip = '*'" > ~/.jupyter/jupyter_notebook_config.py

# Delete Directory
RUN rm -rf /tmp/tensorflow /tmp/bazel /tmp/tensorflow_pkg /root/.cache /var/lib/apt/lists/* 

CMD []
