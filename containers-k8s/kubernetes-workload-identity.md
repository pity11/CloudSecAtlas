---
tags: [Security, Kubernetes, Container, WorkloadIdentity]
---

# Kubernetes 与工作负载身份

## 对象模型

Kubernetes 是声明式控制系统，不只是批量运行 Docker。核心对象包括 API Server、etcd、controller、scheduler、kubelet、Pod、Deployment、Service、Ingress、Namespace、ServiceAccount、Secret 和 CRD。

```text
用户或控制器 -> API Server -> etcd 中的期望状态
                           -> controller/kubelet -> 实际工作负载
```

## 两套身份与两层权限

Kubernetes RBAC 决定主体对 Kubernetes API 对象的动作；云 IAM 决定节点或 Pod 对云 API 的动作。托管集群必须同时分析两者，它们可通过 workload identity、IRSA 等机制关联。

RBAC 需要理解 Role/ClusterRole、RoleBinding/ClusterRoleBinding、verb、resource、subresource 和 impersonation。`create pods`、`pods/exec`、`secrets get`、修改 workload 和绑定角色等权限，都可能间接获得更高能力。

## Pod 安全边界

审查：

- privileged、hostPID、hostNetwork、hostPath；
- capabilities、seccomp、AppArmor、SELinux；
- runAsNonRoot、readOnlyRootFilesystem、allowPrivilegeEscalation；
- ServiceAccount token 是否自动挂载；
- 镜像来源、tag、签名和拉取策略；
- 资源限制和探针。

容器边界不是虚拟机边界。若工作负载能访问宿主 socket、敏感 hostPath、kubelet 或高权限 ServiceAccount，风险可能扩展到节点或集群。

## 网络与入口

Service 提供发现和负载均衡，Ingress/Gateway 暴露应用；NetworkPolicy 控制 Pod 网络，但效果依赖 CNI。默认允许的集群需要显式设计入站和出站，特别是 DNS、元数据服务和控制平面访问。

## Secret 与供应链

Kubernetes Secret 默认只是 API 对象编码，不等同于端到端秘密管理。需要 etcd 静态加密、RBAC、审计、外部 secret 管理和轮换。Admission policy 可以限制不安全镜像和工作负载配置。

## 审计路径

```text
外部入口 -> Ingress/Service -> Pod -> ServiceAccount/RBAC
-> Node/Cloud identity -> Secret/Data -> Audit logs
```

完整结论应解释一个 Pod 为什么能访问某个 Secret 和某个云资源，并能从 manifest、RBAC 和网络策略还原真实路径，给出隔离与回归验证。

## References

- [Kubernetes Security Checklist](https://kubernetes.io/docs/concepts/security/security-checklist/)
- [Securing a Cluster](https://kubernetes.io/docs/tasks/administer-cluster/securing-a-cluster/)
- [Application Security Checklist](https://kubernetes.io/docs/concepts/security/application-security-checklist/)
- [Kubernetes Goat](https://github.com/madhuakula/kubernetes-goat)
- [OWASP GKE Goat](https://github.com/OWASP/www-project-gke-goat)
