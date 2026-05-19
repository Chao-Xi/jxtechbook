# 2026 DevOps Interview

## 1 System Software Engineer / Platform Operations 面试题库（Python + Linux Shell + 运维平台）

下面这份偏向“高级系统工程师 / 平台运维工程师（Platform Operations）”方向，重点覆盖：

* Linux 系统
* Shell Script
* Python 自动化
* 系统设计
* Troubleshooting
* DevOps / CI/CD
* 网络
* 监控
* 高可用
* 实际场景题

适合：

* Senior System Engineer
* SRE
* Platform Operations
* Infrastructure Engineer
* DevOps Engineer

---



我目前主要专注于 Linux 平台运维、自动化和系统稳定性方向，日常工作会使用 Python 和 Shell Script 来做：

* 自动化运维
* 日志分析
* 系统巡检
* CI/CD 流程
* 服务部署
* 故障排查

我比较熟悉 Linux 系统管理，包括：

* systemd
* networking
* process management
* performance tuning
* shell scripting

在自动化方面，我经常使用 Python 编写：

* API integration
* 自动化脚本
* deployment tools
* monitoring scripts
* data processing tools

另外我也有处理 production issue 的经验，例如：

* CPU spike
* memory leak
* disk full
* service crash
* network latency

我比较擅长的问题是：
“如何把重复性的系统操作自动化，并提高平台稳定性”。


### Q2：Linux 中查看 CPU 使用率的命令有哪些？

常见命令

```bash
top
htop
mpstat
sar
vmstat
```

#### top

实时查看：

* CPU
* memory
* load average
* process status

#### vmstat

适合观察：

* context switch
* IO wait
* system performance

```bash
vmstat 1
```

#### mpstat

查看每个 CPU core：

```bash
mpstat -P ALL 1
```

### load average 是什么？

很多人回答不完整，这是高级题。

标准回答

load average 表示：

<mark>**系统中正在运行或者等待 CPU 的任务数量平均值**</mark>。

三个值：

```bash
uptime
```

输出：

```bash
load average: 1.20, 0.80, 0.50
```

分别表示：

* 1分钟
* 5分钟
* 15分钟

平均负载。

面试加分点

load average 不仅包含 CPU running process：

还包括：

* uninterruptible sleep（D state）
* IO wait

所以：

> load 很高不一定 CPU 很高，
> 可能是磁盘 IO 卡住。


### 如何查看哪个进程占用最多内存？

```bash
ps aux --sort=-%mem | head
```

或者：

```bash
top
```

### 如何查找端口被哪个进程占用？

```bash
lsof -i :8080
```

或者：

```bash
netstat -tulpn
```

现代 Linux：

```bash
ss -tulpn
```

### 什么是 zombie process？


标准答案

Zombie process：

> 子进程已经结束，
> 但父进程没有调用 wait() 回收。

因此：

* process entry 仍存在
* PID 未释放

如何处理？

找到 parent PID：

```bash
ps -el | grep Z
```

然后：

* restart parent process
* 或 kill parent

init/systemd 会接管回收。

### Q7：Shell Script 和 Python 什么时候分别使用？

Shell script 适合：

* 系统命令组合
* 文件处理
* deployment script
* cron jobs
* quick automation

Python 更适合：

* 复杂逻辑
* API integration
* data processing
* error handling
* reusable tools

一般：

* 简单系统操作 → Shell
* 复杂自动化 → Python

### 如何在 shell script 中检查命令是否执行成功？

```bash
if [ $? -eq 0 ]; then
    echo "success"
else
    echo "failed"
fi
```

更好的写法：

```bash
if systemctl restart nginx; then
    echo "success"
else
    echo "failed"
fi
```

### Shell Script 如何处理参数？

```bash
$1
$2
$#
$@
```

例子：

```bash
#!/bin/bash

echo "first arg: $1"
echo "total args: $#"
```

### 如何定时执行脚本？

**cron**

```bash
crontab -e
```

例子：

```bash
*/5 * * * * /opt/scripts/check.sh
```

每5分钟执行一次。

### 为什么运维工程师需要 Python？

标准答案

Python 可以用于：

* 自动化运维
* 配置管理
* API 调用
* 日志分析
* 监控
* deployment automation
* cloud integration

Python 的优点：

* 开发快
* 库丰富
* 可维护性好


