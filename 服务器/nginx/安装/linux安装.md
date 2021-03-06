# 1. linux安装
## 1.1. 源码编译安装
## 1.2. centos 自己安装指定版本
> 该安装仅包括安装并启动成功。优化请查看优化部分

### 1.2.1. 报错：用户不在sudoers文件中
解决：
1. 切换至root用户
2. 修改/etc/sudoers文件权限
3. 将想要授权sudo权限的用户添加进去，可仿照root，可/
4. 还原权限

### 1.2.2. 安装wget命令
    $ sudo yum install wget

### 1.2.3. 安装ngnix：
> 官网：https://nginx.org
> 如果是最小安装的centos，使用yum是找不到安装包的。只能使用编译安装。或者把安装包生态安装好（未测试）

1. 添加nginx用户及用户组
    $ useradd nginx -s /sbin/nologin -M
    上面的命令创建了名为nginx的用户和用户组，并且nginx用户无法登录系统（-s /sbin/nologin限制），可以通过id nginx命令查看：
    $ id nginx
    uid=1001(nginx) gid=1001(nginx) groups=1001(nginx)

2. 安装所需环境
> 建议优先安装！否则会报错。

    * gcc安装
        安装nginx需要先将官网下载的源码进行编译，编译依赖gcc环境。
        如果没有gcc环境，编译的时候会报错：
            ./configure: error: C compiler cc is not found
        安装：
            $ sudo yum install -y gcc-c++
    * pcre pcre-devel安装
        rewrite模块需要pcre库。
        zlib安装也需要该库。否则报错
		安装：
			$ sudo yum install -y pcre pcre-devel
    * zlib安装
        zlib库提供了很多压缩和解压缩的方式。nginx使用zlib对http包的内容进行gzip，所以需要安装zlib库。
        如果没有安装，编译安装的时候会报错：
            $ make
            make: *** 没有规则可以创建“default”需要的目标“build”。停止。
            $ make install
            make: *** 没有规则可以创建目标“install”。停止。
        安装：
            $ sudo yum install -y zlib zlib-devel
        安装报错：
            没有该命令：zlib。请使用 /bin/yum --help
        解决：
            缺少pcre pcre-devel，安装该环境。可参考上方
    * openssl安装
        https（ssl协议上传输http）的支持
            $ sudo yum install -y openssl openssl-devel
3. 下载：https://nginx.org/en/download.html
    > 版本介绍

    * Mainline version：Mainline是Nginx目前主力在做的版本，开发板
    * Stable version：最新稳定版，生产环境建议使用该版本
    * Legacy versions：遗留的老版本稳定版
    ```
    $ wget -c https://nginx.org/download/nginx-1.14.0.tar.gz
    ```
4. 解压
        $ tar -zxvf nginx-1.14.0.tar.gz
        $ cd nginx-1.14.0

5. 配置
	需要什么模块功能可以自己选择 ./configure --help 命令可以查看所有模块
    * 使用默认配置(默认被安装到/usr/local/nginx)
            $ ./configure
    * 自定义配置
		```
        ./configure --user=nginx --group=nginx --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-http_gzip_static_module
		```
    * 解决报错
    ./configure: error: invalid option "–group=nginx"
        这里应该是--，而不是-
6. 编译安装
        $ make
        $ sudo make install
    如果make时报错：
        make: *** 没有规则可以创建“default”需要的目标“build”。停止。
    解决：
        安装好zlib后再重新配置（./config）编译（make）即可

7. 配置环境变量
		$ vim /etc/profile
	在合适的位置添加环境变量
		export NGINX_HOME=/usr/local/nginx
		export PATH=$PATH:$NGINX_HOME/sbin
	重新编译/etc/profile文件
		source /etc/profile
	> 重新编译文件时，如果会出现下面的问题
		source /etc/profile
		bash: id: command not found
		bash: tty: command not found
	此时说明在添加环境变量时，有单词写错了，或者是少写了 $PATH，此时需要重新修改 /etc/profile 文件
		
8. 查找安装目录
        $ whereis nginx
        nginx: /usr/local/nginx

9. 操作nginx
    查看nginx进程
        $ ps aux | grep nginx
		$ ps -ef | grep nginx
    开启：
        $ sudo /usr/local/nginx/sbin/nginx
            启动
        $ sudo /usr/local/nginx/sbin/nginx -s stop
            此方式相当于先查出nginx进程id再使用kill命令强制杀掉进程
        $ sudo /usr/local/nginx/sbin/nginx -s quit
            此方式是待nginx进程处理任务完毕后进行停止
        $ sudo /usr/local/nginx/sbin/nginx -s reload
		如果配置好了全局变量，则可：
		$ nginx -s stop
		$ nginx -s reload
		$ nginx -t
		指定配置文件启动
		$ nginx -c /usr/local/nginx/conf/nginx.conf
    报错：
        $ /usr/local/nginx/sbin/nginx
        nginx: [alert] could not open error log file: open() "/usr/local/nginx/logs/error.log" failed (13:permission denied)
    解决：
        sudo
10. 访问
    * 访问成功
            Welcome to nginx!
    * 访问失败
        检测防火墙，查看80端口是否开放

11. 绑定域名/设置根目录
    配置文件目录：/usr/local/nginx/conf/nginx.conf
    * 绑定域名
        将
            listen       80;
            server_name  localhost;
        改为
            listen       80;
            server_name  Brook3.com;
        如果有多个站点，则将server整个复制粘贴。可参照该文件其他server。

    * 设置根目录
        将
            location / {
                root   html;
                index  index.html index.htm;
            }
        改为
            location / {
                root    /var/www;
                index  index.html index.htm index.php;
            }
        将
            location = /50x.html {
                root   html;
            }  
        改为
            location = /50x.html {
                root   /var/www;
            }  

12. 重启nginx
    * 先停止再启动
            $ sudo /usr/local/nginx/sbin/nginx -s quit
            $ sudo /usr/local/nginx/sbin/nginx
    * 平滑重启
            $ sudo /usr/local/nginx/sbin/nginx -s reload

13. 开启php模块
        $ sudo vim /usr/local/nginx/conf/nginx.conf
    将这一段
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}
    修改为
        location ~ \.php$ {
            root           /var/www;
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include        fastcgi_params;
        }
	这里的$document_root即上边的root，所以这一项不可省略！！！
	如果从默认的配置文件中copy的是/scripts，一定要改。否则报错：
		file not found
