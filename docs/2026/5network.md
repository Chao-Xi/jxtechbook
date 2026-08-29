# 9. Networking, Ports & Load Balancing — DevOps Interview Notes

## 9.1 Networking Basics

### 1. IP Address

#### Private IP

RFC1918 私有地址：

| Range                         | CIDR             |
| ----------------------------- | ---------------- |
| 10.0.0.0 – 10.255.255.255     | `10.0.0.0/8`     |
| 172.16.0.0 – 172.31.255.255   | `172.16.0.0/12`  |
| 192.168.0.0 – 192.168.255.255 | `192.168.0.0/16` |

Example:

```text
10.0.1.10
192.168.1.100
172.16.10.20
```

### 2. CIDR

```text
192.168.1.0/24
```

代表：

```text
Network: 192.168.1.0

Subnet mask: 255.255.255.0

Addresses: 256

Usable IPv4 addresses: normally 254
```

面试常问：

> What does `/24` mean?

回答：

> `/24` means the first 24 bits are the network portion, leaving 8 bits for hosts.

## 9.2 Common Ports 🔥


| Service |  Port | Protocol | Purpose                   |
| ------- | ----: | -------- | ------------------------- |
| SSH     |    22 | TCP      | Remote Linux access       |
| FTP     |    21 | TCP      | File transfer             |
| SFTP    |    22 | TCP      | Secure file transfer      |
| Telnet  |    23 | TCP      | Unencrypted remote access |
| SMTP    |    25 | TCP      | Email                     |
| DNS     |    53 | TCP/UDP  | Name resolution           |
| DHCP    | 67/68 | UDP      | IP assignment             |
| HTTP    |    80 | TCP      | Web                       |
| HTTPS   |   443 | TCP      | Secure web                |
| LDAP    |   389 | TCP/UDP  | Directory service         |
| LDAPS   |   636 | TCP      | Secure LDAP               |
| SMB     |   445 | TCP      | Windows file sharing      |
| RDP     |  3389 | TCP      | Windows remote desktop    |

#### DevOps Tools

| Tool                  | Common Port |
| --------------------- | ----------: |
| Jenkins               |        8080 |
| SonarQube             |        9000 |
| Nexus                 |        8081 |
| Prometheus            |        9090 |
| Grafana               |        3000 |
| Kibana                |        5601 |
| Loki                  |        3100 |
| Jaeger                |       16686 |
| Docker Registry       |        5000 |
| Kubernetes API Server |        6443 |
| Kubelet               |       10250 |
| etcd                  |   2379–2380 |

#### Databases

| Database        |  Port |
| --------------- | ----: |
| MySQL / MariaDB |  3306 |
| PostgreSQL      |  5432 |
| MongoDB         | 27017 |
| Redis           |  6379 |
| Cassandra       |  9042 |
| Kafka           |  9092 |
| RabbitMQ        |  5672 |



## 9.3 TCP vs UDP vs ICMP

#### TCP

**Connection-oriented and reliable.**

```text
Client
  |
  | TCP connection
  v
Server
```

Features:

* Connection-oriented
* Reliable delivery
* Ordering
* Retransmission
* Flow control

Typical applications：

```text
HTTP
HTTPS
SSH
FTP
SMTP
Database connections
```

#### UDP

Connectionless.

特点：

* Faster
* No connection establishment
* No guaranteed delivery
* No guaranteed ordering

Typical examples:

```text
DNS
DHCP
VoIP
Streaming
```

### ICMP

Used for network diagnostic/control messages.

Example:

```bash
ping google.com
```



## 9.4 Linux Network Troubleshooting 🔥🔥


### Check IP

```bash
ip a
```

或者：

```bash
ip addr
```

旧命令：

```bash
ifconfig
```

> `ip` is preferred on modern Linux systems.


### Check routing

```bash
ip route
```

Example:

```text
default via 192.168.1.1 dev eth0
```

面试问题：

> The server has an IP address but cannot access the Internet. What do you check?

推荐回答：

```text
1. ip a
2. ip route
3. ping gateway
4. ping 8.8.8.8
5. DNS test
6. firewall/security group
```


## 9.5 Connectivity Troubleshooting

### Ping

```bash
ping google.com
```

测试：

```text
ICMP connectivity
```

注意：

> Ping failure does NOT always mean the service is down.

因为 ICMP 可能被 firewall 禁止。