### 你常用哪些 Python 库？


系统/自动化：

```python
subprocess
os
sys
shutil
pathlib
```

API：

```python
requests
```

并发：

```python
threading
concurrent.futures
asyncio
```

日志：

```python
logging
```

数据：

```python
json
yaml
pandas
```

SSH：

```python
paramiko
```

### subprocess 和 os.system 的区别？

### 推荐答案

`subprocess` 更安全、更强大。

例如：

```python
import subprocess

result = subprocess.run(
    ["ls", "-l"],
    capture_output=True,
    text=True
)

print(result.stdout)
```

优势：

* 获取 stdout/stderr
* timeout
* 更安全
* 不依赖 shell

### 如何在 Python 中处理异常？

```python
try:
    result = 10 / 0
except ZeroDivisionError as e:
    print(e)
finally:
    print("done")
```

### 如何写一个日志系统？

使用 logging：

```python
import logging

logging.basicConfig(
    filename="app.log",
    level=logging.INFO,
    format="%(asctime)s %(levelname)s %(message)s"
)

logging.info("service started")
```

### Q16：如何设计一个自动化部署系统？

高级回答结构

包括：

1. Source Control
2. CI/CD Pipeline
3. Artifact Repository
4. Deployment Strategy
5. Rollback
6. Monitoring
7. Alerting

示例

开发提交代码：

→ Jenkins/GitLab CI

→ 自动测试

→ build artifact

→ deploy to staging

→ approval

→ production rollout

加分关键词

* blue-green deployment
* canary deployment
* rollback
* idempotent
* health check

### 什么是 idempotent？

高频高级题。

Idempotent：

> 执行多次结果一致。

例如：

```bash
mkdir -p /data/logs
```

重复执行不会出错。

配置管理工具：

* Ansible
* Puppet

都强调 idempotency。


### 如何排查 Linux 服务器变慢？

标准排查流程

**1 看 load**

```bash
uptime
```

**2 看 CPU**

```bash
top
mpstat
```

**3 看 memory**

```bash
free -m
```

是否：

* swap usage 高
* OOM

**4 看 IO**

```bash
iostat
iotop
```

**5 看 network**

```bash
ss -s
sar -n DEV
```

**6 看 logs**

```bash
journalctl
dmesg
```

### TCP 和 UDP 区别？

| TCP                 | UDP            |
| ------------------- | -------------- |
| connection-oriented | connectionless |
| reliable            | unreliable     |
| ordered             | unordered      |
| slower              | faster         |


TCP 特点

* retransmission
* ACK
* flow control

### 如何检查服务器网络连通性？

```bash
ping
traceroute
telnet
nc
curl
```

检查端口：

```bash
nc -zv host 443
```

### 你如何设计监控系统？

**基础设施**

* CPU
* memory
* disk
* network

**应用**

* response time
* error rate
* throughput

### 日志

集中式日志：

* Elasticsearch
* Logstash
* Kibana

**告警原则**

避免：

* alert storm

需要：

* meaningful alerts
* threshold tuning

### 什么是 Prometheus？

Prometheus 是：

* metrics monitoring system
* time-series database

特点：

* pull model
* exporter
* PromQL


### 生产环境 CPU 100% 怎么办？

推荐回答流程

1. top 找进程
2. pidstat 看线程
3. 看是否：

   * 死循环
   * GC
   * 高并发
4. 查看 logs
5. 分析 recent deployment
6. 必要时：

   * 限流
   * rollback
   * restart

### 磁盘满了怎么办？

排查：

```bash
df -h
du -sh /*
```

常见原因

* log 未 rotate
* core dump
* tmp 文件
* docker image

加分项

```bash
lsof | grep deleted
```

查找：

> 已删除但仍被进程占用的文件。


### 服务无法启动如何排查？

标准流程

1. systemctl status
2. journalctl logs
3. config syntax
4. port conflict
5. permission
6. dependency service
7. resource issue


### 如何保证生产环境稳定性？


包括：

* monitoring
* alerting
* automation
* rollback
* redundancy
* capacity planning
* change management
* incident response


### Troubleshooting

重点：

* CPU high
* memory leak
* disk full
* service down
* network issue


## 2 事件驱动架构（EDA）与发布/订阅（Pub/Sub）

