---
tags: [Security, IaC, CICD, SupplyChain]
---

# IaC、CI/CD 与软件供应链

## Web 与云的交汇点

CI/CD 同时持有源码、构建输入、云凭据和部署能力。一个 PR、依赖或工作流表达式可能越过代码边界，最终修改生产云资源。

## Terraform 模型

- provider 代表对外部 API 的访问能力；
- plan 是预期变更，apply 才改变真实资源；
- state 保存资源映射和大量敏感属性，应视为高价值秘密；
- module 是代码依赖，需要锁定来源与版本并审计；
- workspace 不是强安全边界。

远端 state 需要访问控制、加密、版本恢复和日志。`sensitive = true` 主要抑制部分显示，不保证值不会进入 state。

## IaC 审查问题

- 是否产生公网入口、`0.0.0.0/0`、公开存储或过宽 IAM？
- 默认值是否安全，变量是否可被低信任输入控制？
- 资源策略、信任策略和 KMS 策略是否跨账号过宽？
- 删除保护、备份、日志和加密是否存在？
- plan 与实际部署是否发生漂移？扫描是否针对最终 plan？

## GitHub Actions 信任模型

区分事件、触发者、检出代码、表达式、runner、token、secret、artifact 和 environment。高风险模式包括：

- 在高权限上下文执行不受信 PR 代码；
- 把用户可控字符串拼入 shell；
- 第三方 Action 只固定 tag，而非不可变 commit；
- 自托管 runner 被持久化或跨仓库复用；
- `GITHUB_TOKEN` 权限和云角色信任过宽；
- artifact 或 cache 被低信任任务写入、高信任任务执行。

## OIDC 联邦

GitHub OIDC 可以避免长期云密钥，但必须在云 trust policy 中约束 issuer、audience、repository、branch、tag 和 environment 等 claims。短期凭据不会自动获得最小权限。

## 供应链

需要理解依赖混淆、typosquatting、恶意安装脚本、构建系统污染、制品替换和发布令牌泄漏。应建立来源、版本、哈希或签名、SBOM、构建证明和可重复构建意识。

## 安全闭环

```text
lint/secret scan -> SAST/dependency/IaC scan -> isolated build
-> signed artifact/attestation -> constrained deployment identity
-> runtime monitoring -> rollback
```

工具告警必须回到真实数据流和权限影响，不能把扫描通过当成安全证明。

## References

- [Terraform: Manage sensitive data](https://docs.hashicorp.com/terraform/language/manage-sensitive-data)
- [Terraform state](https://developer.hashicorp.com/terraform/language/state)
- [OWASP CI/CD Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/CI_CD_Security_Cheat_Sheet.html)
- [GitHub Actions OIDC with AWS](https://docs.github.com/en/actions/how-tos/secure-your-work/security-harden-deployments/oidc-in-aws)
- [AWS Terraform security best practices](https://docs.aws.amazon.com/prescriptive-guidance/latest/terraform-aws-provider-best-practices/security.html)
