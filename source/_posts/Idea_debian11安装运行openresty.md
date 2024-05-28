---
title: debian11安装运行openresty
date: 2024-03-25 22:00:00
categories:
- Idea
tags:
- 突发奇想

---

### 下载源压缩包

```bash
wget https://openresty.org/download/openresty-1.21.4.1.tar.gz
```
### 安装依赖
```bash
sudo apt-get install libpcre3-dev libssl-dev perl make build-essential curl
```
### 解压源码
```
tar -xzvf openresty-1.21.4.1.tar.gz
```
### 配置
```bash
cd openresty-1.21.4.1
./configure
```
### 编译和安装
```bash
make -j2
sudo make install
```
### 设置环境
```bash
cd ~
export PATH=/usr/local/openresty/nginx/sbin:/usr/local/openresty/bin:$PATH
```

### 创建工作目录
````bash
mkdir ~/work 
cd ~/work 
mkdir logs/ conf/
````

### 创建配置文件

创建`conf/nginx.conf`，内容如下：

````
worker_processes 1; 
error_log logs/error.log; 
events { 
    worker_connections 1024; 
} 
http { 
    server { 
        listen 8080; 
        location / { 
            default_type text/html; 
            content_by_lua_block { 
                ngx.say("<p>hello, world</p>") 
                } 
            } 
       } 
 }
````


### 运行
````bash
nginx -p `pwd`/ -c conf/nginx.conf
````

### 错误记录

#### 运行目录错误


![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b3deb8c02fa6437890154e357bbad1c1~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1062&h=116&s=16986&e=png&b=212121)

原因：不应该在`~/work/conf`目录下运行，而应该在工作目录`~/work`。

#### 安装`openresty`时未退出`nginx`

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/96c956af8e5347228b028c0b80b930c1~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=743&h=158&s=25274&e=png&b=222222)

报错，显示端口8080被占用。


![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3aa7dd1ae754489198a9f9e0273ee405~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=696&h=117&s=14684&e=png&b=212121)

一查，发现是`nginx`，但是通过`nginx -s stop`退出失败。


![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/84befefb51114a4ba4ac950b1c50ad2c~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1003&h=60&s=8839&e=png&b=212121)

看报错，发现`nginx.pid`的路径有问题。他找的不是原生的`nginx.pid`，而是`openresty`内置的`nginx.pid`

手动打开原生的`nginx.pid`，查看`nginx`的进程id，然后杀死进程。


![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/019585cd821647db9130939918dfd1c7~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=535&h=246&s=22352&e=png&b=212121)

然后手动输入命令运行
````bash
nginx -p `pwd`/ -c conf/nginx.conf
````
运行成功
