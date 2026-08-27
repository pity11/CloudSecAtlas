---
tags: [Security, Cloud, AWS, Foundations]
---

# 云计算与 AWS 资源模型

## 从服务器思维转向资源 API

云不是一批远程服务器，而是一组由 API 管理的资源。控制平面负责创建、授权、配置和审计；数据平面处理真实业务流量。云安全问题通常表现为某个身份通过控制平面改变资源状态，进而影响数据平面。

## 共享责任

供应商负责物理设施和托管服务底层，用户仍负责身份、数据、网络暴露、工作负载、配置和应用。具体边界随 IaaS、容器、Serverless 和 SaaS 变化，不能用“上云更安全”概括。

## AWS 资源语法

需要能够读取 ARN，并识别 partition、service、region、account 和 resource。分析任意资源时记录：

- 所属组织、账号和区域；
- 创建和管理它的主体；
- 身份策略与资源策略；
- 网络入口和出口；
- 数据位置、加密密钥和备份；
- 日志是否覆盖读取、写入和失败事件。

## 最小服务集合

先掌握横向模型，再扩展具体产品：

- IAM、STS、Organizations：身份和边界；
- EC2、Lambda、ECS/EKS：计算与工作负载身份；
- VPC、Security Group、NACL、Load Balancer：网络；
- S3、EBS、RDS：数据；
- KMS、Secrets Manager、SSM Parameter Store：密钥和秘密；
- CloudTrail、Config、CloudWatch、GuardDuty：可见性和检测。

## 风险分析方法

对于公开端点或泄漏凭据，沿以下链条推演：

```text
入口 -> 初始 Principal -> 可调用 API -> 可影响资源
     -> 可获得的新身份或数据 -> 可见日志
```

不要把获得 shell 当成唯一成功标准。读取敏感对象、修改信任策略、创建访问密钥、注入启动脚本或调用高权限函数，都可能构成关键控制平面影响。

## 能力证据

成熟结论应使用 CLI、API 和日志，而不只依赖控制台截图；能够为小型应用画出账号、VPC、计算、存储、身份和日志图，并解释共享责任落在哪一层。

## 实验纪律

云实验只在隔离账号进行：设置预算告警、统一标签、记录资源清单、实验后销毁，并检查跨区域资源。不要在主账号或真实数据环境直接运行攻击工具。

## References

- [AWS Shared Responsibility Model](https://aws.amazon.com/compliance/shared-responsibility-model/)
- [Amazon Resource Names](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference-arns.html)