### DNS

```bash
nslookup google.com
```

或者：

```bash
dig google.com
```

更详细：

```bash
dig google.com
```

检查 DNS resolution：

```text
hostname
   |
   v
DNS
   |
   v
IP address
```


### 9.6 Test a TCP Port

#### telnet

```bash
telnet example.com 443
```

但是现在更推荐：

```bash
nc -zv example.com 443
```

或者：

```bash
nc -zv 10.0.1.10 8080
```

Example：

```text
Connection to 10.0.1.10 8080 port [tcp/*] succeeded!
```

这说明：

> TCP connection to the port succeeded.

**非常重要：**

```bash
nc -zv server 8080
```

**只能证明 TCP port reachable。**

不能证明：

```text
HTTP application is healthy
```

HTTP 应用应该进一步：

```bash
curl http://server:8080/health
```


### 9.7 curl 🔥

测试 HTTP：

```bash
curl http://example.com
```

查看 HTTP headers：

```bash
curl -I https://example.com
```

Verbose：

```bash
curl -v https://example.com
```

测试 API：

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"name":"test"}' \
  https://example.com/api
```

面试非常实用：

```bash
curl -v
```

可以帮助分析：

```text
DNS
TCP connection
TLS
HTTP request
HTTP response
```

### 9.8 traceroute

```bash
traceroute google.com
```

用于查看：

```text
Client
  |
  v
Router 1
  |
  v
Router 2
  |
  v
Router 3
  |
  v
Server
```

Linux 有时需要：

```bash
traceroute -T -p 443 example.com
```

用于 TCP traceroute。


### 9.9 ss vs netstat

现代 Linux 推荐：

```bash
ss -tuln
```

解释：

```text
-t  TCP
-u  UDP
-l  listening
-n  numeric
```

Example：

```bash
ss -tuln
```

查某个端口：

```bash
ss -tulnp | grep 8080
```

也可以：

```bash
lsof -i :8080
```


### 9.10 Firewall

#### iptables

查看：

```bash
sudo iptables -L -v -n
```

允许 SSH：

```bash
sudo iptables -A INPUT \
  -p tcp \
  --dport 22 \
  -j ACCEPT
```

禁止某个 IP：

```bash
sudo iptables -A INPUT \
  -s 192.168.1.100 \
  -j DROP
```

### 面试注意

不要简单说：

> iptables is the firewall.

更准确：

> iptables is a user-space interface for configuring Linux Netfilter packet filtering rules.

### 9.11 Netcat

#### Server

```bash
nc -lvp 8080
```

#### Client

```bash
echo "Hello" | nc 192.168.1.100 8080
```

用途：

* Test connectivity
* Test ports
* Simple TCP/UDP testing
* Troubleshooting


### 9.12 Kubernetes Networking 🔥🔥🔥

#### 查看 Services

```bash
kubectl get svc
```

更详细：

```bash
kubectl get svc -o wide
```

查看 endpoints：

```bash
kubectl get endpoints
```

现代 Kubernetes 推荐：

```bash
kubectl get endpointslices
```

#### 查看 Pod IP

```bash
kubectl get pods -o wide
```

Example：

```text
NAME       READY   STATUS    IP
web-pod    1/1     Running   10.244.1.10
```

### 9.13 Kubernetes Service

核心关系：

```text
Internet
   |
   v
Ingress
   |
   v
Service
   |
   +--------+
   |        |
   v        v
 Pod      Pod
```

Service 提供：

* Stable virtual IP
* Service discovery
* Load balancing
* Pod abstraction

### 9.14 Kubernetes Port Forward

```bash
kubectl port-forward svc/my-service 8080:80
```

访问：

```bash
curl http://localhost:8080
```

注意：

> `port-forward` is primarily a debugging/development mechanism, not a production exposure mechanism.



### 9.15 Docker Networking

List networks：

```bash
docker network ls
```

Inspect：

```bash
docker network inspect bridge
```

Create:

```bash
docker network create mynetwork
```

Run container：

```bash
docker run -d \
  --network=mynetwork \
  nginx
```


## Cloud Networking

### AWS VPC

核心组件：

```text
VPC
 |
 +-- Subnet
 |
 +-- Route Table
 |
 +-- Internet Gateway
 |
 +-- NAT Gateway
 |
 +-- Security Group
 |
 +-- Network ACL
 |
 +-- VPC Endpoint
