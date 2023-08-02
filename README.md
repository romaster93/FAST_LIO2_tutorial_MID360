# FAST LIO2 tutorial

본 포스트는 FAST-LIO2를 사용하는데 기본적인 내용들을 담고있다. <br>

해당 내용들은 코드를 보지 않고 파일 구성만 알고있다면 쉽게 따라 할 수 있다. <br>

또한 기본적인 내용 이외에 LIVOX의 MID360을 사용하기 위한 내용도 포함되어있으니 필요하다면 참고하면 된다. <br>

reference link : https://github.com/hku-mars/FAST_LIO<br><br>

## **1. Dependency Install**

먼저 사용에 필요한 의존성 파일 및 라이브러리를 설치한다. <br>

현재 작성자가 사용하고 있는 환경은 ROS1 noetic을 사용중이며, ROS1 melodic 환경에서도 테스트를 진행했었다. (잘 작동했다.) <br>

실제로 작동하는거랑 simulation에서 작동하는거랑 livox mid 360 lidar 구성이 조금 다른데 simulation 작동은 추후에 업데이트 할 예정이다. <br><br>

## **1.1 PCL and Eigen** <br>

필자는 ros의 기본 패키지로 pcl와 Eigen을 설치했다.<br>

### **PCL** <br>

    sudo apt-get install ros-noetic-pcl-ros

만약 최신버전이 아니라 다른 버전이 필요하면 github에서 받아서 build 해주면된다. <br>

link : https://github.com/PointCloudLibrary/pcl/releases/tag/pcl-<your_version>

    tar xvfz pcl-pcl-<version>.tar.gz

    cd pcl-pcl-<version> && mkdir build && cd build

    cmake ..

    sudo make install -j16


### **Eigen** <br>

    sudo apt-get install libeigen3-dev
<br><br>

## **1.2 Livox lidar driver (Important)** <br>

사용자가 만약에 livox lidar driver2 를 사용하는 하드웨어를 사용하더라도 FAST-LIO2 패키지를 build 하기 위해서는 해당 패키지가 반드시 필요하다. <br>

### **LIVOX SDK INSTALL** <br>

just follow command

    git clone https://github.com/Livox-SDK/Livox-SDK.git
    cd Livox-SDK
    cd build && cmake ..
    make
    sudo make install

<br>

### **Livox ros driver INSTALL** <br>

기본 livox ros driver 패키지 설치 가이드를 보면 ws_livox라는 폴더에 따로 설치하는걸 권유하는데 그렇게 관리하면 경로 설정을 추가로 진행해야되기 때문에 필자는 기존 workspace인 catkin_ws에서 진행했다. <br>
또한 필자는 catkin_make 보다 catkin build를 선호하는편이라 catkin build를 사용했다. <br>

    cd catkin_ws/src
    git clone https://github.com/Livox-SDK/livox_ros_driver.git
    cd ..
    cd catkin build 
    source /devel/setup.sh

설치가 다 됬으면 FAST-LIO2 설치를 진행한다. <br>


## **2. FAST-LIO2 Install**

    cd catkin_ws/src
    git clone https://github.com/hku-mars/FAST_LIO.git
    cd FAST_LIO
    git submodule update --init
    cd ../..
    catkin build
    source devel/setup.bash

여기까지 하면 기본적인 설치는 다 끝났다. <br>

## **3. livox ros driver2 install for livox mid360**
livox ros driver2도 마찬가지로 ws_livox라는 workspace를 따로 설치하라고 하는데 나는 관리하기 불편해서 사용하던 workspace에 그대로 설치했다. <br>
해당 ros driver는 앞서 설치한 버전1과 겹치는 부분이 조금 있어서 설치할때 사용하는 스크립트 파일을 약간 수정했다. <br>

    git clone https://github.com/Livox-SDK/livox_ros_driver2.git

(수정한 스크립트 파일은 github 파일을 참고)<br>

    source /opt/ros/noetic/setup.sh
    ./build.sh ROS1
    cd catkin_ws
    catkin build livox_ros_driver2
    source /devel/setup.bash

설치가 완료되면 config 파일을 수정해야한다. <br>

    cd catkin_ws/src/livox_ros_driver2/config
    vim MID360_config.json

    "host_net_info" : 
      "cmd_data_ip" : "192.168.1.5",  --> lidar ip로 수정
      "cmd_data_port": 56101,
      "push_msg_ip": "192.168.1.5",  --> lidar ip로 수정
      "push_msg_port": 56201,
      "point_data_ip": "192.168.1.5", --> lidar ip로 수정
      "point_data_port": 56301,
      "imu_data_ip" : "192.168.1.5",  --> lidar ip로 수정

    "lidar_configs" : 
      "ip" : "192.168.1.1XX",  --> lidar의 시리얼 번호 끝 2자리를 입력 

msg_MID360.launch 파일 수정

	<arg name="bd_list" default="100000000000000"/>   --> 해당 value 제품의 시리얼 넘버로 수정


launch 파일을 위에서는 msg파일을 수정했는데 rviz로 데이터를 같이 보길 희망하면 rviz가 포함된 launch파일도 똑같이 수정하여 실행하면 된다. <br>

    roslaunch livox_ros_driver2 msg_MID360.launch






























