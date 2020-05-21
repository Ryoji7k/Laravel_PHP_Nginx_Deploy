# PHP_Laravel_Nginx_Deploy

### Enviroment

```
・ EC2(amazon linux2) + PHP(php-fpm) + Nginx + Laravel + RDS + S3
```

### Install

```
$ yum update
$ amazon-linux-extras install nginx1.12
$ amazon-linux-extras install php7.2
$ yum -y install mysql
$ yum install git
$ yum install --enablerepo=remi,remi-php70 php php-devel php-mbstring php-pdo php-gd php-xml
```

### PHP-FPM conf (Use TCP)

```
[/etc/php-fpm/.../www.conf]
user = ec2-user
group = ec2-user

listen = 127.0.0.1:9000

↓↓↓ comment out ↓↓↓ 
;listen = /var/run/php-fpm.sock
```

### Nginx Conf (Use TCP)

```
[/etc/nginx/nginx.conf]
user ec2-user

↓↓↓ comment out ↓↓↓ 
# server {
    # ・・・・
    # ・・・・
    # ・・・・
    # ・・・・
#}
```

```
[/etc/nginx/conf.d/server.conf]
server {
    listen 80;
    server_name xxxxxxx;  <= Change
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_redirect off;
    proxy_max_temp_file_size 0;

    root   /var/www/html/app_name/public; <= Change(app_name)
    index  index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location /css/ {
        alias /var/www/html/app_name/public/css/; <= Change(app_name)
    }

    location /js/ {
        alias /var/www/html/app_name/public/js/; <= Change(app_name)
    }

    location ~ \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
}
```

### User & Group Change

```
$ sudo chown -R ec2-user /var/lib/nginx
```

### Composer Install

```
$ curl -sS https://getcomposer.org/installer | php
$ mv composer.phar /usr/local/bin/composer
```

### APP Clone

```
[/var/www/html]
$ git clone URL

[/var/www/html/app]
$ composer install -o
$ cp .env .env_example

$ php artisan key:generate
$ php artisan storage:link
$ php artisan config:clear
$ php artisan route:cache

$ chmod -R 777 storage
$ chmod -R 777 bootstrap/cache

$ php artisan migrate
```

### Nginx Basic Auth

```
$ yum install httpd-tools
$ htpasswd -c /etc/nginx/.htpasswd username
$ vi /etc/nginx/conf.d/server.conf
$ service nginx restart
```

### Service Restart

```
$ sudo service php-fpm restart
$ sudo service nginx restart
$ netstat -a --tcp
(confirm TCP localhost:9000)
```

### Other

Laravel Cache Clear

```
$ php artisan cache:clear
```

```
$ php artisan config:clear
```

```
$ php artisan route:clear
```

```
$ php artisan view:clear
```

```
$ composer dump-autoload
$ php artisan clear-compiled
$ php artisan optimize
$ php artisan config:cache
```