```

VPC

```bash
aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16
```

Subnet

```bash
aws ec2 create-subnet \
  --vpc-id <vpc-id> \
  --cidr-block 10.0.1.0/24
```

### 9.17 AWS Security Group

Security Group 是：

> Stateful virtual firewall associated with AWS resources such as EC2 instances.

Example：

```text
Internet
   |
   | TCP 443
   v
Security Group
   |
   v
EC2
```

**Stateful**

如果 inbound：

```text
Client ---> Server
```

允许 TCP 443，

response traffic：

```text
Server ---> Client
```

会自动允许返回。

这是非常常见的面试题。

### 9.18 Security Group vs NACL

|                | Security Group          | Network ACL            |
| -------------- | ----------------------- | ---------------------- |
| Level          | Instance/ENI            | Subnet                 |
| Stateful       | Yes                     | No                     |
| Rules          | Allow                   | Allow/Deny             |
| Return traffic | Automatic               | Must explicitly allow  |
| Typical use    | Resource-level security | Subnet-level filtering |

面试回答：

> Security Groups are stateful and operate at the resource/ENI level, while Network ACLs are stateless and operate at the subnet level.


###  9.19 Public vs Private Subnet

**Public Subnet**

通常：

```text
Subnet
 |
Route Table
 |
Internet Gateway
 |
Internet
```

**Private Subnet**

通常：

```text
Private Subnet
      |
      v
NAT Gateway
      |
      v
Internet Gateway
      |
      v
Internet
```

关键点：

> **Private subnet resources can initiate outbound Internet traffic through NAT Gateway, but they are not directly reachable from the Internet through the NAT Gateway.**

###  9.20 Reverse Proxy 🔥🔥🔥

#### What is a Reverse Proxy?

Client doesn't directly communicate with backend.

```text
Client
   |
   v
Reverse Proxy
   |
   +--------+
   |        |
   v        v
Backend1 Backend2
```

Examples：

```text
Nginx
HAProxy
Apache
Envoy
Traefik
```

用途：

* Load balancing
* TLS termination
* Authentication
* Routing
* Caching
* Compression
* Security
* Hide backend infrastructure


### 9.21 Forward Proxy vs Reverse Proxy

这是很好的面试题。

#### Forward Proxy

```text
Client
  |
  v
Proxy
  |
  v
Internet
```

Proxy represents the **client**.

Example：

```text
Corporate proxy
```

#### Reverse Proxy

```text
Client
  |
  v
Reverse Proxy
  |
  v
Backend
```

Proxy represents the **server**.

Example：

```text
Nginx
HAProxy
Ingress Controller
```

一句话记忆：

> Forward proxy hides clients; reverse proxy hides servers.

### 9.22 Nginx Reverse Proxy

修正你原来的 typo：

```nginx
proxy_pass
```

不是：

```nginx
roxy_pass
```

Example：

```nginx
upstream backend_servers {
    server server1.example.com:8080;
    server server2.example.com:8080;
}

server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://backend_servers;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### 9.23 Nginx Load Balancing

```nginx
upstream backend_servers {
    server server1.example.com:8080;
    server server2.example.com:8080;
}
```

Default：

```text
Round Robin
```

其他常见方式：

```text
round robin
least_conn
ip_hash
```

例如：

```nginx
upstream backend_servers {
    least_conn;

    server server1.example.com:8080;
    server server2.example.com:8080;
}
```

###  9.24 HAProxy 🔥🔥

HAProxy 非常值得 DevOps 面试准备。

基本结构：

```text
Client
   |
   v
Frontend
   |
   v
Backend
   |
   +-------+
   |       |
   v       v
Server1 Server2
```

Example：

```haproxy
frontend http_front
    bind *:80
    default_backend backend_servers

backend backend_servers
    balance roundrobin

    server server1 server1.example.com:80 check
    server server2 server2.example.com:80 check
```

#### `check`

```text
server server1 server1.example.com:80 check
```

意味着 HAProxy 会进行 health checking。

如果：

```text
Server1 = unhealthy
```

HAProxy 会停止把正常流量发送给 Server1。

---

# 9.25 Load Balancing Algorithms

面试建议至少知道：

### Round Robin

```text
Request 1 -> Server1
Request 2 -> Server2
Request 3 -> Server1
Request 4 -> Server2
```

### Least Connections

发送给当前连接数最少的 server。

