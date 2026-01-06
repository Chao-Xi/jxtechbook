# SRE Interview Questions and Answers - Chinese

### 1. What is the difference between SRE and DevOps?

**SRE 和 DevOps 有什么区别？**

**DevOps 是一种文化理念和实践方式，目标是把软件开发（Dev）和运维（Ops）结合在一起。**

它的目标是缩短系统开发周期，在保证软件质量的同时，实现持续交付（continuous delivery）。

**SRE 是由 Google 提出的理念，把 software engineering 的方法用于 infrastructure 和 operations 问题。**

SRE 通过可衡量的 Service Level Objectives（SLOs）来保障系统可靠性，关注减少 toil、自动化，并通过 error budgets 在系统稳定性和功能迭代速度之间做平衡。

**SRE 特点**

- 起源：Google

- 关注重点：Reliability、automation、monitoring

- 核心原则：SLOs、SLIs、Error Budget

- 工具方式：使用 software engineering 工具

**DevOps 特点**

- 一种文化运动

- 强调 CI/CD 和团队协作

- 自动化、文化、共同责任

- 重点是 CI/CD pipeline 和版本控制


### 2. What are SLIs, SLOs, and SLAs? How do they relate?

**什么是 SLIs、SLOs 和 SLAs？它们之间有什么关系？**

- SLI（Service Level Indicator）
	- 用来量化衡量服务表现的指标
	- 例如：latency、uptime、request success rate

- SLO（Service Level Objective）
	- 针对某个 SLI 设定的目标值
	- 例如：30 天内 99.9% 的请求必须成功

- SLA（Service Level Agreement）
	- 服务提供方和客户之间的正式协议，包含未达成 SLO 时的赔偿或处罚

关系说明：

- SLIs → 用来衡量服务表现

- SLOs → 基于 SLI 设定的目标

- SLAs → 基于 SLO 的对外合同


### 3. What is an Error Budget and how do you use it?

**什么是 Error Budget？如何使用？**

**Error Budget 是在不违反 SLO 的前提下，系统允许出现的错误或宕机额度。**

**它体现了系统稳定性和创新速度之间的平衡。**

如果 SLO 是 30 天内 99.9% uptime：

* 总分钟数：43,200

* 允许宕机时间：0.1% × 43,200 = 43.2 分钟

使用方式：

* **如果 Error Budget 用完 → 暂停功能发布，优先修复稳定性问题**

* **如果 Error Budget 还有余量 → 可以承担更多风险，比如发布新功能或实验**


Error Budget 让 Dev 和 Ops 能基于数据做决策。


### 4. How do you implement monitoring for a distributed system?

#### 如何为分布式系统实现 monitoring？

**分布式系统的 monitoring 需要收集 metrics、logs 和 traces。**

实现方式包括：

* **Metric Collection**
	* 使用 Prometheus、Datadog、CloudWatch 收集 CPU、memory、latency 等指标
* **Logging**
	* 使用 ELK、Fluentd、Loki 进行日志集中管理
* **Tracing**
	* 使用 OpenTelemetry、Jaeger、Zipkin 实现 distributed tracing
* **Dashboards**
	* 用 Grafana 或 Datadog 可视化关键指标
* **Alerting**
* 	为 SLO 违规、资源耗尽、异常行为设置告警

最佳实践：

* 明确关键 SLIs（如 latency、availability）
* 同时使用 black-box 和 white-box monitoring
* 告警关注“症状”，而不是“原因”


### 5. What are common causes of high latency and how to troubleshoot?

#### Web 应用 latency 高的常见原因及排查方法

**常见原因：**

* **数据库慢查询**
* **应用层瓶颈（循环、锁）**
* **网络问题（丢包、拥塞）**
* 上游服务响应慢

**排查步骤：**

1. **使用 APM 工具找慢接口**
2. **查看 logs 是否有 timeout 或 retry**
3. **查看数据库 slow log**
4. 使用 traceroute、ping、mtr 检查网络
5. 用 JMeter、Locust 做压测

