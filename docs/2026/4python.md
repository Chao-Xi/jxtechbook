

# 2026 Python for DevOps

## 0. Virtual Environment

### Create

```bash
python3 -m venv venv
```

### Activate — Linux/macOS

```bash
source venv/bin/activate
```

### Activate — Windows CMD

```cmd
venv\Scripts\activate
```

### Activate — Windows PowerShell

```powershell
.\venv\Scripts\Activate.ps1
```

### Deactivate

```bash
deactivate
```

### Interview Point

> Why use a virtual environment?

To isolate Python dependencies between projects and avoid version conflicts.

## 1. File Operations

### Read a file

```python
with open("file.txt", "r") as file:
    content = file.read()

print(content)
```

### Read line by line

```python
with open("file.txt", "r") as file:
    for line in file:
        print(line.strip())
```

**Interview tip:**

For large files, prefer iterating line-by-line rather than `read()` because it avoids loading the entire file into memory.


### Write a file

```python
with open("output.txt", "w") as file:
    file.write("Hello, DevOps!")
```

### Append

```python
with open("output.txt", "a") as file:
    file.write("\nNew line")
```

### Common modes

| Mode | Meaning           |
| ---- | ----------------- |
| `r`  | Read              |
| `w`  | Write / overwrite |
| `a`  | Append            |
| `r+` | Read + write      |
| `rb` | Read binary       |
| `wb` | Write binary      |


## 2. Environment Variables

Very important for DevOps.

### Read

```python
import os

db_user = os.getenv("DB_USER")
print(db_user)
```

Better:

```python
db_user = os.getenv("DB_USER", "default_user")
```

This provides a default value.

### Set

```python
import os

os.environ["APP_ENV"] = "production"
```

### DevOps Best Practice

Don't do:

```python
PASSWORD = "MyPassword123"
```

Instead:

```python
password = os.getenv("DB_PASSWORD")
```

Typical sources:

```text
Environment variables
Azure Key Vault
AWS Secrets Manager
HashiCorp Vault
Kubernetes Secrets
Jenkins Credentials
```


## 3. Subprocess Management

One of the **most important Python DevOps topics**.

### Execute command

```python
import subprocess

result = subprocess.run(
    ["ls", "-l"],
    capture_output=True,
    text=True
)

print(result.stdout)
```

### Check return code

```python
result = subprocess.run(
    ["ls", "-l"],
    capture_output=True,
    text=True
)

if result.returncode == 0:
    print("Command succeeded")
else:
    print("Command failed")
    print(result.stderr)
```

### Fail automatically

```python
subprocess.run(
    ["ls", "-l"],
    check=True
)
```

If the command fails, Python raises `subprocess.CalledProcessError`.

### Interview Question

**Q: Why use `subprocess.run()` instead of `os.system()`?**

Good answer:

> `subprocess.run()` provides better control over arguments, stdout, stderr, return codes, timeouts and error handling. It is generally preferred for automation scripts.


## 4. API Requests

Install:

```bash
pip install requests
```

### GET

```python
import requests

response = requests.get(
    "https://api.example.com/data",
    timeout=10
)

response.raise_for_status()

data = response.json()

print(data)
```

### Important interview point

Always consider:

```python
timeout=10
```

and:

```python
response.raise_for_status()
```

Otherwise your automation script can hang indefinitely or silently continue after an HTTP error.


### POST

```python
import requests

payload = {
    "name": "DevOps"
}

response = requests.post(
    "https://api.example.com/data",
    json=payload,
    timeout=10
)

response.raise_for_status()
```



## 5. JSON

### Read JSON

```python
import json

with open("data.json", "r") as file:
    data = json.load(file)

print(data)
```

### Write JSON

```python
import json

data = {
    "name": "DevOps",
    "type": "Workflow"
}

with open("output.json", "w") as file:
    json.dump(data, file, indent=4)
```

### JSON string ↔ Python object

```python
json.loads()
```

String → Python object.

```python
json.dumps()
```

Python object → String.

### Interview

```text
json.load()   → file → Python object
json.loads()  → string → Python object

json.dump()   → Python object → file
json.dumps()  → Python object → string
```


## 6. Logging

Use `logging` instead of `print()` in production automation.

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s - %(levelname)s - %(message)s"
)

logging.info("Deployment started")
logging.warning("Disk usage is high")
logging.error("Deployment failed")
```

Common levels:

```text
DEBUG
INFO
WARNING
ERROR
CRITICAL
```

### Interview Question

**Why logging instead of print?**

> Logging provides severity levels, timestamps, formatting and integration with centralized logging systems.

For example:

```text
Python
  ↓
stdout/stderr
  ↓
Docker / Kubernetes
  ↓
Fluent Bit
  ↓
Log Analytics / Elasticsearch / Loki
```



## 7. Database — SQLite

```python
import sqlite3

conn = sqlite3.connect("example.db")

cursor = conn.cursor()

cursor.execute("""
CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY,
    name TEXT
)
""")

conn.commit()
conn.close()
```

### Better pattern

```python
import sqlite3

with sqlite3.connect("example.db") as conn:
    conn.execute("""
        CREATE TABLE IF NOT EXISTS users (
            id INTEGER PRIMARY KEY,
            name TEXT
        )
    """)
```

The context manager handles commit/rollback behavior more safely.


## 8. SSH Automation — Paramiko

Install:

```bash
pip install paramiko
```

Basic example:

```python
import paramiko

ssh = paramiko.SSHClient()

ssh.set_missing_host_key_policy(
    paramiko.AutoAddPolicy()
)

ssh.connect(
    hostname="server.example.com",
    username="user",
    password="password"
)

stdin, stdout, stderr = ssh.exec_command("uptime")

print(stdout.read().decode())
print(stderr.read().decode())

ssh.close()
```

### Better security

Don't hardcode passwords.

Use:

```python
import os

password = os.getenv("SSH_PASSWORD")
```

Or preferably use SSH keys.

### Interview Question

**Paramiko vs Ansible?**

| Paramiko                   | Ansible                               |
| -------------------------- | ------------------------------------- |
| Python SSH library         | Configuration management tool         |
| Programmatic control       | Declarative automation                |
| Need to write logic        | Built-in modules                      |
| Good for custom automation | Good for infrastructure configuration |


## 9. Error Handling

Basic:

```python
try:
    result = risky_operation()

