# Pixhawk

In this post, I'm running MAVROS in the TX2 connected to the Pixhawk 4 with PX4 and QGroundControl. First of all, we need to install ROS Melodic from http://wiki.ros.org/melodic/Installation/Ubuntu

## Installation:

### 1. Configure your Ubuntu repositories
Configure your Ubuntu repositories to allow "restricted," "universe," and "multiverse." You can follow the Ubuntu guide for instructions on doing this.

### 2. Setup your sources.list
Setup your computer to accept software from packages.ros.org.

sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'

### 3. Set up your keys

sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654

### 4. Installation
First, make sure your Debian package index is up-to-date:

sudo apt update

sudo apt install ros-melodic-desktop-full

### 5. Initialize rosdep
Before you can use ROS, you will need to initialize rosdep. rosdep enables you to easily install system dependencies for source you want to compile and is required to run some core components in ROS.

sudo rosdep init
rosdep update

### 6. Environment setup
It's convenient if the ROS environment variables are automatically added to your bash session every time a new shell is launched:

echo "source /opt/ros/melodic/setup.bash" >> ~/.bashrc
source ~/.bashrc

### 7. Dependencies for building packages
Up to now you have installed what you need to run the core ROS packages. To create and manage your own ROS workspaces, there are various tools and requirements that are distributed separately. For example, rosinstall is a frequently used command-line tool that enables you to easily download many source trees for ROS packages with one command.

To install this tool and other dependencies for building ROS packages, run:


sudo apt install python-rosinstall python-rosinstall-generator python-wstool build-essential
