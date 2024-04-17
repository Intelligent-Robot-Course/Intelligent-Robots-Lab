# LAB7

## Lab7 task:
* 参考part1引导学习迁移代码并在仿真中进行测试
* 在真实场景中复现part2 part3

## PART 1 Obstacle avoidance for robots

**以下代码为之前在turtle bot3仿真中进行的工作，直接用于Limo无法正常运行，因此需要进行迁移操作。本节将简要介绍如何进行代码迁移。请务必阅读文字引导，不要直接复制粘贴代码。**

迁移步骤：
*  更改仿真环境
*  更改算法订阅及发布topic名称
*  更改数据格式，使得数据对齐

### Example 1: detect obstacle

首先克隆仓库至工作目录（以 `~/` 为例）

```
git clone https://github.com/ZhanyuGuo/smartcar_ws.git
```

大部分代码不需要更改，因此可先按照原先步骤运行，需要更该部分將用**粗体标注**

step 1:

```
$cd ~/smartcar_ws/src
$catkin_create_pkg smartcar_demo actionlib actionlib_msgs geometry_msgs interactive_markers nav_msgs rospy sensor_msgs std_msgs visualization_msgs
```

step 2:

```
$cd smartcar_demo
$mkdir scripts
$cd scripts
$touch smartcar_obstacle.py
```



smartcar_obstacle.py

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*
import rospy
import math
from sensor_msgs.msg import LaserScan
from geometry_msgs.msg import Twist
 
LINEAR_VEL = 0.20
STOP_DISTANCE = 2
LIDAR_ERROR = 0.6
SAFE_STOP_DISTANCE = STOP_DISTANCE + LIDAR_ERROR
 
