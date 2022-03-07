# docker-compose搭建lnmp

#### **先决条件**

- 首先需要安装docker

- 安装docker-compost

  

## 1、创建lnmp工作目录

```shell
#创建三个目录
mkdir lnmp && cd lnmp
mkdir -p nginx/conf php mysql/data lnmp/www


#编写nginx 配置文件  nginx/conf/default.conf
vim nginx/conf/default.conf

server {
    listen       80;
    root   /usr/share/nginx/html;
    index   index.html index.htm index.php;


    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    location / {
        index  index.html index.htm index.php ;
        try_files $uri $uri/ /index.php?$query_string;
        autoindex  on;
    }


    location ~ \.php$ {
        #php73是容器命名
        fastcgi_pass   php:9000;
        fastcgi_index  index.php;
        include        fastcgi_params;
        fastcgi_param  PATH_INFO $fastcgi_path_info;
        fastcgi_param  SCRIPT_FILENAME  /var/www/html/$fastcgi_script_name;
    }

}


```

## 2、编写php镜像文件Dockerfile

 因为php需要安装一些扩展文件 使用dockerfile进行镜像构建

```dockerfile
vim php/Dockerfile

# 基础
FROM php:7.2-fpm

# 修改时区
ENV TZ Asia/Shanghai
RUN date -R

# 换源
RUN echo 'deb http://mirrors.ustc.edu.cn/debian stable main contrib non-free' >/etc/apt/sources.list
RUN echo 'deb http://mirrors.ustc.edu.cn/debian stable-updates main contrib non-free' >>/etc/apt/sources.list
RUN apt-get update --fix-missing && apt-get install -y libpng-dev libjpeg-dev libfreetype6-dev  \
        && docker-php-ext-configure gd --with-freetype-dir=/usr/include --with-jpeg-dir=/usr/include/jpeg \
        && docker-php-ext-install gd mysqli opcache pdo_mysql gd zip

ENV PHPREDIS_VERSION 5.0.1
ENV PHPXDEBUG_VERSION 2.6.0
ENV PHPSWOOLE_VERSION 4.5.10

RUN pecl install redis-$PHPREDIS_VERSION \
    && pecl install xdebug-$PHPXDEBUG_VERSION \
    && pecl install swoole-$PHPSWOOLE_VERSION \
    && docker-php-ext-enable redis xdebug swoole

RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
     && php composer-setup.php \
     && php -r "unlink('composer-setup.php');" \
     && mv composer.phar /usr/local/bin/composer \
     && composer config -g repo.packagist composer https://packagist.phpcomposer.com
RUN apt-get install -y git

RUN rm -rf /var/cache/apt/* \
    && rm -rf /var/lib/apt/lists/*
RUN mkdir /var/lib/sessions \
    && chmod o=rwx -R /var/lib/sessions
#容器启动时执行指令
CMD ["php-fpm"]
```



## 3、创建compose 的yml文件



```yaml
version: "3.9"
services:
  #配置nginx
  nginx:
    #镜像名称 nginx:latest
    image: nginx
     #自定义容器的名称
     #container_name: c_nginx
    ports:
      - "80:80"

    #lnmp目录和容器的/usr/share/nginx/html目录进行绑定，设置rw权限
    #将宿主机的~/lnmp/nginx/conf/default.conf和容器的/etc/nginx/conf.d/default.conf进行绑定
    volumes:
      - ./data/:/usr/share/nginx/html:rw
      - ./nginx/conf/default.conf:/etc/nginx/conf.d/default.conf
    #设置环境变量，当前的时区
    environment:
      TZ: "Asia/Shanghai"
    #容器是否随docker服务启动重启
    restart: always
    #容器加入名为lnmp的网络
    networks:
      - lnmp

  #配置php
  php:
    #image: php:7.4-fpm   //由于php扩展比较多  直接build自己的Dockerfile 不需要官方镜像
    build: ./php
    container_name: php
    volumes:
      - ./data/:/var/www/html/:rw
    restart: always
    cap_add:
      - SYS_PTRACE
    networks:
      - lnmp

  #配置mysql
  mysql:
    image: mysql:latest   
    ports:
      - "3306:3306"
    volumes:
      - ./mysql/data/:/var/lib/mysql/:rw
    restart: always
    networks:
      - lnmp
    #设置密码
    environment:
      MYSQL_ROOT_PASSWORD: "123456"
      TZ: :"Asia/Shanghai"
networks:
  lnmp:
```



## 4、执行compose命令

```shell
#构建镜像环境
docker-compose up   # -d 为后台执行
```



## 5、扩展命令

```shell
docker-compose ps  查看compose服务
docker-compose run web env #查看web服务的环境变量
docker-compose stop 停止服务
docker-compose down  #关闭并删除改服务容器   #传递--volumnes同时删除使用的数据卷
```





## 6、php7.4 dockerfile 

 ```dockerfile
 FROM php:7.4-fpm
 
 
 # 设置时区
 ENV TZ=Asia/Shanghai
 RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
 
 # 换源(国内源)
 #RUN echo 'deb http://mirrors.ustc.edu.cn/debian stable main contrib non-free' >/etc/apt/sources.list
 #RUN echo 'deb http://mirrors.ustc.edu.cn/debian stable-updates main contrib non-free' >>/etc/apt/sources.list
 
 # 更新安装依赖包和PHP核心拓展
 RUN apt-get update && apt-get install -y \
         libfreetype6-dev  libjpeg-dev libjpeg62-turbo-dev  libpng-dev \
     && docker-php-ext-configure gd --with-freetype --with-jpeg \
     && docker-php-ext-install -j$(nproc) gd mysqli opcache pdo_mysql 
 
 #7.4添加zip扩展有问题 
 #RUN apk add libzip-dev && docker-php-ext-install zip
 
 
 ENV PHPREDIS_VERSION 5.0.1
 ENV PHPXDEBUG_VERSION 3.1.3
 ENV PHPSWOOLE_VERSION 4.8.7
 
 RUN pecl install redis-$PHPREDIS_VERSION \
     && pecl install xdebug-$PHPXDEBUG_VERSION \
     && pecl install swoole-$PHPSWOOLE_VERSION \
     && docker-php-ext-enable redis xdebug swoole
 
 
 
 # 安装 Composer
 ENV COMPOSER_HOME /root/composer
 RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
 ENV PATH $COMPOSER_HOME/vendor/bin:$PATH
 
 CMD ["php-fpm"]
 
 WORKDIR /var/www/html
 ```