except Exception as e:
    print(f"Error: {e}")
```

Better:

```python
try:
    result = risky_operation()

except ValueError as e:
    print(f"Invalid value: {e}")

except ConnectionError as e:
    print(f"Connection failed: {e}")

finally:
    print("Cleanup")
```

### DevOps principle

Avoid:

```python
except:
    pass
```

because it hides failures.

## 10. Docker SDK

Install:

```bash
pip install docker
```

## List containers

```python
import docker

client = docker.from_env()

containers = client.containers.list()

for container in containers:
    print(container.name)
```

### Create container

```python
import docker

client = docker.from_env()

container = client.containers.run(
    "ubuntu",
    "echo Hello World",
    detach=True
)

print(container.logs().decode())
```

#### Interview Question

**How does Docker SDK connect to Docker?**

```python
docker.from_env()
```

It uses the Docker environment/configuration, typically communicating with the Docker daemon through its socket.

## 11. YAML

Install:

```bash
pip install pyyaml
```

### Read YAML

```python
import yaml

with open("config.yaml", "r") as file:
    config = yaml.safe_load(file)

print(config)
```

### Write YAML

```python
import yaml

data = {
    "name": "DevOps",
    "version": "1.0"
}

with open("output.yaml", "w") as file:
    yaml.safe_dump(data, file)
```

#### Important

Prefer:

```python
yaml.safe_load()
```

over:

```python
yaml.load()
```

for untrusted YAML.

## 12. Command-Line Arguments

Use `argparse`.

```python
import argparse

parser = argparse.ArgumentParser(
    description="DevOps automation tool"
)

parser.add_argument(
    "--environment",
    required=True
)

parser.add_argument(
    "--version",
    required=True
)

args = parser.parse_args()

print(args.environment)
print(args.version)
```

Run:

```bash
python deploy.py --environment production --version 1.2.0
```

#### Interview advantage

This is much better than hardcoding:

```python
environment = "production"
```

because the same script can be used by:

```text
Developer
Jenkins
GitHub Actions
GitLab CI
Azure DevOps
Cron
```


## 13. System Resource Monitoring

Install:

```bash
pip install psutil
```

```python
import psutil

cpu = psutil.cpu_percent(interval=1)
memory = psutil.virtual_memory().percent
disk = psutil.disk_usage("/").percent

print(f"CPU: {cpu}%")
print(f"Memory: {memory}%")
print(f"Disk: {disk}%")
```

Very useful for:

```text
Health checks
Monitoring
Auto-remediation
Capacity management
Troubleshooting
```


## 14. Flask Health Check

Your original example needs indentation correction.

```python
from flask import Flask, jsonify

app = Flask(__name__)


@app.route("/health", methods=["GET"])
def health_check():
    return jsonify({
        "status": "healthy"
    })


if __name__ == "__main__":
    app.run(
        host="0.0.0.0",
        port=5000
    )
```

Test:

```bash
curl http://localhost:5000/health
```

Response:

```json
{
    "status": "healthy"
}
```

#### DevOps use cases

Flask can provide:

```text
/health
/ready
/metrics
/version
```

In Kubernetes:

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 5000

readinessProbe:
  httpGet:
    path: /health
    port: 5000
```


## 15. Scheduling Tasks

Install:

```bash
pip install schedule
```

```python
import schedule
import time


def job():
    print("Running scheduled job...")


schedule.every(1).minutes.do(job)

while True:
    schedule.run_pending()
    time.sleep(1)
```

#### Interview point

For production Linux scheduling, you will often use:

```text
cron
systemd timer
Kubernetes CronJob
Jenkins
Airflow
Cloud scheduler
```

rather than keeping a Python process running forever.

## 16. Git Automation

Install:

```bash
pip install GitPython
```

Example:

```python
import git

repo = git.Repo("/path/to/repo")

repo.git.add("file.txt")

repo.index.commit(
    "Add file.txt"
)
```

Push:

```python
origin = repo.remote(name="origin")
origin.push()
```

#### Interview point

Python can automate Git operations, but CI/CD systems commonly use:

```bash
git
```

directly in pipeline steps.


## 17. Email Notifications

Basic SMTP:

```python
import os
import smtplib

from email.mime.text import MIMEText


msg = MIMEText(
    "Deployment completed successfully."
)

msg["Subject"] = "Deployment Notification"
msg["From"] = os.getenv("EMAIL_FROM")
msg["To"] = os.getenv("EMAIL_TO")


with smtplib.SMTP("smtp.example.com", 587) as server:
    server.starttls()

    server.login(
        os.getenv("SMTP_USER"),
        os.getenv("SMTP_PASSWORD")
    )

    server.send_message(msg)
```

#### Interview security point

Never:

```python
server.login(
    "admin",
    "Password123"
)
```

Use:

```text
Environment variables
Secret Manager
Key Vault
Jenkins Credentials
Kubernetes Secrets
```


## 18. Virtual Environment Automation

Your original example has an important problem.

This:

```python
os.system("source myenv/bin/activate")
```

does **not** activate the environment for the parent Python process.

Instead, create the environment:

```python
import subprocess

subprocess.run(
    ["python3", "-m", "venv", "myenv"],
    check=True
)
```

Then run commands using the environment's Python directly:

```python
subprocess.run(
    ["myenv/bin/python", "script.py"],
    check=True
)
```

This is much more reliable for automation.

## 19. Jenkins Automation Through REST API

Install:

```bash
pip install requests
```

Basic example:

```python
import requests

url = "https://jenkins.example.com/job/my-job/build"

response = requests.post(
    url,
    auth=("username", "api-token"),
    timeout=10
)

response.raise_for_status()

print(response.status_code)
```

#### Important Jenkins concept

Use an **API token**, not the user's actual password.

Typical architecture:

```text
Python Script
      |
      | REST API
      ↓
   Jenkins
      |
      ↓
Pipeline
      |
      ↓
Build → Test → Scan → Deploy
```

---

## 20. DevOps Python Automation — Must-Know Libraries

For your interview, I would prioritize these:

