# Topic3-LEMP

## Executive Summary

- LEMP là một bộ công nghệ phổ biến được thiết kế để vận hành và quản lý các ứng dụng web động. Hệ thống này đảm nhiệm toàn bộ quy trình fullstack
- Nginx và Apache:
  - Apache xử lý kết nối bằng một tiến trình riêng biệt gây tốn tài nguyên, Nginx sử dụng mô hình hướng sự kiện event-driven. Cơ chế này cho phép xử lí hàng trăm nghìn kết nối đồng thời với lượng tiêu thị RAM và CPU cực thấp.
  - Ngoài ra là máy chủ xử lí các yêu cầu HTTP và file tĩnh, có thể dùng làm load balance, proxy ngược.
  -
- L (Linux): Hệ điều hành gốc (Bạn đang dùng Ubuntu 22.04). Nó quản lý tài nguyên phần cứng, mạng và bộ nhớ.

- E (Nginx): Đọc là Engine-X. Đây là trái tim của mô hình. Nginx đóng vai trò là Web Server, tiếp nhận các yêu cầu (Request) từ trình duyệt.

- M (MySQL): Hệ quản trị cơ sở dữ liệu. Nơi lưu trữ thông tin có cấu trúc của WordPress và Laravel.

- P (PHP): Ngôn ngữ xử lý logic. Trong mô hình LEMP, nó chạy dưới dạng một dịch vụ riêng biệt gọi là PHP-FPM (FastCGI Process Manager).

- Luồng xử lý dữ liệu:
  - Client gửi Request. Request này đến cổng 443 (https) của VPS
  - Nginx tiếp nhận: Nginx xem file cấu hình trong sites-enaboled để biết thư mục gốc nằm ở đâu
  - Nếu là File tĩnh (Ảnh, CSS, JS): Nginx tự mình trả về luôn cho trình duyệt.
  - Nếu là File động (.php): Nginx không tự đọc được. Nó sẽ chuyển (Proxy) yêu cầu này sang PHP-FPM thông qua một "đường ống" gọi là Unix Socket
  - PHP-FPM nhận code, thực thi logic, kết nối với MySQL nếu cần lấy dữ liệu
  - PHP đóng gói thành HTML, gửi ngược lại cho Nginx. Nginx gửi về trình duyệt.

- Nếu có nhiều backend thì làm sao Nginx biết Request đó của be nào.
  - Ví dụ trường hợp một domain có nhiều backend

```bash
example.com/api → NodeJS
example.com/admin → Spring
example.com/ → React

# Nginx config
server {
    listen 80;
    server_name example.com;

    location /api/ {
        proxy_pass http://localhost:3000;
    }

    location /admin/ {
        proxy_pass http://localhost:8080;
    }

    location / {
        proxy_pass http://localhost:5173;
    }
}
```

- Nginx dựa vào path (URL) để định tuyến request đến các backend tương ứng. Trong trường hợp này, Nginx đóng vai trò reverse proxy và có thể được xem như một API Gateway đơn giản trong mô hình microservices, giúp phân tách và điều phối request đến nhiều service khác nhau.

- Ngoài ra, Nginx còn có thể thực hiện load balancing, sử dụng các thuật toán như round-robin để phân phối request đều giữa các backend, giúp tăng khả năng chịu tải và tính sẵn sàng của hệ thống.

- Quy trình kết nối giữa client và nginx nếu có https
  - client gửi request
  - Nginx gửi certificate (có public key)
  - Client kiểm tra có hợp lệ có đúng domain có bi fake không
  - Client: tạo một khóa tạm thời symmetric key
  - Mã hóa bằng public key và chỉ có private key của server mới mở được
  - Sau đó chỉ dùng symmetric encryption (AES)

## Table of Contents

1. Xây dựng mô hình LEMP: Cài dặt Linux, Nginx, MySQL và PHP 8.1

2. Cài đặt web: Triển khai một trang web WordPress và một trang Laravel mặc định

3. Cấu hình SSL: Cài dặt chứng chỉ cho 2 domain (Sử dụng Certbot)

4. Tài khoản FTP: Tạo tài khoản FTP để quản lý file