**常见优化方式：**

* **使用 Redis 做缓存**
* 添加数据库索引
* **优化代码逻辑**
* 使用 load balancer 分流

### **6. Describe a runbook.**

#### 什么是 runbook？好的 runbook 应该包含什么？

runbook 是一份处理已知问题或事故的操作文档。

应包含：

* 问题描述
* 触发条件
* 处理步骤
* 修复后的验证方式
* 回滚方案
* 升级或负责人信息

示例：Disk Full on Web Server

* SSH 登录服务器
* 使用 du -sh * 找大文件
* 清理或归档日志
* 验证磁盘空间

**好处**：

* 缩短 MTTR
* 减少 on-call 压力
* 保证处理方式一致


### 7. What is toil in SRE?

#### SRE 中的 toil 是什么？如何减少？

Toil 指重复、手工、可自动化、且不创造长期价值的工作。

**示例：**

* 重启服务
* 手动轮转日志
* 处理重复工单

**减少方法：**

* 自动化（Ansible、Terraform）
* Self-healing（Kubernetes 自动重启）
* 优化告警，减少噪音
* 使用 Bot（如 Slack bot）

目标：toil 不超过 SRE 工作量的 50%

### 8. How do you design high availability in cloud?

**如何在云环境中设计高可用架构？**

**核心原则：**

* Multi-Zone（多 AZ）
* Load Balancer
* Auto-scaling
* Stateless 服务
* 使用 Managed DB
* Health checks

**AWS 示例：**

* ALB + 多 AZ EC2 Auto Scaling Group
* Aurora 跨 AZ failover
* Route53 做 DNS 切换

**目标是：resilient、scalable、self-healing**


### 9. Horizontal vs Vertical Scaling

**水平扩展和垂直扩展的区别**

* **Horizontal scaling**
	* 加机器，更适合云环境，容错能力强

* **Vertical scaling**
	* 升级单机配置，适合不支持分布式的应用


### 10. Explain blue-green deployment

**什么是 blue-green deployment？**

使用两个环境：

* Blue：当前生产环境
* Green：新版本环境

流程：

1. 部署到 Green
1. 测试
1. 切流量
1. 出问题可快速回滚

**优点：**

1. 零停机
1. 回滚简单
1. 发布风险低	


### 11. How do you ensure fault tolerance in a microservices architecture?

#### 如何在 microservices 架构中保证 fault tolerance？

**fault tolerance 指的是：即使部分组件失败，系统仍能继续运行。**

**常用技术手段：**

* **Retries with exponential backoff**
	* 失败后重试，但重试间隔逐步变长，避免雪崩

* Circuit Breakers（如 Hystrix、Resilience4j）
	* 下游服务异常时，快速失败，避免拖垮整个系统
	
* **Bulkheads**
	* 将资源隔离，防止一个服务影响其他服务
	
* **Fallback mechanisms**
	* 返回默认值或缓存数据

* **Health checks**
	* 将不健康实例从 load balancer 或 service mesh 中移除

最佳实践：

* 独立监控每个 microservice
* 使用 service mesh（如 Istio）
* 使用 Kubernetes 提供的自愈能力


### 12. What is chaos engineering, and why is it important?

#### 什么是 chaos engineering？为什么重要？

**Chaos engineering 是主动向系统注入故障，用来测试系统稳定性的方法。**

常用工具：

* Chaos Monkey
* Gremlin
* Litmus

目的：

* 找出系统薄弱点
* 验证 monitoring 和 alerting 是否有效
* 提高对生产系统的信心

常见场景：

* 随机终止实例
* 阻断服务间网络
* 人为增加 latency

结果是：通过提前发现问题，让系统更可靠

### 13. How do you handle noisy alerts?

#### **如何处理 noisy alerts（告警噪音）？**

