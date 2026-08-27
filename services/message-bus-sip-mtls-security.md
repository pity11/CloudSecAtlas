---
date: 2026-08-24
tags:
  - Security
  - NATS
  - MQTT
  - SIP
  - TLS
  - Protocol
---
# 消息总线与 SIP-mTLS 安全

> 仅用于明确授权的协议分析。重点是协议模型、信任边界和错误设计，不记录某台靶机的账号、密码或完整利用报文。

## NATS：文本协议之上的消息信任边界

NATS 是轻量消息总线，常用于发布/订阅和请求/响应。典型 TCP 连接流程：

```text
Server -> INFO {...}\r\n
Client -> CONNECT {...}\r\n
Client -> PING\r\n
Server -> PONG\r\n
```

常见操作包括：

- `PUB <subject> ...`：向主题发布消息；
- `SUB <subject> <sid>`：订阅主题；
- `>`：多级通配主题，权限很大；
- JetStream：在相同协议和端口上增加持久化流与消费者。

TLS、认证和细粒度权限都可配置，但不能假定内部部署一定启用。测试时应区分三层：

1. 能否建立连接；
2. 凭据是否通过认证；
3. 通过认证后能发布/订阅哪些 subject。

认证成功不等于拥有全主题权限。`Authorization Violation` 与 `Permissions Violation` 分别反映身份和授权阶段；错误差异可用于诊断配置，但过于稳定、详细的差异也可能成为凭据枚举信号。

## 消息到 shell：最危险的反模式

危险数据流：

```text
NATS subject -> worker 收到消息 -> os.system()/shell=True -> 主机命令
```

这会把“能发布某主题”直接等价为“能以 worker 身份执行命令”。若另一个 root 服务订阅管理主题并使用同样模式，消息总线 ACL 就变成主机权限边界；一个过宽的发布权限会直接升级为 root RCE。

安全设计：消息体必须是结构化任务描述；任务类型使用枚举；参数逐字段校验；执行固定程序并传参数数组；worker 使用专用低权限账户、隔离文件系统和出站网络；普通业务账户不能发布管理主题；服务端和客户端都记录 subject、主体和结果，但不记录秘密。

## NATS 枚举思路

在授权环境中，先读取 `INFO` 横幅，确认版本、TLS 和认证要求；使用最小请求区分认证失败、权限失败和正常心跳；随后基于明确授权测试有限 subject，而不是一开始就全通配监听。

如果出现周期心跳，可从 subject 命名和帮助/请求响应通道推断服务职责。推断必须由响应或配置验证，不能因为主题名像 `admin` 就直接断言可执行命令。

防守侧应：启用 TLS；避免明文和弱口令；每个服务使用独立身份；精确配置 publish/subscribe allowlist；禁止不必要的 `>`；管理账户与业务账户分离；定期审计 JetStream 中的持久化敏感消息。

## MQTT：连接权限不等于主题权限

MQTT 同样采用发布/订阅模型。安全判断必须拆成三层：

```text
能否连接 broker -> 是否通过身份认证 -> 可发布或订阅哪些 topic
```

匿名连接不必然等于全主题可读，但“允许匿名连接 + 缺少订阅 ACL”会使消息成为公开数据。MQTT 通配符中，`+` 匹配单层 topic，`#` 匹配当前层以下的所有 topic；允许不受信客户端订阅 `#`，等价于授予其观察当前和未来全局消息的能力。

取证时需要区分两种稍后出现的消息：

- retained message：broker 为某个 topic 保存最近一条保留消息，新订阅者可以立即收到；
- 周期消息：由设备或任务按周期重新发布，观察时间太短可能误判为没有数据。

观察窗口应覆盖已知业务周期，并记录 topic、发布频率、QoS、retain 标志和客户端身份。不能因为短时间静默就下结论，也不能无限期全量监听超出授权范围的主题。

密码、SSH 凭据和长期 Token 不应进入消息体。TLS 只能保护链路，任何具有订阅权的客户端仍可读取内容。生产配置应禁用匿名访问，为每个设备或服务分配独立身份，对 publish 和 subscribe 分别配置最小 topic ACL，限制通配符订阅，轮换已暴露秘密，并审计异常全局订阅和敏感字段。

## SIP、SIPS 与 SDP

SIP 是建立、修改和终止会话的信令协议。SIPS 通常表示 SIP 经 TLS 传输；TLS 保护传输机密性和完整性，但不自动保证应用层授权正确。

