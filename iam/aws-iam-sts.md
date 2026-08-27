---
tags: [Security, AWS, IAM, STS]
---

# AWS IAM 与 STS

## IAM 是云安全主干

先区分 Principal、Action、Resource、Condition 和 Context。用户、角色、服务和联合身份都可成为主体；权限不是由角色名称决定，而是所有适用策略在请求上下文中的求值结果。

## 策略求值

核心规则是默认拒绝；适用的显式 Allow 才能放行；适用的显式 Deny 通常覆盖 Allow。完整结果还受以下集合交互影响：

- identity-based policy；
- resource-based policy；
- permissions boundary；
- Organizations SCP/RCP；
- session policy；
- role trust policy 与 STS 会话上下文。

分析访问结果时，应逐项列出请求主体、动作、资源、条件键和所有边界，而不是根据策略名称猜测。

## Role 与 STS

Role 不持有长期凭据，通过 `AssumeRole` 等操作获得临时会话。Trust policy 回答谁能扮演此角色，permissions policy 回答扮演后可以做什么。需要理解 session ARN、role session name、ExternalId、MFA、source identity、session tags 和 role chaining。

## 常见权限提升结构

不要只背命令，应识别能力组合：

- 能修改高权限角色的策略或 trust；
- 能 `PassRole`，同时能创建或更新 Lambda、EC2、ECS、Glue 等消费者；
- 能创建访问密钥、登录配置或新策略版本；
- 能控制高权限代码的执行位置或输入；
- 能读取函数环境变量、实例用户数据、秘密或状态文件。

`iam:PassRole` 自身不产生凭据。它允许将角色交给 AWS 服务，实际影响由可创建的服务资源和角色权限共同决定。

## 最小权限工程

- 优先临时凭据与工作负载身份，减少长期 Access Key；
- 收紧 Action、Resource 和 Condition，而不只看策略名称；
- 使用 Access Analyzer、CloudTrail 和实际使用记录迭代权限；
- 使用 SCP 和 permissions boundary 设置护栏，但不把护栏误认为资源授权；
- 对跨账号信任显式限制组织、账号、ExternalId 或来源条件。

## 分析模板

```text
请求：Principal / Action / Resource / Context
允许来源：
限制边界：
显式拒绝：
信任关系：
最终判断与证据：
```

完整分析应能手工解释跨账号 AssumeRole 和后续资源访问的求值过程，并从 CloudTrail 找到主体、会话、源 IP、请求参数和结果。

## References

- [AWS IAM policy evaluation logic](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html)
- [Explicit and implicit denies](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic_policy-eval-denyallow.html)
- [IAM and STS CloudTrail logging](https://docs.aws.amazon.com/en_en/IAM/latest/UserGuide/cloudtrail-integration.html)
- [Strategies for achieving least privilege at scale](https://aws.amazon.com/blogs/security/strategies-for-achieving-least-privilege-at-scale-part-1/)
