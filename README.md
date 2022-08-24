# RaspberryPi4_Yolov3

## HardWare Spec
  - Raspberry Pi 4 + SSD
  - Raspberry Pi 3B+
  - Samsung SC-FD110B FHD (Camera)
  - Raspberry pi Camera module v.2.1 (element14)
 
## Server Envrionment
  - Rasbian (Raspberry Pi 4)
  - Ubuntu LTS 20.04.0

## Step 1. Install Opencv

  - raspberrypi upgrade
  
  ```
  sudo apt-get -y update && sudo apt-get -y  upgrade
  sudo apt-get -y install python3-dev
  ```
  
  - install python package
  
  ```
  pip3 install opencv-python
  ```
  
  - install opencv library
  
  ```
  pip3 install opencv-contrib-python==4.1.0.25
  ```
  
  ! if error occurs in this process folllowing below
  
  ```
  pip3 install opencv-contrib-python 
sudo apt-get install -y libatlas-base-dev libhdf5-dev libhdf5-serial-dev libatlas-base-dev libjasper-dev  libqtgui4  libqt4-test
  ```
  
  or
  
  ```
  sudo apt-get install libatlas-base-dev
  ```
  
## Test openCV

- save the image lenna.png
  
  ![image](https://user-images.githubusercontent.com/81907470/178634964-577db4ff-8659-42d0-9dc2-ef3af2441ddb.png)

- make test.py

  ```python
  import cv2

img = cv2.imread("<path>/lenna.png")
cv2.imshow("Test",img)

img_canny = cv2.Canny(img, 50, 150)
cv2.imshow("Test img Edge", img_canny)

cv2.waitKey(0)
cv2.destroyAllWindows()
```
  
 - run test.py
  
  ```python
  python3 test.py
  ```

# Step 2. install Yolo 

## install darknet

- clone darkent

```
git clone https://github.com/AlexeyAB/darknet
```

- set config

```
sudo vi Makefile
```

```
# OpenCV = 0 to 1
OpenCV = 1
```
( if you wanna use GPU or others, change to 1)

## make compile

```
make
```

## Test yolov3

- download pre-trained weights file

```
wget https://pjreddle.com/media/files/yolov3.weights
```

- check sample image file

```
#darknet/data
ls data
```

  - test

```
./darknet detect cfg/yolov3.cfg ./yolov3.weights data/dog.jpg
```

# Step3. prepare Dataset 

  ## Install Yolomark
  
    - clone github
  
```
  git clone https://github.com/AlexeyAB/Yolo_mark
```
  
    - compile yolomark
  
```
cmake .
make
chmod u+x ./linux_mark.sh
./linux_mark.sh
```
  
  ![image](https://user-images.githubusercontent.com/81907470/178654309-7a3d0b27-7e98-42b1-b0bb-965644588ab8.png)

  ## Delete sample image
  
  ```
  cd yolo_mark/x64/Release/data/img
  rm *
  ```
  
  ## Dataset images download
  
  - https://kdx.kr/data/view/31083
  
  ![image](https://user-images.githubusercontent.com/81907470/181196780-49eed608-61c2-41cc-9d6b-1fc00ae99f4a.png)


  ## How to use Yolo_Mark
  
  https://github.com/AlexeyAB/Yolo_mark


# Step 4. Custom weights

  ```
  cd darknet
  mkdir custom
  ```
  
  ## copy dataset & train.txt
  
  ```
   mkdir x64
   cd x64
   mkdir Release
   cd Release
   mkdir data
   cd data
   
   cp ../../Yolo_mark/x64/Realse/data/img -r 
   cp ../../Yolo_mark/x64/Realse/data/train.txt 
  ```
  
  ## make configuration
  
  ```
   cd cfg
   cp yolov3.cfg ./custom/custom.cfg
   cd ../../custom
    vim custom.cfg
  ```
  
  ```
[convolutional]
size=1
stride=1
pad=1
filters=60
activation=linear


[yolo]
mask = 0,1,2
anchors = 10,13,  16,30,  33,23,  30,61,  62,45,  59,119,  116,90,  156,198,  373,326
classes=15
num=9
jitter=.3
ignore_thresh = .7
truth_thresh = 1
random=1
  ```
  
  edit classes in [yolo] , filters in [convolutional] above yolo
  ! there are 3 parts in the file
  
    classes = number of class in dataset
    filters = (classes + 5 ) * 3
     
 ## make custom.names
 
 - path: ~/darknet/custom
 
 ```
 vim custom.names
 ```
 
 - insert class name
 
 ```
  (ex)
  dog
  data
 ```

  ## make custom.data 
  
  - path: ~/darknet/custom
  
  ```
  vim custom.data
  ```
  
  - if you apply valid insert valid info as well
  -  insert number of classes, train set path, (validation set path), names path, weight path
  
  ```
  classes= 15
train  = custom/train.txt
# valid  = custom/validation.txt
names = custom/custom.names
backup = backup/
  ```

## download pretrained model

- path: ~/darknet/custom

```
wget https://pjreddie.com/media/files/darknet53.conv.74
```

# Execute trainning

```
./darknet detector train custom/custom.data custom/custom_yolov3.cfg darknet53.conv.74 | tee backup/train.log
```

 ! using tee to make logfile

![image](https://user-images.githubusercontent.com/81907470/179916489-c106697a-4e25-4a36-8627-6e9404f7cadc.png)


# Step 5. Place a equipment

![image](https://user-images.githubusercontent.com/81907470/179916788-9a53d2ba-b089-4216-8dec-3255b3e2bcea.png)


![image](https://user-images.githubusercontent.com/81907470/179916731-9c95f7e2-f0a2-4dd2-aa39-aef27fbba8cb.png)

# Step 6. Install motion

## Set Configuration

```
sudo raspi-config
```

- Interfacing Option > Camera > Enable

![image](https://user-images.githubusercontent.com/81907470/180117311-c25841b1-85ce-4470-adfb-0bdd43c5d7a9.png)

## Check Camera

```
vcgencmd get_camera
```

![image](https://user-images.githubusercontent.com/81907470/180117435-1dbdc886-27cb-4723-8ee3-1ebdae4dd118.png)

## Install motion & setting

- Install motion
```
sudo apt-get install motion
```

- setting

```
sudo nano /etc/default/motion
```

### due to external connection allowed

- demon off > demon on : background operation 
- webcam_localhost on > off
- stream_port [port number]

### motion setting

- emulate_motion off (default : off)
- threshold 1500 (motion sensitivity)

## Execute motion

```
sudo motion
```

- http://[rasp ip]: stream_port

# Step 7. Shell Script & Crontab

## make Shell file

- path: home/pi/camera

```
vim camera_config.sh
sudo chmod 777 camera_config.sh
```

- create directory as date
- capture a pic & save as datetime

```

#! /bin/bash

DATE1=$(date +"%Y-%m-%d")
CreateDIR=/home/pi/camera/images/$DATE1

if [ ! -d $CreateDIR ]; then
        mkdir $CreateDIR
fi

DATE2=$(date +"%Y-%m-%d_%H%M")
raspistill -q 100 -t 1000 -w 800 -h 600 -o  /home/pi/camera/images/$DATE1/$DATE2.jpg

```

## Edit crontab

- take a pic every 5 min

```
# every 5 min take pic

*/5 * * * * /home/pi/camera/camera_config.sh

# * * * * * /home/pi/camera/camera_config.sh

```

## Result

![image](https://user-images.githubusercontent.com/81907470/180920763-c5e2b897-91a9-433c-a923-bc9d7214e2e4.png)

# Step 8. Tesseract OCR

  ## find out x,y postion
  
  - path: /home/pi/camera
  
  - take sample image
  ```
  raspistill -q 100 -t 1000 -w 800 -h 600 -o set_index.jph
  ```
  - Make set_index.py
  
  ```  
import cv2

img = cv2.imread("./set_index.jpg")

x_pos, y_pos, width, height = cv2.selectROI("location", img, False)
print("x, y :: ", x_pos, y_pos)
print("width, height :: ", width, height)

cv2.destroyAllWindows()
  ```
  
  ```
  python3 set_index.py
  ```
  
  - copy x & y , width & height 
  
  ![image](https://user-images.githubusercontent.com/81907470/180930117-16e1fc8e-a32c-46f6-ae54-d5443673d6ee.png)

  ![image](https://user-images.githubusercontent.com/81907470/180930133-5b0014a6-2483-4fd2-bec2-ce8f4d990762.png)

  
## Install tesseract

```
sudo apt install tesseract-ocr tesseract-ocr-kor

sudo apt install tesseract-ocr-script-hang tesseract-ocr-script-hang-vert

sudo pip3 install pytesseract

pip3 install pytesseract
  
```
  
## Make ocr.py
  
- path: home/pi/camera

```
vim ocr.py
```
  
```python
from pytesseract import *
import re
import numpy as np
import cv2

img = cv2.imread("./set_index.jpg")

## !!!! paste here !!!
x=174;y=147;w=563;h=87;
  
roi = img[y:y+h, x:x+w]
img2 = roi.copy()

text = pytesseract.image_to_string(img2, config='--psm 6')

print(text)
```
  
## Result 1
  
```
  python3 ocr.py
```
  
  ![image](https://user-images.githubusercontent.com/81907470/180930349-8e9e0f06-1827-4c83-acc9-7b0c483c630f.png)

  
  ![image](https://user-images.githubusercontent.com/81907470/180930319-bd28ce8d-9bf2-4879-b908-09648a1d7e96.png)

## Result 2 (jupyter)
  
  - Added preprocessing
     
    - convert gray, filter2D, dilation
  
  ![image](https://user-images.githubusercontent.com/81907470/180964617-4a4d566b-4b17-4fd4-b9fb-4ce2c1731e80.png)
![image](https://user-images.githubusercontent.com/81907470/180964647-2944a8a5-f1bd-4e88-9efc-7d028ec3ad4f.png)
![image](https://user-images.githubusercontent.com/81907470/180964700-7294074b-896a-4be8-ab1f-76278cfe26e9.png)

  - full code
  
  ```
  import pytesseract
import re
import numpy as np
import cv2
from matplotlib import pyplot as plt

#설치 위치 설정
pytesseract.pytesseract.tesseract_cmd=r"C:\Program Files\Tesseract-OCR\tesseract.exe"
  
  img = cv2.imread("./set_index.jpg", cv2.IMREAD_GRAYSCALE)

x=174;y=147;w=563;h=87;
roi = img[y:y+h, x:x+w]
img2 = roi.copy()

plt.imshow(img2, cmap="gray"), plt.axis("off") # 이미지 출력
plt.show()
result = pytesseract.image_to_string(img2, config='--psm 7 --oem 3 -c tessedit_char_whitelist=0123456789')

print(result)
  
  #kernel 만들기

kernel = np.array([[0,-1,0],[-1,5,-1],[0,-1,0]])

# 이미지를 선명하게 한다

img2 = cv2.filter2D(img2, -1, kernel)
plt.imshow(img2, cmap="gray"), plt.axis("off") # 이미지 출력
plt.show()

result = pytesseract.image_to_string(img2, config='--psm 7 --oem 3 -c tessedit_char_whitelist=0123456789')

print(result)
  
  def dilation(image):
    kernel = np.ones((3,3),np.uint8)
    img = cv2.dilate(image, kernel, iterations=1)
    plt.imshow(img,  cmap="gray"), plt.axis("off")
    plt.show()
    result = pytesseract.image_to_string(img, config='--psm 7 --oem 3 -c tessedit_char_whitelist=0123456789')
    return result

result = dilation(img2)
print(result)

  ```
  
## Result 3 (Logging system) !!final!!

- path: /home/pi/camera 

### 1. Edit camera_config.sh
 
  - 1.1 take a image naming by datetime (.jpg file)
  - 1.2 make directory naming by date & save jpg
  - 1.2 run ocr.py file for ocr

### 2. Edit ocr.py

  - 2.1 read img & preprocessing
  - 2.2 execute OCR
  - 2.3 run excel.py for save value

### 3. Make excel.py

  - 3.1 read excel
  - 3.2 save result (date, value) value to excel

### 4. crontab -e

  - 4.1 run shell script every 5 min
  
  

- camera_config.sh

```

#! /bin/bash

DATE1=$(date +"%Y-%m-%d")
CreateDIR=/home/pi/camera/images/$DATE1

if [ ! -d $CreateDIR ]; then
        mkdir $CreateDIR
fi

DATE2=$(date +"%Y-%m-%d_%H%M")
raspistill -q 100 -t 1000 -w 800 -h 600 -o  /home/pi/camera/images/$DATE1/$DATE2.jpg

path="/home/pi/camera/images/$DATE1/$DATE2.jpg"
echo "$path" > /home/pi/camera/flag.txt

sleep 3

python3 /home/pi/camera/ocr.py

```

- ocr.py

```
import pytesseract
import re
import numpy as np
import cv2
import excel
import os

# 이미지 경로 불러들이기

file = open("/home/pi/camera/flag.txt")
image_path = file.readlines()[0].replace("\n","")
print(image_path)
# 이미지 자르기

img = cv2.imread(image_path, cv2.IMREAD_GRAYSCALE)

x=174;y=147;w=563;h=87;
roi = img[y:y+h, x:x+w]
img2 = roi.copy()

result = pytesseract.image_to_string(img2, config='--psm 7 --oem 3 -c tessedit_char_whitelist=0123456789')
print(result)

# kernel 만들기

kernel = np.array([[0,-1,0],[-1,5,-1],[0,-1,0]])

# 이미지를 선명하게 한다

img3 = cv2.filter2D(img2, -1, kernel)

result = pytesseract.image_to_string(img3, config='--psm 7 --oem 3 -c tessedit_char_whitelist=0123456789')
print(result)

# dilation 주기

kernel = np.ones((3,3),np.uint8)
img4 = cv2.dilate(img3, kernel, iterations=1)

result = pytesseract.image_to_string(img4, config='--psm 7 --oem 3 -c tessedit_char_whitelist=0123456789')
arr = result.split('\n')[0:-1]
result = '\n'.join(arr)

print(type(result))
print("결과값 :: " + result)

with open('text_result.txt', mode ='w') as file:
    file.write(result)

# 리스트 생성 및 excel,excel() 호출

date = image_path.split('/')[-1].split('.')[0]
print("date 출력::"+date)
result_list = [date, result]
excel.excel(result_list)

```

- excel.py

```
import pandas as pd

def excel(result_list):
    date = result_list[0]
    value = 0
    if (result_list[1] == ""):
        value = 0;
    else: value = result_list[1];

    # 파일명
    filename = "result.xlsx"
    #엑셀파일 읽기
    df_excel = pd.read_excel(filename, engine = "openpyxl", header = 0);
    print(df_excel)
    #결과값 리스트
    result_df = pd.DataFrame({'날짜':[date],'결과값':[value]})
    print(result_df)
    #엑셀파일에 값 추가
    df_excel = pd.concat([df_excel, result_df])
    #df_excel.drop(['Unnamed: 0'], axis = 1 ,inplace = True)
    #엑셀파일에 저장
    print(df_excel)
    df_excel.to_excel('result.xlsx',index=False)

```
  
![image](https://user-images.githubusercontent.com/81907470/181185716-72e643ec-699f-4488-9dcf-9ae7479cecda.png)

![image](https://user-images.githubusercontent.com/81907470/181185824-a3c0151c-bc8b-4c64-ad11-8cd6e1c1b87e.png)

  
### Sammary
  
  - need to intergrate [result 2] with [result 3]
  - meaningful ocr result is only return in jupyter envrionment
  - need to set juypyter encrionmet in ubuntu server of find other ways
  

# Testing Board

## 1. test with pre-trained weights + gas meter images

- used image

![gasmeter](https://user-images.githubusercontent.com/81907470/178641063-a31ae2b6-9ad0-4d91-b0d2-5185982da92d.jpg)

- result 

![image](https://user-images.githubusercontent.com/81907470/178641002-638e4b60-9007-42a5-a23d-150a8e12b8d5.png)

   ### summary

  - 사전학습된 weights를 이용할 경우 가스 계측기가 parking meter 로 분류됨
  - weights를 직접 학습시켜야할 필요성
  - 한국데이터거래소(https://kdx.kr/data/view/31083)의 계량기 이미지를 이용한 학습계획
  
## 2. setup equipment and test

- take a pic by motion

![image](https://user-images.githubusercontent.com/81907470/179916995-021f4169-80cd-4c47-b7a9-8707e3a4389a.png)

- take a pic by raspistill

![image](https://user-images.githubusercontent.com/81907470/179917014-0af51855-3869-4cf2-a359-d37b3618a2d0.png)

  ### summary
    
    카메라 설치 후 사진의 화질이 낮아 OCR 문자인식에 장애가 발생될 것이라고 예상됨, 화질부분의 개선이 필요
    ( 예상 원인: 에어컨 시래기에서 발생하는 진동에 의한 화질저하, 카메라 성능 등)
    >> fixed 카메라 초점 문제 였음
  
## 3. Custom trainning result
  
    ![image](https://user-images.githubusercontent.com/81907470/181186500-532e92d7-4bce-47f7-bf68-21e0ef0d9a74.png)

    ![image](https://user-images.githubusercontent.com/81907470/181186600-0be52f34-8e17-442a-b194-d1c98f4529c9.png)
  
- 학습에 많은 시간이 걸림  약 4일) > 중단시키고 학습된 weight를 테스트 해보았으나 유의미한 결과가 도출되지않음
- google colab을 이용하여 weight를 학습시킨 후 weight 파일을 서버로 옮겨 테스트를 진행할 예정

# Fix Issues 

## [Open CV] Import Error: numpy.core.multiarray failed to import

  - check numpy version
  
  ```
  pip3 install -U numpy
  ```
  
## [Open CV] Package opencv was not found in the pkg-config search path
  
  https://stackoverflow.com/questions/15320267/package-opencv-was-not-found-in-the-pkg-config-search-path

## [NextCloud] Acess through untrusted domain

- IP Issues (changed ip)

```
sudo nano /var/www/html/nextcloud/config/config.php
```

```
<?php
$CONFIG = array (
  'instanceid' => 'oczal9md4xhg',
  'passwordsalt' => '4MeO9A2YvAbTnK4IVjn4SFjN0yUx/k',
  'secret' => 'FPu2r143kX+l11wUdVYXy+0mMXtqivYK4AWmkGCARKhN62KW',
  'trusted_domains' =>
  array (
    0 => '[!!! change here !!!]',
  ),
  'datadirectory' => '/var/www/html/nextcloud/data',
  'dbtype' => 'mysql',
  'version' => '24.0.3.2',
  'overwrite.cli.url' => 'http://[!!! change here !!!]',
  'dbname' => 'nextcloud',
  'dbhost' => 'localhost',
  'dbport' => '',
  'dbtableprefix' => 'oc_',
  'mysql.utf8mb4' => true,
  'dbuser' => 'nextcloud',
  'dbpassword' => '****',
    'installed' => true,
);

```

## [Camera Module] Camera v2 from element14 initialization issues
 
https://community.element14.com/products/raspberry-pi/f/forum/46625/raspiberry-pi-camera-module-error---mmal-mmal_vc_component_enable-failed-to-enable-component-enospc

![image](https://user-images.githubusercontent.com/81907470/180176931-b5a8b33f-f8d5-4fe9-a4d2-fbf4fc4291db.png)

## [Pandas to Excel] Unamed column auto create

- Before convert & save to excel , delete 'Unnamed : 0'
- #df_excel.drop(['Unnamed: 0'], axis = 1 ,inplace = True)

```
    # 파일명
    filename = "result.xlsx"
    #엑셀파일 읽기
    df_excel = pd.read_excel(filename, engine = "openpyxl", header = 0);
    print(df_excel)
    #결과값 리스트
    result_df = pd.DataFrame({'날짜':[date],'결과값':[value]})
    print(result_df)
    #엑셀파일에 값 추가
    df_excel = pd.concat([df_excel, result_df])
    #df_excel.drop(['Unnamed: 0'], axis = 1 ,inplace = True)
    #엑셀파일에 저장
    print(df_excel)
    df_excel.to_excel('result.xlsx',index=False)

```


