# 制作nginx + php RPM包

### 编译nginx
> 下载包
```
wget https://github.com/Refinitiv/nginx-sticky-module-ng/archive/1.2.6.tar.gz -P usr/local/src
wget https://github.com/vozlt/nginx-module-vts/archive/v0.1.18.tar.gz -P usr/local/src
wget https://github.com/maxmind/libmaxminddb/releases/download/1.3.2/libmaxminddb-1.3.2.tar.gz -P /usr/local/src
```

> 安装geoIP需要的libmaxminddb库
```
cd /usr/local/src/libmaxminddb
./configure --prefix=/usr/local/libmaxminddb
make && make install
echo "/usr/local/libmaxminddb/lib" > /etc/ld.so.conf.d/libmaxminddb.conf
ldconfig
```

> 安装jemalloc内存优化
```
wget https://github.com/jemalloc/jemalloc/releases/download/5.2.1/jemalloc-5.2.1.tar.bz2 -P /usr/local/src
bzip2 jemalloc-5.2.1.tar.bz2 && tar xf jemalloc-5.2.1.tar
cd jemalloc-5.2.1 && ./configure --prefix=/usr/local/jemalloc
make && make install
echo "/usr/local/jemalloc/lib" > /etc/ld.so.conf.d/jemalloc.conf
ldconfig
```

> 安装依赖
```
yum -y install perl-devel perl-ExtUtils-Embed gd-devel
```

> 安装nginx
```
wget https://openresty.org/download/openresty-1.15.8.2.tar.gz -P /usr/local/src

./configure --prefix=/usr/local/nginx \
            --sbin-path=/usr/local/nginx/sbin/nginx \
            --conf-path=/usr/local/nginx/nginx.conf \
            --http-log-path=/data/logs/nginx/access.log \
            --error-log-path=/data/logs/nginx/error.log \
            --modules-path=/usr/local/nginx/modules \
            --pid-path=/usr/local/nginx/run/nginx.pid \
            --lock-path=/usr/local/nginx/lock/nginx.lock \
            --http-client-body-temp-path=/usr/local/nginx/temp/client_body_temp \
            --http-proxy-temp-path=/usr/local/nginx/temp/proxy_temp \
            --http-fastcgi-temp-path=/usr/local/nginx/temp/fastcgi_temp \
            --http-uwsgi-temp-path=/usr/local/nginx/temp/uwsgi_temp \
            --http-scgi-temp-path=/usr/local/nginx/temp/scgi_temp \
            --user=www \
            --group=www \
            --with-pcre \
            --with-http_realip_module \
            --with-http_ssl_module \
            --with-http_v2_module \
            --with-http_sub_module \
            --with-http_stub_status_module \
            --with-http_gzip_static_module \
            --with-http_geoip_module=dynamic \
            --with-http_image_filter_module=dynamic \
            --with-http_perl_module=dynamic \
            --with-stream=dynamic \
            --with-stream \
            --with-stream_ssl_module \
            --with-stream_ssl_preread_module \
            --add-module=/usr/local/src/nginx-1.15.8/build/nginx-plugin-master \
            --add-module=/usr/local/src/nginx-1.15.8/build/nginx-sticky-module-ng-1.2.6 \
            --add-module=/usr/local/src/nginx-1.15.8/build/nginx-module-vts-0.1.18 \
            --add-module=/usr/local/src/nginx-1.15.8/build/ngx_http_geoip2_module-3.3 \
            --with-cc-opt='-O2' \
            --with-ld-opt="-ljemalloc"

make && make install
```

> 提供环境变量
```
cat << EOF > /etc/profile.d/nginx.sh
export PATH=$PATH:/usr/local/nginx/sbin/nginx
EOF
```
> 提供service启动文件
```
cat << EOF > /usr/lib/systemd/system/nginx.service
[Unit]
Description=nginx - high performance web server
Documentation=http://nginx.org/en/docs
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/usr/local/nginx/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t -c /usr/local/nginx/nginx.conf
ExecStart=/usr/sbin/nginx -c /usr/local/nginx/nginx.conf
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
EOF
```

> 提供日志轮转文件
```
cat << EOF > /etc/logrotate.d/nginx
/data/logs/nginx/*.log {
         daily
         missingok
         rotate 14
         compress
         delaycompress
         notifempty
         create 640 www www
         sharedscripts
         postrotate
                 [ ! -f /usr/local/nginx/run/nginx.pid ] || kill -USR1 `cat /usr/local/nginx/run/nginx.pid`
         endscript
}
EOF
```
---
### 编译php

