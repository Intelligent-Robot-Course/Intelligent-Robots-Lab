# Intelligent-Robots-Lab4

修订：12232434 孙耀威（今天也是没有犯病的一天～）



Lab4 materials for the intelligent robotics course

> **Objectives:**
>
> 1. Gazebo 
>
> 2. Turtlebot3 simulation 
>
> 3. Network Configuration 
>
> 4. Robot sensor

## 课程任务 task
1. 安装并打开gazebo
2. 使用gazebo进行简单的turtlebot3的仿真
3. 学习ros的网络配置（为后续实际远程使用turtlebot3或者其他小车准备）
4. 学会将摄像头图像传至ros并进行简单的实物标定

（今天又是跟着~~麻瓜式~~教程的一天呢）

<br>

## Part 1: Gazebo

**The introduction of gazebo** 

- What is Gazebo? 
  - Gazebo is a 3D dynamic simulator with the ability to accurately and efficiently simulate populations of robots in complex indoor and outdoor environments. 

- A few key features of Gazebo include: 
  - multiple physics engines 
  - a rich library of robot models and environments, 
  - a wide variety of sensors, IMU, LIDAR, and camera... 
  - convenient programmatic and graphical interfaces 

- We can use Gazebo to: 
  - testing robotics algorithms, 
  - designing robots, 
  - performing regression testing with realistic scenarios



**Gazebo Installation**

- Integration of ROS and Gazebo

  - If you chose **desktop-full** installation when you installed ROS, Gazebo comes with it

  - Or Install gazebo packages individually by: 

    ```
    sudo apt install ros-noetic-gazebo 
    ```
- Metapackage gazebo_ros_pkgs
    - A set of packages provides wrappers around the stand-alone Gazebo. They provide the necessary interfaces to simulate a robot in Gazebo using ROS messages, services and dynamic reconfigure. 
    - Read the installation tutorial from this:http://gazebosim.org/tutorials?tut=ros_installing&cat=connect_ros



**Run Gazebo**

这里建议使用vmware跑ubuntu的人先把vmware的3d图像加速关闭
关闭方法：https://blog.csdn.net/chenguangbiao/article/details/120200226#:~:text=%E7%82%B9%E5%87%BB%E8%99%9A%E6%8B%9F%E6%9C%BA%EF%BC%8C%E7%82%B9%E8%87%B3%E6%9C%80%E4%B8%8B%E6%96%B9%E7%9A%84%E8%AE%BE%E7%BD%AE%203.%E7%82%B9%E5%87%BB%E6%98%BE%E7%A4%BA%E5%99%A8%EF%BC%8C,%E5%B0%86%E5%8F%B3%E4%B8%8A%E8%A7%92%E7%9A%84%E5%8A%A0%E9%80%9F3D%E5%9B%BE%E5%83%8F%E7%9A%84%E5%8B%BE%E5%8F%96%E6%B6%88%EF%BC%8C%E7%82%B9%E5%87%BB%E7%A1%AE%E5%AE%9A%E3%80%82%204.%E5%90%AF%E5%8A%A8%E8%99%9A%E6%8B%9F%E6%9C%BA%E5%90%8E


- There are two ways to run Gazebo, one by it alone, the other from ROS:

  ```
  gazebo
  ```

  ```
  rosrun gazebo_ros gazebo
  ```
  
  ![gazebo](imgs/part1_gazebo.png)



**Download all gazebo models**

- Model files load slowly and may take a long time, so we manually download the model.

- Download the model from: https://github.com/osrf/gazebo_models （自己手动进github连接里下载模型）


- Find the **.gazebo** folder in your home path (this directory is hidden, use Ctrl+h to show the hidden directory)

  ![gazebo](imgs/img1.png)


- Create the **models** folder under this folder, and Unzip the downloaded zip file into the **models** folder

  ![gazebo](imgs/img2.png)


<br>

## Part 2: Turtlebot3 Simulation

**Introduction of TurtleBot3**

