# RaspberryPi4_Yolov3

## HardWare Spec
  -Raspberry Pi 4 + SSD
  -Samsung SC-FD110B FHD (Camera)
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

  - save the image 
  
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

  
