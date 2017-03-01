# 编译php7为独立守护进程[ php-fpm ]


---
说明： 我是以ubuntu进行的编译，centos可能有些地方不一样

### 1. 如果要使用sql server需要先装freetds
安装方法：
```shell 
wget http://ibiblio.org/pub/Linux/ALPHA/freetds/stable/freetds-stable.tgz
tar xf freetds-stable.tgz
cd freetds-stable
./configure --prefix=/usr/local/freetds --with-tdsver=7.1  --enable-msdblib
make && make install
```

### 2. 编译php7
注： 因为我本地有多个php，所以安装在/usr/local/7finalphp/下 
配置文件在/etc/7finalphp.d/下 
php-fpm的配置文件在/usr/local/7finalphp/etc/php-fpm.conf
```shell
tar xf php-7.0.0.tar.bz2 
cd php-7.0.0
useradd -r 7finalphp
./configure  --prefix=/usr/local/7finalphp --with-pdo-mysql=mysqlnd --with-mysqli=mysqlnd --enable-mbstring --enable-fpm --with-freetype-dir --with-jpeg-dir --with-png-dir --with-zlib --with-libxml-dir=/usr --enable-xml --enable-sockets --with-mcrypt --with-bz2 --with-config-file-path=/etc/7finalphp/php.ini --with-config-file-scan-dir=/etc/7finalphp.d/ --with-fpm-user=7finalphp --with-fpm-group=7finalphp --with-pear --with-curl --with-pdo_sqlite --with-pdo_dblib=/usr/local/freetds
make && make install
```

### 3. 为php提供配置文件
```shell
cp php.ini-production /etc/7finalphp.d/php.ini
```

### 4. 配置php-fpm
为php-fpm提供SysV init脚本，并将其添加至服务列表：
```shell
cp sapi/fpm/init.d.php-fpm  /etc/init.d/7finalphp
chmod +x /etc/init.d/7finalphp
```
centos 6.6为
``` 
cp sapi/fpm/init.d.php-fpm  /etc/rc.d/init.d/7finalphp
chmod +x /etc/init.d/7finalphp
```
编辑php-fpm的配置文件：
```
cp /usr/local/7finalphp/etc/php-fpm.conf.default  /usr/local/7finalphp/etc/php-fpm.conf
cp /usr/local/7finalphp/etc/php-fpm.d/www.conf.default  /usr/local/7finalphp/etc/php-fpm.d/www.conf
vim /usr/local/php/etc/php-fpm.conf
```
配置fpm的相关选项为你所需要的值，并启用pid文件（如下最后一行）：
    
    error_log = /var/log/7finalphp.log
    pid = /usr/local/php/var/run/php-fpm.pid 

### 5. 启动
```
/etc/init.d/7finalphp start
 netstat -tnl | grep :9000
```
看见监听本地9000端口说明启动成功。