### 2-1 Event-Driven Architecture / Pub-Sub 面试题（高级系统工程师）

这部分是很多 Senior Platform Operations / System Software Engineer 的高级考点。

面试官通常想确认你是否真正理解：

* 分布式系统
* 异步架构
* 消息队列
* 系统解耦
* 高可用
* 消息可靠性
* 可扩展性
* 生产问题排查

重点平台：

* Amazon Web Services 的 Amazon SNS / Amazon SQS
* Google Cloud 的 Google Cloud Pub/Sub
* Microsoft 的 Azure Service Bus

### 2-2 什么是 Event-Driven Architecture（EDA）？


Event-Driven Architecture 是一种：

> 通过事件（events）进行服务之间通信的架构模式。

系统中的组件：

* **Producer（事件生产者）**
* **Broker / Message Bus**
* **Consumer（事件消费者）**

彼此解耦。

举例

电商系统：

用户下单后：

```text
Order Service
    ↓
Publish Event: OrderCreated
    ↓
Message Broker
    ↓
Inventory Service
Payment Service
Notification Service
Shipping Service
```

每个服务独立处理自己的逻辑。


EDA 的核心价值：

* **loose coupling**
* **scalability**
* **asynchronous processing**
* **fault isolation**
* **extensibility**

### 2-3 什么是 Pub/Sub Pattern？

Pub/Sub（Publish/Subscribe）：

Publisher 发布消息，

Subscriber 订阅消息。

Publisher 不需要知道：

* 谁消费
* 消费多少次
* 消费逻辑

**解耦**

Producer 和 Consumer 独立。

**异步**

不阻塞主流程。

**可扩展**

增加 consumer 不影响 producer。

### 2-4 Queue 和 Pub/Sub 的区别？

#### Queue（点对点）

例如：

Amazon SQS

**一个消息：**

通常只被一个 consumer 处理。

```text
Producer → Queue → One Consumer
```

#### Pub/Sub（广播）

例如：

Amazon SNS

一个消息：

可以被多个 subscriber 接收。

```text
Publisher
   ↓
Topic
 ↙ ↓ ↘
A  B  C
```

####  高级回答

很多系统：

SNS + SQS 组合使用。

例如：

```text
SNS Topic
   ↓
多个 SQS Queue
   ↓
不同服务消费
```

这样：

* fan-out
* decoupling
* **buffering**

### 2-5 SNS 和 SQS 如何配合使用？

#### 标准回答

**SNS：负责广播**

**SQS：负责消息持久化和异步消费**。

典型架构

```text
Producer
   ↓
SNS Topic
   ↓
SQS Queue A
SQS Queue B
SQS Queue C
```

不同服务：

* **independently consume**
* **retry independently**
* **scale independently**

#### 这样设计的优点：

* 解耦
* backpressure buffering
* retry isolation
* failure isolation


### 2-6 什么是 Dead Letter Queue（DLQ）？

**DLQ： 用于存放消费失败多次的消息**

例如：

```text
Main Queue
   ↓ failed 5 times
DLQ
```

使用场景

避免：

* poison message 无限重试
* 阻塞正常消息

**DLQ 的价值**：

* troubleshooting
* replay
* operational visibility


### 2-7 什么是 Visibility Timeout？

**Consumer 读取消息后： 消息不会立即删除**。

**而是：暂时隐藏**。

如果：

**Consumer 成功处理**：

```text
DeleteMessage
```

否则：

timeout 后重新出现。

防止：

* consumer crash
* message loss

**Visibility timeout 需要： 匹配业务处理时间**。

- 太短： 会重复消费。

- 太长： 失败恢复慢。

### 2-8 如何保证消息不丢失？

#### Durable Queue

例如：

* SQS
* Service Bus
* Pub/Sub durable subscription


- Acknowledgement: consumer 成功后才 ack。
- Retry: 失败自动重试。
- DLQ: 隔离异常消息。
- Multi-AZ / replication: 云服务高可用。

#### Idempotent Consumer

即使重复消费： 结果也正确。

### 2-9 Google Pub/Sub 的 pull 和 push subscription 区别？

**Pull**

consumer 主动拉取：

```text
Consumer → Pull Message
```

优点：

* 控制消费速率
* backpressure handling

**Push**

Pub/Sub 主动推送：

```text
Pub/Sub → HTTP Endpoint
```