SIP 报文与 HTTP 类似：

```text
请求行
若干 Header
空行
可选消息体
```

常见方法：

- `OPTIONS`：查询能力或探测可用性；
- `REGISTER`：注册用户位置；
- `INVITE`：发起会话；
- `MESSAGE`：发送即时消息；
- `BYE`：结束会话。

`INVITE` 常携带 SDP。SDP 描述媒体会话参数，例如地址、端口、传输协议和编码；它不是媒体流本身。

SIP 通常要求 CRLF (`\r\n`) 分隔行，Header 与消息体之间必须有空行。`Content-Length` 应是消息体字节数，而不是字符数；编码不同时两者可能不同。手工调试时应先构造文件并检查原始字节，避免终端自动换行或长度错误。

## TLS 与 mTLS

普通 TLS 的典型身份关系：客户端验证服务端证书。mTLS 在此基础上增加服务端验证客户端证书。

```text
客户端信任 CA -> 验证服务端证书
服务端信任 CA -> 验证客户端证书
握手成功       -> 建立加密且双向认证的通道
```

证书、私钥和 CA 的角色不同：

- `client.crt`：客户端证书和公钥身份；
- `client.key`：证明该客户端持有私钥；
- `ca.crt`：验证对端证书的信任锚；
- CSR：证书签名请求，通常不含私钥，但会暴露 subject 和公钥信息。

常用检查：

```bash
openssl x509 -in client.crt -noout -subject -issuer -serial -dates
openssl x509 -in client.crt -noout -pubkey | openssl sha256
openssl pkey -in client.key -pubout | openssl sha256
openssl verify -CAfile ca.crt client.crt
```

两条公钥摘要一致，说明证书与私钥匹配；`openssl verify` 成功，说明证书链能由指定 CA 验证。这两个结论不能互相替代。

## mTLS 常见误区

- 握手成功只证明证书被信任，不代表该证书可以访问所有账户或动作。
- 把通用设备证书复制进固件镜像，会使所有拿到镜像的人共享同一设备身份。
- 服务端在 SIP/SDP 响应中返回明文账号或口令，是应用层敏感数据暴露；TLS 只能防窃听，不能修复“把秘密发给了不该获得它的已认证客户端”。
- `No client certificate CA names sent` 不一定表示服务端不要求客户端证书，应结合握手结果和服务端告警判断。
- 为兼容旧设备降低 OpenSSL 安全级别会启用弱算法，只应在隔离的兼容测试中临时使用，不能作为生产修复。

## IoT 设备应同时按服务端和客户端分析

路由器、网关等设备不只“对外开放 Web 管理页”，也会主动连接配置下发、语音、监控和升级平台。获得设备侧文件读取能力后，应建立出站信任清单：

```text
配置文件 -> 后端地址/协议
证书与密钥 -> 设备身份
启动脚本/进程 -> 连接时序
响应数据 -> 本地凭据、配置或命令
```

这类链条的根因通常是多个边界同时失守：设备 Web 命令注入、镜像内共享私钥、后端仅以证书粗粒度授权，以及控制面返回过量敏感数据。

## 安全测试与修复检查表

### NATS

- 是否强制 TLS 和强凭据；
- 匿名账户、默认账户和弱口令是否禁用；
- 每个服务的 publish/subscribe subject 是否最小化；
- 消息是否直接进入 shell、模板或解释器；
- 管理 subject 是否与普通业务 subject 隔离；
- 持久化消息中是否包含秘密。

### MQTT

- 是否禁用匿名连接并强制 TLS 与独立客户端身份；
- publish 与 subscribe ACL 是否分别限制到必要 topic；
- 普通客户端是否能使用 `#` 或越级使用 `+`；
- retained 或周期消息中是否包含口令、Token、私钥或管理指令；
- 消息泄露后是否同时轮换凭据并排查跨服务复用。

### SIP/mTLS

- 证书链、有效期、EKU、SAN 和吊销策略是否正确；
- 每设备是否使用唯一私钥，并能轮换和撤销；
- mTLS 身份是否映射到精确的账户和动作授权；
- OPTIONS/INVITE/错误响应是否泄露内部状态或凭据；
- SIP Header、SDP 和 `Content-Length` 是否严格解析；
- 旧算法兼容是否被限制在隔离范围。

## 关联笔记

- [Web 与 API 常见漏洞原理](https://github.com/pity11/WebSecAtlas/blob/main/fundamentals/web-api-vulnerability-principles.md)