- TurtleBot3 is a small, low-cost, fully programmable, ROS-based mobile robot.
- It is intended for educational, research, product prototyping and hobby application purposes. 
- TurtleBot3 aims to drastically reduce the size and price of the platform without sacrificing performance, functionality and quality. 
- Since different options, such as chassis, computer and sensors are provided for selection. TurtleBot3 can be customized in various ways.
- TurtleBot3 adopts SBC (Single Board Computer) which is more economical and smaller than existing PC and suitable for embedded system, and it also applies the latest technologies such as distance sensor and 3D printing.



**Prerequisites**

- Install dependency package

  ```
  # ubuntu20 下载这个
  sudo apt-get install ros-noetic-joy ros-noetic-teleop-twist-joy ros-noetic-teleop-twist-keyboard ros-noetic-laser-proc ros-noetic-rgbd-launch ros-noetic-depthimage-to-laserscan ros-noetic-rosserial-arduino ros-noetic-rosserial-python ros-noetic-rosserial-server ros-noetic-rosserial-client ros-noetic-rosserial-msgs ros-noetic-amcl ros-noetic-map-server ros-noetic-move-base ros-noetic-urdf ros-noetic-xacro ros-noetic-compressed-image-transport ros-noetic-rqt-image-view ros-noetic-gmapping ros-noetic-navigation ros-noetic-interactive-markers -y
  ```

    ```
  # ubuntu18 下载这个(18的没有测试，可能有问题)
  sudo apt-get install ros-melodic-joy ros-melodic-teleop-twist-joy ros-melodic-teleop-twist-keyboard ros-melodic-laser-proc ros-melodic-rgbd-launch ros-melodic-depthimage-to-laserscan ros-melodic-rosserial-arduino ros-melodic-rosserial-python ros-melodic-rosserial-server ros-melodic-rosserial-client ros-melodic-rosserial-msgs ros-melodic-amcl ros-melodic-map-server ros-melodic-move-base ros-melodic-urdf ros-melodic-xacro ros-melodic-compressed-image-transport ros-melodic-rqt-image-view ros-melodic-gmapping ros-melodic-navigation ros-melodic-interactive-markers -y
  ```

- Install turtlebot3 and simulation (Run commands below in terminal line by line)

  ```
  mkdir -p ~/turtlebot3_ws/src/
  cd ~/turtlebot3_ws/src/
  git clone https://github.com/ROBOTIS-GIT/turtlebot3_msgs.git
  git clone https://github.com/ROBOTIS-GIT/turtlebot3.git
  git clone https://github.com/ROBOTIS-GIT/turtlebot3_simulations.git
  cd ~/turtlebot3_ws && catkin_make
  echo "source ~/turtlebot3_ws/devel/setup.bash" >> ~/.bashrc
  ```



**Launch the virtual turtlebot3 in empty world**

- Identify the model used

  ```java
  echo "export TURTLEBOT3_MODEL=burger" >> ~/.bashrc
  //TURTLEBOT3_MODEL is the name of the model you are using in burger, waffle, waffle_pi
  source ~/.bashrc
  roslaunch turtlebot3_gazebo turtlebot3_empty_world.launch
  ```

- Start a keyboard control in new terminal

  ```
  roslaunch turtlebot3_teleop turtlebot3_teleop_key.launch
  ```
  
  ![teleop](imgs/part2_teleop.gif)



**Launch the virtual turtlebot3 in TurtleBot3 World**

在开启下面教程之前，把上面跑的空世界的终端关掉
- New terminal, load gazebo

  ```java
  #提前声明机器人模型，waffle的模型拥有相机和雷达两个传感器
  export TURTLEBOT3_MODEL=waffle
  //TURTLEBOT3_MODEL is the name of the model you are using in burger, waffle, waffle_pi
  
  #启动turtlebot3仿真，在这里如果没有提前声明模型而使用默认模型会发现没有相机图像
  #这是因为默认模型支配只有雷达的传感器
  roslaunch turtlebot3_gazebo turtlebot3_world.launch
  ```
  
  使用vmware可能会出现模型是透明的，能看到背景的网格被模型遮挡，但却看不到模型
  可能的解决方法是关闭vmware的3d图像加速