优点：

* 简单
* 实时

缺点：

* consumer 不易控流

### 2-10 什么是 message ordering？

**保证消息按顺序消费**。

例如：

```text
OrderCreated
OrderPaid
OrderShipped
```

- 顺序保证通常会：降低吞吐量。

- 因为：需要 serialization。

### 2-11 Queue 和 Topic 在 Azure Service Bus 中的区别？

- **Queue 单消费者模型**。

- **Topic: 发布订阅模型**。

多个 subscription：

```text
Topic
  ↓
Sub A
Sub B
Sub C
```

### 2-12 什么是 Peek-Lock？

类似 SQS visibility timeout。

- **message： 先 lock**。

- **consumer：处理成功后 complete**。

否则：

**lock timeout 后重新投递。**


### 2-13 At-most-once、At-least-once、Exactly-once 区别？

<mark>**At-most-once** 最多一次</mark>

- 可能丢消息。
- 不重复。

<mark>**At-least-once 至少一次。**</mark>

- 不丢消息。
- 可能重复。

</mark>**Exactly-once 精确一次。**</mark>

- 最理想。
- 但实现复杂且昂贵。


现实中：

很多系统实际采用：

> At-least-once + Idempotency

因为：

**Exactly-once 成本高**。

### 什么是 Idempotency？

支付：

**不能因为消息重复： 扣款两次**。

**唯一 transaction ID**

```text
payment_id
```

已处理过则忽略。

**数据库唯一约束**

```sql
UNIQUE(transaction_id)
```

### 如何处理重复消息？

1. Idempotency key
2. Deduplication
3. Stateful processing
4. Database constraints

### 如何处理消息积压（backlog）？

**1. consumer 太慢？**

检查：

* CPU
* DB latency
* downstream API


**2. producer 突增？** traffic spike。

**解决方案**

- scale out consumers
- batch processing
- optimize downstream
- increase partitions

### 如何设计一个高可用异步系统？

1. Producer：发布事件。
2. Durable Broker：
    * Amazon SQS
    * Google Cloud Pub/Sub
3. **Retry + DLQ: 失败隔离**。
4. **Horizontal Scaling: consumer auto-scaling**。
5. Monitoring
  * queue depth
  * processing latency
  * error rate

6. Idempotency: 避免重复副作用。
7. Tracing: distributed tracing。
	* Jaeger
  * OpenTelemetry

- **Reliability** retry / DLQ / idempotency
- **Scalability**: partitioning / consumer group / horizontal scaling
- **Resilience**: failure isolation / backpressure / circuit breaker
- **Observability**: tracing / metrics /structured logging


## Python + Linux Shell Scripting + Automation + System Administration + Troubleshooting

* Linux 基础是否扎实
* Python 自动化能力
* Shell scripting 能力
* Troubleshooting 思维
* Production 经验
* Reliability / Scalability 思维

### Subprocess 和 os.system 的区别？

**os.system**

```python
import os
os.system("ls -l")
```

缺点：

* 不容易获取输出
* 错误处理弱
* 安全性较差

**subprocess（推荐）**

```python
import subprocess

result = subprocess.run(
    ["ls", "-l"],
    capture_output=True,
    text=True
)

print(result.stdout)
```

subprocess 的优势：

* 获取 stdout/stderr
* timeout control
* 更安全
* 不依赖 shell
* 更适合 production automation

### 如何处理 Python 异常？


```python
try:
    do_work()
except ValueError as e:
    print(e)
finally:
    cleanup()
```

生产环境中：

不能：

```python
except:
    pass
```
**specific exception**

```python
except FileNotFoundError:
```

**logging**

```python
logging.exception()
```

**retry**

适用于 transient failure。

### 什么是 GIL

GIL：

Global Interpreter Lock。

CPython 中：

同一时刻：

**只有一个线程执行 Python bytecode。**

影响

- CPU-bound：
- 多线程不能真正并行。
- IO-bound：

**IO-bound**

* threading
* asyncio

**CPU-bound**

* multiprocessing
* native extensions

### threading 和 multiprocessing 区别？

**threading** 共享内存。

适合：

IO-heavy task。

例如：

* API calls
* file operations

**multiprocessing** 多个独立进程。

适合：

- CPU-intensive tasks。
- multiprocessing： 绕过 GIL。

