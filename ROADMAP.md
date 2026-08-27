# Cloud Security Long-term Roadmap

这是一张云安全主题与能力证据地图。

## 证据等级

- **L1 识别**：知道概念、风险边界和常见误区。
- **L2 操作**：能在授权环境中配置、查询和复现实验。
- **L3 分析**：能解释有效权限、攻击路径、日志证据与修复机制。
- **L4 产出**：形成可测试的工具、数据集、检测、公开文章或研究原型。

每个主题至少记录：威胁模型、最小环境、复现步骤、观测证据、修复、回归测试、成本与销毁方式。

## A. 云计算与安全基础

- 共享责任模型，以及身份、网络、数据、工作负载、控制面的边界。
- Organization / Account、Tenant / Subscription、Organization / Project 的资源层级。
- Region、Availability Zone、控制面与数据面、资源生命周期和计费模型。
- CLI、SDK、API、控制台之间的对应关系；配置文件、环境变量和凭据链。
- 资产清单、标签、配额、预算告警和实验资源销毁。

**阶段出口：** 不依赖控制台截图，能用 CLI/API 解释一个资源如何被创建、授权、访问、审计和删除。

**专题笔记：** [云计算与 AWS 资源模型](foundations/cloud-resource-model.md)

## B. 身份、权限与信任关系

- User、Role、Service Principal、Managed/Workload Identity 的区别。
- 身份策略、资源策略、信任策略、显式拒绝和最终有效权限。
- STS、临时凭据、角色扮演、会话策略、联合身份和 OIDC。
- 权限边界、组织级约束、跨账号信任、PassRole 类委派风险。
- 密钥轮换、秘密管理、凭据泄露调查与无长期密钥部署。
- 以 AWS IAM 建模，再映射 Azure Entra ID/RBAC 与 GCP IAM。

**候选项目：** `cloud-identity-graph`——将身份、策略、资源和信任关系归一化为可查询的有效权限/攻击路径图。

**专题笔记：** [AWS IAM 与 STS](iam/aws-iam-sts.md)

## C. 网络、数据与托管服务安全

- VPC/VNet、子网、路由、网关、安全组、NACL、防火墙和私网端点。
- 对象存储、数据库、消息队列、Serverless、KMS、Secrets Manager 的授权边界。
- 公网暴露、错误资源策略、元数据服务、SSRF 到云凭据的路径。
- 静态加密、传输加密、密钥策略、备份、快照和跨区域复制风险。
- DNS、负载均衡、API Gateway、CDN/WAF 对真实流量路径的影响。

**阶段出口：** 能从“入口—身份—服务—数据—日志”完整解释并修复一条攻击路径。

**专题笔记：** [云网络、数据与工作负载安全](services/cloud-network-data-workload-security.md)

## D. IaC、CI/CD 与软件供应链

- Terraform 资源图、模块、provider、state、plan 与漂移风险。
- CloudFormation/Bicep 等声明式配置的安全审查思路。
- Policy as Code、IaC 静态扫描、例外管理和回归测试。
- GitHub Actions OIDC、短期云凭据、环境保护和最小权限工作流。
- 镜像、依赖、SBOM、制品签名、来源证明与部署准入。
- CI 日志、缓存、Artifact、第三方 Action 和自托管 Runner 的秘密暴露面。

**阶段出口：** 能让一个错误配置在提交阶段失败，并证明部署身份没有不必要权限。

**专题笔记：** [IaC、CI/CD 与软件供应链](services/iac-cicd-supply-chain-security.md)

## E. 容器与 Kubernetes 安全

- 镜像分层、能力集、namespace/cgroup、挂载、socket 与特权容器。
- Kubernetes API、ServiceAccount、Token、RBAC 和工作负载身份。
- Secret、NetworkPolicy、Pod Security、Admission、审计日志和运行时检测。
- 从 Pod 到节点、控制面和云身份的边界及逃逸/横向移动前提。
- 托管 Kubernetes 的云 IAM、集群 RBAC 与网络策略交叉关系。

**候选项目：** `k8s-identity-lab`——用最小集群呈现 ServiceAccount、RBAC、工作负载身份和云 IAM 的组合路径。

**专题笔记：** [Kubernetes 与工作负载身份](containers-k8s/kubernetes-workload-identity.md)

## F. 安全态势、检测与响应

- CloudTrail/Activity Log/Audit Logs、Config、Flow Logs 和服务级数据事件。
- CSPM、基线、资产变化、误配置检测和风险优先级。
- GuardDuty 等原生检测的输入、盲区和可验证性。
- 访问密钥泄露、异常角色扮演、公开存储、策略篡改等事件调查。
- 遏制、凭据撤销、会话失效、证据保全、恢复和事后回归。
- 使用 Prowler、Stratus Red Team 等工具时理解其模型，而非只保存扫描结果。

**候选项目：** `cloud-detection-validation-lab`——以安全原子行为生成日志，自动验证检测、调查线索和修复状态。

**专题笔记：** [云日志、检测与事件响应](detection-response/cloud-logging-detection-incident-response.md)

## G. 授权攻击模拟与攻击路径分析

- 初始身份确认、资源枚举、有效权限推导和最小噪声验证。
- 权限提升、跨角色/跨账号移动、持久化、数据访问和日志规避的前置条件。
- CloudGoat、Pacu、CloudFox 等靶场/工具的适用边界与输出解释。
- 将每次攻击模拟映射到日志、检测、控制措施和修复回归。
- 所有实验限定在本人账号、专用实验账号或明确授权环境。

**阶段出口：** 不把工具输出当结论，能以证据证明攻击路径成立或不成立。

## H. 研究与公开产出

- 从实验中积累：失败案例、策略语义差异、检测盲区和可测量研究问题。
- 形成公开可复现的最小数据集、基准、攻击图、检测评价或安全工具。
- 先把单云问题做深，再判断是否需要多云统一建模。
- 对威胁有效性、误报、覆盖率、成本和可重复性设置评价指标。

成熟课题可以独立建仓；例如 Agent/工具调用与云权限的交叉问题可进入独立的 `AgentBlastLab`，但它不是本 Fieldbook 的替代品。

## 能力里程碑

### M1：建立云资源模型

能用 CLI/API 完成资源生命周期管理，解释共享责任、控制面/数据面和审计来源。

### M2：把 IAM 学到 L3

能手算并验证有效权限，分析信任关系、临时凭据和跨账号路径。

### M3：完成服务级攻击闭环

至少对身份、网络、对象存储、Serverless/元数据等路径完成复现—检测—修复—回归。

### M4：覆盖云原生工程链路

把 IaC、CI/CD、容器/Kubernetes、供应链和检测响应连成一个可重复实验环境。

### M5：形成研究能力

持续提出可证伪问题，建设工具或数据集，并产出可复现的公开成果。

## 维护规则

- AWS 是第一主线，不同时背诵三家云的产品名；跨云比较围绕身份、资源、策略和日志语义。
- 每个实验写明费用、区域、权限和销毁步骤；默认短生命周期资源。
- 不提交长期凭据、Session Token、Kubeconfig、Terraform state 或未脱敏日志。
