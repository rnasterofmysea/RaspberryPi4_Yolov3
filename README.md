# RaspberryPi4_Yolov3

## HardWare Spec
  - Raspberry Pi 4 + SSD
  - Samsung SC-FD110B FHD (Camera)
  - Raspberry pi Camera module
  - 
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


# Next Cloud

## Package upgrade & update & install

```
sudo apt upgrade
sudo apt update
sudo apt install nginx mariadb-server php php-fpm php-mysql php-zip php-common php-zip php-xml php-mbstring php-gd php-curl -y
```

## Check packages

- check raspberry pi IP address

```
ifconfig
```

- connect [IP address]

![image](https://user-images.githubusercontent.com/81907470/180170222-c88f7b95-04ea-48c1-a796-c577e782a17e.png)

## Database setting

- Enter mariadb as root

```
sudo mariadb -u root
```
- Create database

```
MariaDB [(none)]> CREATE DATABASE nextcloud;
```

- Create new account

```
MariaDB [(none)]> CREATE USER 'nextcloud'@'localhost' IDENTIFIED BY '1234';
```

- Grant setting

```
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud'@'localhost';
```

```
quit
```

## Install NextCloud

- Download zip file

```
wget https://download.nextcloud.com/server/releases/latest.zip
```

- unzip zip file
```
sudo rm /var/www/html/*
sudo unzip ./latest.zip -d /var/www/html/
sudo chown -R www-data:www-data /var/www/html
(auth setting)
```

## Nginx setting

```
sudo nano /etc/nginx/sites-enabled/default
```

- delete code > copy & paste 

```
upstream php-handler {
    server unix:/var/run/php/php7.4-fpm.sock;
}

server {
    listen 80;
    listen [::]:80;
    server_name localhost;

    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Robots-Tag none;
    add_header X-Download-Options noopen;
    add_header X-Permitted-Cross-Domain-Policies none;
    add_header Referrer-Policy no-referrer;
    fastcgi_hide_header X-Powered-By;

    root /var/www/html/nextcloud;

    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }

    location = /.well-known/carddav {
      return 301 $scheme://$host:$server_port/remote.php/dav;
    }
    location = /.well-known/caldav {
      return 301 $scheme://$host:$server_port/remote.php/dav;
    }

    client_max_body_size 512M;
    fastcgi_buffers 64 4K;

    gzip on;
    gzip_vary on;
    gzip_comp_level 4;
    gzip_min_length 256;
    gzip_proxied expired no-cache no-store private no_last_modified no_etag auth;
    gzip_types application/atom+xml application/javascript application/json application/ld+json application/manifest+json application/rss+xml applicaEnter this intion/vnd.geo+json application/vnd.ms-fontobject application/x-font-ttf application/x-web-app-manifest+json application/xhtml+xml application/xml font/opentype image/bmp image/svg+xml image/x-icon text/cache-manifest text/css text/plain text/vcard text/vnd.rim.location.xloc text/vtt text/x-component text/x-cross-domain-policy;

    location / {
        rewrite ^ /index.php;
    }

    location ~ ^\/(?:build|tests|config|lib|3rdparty|templates|data)\/ {
        deny all;
    }
    location ~ ^\/(?:\.|autotest|occ|issue|indie|db_|console) {
        deny all;
    }

    location ~ ^\/(?:index|remote|public|cron|core\/ajax\/update|status|ocs\/v[12]|updater\/.+|oc[ms]-provider\/.+)\.php(?:$|\/) {
        fastcgi_split_path_info ^(.+?\.php)(\/.*|)$;
        set $path_info $fastcgi_path_info;
        try_files $fastcgi_script_name =404;
        include fastcgi_params;
        fastcgi_read_timeout 1800;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $path_info;
        fastcgi_param modHeadersAvailable true;
        fastcgi_param front_controller_active true;
        fastcgi_pass php-handler;
        fastcgi_intercept_errors on;
        fastcgi_request_buffering off;
    }

    location ~ ^\/(?:updater|oc[ms]-provider)(?:$|\/) {
        try_files $uri/ =404;
        index index.php;
    }

    location ~ \.(?:css|js|woff2?|svg|gif|map)$ {
        try_files $uri /index.php$request_uri;
        add_header Cache-Control "public, max-age=15778463";
        add_header X-Content-Type-Options nosniff;
        add_header X-XSS-Protection "1; mode=block";
        add_header X-Robots-Tag none;
        add_header X-Download-Options noopen;
        add_header X-Permitted-Cross-Domain-Policies none;
        add_header Referrer-Policy no-referrer;

        access_log off;
    }

    location ~ \.(?:png|html|ttf|ico|jpg|jpeg|bcmap)$ {
        try_files $uri /index.php$request_uri;
        access_log off;
    }
}
```
- Reload nginx

```
sudo nginx -s reload
```

## Check Nginx

![image](https://user-images.githubusercontent.com/81907470/180174568-7d555cab-84fe-49bd-bbe8-c73274b3a416.png)


![image](https://user-images.githubusercontent.com/81907470/180174494-4cccb544-cab2-4ac0-9c5a-c27463bd7c63.png)

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
  
# Fix Issues 

## [Open CV] Import Error: numpy.core.multiarray failed to import

  - check numpy version
  
  ```
  pip3 install -U
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

