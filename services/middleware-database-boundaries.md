---
date: 2026-08-24
tags: [Security, Middleware, Database, Container]
---
# 基础服务、中间件与数据库安全边界

## 服务暴露的三个维度

发现一个端口后，不应立即按产品漏洞表匹配。先回答：谁能到达、是否认证、成功操作以哪个 OS/应用身份执行。FTP、SMB、WebDAV、NFS、Redis、消息队列和管理控制台的危险，往往来自部署边界而非协议天然不安全。

只绑定回环地址的服务仍需审计：获得任意低权限 shell、SSRF、端口转发或同机容器后，可达性边界会变化。反过来，外网不可达不能补偿弱认证，因为横向移动后仍会暴露。

## 端口可达性、协议匹配与最小探针

`open` 只表示 TCP 连接被某个进程接受；Nmap 的 `SERVICE VERSION`、端口号和 Banner 都只是服务识别假设。浏览器只能正确理解 HTTP/HTTPS，不能用“浏览器打不开 6379”推断 Redis 不可达。若 curl 向非 HTTP 服务发送 `GET / HTTP/1.1`，收到协议错误、空响应或连接重置，通常说明协议不匹配，而不是网络不通。

每种服务应使用对应协议的最小、低影响探针：

```bash
# TCP handshake
nc -vz -w 3 <TARGET> <PORT>

# Redis RESP PING
printf '*1\r\n$4\r\nPING\r\n' | nc -w 3 <TARGET> 6379

# HTTP status, headers, and body
curl -i --max-time 5 http://<TARGET>:<PORT>/
```

Redis RESP 是长度前缀协议。上述请求表示一个只包含 `PING` 的数组；`+PONG` 通常表示正常响应，`-NOAUTH` 表示服务可达但需要认证。超时、拒绝连接和协议错误必须分别记录。

服务识别应结合协议行为、标准命令覆盖、错误格式、进程路径、启动参数、包管理器归属和文件哈希。某服务自称 Redis 7.x，但缺少 `ACL`、`MODULE`、`EVAL` 等大量标准命令，或 `INFO` 返回固定伪造字段时，应考虑自定义模拟器、蜜罐或后门，不能直接套用 Redis CVE。

对非标准协议程序，若能通过授权文件读取取得正在运行的二进制，可使用 [Go 二进制逆向与运行时元数据](https://github.com/pity11/WebSecAtlas/blob/main/code-audit/go-runtime-metadata.md) 的流程验证命令分发、认证和危险系统调用。隐藏命令只有在控制流确实到达文件写入、代码加载或命令执行 sink 时，才构成安全结论。

## 文件服务

- WebDAV：可写目录若同时允许 Web 服务器执行脚本，就从“上传”变成代码执行；若服务跟随可控符号链接，还可能越过共享根。
- FTP/SMB：匿名或 guest 访问可能泄露备份和配置；可写共享与计划任务/部署目录相连时会形成执行链。
- NFS：采用数字 UID/GID 信任，`no_root_squash` 和宽网段可写导出尤其危险。

原则是分离“可上传”和“可执行”，规范化并约束最终路径，拒绝跟随不可信符号链接，以专用低权限账户运行，并把共享根与系统路径隔离。

## AJP 与后端协议

AJP 是前端 Web 服务器与 Tomcat 等后端之间的二进制协议。它通常不应直接暴露给不可信网络，因为前端原本承担的路径规范化、认证和 Header 约束可能被绕开。

防护：仅监听回环或独立后端网段；启用协议 secret/认证；限制网络 ACL；及时升级；确认反向代理与后端对路径、Header 和认证语义一致。类似原则也适用于 FastCGI、uwsgi、Docker API 等“原本只给内部组件使用”的控制协议。

## 控制面与健康检查

Consul、Jenkins、容器管理器、CMS 插件后台等控制面常同时掌握凭据、拓扑和任务执行能力。读取 KV 可能得到下游秘密；创建健康检查可能成为内部端口探测器；脚本控制台和插件安装通常等价于服务账户代码执行。

控制面必须单独网络隔离、强认证/MFA、最小 RBAC、审计和秘密脱敏。不要把“管理 UI 有登录页”视为充分边界。

## MySQL UDF

MySQL UDF 允许服务器从 `plugin_dir` 加载本地共享库并注册 SQL 函数。若数据库高权限账户能够把文件写入插件目录，且 `secure_file_priv` 等配置允许，就可能把 SQL 权限转换成数据库服务账户的 OS 代码执行。

降低风险：数据库服务账户不可写插件目录；限制 FILE/CREATE FUNCTION 等权限；设置严格 `secure_file_priv`；数据库不以 root 运行；监控插件目录与函数元数据。

## SQL Server 的信任边界

- `xp_cmdshell` 把 SQL 权限连接到 OS 命令执行，实际身份取决于 SQL Server 服务账户或代理凭据；应默认关闭并严格限制 sysadmin。
- SQL Agent 作业可执行 PowerShell、CmdExec 等步骤；Agent 角色、proxy 和服务账户共同决定权限。
- 数据库 `TRUSTWORTHY`、数据库 owner 和模块执行上下文组合不当，会让 `db_owner` 越过数据库边界取得实例级权限。不要让不可信数据库由高权限登录拥有，避免依赖 TRUSTWORTHY，使用签名模块和最小权限。

## Redis 与消息系统

无认证 Redis、NATS、RabbitMQ 或 MQTT 可能泄露数据、凭据和内部主题，并允许发布任务或改配置。危险程度由消费者决定：一条普通消息若被高权限 worker 当成命令、模板或文件路径，消息写权限就会升级为执行权限。

Redis 枚举应先从只读、低影响命令开始，例如 `PING`、`INFO`、`DBSIZE`、`SCAN` 和当前身份或 ACL 查询。不要因为发现未认证访问就执行 `FLUSHALL`、改写 `dir/dbfilename`、写 SSH key 或加载模块。尤其要区分真实 Redis 与只实现少量 RESP 命令的自定义服务：协议兼容不等于产品和版本相同，Banner 也不构成 CVE 证据。

需要网络隔离、认证与 TLS、主题或队列级 ACL、消息 schema、消费者端授权和幂等性；秘密不能长期作为普通消息或 KV 明文保存。NATS、MQTT 和 SIP 细节见 [消息总线与 SIP-mTLS 安全](message-bus-sip-mtls-security.md)。

## 后门与“版本匹配”的证据标准

某端口、Banner 或版本号只形成假设。后门服务可能监听非标准端口，合法服务也可能被伪造 Banner。验证应结合进程路径、包来源、文件哈希、监听进程、启动单元和网络行为。版本型漏洞必须核实准确构建、发行版 backport 和真实配置，不能只根据扫描器标签下结论。

## 关联笔记

- [消息总线与 SIP-mTLS 安全](message-bus-sip-mtls-security.md)
- [身份认证、JWT 与 OAuth 安全](https://github.com/pity11/WebSecAtlas/blob/main/fundamentals/identity-jwt-oauth-security.md)
- [Go 二进制逆向与运行时元数据](https://github.com/pity11/WebSecAtlas/blob/main/code-audit/go-runtime-metadata.md)
