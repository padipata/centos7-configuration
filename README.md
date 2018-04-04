### mac连接服务器

```
// 切换到 root 权限下
sudo -i

// 连接
ssh root@xxx.xxx.xxx.xxx
```

### 修复 wget 漏洞
```
yum update wget
```

### 测试硬盘速度
```
// 写入速度
dd if=/dev/zero of=./largefile bs=1M count=500
// 清理内存（读取前必须进行，不然不准确）
sudo sh -c "sync && echo 3 > /proc/sys/vm/drop_caches"
// 读取速度
dd if=./largefile of=/dev/null bs=4k
```

### 安装lnmp
```
wget -c http://soft.vpser.net/lnmp/lnmp1.4.tar.gz

tar zxf lnmp1.4.tar.gz

cd lnmp1.4

./install.sh lnmp

// 注意的是MySQL 5.6,5.7及MariaDB 10必须在1G以上内存的更高配置上才能选择！
```

### 配置CA证书
```
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
```
lnpm vhost add
//ssl文件放在 /home/ssl 目录下（ss.key、ssy.pem）
```

### lnmp 删除站点
```
lnpm vhost del
chattr -i 网站目录/.user.ini
rm -rf 网站目录
```

### nginx 默认配置
```
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



