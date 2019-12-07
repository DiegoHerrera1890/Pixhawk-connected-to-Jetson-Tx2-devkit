# Running MAVROS with Jetson TX2 and PixHawk 4

In this post, I'm running MAVROS in the TX2 connected to the Pixhawk 4 with PX4 and QGroundControl. First of all, we need to install ROS Melodic from [the ROS official website](http://wiki.ros.org/melodic/Installation/Ubuntu "ROS-Melodic for Ubuntu 18.04") 

## 1 ROS-Installation:

### 1.1 Configure your Ubuntu repositories
Configure your Ubuntu repositories to allow "restricted," "universe," and "multiverse." You can follow the Ubuntu guide for instructions on doing this.

### 1.2 Setup your sources.list
Setup your computer to accept software from packages.ros.org.

`sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'`

### 1.3 Set up your keys

`sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654`

### 1.4 Installation
First, make sure your Debian package index is up-to-date:

`sudo apt update 
 sudo apt install ros-melodic-desktop-full`

### 1.5 Initialize rosdep
Before you can use ROS, you will need to initialize rosdep. rosdep enables you to easily install system dependencies for source you want to compile and is required to run some core components in ROS.

`sudo rosdep init
 rosdep update`

### 1.6 Environment setup
It's convenient if the ROS environment variables are automatically added to your bash session every time a new shell is launched:

`echo "source /opt/ros/melodic/setup.bash" >> ~/.bashrc
 source ~/.bashrc`

### 1.7 Dependencies for building packages
Up to now you have installed what you need to run the core ROS packages. To create and manage your own ROS workspaces, there are various tools and requirements that are distributed separately. For example, rosinstall is a frequently used command-line tool that enables you to easily download many source trees for ROS packages with one command.

To install this tool and other dependencies for building ROS packages, run:

`sudo apt install python-rosinstall python-rosinstall-generator python-wstool build-essential`

## 2 MAVROS-Installation:
After installing ROS-melodic we need to create our catkin workspace and install MAVROS and then build everything.

`jetson@desktop:~$ sudo apt-get install python-catkin-tools python-rosinstall-generator -y`

Create the workspace: unneeded if you already has workspace:

`jetson@desktop:~$ mkdir -p ~/catkin_ws/src
jetson@desktop:~$ cd ~/catkin_ws
jetson@desktop:~$ catkin init
jetson@desktop:~$ wstool init src`

Install MAVLink
we use the Kinetic reference for all ROS distros as it's not distro-specific and up to date:

`jetson@desktop:~$ rosinstall_generator --rosdistro kinetic mavlink | tee /tmp/mavros.rosinstall`

Install MAVROS: get source (upstream - released):

`jetson@desktop:~$ rosinstall_generator --upstream mavros | tee -a /tmp/mavros.rosinstall`

Create workspace & deps:

`jetson@desktop:~$ wstool merge -t src /tmp/mavros.rosinstall
jetson@desktop:~$ wstool update -t src -j4
jetson@desktop:~$ rosdep install --from-paths src --ignore-src -y`

Install GeographicLib datasets:

`jetson@desktop:~$ sudo ./src/mavros/mavros/scripts/install_geographiclib_datasets.sh`

Build source:

`jetson@desktop:~$ catkin build`

Make sure that you use setup.bash or setup.zsh from workspace.
Else rosrun can't find nodes from this workspace.

`jetson@desktop:~$ source devel/setup.bash ` \n
 [I'm a reference-style link][Arbitrary case-insensitive reference text] 
