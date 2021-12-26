---
layout: post
title: "[HoloLens2] 프로젝트 생성"
image: /assets/img/hololens2_logo.jpeg
category: devlog
tags: hololens2
---

* toc
{:toc}


## Setup

HoloLens2 를 개발하기 위해 설치해야되는 프로그램 목록은 [공식사이트](https://docs.microsoft.com/ko-kr/windows/mixed-reality/develop/install-the-tools)에서 확인 가능하다.

Visual Studio 의 워크로드는 Visual Studio Installer 를 통해 설치 및 확인 가능하다.

추가적으로 Unity Hub 과 적절한 버전의 Unity를 설치하고,

마지막으로 [Mixed Reality Feature Tool](https://docs.microsoft.com/ko-kr/windows/mixed-reality/develop/unity/welcome-to-mr-feature-tool)을 설치해주면 된다.

## Activate Developer Mode

HoloLens2 장치에서 *Settings > Update & Security > For developers* 에서 Developer Mode 를 활성화 하는 것이 가능하다. (이미지는 [이 곳](https://docs.microsoft.com/en-us/windows/mixed-reality/develop/advanced-concepts/using-the-windows-device-portal)에서 확인)

> Developer Mode 가 활성화가 안되는 경우는 장치가 Home Edition 이거나 ([참조](https://docs.microsoft.com/en-us/windows/mixed-reality/develop/advanced-concepts/using-the-windows-device-portal)) 장치의 문제일 수도 있다. 개인적으로는 장치에 문제가 있었는데, Widows 를 업데이트 하여 해결하였다.

## Device Portal

앱 내부 데이터나 장치의 상세 정보를 살펴봐야 할 경우 Device Portal 을 사용하면 된다. 자세한 설명은 [공식사이트](https://docs.microsoft.com/ko-kr/windows/mixed-reality/develop/advanced-concepts/using-the-windows-device-portal)에 자세히 설명되어 있으니 참고하면 된다.

## Start Project

아래는 Microsoft 홈페이지에 있는 [튜토리얼](https://docs.microsoft.com/ko-kr/learn/paths/beginner-hololens-2-tutorials/)을 따라한 내용을 정리해 둔 것이다.

**(1) "3D" 템플릿을 선택후 프로젝트 생성**

![img](/assets/img/Hololens2_start_project/01.png)

**(2) 프로젝트 생성 후 *File > Build Settings* 클릭**

**(3) "Universal Windows Platform" 선택 후 Target Device, Architecture 수정 후 Switch Platform 클릭**

![img](/assets/img/Hololens2_start_project/02.png)

**(4) 별도로 설치한 "Mixed Reality Feature Tool" 프로그램 실행**

![img](/assets/img/Hololens2_start_project/03.png)

**(5) 생성한 Unity 프로젝트 선택 후 Discover Features 클릭**

![img](/assets/img/Hololens2_start_project/04.png)

**(6) "Mixed Reality Toolkit" 에서 Mixed Reality Toolkit Foundation 과 "Platform Support" 에서 Mixed Reality OpenXR Plugin 선택 후 Get Features 클릭**

![img](/assets/img/Hololens2_start_project/05.png)

**(7) 이후로 Exit 버튼이 나올때 까지 파란색 활성화된 버튼을 순서대로 클릭**

![img](/assets/img/Hololens2_start_project/06.png)

![img](/assets/img/Hololens2_start_project/07.png)

![img](/assets/img/Hololens2_start_project/08.png)

**(8) Unity 프로그램으로 돌아와서 다음과 같은 팝업창이 뜨면 Yes 클릭**

![img](/assets/img/Hololens2_start_project/09.png)

**(9) Unity 프로그램이 다시 켜지면 아래와 같은 팝업창을 확인 가능. 팝업창에서 Unity OpenXR Plugin 클릭 후 Show XR Plug-in Management Settings 클릭**

![img](/assets/img/Hololens2_start_project/10.png)

![img](/assets/img/Hololens2_start_project/11.png)

**(10) XR Plug-in Management 에서 OpenXR, Microsoft HoloLens feature group 선택**

![img](/assets/img/Hololens2_start_project/12.png)

**(11) OpenXR 에서 "Depth Submission Mode" 를 Depth 16 Bit 로 선택, 나머지는 적절히 선택**

![img](/assets/img/Hololens2_start_project/13.png)

**(12) 다시 팝업창으로 넘어와서 next 를 클릭하다가 Apply 클릭**

![img](/assets/img/Hololens2_start_project/14.png)

![img](/assets/img/Hololens2_start_project/15.png)

**(13) *Mixed Reality > Toolkit > Add to Scene and Configure* 클릭하여 새로운 화면 추가**

![img](/assets/img/Hololens2_start_project/16.png)

## Run Application

작성한 프로젝트를 HoloLens2 장치에서 실행하는 과정은 다음과 같다.

먼저 *File > Build Settings* 를 클릭한 후, Build and Run On 을 원하는 build 방식으로 선택해준다. 그리고 build 버튼을 눌러 원하는 위치에 Application 을 build 해준다.

![img](/assets/img/Hololens2_start_project/17.png)

buid 가 완료되면, `.sln` 포맷의 파일이 보일건데 Visual Studio 2019 를 사용해 열어주면된다.

![img](/assets/img/Hololens2_start_project/18.png)

마지막으로 배포 모드는 Release, 프로세서는 장치의 프로세스 (ARM or ARM64), 장치는 디바이스 를 선택 후 `Ctrl + F5` 를 눌러 디버깅하지 않고 시작을 해주면 된다.

![img](/assets/img/Hololens2_start_project/19.png)

###  Connect USB

처음 PC 와 HoloLens2 장치를 USB로 연결하기 위해서는 개발자 설정에서 직접 추가해 주어야한다. 먼저 USB 를 연결하고, Visual Studio 2019 를 사용하여 Application 을 실행시키면 PIN을 입력해라는 문구가 뜨는데, 이 때 *Settings > Update & Security > For developers* 에서 USB의 pair 버튼을 눌러 PIN을 얻고, 해당 번호를 입려해주면 된다. 

