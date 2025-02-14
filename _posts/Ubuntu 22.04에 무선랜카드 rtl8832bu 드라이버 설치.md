---
layout: post
title: "Ubuntu 22.04에 무선랜카드 rtl8832bu 드라이버 설치 "
date: 2025-02-14 00:01:30 -0400
categories: tips
use_math: true
---


#### 환경
- Ubuntu 22.04, iptime AX2000U 무선랜카드

기존에 window11에서 AX2000U 무선랜카드를 쓰고 있다가 Ubuntu 22.04를 

듀얼부팅으로 깔게 되면서 AX2000U가 linux를 지원하는지 알아봤는데,

window만 지원한다고 되어있었다..

찾아보다가 iptime ax900ua 무선랜카드가 linux를 지원한다고 해서 구입한 후에

iptime 홈페이지에서 드라이버까지 설치해봤지만, No WIFI Adapater만 떴다...

고객 센터 전화해봤지만 칩셋회사에서 배포하는 드라이버라 불량이 아니라면 환불해주겠다는 말 뿐..

ax900ua는 환불하고 마지막 도전으로 ax2000u 으로 도전해봤는데 성공!

나중에 안까먹게 기록해둬야지

#### 드라이버 설치 방법

ax2000u는 쿠팡 제품 소개글을 보면 RTL8823BU 칩셋이라고 되어있는데,

윈도우에서 iptime 검색기(?)로 보니 RTL8832BU 칩셋이라고 되어있었다.

그래서 ubuntu에 rtl8832bu 드라이버를 설치를 하면 된다는 결론
(왜 소개글이 잘못되어있을까..)

'''
lsusb # 먼저 무선랜카드의 device id 확인 (0bda:####)
sudo vim /lib/udev/rules.d/40-usb_modeswitch.rules
'''

맨 아랫줄에

ATTR{idVendor}=="0bda", ATTR{idProduct}=="####", RUN+="usb_modeswitch '/%k'"

추가

'''
sudo apt-get update
sudo apt-get install make gcc linux-headers-$(uname -r) build-essential git
'''

한번 해주고

'''

git clone https://github.com/lwfinger/rtl8852bu
cd rtl8852bu
make
sudo make install

'''

해주면 끝

한 3일동안 삽질한 것 같다..

그래도 원래 있던 무선랜카드로 되어서 돈아껴서 다행 ㅋㅋㅋ