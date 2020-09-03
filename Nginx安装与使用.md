
### 一、环境准备
安装编译工具及库文件
```
yum -y install make zlib zlib-devel gcc-c++ libtool  openssl openssl-devel
```
安装PCRE，让NGINX支持Rewrite功能
```
yum install pcre
```
如编辑报错 ./configure: error: the HTTP rewrite module requires the PCRE library.
```
yum -y install pcre-devel
```
### 二、下载安装
下载
```
wget http://nginx.org/download/nginx-1.13.7.tar.gz
```
如wget命令不能用，下载解压版http://nginx.org/download/
解压
```
tar -zxvf nginx-1.13.7.tar.gz
```
进入目录
```
cd nginx-1.13.7
```
编译并指定目录
```
./configure --prefix=/root/zhaohaipeng/ztuo/nginx   
```
安装
```
make && make install
```
### 三、使用
1. 找到nginx命令所在目录
which nginx
结果：/usr/sbin/nginx
2. 查看配置文件所在目录（/usr/sbin/nginx 为上述命令运行结果）
/usr/sbin/nginx -t
3. 启动
cd nginx目录
sbin/nginx 启动
4. 常用命令
```
nginx -s reload //重启
```
### F&A
1. nginx 报403
修改nginx.config 第一行 user为当前启动niginx的用户 比如 root