**noisy alerts 会导致 alert fatigue，让工程师忽略真正重要的问题。**

解决方法：

* **使用去重和限频（deduplication、rate-limiting**）
* **调整告警阈值**
* **按严重程度分级（P1、P2）**
* **维护期间静默告警**
* **使用 anomaly detection**
* **只对“用户影响”告警，而不是底层原因**


### 14. How does SRE measure system reliability?

#### SRE 如何衡量系统可靠性？

**主要通过 SLIs 和 SLOs 来衡量：**

* Availability：如 99.99%
* Latency：如 95% 请求 < 200ms
* Durability：不丢数据
* Error rate：失败请求 < 0.1%

**常用工具**：

* Prometheus
* CloudWatch
* Datadog
* New Relic

**系统可靠性 = 实际表现与 SLO 的接近程度**


### 15. What are MTTR, MTBF, and MTTD?

#### 什么是 MTTR、MTBF 和 MTTD？

* MTTR（Mean Time to Repair）：修复故障平均时间
* MTBF（Mean Time Between Failures）：两次故障之间的平均时间
* MTTD（Mean Time to Detect）：发现故障所需时间

**为什么重要？**

* 衡量运维效率
* 观察系统稳定性趋势
* 指导 incident response 改进

👉 低 MTTR / MTTD + 高 MTBF = 好系统

### 16. Canary vs Rolling deployment

**Canary 和 Rolling deployment 的区别**

* **Canary deployment**
	* 先给少量用户发布，用于风险检测
	
* **Rolling deployment**
	* 分批替换实例，适合小改动

👉 Canary 更安全，Rolling 更简单


### 17. How does Kubernetes help SRE?

**Kubernetes 如何帮助 SRE？**

Kubernetes 提供了大量提升可靠性的能力：

* Auto-scaling
* Self-healing（Pod 自动重启）
* Rolling updates
* Health checks（liveness / readiness）
* Namespaces & RBAC

SRE 使用 Kubernetes 实现自动化运维和稳定性保障。

###18. What is a blameless postmortem?

#### **什么是 blameless postmortem？**

blameless postmortem 强调“不追责个人”，只关注系统问题。

**好处**：

* 鼓励透明沟通
* 建立安全的团队文化
* 找到系统性问题

**通常包含：**

* 事件摘要
* 时间线
* Root cause
* 改进项
* 经验总结


### 19. How do you scale databases in high traffic systems?

#### 高并发系统中如何扩展数据库？

**Vertical scaling：**

* 增加 CPU / RAM（有上限）

**Horizontal scaling：**

* Sharding
* Read Replicas
* Caching（Redis / Memcached）
* Connection pooling

**常配合 ProxySQL 或 Aurora 使用。**


### 20. What is service mesh and observability?

#### **什么是 service mesh？它如何帮助 observability？**

service mesh 是一层管理服务间通信的基础设施层。

常见实现：

* Istio
* Linkerd
* Consul

**能力：**

* Traffic management
* Security（mTLS）
* Observability
* Fault injection

**在 observability 方面：**

* 自动收集 metrics 和 traces
* 提供服务级别可观测性


### 21. How do you reduce MTTR?

如何降低 MTTR？

* 强监控和实时告警
* 清晰的 runbook
* Auto-remediation
* Incident drill 演练
* 支持快速 rollback 的 CI/CD

### 22. Describe SRE on-call process

#### 典型的 SRE on-call 流程

* Monitoring 发现问题
* PagerDuty / Opsgenie 通知
* 查看 logs 和 dashboard
* 缓解问题（restart、scale）
* 必要时升级
* 事后 postmortem


### 23. How do you manage secrets in production?

#### 如何管理生产环境的 secrets？

**工具：**

* Vault
* AWS Secrets Manager
* Azure Key Vault
* Kubernetes Secrets

**最佳实践：**

* 加密存储和传输
* 定期轮换
* 使用 IAM 控制权限
* 不在代码中写死 secrets


