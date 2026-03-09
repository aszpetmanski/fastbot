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

# Download repository

Clone the repository inside the ROS 2 workspace source folder:

cd /home/user/ros2_ws/src
git clone https://github.com/aszpetmanski/fastbot.git
# Install remaining ROS 2 dependencies
cd /home/user/ros2_ws
rosdep install --from-paths src --ignore-src -r -y
# Build workspace
cd /home/user/ros2_ws
rm -rf build install log
colcon build
source install/setup.bash
# Launch robot drivers
ros2 launch fastbot_bringup bringup.launch.xml
# Launch file contents

The bringup.launch.xml file starts:

motor drivers

robot state publisher

LiDAR driver

camera driver

## IMPORTANT --> CAMERA NOTES <-- IMPORTANT

The FastBot camera uses the v4l2_camera ROS 2 driver included in this repository.

The launch file includes:

<node pkg="v4l2_camera" exec="v4l2_camera_node" name="v4l2_camera_node" namespace="$(var robot_name)" output="screen">
    <param name="image_size" value="[320, 240]"/>
    <param name="camera_frame_id" value="$(var robot_name)_camera"/>
</node>

For the Raspberry Pi camera to work correctly with v4l2_camera,
the Raspberry Pi must use the legacy bm2835 mmal camera stack instead of libcamera.

# Edit
/boot/firmware/config.txt

Set:

camera_auto_detect=0

And add at the end of the file under the final [all] section:

start_x=1
# Then reboot
sudo reboot
# After reboot, verify the camera driver
v4l2-ctl -D

The output should show:

Driver name : bm2835 mmal