- 解决方法：https://blog.csdn.net/chenguangbiao/article/details/120200226#:~:text=%E7%82%B9%E5%87%BB%E8%99%9A%E6%8B%9F%E6%9C%BA%EF%BC%8C%E7%82%B9%E8%87%B3%E6%9C%80%E4%B8%8B%E6%96%B9%E7%9A%84%E8%AE%BE%E7%BD%AE%203.%E7%82%B9%E5%87%BB%E6%98%BE%E7%A4%BA%E5%99%A8%EF%BC%8C,%E5%B0%86%E5%8F%B3%E4%B8%8A%E8%A7%92%E7%9A%84%E5%8A%A0%E9%80%9F3D%E5%9B%BE%E5%83%8F%E7%9A%84%E5%8B%BE%E5%8F%96%E6%B6%88%EF%BC%8C%E7%82%B9%E5%87%BB%E7%A1%AE%E5%AE%9A%E3%80%82%204.%E5%90%AF%E5%8A%A8%E8%99%9A%E6%8B%9F%E6%9C%BA%E5%90%8E

- New terminal, load simulation 

  这个命令是让turtlebot随机的/自动的在场地里无碰撞的行驶，这样能在后面进行传感器数据检查时看到不同的数据
  ```
  roslaunch turtlebot3_gazebo turtlebot3_simulation.launch
  ```

- New terminal, load rviz
  
- 打开rviz，在rviz里查看相机传感器和雷达传感器的数据
  ```
  roslaunch turtlebot3_gazebo turtlebot3_gazebo_rviz.launch
  ```

  ![turtlebot-world2](imgs/part2_turtlebot3-world2.png)
  
  ![turtlebot-world1](imgs/part2_turtlebot3-world1.gif)

- rviz的操作
上面的命令会打开rviz界面

![turtlebot-world1](imgs/img41.png)

首先先在左边的控制栏找到laserscan，然后把它底下的size从0.01调整成0.1

然后你就能看到在右边的雷达数据点就变大了

恩，只是调整了个大小
![turtlebot-world1](imgs/img42.png)

这里需要添加相机图像，首先在rviz左下角点击add名然后在弹出的界面下点击by topic
在其中选中/camera/rgb/image_raw中的image然后点确定就好了
你会在rviz左边看到图像数据

![turtlebot-world1](imgs/img43.png)
**Gazebo example: TurtleBot3**

- Learn more about Gazebo in link: http://gazebosim.org/tutorials
- A Gazebo application example: http://emanual.robotis.com/docs/en/platform/turtlebot3/simulation/#
  - In this application, we show the convenience and power of Gazebo in simulation of SLAM and navigation.


<br>

## Part 3: Network Configuration

该实验用来教学如何通过ros进行多机器交互（比如两台电脑，又比如一台机器人跟一台电脑）

**Check the IP of both the host and remote PC** 

```
ifconfig 
```



**Clock synchronization**

```
sudo apt-get install chrony
sudo apt-get install ntpdate
sudo ntpdate ntp.ubuntu.com 
```



**Confirm Connectivity(Method 1)**

- Install ssh on both computers 

  建议在ssh之前先两台机器互相ping以下检查是否通信正常

  ```
  sudo apt-get install openssh-server
  ```

  - On remote PC (X for graphic transmission):

    ```
    ssh [-X] username@<TURTLEBOTP_IP> 
    ```

  - Try precious command on remote PC again



**Run ROS on distributed system(Method 2)**

ROS can run on multiple machines trough network 

- Set variables in host machine

  ```
  echo "export ROS_MASTER_URI=http://localhost:11311" >> ~/.bashrc
  echo "export ROS_HOSTNAME=IP_OF_TURTLEBOT" >> ~/.bashrc 
  ```

- Set variables in remote PC

  ```
  echo "export ROS_MASTER_URI=http://IP_OF_TURTLEBOT:11311" >> ~/.bashrc 
  echo "export ROS_HOSTNAME=IP_OF_PC" >> ~/.bashrc
  ```
在设置完参数后理论上就能进行ros通信了

- You can also write in ~/.bashrc by editor directly



**Control your turtlebot**

- In turtlebot (or ssh to turtlebot): 
  
  ```
  roslaunch turtlebot_bringup minimal.launch
  ```
  
- In remote PC: 
  
  ```
  roslaunch turtlebot_teleop keyboard_teleop.launch
  ```
如果这两条命令无法运行的花，可以把之前的lab里控制小海龟的部分复现在这里

