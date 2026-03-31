# LAB5 - limo vehicle Simulation

-- Course: *Intelligent Robotics – Professor: Qi Hao*

-- Environment: Ubuntu 20.04, ros-noetic

编写 孙耀威 12232434

[TOC]

# 1.gazebo中limo小车的移动仿真

## 安装


 limo_description: The file is the function package of model file

 limo_gazebo_sim: The folder is gazebo simulation function package

### 下载环境

 Download and install ros-control function package, ros-control is the robot control middleware provided by ROS

```
# ubuntu18
sudo apt-get install ros-melodic-ros-control

# ubuntu20
sudo apt-get install ros-noetic-ros-control
```

 Download and install ros-controllers function package, ros-controllers are the kinematics plug-in of common models provided by ROS

```
# ubuntu18
sudo apt-get install ros-melodic-ros-controllers

# ubuntu20
sudo apt-get install ros-noetic-ros-controllers
```

 Download and install gazebo-ros function package, gazebo-ros is the communication interface between gazebo and ROS, and connect the ROS and Gazebo

```
# ubuntu18
sudo apt-get install ros-melodic-gazebo-ros

# ubuntu20
sudo apt-get install ros-noetic-gazebo-ros
```

 Download and install gazebo-ros-control function package, gazebo-ros-control is the communication standard controller between ROS and Gazebo

```
# ubuntu18
sudo apt-get install ros-melodic-gazebo-ros-control

# ubuntu20
sudo apt-get install ros-noetic-gazebo-ros-control
```

 Download and install joint-state-publisher-gui package.This package is used to visualize the joint control.

```
# ubuntu18

sudo apt-get install ros-melodic-joint-state-publisher-gui 
# ubuntu20
sudo apt-get install ros-noetic-joint-state-publisher-gui 
```

 Download and install rqt-robot-steering plug-in, rqt_robot_steering is a ROS tool closely related to robot motion control, it can send the control command of robot linear motion and steering motion, and the robot motion can be easily controlled through the sliding bar

```
# ubuntu18
sudo apt-get install ros-melodic-rqt-robot-steering 

# ubuntu20
sudo apt-get install ros-noetic-rqt-robot-steering 
```

 Download and install teleop-twist-keyboard function package, telop-twist-keyboard is keyboard control function package, the robot can be controlled to move forward, left, right and backward through "i", "j", "l",and "," on the keyboard

```
# ubuntu18
sudo apt-get install ros-melodic-teleop-twist-keyboard 

# ubuntu20
sudo apt-get install ros-noetic-teleop-twist-keyboard 
```

### 安装limo 仿真库

#### 1.	Create workspace, download simulation model function package and compile

 Open a new terminal and create a workspace named limo_ws, enter in the terminal:

```
mkdir limo_ws
```

 Enter the limo_ws folder

```
cd limo_ws
```

 Create a folder to store function package named src
```
mkdir src
```

 Enter the src folder

```
cd src
```

 Initialize folder

```
catkin_init_workspace
```

 Download simulation model function package

```
git clone https://github.com/Intelligent-Robot-Course/ugv_gazebo_sim.git
```

 Enter the limo_ws folder

```
cd limo_ws
```

 Confirm whether the dependency of the function package is installed

```
rosdep install --from-paths src --ignore-src -r -y 
```

 Compile

```
catkin_make
```

## 2.	在rviz中查看limo小车的模型


 Enter the limo_ws folder

```
cd limo_ws
```

Declare the environment variable

```
source devel/setup.bash
```

Run the start file of limo and visualize the model in Rviz

```
roslaunch limo_description display_models.launch 
```

![img](./images/img3.png) 

## 3.	Start the gazebo simulation environment of limo and control limo movement in the gazebo

在gazebo中进行limo小车的运动仿真

Enter the limo_ws folder

```
cd limo_ws
```

Declare the environment variable

```
source devel/setup.bash
```

Start the simulation environment of limo, limo have two movement mode, the movement mode is Ackerman mode