但：

* memory cost 更高
* IPC 更复杂

### asyncio 适合什么场景？

适合：

* 高并发 IO
* 网络服务
* API aggregation
* async message processing


```python
import asyncio

async def worker():
    await asyncio.sleep(1)

asyncio.run(worker())
```


asyncio：不是多线程。而是： cooperative multitasking。

### 如何写一个生产级 logging？


```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s %(levelname)s %(message)s",
    filename="app.log"
)
```

生产环境-需要：

* log rotation
* structured logging
* **correlation ID**
* **centralized logging**

例如：

* Elasticsearch
* Fluentd
* Splunk

### 如何实现 retry？


```python
import time

for retry in range(5):
    try:
        connect()
        break
    except Exception:
        time.sleep(2 ** retry)
```


### Shell Script 和 Python 如何选择？

Shell：

适合：

* 系统命令组合
* quick automation
* cron jobs
* deployment scripts

Python：

适合：

* 复杂逻辑
* API integration
* maintainable automation
* large-scale tools

### 如何检查 shell command 是否执行成功？


```bash
if [ $? -eq 0 ]; then
    echo success
fi
```

更好的写法

```bash
if systemctl restart nginx; then
    echo success
else
    echo failed
fi
```

### set -e 和 set -u 是什么？

`set -e` command fail 时： script 立即退出。

`set -u` 使用未定义变量时报错。


```bash
set -euo pipefail
```

pipefail

**pipeline 任一命令失败：整体失败。**


### grep、awk、sed 区别？

**Grep 查找文本。**

**Awk 文本分析。**

```bash
awk '{print $1}'
```
**Sed 文本替换。**

```bash
sed 's/old/new/g'
```


### **如何监控日志实时变化？**

```bash
tail -f app.log
```

### **如何找出大文件？**

```bash
find / -type f -size +1G
```

### 如何统计某 IP 出现次数？

```bash
awk '{print $1}' access.log | sort | uniq -c
```

### 如何排查 Linux 系统变慢？

**1. 查看 load**

```bash
uptime
```

**2. CPU**

```bash
top
mpstat
```

**3. memory**

```bash
free -m
```

检查：

* swap
* OOM

**4. IO**

```bash
iostat
iotop
```

**5. network**

```bash
ss -s
sar -n DEV
```
**6. logs**

```bash
journalctl
dmesg
```

### load average 是什么？

<mark>**系统运行队列平均长度。**</mark>

包括：

* runnable process
* uninterruptible sleep（IO wait）


**load 高： 不一定 CPU 高**。

可能：

* disk IO issue
* NFS hang

## 什么是 zombie process？


**子进程结束： 父进程未 wait()。 process entry 未释放。**

```bash
ps -el | grep Z
```

* restart parent
* kill parent


### 如何查看端口占用？

```bash
ss -tulpn
```

或者：

```bash
lsof -i :8080
```

### 如何检查磁盘 IO 性能？

**常用命令**

```bash
iostat
iotop
vmstat
```

重点关注：

* await
* util
* queue depth


### 生产环境 CPU 100% 怎么办？

1. top 找进程
2. pidstat 看线程

```bash
pidstat -t
```

3. 查看 recent deployment
4. 查看 logs
5. 判断：

  * dead loop
  * traffic spike
  * GC issue
  * DB latency


### 磁盘满了怎么办？

**排查**

```bash
df -h
du -sh /*
```

**常见原因**

* logs
* core dumps
* docker images
* temp files

```bash
lsof | grep deleted
```

查： 已删除但仍占空间文件。


### 如何用 Python 实现服务器健康检查？

```python
import psutil

cpu = psutil.cpu_percent()
mem = psutil.virtual_memory().percent

print(cpu, mem)
```

生产环境：

需要：

* timeout
* retries
* alerting


### Python 如何实现并发执行？

**IO-heavy**：

```python
ThreadPoolExecutor
```

```python
from concurrent.futures import ThreadPoolExecutor
```

### 如何提高 automation reliability？

**retry / timeout /rollback / idempotency / structured logging / monitoring /circuit breaker**


### 如何减少人为错误？

* automation
* standardization
* peer review
* CI/CD
* runbooks

**减少 manual operation**。

