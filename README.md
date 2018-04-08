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
 root >>  scp -P 22 /Users/Mr.peng/Downloads/214577254120798/ssl.key root@xxx.xxx.xxx.xxx:/home/ssl
 root >>  scp -P 22 /Users/Mr.peng/Downloads/214577254120798/ssl.pem root@xxx.xxx.xxx.xxx:/home/ssl

4. 当后期若需证书改名
root >> cd /home/ssl
root >> mv ssl.key ssl2.key
root >> mv ssl.pem ssl2.pem
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

# 允许4000端口访问
sudo firewall-cmd --permanent --zone=public --add-port=4000/tcp
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
