## FastBot Robot Drivers

This repository contains the required ROS 2 packages to launch the FastBot robot drivers.

Included packages:

- fastbot_bringup

- fastbot_description

- serial_motor

- serial_motor_msgs

- lslidar_driver

- lslidar_msgs

- v4l2_camera

# Tested environment

This repository was tested on:

- Ubuntu Server 22.04 (64-bit)

- ROS 2 Humble

The instructions below assume that Ubuntu Server 22.04 and ROS 2 Humble are already installed on the Raspberry Pi.

## ROS 2 prerequisite installation

If ROS 2 Humble is not installed yet, install it first.

# Update system packages
sudo apt update && sudo apt upgrade -y
# Set locale
locale

sudo apt update
sudo apt install -y locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

locale
# Enable required repositories
sudo apt install -y software-properties-common
sudo add-apt-repository universe
# Add ROS 2 GPG key
sudo apt update
sudo apt install -y curl
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
# Add ROS 2 repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
# Install development tools
sudo apt update
sudo apt install -y ros-dev-tools git-all
# Install ROS 2 Humble
sudo apt update
sudo apt upgrade -y
sudo apt install -y \
  ros-humble-ros-base \
  ros-humble-demo-nodes-cpp \
  ros-humble-teleop-twist-keyboard \
  ros-humble-rmw-cyclonedds-cpp \
  ros-humble-joint-state-publisher
# Source ROS 2 Humble in .bashrc
nano ~/.bashrc

Add this line at the bottom of the file:

source /opt/ros/humble/setup.bash

Then source the file:

source ~/.bashrc
## Download repository

Clone the repository inside the ROS 2 workspace source folder:

cd /home/user/ros2_ws/src
git clone https://github.com/aszpetmanski/fastbot.git
# Install required system and ROS 2 dependencies

On a fresh Ubuntu / ROS 2 Humble installation, the following packages were required:

sudo apt update
sudo apt install -y \
  ros-humble-xacro \
  ros-humble-camera-info-manager \
  ros-humble-cv-bridge \
  ros-humble-image-transport \
  ros-humble-image-transport-plugins \
  ros-humble-diagnostic-updater \
  ros-humble-pcl-conversions \
  libboost-all-dev \
  libpcl-dev \
  libpcap-dev \
  v4l-utils

Then install any remaining dependencies with rosdep:

cd /home/user/ros2_ws
rosdep install --from-paths src --ignore-src -r -y
# Build workspace
cd /home/user/ros2_ws
rm -rf build install log
colcon build
source install/setup.bash
## Device aliases required by this project

This project expects the following device aliases:

/dev/arduino_nano

/dev/lslidar

Do not assume that your robot will use the same /dev/ttyUSB* or /dev/ttyACM* numbers as mine.
Those values may differ between robots and between boots.

Before launching the robot, identify your devices first.

# Check connected serial devices
ls -l /dev/serial/by-id
ls -l /dev/ttyUSB*
ls -l /dev/ttyACM*

If needed, inspect device attributes:

udevadm info -a -n /dev/ttyUSB0 | grep '{idVendor}\|{idProduct}\|{serial}' | head -n 20
udevadm info -a -n /dev/ttyACM0 | grep '{idVendor}\|{idProduct}\|{serial}' | head -n 20
Create udev aliases

# Create a udev rules file:

sudo nano /etc/udev/rules.d/99-fastbot.rules

Add rules that map your motor controller to /dev/arduino_nano and your LiDAR to /dev/lslidar.

Example from my FastBot:

SUBSYSTEM=="tty", KERNEL=="ttyUSB*", ATTRS{idVendor}=="1a86", ATTRS{idProduct}=="7523", SYMLINK+="arduino_nano"
SUBSYSTEM=="tty", ATTRS{idVendor}=="1a86", ATTRS{idProduct}=="55d4", ATTRS{serial}=="5A6D012303", SYMLINK+="lslidar"

On my robot, this resulted in:

/dev/arduino_nano -> /dev/ttyUSB0

/dev/lslidar -> /dev/ttyACM0

Your mapping may be different, so verify your own devices before creating the rules.

# Reload rules
sudo udevadm control --reload-rules
sudo udevadm trigger

If needed, reconnect the USB devices or reboot:

sudo reboot
Verify aliases
ls -l /dev/arduino_nano
ls -l /dev/lslidar
## Camera setup for Raspberry Pi

The FastBot camera uses the v4l2_camera ROS 2 driver.

For the Raspberry Pi camera to work correctly with v4l2_camera, the Raspberry Pi must use the legacy bm2835 mmal camera stack instead of libcamera / unicam.

# Edit Raspberry Pi camera configuration
sudo nano /boot/firmware/config.txt

Find this line and set it to:

camera_auto_detect=0

Then add this at the very end of the file under the final [all] section:

start_x=1

The end of the file should look similar to this:

[all]
start_x=1
# Reboot the Raspberry Pi
sudo reboot
Verify the camera driver after reboot
v4l2-ctl -D

The output must show:

Driver name : bm2835 mmal

If it still shows:

Driver name : unicam

then the wrong camera stack is still active and v4l2_camera will fail to stream correctly.

## Launch robot drivers
cd /home/user/ros2_ws
source /opt/ros/humble/setup.bash
source install/setup.bash
ros2 launch fastbot_bringup bringup.launch.xml
## Launch file contents

The bringup.launch.xml file starts:

motor drivers

robot state publisher

LiDAR driver

camera driver

## Check topics
# LiDAR
ros2 topic list | grep scan
# Camera
ros2 topic list | grep image

Expected camera topics include:

/image_raw
/image_raw/compressed
# Motor commands

To command the robot to move forward:
'''
ros2 topic pub /fastbot/cmd_vel geometry_msgs/msg/Twist "{linear: {x: 0.1}, angular: {z: 0.0}}
'''


ros2 topic pub /fastbot/cmd_vel geometry_msgs/msg/Twist "{linear: {x: 0.1}, angular: {z: 0.0}}"



