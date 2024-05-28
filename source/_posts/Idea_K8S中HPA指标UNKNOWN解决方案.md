---
title: K8S中HPA指标UNKNOWN解决方案
date: 2024-4-23 09:00:00
categories:
- Idea
tags:
- 突发奇想

---

```bash
kubectl describe hpa 
```

日志里如果报错
```
unable to get metrics for resource memory: unable to fetch metrics from resource metrics API: 
the server could not find the requested resource (get pods.metrics.k8s.io)
```

解决方案：部署*metrics-server*

- [下载资源清单](https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml)

- 对`components.yaml`做出如下修改：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8dc8c68f12d7407798cd748515125496~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=723&h=250&s=20269&e=png&b=1f1f1f)
```yaml
- --kubelet-insecure-tls
image: registry.cn-hangzhou.aliyuncs.com/google_containers/metrics-server:v0.7.1
```

- 执行命令
```bash
kubectl apply -f components.yaml
```

- 检查Metrics Server部署

```bash
kubectl get deployment metrics-server -n kube-system
```

- 检查Metrics Server日志
```bash
kubectl logs -n kube-system deployment/metrics-server
```

- 验证Metrics API Endpoint
```bash
kubectl get --raw /apis/metrics.k8s.io/v1beta1/nodes
kubectl get --raw /apis/metrics.k8s.io/v1beta1/pods
```