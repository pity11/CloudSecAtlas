---
tags:
  - Index
---
# 云安全资源

## T Wiki

[T Wiki](https://github.com/teamssix/twiki) 是可本地运行的云安全知识库。启动方式：

```bash
docker run --name twiki -d -p 7777:80 teamssix/twiki:main
```

启动后访问 `http://localhost:7777/`。镜像版本应按实验需要固定，避免未来更新改变实验结果。

## Kubernetes Goat

[Kubernetes Goat](https://github.com/madhuakula/kubernetes-goat) 用于隔离环境中的 Kubernetes 安全实验。使用前需要明确集群费用、暴露面和销毁步骤。
