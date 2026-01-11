## Ingress / Gateway API：Service 之上的七层流量是怎么走的？

> HTTP / HTTPS 请求是如何被路由到具体服务、具体路径的？

### 01 为什么 Service 解决不了七层流量？

Service 的本质是 四层负载均衡（L4）：

* 只关心 IP + Port
* 不理解 HTTP 路径、域名、Header
* 无法做：
	* **/api 转发到 A 服务**
	* **/admin 转发到 B 服务**
	* example.com / api.example.com 分流

### 02 Ingress：经典的七层入口方案

#### Ingress 是什么？

Ingress 是一组 **七层路由规则声明，本身并不转发流量**。

真正干活的是：Ingress Controller常见实现：

* Nginx
* Ingress
* Traefik
* HAProxy
* Kong

#### Ingress 的核心能力

Ingress 解决三件事：

* 1. 域名路由
* 2. 路径转发
* 3. TLS 终止

示意流程：

```
Client
  ↓
Ingress Controller（Nginx / Traefik）
  ↓
Service
  ↓
Pod
```

#### Ingress 流量到底怎么走？

以 Nginx Ingress 为例：

* 1. 客户端访问 https://api.example.com/v1
* 2. 流量到达 Ingress Controller Pod
* 3. Controller 根据 Ingress 规则匹配：
	* Host
	* Path
* 4. 转发到对应 Service
* 5. Service 再通过 kube-proxy 转到 Pod


一句话：Ingress = 七层规则，Service = 四层转发

### 03 Ingress 的问题：它已经“老了”

Ingress 在大规模场景下逐渐暴露问题：

* API 能力有限（HTTP 为主）
* TCP / UDP 支持不统一
* 扩展依赖 annotation（厂商私有）
* 角色不清晰（入口、监听器、路由混在一起）

于是 Kubernetes 社区推出了新一代标准：

### 04 Gateway API：下一代流量入口标准

Gateway API 不是 Ingress 的简单升级，而是 **重新设计了一整套模型**。

**核心思想：角色解耦**

组件 : 职责

* GatewayClass   网关实现（由谁提供）
* Gateway       实例（监听哪些端口）
* Route    路由规则（HTTP/TCP/UDP）
* Backend   后端 Service


#### Gateway API 的流量路径

```
Client
  ↓
Gateway（监听 80/443）
  ↓
HTTPRoute / TCPRoute
  ↓
Service
  ↓
Pod
```

它明确区分了：

* 谁负责监听端口
* 谁负责路由规则
* 谁负责后端服务

### 05 Ingress vs Gateway API 对比


```
维度   Ingress    Gateway API
协议支持 主要 HTTP  HTTP / TCP / UDP
扩展性  Annotation  原生 API
角色拆分 单一资源  多角色解耦
多团队协作  困难  非常友好
云厂商支持 成熟 正在成为主流
```

结论：

* Ingress：够用、成熟、简单
* Gateway API：未来、规范、可扩展


### 06 一个现实中的最佳实践建议

#### 什么时候用 Ingress？

* 中小规模集群
* 以 HTTP 为主
* 希望快速落地


#### 什么时候用 Gateway API？

* 多租户 / 多团队
* 同时需要 HTTP + TCP
* 对流量治理要求高
* 云厂商或 Service Mesh 环境

### 07 和 Service / kube-proxy 的关系再梳理一次

完整流量链路可以这样理解：

```
Client
  ↓
Ingress / Gateway（七层）
  ↓
Service（四层）
  ↓
kube-proxy（iptables / IPVS）
  ↓
Pod
```

* Ingress / Gateway：决定“转给谁
* ”Service：决定“打到哪个
*  Pod”kube-proxy：真正改写和转发流量


## Kubernetes 生产环境常见故障 Top 20：90% 的事故都逃不出这张清单

### 01 Pod 启动与运行类故障（最常见）

**1） Pod 一直 Pending**

常见原因：