* **Reliability**: retry / timeout / rollback
* **Observability**:  metrics /  logs / tracing
* **Scalability**: concurrency / async processing
* **Fault Tolerance**: graceful degradation / failure isolation


## Part 3 Linux 系统与 Shell 脚本

### 1. 问题：如何在不重启进程的情况下，将正在运行的 Python 应用的调试日志级别从 INFO 改为 DEBUG？

**答案**  

- 使用 **信号机制**：在 Python 中注册信号处理器（如 `SIGUSR1`），收到信号后调用 `logging.setLevel(logging.DEBUG)`。  
- 通过 `/proc/<pid>/fd/` 写入控制命令（若应用实现控制管道）。  
- 使用 **gdb** attach 并调用 Python 内部函数（复杂场景）。  
- 或让应用监听 Unix socket，动态重载配置。


### 2. 问题：解释 Linux 中 load average 的含义，什么样的值算是异常？排查思路是什么？

**答案**  

- **<mark>Load average 是运行队列中（R 状态 + 不可中断睡眠 D 状态）的平均进程数</mark>**。  
- 单核 CPU 下，load > 1 表示过载；多核需除以核心数。  
- 异常排查：  
  1. `top` / `htop` 查看 CPU、内存、I/O wait。  
  2. 若 `wa` 高 → `iostat -x 1` 看磁盘利用率。  
  3. 若大量 D 状态进程 → `ps aux | awk '$8=="D" {print}'`，检查 NFS、磁盘故障。  
  4. 若 R 状态多但 CPU 不高 → 可能是锁竞争或线程调度问题。


### 3. 问题：写一个 Shell 脚本，监控某个日志文件在最近 1 分钟内出现 “ERROR” 的次数，如果超过 10 次就发送告警（用 echo 模拟）。

```bash
#!/bin/bash
LOG_FILE=/var/log/app.log
TIMESTAMP=$(date -d '1 minute ago' '+%Y-%m-%d %H:%M')
COUNT=$(awk -v ts="$TIMESTAMP" '$0 > ts && /ERROR/' "$LOG_FILE" | wc -l)
if [ "$COUNT" -gt 10 ]; then
    echo "ALERT: Errors in last minute: $COUNT"
fi
```
（更精确可用 `grep --after-context` 或 `tail -n 1000` + 时间戳比较）

### 4. 问题：你如何设计一个 Python 脚本，定期（如每分钟）检查 1000 台服务器的磁盘使用率，超过阈值时触发修复动作（如清理旧日志）？要求考虑高效、失败重试、并发。

**答案**  

- 使用 `concurrent.futures.ThreadPoolExecutor`（I/O 密集型）。  
- 每台任务包括 SSH（`paramiko` 或 `asyncssh`）或通过 API 调用。  
- 设计状态存储（Redis 或 SQLite），记录成功/失败/阈值触发。  
- 重试机制：指数退避，最多 3 次。  
- 清理动作可通过远程命令执行一个标准脚本（幂等）。  
- 增加超时控制（每个节点 10 秒）。  
- 示例伪代码：

```python
with ThreadPoolExecutor(max_workers=50) as executor:
    futures = {executor.submit(check_host, host): host for host in hosts}
    for future in as_completed(futures, timeout=300):
        try:
            result = future.result()
        except Exception as e:
            log_error(e)
```



### 5. 问题：Python 中 GIL 对系统工程师编写自动化工具的影响？如何绕过？

**答案**  

- GIL 限制同一时刻只有一个线程执行 Python 字节码，影响 CPU 密集型多线程。  
- 绕过方法：  
  - 使用多进程（`multiprocessing`）做并行计算。  
  - 对 I/O 密集型任务，线程仍有优势（GIL 在 I/O 时释放）。  
  - 关键代码用 C 扩展（如 NumPy）或 `asyncio`（单线程并发）。  
  - 使用 `subprocess` 调用外部工具（grep/awk），让外部多进程工作。

---

### 6. 问题：你在 Python 中如何优雅地处理大量配置文件的变更（如 500 个 Nginx 虚拟主机配置）并进行原子性部署？

**答案**  

- 生成配置到临时目录 `/tmp/nginx_conf_<timestamp>/`。  
- 用 `filecmp.dircmp` 对比新旧差异。  
- 调用 `nginx -t` 验证新配置。  
- 通过 `shutil.move` 或 `os.rename` 将目录原子性替换为 `/etc/nginx/sites-enabled`。  
- 重载 Nginx（`systemctl reload nginx`），若失败则 rollback。  
- 所有操作记录日志并支持一键回滚。


