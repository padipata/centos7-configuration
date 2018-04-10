# Record the process and pit points of centOS 7.4

> 记录自己配置centOS 7.4 过程及坑点

### mac连接服务器

```shell
// 切换到 root 权限下
sudo -i

// 连接
ssh root@xxx.xxx.xxx.xxx
```

### 修复 wget 漏洞
```shell
yum update wget
```

### 测试硬盘速度
```shell
// 写入速度
dd if=/dev/zero of=./largefile bs=1M count=500
// 清理内存（读取前必须进行，不然不准确）
sudo sh -c "sync && echo 3 > /proc/sys/vm/drop_caches"
// 读取速度
dd if=./largefile of=/dev/null bs=4k
```

### 安装lnmp
```shell
wget -c http://soft.vpser.net/lnmp/lnmp1.4.tar.gz

tar zxf lnmp1.4.tar.gz

cd lnmp1.4

./install.sh lnmp

// 注意的是MySQL 5.6,5.7及MariaDB 10必须在1G以上内存的更高配置上才能选择！
```

### 配置CA证书
```shell
1. 从阿里云上申请免费的ca证书（有点隐秘，需要找到“一个域名”）

2. 阿里的ca证书有一个 pem 文件和 key 文件，解压后，分别把名字改为 'ssl.pem'，'ssl.key'

3. Mac将本地文件上传到服务器上（在服务器中新建 /home/ssl 文件夹）
	scp -P 22 /Users/Mr.peng/Downloads/214577254120798/ssl.key root@xxx.xxx.xxx.xxx:/home/ssl
	scp -P 22 /Users/Mr.peng/Downloads/214577254120798/ssl.pem root@xxx.xxx.xxx.xxx:/home/ssl

4. 当后期若需证书改名
	cd /home/ssl
	mv ssl.key ssl2.key
	mv ssl.pem ssl2.pem
```

### inmp 新建站点
```shell
lnpm vhost add
//ssl文件放在 /home/ssl 目录下（ss.key、ssy.pem）
```

### lnmp 删除站点
```shell
lnpm vhost del
chattr -i 网站目录/.user.ini
rm -rf 网站目录
```

### nginx 默认配置
```shell
// 查看配置文件位置信息
nginx -V

// 站点配置文件
lnmp 默认网站配置文件：/usr/local/nginx/conf/nginx.conf
lnmp 默认网站配置文件：/usr/local/nginx/conf/nginx.conf 和 /usr/local/apache/conf/extra/httpd-vhosts.conf
lnmp 默认网站配置文件：/usr/local/apache/conf/extra/httpd-vhosts.conf
新增站点配置文件：/usr/local/nginx/conf/vhost/域名.conf

// lnmp 管理页面
/home/wwwroot/default2（已关闭，防止被攻击）
```

### 修改 nginx 配置
![](files/53648748.png)
![](files/53887871.png)


### 重启 nginx
```shell
lnmp nginx reload
//有时候报错：
Reload service nginx... nginx: [error] open() "/usr/local/nginx/logs/nginx.pid" failed (2: No such file or directory) done

//解决：
/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
```


### 查看 nginx 状态
```shell
ps -ef | grep nginx
```

### 监听端口
```shell
#监听全部
netstat -ntlp

#监听81端口
netstat -inp | grep 81
```

### 安装 node 环境

+ 首先安装 nvm

```shell
cd /usr/local/src

//下载安装包
wget https://github.com/cnpm/nvm/archive/v0.23.0.tar.gz

//解压
tar zxvf v0.23.0.tar.gz
rm -rf v0.23.0.tar.gz
cd nvm-0.23.0

//添加全局依赖
./install.sh
source ~/.bash_profile

//查看 nvm 是否安装成功
nvm
```

+ 安装 node 

```shell
//查看 node 版本列表
nvm ls-remote

//安装，我这里选用 v8.9.0 版本
nvm install v8.9.0

//设置为默认版本
nvm use node
nvm alias default v8.9.0

//查看 node 是否安装成功
node --version
npm --version
```

+ 安装 cnpm

```shell
npm install -g cnpm --registry=https://registry.npm.taobao.org
cnpm --version
```

### 防火墙
```shell
//查看firewalld状态（dead为未开启）
systemctl status firewalld

//开启防火墙
systemctl start firewalld

//关闭防火墙
systemctl stop firewalld
```

### 部署 gitlab（首先执行上面的开启防火墙）
```shell
// 安装配置ssh，配置防火墙
sudo yum install -y curl policycoreutils-python openssh-server
sudo systemctl enable sshd
sudo systemctl start sshd
sudo firewall-cmd --permanent --add-service=http
sudo systemctl reload firewalld

// 安装配置postfix，用于邮件通知
sudo yum install postfix
sudo systemctl enable postfix
sudo systemctl start postfix

// 安装 gitlab 包
curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
sudo yum install gitlab-ce

// 修改访问路径
vim /etc/gitlab/gitlab.rb
external_url 'http://ip_address:new-port'

// 重置配置
sudo gitlab-ctl reconfigure
//往往会出现：ruby_block[supervise_redis_sleep] action run，会一直卡无法往下进行！
1、按住CTRL+C强制结束；
2、运行：sudo systemctl restart gitlab-runsvdir；
3、再次执行：sudo gitlab-ctl reconfigure

//查看当前配置信息
vim /var/opt/gitlab/gitlab-rails/etc/gitlab.yml

// 开启
sudo gitlab-ctl start

# 允许81端口访问
sudo firewall-cmd --permanent --zone=public --add-port=81/tcp
```

