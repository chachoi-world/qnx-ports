**NOTE**: QNX ports are only supported from a Linux host operating system

# Compile the port for QNX in a Docker container

Currently the port is supported for QNX SDP 7.1 and 8.0.

We recommend that you use Docker to build ros2 for QNX to ensure the build environment consistency.

```bash
# Set QNX_SDP_VERSION to be qnx800 for SDP 8.0 or qnx710 for SDP 7.1
export QNX_SDP_VERSION=qnx800

# Create a workspace
mkdir -p ~/qnx_workspace && cd ~/qnx_workspace
git clone https://gitlab.com/qnx/everywhere/qnx-ports.git && cd qnx-ports

# Build the Docker image and create a container
./docker-build-qnx-image.sh
./docker-create-container.sh

# Once you're in the image, set up environment variables
source ~/qnx800/qnxsdp-env.sh
source /usr/local/qnx/env/bin/activate

# Build googletest
PREFIX="/usr" QNX_PROJECT_ROOT="$(pwd)/googletest" make -C qnx-ports/googletest install -j$(nproc)

# Import ros2 packages
cd ~/qnx_workspace/qnx-ports/ros2
mkdir -p src
vcs import src < ros2.repos

# Run required scripts
./scripts/colcon-ignore.sh
./scripts/patch.sh

# Build ros2
./scripts/build-ros2.sh
```

After the build completes, ros2_humble.tar.gz will be created at QNX_TARGET/CPUVARDIR/ros2_humble.tar.gz

## Build on host without using Docker

Don't forget to source qnxsdp-env.sh in your SDP.

```bash
# Set QNX_SDP_VERSION to be qnx800 for SDP 8.0 or qnx710 for SDP 7.1
export QNX_SDP_VERSION=qnx800

# Create a workspace
mkdir -p ~/qnx_workspace && cd ~/qnx_workspace

# Clone repos
git clone https://gitlab.com/qnx/everywhere/qnx-ports.git
git clone -b qnx_v1.13.0 https://gitlab.com/qnx/libs/googletest.git

# Install python 3.11
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
sudo apt-get install -y python3.11-dev python3.11-venv python3.11-distutils software-properties-common rename

# Create a python 3.11 virtual environment
python3.11 -m venv env
source env/bin/activate

# Install required python packages
pip install -U \
  pip \
  empy \
  lark \
  Cython \
  wheel \
  colcon-common-extensions \
  vcstool \
  catkin_pkg \
  argcomplete \
  flake8-blind-except \
  flake8-builtins \
  flake8-class-newline \
  flake8-comprehensions \
  flake8-deprecated \
  flake8-docstrings \
  flake8-import-order \
  flake8-quotes \
  pytest-repeat \
  pytest-rerunfailures \
  pytest

# source qnxsdp-env.sh
source ~/qnx800/qnxsdp-env.sh

# Build and install googletest
PREFIX="/usr" QNX_PROJECT_ROOT="$(pwd)/googletest" make -C qnx-ports/googletest install -j$(nproc)

# Import ros2 packages
cd ~/qnx_workspace/qnx-ports/ros2
mkdir -p src
vcs import src < ros2.repos

# Run scripts to ignore some packages and apply QNX patches
./scripts/colcon-ignore.sh
./scripts/patch.sh

# Set LD_PRELOAD to the host libzstd.so for x86_64 SDP 7.1 builds
export LD_PRELOAD=$LD_PRELOAD:/usr/lib/x86_64-linux-gnu/libzstd.so

# Build ros2
./scripts/build-ros2.sh
```

# How to run tests

Use scp to move ros2_humble.tar.gz to the target

```bash
scp ros2_humble.tar.gz root@<target-ip-address>:/
```

```bash
ssh root@<target-ip-address>
cd /
tar -xvzf ./ros2_humble.tar.gz
```

### Prepare Target

```bash
# Update system time
ntpdate -sb 0.pool.ntp.org 1.pool.ntp.org

# Install pip and packaging
mkdir -p /data
export TMPDIR=/data
python3 -m ensurepip
pip3 install packaging pyyaml lark
export PYTHONPATH=$PYTHONPATH:/opt/ros/humble/usr/lib/python3.11/site-packages/
```

### Running the Listner Talker Demo on RPI4

Run listener in a terminal:

```bash
cd /opt/ros/humble
. /opt/ros/humble/setup.bash
python3 ./bin/ros2 run demo_nodes_cpp listener
```

Run talker in another terminal:

```bash
cd /opt/ros/humble
. /opt/ros/humble/setup.bash
python3 ./bin/ros2 run demo_nodes_cpp talker
```

### Running the dummy robot demo on RPI4

Launch the dummy robot demo node on RPI4.
```bash
cd /opt/ros/humble
. /opt/ros/humble/setup.bash
python3 ./bin/ros2 dummy_robot_bringup dummy_robot_bringup.launch.py
```

Install ROS2 Humble on your Ubuntu host.

There is no QNX port for rviz2.

Follow the instructions at https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debians.html.

Start rviz2
```bash
source <ROS2_INSTALL_FOLDER>/setup.bash
rviz2
```

Please refer to https://docs.ros.org/en/humble/Tutorials/Demos/dummy-robot-demo.html for more details about the dummy robot demo.
