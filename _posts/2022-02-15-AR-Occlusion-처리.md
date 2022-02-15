---
clayout: post
title: "AR Occlusion 처리"
date: 2022-02-15 23:38:30 -0400
categories: SLAM
use_math: true
---



# Point Cloud map을 이용한 AR Occlusion 처리

2021-2학기 때 수강했던 최신컴퓨터공학이란 과목에서 AR에 관련된 프로젝트를 진행했던 내용이다.

AR에서 객체를 증강시키는데, 이 객체가 실제 어떤 구조물이나 움직이는 사람 등에 의해 자연스럽게 가려지는 모션이 사용자가 볼 때 몰입감을 더 해줄수 있다는 것을 알았고, ISMAR 2021에서 이와 관련된 논문인 Long-Range Augmented Reality with Dynamic Occlusion Rendering을 찾았고 리딩을 진행하였다. 이 논문 자체는 Dynamic Occlusion에 대한 처리에 중점을 맞춰서 Depth를 매 프레임마다 prediction하는 것이 아니라 object detection 후 tracking을 하여 mask를 이용해서 depth 정보를 갖고와서 처리속도를 개선하는 내용이다. 이 논문에서는 static한 구조물에 의해 가려지는 것은 아래 그림의 (b)와 같이 Terrain model을 viewpoint에 projection 시켜서 

![Screenshot from 2022-02-15 23-50-21](https://user-images.githubusercontent.com/67038853/154086381-632a9ac8-ae0b-4923-b442-df4959d5bc28.png)

depth 정보를 가져왔다고 하였다. 이 논문에서 이용한 Terrain model이 어떤 형태인지는 파악하지 못해서 outdoor에서 주차되어 있는 차들의 depth도 나오는 것이 신기하였지만 대충 3d point cloud map을 이용해서 viewpoint로 projection 시켜서 depth를 얻는구나 하고 이해를 하였다.

그래서 일단 static한 구조물에 의해 객체가 가려지는 것을 구현하기 위해서 ORB SLAM으로 RGBD 카메라로 3d point cloud map 생성 -> mono camera로 sparse point cloud map을 미리 만든 후 localization만 수행하면서 AR 객체 증강 -> AR 객체의 각 vertex와 depth를 비교하여 가려지는 부분은 제외하여 객체 증강으로 계획을 세웠다.

mono camera로 localization하는 부분을 더 robust한 learning based feature인 GCNv2 SLAM을 이용하여 더 정확한 카메라위치를 얻고자 하였다.

#### 3d point cloud map 생성

![image](https://user-images.githubusercontent.com/67038853/154089955-78b94f33-059f-4d3d-bcc1-1a7f7d4b8f42.png)

https://taeyoung96.github.io/slamtip/ORBSLAM2withPCL/ 를 참고하여 depthmap도 제공하는 tum dataset으로 orb slam을 이용해서 dense point cloud map을 얻었다. 

![image](https://user-images.githubusercontent.com/67038853/154090395-4c4c3443-3afd-43f7-a197-b44916c547cd.png)

가까이서 보면 보다시피 퀄리티가 좋진 않은것 같은데, 어떤 viewpoint로 projection 시켰을 때 depth가 dense하게 나올지가 의문이었다. 3d point의 개수는 약 48만개였다.

#### Localization using sparse point cloud map

![image](https://user-images.githubusercontent.com/67038853/154090808-dc731b4b-a236-4f1e-9c86-92997ade91c5.png)

GCNv2 SLAM : https://github.com/jiexiong2016/GCNv2_SLAM

map binary로 저장 기능 : https://github.com/Alkaid-Benetnash/ORB_SLAM2

GCNv2 SLAM에 map을 .bin 파일로 저장 후, map을 한번 만든 이후에는 map을 load하여 localization 기능만 수행하도록 하였다. 저장된 keyframe 위치와 비교해본 결과 거의 비슷한 경로를 그리면서 위치를 추정하였다. 

#### Dense point cloud map projection

![image](https://user-images.githubusercontent.com/67038853/154093326-29b5d5c2-2221-4d07-a785-19d08d508f1b.png)

오른쪽 사진이 위에서 생성했던 dense point cloud map을 현재 camera 위치로 projection시킨 모습이고, 아래 사진은 증강시킨 AR 객체가 아직 occlusion 처리가 되지 않고 있는 모습이다. point cloud map을 projection시킬 때 depth를 비교해서 제일 작은 depth가 표시되도록 하였고, 거리에 따라 색을 나타내었다. 보다시피 depth map이 sparse하여 제대로 depth 비교가 힘들어 보였다. 나는 AR 객체의 각 vertex와 비교할 것이기 때문에 depth map이 dense해야 더 정확한 occlusion 처리가 가능할 것으로 보였다. 그래서 이 sparse한 depth map을 interpolation을 통해서 dense하게 만들어주려고 했는데, depth map을 보다보니 sparse한 point 사이 더 거리가 먼 point들이 있었다. 그래서 제일 가까운 포인트들과 interpolation을 할 경우 정확한 depth가 나오지 않겠다고 판단을 했고, minimum값을 추출하는 filter를 씌우기로 했다. 이 filter의 크기는 11*11로, window 내에서 가장 작은 depth 값을 뽑는 filter이다. 이 때 filter 내에 depth가 없는 경우가 있어 0을 뽑을수 있으므로 0이 아닌 최소값을 추출하는 filter를 씌웠다.

#### 결과물

![ar_occlusion](assets/ar_occlusion.gif)

cube의 각 vertex를 depth map과 비교한 뒤에 occlusion 처리를 해주었다. 물론 보기에 정확해보이진 않았는데, 다른 더 복잡한 객체를 증강시켰을 때는 어떤 모습을 보일지 확인해보고 싶었는데 이번 프로젝트에서 그러한 점을 확인해보지 못한 것이 아쉬웠다.