

以下都是基于`Ubuntu 16.04 LTS`安装的，主要适合容器内的环境安装，如果不是容器内的环境，建议先切换为root 用户，不然可能权限不够。


### 安装 python 3.7
默认ubuntu带了python2.X的版本，所以python2.X的版本就没必要安装了
```shell
add-apt-repository ppa:deadsnakes/ppa -y && apt-get update && apt-get install python3-pip python-pip python3.7 libnss3 -y && rm /usr/bin/python && ln -s /usr/bin/python3.7 /usr/bin/python 
```

### 安装 node14 
```shell
apt-get -y update && curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash - && apt-get -y install  nodejs && npm install npm@latest -g && npm install yarn -g && npm install pm2 -g  
```

### 安装git

```shell
sudo apt-get -y update && sudo apt-get install git -y 
```

### 安装php 5.6 ,包含redis，mysql，mongodb 的依赖
```shell
echo '构建开始，更新php-fpm的配置' && sudo add-apt-repository ppa:ondrej/php -y && sudo apt-get update && sudo apt-get -y install php5.6 php5.6-mcrypt php5.6-mbstring php5.6-curl php5.6-cli php5.6-mysql php5.6-gd php5.6-intl php5.6-xsl php5.6-zip && apt-get install php5.6-fpm  &&  sudo apt-get install -y --force-yes autoconf m4 libssl-dev git pkg-config &&  apt-get install php5.6-dev -y &&  cd /data &&  sudo wget http://pecl.php.net/get/redis-2.2.8.tgz &&  sudo tar -xzvf redis-2.2.8.tgz &&  cd redis-2.2.8 &&  sudo /usr/bin/phpize &&  sudo ./configure --with-php-config=/usr/bin/php-config &&  sudo make && sudo make install &&  echo '安装完php-redis依赖' &&  cd /data &&  sudo wget http://pecl.php.net/get/mongo-1.6.14.tgz &&  sudo tar -xzvf mongo-1.6.14.tgz &&  cd mongo-1.6.14/ &&  sudo /usr/bin/phpize &&  sudo ./configure --with-php-config=/usr/bin/php-config &&  sudo make && sudo make install &&  echo '安装完php mongo依赖' &&  cd /data &&  sudo wget  http://pecl.php.net/get/mongodb-1.2.8.tgz &&  sudo tar -xzvf mongodb-1.2.8.tgz &&  cd mongodb-1.2.8/ &&  sudo /usr/bin/phpize &&  sudo ./configure --with-php-config=/usr/bin/php-config &&  sudo make && sudo make install &&  echo '安装完mongodb依赖' && echo '启动php-fpm' && /usr/sbin/php-fpm5.6 && echo '构建完成'
```

配置文件分别在这里几个目录文件下(一般都不要使用默认的，使用自己的替换掉下面这几个再启动)：

- /etc/php/5.6/fpm/php.ini
- /etc/php/5.6/cli/php.ini
- /etc/php/5.6/fpm/php-fpm.conf


修改了php全局配置之后，执行一下命令来重新php服务，才会生效
```shell
pgrep php-fpm | xargs sudo kill -USR2
```

php 记录慢日志的地址
```shell
/usr/local/php/logs
```


### 安装maven
```shell
sudo apt-get -y update && sudo apt install maven -y && sudo apt-get install git -y 
```

默认配置的maven 配置在这里，一般需要修改一下里面的远程仓库地址为 国内的，例如可以换成阿里云的maven仓库地址：https://maven.aliyun.com/nexus/content/groups/public/
```shell
/etc/maven/settings.xml
```



### 安装jdk8
```shell
sudo apt-get update;sudo apt-get install openjdk-8-jdk -y;java -version
```

### 安装nginx
```shell
sudo apt update
sudo apt install nginx -y
```
安装后就会自动启动了的，这时候执行`ps -ef | grep nginx`可以看到nginx已经启动了，然后执行`nginx -t` 可以看到nginx默认的配置文件在哪里