- 节点资源不足（CPU / Memory）
- NodeSelector / Affinity 不匹配
- 存储 PVC 未绑定
- Taint 未被 Toleration 容忍

👉 第一时间看：kubectl describe pod

**2）ImagePullBackOff / ErrImagePull**

- 镜像名或 tag 错误
- 私有仓库未配置 ImagePullSecret
- 节点无法访问镜像仓库

👉 看事件比看日志更重要

**3） CrashLoopBackOff**

常见原因：

- 应用启动即崩溃
- 配置文件错误
- 端口冲突
- 探针配置过严

👉 重点关注：

- 容器启动日志
- restartCount

**4） Readiness 一直失败，Pod 不接流量**

常见原因：

- 探针路径 / 端口错误
- 应用启动慢
- 依赖服务未就绪

👉 表现是：Pod Running，但 Service 无流量

**5） Liveness 探针导致频繁重启**

典型问题：

* 探针当成健康检查用
* 网络抖动被误判为“应用挂了”

👉 生产环境慎用激进 liveness


### 02 节点与资源类故障

**6） Node NotReady**

常见原因：

* kubelet 异常
* 磁盘满
* 内存耗尽
* 网络异常

👉 看 node 条件和 kubelet 日志


**7） Pod 被 OOMKilled**


原因：

* 内存 limit 太小
* 应用存在内存泄漏


👉 OOMKilled ≠ Kubernetes 问题，是资源设计问题


**8） CPU 被限流，应用变慢**

典型表现：

* CPU limit 设置过小
* 实际负载高峰超出预期

👉 CPU limit 是“硬上限”，别随便设


**9） 磁盘压力（DiskPressure）**

常见来源：

* 容器日志未轮转
* emptyDir 使用不当
* 镜像垃圾未清理

👉 很多 Node NotReady 的根因在磁盘


### 03 网络与 Service 类故障

**10） Service 能访问，但偶尔超时**

常见原因：

* kube-proxy 规则异常
* Pod readiness 抖动
* Conntrack 表满

👉 网络问题一定要结合节点看


**11） Pod 之间无法通信**

可能原因：

* CNI 异常
* NetworkPolicy 拦截
* MTU / IPIP / VXLAN 问题

👉 先确认：是不是“被策略挡住了”

**12） NodePort 能访问，ClusterIP 不行**

多见于：

* kube-proxy 异常
* iptables / IPVS 规则损坏

👉 重启 kube-proxy 有时就是答案

**13） Ingress 访问异常**

常见问题：

* Service 端口写错
* Ingress Controller 异常
* TLS 证书错误

👉 Ingress 问题先看 Controller 日志

### 04 发布、升级与控制器类故障

**14） 滚动升级卡住**

原因：

* readiness 一直失败
* maxUnavailable 配置不合理

👉 Deployment 不会“强行升级”

**15） 升级后业务异常但 Pod 正常**

典型原因：

* 配置变更问题
* 依赖服务版本不兼容

👉 Kubernetes 只管“活着”，不管“对不对”

**16） Replica 数量不对**

可能原因：

* HPA 生效
* 手动改了 ReplicaSet
* Controller 冲突

👉 永远以 Deployment 为准

### 05 权限、安全与配置类故障

**17） Pod 无法访问 API Server**

原因：

* ServiceAccount 权限不足
* Token 未挂载
* RBAC 拒绝

**👉 看错误信息里有没有 Forbidden**

**18） 配置更新了但 Pod 没生效**

常见于：

* ConfigMap / Secret 未重启 Pod
* 使用 env 注入而非 volume

👉 配置 ≠ 自动热更新

**19） Job / CronJob 不执行**

原因：

* 并发策略限制
* 历史 Job 未清理
* 时间同步问题

👉 看 Job 状态，而不是 Pod

**20） 集群“看起来都正常”，但业务不可用**

最危险的一类问题：

* 监控缺失
* 没有 SLI / SLO
* 只看 Pod 状态，不看业务指标

👉 Kubernetes 健康 ≠ 业务健康