| Library      | Purpose             | Priority |
| ------------ | ------------------- | -------: |
| `os`         | Environment/files   |    ⭐⭐⭐⭐⭐ |
| `subprocess` | Execute commands    |    ⭐⭐⭐⭐⭐ |
| `requests`   | REST APIs           |    ⭐⭐⭐⭐⭐ |
| `json`       | JSON processing     |    ⭐⭐⭐⭐⭐ |
| `logging`    | Application logging |    ⭐⭐⭐⭐⭐ |
| `argparse`   | CLI tools           |    ⭐⭐⭐⭐⭐ |
| `pathlib`    | File management     |    ⭐⭐⭐⭐⭐ |
| `yaml`       | YAML/K8s config     |     ⭐⭐⭐⭐ |
| `psutil`     | System monitoring   |     ⭐⭐⭐⭐ |
| `paramiko`   | SSH                 |     ⭐⭐⭐⭐ |
| `docker`     | Docker API          |      ⭐⭐⭐ |
| `GitPython`  | Git automation      |      ⭐⭐⭐ |
| `sqlite3`    | Database            |       ⭐⭐ |
| `flask`      | REST/health API     |      ⭐⭐⭐ |
| `schedule`   | Scheduling          |       ⭐⭐ |


## 21. You Should Add `pathlib`

For a modern Python DevOps interview, I strongly recommend adding this.

Instead of:

```python
import os

files = os.listdir("/tmp")
```

Use:

```python
from pathlib import Path

path = Path("/tmp")

for file in path.iterdir():
    print(file)
```

Check existence:

```python
if Path("/tmp/config.yaml").exists():
    print("File exists")
```

Create directory:

```python
Path("/tmp/myapp").mkdir(
    parents=True,
    exist_ok=True
)
```

Find logs:

```python
for file in Path("/var/log").glob("*.log"):
    print(file)
```

This is excellent for DevOps automation.


## 22. Python DevOps Interview — Core Automation Pattern

This is one pattern I recommend you memorize:

```python
import os
import subprocess
import logging


logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s %(levelname)s %(message)s"
)


def run_command(command):
    logging.info("Running: %s", command)

    result = subprocess.run(
        command,
        capture_output=True,
        text=True
    )

    if result.returncode != 0:
        logging.error(result.stderr)
        raise RuntimeError(
            f"Command failed: {command}"
        )

    return result.stdout


def main():
    environment = os.getenv(
        "APP_ENV",
        "development"
    )

    logging.info(
        "Environment: %s",
        environment
    )

    output = run_command(
        ["docker", "ps"]
    )

    print(output)


if __name__ == "__main__":
    main()
```

This combines:

```text
Environment variables
        +
Logging
        +
Subprocess
        +
Error handling
        +
Functions
        +
if __name__ == "__main__"
```

That is much closer to what you may actually be asked to write during a **DevOps / Platform Operations interview**.

## 21. Database Migration — Alembic

### 核心概念

Alembic 是 Python 生态中常用于 **SQLAlchemy 数据库 schema migration** 的工具。

典型流程：

```bash
alembic revision --autogenerate -m "add users table"
alembic upgrade head
```

回滚：

```bash
alembic downgrade -1
```

查看 migration：

```bash
alembic history
alembic current
```

### 面试重点

**Q: Why do we need database migration?**

> Database migration allows us to version-control database schema changes and apply them consistently across development, testing, and production environments.

### DevOps 关注点

不要简单地在 CI/CD 中：

```bash
alembic upgrade head
```

然后就结束。

Production 中还要考虑：

* backup
* backward compatibility
* rollback
* database locking
* migration ordering
* zero-downtime deployment

例如：

```text
Deploy v1
   ↓
DB migration
   ↓
Deploy v2
```

更安全的方式通常是：

```text
Add new column
      ↓
Deploy application that supports old + new schema
      ↓
Migrate data
      ↓
Remove old column later
```

这叫 **expand-and-contract migration**。

### 面试问题

> How would you handle database schema changes without downtime?

关键词：

**backward compatible + expand/contract + rolling deployment**


## 22. Unit Testing — unittest

你的代码：

```python
import unittest

def add(a, b):
    return a + b


class TestMathFunctions(unittest.TestCase):

    def test_add(self):
        self.assertEqual(add(2, 3), 5)


if __name__ == '__main__':
    unittest.main()
```

运行：

```bash
python -m unittest
```

或者：

```bash
python -m unittest discover
```

### DevOps 为什么需要测试？

CI/CD pipeline 中：

```text
Git Push
   ↓
Lint
   ↓
Unit Test
   ↓
Build
   ↓
Security Scan
   ↓
Deploy
```

测试是 deployment gate。

#### 常见面试问题

**Q: What is unit testing?**

> Unit testing tests a small, isolated piece of code independently from external dependencies.

例如：

```text
Function
   ↓
Input
   ↓
Expected Output
```

#### unittest vs pytest

面试可以这样回答：

| unittest        | pytest                 |
| --------------- | ---------------------- |
| Python built-in | Third-party            |
| class-based 常见  | function-based 更常见     |
| verbose         | 简洁                     |
| unittest.mock   | pytest fixtures        |
| 标准库             | DevOps/现代 Python 项目很常见 |


## 23. Data Transformation — Pandas

```python
import pandas as pd

df = pd.read_csv("data.csv")

df["new_column"] = df["existing_column"] * 2

df.to_csv("output.csv", index=False)
```

#### DevOps 使用场景

Pandas 不只是 Data Science。

可以用于：

* log analysis
* CSV processing
* report generation
* deployment metrics
* cost analysis
* inventory processing
* test result analysis

例如：

```text
AWS billing CSV
       ↓
Pandas
       ↓
Group by account
       ↓
Calculate cost
       ↓
CSV/Excel report
```

#### 面试重点

如果数据非常大：

> Would you use pandas for a 100 GB CSV?

不一定。

因为 Pandas 通常需要把大量数据放入 memory。

可以考虑：

* streaming
* chunksize
* Spark
* database processing
* DuckDB

例如：

```python
for chunk in pd.read_csv("large.csv", chunksize=10000):
    process(chunk)
```


## 24. Python for Infrastructure as Code — boto3

