# Setting up Ubuntu 14.04 for silverstripe on HHVM

**Update packages**  
1. `apt-get update && apt-get upgrade`  
**Install NGINX**  
2. `apt-get install nginx`  
**Install HHVM**  
3. `apt-get install software-properties-common`  
4. `apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0x5a16e7281be7a449`  
5. `add-apt-repository "deb http://dl.hhvm.com/ubuntu $(lsb_release -sc) main"`  
6. `apt-get update`  
7. `apt-get install hhvm`  
**Configure NGINX & HHVM** [more info](https://github.com/facebook/hhvm/wiki/FastCGI)  
8. `/usr/share/hhvm/install_fastcgi.sh`  
9. `/etc/init.d/hhvm restart`  
10. `/etc/init.d/nginx restart`  
11. `update-rc.d hhvm defaults`  
12. `/usr/bin/update-alternatives --install /usr/bin/php php /usr/bin/hhvm 60`  
# HHVM configuration files  
- /etc/hhvm/php.ini
- /etc/hhvm/server.ini
# Silverstripe nginx config file  
**silverstripe.config**  
```nginx
fastcgi_buffer_size 32k;
fastcgi_busy_buffers_size 64k;
fastcgi_buffers 4 32k;

location / {
    try_files $uri /framework/main.php?url=$uri&$query_string;
}

error_page 404 /assets/error-404.html;
error_page 500 /assets/error-500.html;

location ^~ /assets/ {
    try_files $uri =404;
}
location ~ /(mysite|framework|cms)/.*\.(php|php3|php4|php5|phtml|inc)$ {
    deny all;
}
location ~ /\.. {
    deny all;
}
location ~ \.ss$ {
    satisfy any;
    allow 127.0.0.1;
    deny all;
}
location ~ web\.config$ {
    deny all;
}
location ~ \.ya?ml$ {
    deny all;
}
location ^~ /vendor/ {
    deny all;
}
location ~* /silverstripe-cache/ {
    deny all;
}
location ~* composer\.(json|lock)$ {
    deny all;
}
location ~* /(cms|framework)/silverstripe_version$ {
    deny all;
}
```

# Example config file for a website  
**sites-available/mysite**  
```nginx
server {
    listen 80;
    root /var/www/mysite;
    server_name localhost;

    error_log /var/log/nginx/mysite.error.log;
    access_log /var/log/nginx/mysite.access.log;

    location ~ /\.git {
        deny all;
    }

    include /etc/nginx/hhvm.conf;
    include /etc/nginx/silverstripe.conf;
}
```
