# Deploy with Nginx Web Server in Ubuntu Server

## Deploy Front End Sites

- Cài đặt Nginx
``` bash
sudo apt update
sudo apt install nginx
```
- Mở port tường lửa
``` bash
sudo ufw allow app list
sudo ufw allow 'Nginx HTTP'
sudo ufw enable
```
- Cấu hình nginx vào systemd
``` bash
sudo systemctl restart nginx
sudo systemctl status nginx
```

- Một vài tùy chọn trong systemctl cần nhớ
    + start
    + reload
    + disable
    + enable
    + status

- Đổi IP Address trên Ubuntu Server
``` bash
sudo nano /etc/netlan/{file}.yaml
sudo netplan --debug apply
```
[00-installer-config.yaml](00-installer-config.yaml)

- Cho phép giao thức ICMP hoạt động trên tường lửa
``` bash
sudo nano /etc/ufw/before.rules
# Add
-A ufw-before-output -p icmp --icmp-type echo-request -j ACCEPT
-A ufw-before-input -p icmp --icmp-type echo-request -j ACCEPT
-A ufw-before-forward -p icmp --icmp-type echo-request -j ACCEPT

sudo ufw reload
```
[before.rules](before.rules)


- Tạo thư mục tại đường dẫn /var/www/html
- Cho code web đã được build vào thư mục /var/www/html/...
- Cấp quyền của user cho thư mục đó.
``` bash
sudo chown -R nghialv:nghialv /var/www/html/...
sudo chmod -R 755 /var/www/html/...
```

- Chỉnh sửa file nginx.conf tại thư mục /etc/nginx
[nginx.conf](nginx.conf)

- Tạo file cấu hình cho trang web tại thư mục /etc/nginx/sites-available
[file-config-in-sites-available](front-end-site-config-file)
- Link file sites-available đó đến file sites-enabled
``` bash
sudo ln -s /etc/nginx/sites-available/... /etc/nginx/sites-enabled/
```
- Chạy thử xem có bị lỗi gì không
``` bash 
sudo nginx -t
```
- Khởi động lại nginx
``` bash
sudo systemctl restart nginx
```

- Check xem $port có đang chạy không?
``` bash
sudo netstat -nlp | grep $PORT
```

## Deploy Flask API in Nginx with uwsgi

- Tạo thư mục chứa code
- Cài các package cần thiết
``` bash
sudo apt update
sudo apt install python3-pip python3-dev build-essential libssl-dev libffi-dev python3-setuptools
```
- Tạo môi trường, khởi động môi trường và cài cái requirements.txt
``` bash
sudo apt install python3-venv
python3 -m venv myvenv
source myven/bin/activate
```
- Cài đặt Flask Application
``` bash
pip3 install wheel
pip3 install uwsgi flask
pip3 install -r requirements.txt
```
- Mở port trên trường lửa.
``` bash
sudo ufw allow 'Nginx Full'
```
- Check xem API chạy local có ổn không
``` bash
python3 app.py
```
- Cấu hình uWSGI
    + Tạo file [wsgi.py](wsgi.py)
    + Test uWSGI Serving
    ``` bash
    uwsgi --socket 0000:5000 --protocol=http -w wsgi:app
    ```
    + Mở Web browser để kiểm tra url: http://your_ip:5000

- Tạo file .ini trong cùng thư mục. [example.init](fileinit.ini)
- Tạo file .service trong /etc/systemd/system/ [example.init](fileservice.service)
- Khởi động service đã tạo
``` bash
sudo systemctl start fileservice
sudo systemctl enable fileservice
sudo systemctl status fileservice
```
- Tạo file cấu hình nginx trong /etc/nginx/sites-available/ [flask-api-config-file](flask-api-config-file)

- Tạo liên kết đến sites-enabled
``` bash
sudo ln -s /etc/nginx/sites-available/... /etc/nginx/sites-enabled
```
- Kiểm tra nginx
``` bash
sudo nginx -t
```
- Khởi động lại nginx
``` bash
sudo systemctl restart nginx
```
- Kiểm tra log file
    + Error logs tại đường dẫn /var/log/nginx/error.log
    + Access logs tại đường dẫn /var/log/nginx/access.log
    + Process logs
    ``` bash
    sudo journalctl -u nginx
    ```
    + Flask app's uWSGI logs
    ``` bash
    sudo journalctl -u myservice
    ```

## Kiểm tra chịu tải với [Artillery.io](https://artillery.io/)

``` bash
npm install -g artillery
```
- Code để test chịu tải ở folder test_perfomance
- Sửa file credential.csv, config.yaml
- Run artillery
``` bash
artillery run config.yaml
```
- Một vài option của artillery

| Option            | Description                                    |
|-------------------|------------------------------------------------|
| --output, -o      | Write a JSON report to a file                  |
| --environment, -e | Run the test using the specified environment   |
| --config, -c      | Load config section from another file          |
| --overrides       | Override values in the test script dynimically |
|--target, -t       | Set or override target URL for the test        |
|--payload, p       | Set path to an external payload file           |
|--variables, -v    | Set scenario variables dynamically             |
|--insecure, -k     | Turn off TLS verification. Should not be used in production|
|--quiet, -q        | Run in “quiet” mode                           |

- Reference: https://artillery.io/docs/guides/guides/command-line.html