```python
import boto3

ec2 = boto3.resource("ec2")

instances = ec2.instances.filter(
    Filters=[
        {
            "Name": "instance-state-name",
            "Values": ["running"]
        }
    ]
)

for instance in instances:
    print(instance.id, instance.state)
```

### 重点

boto3 是 AWS SDK。

DevOps 可以用它：

```text
Python
  ↓
boto3
  ↓
AWS API
  ↓
EC2 / S3 / IAM / CloudWatch / Lambda
```

### 但是一个非常重要的面试点

**boto3 ≠ Infrastructure as Code。**

如果面试官问：

> Would you use boto3 or Terraform to provision infrastructure?

通常：

### Terraform

用于：

```text
Desired State
     ↓
Terraform
     ↓
Infrastructure
```

强调：

* declarative
* state
* plan
* drift detection
* repeatability

#### boto3

更适合：

* automation
* operational scripts
* AWS API interaction
* dynamic workflows

例如：

```text
Terraform
   ↓
Create EC2 infrastructure

Python/boto3
   ↓
Query EC2
   ↓
Check status
   ↓
Perform operational action
```


## 25. Web Scraping — BeautifulSoup

```python
import requests
from bs4 import BeautifulSoup

response = requests.get(
    "https://example.com",
    timeout=10
)

response.raise_for_status()

soup = BeautifulSoup(
    response.text,
    "html.parser"
)

print(soup.title.string)
```

#### 原始代码的问题

不要在生产代码中：

```python
requests.get(url)
```

没有 timeout。

应该：

```python
requests.get(url, timeout=10)
```

并处理：

```python
response.raise_for_status()
```

#### 面试问题

> What problems do you need to consider when building a web scraper?

回答：

* timeout
* retry
* rate limiting
* robots.txt
* authentication
* pagination
* HTML changes
* malformed data
* proxy
* logging
* error handling


## 26. Remote Execution — Fabric

```python
from fabric import Connection

conn = Connection(
    "user@hostname"
)

result = conn.run("uname -s")

print(result.stdout)
```

Fabric 可以用于：

```text
Python
   ↓
SSH
   ↓
Remote Server
   ↓
Execute command
```

#### 生产环境注意

不建议：

```python
password="your_password"
```

更推荐：

```text
SSH key
   ↓
SSH agent
   ↓
Fabric
```

或者使用：

* Vault
* AWS Secrets Manager
* Azure Key Vault
* CI/CD secret store

面试问题

> How would you securely execute commands on 100 servers?

不要回答：

> Write a Python loop and SSH to every server.

更好的答案：

```text
Configuration Management
        ↓
Ansible
        ↓
Parallel execution
        ↓
Idempotent configuration
```

Fabric 更适合 Python-based remote automation。


## 27. Automating AWS S3 Operations

```python
import boto3

s3 = boto3.client("s3")

s3.upload_file(
    "local_file.txt",
    "bucket-name",
    "s3_file.txt"
)

s3.download_file(
    "bucket-name",
    "s3_file.txt",
    "local_file.txt"
)
```

#### 面试重点

S3 常见 DevOps 自动化：

```text
Build artifact
      ↓
S3
      ↓
Backup
      ↓
Deployment
```

或者：

```text
Application
    ↓
S3
    ↓
Log / Backup / Artifact
```

#### 安全问题

不要：

```python
boto3.client(
    "s3",
    aws_access_key_id="xxx",
    aws_secret_access_key="xxx"
)
```

生产环境优先使用：

```text
IAM Role
   ↓
Temporary credentials
   ↓
boto3
```

例如 EC2：

```text
EC2
 ↓
IAM Instance Role
 ↓
STS temporary credentials
 ↓
boto3
 ↓
S3
```

这是非常常见的面试题。


## 28. Monitoring Application Logs

你的代码是 Python 实现：

```bash
tail -f app.log
```

核心：

```python
import time


def tail_f(file):
    file.seek(0, 2)

    while True:
        line = file.readline()

        if not line:
            time.sleep(0.1)
            continue

        print(line, end="")


with open("app.log", "r") as log_file:
    tail_f(log_file)
```

#### 面试重点

如果问：

> How do you monitor logs in production?

不要回答：

> Python tail -f.

应该回答：

```text
Application
    ↓
Structured logs
    ↓
Log collector
    ↓
Centralized logging
    ↓
Search / Alert / Dashboard
```

例如：

* ELK
* Loki
* Azure Monitor
* CloudWatch
* Splunk

#### Kubernetes

更不要在 container 内自己 tail log。

通常：

```text
stdout/stderr
    ↓
Container runtime
    ↓
Log collector
    ↓
Centralized logging
```


## 29. Docker Container Health Check

```python
import docker

client = docker.from_env()

container = client.containers.get("container_id")

print(
    container.attrs["State"]["Health"]["Status"]
)
```

#### 可能的问题

如果 Docker container 没有配置 healthcheck：

```python
container.attrs["State"]["Health"]
```

可能不存在。

所以生产代码需要防御：

```python
health = container.attrs["State"].get("Health")

if health:
    print(health["Status"])
else:
    print("No health check configured")
```

#### Docker

Dockerfile：

```dockerfile
HEALTHCHECK CMD curl --fail http://localhost:8080/health || exit 1
```

#### Kubernetes

Kubernetes 更重要：

```text
livenessProbe
readinessProbe
startupProbe
```

面试经常问：

#### Liveness

> Is the application alive?

失败：

```text
restart container
```

#### Readiness

> Can the application receive traffic?

失败：

```text
remove from Service endpoints
```

#### Startup

> Has the application finished starting?

适合：

```text
slow-starting applications
```


## 30. Rate-Limited APIs

原始代码：

```python
while True:
    response = requests.get(url)

    if response.status_code == 200:
        print(response.json())
        break

    elif response.status_code == 429:
        time.sleep(60)

    else:
        print("Error:", response.status_code)
        break
```

可以改得更 production-ready。

#### Exponential Backoff

```python
import time
import requests

url = "https://api.example.com/data"

max_retries = 5

for attempt in range(max_retries):
    response = requests.get(url, timeout=10)

    if response.status_code == 200:
        print(response.json())
        break

    if response.status_code == 429:
        delay = 2 ** attempt
        time.sleep(delay)
        continue

    response.raise_for_status()
```