在远程机器上打开小海龟

在本地机器上运行键盘控制

加油！这部分我就不放教程了，自行翻之前的lab


**Which method to use depends:** 

- l A robot may work in a place hard to reach, or more usually, it could not have an I/O device. 
- SSH is just use the host’s resources. 
- Distributed system can do some calculation in remote nodes (like MATLAB)


<br>

## Part 4: Robot Sensor

### Camera

这部分教学如何通过将相机图像传至ros，详情参考lab3部分

### 打开相机并查看图像
*Remenber change setting usb to usb3.1 and open camera if you use VMWare*

For ubuntu 18.04:

```bash
sudo apt-get install ros-melodic-uvc-camera
sudo apt-get install ros-melodic-image-*
sudo apt-get install ros-melodic-rqt-image-view
roscore
rosrun uvc_camera uvc_camera_node
rostopic list
rosrun image_view image_view image:=/image_raw
rqt_image_view
```

For ubuntu 20.04:
```
sudo apt-get install ros-noetic-camera-info-manager ros-noetic-image-view
cd ~/catkin_ws/src

# 默认的分支是main，适合ubuntu 20.04的分支是develop，执行git clone后需要切换到的develop分支：https://wiki.ros.org/usb_cam
git clone https://github.com/ros-drivers/usb_cam.git
cd ..
catkin_make
source devel/setup.bash
# open camera
roslaunch usb_cam usb_cam-test.launch
```


**Camera Calibration**

相机标定

Camera calibration is necessary if you are measuring distance from images acquired with a stereo camera or processing images for object recognition.

- Prepare a Chessboard for calibration
  
- 自己打印
- 
  ![camera-calibration](imgs/cal_img.png)
- 
  ```
  #ubuntu18
  rosrun camera_calibration cameracalibrator.py --size 6x4 --square 0.024 image:=/image_raw camera:=/camera
  
  #ubuntu20
  rosrun camera_calibration cameracalibrator.py --size 6x4 --square 0.024 image:=/usb_cam/image_raw camera:=/usb_cam
  ```

  ![camera-calibration](imgs/part4_camera-calibration.png)

（此处该有标定教程）

https://blog.csdn.net/Will_Ye/article/details/125064472

ubuntu18
  ```
  cd /tmp 
  tar -xvzf calibrationdata.tar.gz 
  mv ost.txt ost.ini 
  rosrun camera_calibration_parsers convert ost.ini camera.yaml 
  mkdir ~/.ros/camera_info 
  mv camera.yaml ~/.ros/camera_info/ 
  rosrun uvc_camera uvc_camera_node
  ```

ubuntu20直接跟着链接的教程修改launch文件，在里面添加标定文件并成功重运行就好



**Kinect**

（下面部分仅供含有kinect设备的人参考，该设备常见于turtlebot3实车上，没有的人应该也许不需要跑下面的代码）


Kinect is an XBOX game device which contains a camera and an infrared depth camera.

- Install needed packages: 

  ```
  sudo apt-get install ros-noetic-openni-* ros-noetic-openni2-* ros-noetic-freenect-* 
  ```

- Check these packages: 

  ```
  rospack profile 
  ```

- Set the environment variable: 

  ```
  echo "export TURTLEBOT_3D_SENSOR=kinect" >> ~/.bashrc
  ```

- Test Kinect if it works well 

  - In host machine: 

    ```
    roslaunch turtlebot_bringup minimal.launch
    roslaunch freenect_launch freenect-registered-xyzrgb.launch 
    ```

  - In remote PC: 

    ```
    rosrun rqt_image_view rqt_image_view image:=/camera/rgb/image_color
    ```



**Sensor Information**

- Kinect can provide image as well as depth information
  - Lidar
  - imu
  - Odometry - Odometry is another important sensor. It provides *nav_msgs/Odometry* message by base, defaultly */odom*


<br>


## Lab References

**Further References**

- Gezebo 
  - http://gazebosim.org

- TurtleBot 
  - http://wiki.ros.org/Robots/TurtleBot 
- TurtleBot3 
  - http://emanual.robotis.com/docs/en/platform/turtlebot3

- 标定教程参考
  - https://blog.csdn.net/Will_Ye/article/details/125064472