5. Cấu hình MySQL: Thiết lập MySQL, bật truy cập bằng xa bằng user root

# LEMP

![alt text](image-2.png)

- Kiểm tra phiên bản Ubuntu của VPS

1. Cài NGINX

```bash
sudo apt update
sudo apt install nginx -y
```

![alt text](image.png)

- Kiểm tra phiên bản và trạng thái hoạt động

2. Cài MySQL

```bash
sudo apt install mysql-server -y
sudo mysql_secure_installation
```

![alt text](image-1.png)

- Kiêm tra phiên bản và trạng thái hoạt động của MySQL

![alt text](image-3.png)

- Kiểm tra xem đăng nhập được bảng điều khiển MySQl không

3. Cài đặt PHP8.1-fpm

```bash
sudo apt install php8.1-fpm php-mysql
sudo systemctl status php8.1-fpm
```

![alt text](image-4.png)

- Kiểm tra trạng thái

4. Cấu hình Nginx để xử dụng PHP

```bash
# Cho WordPress
sudo mkdir -p /var/www/wp.vietduc.vietnix.tech
# Cho Laravel
sudo mkdir -p /var/www/laravel.vietduc.vietnix.tech

# Gán quyền cho user hiện tại có thể upload file
sudo chown -R $USER:$USER /var/www/wp.vietduc.vietnix.tech
sudo chown -R $USER:$USER /var/www/laravel.vietduc.vietnix.tech
```

- Cấu hình cho WordPress:

```bash
sudo nano /etc/nginx/sites-available/wp.vietduc.vietnix.tech
```

```bash
server {
    listen 80;
    server_name wp.vietduc.vietnix.tech;
    root /var/www/wp.vietduc.vietnix.tech;

    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

- Giải thích file:
  - Lắng nghe HTTP (Port 80)
  - root: thư muc gốc nơi chứ code

- Cài đặt wordpress

```bash
CREATE DATABASE wordpress_db;
EXIT;