例如：

```text
Attempt 1 → 1 sec
Attempt 2 → 2 sec
Attempt 3 → 4 sec
Attempt 4 → 8 sec
Attempt 5 → 16 sec
```

#### 更好的答案

生产环境还可以考虑：

```text
Exponential Backoff
+
Jitter
+
Retry Limit
+
Retryable Status Codes
+
Timeout
```

#### 面试问题

> Why should we use exponential backoff instead of retrying immediately?

因为立即 retry 可能：

```text
Server overloaded
      ↓
Client retries
      ↓
More load
      ↓
Server gets worse
      ↓
More retries
```

形成 **retry storm**。


## 31. Docker Compose Integration

```python
import subprocess

subprocess.run(
    ["docker", "compose", "up", "-d"],
    check=True
)

subprocess.run(
    ["docker", "compose", "down"],
    check=True
)
```

#### 注意

现代 Docker 推荐：

```bash
docker compose
```

而不是老版本：

```bash
docker-compose
```

#### 面试重点

Docker Compose 适合：

```text
Local development
Integration testing
Small environments
POC
```

例如：

```text
Application
   +
PostgreSQL
   +
Redis
   +
RabbitMQ
```

一起启动。

Production Kubernetes 环境通常会考虑：

```text
Kubernetes
Helm
Argo CD
Terraform
```

##  32. Terraform Execution

你的代码：

```python
import subprocess

subprocess.run(
    ["terraform", "init"],
    check=True
)

subprocess.run(
    ["terraform", "apply", "-auto-approve"],
    check=True
)
```

### 非常重要的面试问题

> Would you run terraform apply -auto-approve directly from Python?

Production 中通常不建议这么简单地做。

更合理：

```text
Git
 ↓
Terraform Plan
 ↓
Review / Policy Check
 ↓
Approval
 ↓
Terraform Apply
```

如果要求 fully automated：

```text
Git
 ↓
CI Pipeline
 ↓
terraform fmt
 ↓
terraform validate
 ↓
terraform plan
 ↓
Policy/Security scan
 ↓
terraform apply
```

#### Terraform 常见命令

```bash
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
terraform destroy
terraform state list
```



## 33. Prometheus Metrics

原始代码：

```python
import requests

response = requests.get(
    "http://localhost:9090/metrics"
)

metrics = response.text.splitlines()

for metric in metrics:
    print(metric)
```

#### 一个概念需要纠正

Prometheus 的标准架构通常是：

```text
Application
    ↓
/metrics
    ↑
Prometheus scrape
```

不是：

```text
Python
 ↓
Prometheus /metrics
```

应用暴露：

```text
http://application:8080/metrics
```

Prometheus：

```text
Prometheus
    ↓
scrape
    ↓
Application /metrics
```

### Python 应用

可以使用：

```python
from prometheus_client import Counter
```

例如：

```python
from prometheus_client import Counter

requests_total = Counter(
    "http_requests_total",
    "Total HTTP requests"
)

requests_total.inc()
```

### 面试关键词

Prometheus：

* metrics
* scraping
* labels
* PromQL
* alerting
* Grafana


## 34. pytest

```python
def add(a, b):
    return a + b


def test_add():
    assert add(2, 3) == 5
```

运行：

```bash
pytest
```

更常见：

```bash
pytest -v
```

覆盖率：

```bash
pytest --cov
```

#### DevOps Pipeline

```yaml
- name: Install dependencies
  run: pip install -r requirements.txt

- name: Run tests
  run: pytest -v

- name: Coverage
  run: pytest --cov
```

#### 面试问题

> What is a fixture in pytest?

Fixture 用于准备测试环境/共享测试资源。

例如：

```python
import pytest


@pytest.fixture
def user():
    return {
        "name": "Jacob"
    }


def test_user(user):
    assert user["name"] == "Jacob"
```


## 35. Creating Webhooks — Flask

```python
from flask import Flask, request

app = Flask(__name__)


@app.route("/webhook", methods=["POST"])
def webhook():
    data = request.json

    print("Received data:", data)

    return "OK", 200


if __name__ == "__main__":
    app.run(port=5000)
```

#### DevOps 场景

非常典型：

```text
GitHub
   ↓
Webhook
   ↓
Flask
   ↓
Process event
   ↓
CI/CD
```

例如 GitHub：

```text
push
pull_request
release
```

#### Production 必须考虑

这个代码直接暴露出去是不安全的。

需要：

* authentication
* webhook signature verification
* HTTPS
* rate limiting
* replay protection
* input validation
* logging

#### 一个非常好的面试答案

> I would not trust the webhook payload directly. I would validate the request signature, authenticate the sender, validate the payload schema, and make the handler idempotent.


## 36. Jinja2 Configuration Templates

```python
from jinja2 import Template

template = Template(
    "Hello, {{ name }}!"
)

rendered = template.render(
    name="DevOps"
)

print(rendered)
```

DevOps 中更常见：

```text
template
    ↓
variables
    ↓
configuration
```

例如：

```jinja2
server:
  host: {{ host }}
  port: {{ port }}
```

然后：

```python
template.render(
    host="10.0.0.10",
    port=8080
)
```

#### 应用场景

* Ansible
* configuration generation
* Kubernetes YAML
* Nginx configuration
* application config

#### 面试问题

> Why use templates instead of hardcoding configuration?

因为：

```text
Same template
     +
Dev variables
     ↓
Dev config

Same template
     +
Prod variables
     ↓
Prod config
```

减少 duplication。


## 37. Encryption / Decryption

```python
from cryptography.fernet import Fernet

key = Fernet.generate_key()

cipher_suite = Fernet(key)

encrypted_text = cipher_suite.encrypt(
    b"Secret Data"
)

decrypted_text = cipher_suite.decrypt(
    encrypted_text
)

print(decrypted_text.decode())
```

#### 面试中的陷阱

**Encryption key 不能和 encrypted data 一起硬编码。**

不要：

```python
key = Fernet.generate_key()
```

然后每次 application restart 都生成新 key。

否则之前的数据无法解密。

Production：

```text
Application
    ↓
Key Vault / Secrets Manager
    ↓
Encryption key
```

