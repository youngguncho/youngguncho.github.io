---
layout: post
title:  "Setup Graphic driver and Cuda"
date:   2017-04-16 22:07:58 +0900
tags: [Ubuntu]
description: >
    Ubuntu 환경에서 그래픽 카드와 쿠다 설치에 대한 포스트

---

Ubuntu를 처음 설치하면 가장 문제가 되는 부분이 바로 그래픽 드라이버를 잡는 부분이다. [Nvidia](http://www.nvidia.co.kr/Download/index.aspx?lang=kr)에서 그래픽 드라이버를 받아서 설치해도 되지만 최신 그래픽 카드에서는 무한 로그인!! 문제가 발생하는 경우가 잦다. 따라서 이번 포스트에서는 Ubuntu 16.04 + GTX 1060, GTX 1070(labtop)에서 확인한 설치방법에 대해 소개한다.

## apt-get을 이용해 Nvidia 최신 드라이버 설치하기
Ubuntu repository에 기본으로 추가되어 있는 nvidia 그래픽 드라이버는 이전 버전의 드라이버 이기 때문에 최신 드라이버 ppa를 추가해야 한다. ppa를 추가한 후 apt-get update를 해서 ppa update를 하고 autoinstall로 그래픽 드라이버 및 다른 ubuntu driver들도!!! 함께 설치할 수 있다.

1. 유선 또는 무선 랜 설정
    그래픽 드라이버를 받기 위해 유선 또는 무선랜을 연결한다.

2. ctrl+alt+F1으로 터미널 모드 들어가기 (ctrl+alt+f7로 다시 gui모드 들어갈 수도 있다.)
    터미널 모드로 들어가서 lightdm을 꺼서 그래픽 드라이버 설치를 위한 준비를 한다.
    ```
    sudo service lightdm stop
    ```

3. 최신 그래픽 드라이버 ppa 추가 및 그래픽 드라이버 설치
    ```
    sudo add-apt-repository ppa:graphics-drivers/ppa
    sudo apt-get update
    sudo ubuntu-drivers autoinstall
    sudo reboot
    ```
4. (option) 무한 로그인 문제를 예방하기 위한 조치
    * .Xauthority 파일 설정: 터미널에서 다음 명령어 입력
     ```mv ~/.Xauthority ~/.Xauthority.bak```
    * Nouveau 설정:         ```/etc/modprobe.d/blacklist.conf``` 에서 맨 아래에 다음 줄을 추가
        ```black nouveau```

5. 그래픽 드라이버 설치 확인: Terminal 창에서 ```nvidia-settings```를 입력하면 nvidia X server setting가 나오는데 System information에서 드라이버 버전을 확인할 수 있다.
    <img align="middle" src="/image/posts/Ubuntu/graphic-driver/nvidia-settings.png" width="70%">
