---
title: 通过 Telegram 机器人发送来自 Alertmanager 的告警消息
date: 2024-06-12 22:00:00
categories:
- Idea
tags:
- 突发奇想

---

## 概述

本告警机器人用于接收来自 Alertmanager 的告警消息，并通过 Telegram 机器人将这些消息发送到指定的 Telegram 群组。

## 系统要求

- **操作系统**：Debian 11
- **Python**：3.x 版本
- **Docker**：20.10 及以上版本

## 准备工作

1. **Telegram Bot**：
   - 创建一个 Telegram Bot 并获取 Bot Token。
   - 获取目标群组的 Chat ID。

2. **Alertmanager 配置**：
   - 确保 Prometheus 和 Alertmanager 正常运行。
   - 在 Alertmanager 的配置文件中添加 webhook 接收器（具体操作见下方）。
   - 确保 Prometheus 目录下已经编写好了告警规则。

## 文件结构

```
alertmanager-to-telegram/
├── Dockerfile
├── requirements.txt
├── alertmanager_to_telegram.py
└── README.md
```

## 步骤

### 1. 编写 `alertmanager_to_telegram.py`

这是告警机器人的主程序，使用 Flask 接收来自 Alertmanager 的 webhook 并调用 Telegram API 发送消息。

```python
from flask import Flask, request, jsonify
import requests

app = Flask(__name__)

# 配置 Telegram 相关参数
TELEGRAM_BOT_TOKEN = 'your_bot_token'
TELEGRAM_CHAT_ID = 'your_chat_id'

def send_message_to_telegram(text):
    url = f"https://api.telegram.org/bot{TELEGRAM_BOT_TOKEN}/sendMessage"
    payload = {
        'chat_id': TELEGRAM_CHAT_ID,
        'text': text,
        'parse_mode': 'Markdown'
    }
    response = requests.post(url, json=payload)
    return response

@app.route('/webhook', methods=['POST'])
def webhook():
    try:
        data = request.json
        print("Received data:", data)
    
        alerts = data.get('alerts', [])
        for alert in alerts:
            status = alert.get('status', 'unknown')
            labels = alert.get('labels', {})
            annotations = alert.get('annotations', {})
            instance = labels.get('instance', 'unknown')
            summary = annotations.get('summary', '-')
            description = annotations.get('description', 'No description')
            
            if status == 'firing':
                message = f"*告警:* {summary}\n*主机:* {instance}\n*描述:* {description}"
            elif status == 'resolved':
                message = f"*恢复:* {summary}\n*主机:* {instance}\n*描述:* {description} 已恢复"

            send_message_to_telegram(message)
    
        return jsonify({'status': 'success'})
    except RetryAfter:
        sleep(30)
        send_message_to_telegram(message)
        return jsonify({'status': 'success'})
    except TimedOut as e:
        sleep(60)
        bot.sendMessage(chat_id=chatID, text=message)
        return jsonify({'status': 'success'})
    except NetworkError as e:
        sleep(60)
        bot.sendMessage(chat_id=chatID, text=message)
        return jsonify({'status': 'success'})
    except Exception as error:       
        bot.sendMessage(chat_id=chatID, text="Error: "+str(error))
        app.logger.info("\t%s",error)
        return jsonify({'status': 'fail'})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5001)
```

### 2. 创建 `requirements.txt`

列出所有需要安装的 Python 库：

```
Flask
requests
gunicorn
```

### 3. 编写 `Dockerfile`

编写 Dockerfile 以便构建 Docker 镜像：

```Dockerfile
# 使用官方 Python 镜像作为基础镜像
FROM python:3.9-slim

# 设置工作目录
WORKDIR /app

# 复制当前目录下所有文件到工作目录
COPY . .

# 安装依赖
RUN pip install --no-cache-dir -r requirements.txt

# 暴露 Flask 运行的端口
EXPOSE 5001

# 使用 gunicorn 运行 Flask 应用
CMD ["gunicorn", "-w", "4", "-b", "0.0.0.0:5001", "alertmanager_to_telegram:app"]
```

### 4. 构建和运行 Docker 镜像

在项目目录下运行以下命令：

```bash
docker build -t alertmanager-to-telegram .
docker run -d -p 5001:5001 --name alertmanager-to-telegram alertmanager-to-telegram
```

### 5. 配置 Alertmanager

在 Alertmanager 的配置文件中添加以下配置：

```yaml
route:
  group_by: ['alertname']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 1h
  receiver: 'telegram-webhook'

receivers:
  - name: 'telegram-webhook'
    webhook_configs:
      - url: 'http://your_server_ip:5001/webhook'
```

重新加载 Alertmanager 配置。

### 6. 测试告警

- 制造一个触发条件，检查是否收到告警消息。
- 恢复正常状态，检查是否收到恢复消息。

## 结语

本告警机器人旨在通过 Telegram 实时通知系统异常，方便运维人员及时处理。请根据实际需求进行适当调整和优化。