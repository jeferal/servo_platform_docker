# This image starts from the base image of nvidia ubuntu cuda
# it will install Opencv 4.5 from source, downloading from github
# and then compiling with cuda suport

FROM nvidia/cuda:11.4.3-cudnn8-devel-ubuntu20.04

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

# Download opencv and opencv_contrib 4.8.0
RUN cd /opt && \
    wget -O opencv.zip https://github.com/opencv/opencv/archive/refs/tags/4.8.0.zip && \
    wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/refs/tags/4.8.0.zip && \
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
    /opt/opencv-venv/bin/pip3 install numpy pycuda

# Compile and install opencv
RUN cd /opt && \
    mkdir build && \
    cd build && \
    cmake -D CMAKE_BUILD_TYPE=RELEASE \
        -D CMAKE_INSTALL_PREFIX=/usr/local \
        -D WITH_TBB=ON \
        -D ENABLE_FAST_MATH=1 \
        -D CUDA_FAST_MATH=1 \
        -D WITH_CUBLAS=1 \
        -D WITH_CUDA=ON \
        -D BUILD_opencv_cudacodec=OFF \
        -D WITH_CUDNN=ON \
        -D OPENCV_DNN_CUDA=ON \
        -D CUDA_ARCH_BIN=8.6 \
        -D WITH_V4L=ON \
        -D WITH_QT=OFF \
        -D WITH_OPENGL=ON \
        -D WITH_GSTREAMER=ON \
        -D OPENCV_GENERATE_PKGCONFIG=ON \
        -D OPENCV_PC_FILE_NAME=opencv.pc \
        -D OPENCV_ENABLE_NONFREE=ON \
        -D INSTALL_PYTHON_EXAMPLES=ON \
        -D OPENCV_PYTHON3_INSTALL_PATH=/opt/opencv-venv/lib/python3.8/site-packages \
        -D PYTHON_EXECUTABLE=/opt/opencv-venv/bin/python3 \
        -D INSTALL_C_EXAMPLES=ON \
        -D OPENCV_EXTRA_MODULES_PATH=/opt/opencv_contrib/modules \
        -D BUILD_EXAMPLES=ON ../opencv && \
    make -j32 && \
    make install && \
    ldconfig

RUN apt-get install -y libcanberra-gtk-module libcanberra-gtk3-module

# Install opencv zoo
RUN cd /opt && \
    git clone https://github.com/opencv/opencv_zoo.git && \
    cd opencv_zoo && \
    apt -y install git-lfs && \
    git lfs install && \
    git lfs pull

RUN apt install -y nano

RUN apt-get install -y lsb-core

# Install ROS noetic
RUN sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'

RUN apt-get install curl -y && \
    curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | apt-key add -

RUN apt-get update && \
    apt-get install ros-noetic-desktop-full -y

SHELL ["/bin/bash", "-c"]
# Source ROS
RUN echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc

# Create a catkin workspace
RUN source /opt/ros/noetic/setup.bash && \
    mkdir -p ~/base_ws/src && \
    cd ~/base_ws/ && \
    git clone https://github.com/jeferal/servo_platform.git src && \
    catkin_make && \
    # Add source to bashrc
    echo "source ~/base_ws/devel/setup.bash" >> ~/.bashrc

# Install terminator
RUN apt-get install -y terminator

# Install librealsense
RUN apt-key adv --keyserver keyserver.ubuntu.com  --recv-key F6E65AC044F831AC80A06380C8B3A55A6F3EFCDE || sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-key && \
    apt-get install software-properties-common -y && \
    add-apt-repository "deb https://librealsense.intel.com/Debian/apt-repo $(lsb_release -cs) main" -u && \
    apt-get install apt-utils -y && \
    apt-get install librealsense2-utils librealsense2-dev -y

# Install ROS wrapper
RUN source /opt/ros/noetic/setup.bash && \
    mkdir -p ~/base_deps_ws/src && \
    apt-get install ros-noetic-ddynamic-reconfigure -y && \
    cd ~/base_deps_ws/src && \
    git clone https://github.com/IntelRealSense/realsense-ros.git && \
    cd ~/base_deps_ws/src/realsense-ros/ && \
    git checkout 2.3.2 && \
    cd ~/base_deps_ws/ && \
    catkin_make && \
    # Add source to bashrc
    echo "source ~/base_deps_ws/devel/setup.bash" >> ~/.bashrc