例如：

```text
Azure Key Vault
AWS Secrets Manager
HashiCorp Vault
```

#### Encryption vs Hashing

这是非常高频的问题。

| Encryption | Hashing               |
| ---------- | --------------------- |
| 可逆         | 不可逆                   |
| 需要 key     | 通常无 key               |
| 加密数据       | 验证数据                  |
| AES/Fernet | SHA-256/bcrypt/Argon2 |

例如：

```text
Password
   ↓
Hash
   ↓
Database
```

而：

```text
Secret data
   ↓
Encryption
   ↓
Database
```


## 38. Error Monitoring — Sentry

```python
import sentry_sdk

sentry_sdk.init(
    dsn="your_sentry_dsn"
)
```

然后：

```python
try:
    divide(1, 0)
except ZeroDivisionError as e:
    sentry_sdk.capture_exception(e)
```

#### DevOps 关注点

Sentry 是 application error monitoring。

可以收集：

```text
Exception
Stack trace
Request
Environment
Release
User context
```

#### Observability 三大支柱

面试非常重要：

```text
Observability
   ├── Logs
   ├── Metrics
   └── Traces
```

例如：

```text
Logs       → What happened?
Metrics    → How much/how often?
Traces     → Where did the request go?
```


## 39. CI — GitHub Actions

你的 YAML 需要注意缩进。

推荐：

```yaml
name: CI

on:
  push:
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Run tests
        run: |
          pytest -v
```

#### DevOps 面试一定要掌握 Pipeline Architecture

例如：

```text
Developer
    ↓
Git Push
    ↓
CI
 ┌───────────────┐
 │ Lint          │
 │ Unit Test     │
 │ SAST          │
 │ Dependency    │
 │ Scan          │
 └───────────────┘
    ↓
Build
    ↓
Container Image
    ↓
Image Scan
    ↓
Registry
    ↓
CD
    ↓
Dev
    ↓
Test
    ↓
Production
```

#### 常见面试问题

**Q: How do you prevent a bad build from reaching production?**

回答：

> I would implement quality gates in CI/CD, including unit tests, code quality checks, security scanning, image scanning, and deployment validation before promoting artifacts to production.

## 40. FastAPI

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {
        "item_id": item_id
    }
```

运行：

```bash
uvicorn main:app --host 0.0.0.0 --port 8000
```

#### 为什么 DevOps 面试会问 FastAPI？

因为它涉及：

```text
API
 ↓
Container
 ↓
Kubernetes
 ↓
Ingress
 ↓
Load Balancer
 ↓
Monitoring
```

非常符合 Platform / DevOps 场景。

### FastAPI 自带

Swagger：

```text
/docs
```

OpenAPI：

```text
/openapi.json
```

### Docker

例如：

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

然后：

```bash
docker build -t my-api .
docker run -p 8000:8000 my-api
```



更好的方式是把它们串起来：

```text
                Developer
                    │
                    ▼
                 GitHub
                    │
                    ▼
             GitHub Actions
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
       pytest              Security Scan
          │                   │
          └─────────┬─────────┘
                    ▼
              Docker Build
                    │
                    ▼
              Image Registry
                    │
                    ▼
                Terraform
                    │
                    ▼
              Cloud Infrastructure
                    │
                    ▼
                Kubernetes
                    │
             ┌──────┴──────┐
             ▼             ▼
          FastAPI        Database
             │
             ▼
        Prometheus
             │
             ▼
          Grafana

Python Automation
 ├── boto3
 ├── S3
 ├── API calls
 ├── Retry
 ├── Log processing
 └── Remote execution
```

这才是 **DevOps Engineer 的完整故事**。


## 🔥 面试官最可能追问的 10 个问题

你尤其应该把下面 10 个问题练到可以直接回答：

### 1.

**Why would you use Terraform instead of boto3 to provision infrastructure?**

关键词：

> Declarative / State / Idempotency / Plan / Drift detection.

---

### 2.

**How do you securely access AWS from Python?**

不要说：

> Access Key + Secret Key in source code.

应该：

> IAM Role / Workload Identity / Temporary credentials.

---

### 3.

**How do you implement retry logic for an API?**

关键词：

> Timeout + exponential backoff + jitter + retry limit + retryable errors.

---

### 4.

**How would you monitor a Python application in production?**

回答：

```text
Logs
Metrics
Traces
Alerts
Dashboards
```

---

### 5.

**What is the difference between liveness and readiness probes?**

这是 Kubernetes 高频题。

---

### 6.

**How do you manage secrets in CI/CD?**

回答：

```text
Secret Manager / Key Vault / Vault
        ↓
Pipeline
        ↓
Temporary credential
```

不要：

```text
password in YAML
```

---

### 7.

**How would you deploy a database schema change without downtime?**

关键词：

> Backward compatibility + expand/contract migration + rolling deployment.

---

### 8.

**How do you design a reliable webhook?**

关键词：

> Authentication + signature verification + validation + idempotency + retry + HTTPS.

---

### 9.

**What happens when a Kubernetes pod becomes unhealthy?**

需要区分：

```text
liveness
readiness
startup
```

---

### 10.

**How would you design a production CI/CD pipeline?**

可以直接背这个框架：

```text
Git
 ↓
Lint
 ↓
Unit Test
 ↓
SAST
 ↓
Dependency Scan
 ↓
Build
 ↓
Container Scan
 ↓
Push Image
 ↓
Deploy Dev
 ↓
Integration Test
 ↓
Deploy Test
 ↓
Validation
 ↓
Production
 ↓
Monitoring
```



## 41. Log Aggregation — ELK Stack

你的例子：

```python
from elasticsearch import Elasticsearch

es = Elasticsearch(["http://localhost:9200"])

log = {
    "level": "info",
    "message": "This is a log message"
}

es.index(index="logs", document=log)
```

> 新版 Elasticsearch Python client 更推荐 `document=`，而不是老版本常见的 `body=`。


### ELK 是什么？

```text
Application
     ↓
    Logs
     ↓
 Log Collector
     ↓
 Logstash / Beats / Fluent Bit
     ↓
Elasticsearch
     ↓
   Kibana
