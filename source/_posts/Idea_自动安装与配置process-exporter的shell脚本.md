---
title: 自动安装与配置process-exporter的shell脚本
date: 2024-04-03 22:00:00
categories:
- Idea
tags:
- 突发奇想

---

脚本内容可按需求自行修改

### install.sh


```bash
#! /usr/bin/bash

# 定义 Prometheus 目录变量
prometheus_dir="/opt/prometheus"

# 下载 process-exporter 安装包
wget https://github.com/ncabatoff/process-exporter/releases/download/v0.7.10/process-exporter-0.7.10.linux-amd64.tar.gz

# 解压 process-exporter
tar -xf ./process-exporter-0.7.10.linux-amd64.tar.gz -C "$prometheus_dir/"

mv "$prometheus_dir/process-exporter-0.7.10.linux-amd64" "$prometheus_dir/process-exporter"

# 检查 Prometheus 用户是否存在，不存在则创建
if ! id "prometheus" &>/dev/null; then
    useradd -M -s /usr/sbin/nologin prometheus
fi

# 更改文件所有者为 prometheus 用户
chown prometheus:prometheus -R "$prometheus_dir"

# 创建 process-exporter 监控配置文件
# 可自定义内容
cat << EOF > "$prometheus_dir/process-exporter/config.yaml"
process_names:
    - name: "{{.Comm}}" # 匹配模板
    cmdline: 
    - '.+' # 匹配所有名称
EOF

# 运行
"$prometheus_dir/process-exporter/process-exporter" -config.path "$prometheus_dir/process-exporter/config.yaml"

```

### config.sh


```bash
#! /usr/bin/bash

path_of_config_file="/opt/prometheus/prometheus/prometheus.yml"
host_of_process_exporter="localhost:9256"
# 修改Prometheus主机上的配置文件，追加对应的job
# 可自定义内容
cat << EOF >> "$path_of_config_file"
- job_name: 'process-exporter'
  scrape_interval: 15s
  static_configs:
  - targets: ["$host_of_process_exporter"]
    labels:
      instance: process-exporter服务器
EOF

# 重载Prometheus配置
systemctl restart prometheus
```
