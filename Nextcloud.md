
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
![image](https://user-images.githubusercontent.com/90185805/190652682-ec5dc013-bc73-495f-bce6-d3426a8153ee.png)


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
- php 버전 확인 
```
php -v
```
![image](https://user-images.githubusercontent.com/90185805/190646402-1d7eec5a-a14f-414a-afb6-c3340b751e7b.png)

```
sudo nano /etc/nginx/sites-enabled/default
```

- delete code > copy & paste


```nginx
upstream php-handler {
    server unix:/var/run/php/php8.1-fpm.sock;
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
- nginx 상태 확인
```
service nginx status
```

![image](https://user-images.githubusercontent.com/90185805/190648530-77738125-a43b-4e92-93c6-722ec7cb4959.png)

- http://도메인 주소 또는 IP
```
http://127.0.0.1/

http://localhost/
```
![image](https://user-images.githubusercontent.com/90185805/190655561-e73bfca0-f13d-4df3-af0e-256a41fa1b59.png)


## nextcloud와 로컬PC 연동

- 로컬PC의 nextlcoud 실행 후 프로필 => 설정

![image](https://user-images.githubusercontent.com/90185805/190653934-994dbb79-900b-4b92-8e32-20721733394f.png)

- 동기화 폴더 연결 추가

![image](https://user-images.githubusercontent.com/90185805/190653975-a5ef5c95-e46c-4ec9-b1c7-b88fb4f2f62c.png)

- 동기화 할 로컬 폴더 선택

![image](https://user-images.githubusercontent.com/90185805/190654016-aee03819-620a-4780-9479-572898a50cf7.png)

- nextcloud의 원격 대상 폴더 선택

![image](https://user-images.githubusercontent.com/90185805/190654057-7d2ac610-3dfd-4cf8-a4fa-ee41415b1b72.png)

- 동기화하지 않을 원격 폴더 선택

![image](https://user-images.githubusercontent.com/90185805/190654203-3a0baaea-4502-4845-b4c5-57abb891ac7b.png)

- nextcloud와 로컬 PC 연동 완료

![image](https://user-images.githubusercontent.com/90185805/190654247-7c4fd149-ad28-443c-aa3d-2889125da56e.png)

- 동기화 확인

![image](https://user-images.githubusercontent.com/90185805/190654278-7219624b-e447-4ac3-b4d9-4027ba5296db.png)