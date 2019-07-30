DNMP（Docker + Nginx + Redis+MySQL +PHP7/5）是一款全功能的**LNRMP一键安装程序**。

DNMP项目特点：
1. `100%`开源
2. `100%`遵循Docker标准
2. 支持**多版本PHP**共存，任意可切换（PHP5.3、PHP5.4、PHP5.6、PHP7.2)
3. 支持绑定**任意多个域名**
4. 支持**HTTPS和HTTP/2**
5. PHP源代码位于Host中
6. MySQL数据位于host中
7. 所有配置文件可在Host中直接修改
8. 所有日志文件可在Host中直接查看
9. 直接拉取镜像使用,无需再构建
10.已经安装多种扩展

`amqp, apc, apcu, bcmath, bz2, calendar, Core, ctype, curl, date, dba, dom, enchant, ereg, exif, fileinfo, filter, ftp, gd, gettext, gmp, hash, iconv, igbinary, imagick, imap, interbase, intl, json, ldap, libxml, mbstring, mcrypt, memcache, memcached, mongo, mongodb, msgpack, mysql, mysqli, mysqlnd, openssl, pcntl, pcre, PDO, pdo_dblib, PDO_Firebird, pdo_mysql, pdo_pgsql, pdo_sqlite, pgsql, phalcon, Phar, posix, pspell, rdkafka, readline, recode, redis, Reflection, session, shmop, SimpleXML, snmp, soap, sockets, SPL, SQLite, sqlite3, standard, swoole, sysvmsg, sysvsem, sysvshm, tidy, tokenizer, uploadprogress, wddx, xml, xmlreader, xmlrpc, xmlwriter, xsl, Zend OPcache, zip, zlib`
## 1.项目结构
目录说明：
```
/
├── conf                    配置文件目录
│   ├── conf.d              Nginx用户站点配置目录
│       ├── certs           ssl证书
│       ├── enabled         pathinfo配置文件
│       ├── rewrite         各类伪静态文件
│   ├── nginx.conf          Nginx默认配置文件
│   ├── mysql.cnf           MySQL用户配置文件
│   ├── php-fpm.conf        PHP-FPM配置文件（部分会覆盖php.ini配置）
│   └── php.ini             PHP默认配置文件
├── log                     Nginx日志目录
├── mysql                   MySQL数据目录
├── www                     PHP代码目录

```
结构示意图：



## 2. 快速使用
1. 本地安装`git`、`docker`和`docker-compose`。
2. `clone`项目：
    ```
    $ git clone https://github.com/jingmian/dnmpmy.git
    ```
3. 如果不是`root`用户，还需将当前用户加入`docker`用户组：
    ```
    $ sudo gpasswd -a ${USER} docker
    ```
4. 启动：
    ```
    $ cd dnmpmy
    $ cp docker-compose-sample.yml docker-compose.yml
    $ cp env.sample .env
    $ docker-compose up
    ```
5.  请在主机的`hosts`文件中加上如下两行：
    ```
    127.0.0.1 www.site1.com
    127.0.0.1 www.site2.com
    ```
    
    * Linux和Mac的`hosts`文件位置： `/etc/hosts`
    * Windows的`hosts`文件位置： `C:\Windows\System32\drivers\etc\hosts`
    
    然后通过浏览器这两个地址就能看到效果，其中：
    
    * Site1和localhost是同一个站点，是经典的http站，
    * Site2是自定义证书的**https站点**，浏览器会有安全提示，忽略提示访问即可。
6. 在浏览器中访问 `www.site1.com`，即可：


PHP代码在这个目录：`./www/site1/`。


## 3. 切换PHP版本？
默认情况下，我们同时创建 **PHP5.3、PHP5.4、PHP5.6和PHP7.2** 三个PHP版本的容器，

切换PHP仅需修改 Nginx 的`fastcgi_pass`选项，

例如，示例的**www.site1.com**用的是PHP5.4，Nginx 配置：
```
    fastcgi_pass   php54:9000;
```
要改用PHP7.2，修改为：
```
    fastcgi_pass   php72:9000;
```
再 **重启 Nginx** 生效。


### 3.1 设置伪静态,集成大部分伪静态封装,如thinkphp,larvel,yii,ci
伪静态文件都在conf/conf.d/rewrite/下面,只需在对应网站的配置文件添加下面这代码即可
```
     include conf.d/rewrite/thinkphp.conf;
```

## 5. 使用Log

Log文件生成的位置依赖于conf下各log配置的值。

### 5.1 Nginx日志
Nginx日志是我们用得最多的日志，所以我们单独放在根目录`log`下。

`log`会目录映射Nginx容器的`/var/log/dnmp`目录，所以在Nginx配置文件中，需要输出log的位置，我们需要配置到`/var/log/dnmp`目录，如：
```
error_log  /var/log/dnmp/nginx.site1.error.log  warn;
```


### 5.1 PHP-FPM日志
大部分情况下，PHP-FPM的日志都会输出到Nginx的日志中，所以不需要额外配置。

如果确实需要，可按一下步骤开启。

1. 在主机中创建日志文件并修改权限：
    ```bash
    $ touch log/php-fpm.error.log
    $ chmod a+w log/php-fpm.error.log
    ```
2. 主机上打开并修改PHP-FPM的配置文件`conf/php-fpm.conf`，找到如下一行，删除注释，并改值为：
    ```
    php_admin_value[error_log] = /var/log/dnmp/php-fpm.error.log
    ```
3. 重启PHP-FPM容器。

### 5.2 MySQL日志
因为MySQL容器中的MySQL使用的是`mysql`用户启动，它无法自行在`/var/log`下的增加日志文件。所以，我们把MySQL的日志放在与data一样的目录，即项目的`mysql`目录下，对应容器中的`/var/lib/mysql/`目录。
```bash
slow-query-log-file     = /var/lib/mysql/mysql.slow.log
log-error               = /var/lib/mysql/mysql.error.log
```
以上是mysql.conf中的日志文件的配置。

## 6. 使用composer
dnmp默认已经在容器中安装了composer，使用时先进入容器：
```
$ docker exec -it dnmpmy_php_1 /bin/bash
```
然后进入相应目录，使用composer：
```
# cd /var/www/html/site1
# composer update
```
因为composer依赖于PHP，所以，是必须在容器里面操作composer的。

## 7. phpmyadmin和phpredisadmin
本项目默认在`docker-compose.yml`中开启了用于MySQL在线管理的*phpMyAdmin*，以及用于redis在线管理的*phpRedisAdmin*，可以根据需要修改或删除。

### 7.1 phpMyAdmin
phpMyAdmin容器映射到主机的端口地址是：`8080`，所以主机上访问phpMyAdmin的地址是：
```
http://localhost:8080
```

MySQL连接信息：
- host：(本项目的MySQL容器网络)
- port：`3306`
- username：root（默认）
- password：123456（在docker-compose.yml初始化就要配置好）


Redis连接信息如下：
- host: (本项目的Redis容器网络)
- port: `6379`

### 7.2 php连接mysql
```
host地址需要填写:dnmpmy_mysql_1
```


## 8 在正式环境中安全使用
要在正式环境中使用，请：
1. 在php.ini中关闭XDebug调试
2. 增强MySQL数据库访问的安全策略
3. 增强redis访问的安全策略


## License
MIT