### 25. How do you conduct capacity planning?

#### 如何做 capacity planning？

* 收集历史 metrics
* 预测增长趋势
* 压测
* 预留 30% buffer
* 定期复盘


### 26. Monitoring vs Observability

#### Monitoring 和 Observability 的区别

* Monitoring：已知问题的告警
* Observability：通过 metrics / logs / traces 理解系统内部状态

三大支柱：

* Metrics
* Logs
* Traces


### 27. How do you ensure zero-downtime deployments?

#### 如何实现零停机发布？

* Blue-Green 或 Canary
* Load balancer 健康检查
* Readiness / Liveness probes
* 滚动发布参数控制


### 28. What is IaC?

#### 什么是 Infrastructure as Code？

**IaC 是用代码管理基础设施。**

工具：

* Terraform
* Pulumi
* AWS CDK
* Ansible

好处：

* 可追溯
* 可复现
* 易回滚
* 自动化

### 29. What is GitOps?

什么是 GitOps？

GitOps 使用 Git 作为唯一真实来源。

流程：

* 配置写入 Git
* Agent（如 ArgoCD）同步
* 自动修正偏差

### 30. How do you test disaster recovery?

**如何测试 disaster recovery？**

* 定义 RPO / RTO
* 模拟 region 故障
* 演练 DB failover
* 验证备份恢复
* 使用 chaos engineering


### 31. What is rate limiting?

**什么是 rate limiting？**

限制请求数量，防止系统被压垮。

工具：

* NGINX
* Envoy
* API Gateway

### 32. How do you secure CI/CD?

**如何保证 CI/CD 安全？**

* 静态扫描
* Secrets 扫描
* 制品签名
* 最小权限
* MFA


### 33. Symptom vs Cause alerts

**症状告警 vs 原因告警**

* Symptom：用户可感知问题
* Cause：底层资源问题

SRE 更偏向 Symptom。


### 34. Cascading failures

#### 如何处理 cascading failures？

* Circuit breaker
* Timeout + retry
* Bulkhead
* 监控依赖健康

### 35. Load shedding

什么是 load shedding？

**高负载时丢弃低优先级请求，保护系统。**


### 36. Distributed tracing

**什么是 distributed tracing？**

跨服务追踪请求路径。

工具：OpenTelemetry、Jaeger、Zipkin

### 37. Principle of least privilege

**最小权限原则**

只给完成任务所需的权限。

### 38. Configuration drift

**什么是 configuration drift？**

实际系统状态与声明状态不一致。

解决方案：

* IaC
* GitOps
* Drift detection

### 39. Alert fatigue

**什么是 alert fatigue？**

告警过多导致工程师忽视告警。

**解决：减少、分级、定期复盘**。

### 40. Incident lifecycle

**Incident 生命周期**

* Detection
* Response
* Resolution
* Postmortem
* Action Items

### 41. High availability system

**如何设计高可用系统？**

* 冗余
* 自动 failover
* 健康检查
* 跨地域
* Stateless

### 43. Handling traffic spikes

**如何应对突发流量？**

* Auto-scaling
* Queue
* CDN
* Cache
* 预热服务

### 44. Error budget enforcement

**如何使用和约束 error budget？**

* 跟踪可靠性
* 用完就暂停发布
* 与发布节奏绑定

### 45. Ephemeral environments

什么是 ephemeral environment？

**临时、可销毁的测试环境。**

### 46. Monitoring service dependencies

**如何监控服务依赖？**

* Distributed tracing
* Service mesh
* Dependency grap


### 49. Auto-remediation

#### 什么是 auto-remediation？

自动处理故障。

示例：

* 自动重启 Pod
* 自动扩容
* 自动替换实例


### 50. Preparing for major launches

**SRE 如何准备大型发布？**

压测

* Chaos 演练
* 容量规划
* 回滚方案
* 监控确认
* 强化 on-call