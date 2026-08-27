---
tags: [Security, Cloud, Network, Data, WorkloadIdentity]
---

# 云网络、数据与工作负载安全

## 用可达性与身份共同分析

云访问很少只由网络决定。判断一次访问，需要同时回答：网络路径是否可达、服务是否认证、主体是否授权、资源策略是否允许，以及数据是否能被密钥解密。

## VPC 模型

- subnet 与 route table 决定路径，不天然等于安全区；
- Security Group 是有状态的资源级规则，NACL 是无状态的子网级规则；
- Internet Gateway、NAT Gateway、VPC Endpoint 和 Transit Gateway 改变流量路径与暴露面；
- Load Balancer、CDN 和 API Gateway 之后仍需处理源站绕过和可信代理头；
- DNS、IPv6 和出站流量经常被遗漏。

画网络路径时标注源、目的、端口、路由、过滤点、身份协议和日志来源。

## 数据安全

### 对象存储

S3 应同时审查 identity policy、bucket/access point policy、ACL、Block Public Access、跨账号复制、预签名 URL、版本控制和访问日志。公开访问只是错误配置之一，跨租户读取和过宽写入同样关键。

### KMS

加密不等于授权隔离。需要区分 KMS key policy、IAM policy、grant、加密上下文和调用服务。若主体拥有解密权限，或能让高权限服务代为解密，静态加密不能阻止数据访问。

### Secrets

秘密不得写入镜像层、Git 历史、Terraform state、构建日志或前端包。优先使用短期身份；必须存储的秘密应具有明确范围、轮换、访问日志和失效机制。

## 工作负载身份

EC2 instance profile、ECS task role、Lambda execution role 和 EKS workload identity 都把云权限交给运行代码。应用层 SSRF、RCE、依赖投毒或任意文件读取因此可能转化为云控制平面权限。

关键问题：

1. 工作负载能取得哪种临时凭据？
2. 凭据能做什么，能否跃迁到更高权限？
3. 同一节点或任务中的其他容器能否窃取凭据？
4. 元数据服务、代理和网络策略如何限制访问？

## Serverless

检查事件来源、资源策略、执行角色、环境变量、层与依赖、并发和失败队列。事件内容仍是外部输入；内部事件不等于可信输入。

## 验证标准

对公开 Web 服务进行端到端分析时，应覆盖“请求入口 → 工作负载 → 角色 → 数据 → KMS → 日志”，并提出可验证的最小权限和隔离措施。
