# Cloud Security Fieldbook

面向云身份、资源关系、攻击路径、检测和修复的知识与研究仓库。内容先以 AWS 建模，再映射 Azure、GCP 和 Kubernetes。

完整能力地图与阶段出口见 [`ROADMAP.md`](ROADMAP.md)。

## 内容结构

- [`foundations/`](foundations/)：共享责任、账号/组织、控制面与数据面。
- [`iam/`](iam/)：策略评估、信任关系、临时凭据和权限提升路径。
- [`services/`](services/)：网络、计算、存储、数据库、Serverless、KMS 和 Secrets。
- [`containers-k8s/`](containers-k8s/)：镜像、运行时、ServiceAccount、RBAC 和工作负载边界。
- [`detection-response/`](detection-response/)：审计日志、检测、调查、遏制和恢复。
- [`labs/`](labs/)：授权且成本可控的实验。
- [`references/`](references/)：筛选后的外部资料和工具定位。
- [`templates/`](templates/)：统一实验记录模板。

## 专题入口

- [`services/`](services/)：基础设施、消息系统和身份边界。
- [`references/`](references/)：云安全知识库与靶场入口，以及经过筛选的公开项目。

## 关注主线

1. Principal 拥有什么有效权限，为什么拥有？
2. 资源是否暴露，网络路径是否真实可达？
3. 工作负载可以取得什么临时凭据和秘密？
4. 哪些信任关系能串成跨服务、跨角色或跨账号攻击路径？
5. 哪些日志能够证明行为，如何遏制并验证恢复？

## 仓库边界

- 本仓库保存知识模型、实验记录、攻击路径、检测规则、修复验证和研究问题。
- “掌握”必须留下证据：能解释、能复现、能检测或修复，并能做回归验证。
- 一个方向形成可独立演示、测试和评价的系统后，再从 Fieldbook 孵化成单独项目仓库。

## 安全与成本约束

- 仅使用本人账号、专用实验账号或明确授权环境。
- 默认最小权限、预算告警、短生命周期资源和显式销毁步骤。
- 不提交 Access Key、Session Token、Kubeconfig、Terraform state 或未脱敏日志。
- 攻击模拟优先使用 dry-run、计划输出和官方/本地靶场。
