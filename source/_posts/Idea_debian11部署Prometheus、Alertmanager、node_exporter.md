---
title: debian11部署Prometheus、Alertmanager、node_exporter
date: 2024-04-02 22:00:00
categories:
- Idea
tags:
- 突发奇想

---

### 安装`Prometheus`

#### 创建`prometheus`用户

```bash
useradd -M -s /usr/sbin/nologin prometheus
```

#### 下载二进制文件压缩包

```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.45.4/prometheus-2.45.4.linux-amd64.tar.gz
```

#### 解压安装包内容至`/opt/prometheus/prometheus`

```bash
tar -xvf ./prometheus-2.45.4.linux-amd64.tar.gz -C /opt/prometheus
mv /opt/prometheus/prometheus-2.45.4.linux-amd64 /opt/prometheus/prometheus
```

#### 更改`/opt/prometheus`以及其内容的属主和属组


```bash
chown prometheus:prometheus -R  /opt/prometheus
```
#### 创建`prometheus`服务
创建文件
```bash
vim /etc/systemd/system/prometheus.service
```
插入如下内容后保存退出

```
[Unit]
Description=Prometheus Service  
Documentation=https://prometheus.io/docs/introduction/overview/
After=network-online.target

[Service]
Type=simple  
User=prometheus  
Group=prometheus  
Restart=on-failure
Execstart=/opt/prometheus/prometheus/prometheus  --config.file=/opt/prometheus/prometheus/prometheus.yml  --storage.tsdb.path=/opt/prometheus/prometheus/data  --storage.tsdb.retention.time=60d   --web.enable-lifecycle

[Install]
WantedBy=multi-user.target
```
#### 启动服务

```bash
systemctl start prometheus
```

### 安装`alertmanager`

#### 下载二进制文件压缩包

```bash
wget https://github.com/prometheus/alertmanager/releases/download/v0.27.0/alertmanager-0.27.0.linux-amd64.tar.gz
```

#### 解压安装包内容至`/opt/prometheus/alertmanager`

```bash
tar -xvf ./alertmanager-0.27.0.linux-amd64.tar.gz -C /opt/prometheus
mv /opt/prometheus/alertmanager-0.27.0.linux-amd64 /opt/prometheus/alertmanager
```

#### 更改`/opt/prometheus`以及其内容的属主和属组

```bash
chown prometheus:prometheus -R  /opt/prometheus
```
#### 创建`prometheus`服务
创建文件
```bash
vim /etc/systemd/system/alertmanager.service
```
插入如下内容后保存退出

```
[Unit]
Description=Alert Manager
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=prometheus
Group=prometheus
ExecStart=/opt/prometheus/alertmanager/alertmanager --config.file=/opt/prometheus/alertmanager/alertmanager.yml --storage.path=/opt/prometheus/alertmanager/data
Restart=always

[Install]
WantedBy=multi-user.target
```
#### 启动服务

```bash
systemctl start alertmanager
```


### 整合`alertmanager`和`prometheus`
#### 修改`prometheus`配置

打开`prometheus`配置文件
```bash
vim /opt/prometheus/prometheus/prometheus.yml
```
加入如下内容

```
alerting:
  alertmanagers:
    - static_configs:
      - targets:
      # 根据实际填写alertmanager的地址
        - localhost:9093

rule_files:
# 根据实际名修改文件名
  - "alert.yml"
```
#### 增加触发器配置文件
新建文件

```bash
vim /opt/prometheus/prometheus/alert.yml
```
添加内容

```
groups:
- name: Prometheus alert
  rules:
    # 对任何实例超过30s无法联系的情况发出警报
    - alert: 服务告警
      expr: up == 0
      for: 30s
      labels:
        severity: critical
      annotations:
        instance: "{{ $labels.instance }}"
        description: "{{ $labels.job }} 服务已关闭"
```

#### 检查配置文件

```bash
/opt/prometheus/prometheus/promtool check config /opt/prometheus/prometheus/prometheus.yml
```

#### 重启服务

```bash
systemctl restart prometheus
```

### 安装`Grafana`

```bash
sudo apt-get install -y adduser libfontconfig1 musl
wget https://dl.grafana.com/enterprise/release/grafana-enterprise_10.4.1_amd64.deb
sudo dpkg -i grafana-enterprise_10.4.1_amd64.deb
```

### 安装`node_exporter`
#### 下载二进制文件压缩包

```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
```

#### 解压安装包内容至`/opt/prometheus/node_exporter`

```bash
tar -xvf ./node_exporter-1.7.0.linux-amd64.tar.gz -C /opt/prometheus
mv /opt/prometheus/node_exporter-1.7.0.linux-amd64 /opt/prometheus/node_exporter
```

#### 更改`/opt/prometheus`以及其内容的属主和属组

```bash
chown prometheus:prometheus -R  /opt/prometheus
```
#### 创建`node_exporter`服务
创建文件
```bash
vim /etc/systemd/system/node_exporter.service
```
插入如下内容后保存退出

```
[Unit]
Description=node_exporter
Documentation=https://prometheus.io/
After=network.target

[Service]
User=prometheus
Group=prometheus
ExecStart=/opt/prometheus/node_exporter/node_exporter
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
#### 启动服务

```bash
systemctl start node_exporter
```


### 配置`prometheus`自动拉取`node_exporter`数据
#### 打开`prometheus`配置文件

```bash
vim /opt/prometheus/prometheus/prometheus.yml
```
#### 追加内容

```
- job_name: 'node-exporter'
  scrape_interval: 15s
  static_configs:
  - targets: ['localhost:9100']
    labels:
      instance: node-exporter服务器01
```
### `grafana`添加`prometheus`数据源
进入`grafana`的web图形界面，登录

默认账号：admin，
默认密码：admin

- 新增`prometheus`数据源

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5daa252337ac4cb9870d2119cee7093f~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=2558&h=686&s=94785&e=png&b=1a1d22)
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2a373f37c1f04d698a6df64d71196bc6~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=2217&h=738&s=95024&e=png&b=1f2227)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b47601b1f2414beea5a85677a098dc46~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=895&h=651&s=53554&e=png&b=181b1f)

保存即可

### `grafana`添加`node_exporter`仪表盘

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0cf73f2eddd24e9d90a8b541742aba73~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1717&h=507&s=71299&e=png&b=1a1d22)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8e7d201ecd664ed39859c3b2cd045301~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=983&h=551&s=59774&e=png&b=121318)


![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e4591a5f9ffd4bc283319e1000d292db~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=681&h=425&s=42035&e=png&b=171a1e)

在新页面中，找到`node_exporter`，进入详情页，点击按钮复制ID


![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/140291b991714fbab4fc7fc18ed406ea~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1375&h=890&s=205406&e=png&b=f6f6f6)

返回`grafana`，在输入框中粘贴，然后点击Load按钮

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eb77a96b8f3e40a48c287d5a339eacb9~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=712&h=360&s=34121&e=png&b=181b1f)

Name可自行修改，然后数据源选择刚刚创建的`prometheus`

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/482011e7b8a240b29ff70569106d679c~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=823&h=615&s=62213&e=png&b=181b1f)

最后点击Import即可
