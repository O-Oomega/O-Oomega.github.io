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
ExecStart=/opt/prometheus/prometheus/prometheus \
  --config.file="/opt/prometheus/prometheus/prometheus.yml" \
  --storage.tsdb.path="/opt/prometheus/prometheus/data" \
  --storage.tsdb.retention.time="60d" \
  --web.enable-lifecycle


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

![image.png](/images/Idea/debian11部署Prometheus、Alertmanager、node_exporter/新增数据源1.image)
![image.png](/images/Idea/debian11部署Prometheus、Alertmanager、node_exporter/新增数据源2.image)

![image.png](/images/Idea/debian11部署Prometheus、Alertmanager、node_exporter/新增数据源3.image)

保存即可

### `grafana`添加`node_exporter`仪表盘

![image.png](/images/Idea/debian11部署Prometheus、Alertmanager、node_exporter/添加仪表盘1.image)

![image.png](/images/Idea/debian11部署Prometheus、Alertmanager、node_exporter/添加仪表盘2.image)


![image.png](/images/Idea/debian11部署Prometheus、Alertmanager、node_exporter/添加仪表盘3.image)

在新页面中，找到`node_exporter`，进入详情页，点击按钮复制ID


![image.png](/images/Idea/debian11部署Prometheus、Alertmanager、node_exporter/添加仪表盘4.image)

返回`grafana`，在输入框中粘贴，然后点击Load按钮

![image.png](/images/Idea/debian11部署Prometheus、Alertmanager、node_exporter/添加仪表盘5.image)

Name可自行修改，然后数据源选择刚刚创建的`prometheus`

![image.png](/images/Idea/debian11部署Prometheus、Alertmanager、node_exporter/添加仪表盘6.image)

最后点击Import即可