### 7. 问题：某服务突然 CPU 飙升到 100%，但流量并未增加。请给出从系统到应用层的排查流程。

**答案**  

1. `top` → 确认是 user 还是 system 高，定位进程 PID。  
2. `perf top -p <pid>` 或 `strace -p <pid>` 看系统调用。  
3. `py-spy`（Python）或 `pstack`（C++）查看堆栈热点。  
4. 检查是否进入死循环：  
   - 查看代码变更记录。  
   - 检查是否有正则表达式灾难性回溯。  
5. 检查资源泄漏（文件句柄、内存）。  
6. 使用 `flamegraph` 生成火焰图。  
7. 如果是 Python 应用，启用 `faulthandler` 或使用 `gdb` + `python-gdb.py`。

---

### 8. 问题：用户反映有时连接你的 TCP 服务会超时，但服务本身日志无报错。如何系统化排查？

**答案**  

- 客户端与服务端同时抓包 `tcpdump -w file.pcap`。  
- 分析 TCP 握手时间（SYN → SYN-ACK → ACK）。  
- 检查是否丢包：`netstat -s | grep retrans`。  
- 看服务端 backlog 是否溢出：`ss -lnt` 查看 Send-Q。  
- 检查 `sysctl` 参数：`net.ipv4.tcp_syn_retries`、`net.core.somaxconn`。  
- 若服务使用 systemd，看 `TimeoutStartSec`。  
- 检查防火墙或负载均衡器连接跟踪表满（`nf_conntrack`）。


### 9. 问题：设计一个系统，用于自动回收未使用的云磁盘（如 AWS EBS）。你需要考虑哪些关键点？

**答案**  

- 标记机制：磁盘最后一次挂载时间、挂载的主机是否还存在。  
- 排除标签（如 `"keep: true"`）。  
- 安全回收流程：  
  1. 快照备份（保留 N 天）。  
  2. 从主机卸载。  
  3. 等待 24 小时（冷却期）。  
  4. 删除磁盘并记录审计日志。  
- 通过 Python 脚本 + AWS SDK (boto3) 配合 Lambda + 事件桥接定期执行。  
- 防止误删：需要二次确认（如 Slack 通知 + approve 机制）。

---

### 10. 问题：你们团队管理 2000 台机器，如何保证 SSH 密钥和 sudo 权限策略的一致性和可审计性？

**答案**  

- 使用配置管理工具（Ansible/SaltStack）声明式定义用户、SSH key、sudoers 片段。  
- 所有密钥存储在 Vault（如 HashiCorp Vault）或 LDAP。  
- 集中式 SSH 认证：  
  - 短期证书签发型 SSH CA（`cert-authority`），无需下发密钥。  
- 审计：  
  - 所有 sudo 命令通过 `auditd` 或 `sudo_logs` 转发到 ELK。  
  - 定期扫描 `/home/*/.ssh/authorized_keys` 与 Git 中声明的主干比对。


### 11. 问题：你写了一个 Python 脚本，每天凌晨清理磁盘。但某次运行导致核心配置文件被误删，引发生产事故。如何避免将来发生？

**答案**  

- 强制实现 **dry-run 模式**，默认只输出即将删除的文件。  
- 在生产环境必须通过 `--apply` 参数或环境变量 `CONFIRM_DELETE=1` 才能执行。  
- 删除操作先移到 `/tmp/trash_<date>` 而非直接 `rm`。  
- 单元测试覆盖危险路径。  
- 脚本应校验路径白名单（如只允许 `/var/log/app/*.log`）。  
- 增加 peer review + 在 staging 环境运行 3 天。

### 12. 问题：如何向非技术同事解释 load average 和 CPU 利用率的不同？

**答案**  

- <mark>**CPU 利用率 = 厨师正在做菜的时间比例**。</mark>  
- <mark>**Load average = 排队等着做菜的顾客数 + 正在被服务的顾客数**</mark>。  
- 即使厨师一直忙（CPU 100%），但顾客越来越多（load 高），说明供不应求。  
- 如果厨师忙于洗菜（I/O wait），则利用率不高但 load 高。