### 常用 gitlab 命令
```shell
# 重新应用gitlab的配置 
sudo gitlab-ctl reconfigure 

# 重启gitlab服务 
sudo gitlab-ctl restart 

# 查看gitlab运行状态 
sudo gitlab-ctl status 

#停止gitlab服务 
sudo gitlab-ctl stop 

# 查看gitlab运行日志 
sudo gitlab-ctl tail 

# 停止相关数据连接服务 
gitlab-ctl stop unicorn gitlab-ctl stop sidekiq
```
### 完全卸载 gitlab
```shell
# 停止进程
sudo gitlab-ctl stop

# 卸载（卸载掉ce || ee, 看情况而定）
rpm -e gitlab-ce

# 查看gitlab进程
ps aux | grep gitlab

# 杀掉进程（后面很多个点的那个进程）
kill -9 18777

# 杀掉所有 gitlab 有关的文件
find / -name gitlab | xargs rm -rf
```

### 配置 gitlab 访问站点
```shell
#新建站点
lnmp vhost add (git.yipage.cn)

#修改站点配置
#1.注释 index 指向
#2.添加反向代理
location ~ / {
    proxy_pass http://127.0.0.1:81
}

#测试
nginx -t

#重启
lnmp nginx reload
```

### 监听服务器性能
```shell
#安装dstat
yum -y install detat

#全局监听
dstat -antp 3

#监听消耗cpu最高，内存消耗最大的进程，每3秒打印一次
dstat -antp --top-cpu --top-mem 3
```

### 排查 TCP/IP 错误
```shell
#首先在服务器端请求网站
curl http://xxx.xxx.xxx.xxx:81

#服务器没问题，在客户端发起请求
curl http://xxx.xxx.xxx.xxx:81

#监听81端口下的网络请求包
tcpdump 'tcp port 81'

#若没有动静，说明客户端的请求没有到达服务器端，这个时候就很有可能是防火墙的问题。

#解决1：如果是自配的服务器
#查看firewalld状态（dead为未开启）
systemctl status firewalld
#开启防火墙
systemctl start firewalld
#手动打开81端口的防火墙
sudo firewall-cmd --permanent --zone=public --add-port=81/tcp

#解决2：如果是阿里云或者腾讯云的服务器，需要登录上云平台，找到服务器下的安全组，添加安全组规则，如果不知道要配置的域名是什么，将授权对象设置为 0.0.0.0/0

#解决3：可以通过访问80端口，再通过nginx反向代理到81端口，这样子就可以绕过防火墙。
```

### 部署 jenkins
+ 安装 jdk

```shell
yum -y list java*
yum -y install java-1.8.0-openjdk.x86_64
java
```
+ 安装jenkins

```shell
#下载jenkins包
wget http://pkg.jenkins-ci.org/redhat-stable/jenkins-2.107.1-1.1.noarch.rpm
rpm -ivh jenkins-2.107.1-1.1.noarch.rpm

#修改配置文件，默认端口为8080，如果不冲突则不需要修改
vim /etc/sysconfig/jenkins

#启动jenkins服务
service jenkins start

#重启jenkins
service jenkins restart

#查看jenkins密码并复制
vim /var/lib/jenkins/secrets/initialAdminPassword
```
+ 部署站点

```shell
#部署ca证书
#Linux : 
cd /home & mkdir ssl-build
#Mac :  
scp -P 22 /Users/Mr.peng/Downloads/ca证书/ssl-build/ssl.pem root@39.104.92.85:/home/ssl-build
scp -P 22 /Users/Mr.peng/Downloads/ca证书/ssl-build/ssl.key root@39.104.92.85:/home/ssl-build

#新建站点
lnmp vhost add ( build.yipage.cn )

#修改站点配置
#1.注释 index 指向
#2.添加代理
location ~ / {
    proxy_pass http://127.0.0.1:8090
}

#测试
nginx -t

#重启
lnmp nginx reload
```

+ 完全卸载 jenkins

```shell
#停止服务
sudo service jenkins stop

#卸载jenkins包
sudo yum remove jenkins

#删除偏好设置
sudo rm -r /var/lib/jenkins
```


### 部署api文档管理系统

```shell
#添加域名解析
api.yipage.cn

#传输ca证书
scp -P 22 /Users/Mr.peng/Downloads/ca证书/ssl-api/ssl.pem root@39.104.92.85:/home/ssl-api
scp -P 22 /Users/Mr.peng/Downloads/ca证书/ssl-api/ssl.key root@39.104.92.85:/home/ssl-api

#新建站点
lnmp vhost add (api.yipage.cn)

#下载eolinker开源版
cd /home/wwwroot
git clone https://github.com/padipata/eoapi.git  (3.5.0版本)
cp -rf eoapi/. api.yipage.cn/

#设置权限
chmod 777 -R api.yipage.cn/

#重启nginx
nginx -t
lnmp nginx reload

#访问
https://api.yipage.cn
```