>下载包
```
wget https://www.php.net/distributions/php-7.0.29.tar.gz -P /usr/local/src
yum -y install wget lrzsz vim  epel-release  libxml2 libxml2-devel curl curl-devel libjpeg-turbo libjpeg-turbo-devel libpng-devel libpng freetype-devel freetype icu libicu-devel libicu libmcrypt libmcrypt-devel libxslt libxslt-devel php-mysql  pcre-devel libtool zlib-devel

./configure --prefix=/usr/local/php \
            --with-config-file-path=/usr/local/php/etc \
            --with-config-file-scan-dir=/usr/local/php/conf.d \
            --enable-fpm \
            --with-fpm-user=www \
            --with-fpm-group=www \
            --with-mysqli=mysqlnd \
            --with-pdo-mysql=mysqlnd \
            --with-iconv-dir \
            --with-freetype-dir=/usr/local/freetype \
            --with-jpeg-dir \
            --with-png-dir \
            --with-zlib \
            --with-libxml-dir=/usr \
            --disable-rpath \
            --enable-bcmath \
            --enable-shmop \
            --enable-sysvsem \
            --enable-inline-optimization \
            --with-curl \
            --enable-mbregex \
            --enable-mbstring \
            --enable-ftp \
            --with-gd \
            --with-openssl \
            --with-mhash \
            --enable-pcntl \
            --enable-sockets \
            --with-xmlrpc \
            --enable-zip \
            --enable-soap \
            --with-gettext  \
            --enable-opcache \
            --enable-intl \
            --with-xsl

make -j4 && make install
```

> 提供环境变量
```
cat << EOF > /etc/profile.d/php.sh
export PATH=$PATH:/usr/local/php/bin
EOF
```
> 提供配置文件
```
cat << EOF > /usr/local/php/etc/php.ini
######避免PHP信息暴露在http头中
expose_php = Off
######避免暴露php调用mysql的错误信息
display_errors = Off
;######允许fopen直接打开URL
allow_url_fopen = On
;######在关闭display_errors后开启PHP错误日志（路径在php-fpm.conf中配置）
log_errors = On
;######设置PHP的扩展库路径
#extension_dir = "/usr/local/php/lib/php/extensions/"
extension_dir = "/usr/local/php/lib/php/extensions/no-debug-non-zts-20151012/"
;######设置PHP的opcache和mysql动态库
zend_extension=opcache.so
extension=swoole.so
extension=xxtea.so
extension=redis.so
;######设置PHP的时区
date.timezone = PRC
;######开启opcache
[opcache]
; Determines if Zend OPCache is enabled
opcache.force_restart_timeout=3600
opcache.memory_consumption=1024
opcache.optimization_level=1
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=4096
opcache.revalidate_freq=60
opcache.fast_shutdown=1
opcache.enable=1
opcache.enable_cli=1
; ######设置PHP脚本允许访问的目录（需要根据实际情况配置）
;open_basedir = /usr/share/nginx/html;
max_execution_time=300
memory_limit=256M
EOF
```
> 提供php-fpm文件
```
cat << EOF > /usr/local/php/etc/php-fpm.conf
[global]
pid = /usr/local/php/var/run/php-fpm.pid
error_log = /usr/local/php/var/log/php-fpm.log
log_level = notice

[www]
listen = 127.0.0.1:9000
listen.backlog = -1
listen.allowed_clients = 127.0.0.1
listen.owner = www
listen.group = www
listen.mode = 0666
user = www
group = www
pm = static
pm.max_children = 300
pm.max_requests = 1024
pm.start_servers = 100
pm.min_spare_servers = 20
pm.max_spare_servers = 100
request_terminate_timeout = 30
request_slowlog_timeout = 30
slowlog = var/log/slow.log
php_admin_flag[expose_php] = off
pm.status_path = /status
```
> 提供service文件
```
cat << EOF > /usr/lib/systemd/system/php-fpm.service
[Unit]
Description=The PHP FastCGI Process Manager
After=syslog.target network.target

[Service]
Type=forking
PIDFile=/usr/local/php/var/run/php-fpm.pid
ExecStartPre=/usr/local/php/sbin/php-fpm --test
ExecStart=/usr/local/php/sbin/php-fpm -D
ExecReload=/bin/kill -USR2 $MAINPID
KillSignal=SIGQUIT
TimeoutStopSec=5
KillMode=process
PrivateTmp=true

[Install]
WantedBy=multi-user.target
EOF
```
---
### 安装fpm打包工具
```
yum install ruby rubygens rpm-build ruby-devel -y
gem sources -a http://mirrors.aliyun.com/rubygems
gem install fpm
```
> 提供安装前脚本
```
#!/bin/bash

/usr/sbin/ldconfig

getent group www > /dev/null || groupadd -r www
getent passwd www > /dev/null || useradd -g www -s /sbin/nologin www -M

chown -R www:www /data/logs/nginx

source /etc/profile.d/nginx.sh
source /etc/profile.d/php-fpm.sh

setenforce 0

systemctl daemon-reload
systemctl enable nginx
systemctl start nginx

systemctl enable php-fpm.service
systemctl start php-fpm.serivce
```
> 提供安装后脚本
```

#!/bin/bash


systemctl stop nginx.service

groupdel www
userdel www

yum remove -y libxml2 libxml2-devel libjpeg-turbo libjpeg-turbo-devel libpng-devel libpng freetype-devel freetype icu libicu-devel libicu libxslt libxslt-devel php-mysql pcre-devel libtool zlib-devel librdkafka-devel perl-devel perl-ExtUtils-Embed gd-devel

/usr/bin/rm -rf \
       /usr/local/nginx \
       /usr/sbin/nginx \
       /usr/lib/systemd/system/nginx.service \
       /usr/local/bin/openresty \
       /data/logs/nginx \
       /etc/logrotate.d/nginx \
       /etc/profile.d/nginx.sh

/usr/bin/rm -rf \
       /etc/ld.so.conf.d/jemalloc.conf \
       /etc/ld.so.conf.d/libmaxminddb.conf

/usr/bin/rm -rf \
       /usr/local/libmaxminddb \
       /usr/local/jemalloc

ldconfig

systemctl stop php-fpm.service

/usr/bin/rm -rf \
       /usr/local/php \
       /usr/lib/systemd/system/php-fpm.service \
       /etc/profile.d/php-fpm.sh

source /etc/profile

exit $?
```