```text
Server1: 100 connections
Server2: 20 connections

New request -> Server2
```

### IP Hash

根据 client IP 选择 backend。

用途：

> Session persistence / sticky sessions.


### 9.26 Apache Load Balancing

Apache 使用：

```text
mod_proxy
mod_proxy_http
mod_proxy_balancer
```

Example：

```apache
<Proxy "balancer://mycluster">
    BalancerMember "http://server1.example.com"
    BalancerMember "http://server2.example.com"
</Proxy>

<VirtualHost *:80>
    ServerName example.com

    ProxyPass "/" "balancer://mycluster/"
    ProxyPassReverse "/" "balancer://mycluster/"
</VirtualHost>
```

### 9.27 Kubernetes Ingress 🔥🔥🔥

Ingress 是：

> An API object that defines HTTP/HTTPS routing rules to Services.

架构：

```text
Internet
    |
    v
Load Balancer
    |
    v
Ingress Controller
    |
    +----------------+
    |                |
    v                v
Service A         Service B
    |                |
    v                v
Pods              Pods
```

### 9.28 Ingress vs Ingress Controller

这是非常容易问的。

#### Ingress

定义：

```text
WHAT should happen?
```

例如：

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
```

定义：

```text
example.com/api -> api-service
example.com/web -> web-service
```

#### Ingress Controller

负责：

```text
HOW to implement it?
```

例如：

```text
NGINX Ingress Controller
HAProxy Ingress
Traefik
Istio Gateway
```

一句话：

> Ingress is the routing configuration; the Ingress Controller implements that configuration.

### 9.29 Correct Kubernetes Ingress YAML

你原来的 YAML indentation 需要修正。

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-service
                port:
                  number: 80
```

检查：

```bash
kubectl get ingress
```

详细：

```bash
kubectl describe ingress my-ingress
```


### 9.30 Nginx vs HAProxy vs Kubernetes Ingress

| Technology         | Main Purpose                    | Typical Use                    |
| ------------------ | ------------------------------- | ------------------------------ |
| Nginx              | Reverse proxy + Web server + LB | Web/API traffic                |
| HAProxy            | High-performance LB             | TCP/HTTP load balancing        |
| Apache             | Web server + Reverse proxy      | Legacy/enterprise environments |
| Ingress            | Kubernetes routing API          | Define K8s HTTP routing        |
| Ingress Controller | Implements Ingress              | Nginx/HAProxy/Traefik          |
| Service            | Kubernetes service discovery/LB | Pod access                     |

---

### 9.31 Very Important Interview Concept: L4 vs L7

#### Layer 4 Load Balancing

Based on：

```text
IP
Port
TCP
UDP
```

Example：

```text
Client
  |
TCP 443
  |
  v
L4 Load Balancer
  |
  +----> Server1
  |
  +----> Server2
```

It doesn't necessarily understand HTTP.


#### Layer 7 Load Balancing

Understands：

```text
HTTP
HTTPS
Host
Path
Headers
Cookies
```

Example：

```text
example.com/api
       |
       v
Load Balancer
       |
       +---- /api ---> API server
       |
       +---- /web ---> Web server
```

Nginx / HAProxy / Ingress can perform L7 HTTP routing.


### 9.32 TLS Termination 🔥

Common architecture：

```text
Client
  |
 HTTPS
  |
  v
Load Balancer
  |
 HTTP
  |
  v
Backend
```

The Load Balancer terminates TLS.

Benefits：

* Centralized certificate management
* Reduce TLS overhead on backend
* Easier certificate rotation

Example：

```text
Client
   |
 HTTPS :443
   |
   v
Nginx / HAProxy
   |
 HTTP :8080
   |
   v
Application
```

### 9.33 Health Check

Load balancer doesn't simply send traffic blindly.

Example：

```text
LB
 |
 +---- Server1 ✓
 |
 +---- Server2 ✓
 |
 +---- Server3 ✗
```

Server3 is removed from the traffic pool.

Application health endpoint：

```text
GET /health
```

或者 Kubernetes：

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
```

Very important distinction

**Liveness**

> Is the application alive?

**Readiness**

> Is the application ready to receive traffic?


###  9.34 DevOps Networking Troubleshooting Flow 🔥🔥🔥

面试可以直接用这个思路。

假设：

> User cannot access `https://example.com`.

我会按照以下层次排查：

