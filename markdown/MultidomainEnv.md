# Using Multiple Domain

- Sửa file hosts tại mục C://Windows/System32/drivers/etc/hosts với windows hoặc /etc/hosts với Ubuntu:
    # localhost name resolution is handled within DNS itself.
        + Gán domain cho địa chỉ ip 
        + ex: 127.0.0.1   wablog.com

# Nginx Config

- Tạo nginx.conf 
    + Truy cập /etc/nginx gõ lệnh
```bash
sudo nano nginx.conf
``` 
- Tạo vhost.conf 
    + Truy cập /etc/nginx/conf.d gõ lệnh
```bash
sudo touch vhost.conf
```
+ Chỉnh sửa nội dung file này bằng content sau:
    + server_name theo domain mới
    + error_log theo domain mới
    + root đến địa chỉ project

    ```
    server {
            listen 80;
            server_name wablog.com;
            error_log /var/log/nginx/wablog-error.log;
            root /home/duchoang/PhpProjects/nishimuta-webapp/public;

            add_header X-Frame-Options "SAMEORIGIN";
        add_header X-XSS-Protection "1; mode=block";
        add_header X-Content-Type-Options "nosniff";

        index index.html index.htm index.php;

        charset utf-8;

        location / {
            proxy_read_timeout 150;
            try_files $uri $uri/ /index.php?$query_string;
        }

        location = /favicon.ico { access_log off; log_not_found off; }
        location = /robots.txt  { access_log off; log_not_found off; }

        error_page 404 /index.php;

        location ~ \.php$ {
            fastcgi_read_timeout 150;
            fastcgi_pass unix:/run/php/php8.1-fpm.sock;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
            include fastcgi_params;
        }

        location ~ /\.(?!well-known).* {
            deny all;
        }
    }
    ```
# Permission Setup

Có 3 service cần setup permission
1. php-fpm:
Setup permission trong /etc/php/{version}/fpm/pool.d/www.conf
Với các thông số: user, group, listen user, listen group
2. nginx:
Setup permission trong /etc/nginx/nginx.conf
Với thông số user
3. Repository (Folder chứa codebase)
Setup permission cho folder bằng "chown -R"
Note: 3 service này cần cùng 1 user, group.