```

其中：

| Component     | Purpose                     |
| ------------- | --------------------------- |
| Elasticsearch | Store/search logs           |
| Logstash      | Collect/transform logs      |
| Kibana        | Visualization               |
| Beats         | Lightweight data collectors |

现代 Kubernetes 环境也经常使用：

```text
Application
    ↓
stdout
    ↓
Fluent Bit
    ↓
Elasticsearch
    ↓
Kibana
```

### 面试重点

**Q: Why don't you just store logs in local files?**

因为：

* container 会被删除
* 多台服务器日志分散
* 搜索困难
* 无法集中分析
* 无法统一 alert

Production：

```text
100 Containers
      ↓
Centralized Logging
      ↓
One Search Platform
```

### 高级问题

> How would you design log aggregation for Kubernetes?

可以回答：

```text
Pod stdout/stderr
       ↓
Fluent Bit / Fluentd
       ↓
Kafka / Elasticsearch
       ↓
Kibana
```

如果规模非常大，可以加入 Kafka：

```text
Pods
 ↓
Fluent Bit
 ↓
Kafka
 ↓
Logstash
 ↓
Elasticsearch
 ↓
Kibana
```


## 42. Pandas ETL

你的代码：

```python
import pandas as pd

# Extract
data = pd.read_csv("source.csv")

# Transform
data["new_column"] = (
    data["existing_column"] * 2
)

# Load
data.to_csv(
    "destination.csv",
    index=False
)
```

这就是：

```text
ETL

Extract
   ↓
Transform
   ↓
Load
```

### DevOps 中可以用在哪里？

例如：

```text
Cloud billing
     ↓
CSV
     ↓
Python/Pandas
     ↓
Transform
     ↓
Cost report
```

或者：

```text
Application logs
     ↓
Pandas
     ↓
Analyze errors
     ↓
Generate report
```

### 面试问题

> What if the input file is 100 GB?

不要直接：

```python
pd.read_csv("100GB.csv")
```

因为可能导致内存不足。

使用：

```python
for chunk in pd.read_csv(
    "large.csv",
    chunksize=10000
):
    process(chunk)
```

或者考虑：

* Spark
* database
* DuckDB
* streaming
* distributed processing


## 43. AWS Lambda

代码：

```python
import json


def lambda_handler(event, context):
    return {
        "statusCode": 200,
        "body": json.dumps(
            "Hello from Lambda!"
        )
    }
```

### Lambda 核心概念

Lambda 是：

> Serverless, event-driven compute.

你不需要管理：

```text
OS
VM
Server
Patching
```

AWS 管理 infrastructure。


### Lambda execution model

```text
Event
 ↓
Lambda
 ↓
Handler
 ↓
Response
```

Event 可以来自：

```text
API Gateway
S3
EventBridge
SQS
SNS
CloudWatch
DynamoDB Streams
```

### DevOps 面试问题

**Q: What are the advantages of Lambda?**

回答：

* serverless
* automatic scaling
* pay-per-use
* event-driven
* no server management

### Q: What are the limitations?

重点：

* execution timeout
* memory limit
* cold start
* ephemeral filesystem
* stateless design
* concurrency limits


#### Lambda 最重要的 DevOps 概念：Idempotency

例如：

```text
S3 Event
   ↓
Lambda
   ↓
Process file
```

如果 event 被重复发送：

```text
Event
 ↓
Lambda
 ↓
Lambda again
```

你的 Lambda 不应该造成重复数据。

所以：

> Lambda functions should ideally be idempotent.

## 44. Redis

代码：

```python
import redis

r = redis.Redis(
    host="localhost",
    port=6379,
    db=0
)

r.set("foo", "bar")

print(r.get("foo"))
```

输出通常：

```text
b'bar'
```

如果想得到 string：

```python
r = redis.Redis(
    host="localhost",
    port=6379,
    decode_responses=True
)

print(r.get("foo"))
```

得到：

```text
bar
```

## Redis 在 DevOps 中为什么重要？

最常见：

### 1. Cache

```text
Application
    ↓
Redis
    ↓
Cache hit
```

减少：

```text
Database load
```
### 2. Session

```text
User
 ↓
Application
 ↓
Redis
 ↓
Session
```

### 3. Distributed Lock

例如：

```text
Server A ─┐
Server B ─┼──> Redis Lock
Server C ─┘
```

避免多个 server 同时执行同一个 job。



### 4. Queue / Pub/Sub

Redis 也可以用于：

```text
Pub/Sub
Streams
Queues
```

### 面试问题

> What happens if Redis goes down?

不要只回答：

> Application fails.

需要考虑：

```text
Cache
 ↓
Redis unavailable
 ↓
Fallback to database
```

否则可能产生：

### Cache Stampede

```text
Redis down
   ↓
Thousands requests
   ↓
All hit DB
   ↓
Database overloaded
```

生产环境需要：

* Redis HA
* retry
* timeout
* circuit breaker
* cache fallback
* TTL


## 45. pyngrok

代码：

```python
from pyngrok import ngrok

public_url = ngrok.connect(5000)

print("Public URL:", public_url)

input("Press Enter to exit...")
```

用途：

```text
Local machine
localhost:5000
      ↓
    ngrok
      ↓
Public HTTPS URL
      ↓
Internet
```

#### DevOps 使用场景

非常适合：

* webhook testing
* local API testing
* demo
* POC

例如：

```text
GitHub
   ↓
https://xxxxx.ngrok.app
   ↓
localhost:5000
```

### 面试重点

> Would you use ngrok in production?

通常：

**No.**

它更适合：

```text
Development
Testing
POC
```

Production 应该使用：

```text
Load Balancer
Ingress
API Gateway
Reverse Proxy
```


## 46. Flask-RESTful

代码可以写成：

```python
from flask import Flask
from flask_restful import Resource, Api

app = Flask(__name__)
api = Api(app)


class HelloWorld(Resource):

    def get(self):
        return {
            "hello": "world"
        }


api.add_resource(
    HelloWorld,
    "/"
)


if __name__ == "__main__":
    app.run()
```


#### REST API 面试必须知道 HTTP methods

```text
GET
POST
PUT
PATCH
DELETE
```

例如：

```text
GET    /users
GET    /users/123

