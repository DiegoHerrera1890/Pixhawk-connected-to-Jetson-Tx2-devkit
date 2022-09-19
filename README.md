# Running MAVROS with Jetson TX2 and PixHawk 4

In this post, I'm running MAVROS in the TX2 connected to the Pixhawk 4 with PX4 and QGroundControl. First of all, we need to install ROS Melodic from the [ROS official website](http://wiki.ros.org/melodic/Installation/Ubuntu "ROS-Melodic for Ubuntu 18.04") 

## 1. ROS-Installation:

##### 1.1 Configure your Ubuntu repositories
Configure your Ubuntu repositories to allow "restricted," "universe," and "multiverse." You can follow the Ubuntu guide for instructions on doing this.

#### 1.2 Setup your sources.list
Setup your computer to accept software from packages.ros.org.

`sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'`

#### 1.3 Set up your keys

`sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654`

#### 1.4 Installation
First, make sure your Debian package index is up-to-date:

`sudo apt update 
 sudo apt install ros-melodic-desktop-full`

#### 1.5 Initialize rosdep
Before you can use ROS, you will need to initialize rosdep. rosdep enables you to easily install system dependencies for source you want to compile and is required to run some core components in ROS.

`sudo rosdep init
 rosdep update`

#### 1.6 Environment setup
It's convenient if the ROS environment variables are automatically added to your bash session every time a new shell is launched:

`echo "source /opt/ros/melodic/setup.bash" >> ~/.bashrc
 source ~/.bashrc`

#### 1.7 Dependencies for building packages
Up to now you have installed what you need to run the core ROS packages. To create and manage your own ROS workspaces, there are various tools and requirements that are distributed separately. For example, rosinstall is a frequently used command-line tool that enables you to easily download many source trees for ROS packages with one command.

To install this tool and other dependencies for building ROS packages, run:

`sudo apt install python-rosinstall python-rosinstall-generator python-wstool build-essential`

## 2. MAVROS-Installation:
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

`jetson@desktop:~$ source devel/setup.bash `

 take it from [Mavros github offcial](https://github.com/mavlink/mavros/blob/master/mavros/README.md) 
 
 
 ## 3. JETSON TX2 Configuration
 In this part, I will explain how to setup the JTX2 devkit to connect with the PiXHawk 4 (PX4)
 First, we need to locate the UART 1 in the TX2 devkit
 
 Here's our UART 1:

![alt text](https://github.com/DiegoHerrera1890/Pixhawk-connected-to-Jetson-Tx2-devkit/blob/master/uart_tx2.jpg "UART 1  /dev/ttyTHS2")

 and here we have the serial port header connections:
 
 ![alt text](https://github.com/DiegoHerrera1890/Pixhawk-connected-to-Jetson-Tx2-devkit/blob/master/uart%201.PNG "Pin 1 is GND")
 
 If you're running the last Jetpack(4.2.3) you don't need to change the serial device port from /dev/ttyACM to /dev/ttyTHS2, it is already done in the last jetpack. Perhaps we need to give a permission to the port /dev/ttyTHS2 and add the user to the group running these commands:
 
 `sudo stop modemmanager`
 `sudo usermod -a -G dialout $USER`

## 4. PiXhawk Configuration
before star the software settings we need to connect the Pixhawk to the JTX2 via telem2
![alt text](https://github.com/DiegoHerrera1890/Pixhawk-connected-to-Jetson-Tx2-devkit/blob/master/pixhawk_LI.jpg "telem 2")
![alt text](https://github.com/DiegoHerrera1890/Pixhawk-connected-to-Jetson-Tx2-devkit/blob/master/telem2.PNG "telem 2 pin details")


| PIN | Jetson TX2 | PIN | Telem 2 |
| --- |:----------:| --- |:-------:|
| 1   | GND        |  6  |   GND   |
| 2   | RTS        |   Not connect |
| 3   | NC         |   Not connect |
| 4   | Rx         |  2  |    Tx   |
| 5   | Tx         |  3  |    Rx   |
| 6   | CTS        |   Not connect |

Now we are ready to go to the Pixhawk settings.

Connect the Pixhawk via usb to the PC (running Windows or Mac). Do not connect the battery.
Please refer to this video to setup the pixhawk with basic settings:

https://www.youtube.com/watch?v=6Dk7oSKf4wE&t=482s

after setting the Pixhawk we have to modify some parameters.
To do that open QGroundControl and go to parameters:

![alt text](https://github.com/DiegoHerrera1890/Pixhawk-connected-to-Jetson-Tx2-devkit/blob/master/parameters.PNG "parameters")

Now look for the tab "Mavlink" and change these parameters:

`MAV_1_CONFIG = TELEM 2` 

`MAV_2_CONFIG = TELEM 2`

reboot the vehicle and go again to the mavlink tab and change these parameters:

`MAV_1_MODE = Onboard` 

`MAV_1_RATE= 921600 baud` 

`MAV_1_FORWARD = True`

if these parameters does not appears change these:

`MAV_2_MODE = Onboard`

`MAV_2_RATE= 921600 baud`

`MAV_2_FORWARD = True`

If everything is fine then you are going to see something like this:
![alt text](https://github.com/DiegoHerrera1890/Pixhawk-connected-to-Jetson-Tx2-devkit/blob/master/mavlink_conf.PNG "parameters")

after this go to the Serial Tab and change:

`SER_TEL2_BAUD = 921600`

![alt text](https://github.com/DiegoHerrera1890/Pixhawk-connected-to-Jetson-Tx2-devkit/blob/master/serial_conf.PNG "serie conf")

After this is done reboot the vehicle and disconnect the USB cable from the Windows PC.

##Running MAVROS in the TX2 with the Pixhawk

In the TX2 run: `jetson@desktop:~$ roscore`
In other terminal run:
`roslaunch mavros px4.launch fcu_url:="/dev/ttyTHS2:921600" gcs_url:="udp://:UDP_port@127.0.0.1:14550"`