class Obstacle():
    def __init__(self):
        self._cmd_pub = rospy.Publisher('cmd_vel', Twist, queue_size=1)
        self.obstacle()
        
    def get_scan(self):
        scan = rospy.wait_for_message('scan', LaserScan)   #从‘scan’ 画图读取消息类型laserscan 
        scan_filter = []
       
        samples = len(scan.ranges)  # The number of samples is defined in     返回列表元素个数
                                    # turtlebot3_<model>.gazebo.xacro file,
                                    # the default is 360.
 
        samples_view = 1            # 1 <= samples_view <= samples
        
        if samples_view > samples:  #samples 小于等于1
            samples_view = samples  
 
        if samples_view == 1:       
            scan_filter.append(scan.ranges[0])  #将雷达距离数据range【0】，添加到列表scanfilter中
 
        else:   #samples_view不等于1 ，大于1   ；小于1
            left_lidar_samples_ranges = -(samples_view//2 + samples_view % 2)
            right_lidar_samples_ranges = samples_view//2
            
            left_lidar_samples = scan.ranges[left_lidar_samples_ranges:]  #[m : ] 代表列表中的第m+1项到最后一项
            right_lidar_samples = scan.ranges[:right_lidar_samples_ranges] #[ : n] 代表列表中的第一项到第n项
            scan_filter.extend(left_lidar_samples + right_lidar_samples)  #extend() 函数用于在列表末尾一次性追加另一个序列中的多个值
 
        for i in range(samples_view):   # Python for循环可以遍历任何序列的项目，如一个列表或者一个字符串。
            if scan_filter[i] == float('Inf'):  #正负无穷
                scan_filter[i] = 3.5
            elif math.isnan(scan_filter[i]):   #是不是非数字浮点数
                scan_filter[i] = 0
        
        return scan_filter
 
    def obstacle(self):
        twist = Twist()      
        turtlebot_moving = True
 
        while not rospy.is_shutdown():
            lidar_distances = self.get_scan()
            min_distance = min(lidar_distances) 
            if min_distance < SAFE_STOP_DISTANCE:
                if turtlebot_moving:
                    twist.linear.x = 0.0
                    twist.angular.z = 0.0
                    self._cmd_pub.publish(twist)
                    turtlebot_moving = False
                    rospy.loginfo('Stop!')
            else:
                twist.linear.x = LINEAR_VEL
                twist.angular.z = 0.0
                self._cmd_pub.publish(twist)
                turtlebot_moving = True
                rospy.loginfo('Distance of the obstacle : %f', min_distance)
 
def main():
    rospy.init_node('smartcar_obstacle')
    try:
        obstacle = Obstacle()
    except rospy.ROSInterruptException:
        pass
 
if __name__ == '__main__':
    main()
 
```

**在此需要对订阅及发布节点名称进行改变**

在这段代码中，订阅节点为*/scan*，数据类型为雷达点云；发布节点为*/cmd_vel*，数据类型为差速轮转向信息
但是在Limo通讯中雷达发布节点与turtlebot3并不一致，因此需要更改。
```python
scan = rospy.wait_for_message('/limo/scan', LaserScan)   #从‘/limo/scan’ 画图读取消息类型laserscan 
```


1. In terminal

Turtlebot3 对应仿真环境：

```
$roslaunch smartcar_gazebo smartcar_with_laser_nav.launch
```

**在此需要对仿真环境进行改变**

需要开启Limo对应的仿真环境，因此上条命令需要更改为limo对应指令。
在此算法中，模型为差速轮，因此在**ugv_gazebo_sim/limo/limo_gazebo_sim/launch/limo_four_diff.launch**基础上进行修改。
在该launch文件中，默认世界为空白世界，无障碍物，无法对算法进行测试。因此还需要更改其地图。
再地图素材中*willowgarage.world*较为贴合，可如下更改*world_name*。
```
<arg name="world_name" default="$(find limo_gazebo_sim)/worlds/willowgarage.world"/>
```
运行launch加载地图
```
roslaunch limo_gazebo_sim limo_four_diff.launch
```



2. In new terminal

```
$rosrun smartcar_demo smartcar_obstacle.py
```

**在此需要进行数据对齐**

在跑完之后会发现车辆并无法正常停止，根据输出可发现其最短距离一直浮动，较为诡异，查看代码后发现问题。
*get_scan*函数仅需要返回所有正常点云数据即可，而按照其现有的代码逻辑，仅返回了*scan.ranges[0]*。
turtlebot3的雷达index代表对应的角度，比如*scan.ranges[0]*为正前方深度，但是limo并不是数据与其不同。
可通过rqt对包头进行查看，可在工具栏中：*Plogins->Topics->Topic Monitors*，勾选*/limo/scan*。
发现其角度为[-2.0943,2.0943]，并且包含720个点，因此*scan.ranges[360]*才为正前方。

因此其需要按照如下进行更改。
```
samples_view = 10            # 1 <= samples_view <= samples
        
if samples_view > samples:  #samples 小于等于1
    samples_view = samples  

if samples_view == 1:       
    scan_filter.append(scan.ranges[360])  #将雷达距离数据range【0】，添加到列表scanfilter中

else:   #samples_view不等于1 ，大于1   ；小于1
    left_lidar_samples_ranges = -(samples_view//2 + samples_view % 2)
    right_lidar_samples_ranges = samples_view//2
    
    left_lidar_samples = scan.ranges[left_lidar_samples_ranges+360:360]  #[m : ] 代表列表中的第m+1项到最后一项
    right_lidar_samples = scan.ranges[360:right_lidar_samples_ranges+360] #[ : n] 代表列表中的第一项到第n项
    scan_filter.extend(left_lidar_samples + right_lidar_samples)  #extend() 函数用于在列表末尾一次性追加另一个序列中的多个值

for i in range(samples_view):   # Python for循环可以遍历任何序列的项目，如一个列表或者一个字符串。
    if scan_filter[i] == float('Inf'):  #正负无穷
        scan_filter[i] = 8
    elif math.isnan(scan_filter[i]):   #是不是非数字浮点数
        scan_filter[i] = 0
```
再次运行，小车即可正常运作。

### smartcar explore maze

**这一部分仅提供原先代码，但本质与上个相同，请根据上述引导进行对应代码更改，并且正确运行(包括roslaunch开启gazebo的命令)**

```
$cd ~/smartcar_ws/src/smartcar_demo/scripts
$touch maze_explorer.py
```

maze_explorer.py

```python
#!/usr/bin/env python3

import rospy, time
from geometry_msgs.msg import Twist
from sensor_msgs.msg import LaserScan

def scan_callback(msg):
    global range_front
    global range_right
    global range_left
    global ranges
    global min_front,i_front, min_right,i_right, min_left ,i_left
    
    ranges = msg.ranges
    # get the range of a few points
    # in front of the robot (between 5 to -5 degrees)
    range_front[:5] = msg.ranges[5:0:-1]  
    range_front[5:] = msg.ranges[-1:-5:-1]
    # to the right (between 300 to 345 degrees)
    range_right = msg.ranges[300:345]
    # to the left (between 15 to 60 degrees)
    range_left = msg.ranges[60:15:-1]
    # get the minimum values of each range 
    # minimum value means the shortest obstacle from the robot
    min_range,i_range = min( (ranges[i_range],i_range) for i_range in range(len(ranges)) )
    min_front,i_front = min( (range_front[i_front],i_front) for i_front in range(len(range_front)) )
    min_right,i_right = min( (range_right[i_right],i_right) for i_right in range(len(range_right)) )
    min_left ,i_left  = min( (range_left [i_left ],i_left ) for i_left  in range(len(range_left )) )
    

# Initialize all variables
range_front = []
range_right = []
range_left  = []
min_front = 0
i_front = 0
min_right = 0
i_right = 0
min_left = 0
i_left = 0

# Create the node
cmd_vel_pub = rospy.Publisher('cmd_vel', Twist, queue_size = 1) # to move the robot
scan_sub = rospy.Subscriber('scan', LaserScan, scan_callback)   # to read the laser scanner
rospy.init_node('maze_explorer')

command = Twist()
command.linear.x = 0.0
command.angular.z = 0.0
        
rate = rospy.Rate(10)
time.sleep(1) # wait for node to initialize

near_wall = 0 # start with 0, when we get to a wall, change to 1

# Turn the robot right at the start
# to avoid the 'looping wall'
print("Turning...")
command.angular.z = -0.5
command.linear.x = 0.1
cmd_vel_pub.publish(command)
time.sleep(2)
       
while not rospy.is_shutdown():

    # The algorithm:
    # 1. Robot moves forward to be close to a wall
    # 2. Start following left wall.
    # 3. If too close to the left wall, reverse a bit to get away
    # 4. Otherwise, follow wall by zig-zagging along the wall
    # 5. If front is close to a wall, turn until clear
    while(near_wall == 0 and not rospy.is_shutdown()): #1
        print("Moving towards a wall.")
        if(min_front > 0.2 and min_right > 0.2 and min_left > 0.2):    
            command.angular.z = -0.1    # if nothing near, go forward
            command.linear.x = 0.15
            print ("C")
        elif(min_left < 0.2):           # if wall on left, start tracking
            near_wall = 1       
            print ("A")            
        else:
            command.angular.z = -0.25   # if not on left, turn right 
            command.linear.x = 0.0

        cmd_vel_pub.publish(command)
        
    else:   # left wall detected
        if(min_front > 0.2): #2
            if(min_left < 0.12):    #3
                print("Range: {:.2f}m - Too close. Backing up.".format(min_left))
                command.angular.z = -1.5
                command.linear.x = -0.07
            elif(min_left > 0.15):  #4
                print("Range: {:.2f}m - Wall-following; turn left.".format(min_left))
                command.angular.z = 1.5
                command.linear.x = 0.07
            else:
                print("Range: {:.2f}m - Wall-following; turn right.".format(min_left))
                command.angular.z = -1.5
                command.linear.x = 0.07
                
        else:   #5
            print("Front obstacle detected. Turning away.")
            command.angular.z = -1.0
            command.linear.x = 0.0
            cmd_vel_pub.publish(command)
            while(min_front < 0.3 and not rospy.is_shutdown()):      
                pass
        # publish command 
        cmd_vel_pub.publish(command)
    # wait for the loop
    rate.sleep()
```

1. In terminal

```
$roslaunch smartcar_gazebo smartcar_with_laser_nav.launch
```



2. In new terminal

```
$rosrun smartcar_demo maze_explorer.py
```

 

## PART 2 ROS+ OPENCV

Install Opencv

根据安装的 `ROS` 版本选择须安装的包。下文安装命令中的 `<distro>` 处应填入发行版本（noetic/melodic）。可通过指令 `rosversion -d` 查看本机安装的发行版本。

```
$sudo apt-get install ros-<distro>-vision-opencv libopencv-dev python3-opencv
```



A simple example

```
$sudo apt-get install ros-<distro>-usb-cam # 安装摄像头功能包
$roslaunch usb_cam usb_cam-test.launch # 启动功能包
$rqt_image_view # 可视化工具
```



step1:In your package.xml and CMakeLists.txt (or when you use catkin_create_pkg), add the
following dependencies:

```
$cd ~/smartcar_ws/src
$catkin_create_pkg learning_opencv sensor_msgs roscpp std_msgs image_transport
```

step2:Modify CMakeLists.txt
Add one line in CMakeLists.txt:

```
find_package(catkin REQUIRED COMPONENTS
    image_transport
    cv_bridge
    ...
)
find_package(OpenCV REQUIRED)
```

step3: Create a node(write a cpp file) at `smartcar_ws/src/learning_opencv/src/image_converter.cpp`:

```c++
#include <ros/ros.h>
#include <image_transport/image_transport.h>
#include <cv_bridge/cv_bridge.h>
#include <sensor_msgs/image_encodings.h>
#ifdef ROS_NOETIC
#include <opencv4/opencv2/imgproc/imgproc.hpp>
#include <opencv4/opencv2/highgui/highgui.hpp>
#else 
#include <opencv2/imgproc/imgproc.hpp>
#include <opencv2/highgui/highgui.hpp>
#endif
// Add vision object here
using namespace std;
using namespace cv;
static const std::string OPENCV_WINDOW="Image window";

class ImageConverter
{
  ros::NodeHandle nh_;
  image_transport::ImageTransport it_;
  image_transport::Subscriber image_sub_;
  image_transport::Publisher image_pub_;

public:
  ImageConverter()
    : it_(nh_)
  {
    // Subscrive to input video feed and publish output video feed
    image_sub_ = it_.subscribe("/usb_cam/image_raw", 1,
      &ImageConverter::imageCb, this);
    image_pub_ = it_.advertise("/image_converter/output_video", 1);

    cv::namedWindow(OPENCV_WINDOW);
    
  }

  ~ImageConverter()
  {
    cv::destroyWindow(OPENCV_WINDOW);
  }

  void imageCb(const sensor_msgs::ImageConstPtr& msg)
  {
    cv_bridge::CvImagePtr cv_ptr;
    try
    {
      cv_ptr = cv_bridge::toCvCopy(msg, sensor_msgs::image_encodings::BGR8);
    }
    catch (cv_bridge::Exception& e)
    {
      ROS_ERROR("cv_bridge exception: %s", e.what());
      return;
    }

    if (cv_ptr->image.rows >60&& cv_ptr->image.cols>60){
        cv::circle(cv_ptr->image,cv::Point(50,50),10,CV_RGB(255,0,0));
    }
  
    cv::imshow(OPENCV_WINDOW,cv_ptr->image);
    cv::waitKey(3);

    
    // Output modified video stream
    image_pub_.publish(cv_ptr->toImageMsg());
  }
};


int main(int argc, char** argv)
{
  ros::init(argc, argv, "image_converter");
  ImageConverter ic;
  ros::spin();
  return 0;
}

```

step4: Building node

Add two lines to the bottom of the `smartcar_ws/src/learning_opencv/CMakeLists.txt`:

```
add_executable(image_converter src/image_converter.cpp)
target_link_libraries(image_converter ${catkin_LIBRARIES} ${OpenCV_LIBRARIES})
```

Then build with `catkin_make` under `smartcar_ws/`

step5: Run to see the results (run `roslaunch usb_cam usb_cam-test.launch` in a separate terminal before running following commands)

```
$source devel/setup.bash
$rosrun learning_opencv image_converter
```



## PART3 YOLO ROS

step1: Download code

```
cd ~/smartcar_ws/src
git clone --recursive https://github.com/leggedrobotics/darknet_ros
```

step2: Download weights

```
cd ~/smartcar_ws/src/darknet_ros/darknet_ros/yolo_network_config/weights/
wget http://pjreddie.com/media/files/yolov2.weights
wget http://pjreddie.com/media/files/yolov2-tiny.weights
wget https://pjreddie.com/media/files/yolov3.weights
wget https://pjreddie.com/media/files/yolov3-tiny.weights
```

step3: Building

```
cd ~/smartcar_ws
catkin_make -DCMAKE_BUILD_TYPE=Release
```

step4: Test

1. 发布图像话题, 直接将电脑自带摄像头或连接电脑的USB摄像头采集的图像发布为ROS图像话题并使用。

```
sudo apt-get install ros-noetic-usb-cam
roslaunch usb_cam usb_cam-test.launch
```

2. 运行darknet_ros

   执行darknet_ros进行检测，在运行检测之前需要更改一下文件，使得darknet_ros订阅的话题与usb_cam发布的图片话题对应。打开 darknet_ros/config/ros.yaml文件，找到

```
camera_reading:
	topic:/camera/rgb/image_raw
```

convert to

```
camera_reading:
	topic:/usb_cam/image_raw
```

3. 回到smartcar_ws目录，执行：

```
roslaunch darknet_ros darknet_ros.launch
```



### Yolo for smartcar


```
camera_reading:
	topic:/usb_cam/image_raw
```

convert to

```
camera_reading:
	topic:/limo/color/image_raw
```

再地图素材中*bus.world*较为贴合，可如下更改*world_name*(刚刚更新，需要git pull ugv_gazebo_sim仓库)。
```
<arg name="world_name" default="$(find limo_gazebo_sim)/worlds/bus.world"/>
```
运行launch加载地图
```
roslaunch limo_gazebo_sim limo_four_diff.launch
```

另开teminal:

```
roslaunch darknet_ros darknet_ros.launch
```