POST   /users

PUT    /users/123

PATCH  /users/123

DELETE /users/123
```


#### HTTP status codes

至少掌握：

```text
200 OK
201 Created
204 No Content

400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
429 Too Many Requests

500 Internal Server Error
502 Bad Gateway
503 Service Unavailable
504 Gateway Timeout
```

#### 非常重要

```text
401
```

通常表示：

> Authentication required / invalid authentication.

而：

```text
403
```

通常表示：

> Authenticated but not authorized.

这个经常被问。


## 47. asyncio

代码：

```python
import asyncio


async def main():
    print("Hello")

    await asyncio.sleep(1)

    print("World")


asyncio.run(main())
```

#### 核心概念

`asyncio` 是 Python 的 asynchronous I/O framework。

特别适合：

```text
I/O-bound workloads
```

例如：

* HTTP requests
* database calls
* network operations
* sockets


#### 同步 vs 异步

同步：

```text
Request A
   ↓
wait
   ↓
response
   ↓
Request B
```

异步：

```text
Request A ──────┐
                │
Request B ──────┤
                │
Request C ──────┘
       ↓
responses
```

#### 重要面试问题

> Does asyncio make CPU-intensive code faster?

**通常不是。**

`asyncio` 主要解决：

> I/O-bound concurrency

如果是 CPU-bound：

```text
Image processing
Machine learning
Large computation
```

考虑：

* multiprocessing
* process pool
* distributed computing


## 48. Network Monitoring — Scapy

代码：

```python
from scapy.all import sniff


def packet_callback(packet):
    print(packet.summary())


sniff(
    prn=packet_callback,
    count=10
)
```

Scapy 可以：

* sniff packets
* inspect packets
* construct packets
* analyze network traffic

### DevOps 场景

例如：

```text
Application unavailable
        ↓
Check application
        ↓
Check service
        ↓
Check DNS
        ↓
Check TCP
        ↓
Check network packets
```

Scapy 可以帮助 network troubleshooting。

-
### 但是面试要注意

Scapy packet sniffing 通常需要：

```text
root / elevated privileges
```

而且生产环境不一定直接使用 Scapy。

更常见的 troubleshooting：

```bash
tcpdump
ss
netstat
ping
curl
dig
nslookup
traceroute
```

所以如果面试官问：

> How do you troubleshoot a network issue on Linux?

一个很好的回答是：

```text
DNS
 ↓
ping
 ↓
curl
 ↓
ss
 ↓
tcpdump
 ↓
Application logs
```


## 49. Configuration — configparser

代码：

```python
import configparser

config = configparser.ConfigParser()

config.read("config.ini")

print(
    config["DEFAULT"]["SomeSetting"]
)

config["DEFAULT"]["NewSetting"] = "Value"

with open("config.ini", "w") as configfile:
    config.write(configfile)
```

例如：

```ini
[DEFAULT]
host=localhost
port=8080
environment=dev
```

读取：

```python
config["DEFAULT"]["host"]
```

## DevOps 中的配置管理

需要区分：

#### Configuration

例如：

```text
HOST
PORT
ENVIRONMENT
LOG_LEVEL
```

#### Secret

例如：

```text
PASSWORD
API_KEY
PRIVATE_KEY
TOKEN
```

**不要把 secret 放进 config.ini。**

例如不要：

```ini
[DEFAULT]
database_password=SuperSecret123
```

应该：

```text
Application
     ↓
Secret Manager
     ↓
Password
```

例如：

```text
AWS Secrets Manager
Azure Key Vault
HashiCorp Vault
Kubernetes Secret
```


## 50. WebSocket Client

你的代码：

```python
import websocket


def on_message(ws, message):
    print("Received message:", message)


ws = websocket.WebSocketApp(
    "ws://echo.websocket.org",
    on_message=on_message
)

ws.run_forever()
```

注意：

你的原始代码最后：

```python
ws.run_forever
```

少了 `()`。

应该：

```python
ws.run_forever()
```


## WebSocket vs REST

这是面试非常值得准备的题目。

### REST

```text
Client
  ↓ request
Server
  ↓ response
Client
```

通常：

```text
HTTP
```


#### WebSocket

```text
Client
   ↕
Persistent Connection
   ↕
Server
```

双方都可以主动发送消息。

适合：

* chat
* real-time notification
* stock price
* live dashboard
* multiplayer games
* monitoring

## WebSocket 面试问题

> Why would you use WebSocket instead of REST?

如果需要：

> real-time bidirectional communication

使用 WebSocket。

例如：

```text
REST:

Client → Server
       ← Response

WebSocket:

Client ←→ Server
Client ←→ Server
Client ←→ Server
```



### 1. How would you design centralized logging for Kubernetes?

```text
Pod
 ↓
stdout/stderr
 ↓
Fluent Bit
 ↓
Elasticsearch
 ↓
Kibana
```


### 2. What is the difference between logs, metrics and traces?

```text
Logs    → What happened?
Metrics → How much/how often?
Traces  → Where did the request go?
```

### 3. What are the advantages and limitations of AWS Lambda?

关键词：

```text
Serverless
Event-driven
Auto scaling
Pay-per-use

Cold start
Timeout
Concurrency
Stateless
```

### 4. What would you use Redis for?

```text
Cache
Session
Distributed lock
Queue
Pub/Sub
```

### 5. What happens when Redis becomes unavailable?

回答：

> It depends on whether Redis is a cache or a system of record. For a cache, the application should ideally degrade gracefully and fall back to the database while protecting the database from a cache stampede.

这个回答就明显比：

> Redis down, application down.

专业很多。

### 6. What is the difference between asyncio and multiprocessing?

```text
asyncio
   ↓
I/O-bound
   ↓
Concurrency

multiprocessing
   ↓
CPU-bound
   ↓
Parallelism
```


### 7. What is the difference between REST and WebSocket?

```text
REST
Request/Response

WebSocket
Persistent
Bidirectional
Real-time
```


### 8. How do you manage application configuration?

回答：

```text
Configuration
    ↓
Environment variables / ConfigMap

Secrets
    ↓
Secret Manager / Vault / Key Vault
```

不要把 secret：

```text
Git
Dockerfile
source code
config.ini
```