```text
1. DNS
       |
       v
2. Network connectivity
       |
       v
3. TCP port
       |
       v
4. TLS
       |
       v
5. HTTP
       |
       v
6. Load Balancer
       |
       v
7. Kubernetes Ingress
       |
       v
8. Service
       |
       v
9. Pod
       |
       v
10. Application
```

对应命令：

Step 1 — DNS

```bash
dig example.com
```

Step 2 — Connectivity

```bash
ping example.com
```

Step 3 — TCP

```bash
nc -zv example.com 443
```

Step 4 — TLS/HTTP

```bash
curl -v https://example.com
```

Step 5 — Kubernetes

```bash
kubectl get ingress
kubectl describe ingress my-ingress
kubectl get svc
kubectl get endpoints
kubectl get pods -o wide
```

Step 6 — Pod

```bash
kubectl logs <pod>
```

Step 7 — Test Service directly

```bash
kubectl port-forward svc/my-service 8080:80
```

然后：

```bash
curl http://localhost:8080
```


### 9.35 ⭐ 面试最重要的 15 个问题

建议你把下面 15 个问题练到可以直接回答。

#### Q1. What is a reverse proxy?

> A reverse proxy sits in front of backend servers and forwards client requests to those servers. It can provide load balancing, TLS termination, routing, caching and security.

#### Q2. Reverse proxy vs forward proxy?

> A forward proxy represents clients, while a reverse proxy represents servers.

#### Q3. What is load balancing?

> Load balancing distributes incoming traffic across multiple backend servers to improve availability, scalability and performance.

#### Q4. What is round-robin?

> Requests are distributed sequentially across backend servers.

#### Q5. What is health checking?

> The load balancer periodically checks backend health and removes unhealthy servers from the traffic pool.

#### Q6. L4 vs L7?

> L4 operates based on IP and TCP/UDP connections, while L7 understands application protocols such as HTTP and can route based on host, path, headers or cookies.

#### Q7. What is Kubernetes Ingress?

> Ingress is a Kubernetes API resource that defines HTTP/HTTPS routing rules to Services.

#### Q8. What is an Ingress Controller?

> It is the component that implements the Ingress rules and actually handles incoming traffic.

#### Q9. Service vs Ingress?

> A Service provides stable connectivity and service discovery to Pods, while Ingress provides external HTTP/HTTPS routing to Services.

#### Q10. What is a Security Group?

> A Security Group is a stateful virtual firewall controlling inbound and outbound traffic for AWS resources.

#### Q11. Security Group vs NACL?

> Security Groups are stateful and resource-level; NACLs are stateless and subnet-level.

#### Q12. How do you test whether port 8080 is reachable?

```bash
nc -zv server 8080
```

#### Q13. How do you test an HTTP endpoint?

```bash
curl -v http://server:8080/health
```

#### Q14. `ping` works but application doesn't. What do you check?

```text
1. TCP port
2. Firewall
3. Security Group
4. Application listening port
5. Service
6. Load balancer
7. Application logs
```

#### Q15. Kubernetes application is running but cannot be accessed. What do you check?

```text
Ingress
   ↓
Service
   ↓
Endpoints / EndpointSlices
   ↓
Pod
   ↓
Container port
   ↓
Application
```

### 9.36 最后形成一张 Architecture Cheat Sheet

你面试时脑子里最好形成下面这个模型：

```text
                         Internet
                            |
                            | HTTPS :443
                            v
                    +----------------+
                    | Load Balancer  |
                    | L4 / L7        |
                    +----------------+
                            |
                            v
                    +----------------+
                    | Reverse Proxy  |
                    | Nginx/HAProxy  |
                    +----------------+
                            |
                            v
                  +--------------------+
                  | Kubernetes Ingress |
                  +--------------------+
                     |              |
                     v              v
                Service A        Service B
                     |              |
                 +---+---+      +---+---+
                 |       |      |       |
                Pod     Pod    Pod     Pod
                 |       |      |       |
                 +-------+------+-------+
                         |
                    Application
```

而故障排查思路就是：

```text
DNS
 ↓
IP
 ↓
Route
 ↓
Firewall / SG / NACL
 ↓
TCP Port
 ↓
TLS
 ↓
HTTP
 ↓
Load Balancer
 ↓
Ingress
 ↓
Service
 ↓
Endpoint
 ↓
Pod
 ↓
Application
```
