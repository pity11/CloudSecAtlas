# Reference Repositories

[cloud-security-resources.md](cloud-security-resources.md) 保存知识库与靶场入口；下面是进一步筛选后的公开项目。

筛选口径：近三年仍有维护，优先高 Star、明确威胁模型、可复现实验和防守闭环。Star 为 2026-08-24 附近的近似值。

## 知识与方法

| 仓库 | 规模 | 用法 |
|---|---:|---|
| [Hacking-the-Cloud/hackingthe.cloud](https://github.com/Hacking-the-Cloud/hackingthe.cloud) | 约 2.7k stars | 云攻防知识主索引，按云厂商、战术和服务查攻击路径。 |
| [teamssix/awesome-cloud-security](https://github.com/teamssix/awesome-cloud-security) | 约 2k+ stars | 中文资源导航；只选当前仍维护的上游资料。 |

## AWS 实验与攻击路径

| 仓库 | 规模 | 用法 |
|---|---:|---|
| [RhinoSecurityLabs/cloudgoat](https://github.com/RhinoSecurityLabs/cloudgoat) | 约 3.7k stars | 有意脆弱的 AWS 场景，理解 IAM 与服务组合链。 |
| [RhinoSecurityLabs/pacu](https://github.com/RhinoSecurityLabs/pacu) | 约 5.3k stars | 研究 AWS 枚举和利用模块；仅在授权实验账号使用。 |
| [BishopFox/cloudfox](https://github.com/BishopFox/cloudfox) | 约 2.5k stars | 从有限身份枚举资源、信任关系和潜在攻击路径。 |
| [salesforce/cloudsplaining](https://github.com/salesforce/cloudsplaining) | 约 2.2k stars | 学习 IAM 最小权限、通配符和风险分类。 |

## 防守、验证与云原生

| 仓库 | 规模 | 用法 |
|---|---:|---|
| [prowler-cloud/prowler](https://github.com/prowler-cloud/prowler) | 约 14.6k stars | 将配置检查映射到 CIS/NIST/MITRE，并研究整改证据。 |
| [DataDog/stratus-red-team](https://github.com/DataDog/stratus-red-team) | 高活跃 | 用原子化云攻击生成日志，连接攻击技术与检测。 |
| [madhuakula/kubernetes-goat](https://github.com/madhuakula/kubernetes-goat) | 约 5.7k stars | K8s 配置、RBAC、Secret、镜像和运行时场景。 |

## 使用顺序

1. 用 Hacking the Cloud 理解场景和所需权限。
2. 用 CloudGoat/Kubernetes Goat 在隔离环境复现。
3. 手工使用云 CLI 查询资源和 IAM，不先依赖自动化工具。
4. 再用 CloudFox/Pacu 验证枚举或攻击路径，用 Prowler 检查配置。
5. 用 CloudTrail 等日志和 Stratus Red Team 思路完成检测、遏制与回归。