开启limo的gazebo环境仿真
1. 打开空白的世界诶
```
#这条命令会打开一个空白的世界
roslaunch limo_gazebo_sim limo_ackerman.launch
```
2. 打开含有地图的世界

```
roscd limo_gazebo_sim
cd launch
gedit limo_ackerman.launch
```
可以看到 limo_ackerman.launch 文件，其中第五行有个world_name的参数,默认参数是empty.world

可以选择的世界有两种
```
willowgarage.world
clearpath_playpen.world
```
选择其一替换掉empty.world，然后再运行
```
roslaunch limo_gazebo_sim limo_ackerman.launch
```
这里因为修改了默认模型，所以会打开新的世界而不是空白的世界

下图为使用willowgarage.world的世界中rviz的图像与gazebo的图像

![img](./images/img4.png) 

![img](./images/img5.png) 

### 控制方法1
Start rqt_robot_steering movement control plug-in, the sliding bar can control the robot motion

打开一个简单的控制器，可以通过简单的速度控制和角度控制来控制车辆移动
```
rosrun rqt_robot_steering rqt_robot_steering
```

![img](./images/img1.png) 

### 控制方法2

Four-wheel differential steering movement mode

```
roslaunch limo_gazebo_sim limo_four_diff.launch 
```

### 控制方法3

Control by keyboard, the robot can be controlled to move forward, left, right and backward through "i", "j", "l",and "," on the keyboard

```
rosrun teleop_twist_keyboard teleop_twist_keyboard.py 
```
如果感觉转弯效果不明显，可以按e增加转弯控制的极限

比如原来按转弯前轮只转10度，按了后就变成11度，再按就变12.1度，转弯会逐渐变明显
![img](./images/img2.png) 

## Demo of line detection and controlling

The code is transformed from [turtlebot3 2020 autorace](https://github.com/ROBOTIS-GIT/turtlebot3_autorace_2020).

During the difference between robot models, the controlling performance is worse in limo.

This demo is only an example pipline with only controlling according the line.

### 1. open the world

Add the race world file to gazebo world
```bash
echo "export GAZEBO_MODEL_PATH=$GAZEBO_MODEL_PATH:~/limo_ws/src/ugv_gazebo_sim/limo/models" >> ~/.bashrc
source ~/.bashrc
```
Open the world in gazebo:

```bash
roslaunch limo_gazebo_sim limo_demo.launch
```

![world](./images/img6.png)

### 2. extrinsic camera calibration

The image need to be project to ground as input of line detection.

```bash
roslaunch limo_camera extrinsic_camera_calibration.launch
```

The projected picture is publish to */camera/image_projected_compensated*
You also can config parameters as follow.

```bash
roslaunch limo_camera extrinsic_camera_calibration.launch mode:=calibration
rqt
rosrun rqt_reconfigure rqt_reconfigure
```

![config extrinsic](./images/img7.png)

Use *plugin - Visulization - Image view* to check topic */camera/image_extrinsic_calib/compressed* and topic */camera/image_projected_compensated*.
You need to guarantee the view contain both lines are in */camera/image_projected_compensated*.

![rqt_reconfig1](./images/img8.png)

After adjust the parameter, please change yaml file under *ugv_gazebo/limo/limo_camera/calibration/extrinsic_calibration*

### 3. line detection

After projected image to ground, we need to detect the yellow line (left) and the wite line (right).

 ```bash
 roslaunch limo_detect detect_lane.launch
 ```

 The result is published in */detect/image_lane/compressed*

 ![detect result](./images/img9.png)

 You can also adjuest parameters as extrinsic camera calibration

 ```bash
 roslaunch limo_detect detect_lane.launch mode:=calibration
 rosrun rqt_reconfigure rqt_reconfigure
 ```
![rqt_reconfig2](./images/img10.png)

### 4. controlling 

The controlling part is ordered to minimize the error between current direction and planed direction, and publish to */cmd_vel* .

```bash
roslaunch limo_driving turtlebot3_autorace_control_lane.launch
```

Because there are blind spot at front, it can't follow the path.
