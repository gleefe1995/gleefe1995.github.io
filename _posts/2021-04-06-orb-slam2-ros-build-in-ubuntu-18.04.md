---
layout: post
title: "orb slam2 build in ubuntu 18.04"
date: 2021-04-06 17:05:30 -0400
categories: orbslam2
use_math: true
---



## Prerequisites



#### Pangolin

[Pangolin](https://github.com/stevenlovegrove/Pangolin) 여기서 시키는 대로 설치해주자.

#### Eigen3

```
sudo apt-get install libeigen3-dev
```

#### OpenCV

OpenCV도 깔아주자. ROS를 설치하면 자동으로 깔린다.

#### DBoW2 and g2o

Bag of words랑 g2o는 ORB_SLAM2 Thirdparty 안에 있다.



## Building ORB-SLAM2 



```
//본인 workspace/src에
git clone https://github.com/anliec/ORB_SLAM2.git
```

원작자거는 16.04꺼라 그런지 빌드하면 헤더오류가 나는데, anliec 이분이 다 추가해주신거같다.

```
cd ORB_SLAM2
chmod +x build.sh
./build.sh
```

### KITTI Dataset으로 돌려보기

```
./Examples/Monocular/mono_kitti Vocabulary/ORBvoc.txt Examples/Monocular/KITTIX.yaml PATH_TO_DATASET_FOLDER/dataset/sequences/SEQUENCE_NUMBER
```

[여기](http://www.cvlibs.net/datasets/kitti/eval_odometry.php)서 grayscale 데이터셋을 받으면 되는데, 22GB나 된다. 

./Examples/Monocular/mono_kitti -> kitti dataset을 불러오는 실행파일

Vocabulary/ORBvoc.txt -> bag of words 파일(단어장같은거다)

KITTIX.yaml -> 데이터셋의 카메라 intrinsic parameter.

sequences -> dataset 경로. 데이터셋의 어떤 sequence 폴더를 이용할 것인지 입력하면 된다.

![image](https://user-images.githubusercontent.com/67038853/113674456-2bbea380-96f5-11eb-8e01-b5c0e03ed3d6.png)

 ![image](https://user-images.githubusercontent.com/67038853/113674371-15184c80-96f5-11eb-83af-60b75c2b653b.png)





## ROS Examples



### bashrc 설정

```
gedit ~/.bashrc
export ROS_PACKAGE_PATH=${ROS_PACKAGE_PATH}:PATH/ORB_SLAM2/Examples/ROS //맨 아래에 복붙
//ex
export ROS_PACKAGE_PATH=${ROS_PACKAGE_PATH}:/home/gleefe1995/orb_slam2/ORB_SLAM2/Examples/ROS
//저장한 뒤
source ~/.bashrc
```

여기서 PATH에 본인 ORB_SLAM2 경로를 넣어준다. 터미널에서 ORB_SLAM2로 이동해준다음 pwd를 치면 경로가 나온다. 

![image](https://user-images.githubusercontent.com/67038853/113671398-3f680b00-96f1-11eb-8183-7de6ffa5cf3f.png)

![image](https://user-images.githubusercontent.com/67038853/113671452-5149ae00-96f1-11eb-968c-7e85d2d65a11.png)

추가하고 저장한 뒤 source ~/.bashrc

### build_ros.sh

```
//ORB_SLAM2 폴더에서
chmod +x build_ros.sh
./build_ros.sh
```

### 에러

/usr/bin/ld: CMakeFiles/RGBD.dir//src/usrros_rgbd.cc.o:/ binundefined/ ldreference:  toCMakeFiles /symbolStereo.dir /'src_ZN5boost6system15system_categoryEv/'ros_stereo.cc.o
:/ usrundefined/ libreference/ x86_64to- linuxsymbol- gnu'/_ZN5boost6system15system_categoryEvlibboost_system.so':
 /errorusr /addinglib /symbolsx86_64:- linuxDSO- gnumissing/ libboost_system.so:from  errorcommand  addingline 
symbols: DSO missing from command line
collect2: error: ld returned 1 exit status
collect2: error: ld returned 1 exit status
CMakeFiles/RGBD.dir/build.make:227: recipe for target '../RGBD' failed
make[2]: *** [../RGBD] Error 1
CMakeFiles/Stereo.dir/build.make:227: recipe for target '../Stereo' failed
make[2]: *** [../Stereo] Error 1
CMakeFiles/Makefile2:67: recipe for target 'CMakeFiles/RGBD.dir/all' failed
make[1]: *** [CMakeFiles/RGBD.dir/all] Error 2
make[1]: *** Waiting for unfinished jobs....
CMakeFiles/Makefile2:104: recipe for target 'CMakeFiles/Stereo.dir/all' failed
make[1]: *** [CMakeFiles/Stereo.dir/all] Error 2
Makefile:129: recipe for target 'all' failed
make: *** [all] Error 2

빌드 도중 이러한 에러가 났는데 찾아보니

![image](https://user-images.githubusercontent.com/67038853/113671753-c0bf9d80-96f1-11eb-9918-c4c457e2f20d.png)

ROS/ORB_SLAM2 폴더에 있는 CMakelist에 위와 같이 -lboost_system을 추가해주면 된다.

![image](https://user-images.githubusercontent.com/67038853/113671882-eba9f180-96f1-11eb-933e-cae2659e1a24.png)

빌드성공



### Running Monocular Node

```
rosrun ORB_SLAM2 Mono PATH_TO_VOCABULARY PATH_TO_SETTINGS_FILE
//SETTINGS_FILE에는 본인 카메라 캘리브레이션한 yaml파일을 넣으면 된다.
// ORB SLAM에서 쓰는 yaml파일 형식에 맞게 변환해주어야한다.
rosrun ORB_SLAM2 Mono Vocabulary/ORBvoc.txt Examples/Monocular/test.yaml
```

