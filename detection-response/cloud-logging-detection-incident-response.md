---
tags: [Security, Cloud, Detection, IncidentResponse]
---

# 云日志、检测与事件响应

## 先问日志能证明什么

云调查以控制平面事件为主。日志不是启用后就自动完整：需要核对覆盖的账号和区域、管理事件与数据事件、保存位置、完整性、保留期、查询权限和告警延迟。

## AWS 最小可见性

- CloudTrail：谁在什么上下文调用了哪个 API；
- AWS Config：资源配置及其变化；
- CloudWatch：指标、应用或服务日志与告警；
- VPC Flow Logs：网络元数据，不包含完整内容；
- GuardDuty：基于多类遥测的发现，不替代原始日志；
- S3、Load Balancer、WAF、EKS audit 等服务日志。

## CloudTrail 阅读字段

重点查看 `eventTime`、`eventSource`、`eventName`、`awsRegion`、`sourceIPAddress`、`userAgent`、`userIdentity`、`requestParameters`、`responseElements` 和 `errorCode`。沿 `accessKeyId`、session issuer、source identity 和 role session name 关联临时会话。

一次成功调用可能只是侦察，真实影响需要结合后续资源变化；失败调用也能暴露枚举、自动化或权限探测。

## 检测工程

检测规则应描述行为与上下文，而不是只匹配单个 API：

```text
前提或基线 + 事件序列 + 稀有主体或来源 + 高风险资源变化
```

重点场景包括关闭日志或安全服务、创建长期凭据、修改 trust 或 policy、异常 AssumeRole、公开存储、创建高权限工作负载、批量读取秘密和跨区域活动。

每条规则都应包含数据源、逻辑、例外、严重度、调查步骤、响应动作和可测试样例。可以在自有环境使用 Stratus Red Team 等工具生成行为，分别验证攻击是否发生以及日志和告警是否观察到。

## 响应顺序

1. 保存证据和时间线，确认账号、区域、主体和会话；
2. 限制当前能力：撤销或禁用凭据、收紧策略、隔离工作负载；
3. 查找持久化：新用户或密钥、信任关系、函数代码、启动脚本、CI secret；
4. 评估数据和资源影响；
5. 从可信 IaC 或镜像重建，轮换秘密并补充检测。

不要贸然删除所有资源或只轮换一个密钥。这可能破坏证据，并遗漏通过角色、流水线或工作负载建立的持久化。

## 验证标准

完整能力证据应能从 CloudTrail 事件还原临时身份链和资源变化，并形成一条包含测试数据、调查手册和误报说明的检测规则。

## References

- [AWS CloudTrail User Guide](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html)
- [Stratus Red Team](https://github.com/DataDog/stratus-red-team)
