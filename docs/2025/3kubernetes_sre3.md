

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