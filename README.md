# jetsonNanoPatches

## Table of contents
* [General info](#general-info)
* [Inspiration](#inspiration)
* [Features](#features)
* [Technologies](#technologies)
* [Hardware](#hardware)
* [Software](#software)
* [Preparation](#prepare)
* [Setup](#setup)
* [Patch](#patch)
* [Status](#status)
* [Contact](#contact)

## General info
This project contains a patch for [Open3D](https://github.com/intel-isl/Open3D) to use it with cuda and librealsense on a Jetson Nano.
After you applied the patch you can use Open3D with Python support.
The visualizer has to be compiled separately (C++-version)


You can use your Intel Realsense Camera for mobile 3D reconstruction on the Jetson device.
To get more information about the used hardware or additional components you want to use, visit [https://elinux.org/Jetson_Nano](https://elinux.org/Jetson_Nano) 

## Inspiration
Project is inspired by
[https://github.com/devshank3/JetScan](https://github.com/devshank3/JetScan)

Motivation for this project: Make use of Open3D library with the Intel Realsense Camera D435i on a Nvidia Jetson Nano.
The last update of  devshank3's repository was on 5 Mar 2020. So it doesn't compile well on an updated Jetson Nano.

Why a patch? Why not a fork?
This patch is the easiest way to get already existing versions of Open3d on a Jetson Nano to work. 
This may be a one-time experiment, as the Jetson Nano is not the fastest hardware to experiment with. 

Project is based on
[https://github.com/theNded/Open3D](https://github.com/theNded/Open3D) and [https://github.com/intel-isl/Open3D](https://github.com/intel-isl/Open3D)

## Features

### Patch to get the following features enabled on NVidia Jetson Nano Single Board Computer:

* 3D reconstruction on a Jetson Nano with Open3D (e.g. generating 3D scenes of the real world from Pointcloud Maps)
* (Unofficial) Open3d with cuda support
* Intel's librealSense support
* OpenCV version: 4.4.0 with Cuda support

## Technologies

3D reconstruction on a (mobile) Single Board Computer.

Project contains:
* patch to compile Open3D for the NVidia Jetson Nano sbc with CUDA 10.2 and librealsense support.

The machine intended for this patch is a

```
NVIDIA Jetson Nano (Developer Kit Version)
 L4T 32.4.3 [ JetPack 4.4 ]
   Ubuntu 18.04.5 LTS
   Kernel Version: 4.9.140-tegra
 CUDA 10.2.89
   CUDA Architecture: 5.3
 OpenCV version: 4.4.0
   OpenCV Cuda: YES
 CUDNN: 8.0.0.180
 TensorRT: 7.1.3.0
 Vision Works: 1.6.0.501
 VPI: 0.3.7
```

##  Hardware
* Nvidia Jetson Nano [https://www.nvidia.com/de-de/autonomous-machines/embedded-systems/jetson-nano/](https://www.nvidia.com/de-de/autonomous-machines/embedded-systems/jetson-nano/)
* 5V cooling PWM fan, e.g. Noctua nf-a4x20 5V PWM. [https://noctua.at/en/nf-a4x20-5v-pwm](https://noctua.at/en/nf-a4x20-5v-pwm)
* Intel Realsense Camera - the D435i [https://www.intelrealsense.com/depth-camera-d435i/](https://www.intelrealsense.com/depth-camera-d435i/) is the only hardware tested with this patch , but other cameras may work, too.
* optional: Battery supply for outdoor use. Try one of those (untested) [https://elinux.org/Jetson_Nano#Battery_Packs](https://elinux.org/Jetson_Nano#Battery_Packs)

## Software

Software provided here: a single patch

Software provided by others

* Nivida Jetson Nano sd-card image [https://developer.nvidia.com/jetson-nano-sd-card-image](NVidia)
* Open3D             [http://www.open3d.org/docs/release/index.html](Open3D)
* librealSense       [https://github.com/IntelRealSense/librealsense](Intel)
* OpenCV             [install script by mdegans](https://github.com/mdegans/nano_build_opencv.git)
* jetsonUtilities    [install script by JetsonHacks](https://github.com/jetsonhacks/jetsonUtilities) 
* resizeSwapMemory   [install script by JetsonHacks](https://github.com/JetsonHacksNano/resizeSwapMemory)
* installSwapfile    [install script by JetsonHacks](https://github.com/JetsonHacksNano/installSwapfile)
* cmake              [https://cmake.org/](https://cmake.org/)
* Eigen3             [http://eigen.tuxfamily.org/](http://eigen.tuxfamily.org/)


## Prepare

### Download an SD-card image from the NVidia Website and install it properly.

[https://developer.nvidia.com/jetson-nano-sd-card-image](https://developer.nvidia.com/jetson-nano-sd-card-image)


### Execute the following steps on the Nano:

```
$ sudo ln -s /usr/local/cuda-10.2 /usr/local/cuda
$ export CPATH=/usr/local/cuda-10.2/targets/aarch64-linux/include:$CPATH
$ export LD_LIBRARY_PATH=/usr/local/cuda-10.2/targets/aarch64-linux/lib:$LD_LIBRARY_PATH
$ export PATH=/usr/local/cuda-10.2/bin:$PATH
```
to make sure that cuda and its headers are found at any time.

## Setup

The following instructions are executed on the Jetson Nano 

Most of the software that is used to successfully compile Open3D on the Jetson from source.

These steps are needed as the version of the software installed by default with apt is 
* outdated 
or
* need to be patched
or
* are not available as binaries for this architecture (aarch64)

A detailed how-to for compiling Open3D from source can be found [here](http://www.open3d.org/docs/release/compilation.html) 

That is:
* cmake
* Eigen3 (libeigen-dev)
* OpenCV 4.4.0
* librealSense


# Optional Hardware: Add a fan

### Do it. Cores will get hot during compilation. You might burn your little friend to death if you don't cool it!!!
```
$ mkdir git_src && cd git_src/
$ git clone https://github.com/Pyrestone/jetson-fan-ctl.git
$ cd jetson-fan-ctl/
$ sudo ./install.sh
$ sudo service automagic-fan status
$ sudo nvpmodel -q
# should be set to 10w
$ cd ~
```
# Software setup  (mostly done with scripts)

Increase the swap memory
```
$ cd git_src/
$ mkdir JetsonHacks  && cd JetsonHacks
$ git clone https://github.com/JetsonHacksNano/resizeSwapMemory.git
$ cd resizeSwapMemory/
$ ./setSwapMemorySize.sh -g 4
$ sudo reboot
$ free -h
$ cd ~
```
Update cmake (date now: 04.07.2020) to v3.18.2
```
$ cmake --version
$ wget https://gitlab.kitware.com/cmake/cmake/-/archive/v3.18.2/cmake-v3.18.2.tar.bz2
$ tar xvf cmake-v3.18.2.tar.bz2
$ sudo apt install libssl-dev
$ sudo apt install libcurl4-openssl-dev
$ sudo apt remove --purge cmake
$ sudo apt autoremove
$ cd cmake-v3.18.2/
$ ./boostrap
$ make -j3
$ sudo make install
$ which cmake
$ cmake --version
$ cd ~
```
Update Eigen (date now: 04.07.2020) to 3.3.7
```
$ wget https://gitlab.com/libeigen/eigen/-/archive/3.3.7/eigen-3.3.7.tar.bz2
$ tar xvf eigen-3.3.7.tar.bz2
$ sudo apt remove --purge libeigen3-dev
$ sudo apt autoremove
$ cd eigen-3.3.7/
$ mkdir build && cd build
$ cmake .. -DCMAKE_INSTALL_PREFIX=/usr/local
$ sudo make install
```
4a. Patch Eigen
```
$ sudo nano /usr/local/include/eigen3/Eigen/Core
```
in line 257 under
```
#if defined EIGEN_HAS_CUDA_FP16
```
change
```
#include <host_defines.h>
```
to
```
#include <cuda_runtime_api.h>
```
otherwise cmake will be angry with you later while compiling Open3D ;-)
```
$ cd ~	
```
Install a python virtual environment
```
$ sudo apt install python3-venv
$ cd ~
$ mkdir -p ~/.venv/jetscan
$ python3 -m venv ~/.venv/jetscan
$ source ~/.venv/jetscan/bin/activate
```
Update some libs in venv
```
$ pip3 install --upgrade pip
$ pip install numpy
$ pip install matplotlib
$ pip install cython
$ cd ~
```
Install OpenCV with cuda enabled in the activated python venv
```
$ cd git_src/
$ mkdir mdegans
$ cd mdegans
$ source ~/.venv/jetscan/bin/activate
$ git clone https://github.com/mdegans/nano_build_opencv.git
$ cd nano_build_opencv
```
edit build_opencv.sh with
```
$ nano build_opencv.sh
```
and remove all entries for installing the deb packages for cmake and libeigen as we installed them from source already and then run
```
$ ./build_opencv.sh
$ cd ~/.venv/jetscan/lib/python3.6/site-packages/
$ ln -s /usr/local/lib/python3.6/dist-packages/cv2/python-3.6/cv2.cpython-36m-aarch64-linux-gnu.so cv2.so
$ cd ~
```
    Install a python util to get detailed info about your jetson device
```
$ cd ~/git_src/JetsonHacks
$ git clone https://github.com/jetsonhacks/jetsonUtilities.git
$ cd jetsonUtilities
$ source ~/.venv/jetscan/bin/activate
$ cython2 jetsonInfo.py >*/jetsoninfo.txt
$ cd ~
```
### Open3D installation 

Install Open3D with Cuda support
```
mkdir -p git_src/Open3D && cd git_src/Open3D
git clone --recursive https://github.com/theNded/Open3D.git
```
check that you are on
```
commit 0ca8fd19d355d80e5998efec31858fe19791ccf5
Date:   Wed Sep 2 15:01:08 2020 -0400
Update README.md
```
in git history, otherwise
```
$ git checkout 0ca8fd19d355d80e5998efec31858fe19791ccf5
```
```
$ cd Open3D
$ mkdir build  cd build
$ source ~/.venv/jetscan/bin/activate
$ python --version
```
(should be python3)
```
$ util/scripts/install-deps-ubuntu.sh
$ sudo apt autoremove
$ sudo apt clean
$ cmake ..
$ make -j3
$ make install-pip-package
```
9a. Test if Open3D python works in the virtual environment
```
$ python -c "import open3d"
```
should give you no errors.
```
$ cd ~
```
### Intel Realsense Camera (D435i) installation

Install librealsense
```
$ cd ~/git_src/JetsonHacks
$ git clone https://github.com/JetsonHacksNano/installLibrealsense.git
$ cd installLibrealsense
```
edit buildLibrealsense.sh with
```
$ nano  buildLibrealsense.sh
```
and set in line 6

# Jetson Nano; L4T 32.4.3

and set in line 9
```
LIBREALSENSE_VERSION=v2.38.1
```
and set in line 11
```
NVCC_PATH=/usr/local/cuda-10.2/bin/nvcc
```
edit scripts/installDependencies.sh with
```
$ nano scripts/installDependencies.sh
```
and remove all entries for installing the deb packages for cmake and libeigen as we installed them from source already plug off your camera and then run
```
$ ./buildLibrealsense.sh
```
Here you should be able to use Open3d with Cuda support for your RealSense camera.

## Patch

To apply the patch, simply download the patch with

```
$ wget https://raw.githubusercontent.com/nanotuxi/jetsonNanoPatches/master/Open3DJetsonCudaRealSense.patch
```
into your Open3D root directory and apply it with

```
$ git apply Open3DJetsonCudaRealSense.patch
```

## Status
Project is: _in progress_

## Contact
Created by [@nanotuxi](https://blog.georgmill.de) - feel free to visit my blog!
