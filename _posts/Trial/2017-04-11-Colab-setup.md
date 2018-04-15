---
layout: post
title:  "구글 Colaboratory, Colab에 Pytorch 셋업하기!"
date:   2018-04-11 02:07:58 +0900
tags: [trial]
description: >
  구글 Colaboratory, Colab에 Pytorch 셋업하기!
---
오랜만에 작성하는 포스트입니다. 이번 글은 구글 Colaboratory, 짧게 줄여 Colab에 Pytorch 및 google drive를 셋업하는 방법에 대한 설명입니다. [Deep Learning Turkey](https://medium.com/deep-learning-turkey/google-colab-free-gpu-tutorial-e113627b9f5d)를 참고하였으며 몇 가지 테스트 후 작성하였습니다.


## Google, Colab!
구글 Colaboratory, Colab은 무엇인고 하니, 우선 제공되는 설명을 보아하면 __텍스트, 코드, 코드 출력을 하나의 공동작업 문서로 통합해주는 데이터 분석 도구입니다.__ 하고 표현하고 있습니다. 쉽게 이해하자면, __구글에서 제공하는 무료 딥러닝 클라우드 서비스__ 라고 이해할 수 있습니다. 즉, __Google Colab__ 은 jupyter 노트북 기반으로 딥러닝 서버를 이용할 수 있는 서비스이죠. 아래 그림과 같은 __Tesla K80 GPU__ 를 이용할 수 있으며, Python 언어 기반으로 기본적으로 [Tensorflow](https://www.tensorflow.org/) 및 numpy와 같은 라이브러리가 셋업되어 있지만 추가적으로 [Keras](https://keras.io/)나 [Pytorch](http://pytorch.org/)등의 다른 라이브러리도 설치하여 사용할 수 있습니다.

<img align="middle" src="/image/posts/Trial/2018-04-11-Colab-setup/colab.png" width="70%">

위에서 언급한 것과 같이 Tensorflow는 기본적으로 설치되어 있기 때문에, 여기서는 Pytorch를 설치하는 방법과, 구글 드라이브를 연동해서 사용하는 방법에 대해서 살펴보겠습니다.

## Prepare Google Colab!
Google Colab을 사용하기 위한 기본적은 준비사항에 대해 먼저 살펴보겠습니다. Colab의 노트북 파일들은 구글 드라이브와 연동이 되기 때문에 먼저 드라이브에서 Colab을 사용할 준비를 합니다.

우선 그림과 같이 구글 드라이브에서 __colab__ 폴더와 __dataset__ 폴더를 만들어 줍니다. 여기서 __colab__ 폴더는 코드가, __dataset__ 폴더는 데이터 셋이 들어갈 폴더입니다.
<img align="middle" src="/image/posts/Trial/2018-04-11-Colab-setup/colab0.png" width="100%">


그럼 __colab__ 폴더로 가서 마우스 오른쪽 버튼을 클릭! __더보기->Colaboratory__ 를 추가줍니다. 만약 Colaboratory가 없다면 __연결할 앱 더보기__ 를 선택해서 Google Colaboratory를 연결합니다.
<img align="middle" src="/image/posts/Trial/2018-04-11-Colab-setup/colab0-1.png" width="100%">


## Setting Google Colab!

자, 제대로 Colaboratory가 추가되었다면 그림과 같이 ipython notebook (ipynb), 노트북이 추가됩니다. 처음에는 Untitled.ipynb로 생성되는데 원하는 파일명으로 바꿔줍니다.
<img align="middle" src="/image/posts/Trial/2018-04-11-Colab-setup/colab2.png" width="70%">


그리고 GPU 및 Python version 설정을 위해서 __수정->노트설정__ 에서 Python version 및 GPU 사용을 선택합니다.
<img align="middle" src="/image/posts/Trial/2018-04-11-Colab-setup/colab3.png" width="40%">


## Test Google Colab!
그렇다면 몇 가지 Python code를 실행해봅니다. 각 block은 __shift+enter__ 로 실행할 수 있습니다.
<img align="middle" src="/image/posts/Trial/2018-04-11-Colab-setup/colab4.png" width="100%">

아래와 같이 기본직인 Tensorflow 예제도 바로 해볼 수 있습니다.
<img align="middle" src="/image/posts/Trial/2018-04-11-Colab-setup/colab4-1.png" width="100%">

## Setting Pytorch & Google Drive
### Install Pytorch
Pytorch는 코드 블록에서 pip install을 통해 설치할 수 있습니다. 테스트 결과 기본 __pip3 install__ 로 Pytorch 0.3.1버전을 설치할 수 있는데, colab에서 제공하는 코드로 설치하면 0.3.0이 설치됩니다. (2018년 4월 11일 기준)
설치는 아래의 default 코드나 Colab에서 제공하는 코드 중 하나를 실행하시면 됩니다. 참고로 __!__ 를 붙이면 Python 콘솔이 아닌 터미널에서 명령이 실행되는 모드입니다.

```python
# Default Code
!pip3 install torch torchvision

# Code from Colab
from os import path
from wheel.pep425tags import get_abbr_impl, get_impl_ver, get_abi_tag
platform = '{}{}-{}'.format(get_abbr_impl(), get_impl_ver(), get_abi_tag())

accelerator = 'cu80' if path.exists('/opt/bin/nvidia-smi') else 'cpu'

!pip install -q http://download.pytorch.org/whl/{accelerator}/torch-0.3.0.post4-{platform}-linux_x86_64.whl torchvision
import torch
torch.__version__
```
<img align="middle" src="/image/posts/Trial/2018-04-11-Colab-setup/colab5.png" width="100%">


### Install Google Drive SDK
이제 서버에서 구글 드라이브를 연동에서 사용하기 위해 Google Drive SDK를 설치합니다. 설치 및 인증은 간단히 아래의 코드를 실행하면 됩니다. 코드를 실행하면 설치후에 인증 절차가 시작되는데, 제공되는 링크를 들어가서 복사한 인증 코드를 빈칸에 작성하면 인증이 완료 됩니다. 해보니 두번 인증을 해야하네요. 처음 verification code를 입력하고, 다시 나오는 링크에서 코드를 복사 후 한번 더 인증을 해야합니다. (아마 인증이 timeout되어 두번 된 것 일수도 있습니다.)

```python
!apt-get install -y -qq software-properties-common python-software-properties module-init-tools
!add-apt-repository -y ppa:alessandro-strada/ppa 2>&1 > /dev/null
!apt-get update -qq 2>&1 > /dev/null
!apt-get -y install -qq google-drive-ocamlfuse fuse
from google.colab import auth
auth.authenticate_user()
from oauth2client.client import GoogleCredentials
creds = GoogleCredentials.get_application_default()
import getpass
!google-drive-ocamlfuse -headless -id={creds.client_id} -secret={creds.client_secret} < /dev/null 2>&1 | grep URL
vcode = getpass.getpass()
!echo {vcode} | google-drive-ocamlfuse -headless -id={creds.client_id} -secret={creds.client_secret}
```

<img align="middle" src="/image/posts/Trial/2018-04-11-Colab-setup/colab6.png" width="100%">

### Mount Google Drive
다음은 Google Drive를 서버에 마운트합니다. 개인이 가지고 있는 구글 드라이브의 데이터 및 코드를 사용하기 위함이죠. 여기선 drive폴더에 구글 드라이브의 root가 연동되게 됩니다. !ls drive를 하면 가장 상위 디렉토리의 내용이 나와야 하죠. 제대로 연동이 되었다면 이제 위에서 만들어두었던 __colab__, __dataset__ 폴더에 drive/colab 그리고 drive/dataset으로 접근이 가능합니다!

```python
!ls
!mkdir -p drive
!ls
!google-drive-ocamlfuse drive
!ls drive
```

## Test Pytorch
자, 이제 Pytorch 설치 및 구글 드라이브 연동까지 완료되었으니, pytorch 코드를 돌려봅시다. 우선 __colab__ 폴더에 [Pytorch 코드](https://drive.google.com/open?id=1CJPK7RM657v0ECwLhpnzvV0NAxDsHImt)가 있다고 가정하겠습니다. [링크](https://drive.google.com/open?id=1CJPK7RM657v0ECwLhpnzvV0NAxDsHImt)에서 받으실 수 있습니다. 예제는 MNIST Convnet이며, drive/colab/main.py를 실행하면 확인하실 수 있습니다. 코드를 실행하면 우선 MNIST 데이터 다운로드를 시작하는데, 제가 지정한 drive/dataset으로 데이터가 저장되며 저장 후 트레이닝이 시작되는 것을 확인할 수 있습니다.

```
!python3 drive/colab/main.py
```

## 마무리
이번 포스트에서는 Google Colab을 셋업하는 방법을 살펴보았습니다. 제가 주로 사용하는 코드가 Pytorch이기 때문에 Pytorch기반의, 그리고 구글 드라이브를 연동해서 사용하는 방법을 알아보았습니다. 사실 제대로 사용하려면 더 다양한 기능을 알아야 하지만 우선 이번 포스트는 여기서 마무리하고, 중간중간 업데이트를 하도록 하겠습니다.
