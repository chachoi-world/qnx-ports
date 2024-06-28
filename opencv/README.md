Verified for SDP 8.0

**NOTE**: QNX ports are only supported from a Linux host operating system

# Compile the port for QNX in a Docker container

Pre-requisite: Install Docker on Ubuntu https://docs.docker.com/engine/install/ubuntu/
```bash
# Create a workspace
mkdir -p ~/qnx_workspace && cd ~/qnx_workspace
git clone https://gitlab.com/qnx/everywhere/qnx-ports.git && cd qnx-ports

# Build the Docker image and create a container
./docker-build-qnx-image.sh
./docker-create-container.sh

# Now you are in the Docker container

# source qnxsdp-env.sh in
source ~/qnx800/qnxsdp-env.sh

# source python3.11 venv script
source /usr/local/qnx/env/bin/activate

# Clone numpy
cd ~/qnx_workspace
git clone https://gitlab.com/qnx/libs/opencv.git
git clone https://gitlab.com/qnx/libs/numpy.git && cd numpy
git submodule update --init --recursive
cd ~/qnx_workspace

# Build numpy first
QNX_PROJECT_ROOT="$(pwd)/numpy" make -C qnx-ports/numpy install -j$(nproc)

# Build opencv
BUILD_TESTING="ON" QNX_PROJECT_ROOT="$(pwd)/opencv" make -C qnx-ports/opencv install -j$(nproc)
```

# Compile the port for QNX
```bash
# Clone the repos
git clone https://gitlab.com/qnx/everywhere/qnx-ports.git
# opencv depends on numpy
git clone https://gitlab.com/qnx/libs/numpy.git
git clone https://gitlab.com/qnx/libs/opencv.git

# Build numpy
cd numpy
git submodule update --init --recursive
cd -

# Install python3.11 and gfortran
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt-get install -y python3.11-dev python3.11-venv python3.11-distutils software-properties-common gfortran

# Create a python virtual environment and install necessary packages
python3.11 -m venv env
source env/bin/activate
pip install -U pip Cython wheel

# source qnxsdp-env.sh
source ~/qnx800/qnxsdp-env.sh

# Build numpy first
PREFIX="/usr" QNX_PROJECT_ROOT="$(pwd)/numpy" make -C qnx-ports/numpy install -j$(nproc)

# Build opencv
BUILD_TESTING="ON" QNX_PROJECT_ROOT="$(pwd)/opencv" make -C qnx-ports/opencv install -j$(nproc)
```

# How to run tests

Set up the test environment
```bash
# On your development machine, clone opencv_extra
git clone git@github.com:opencv/opencv_extra.git && cd opencv_extra
git checkout 4.9.0

# scp opencv_extra's testdata to / on your target
scp -r testdata root@<target-ip-address>:/

# scp opencv libraries
scp $QNX_TARGET/aarch64le/usr/local/lib/libopencv* root@<target-ip-address>:/usr/lib

# scp opencv tests
scp -r $QNX_TARGET/aarch64le/usr/local/bin/opencv_tests root@<target-ip-address>:/usr/bin
```

Run tests on the target.
```bash
# ssh into the target
ssh root@<target-ip-address>

# Run tests
cd /usr/bin/opencv_tests
chmod +x

export OPENCV_TEST_DATA_PATH=/testdata

./opencv_perf_calib3d
./opencv_perf_core
./opencv_perf_dnn
./opencv_perf_features2d
./opencv_perf_imgcodecs
./opencv_perf_imgproc
./opencv_perf_objdetect
./opencv_perf_photo
./opencv_perf_stitching
./opencv_perf_video
./opencv_perf_videoio
./opencv_test_calib3d
./opencv_test_core
./opencv_test_dnn
./opencv_test_features2d
./opencv_test_flann
./opencv_test_highgui
./opencv_test_imgcodecs
./opencv_test_imgproc
./opencv_test_ml
./opencv_test_objdetect
./opencv_test_photo
./opencv_test_stitching
./opencv_test_video
./opencv_test_videoio
```
