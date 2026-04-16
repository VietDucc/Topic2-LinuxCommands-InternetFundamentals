# Topic 4: Reserve Proxy (Nginx + Apache)

## Config

1. Chuẩn bị môi trường và Giải nén Source Code

Tiến hành cài đặt các công cụ cần thiết và chuẩn bị cấu trúc thư mục lưu trữ cho các dự án WordPress và Laravel.
Bash

# Cập nhật hệ thống và cài đặt unzip

sudo apt update
sudo apt install unzip -y

Sử dụng giao thức FTP để đưa các tệp tin nguồn (.zip) và tệp tin cơ sở dữ liệu (.sql) vào thư mục /var/www/. 2. Deploy Source Code & Import Database

Thực hiện làm sạch môi trường cũ và cấu hình dữ liệu mới cho từng dự án.
A. Đối với dự án WordPress

    Xóa dữ liệu cũ: sudo rm -rf /var/www/wp.vietduc.vietnix.tech/*

    Giải nén mã nguồn: sudo unzip source_wp.zip -d /var/www/wp.vietduc.vietnix.tech/

    Khởi tạo và Import Database:

```bash
sudo mysql -u root -p -e "DROP DATABASE IF EXISTS wordpress_db; CREATE DATABASE wordpress_db;
sudo mysql -u root -p wordpress_db < /var/www/linhlt_wp_lodoz.sql
```

B. Đối với dự án Laravel

    Giải nén mã nguồn: sudo unzip laravel_source.zip -d /var/www/laravel.vietduc.vietnix.tech/

    Khởi tạo và Import Database:

```bash

sudo mysql -u root -p -e "DROP DATABASE IF EXISTS laravel_db; CREATE DATABASE laravel_db;"
sudo mysql -u root -p laravel_db < /var/www/linhlt_db.sql

    Cấu hình môi trường: Sao chép tệp .env và cập nhật các thông số DB_DATABASE, DB_USERNAME, DB_PASSWORD.
```

3. Chuyển đổi mô hình sang Reverse Proxy (Nginx + Apache)

Triển khai mô hình kết hợp:
A. Cấu hình Apache

Cập nhật ports.conf sang cổng 8080 và thiết lập các VirtualHost để xác định DocumentRoot chính xác cho từng dự án.
B. Cấu hình Nginx

Thiết lập Nginx làm điểm tiếp nhận SSL và sử dụng proxy_pass để đẩy traffic về Backend Apache. Đảm bảo cấu hình proxy_set_header X-Forwarded-Proto $scheme để truyền đúng giao thức (HTTP/HTTPS) vào mã nguồn phía sau. 4. Cấu hình mặc định và Bảo mật (Default Vhost)

Thiết lập Default Vhost để chặn các truy cập trực tiếp qua địa chỉ IP máy chủ. Tất cả yêu cầu không đi qua tên miền chính thức sẽ nhận phản hồi 403 Forbidden. 5. Hiệu chỉnh mã nguồn và Phân quyền

Để hệ thống vận hành ổn định sau khi cấu hình "phần thô", các bước sau đã được thực hiện:

    Phân quyền thư mục: Thiết lập quyền sở hữu cho www-data và phân quyền 755 cho thư mục web để tránh lỗi 403 Forbidden.

    Kết nối Database WordPress: Hiệu chỉnh wp-config.php để khớp với thông tin Database mới.

    Khởi tạo môi trường Laravel: - Phân quyền 775 cho thư mục storage và bootstrap/cache.

        Thực hiện xóa và tối ưu hóa bộ nhớ đệm qua lệnh php artisan config:cache và php artisan route:cache.

6. Kiểm tra và Nghiệm thu

Truy cập Kết quả mong đợi Trạng thái thực tế
https://wp.vietduc.vietnix.tech
https://laravel.vietduc.vietnix.tech
http://221.132.21.144

## Questions & Answer

1. Tại sao Nginx đứng trước Apache?

Nginx (Frontend): Xử lý các kết nối đồng thời cực tốt (Event-driven), gánh SSL và trả file tĩnh (ảnh, css) nhanh.

Apache (Backend): Xử lý file động (PHP) ổn định, hỗ trợ file .htaccess giúp WordPress và Laravel chạy mượt mà mà không cần cấu hình rewrite phức tạp trên Nginx.

- Kết quả deploy thành công Laravel

![alt text](image.png)

- Kết quả deploy wp

![alt text](image-1.png)

2. File cấu hình

![alt text](image-3.png)

- File cấu hình cua apache

![alt text](image-4.png)

- File cấu hình của nginx

```bash
root@training-vietduc:/etc/nginx/sites-available# cat laravel.vietduc.vietnix.tech
#### Laravel


# HTTP -> HTTP
server {
    listen 80;
    server_name laravel.vietduc.vietnix.tech;
    root /var/www/laravel.vietduc.vietnix.tech/public;


    location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff|woff2|ttf|svg)$ {
        try_files $uri =404;
        expires 30d;
        add_header Cache-Control "public, no-transform";
    }

    # Forward HTTP sang Apache port 8080 (HTTP)
    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto http;
    }
}
## Laravel
# HTTPS -> HTTPS
server {
    listen 443 ssl;
    server_name laravel.vietduc.vietnix.tech;
    root /var/www/laravel.vietduc.vietnix.tech/public;

    ssl_certificate /etc/letsencrypt/live/laravel.vietduc.vietnix.tech/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/laravel.vietduc.vietnix.tech/privkey.pem;

    location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff|woff2|ttf|svg)$ {
        try_files $uri =404;
        expires 30d;
        add_header Cache-Control "public, no-transform";
    }

    # Forward HTTPS sang Apache port 8443 (HTTPS)
location / {
    proxy_pass https://127.0.0.1:8443;

    proxy_ssl_server_name on;
    proxy_ssl_name $host;

    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto https;

    proxy_ssl_verify off;
}

    location ~ /\.ht {
        deny all;
    }
}
############################

### Wordpress
sudo nano /etc/nginx/sites-available/wp.vietduc.vietnix.tech
# --- HTTP -> HTTP ---
server {
    listen 80;
    server_name wp.vietduc.vietnix.tech;
    root /var/www/wp.vietduc.vietnix.tech;


    location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff|woff2|ttf|svg)$ {
        try_files $uri =404;
        expires 30d;
        add_header Cache-Control "public, no-transform";
    }

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto http;
    }
}

# --- HTTPS -> HTTPS ---
server {
    listen 443 ssl;
    server_name wp.vietduc.vietnix.tech;
    root /var/www/wp.vietduc.vietnix.tech;

    ssl_certificate /etc/letsencrypt/live/wp.vietduc.vietnix.tech/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/wp.vietduc.vietnix.tech/privkey.pem;

    location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff|woff2|ttf|svg)$ {
        try_files $uri =404;
        expires 30d;
        add_header Cache-Control "public, no-transform";
    }

    location / {
        proxy_pass https://127.0.0.1:8443;

        proxy_ssl_server_name on;
        proxy_ssl_name $host;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;

        proxy_ssl_verify off;
    }

    location ~ /\.ht {
        deny all;
    }
}

sudo nginx -t
sudo systemctl reload nginx
sudo systemctl restart apache2

###########################

## Bật module SSL

sudo a2enmod ssl
sudo systemctl restart apache2

############################

### File config Apache
root@training-vietduc:/etc/apache2/sites-available# cat vietduc_apps.conf
# Vhost for WordPress
<VirtualHost *:8080>
    ServerName wp.vietduc.vietnix.tech
    DocumentRoot /var/www/wp.vietduc.vietnix.tech
    <Directory /var/www/wp.vietduc.vietnix.tech>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>

# Vhost for Laravel
<VirtualHost *:8080>
    ServerName laravel.vietduc.vietnix.tech
    DocumentRoot /var/www/laravel.vietduc.vietnix.tech/public
    <Directory /var/www/laravel.vietduc.vietnix.tech/public>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>

# --- WORDPRESS ---
<VirtualHost *:8443>
    ServerName wp.vietduc.vietnix.tech
    DocumentRoot /var/www/wp.vietduc.vietnix.tech

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/wp.vietduc.vietnix.tech/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/wp.vietduc.vietnix.tech/privkey.pem


    <Directory /var/www/wp.vietduc.vietnix.tech>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>

# --- LARAVEL ---
<VirtualHost *:8443>
    ServerName laravel.vietduc.vietnix.tech
    DocumentRoot /var/www/laravel.vietduc.vietnix.tech/public

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/laravel.vietduc.vietnix.tech/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/laravel.vietduc.vietnix.tech/privkey.pem
    <Directory /var/www/laravel.vietduc.vietnix.tech/public>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
############################

```

![alt text](image-5.png)

- Dùng https ảnh không hiện lên:
  - Là do laravel cố gắng load các file tĩnh qua giao thức http

```bash
    nano /var/www/laravel.vietduc.vietnix.tech/app/Http/Middleware/TrustProxies.php
    protected $proxies = '*'; // Tin tưởng tất cả proxy
```

- Tại sao lúc đầu chỉ có https mà không có http vẫn truy cập được ảnh
  - Do có cấu hình Redirect 301, mọi yêu cầu từ trình duyệt bị ép sang https

- Tại sao chỉ khai báo public mà web vẫn chạy
  - Mặc dù Nginx chỉ "nhìn" vào thư mục /public, nhưng khi một yêu cầu (request) được gửi đến index.php, một cơ chế "kéo theo" sẽ xuất hiện:

  - Nginx gửi request cho Apache/PHP.

  - File public/index.php sẽ chạy lệnh require để kéo toàn bộ code từ thư mục vendor/, app/, routes/ (nằm ở thư mục cha) vào để xử lý.

![alt text](image-7.png)

- Trang web bị điều hướng tới linhlt do trong database có chứa URL này trong bảng option ( siteurl và home) tự thực hiện Redirect 301 sang linhlt. Để khắc phục:
  - Sửa trực tiếp trong database (bảng wp_options) để đổi siteurl và home thành wp.vietduc.vietnix.tech

- Auto HTTPS:
  - Để giá trị trong DB là https
  - Plugin Really Simple SSL: Tự động chuyển http sang https
  - Lệnh Redirect 301 là vĩnh viễn, nên xóa cache trình duyệt để tránh bị "dính" redirect cũ
- Cách khắc phục:
  - Dùng SQL UPDATE để sửa siteurl và home

  ![alt text](image-14.png)
  - Dùng lệnh define trong wp-config.php để "ghi đè" (override)

  ![alt text](image-13.png)
  - Nếu nhận từ cổng 8080 → gán http.
  - Nếu nhận từ cổng 8443 → gán https.
  - Cách này sẽ giúp WordPress "hiểu" được giao thức nào đang được sử dụng, từ đó tạo ra các URL chính xác trong phản hồi.

- Tóm tắt luồng:
  - User gõ wp.vietduc
  - Server nhận request chuyển cho PHP
  - WordPress (PHP): Đọc DB thấy siteurl = linhlt.
  - WordPress (PHP): Gửi phản hồi 301: "Chuyển sang linhlt".
  - Trình duyệt: Nhận lệnh 301 -> Tự động đổi thanh địa chỉ sang linhlt -> Load lại từ đầu với domain mới.
- Trong wordpress có Canonical Redicrect. Nhiệm vụ là đảm bảo trang web chỉ có 1 url chính

- Kết quả truy cập http

![alt text](image-9.png)

![alt text](image-10.png)

-

- Kết quả truy cập https:

![alt text](image-11.png)

![alt text](image-12.png)

- Nhận biết ảnh do Nginx trả về hay do Apache trả về:
  - Nếu thấy Server: nginx/1.18.0: Điều này có nghĩa là Nginx là điểm cuối cùng gửi ảnh

  - Nếu thấy X-Powered-By: PHP/...: Thường là do Apache/PHP xử lý và đẩy ngược lại Nginx.

![alt text](image-15.png)
