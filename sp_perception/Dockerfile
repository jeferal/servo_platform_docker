# This image starts from the base image of nvidia ubuntu cuda
# it will install Opencv 4.5 from source, downloading from github
# and then compiling with cuda suport

FROM nvidia/cuda:11.2.2-cudnn8-devel-ubuntu20.04

ENV DEBIAN_FRONTEND=noninteractive 

# Update the system
RUN apt-get update && apt-get upgrade -y

# Install generic tools
RUN apt-get install -y build-essential cmake pkg-config unzip yasm git checkinstall

# Install image and video I/O libraries
RUN apt-get install -y libjpeg-dev libpng-dev libtiff-dev

# Video/Audio Libs - FFMPEG, GSTREAMER, x264 and so on
RUN apt-get install -y libavcodec-dev libavformat-dev libswscale-dev libavresample-dev \
    libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev \
    libxvidcore-dev x264 libx264-dev libfaac-dev libmp3lame-dev libtheora-dev \
    libfaac-dev libmp3lame-dev libvorbis-dev

# Install OpenCore
RUN apt-get install -y libopencore-amrnb-dev libopencore-amrwb-dev

# Cameras programming interface libs
RUN apt-get install -y libdc1394-22 libdc1394-22-dev libxine2-dev libv4l-dev v4l-utils && \
    cd /usr/include/linux && \
    ln -s -f ../libv4l1-videodev.h videodev.h

# GTK lib for the graphical user functionalites coming from OpenCV highghui module
RUN apt-get install -y libgtk-3-dev

# Python
RUN apt-get install -y python3-dev python3-pip && \
    pip3 install -U pip numpy && \
    apt-get -y install python3-testresources

# Parallelism library C++ for CPU
RUN apt-get install -y libtbb-dev

# Optimization libraries for OpenCV
RUN apt-get install -y libatlas-base-dev gfortran

# Optional libraries
RUN apt-get install -y libprotobuf-dev protobuf-compiler \
    libgoogle-glog-dev libgflags-dev \
    libgphoto2-dev libeigen3-dev libhdf5-dev doxygen

# Install wget
RUN apt-get install -y wget

# Download opencv and opencv_contrib 4.5.2
RUN cd /opt && \
    wget -O opencv.zip https://github.com/opencv/opencv/archive/refs/tags/4.5.2.zip && \
    wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/refs/tags/4.5.2.zip && \
    unzip opencv.zip && \
    unzip opencv_contrib.zip && \
    mv opencv-4.5.2 opencv && \
    mv opencv_contrib-4.5.2 opencv_contrib && \
    rm opencv.zip && \
    rm opencv_contrib.zip

# Compile and install opencv
RUN cd /opt/opencv && \
    mkdir build && \
    cd build && \
    cmake -D CMAKE_BUILD_TYPE=RELEASE \
        -D CMAKE_INSTALL_PREFIX=/usr/local \
        -D WITH_CUDA=ON \
        -D CUDA_ARCH_BIN=7.5 \
        -D CUDA_ARCH_PTX="" \
        -D WITH_CUDNN=ON \
        -D OPENCV_DNN_CUDA=ON \
        -D ENABLE_FAST_MATH=ON \
        -D CUDA_FAST_MATH=ON \
        -D WITH_CUBLAS=ON \
        -D WITH_OPENGL=ON \
        -D WITH_GSTREAMER=ON \
        -D OPENCV_GENERATE_PKGCONFIG=ON \
        -D OPENCV_PC_FILE_NAME=opencv.pc \
        -D INSTALL_PYTHON_EXAMPLES=ON \
        -D INSTALL_C_EXAMPLES=ON \
        -D OPENCV_EXTRA_MODULES_PATH=/opt/opencv_contrib/modules \
        -D BUILD_EXAMPLES=ON .. && \
    make -j8 && \
    make install && \
    ldconfig
