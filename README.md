# RaspberryPi4_Yolov3

## HardWare Spec
  - Raspberry Pi 4 + SSD
  - Samsung SC-FD110B FHD (Camera)
## Server Envrionment
  - Rasbian (Raspberry Pi 4)

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


# Testing Board

## 1. test with pre-trained weights + gas meter images

- used image

![gasmeter](https://user-images.githubusercontent.com/81907470/178641063-a31ae2b6-9ad0-4d91-b0d2-5185982da92d.jpg)

- result 

![image](https://user-images.githubusercontent.com/81907470/178641002-638e4b60-9007-42a5-a23d-150a8e12b8d5.png)

## 2. setup equipment and test

- take a pic by motion

![image](https://user-images.githubusercontent.com/81907470/179916995-021f4169-80cd-4c47-b7a9-8707e3a4389a.png)

- take a pic by raspistill

![image](https://user-images.githubusercontent.com/81907470/179917014-0af51855-3869-4cf2-a359-d37b3618a2d0.png)

 ### summary

  - 사전학습된 weights를 이용할 경우 가스 계측기가 parking meter 로 분류됨
  - weights를 직접 학습시켜야할 필요성
  - 한국데이터거래소(https://kdx.kr/data/view/31083)의 계량기 이미지를 이용한 학습계획

# Fix Issues 

## Import Error: numpy.core.multiarray failed to import

  - check numpy version
  
  ```
  pip3 install -U
  ```
  
## Package opencv was not found in the pkg-config search path
  
  https://stackoverflow.com/questions/15320267/package-opencv-was-not-found-in-the-pkg-config-search-path
 