# Tải và giải nén wordpress
cd /var/www/wp.vietduc.vietnix.tech
# Xóa file index.html cũ
rm index.html
# Tải WordPress
wget https://wordpress.org/latest.tar.gz
# Giải nén
tar -xvf latest.tar.gz
# Di chuyển code ra thư mục gốc và xóa thư mục thừa
mv wordpress/* .
rm -rf wordpress latest.tar.gz

# Cấp quyền
sudo chown -R www-data:www-data /var/www/wp.vietduc.vietnix.tech
sudo chmod -R 755 /var/www/wp.vietduc.vietnix.techduoc
```

![alt text](image-15.png)

- Password WordPress: gDuoAMGwvxsFz2GAng

- Kết quả:

![alt text](image-16.png)

- Cấu hình cho Laravel:

```bash
sudo nano /etc/nginx/sites-available/laravel.vietduc.vietnix.tech
```

```bash
server {
    listen 80;
    server_name laravel.vietduc.vietnix.tech;
    root /var/www/laravel.vietduc.vietnix.tech/public;

    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
    }
}
```

- Cài đặt laravel
  Bước 1: Cài đặt các thư viện PHP bổ trợ

```bash
sudo apt update
sudo apt install php8.1-mbstring php8.1-xml php8.1-bcmath php8.1-curl php8.1-zip -y
```

Bước 2: Dùng Composer tạo dự án Laravel

```bash
cd /var/www

composer create-project laravel/laravel laravel.vietduc.vietnix.tech
```

Bước 3: Cấp quyền cho Nginx

```bash
sudo chown -R www-data:www-data /var/www/laravel.vietduc.vietnix.tech/storage
sudo chown -R www-data:www-data /var/www/laravel.vietduc.vietnix.tech/bootstrap/cache
sudo chmod -R 775 /var/www/laravel.vietduc.vietnix.tech/storage
```

Bước 4: Kiểm tra

![alt text](image-7.png)

- Kích hoạt và kiểm tra:

```bash
# Tạo link liên kết để kích hoạt
sudo ln -s /etc/nginx/sites-available/wp.vietduc.vietnix.tech /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/laravel.vietduc.vietnix.tech /etc/nginx/sites-enabled/

# Hủy bỏ site mặc định
sudo unlink /etc/nginx/sites-enabled/default

sudo nginx -t

sudo systemctl reload nginx
```

5. Cấu hình SSL sử dụng Certbot

```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx
```

- Sau đó nhập email để nhận thông báo gia hạn
- Chọn 1: laravel.vietduc.vietnix.tech để bảo mật tên miền này
- Sau khi xong Certbot sẻ tự sửa config Nginx, tự cài SSL

![alt text](image-10.png)

- Kết quả:

![alt text](image-8.png)

![alt text](image-9.png)

6. Tạo tài khoản FTP

```bash
# Cai dat dich vu
sudo apt update

sudo apt install vsftpd -y

# Tao User va cap quyen
sudo adduser ftpuser

sudo usermod -aG www-data ftpuser

sudo chown -R www-data:www-data /var/www/
sudo chmod -R 775 /var/www/

# Cau hinh cho phep ghi file
sudo nano /etc/vsftpd.conf

# Bo # truoc dong write_enable=YES
sudo systemctl restart vsftpd
```

- Kiểm tra hoạt động:

![alt text](image-11.png)

![alt text](image-12.png)

7. Bật remote MySQL cho user Root

```bash
# Tìm dòng bind-address = 127.0.0.1 và sửa thành:

bind-address = 0.0.0.0
```

![alt text](image-13.png)

- Cấp quyền Remote cho Root

```bash
sudo mysql -u root -p

## Cập nhật user root để chấp nhận kết nối từ xa
# Cho phép dùng mật khẩu đơn giản
SET GLOBAL validate_password.policy=0;
SET GLOBAL validate_password.length=4;

CREATE USER IF NOT EXISTS 'root'@'%' IDENTIFIED BY '1111';

GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;

FLUSH PRIVILEGES;

# Đổi mật khẩu
ALTER USER 'root'@'%' IDENTIFIED BY 'Vietnix123.';

# Đổi cho user root chạy nội bộ (localhost)
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Vietnix443.';

FLUSH PRIVILEGES;
# Kiem tra ở máy khác
mysql -h 221.132.21.144 -u root -p
```

- Kết quả khi remote vào mysql:

![alt text](image-14.png)

8. Cài đặt PHPMySQLAdmin

```bash
sudo nano /etc/nginx/sites-available/phpmyadmin_ip

server {
    listen 80;
    server_name 221.132.21.144; # Điền IP VPS của bạn vào đây

    root /var/www/html;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }

    # Cấu hình xử lý PHP
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
    }

    # Bảo mật: Không cho phép xem các file ẩn .ht
    location ~ /\.ht {
        deny all;
    }
}


sudo mkdir -p /var/www/html

sudo ln -s /usr/share/phpmyadmin /var/www/html/phpmyadmin

# Kích hoạt site mới
sudo ln -s /etc/nginx/sites-available/phpmyadmin_ip /etc/nginx/sites-enabled/

sudo nginx -t
sudo systemctl reload nginx


sudo chmod -R 755 /usr/share/phpmyadmin
sudo chown -R www-data:www-data /usr/share/phpmyadmin
```

pass MyPHPAdmin :Admin234.
user: root

9. 1 path ở web wp.vietduc.vietnix.tech làm sao khi truy cập vào wp.vietduc.vietnix.tech/api/ hiển thị ra 1 file có nội dung bất kì

```bash
# Tạo file và nội dung cho API
# Tạo thư mục api nằm trong thư mục web của WordPress
sudo mkdir -p /var/www/wp.vietduc.vietnix.tech/api

# Tạo một file index.html với nội dung bất kỳ
echo "Data on API" | sudo tee /var/www/wp.vietduc.vietnix.tech/api/index.html


# Cau hinh nginx de nhan dien path
sudo nano /etc/nginx/sites-available/wp.vietduc.vietnix.tech

# Them doan nay vao
# Cấu hình riêng cho đường dẫn /api/
    location /api/ {
        alias /var/www/wp.vietduc.vietnix.tech/api/;
        index index.html;
        try_files $uri $uri/ =404;
    }

# Kiem tra
sudo nginx -t
sudo systemctl reload nginx
```
