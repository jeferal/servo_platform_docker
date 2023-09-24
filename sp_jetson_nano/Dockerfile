# Use the base image with the desired platform (linux/arm64)
FROM --platform=linux/arm64 nvcr.io/nvidia/l4t-tensorrt:r8.0.1-runtime

# Set the environment variable for non-interactive mode
ENV DEBIAN_FRONTEND=noninteractive

# Update the system
RUN apt-get update && apt-get upgrade -y

# Install generic tools
# install the dependencies

RUN apt-get install -y build-essential cmake git unzip pkg-config zlib1g-dev \
    libjpeg-dev libjpeg8-dev libjpeg-turbo8-dev libpng-dev libtiff-dev \
    libavcodec-dev libavformat-dev libswscale-dev libglew-dev \
    libgtk2.0-dev libgtk-3-dev libcanberra-gtk* \
    python3-dev python3-numpy python3-pip \
    libxvidcore-dev libx264-dev libgtk-3-dev \
    libtbb2 libtbb-dev libdc1394-22-dev libxine2-dev \
    gstreamer1.0-tools libv4l-dev v4l-utils qv4l2 \
    libgstreamer-plugins-base1.0-dev libgstreamer-plugins-good1.0-dev \
    libavresample-dev libvorbis-dev libxine2-dev libtesseract-dev \
    libfaac-dev libmp3lame-dev libtheora-dev libpostproc-dev \
    libopencore-amrnb-dev libopencore-amrwb-dev \
    libopenblas-dev libatlas-base-dev libblas-dev \
    liblapack-dev liblapacke-dev libeigen3-dev gfortran \
    libhdf5-dev protobuf-compiler \
    libprotobuf-dev libgoogle-glog-dev libgflags-dev

# Install wget
RUN apt-get install -y wget

# Download opencv, opencv_contrib 4.8.0 and opencv_zoo
RUN cd /opt && \
    wget -O opencv.zip https://github.com/opencv/opencv/archive/refs/tags/4.8.0.zip && \
    wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/refs/tags/4.8.0.zip && \
    git clone https://github.com/opencv/opencv_zoo.git && \
    unzip opencv.zip && \
    unzip opencv_contrib.zip && \
    mv opencv-4.8.0 opencv && \
    mv opencv_contrib-4.8.0 opencv_contrib && \
    rm opencv.zip && \
    rm opencv_contrib.zip

# Create a virtual environment for opencv
RUN pip3 install virtualenv && \
    virtualenv -p python3 /opt/opencv-venv && \
    sh /opt/opencv-venv/bin/activate && \
    /opt/opencv-venv/bin/pip3 install numpy

ENV CUDNN_VERSION 7.3.1.28

ENV CUDNN_PKG_VERSION=${CUDA_VERSION}-1

LABEL com.nvidia.cudnn.version="${CUDNN_VERSION}"

ARG libcudnn7_URL=https://onedrive.live.com/download?cid=54AD8562A32D8752&resid=54AD8562A32D8752%21376196&authkey=ADTDdL0bhMWq4vM
ARG libcudnn7_dev_URL=https://onedrive.live.com/download?cid=54AD8562A32D8752&resid=54AD8562A32D8752%21376197&authkey=APizXm-di7JPR0Y
ARG libcudnn7_doc_URL=https://onedrive.live.com/download?cid=54AD8562A32D8752&resid=54AD8562A32D8752%21376195&authkey=ADqH53K9oRnkO-8

RUN curl -sL $libcudnn7_URL -o libcudnn7_$CUDNN_VERSION-1+cuda10.0_arm64.deb && \
    echo "92867c0a495f84ec11d108f84b776620 libcudnn7_$CUDNN_VERSION-1+cuda10.0_arm64.deb" | md5sum -c - && \
    dpkg -i libcudnn7_$CUDNN_VERSION-1+cuda10.0_arm64.deb && \
    rm libcudnn7_$CUDNN_VERSION-1+cuda10.0_arm64.deb

RUN curl -sL $libcudnn7_dev_URL -o libcudnn7-dev_$CUDNN_VERSION-1+cuda10.0_arm64.deb && \
    echo "dd0fbfa225b2374b946febc98e2cdec4 libcudnn7-dev_$CUDNN_VERSION-1+cuda10.0_arm64.deb" | md5sum -c - && \
    dpkg -i libcudnn7-dev_$CUDNN_VERSION-1+cuda10.0_arm64.deb && \
    rm libcudnn7-dev_$CUDNN_VERSION-1+cuda10.0_arm64.deb

RUN curl -sL $libcudnn7_doc_URL -o libcudnn7-doc_$CUDNN_VERSION-1+cuda10.0_arm64.deb && \
    echo "9478c16ceeaaca937d4d26b982e48bd1 libcudnn7-doc_$CUDNN_VERSION-1+cuda10.0_arm64.deb" | md5sum -c - && \
    dpkg -i libcudnn7-doc_$CUDNN_VERSION-1+cuda10.0_arm64.deb && \
    rm libcudnn7-doc_$CUDNN_VERSION-1+cuda10.0_arm64.deb

RUN cd /opt && \
    mkdir build && \
    cd build && \
    cmake \ 
        -D CMAKE_BUILD_TYPE=RELEASE \
        -D CMAKE_INSTALL_PREFIX=/usr \
        -D OPENCV_EXTRA_MODULES_PATH=/opt/opencv_contrib/modules \
        -D EIGEN_INCLUDE_PATH=/usr/include/eigen3 \
        -D WITH_OPENCL=OFF \
        -D WITH_CUDA=ON \
        -D CUDA_ARCH_BIN=5.3 \
        -D CUDA_ARCH_PTX="" \
        -D WITH_CUDNN=ON \
        -D WITH_CUBLAS=ON \
        -D ENABLE_FAST_MATH=ON \
        -D CUDA_FAST_MATH=ON \
        -D OPENCV_DNN_CUDA=ON \
        -D ENABLE_NEON=ON \
        -D WITH_QT=OFF \
        -D WITH_OPENMP=ON \
        -D BUILD_TIFF=ON \
        -D WITH_FFMPEG=ON \
        -D WITH_GSTREAMER=ON \
        -D WITH_TBB=ON \
        -D BUILD_TBB=ON \
        -D BUILD_TESTS=OFF \
        -D WITH_EIGEN=ON \
        -D WITH_V4L=ON \
        -D WITH_LIBV4L=ON \
        -D WITH_PROTOBUF=ON \
        -D OPENCV_ENABLE_NONFREE=ON \
        -D INSTALL_C_EXAMPLES=OFF \
        -D INSTALL_PYTHON_EXAMPLES=OFF \
        -D OPENCV_PYTHON3_INSTALL_PATH=/opt/opencv-venv/lib/python3.8/site-packages \
        -D PYTHON_EXECUTABLE=/opt/opencv-venv/bin/python3 \
        -D OPENCV_GENERATE_PKGCONFIG=ON \
        -D BUILD_EXAMPLES=OFF ../opencv && \
    make -j32 && \
    make install && \
    ldconfig

RUN apt-get install -y libcanberra-gtk-module libcanberra-gtk3-module