> 创建打包目录
```
mkdir -pv tmp/data/logs/nginx/
mkdir -pv tmp/{ld.so.conf.d,logrotate.d,profile.d}
mkdir -pv tmp/usr/local/
mkdir -pv tmp/usr/lib/systemd/system

cp -a /usr/local/nginx tmp/usr/local/nginx
cp -a /usr/local/jemalloc tmp/usr/local/jemalloc
cp -a /usr/local/libmaxminddb tmp/usr/local/libmaxminddb
cp -a /usr/local/php tmp/usr/local/php
cp -a /etc/ld.so.conf.d/libmaxminddb.conf tmp/etc/ld.so.conf.d/
cp -a /etc/ld.so.conf.d/jemalloc.conf tmp/etc/ld.so.conf.d/
cp -a /etc/logrotate.d/nginx tmp/etc/logrotate.d/
cp -a /etc/profile.d/nginx.sh tmp/etc/profile.d/
cp -a /etc/profile.d/php.sh tmp/etc/profile.d/
```
> 提供打包脚本
```
/usr/local/bin/fpm -f -s dir \
                   -t rpm  \
                   -n lnp \
                   -v ngx1.15.8_php7.0.29_el7 \
                   -C tmp/ \
                   -p /usr/local/src/ \
                   -d "epel-release,libxml2,libxml2-devel,curl,curl-devel,libjpeg-turbo,libjpeg-turbo-devel,libpng-devel,libpng,freetype-devel,freetype,icu,libicu-devel,libicu,libxslt,libxslt-devel,php-mysql,pcre-devel,libtool,zlib-devel,openssl,openssl-devel,librdkafka-devel,perl-devel,perl-ExtUtils-Embed,gd-devel" \
                   --description "nginx1.15.8_el7 + php-7.0.29_el7" \
                   --after-install /usr/local/src/install_after.sh \
                   --after-remove /usr/local/src/remove_after.sh \
                   --verbose \
                   --license 'TX' \
                   -m "bobby" \
                   --vendor "bobby@itcom888.com"

```
