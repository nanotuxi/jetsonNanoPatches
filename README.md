# jetsonNanoPatches
Patches to use Open3D on the Jetson Nano with cuda10.2  and librealsense

download the patch with

```
$ wget https://raw.githubusercontent.com/nanotuxi/jetsonNanoPatches/master/Open3DJetsonCudaRealSense.patch
```
into your Open3D root directory and apply it with

```
$ git apply Open3DJetsonCudaRealSense.patch
```
To work with an Intel Realsense Camera (here: D435i) on a
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
Install Opend3D with Cuda and libRealsense support:

Pre:
in a shell enter
```
$ sudo ln -s /usr/local/cuda-10.2 /usr/local/cuda    
$ export CPATH=/usr/local/cuda-10.2/targets/aarch64-linux/include:$CPATH
$ export LD_LIBRARY_PATH=/usr/local/cuda-10.2/targets/aarch64-linux/lib:$LD_LIBRARY_PATH
$ export PATH=/usr/local/cuda-10.2/bin:$PATH
```
	
to make sure that cuda and its headers are found at any time.

1. Install a fan for the jetson device
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
2. Increase the swap memory
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
3. Update cmake (date now: 04.07.2020) to v3.18.2
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
4. Update Eigen (date now: 04.07.2020) to 3.3.7
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
5. Install a python virtual environment
```
$ sudo apt install python3-venv
$ cd ~
$ mkdir -p ~/.venv/jetscan
$ python3 -m venv ~/.venv/jetscan
$ source ~/.venv/jetscan/bin/activate
 ```
6. Update some libs in venv
```
$ pip3 install --upgrade pip
$ pip install numpy
$ pip install matplotlib
$ pip install cython
$ cd ~
```
7. Install OpenCV with cuda enabled in the activated python venv

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
  and remove all entries for installing the deb packages for cmake and libeigen
  as we installed them from source already
  and then run
```
$ ./build_opencv.sh
$ cd ~/.venv/jetscan/lib/python3.6/site-packages/
$ ln -s /usr/local/lib/python3.6/dist-packages/cv2/python-3.6/cv2.cpython-36m-aarch64-linux-gnu.so cv2.so
$ cd ~
```
8. Install a python util to get detailed info about your jetson device
```
$ cd ~/git_src/JetsonHacks
$ git clone https://github.com/jetsonhacks/jetsonUtilities.git
$ cd jetsonUtilities
$ source ~/.venv/jetscan/bin/activate
$ cython2 jetsonInfo.py >*/jetsoninfo.txt
$ cd ~
```
9. Install Open3D with Cuda support
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
10. Install librealsense
```
$ cd ~/git_src/JetsonHacks
$ git clone https://github.com/JetsonHacksNano/installLibrealsense.git
$ cd installLibrealsense
```
edit buildLibrealsense.sh  with 
```
$ nano  buildLibrealsense.sh
```
and set in line 6
```
# Jetson Nano; L4T 32.4.3
```
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
and remove all entries for installing the deb packages for cmake and libeigen
 as we installed them from source already
 plug off your camera
 and then run	
 ```
 $ ./buildLibrealsense.sh
 ```
Here you should be able to use Open3d with Cuda support for your RealSense camera.
