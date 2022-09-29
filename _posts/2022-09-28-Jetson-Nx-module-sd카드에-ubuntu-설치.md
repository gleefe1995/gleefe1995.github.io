---
layout: post
title: "Jetson Nx module sd카드에 ubuntu 설치 "
date: 2022-09-28 00:01:30 -0400
categories: SLAM
use_math: true
---

![20220927_130823](https://user-images.githubusercontent.com/67038853/192928423-3d27e551-9a85-46a4-8de3-c3cfd519385c.jpg)

#### BSP 다운로드

https://www.avermedia.com/professional/download/en715#parentHorizontalTab4

![image](https://user-images.githubusercontent.com/67038853/192920231-372b9e21-e888-454b-b551-a8b5f8596c5e.png)

![image](https://user-images.githubusercontent.com/67038853/192919927-cc04e2f7-ec1a-49f8-8b92-95a104e608ee.png)

EN715 보드 선택 후 4.5.1 다운로드

zip 파일 압축해제 후에 

```bash
cat EN715-NX-R1.0.7.4.5.1.tar.gz.00* > EN715-NX-R1.0.7.4.5.1.tar.gz
sudo tar zxf EN715-NX-R1.0.7.4.5.1.tar.gz

cd JetPack_xxxxxxxxxxx/Linux_for_Tegra
```

#### SD카드 셋팅

1. SD카드 리더기에 Micro SD카드 넣고 PC에 꽂는다

2. 해당 SD 카드의 장치명 찾기

   ```bash
   sudo fdisk -l
   ```

   /dev/sdc 와 같은 장치명 찾는다.

3. 해당 SD카드를 GPT 파티션으로 변경. gdisk를 수행하고 커맨드 입력한다.

   ```bash
   $ sudo gdisk /dev/sdx
     "o" -> clear all current partition data
     "n" -> create new partition
     "1" -> partition number /dev/sdx1
     "40M"first sectors -> Press enter or "+32G" last sectors
     "Linux filsystem" -> using default type
     "c" -> partition's name "PARTLABEL"
     "w" -> write to disk and exit. 
   ```

   - `o`를 통해 장치 초기화
   - n을 통해 새로운 파티션 생성
     - 파티션 넘버는 `1` 지정
     - 처음 섹터는 `40M` 지정, 라스트 섹터는 `엔터`로 넘어감 ~~혹은 32G~~.
     - `+32G`를 입력할 경우 나중에 PARTUUID를 못얻는 경우가 있다. 그래서 웬만하면 `Enter`키로 넘어가자.
   - `c`를 통해 파티션의 이름을 `PARTLABEL`로 설정
   - `W`를 통해 쓰고 저장 후 종료

4. 파티션의 PARTUUID를 얻고 rootfs로 복사한다. 현재 경로는 JetPack_xxxxxxxxxxx/Linux_for_Tegra/ 

   ```
   $ sudo mkfs.ext4 /dev/sdx1
   $ sudo blkid /dev/sdx1
     > 여기서 나오는 PARTUUID를 잘 기억한다. (그냥 UUID 혹은 PTUUID와 다르다)
   $ sudo mount /dev/sdx1 /mnt
   // bootloader/l4t-rootfs-uuid.txt 파일은 만들어야함
   // 206ba185-02ac-433f-a995-0a4fda797f91 와 같이 숫자만 입력
   $ echo '{아까 기억한 PARTUUID}' > bootloader/l4t-rootfs-uuid.txt
   $ cd ./rootfs/
   $ sudo tar -cpf - * | ( cd /mnt/ ; sudo tar -xpf - )
   $ sudo umount /mnt
   ```

   blkid 명령어에서 PARTUUID가 안뜬다면 GPT 파티션 부분부터 다시 실행한다.

   #### 설치

   PC에 SD카드 리더기가 꽂혀있는 상태에서 진행한다. EN715 보드는 recovery mode로 부팅하여 usb로 pc와 연결한다. 인터넷선은 연결할 필요(X)

   ```bash
   $ sudo ./flash.sh jetson-xavier-nx-en715 mmcblk1p1
   ```

   external로 실행하니 UUID를 못찾는 문제가 있었다.

설치 완료 후에 모니터에 연결후 보드 전원을 키면 NVIDIA 화면이 뜨면서 켜진다. 기본 계정은 nvidia, 비밀번호도 nvidia. bootloader에서 순서는 바꿀필요 없었다.