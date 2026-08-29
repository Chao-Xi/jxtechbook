
# 2026 DevOps Interview – IaC & Configuration Management

## 0. First: IaC 和 Configuration Management 有什么区别？

### Infrastructure as Code — IaC

主要解决：

> **Infrastructure 应该是什么样？**

例如：

```text
VPC
Subnet
VM
Load Balancer
Database
Kubernetes Cluster
```

典型工具：

```text
Terraform
Bicep
CloudFormation
Pulumi
```


### Configuration Management

主要解决：

> **机器/系统里面应该配置成什么样？**

例如：

```text
Install Nginx
Create user
Configure SSH
Modify config file
Start service
Deploy application
```

典型：

```text
Ansible
Chef
Puppet
```

## 1. Terraform

Terraform 是典型的 **declarative Infrastructure as Code** 工具。

核心思想：

```text
Terraform Configuration
        |
        v
Desired Infrastructure
        |
        v
Terraform
        |
        v
Cloud Provider
```

例如：

```hcl
resource "aws_instance" "app_server" {
  ami           = "ami-08d70e59c07c61a3a"
  instance_type = "t2.micro"

  tags = {
    Name = var.instance_name
  }
}
```

Terraform 关注的是：

```text
What should exist?
```

而不是：

```text
How do I manually create it?
```

### 2. Terraform Standard File Structure

面试建议掌握：

```text
terraform/
├── main.tf
├── variables.tf
├── outputs.tf
├── providers.tf
├── versions.tf
├── terraform.tfvars
└── modules/
```

`main.tf`

资源：

```hcl
resource "aws_instance" "app_server" {
  ami           = "ami-xxxx"
  instance_type = "t2.micro"

  tags = {
    Name = var.instance_name
  }
}
```

`variables.tf`

输入参数：

```hcl
variable "instance_name" {
  description = "EC2 instance name"
  type        = string
  default     = "ExampleAppServerInstance"
}
```


**`outputs.tf`**

输出：

```hcl
output "instance_id" {
  value = aws_instance.app_server.id
}

output "instance_public_ip" {
  value = aws_instance.app_server.public_ip
}
```

### 3. Terraform Provider

Provider 告诉 Terraform：

> How to communicate with the infrastructure platform.

例如 AWS：

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.16"
    }
  }

  required_version = ">= 1.2.0"
}

provider "aws" {
  region = "us-west-2"
}
```

Azure 则通常是：

```hcl
provider "azurerm" {
  features {}
}
```

**4. Terraform Workflow**

这个一定要背：

```text
terraform init
       ↓
terraform fmt
       ↓
terraform validate
       ↓
terraform plan
       ↓
terraform apply
```

`terraform init`

初始化：

```bash
terraform init
```

主要做：

```text
Download providers
Initialize backend
Initialize modules
```

**`terraform fmt`**

```bash
terraform fmt
```

格式化 Terraform configuration。

**`terraform validate`**

```bash
terraform validate
```

验证 configuration 是否有效。

注意：

> `validate` 不代表你的 infrastructure 一定能成功创建。

**`terraform plan`**

```bash
terraform plan
```

查看 Terraform 准备做什么。

例如：

```text
+ create
~ update
- destroy
```

**`terraform apply`**

```bash
terraform apply
```

真正执行变化。

生产环境常见：

```bash
terraform plan -out=tfplan
terraform apply tfplan
```

这个比直接：

```bash
terraform apply
```

更适合 CI/CD。

### 5. Terraform State

这是 Terraform 面试**最重要的主题之一**。

Terraform 需要知道：

```text
What resources does Terraform manage?
```

所以需要：

```text
terraform.tfstate
```

关系：

```text
Terraform Code
      |
      v
Desired State

terraform.tfstate
      |
      v
Terraform's knowledge of managed resources

Cloud
      |
      v
Actual Infrastructure
```

Terraform 根据这些信息计算：

```text
Plan
```

### 6. Remote State

生产环境不要把：

```text
terraform.tfstate
```

简单地放在个人 laptop。

应该使用：

```text
Remote Backend
```

例如：

```text
Terraform Cloud
Azure Storage
AWS S3
Google Cloud Storage
```

### AWS Example

你原来的：

```hcl
backend "s3" {
  bucket         = "my-tfstate-bucket"
  key            = "terraform/state"
  region         = "us-west-2"
  encrypt        = true
  dynamodb_table = "terraform-locks"
}
```

**注意**

这里有一个**重要的现代化修正**：

传统上 DynamoDB 被用于 Terraform state locking，但当前 Terraform / S3 backend 的 locking 能力已经发生变化，因此面试时不要死背：

```text
S3 + DynamoDB = mandatory
```

更好的回答：

> Use a remote backend with state locking appropriate to the backend and Terraform version.

如果面试官问 Azure：

```text
Azure Storage Account
        |
        v
Terraform State
```

是非常典型的企业方案。

### 7. Why Terraform State Locking?

假设：

```text
Developer A
    |
    v
terraform apply

Developer B
    |
    v
terraform apply
```

同时修改：

```text
terraform.tfstate
```

可能导致：

```text
State corruption
Race condition
Conflicting changes
```

所以生产环境需要：

```text
Remote State
+
State Locking
```

### 8. Terraform Modules

Modules 用于：

```text
Reuse
Standardization
Maintainability
```

例如：

```text
modules/
├── vpc/
├── aks/
├── vm/
└── database/
```

使用：

```hcl
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"

  name = "my-vpc"
  cidr = "10.0.0.0/16"

  azs = [
    "us-west-2a",
    "us-west-2b"
  ]

  public_subnets = [
    "10.0.1.0/24",
    "10.0.2.0/24"
  ]

  private_subnets = [
    "10.0.3.0/24",
    "10.0.4.0/24"
  ]
}
```

面试：

> Why use modules?

回答：

```text
Reusability
Consistency
Standardization
Reduce duplication
Easier maintenance
```

### 9. Terraform Variables

推荐：

```text
variables.tf
```

定义：

```hcl
variable "environment" {
  type    = string
  default = "dev"
}
```

使用：

```hcl
tags = {
  Environment = var.environment
}
```

生产环境：

```text
dev.tfvars
test.tfvars
prod.tfvars
```

例如：

```bash
terraform apply -var-file=prod.tfvars
```

### 10. Terraform Secrets

不要：

```hcl
password = "MyPassword123"
```

更好的方式：

```text
Azure Key Vault
AWS Secrets Manager
HashiCorp Vault
Environment variables
CI/CD secret store
```

尤其注意：

> Terraform state itself may contain sensitive values.

所以：

```text
不要只保护 .tf 文件
```

还要保护：

```text
terraform.tfstate
```

### 11. Important Terraform Commands

| Command                | Purpose                  |
| ---------------------- | ------------------------ |
| `terraform init`       | Initialize               |
| `terraform fmt`        | Format                   |
| `terraform validate`   | Validate                 |
| `terraform plan`       | Preview                  |
| `terraform apply`      | Apply                    |
| `terraform destroy`    | Destroy                  |
| `terraform show`       | Show state/plan          |
| `terraform output`     | Show outputs             |
| `terraform state list` | List state resources     |
| `terraform state show` | Show one resource        |
| `terraform import`     | Import existing resource |
| `terraform providers`  | Show providers           |

### 12. `terraform import`

面试经常问：

> What if the infrastructure already exists but was not created by Terraform?

例如：

```text
Existing Azure VM
```

可以：

```bash
terraform import azurerm_linux_virtual_machine.example <resource-id>
```

核心概念：

```text
Existing Resource
       ↓
terraform import
       ↓
Terraform State
```

但是：

> `terraform import` does not automatically generate the complete Terraform configuration for the resource.

这是一个很容易被坑的点。



### 13. Terraform Taint —— 注意！

你原资料：

```bash
terraform taint <resource>
```

这是旧知识。

现代 Terraform 更推荐：

```bash
terraform apply -replace="aws_instance.app_server"
```

例如：

```bash
terraform apply \
  -replace="aws_instance.app_server"
```

面试可以说：

> `terraform taint` is a legacy approach; modern Terraform uses `-replace`.

### 14. Terraform CI/CD

一个非常标准的 pipeline：

```text
Git
 |
 v
Terraform fmt
 |
 v
Terraform validate
 |
 v
Terraform plan
 |
 v
Security Scan
 |
 v
Approval / Policy Gate
 |
 v
Terraform apply
```

Production 更推荐：

```text
terraform plan
      |
      v
Save plan
      |
      v
Review
      |
      v
terraform apply tfplan
```



## Ansible

Ansible 和 Terraform 的区别非常重要。

Terraform：

```text
Infrastructure
```

Ansible：

```text
Configuration
Application deployment
OS management
Automation
```

例如：

```text
Terraform
   ↓
Create VM

Ansible
   ↓
Install Nginx
Configure Nginx
Create users
Deploy application
Start service
```


#### Ansible Architecture

经典：

```text
             Ansible Control Node
                     |
          ┌──────────┼──────────┐
          |          |          |
         SSH        SSH        SSH
          |          |          |
          v          v          v
        Web1       Web2       DB1
```

Ansible 通常是 **agentless**。

这是面试重点：

> Ansible generally does not require an agent on managed Linux hosts; it commonly uses SSH.


#### Inventory

例如：

```ini
[web]
web1 ansible_host=192.168.1.10 ansible_user=ubuntu
web2 ansible_host=192.168.1.11 ansible_user=ubuntu

[db]
db1 ansible_host=192.168.1.20 ansible_user=ubuntu
```

执行：

```bash
ansible -i inventory.ini all -m ping
```


#### Ad-Hoc Commands

检查：

```bash
ansible all -m ping
```

执行 uptime：

```bash
ansible all -a "uptime"
```

安装 nginx：

```bash
ansible web \
  -m apt \
  -a "name=nginx state=present" \
  --become
```

#### Ansible Playbook

最重要：

```yaml
---
- name: Install Nginx
  hosts: web
  become: true

  tasks:
    - name: Install Nginx
      ansible.builtin.apt:
        name: nginx
        state: present
```

运行：

```bash
ansible-playbook install_nginx.yml
```


#### Ansible Modules

常用：

```text
apt
yum
package
copy
template
file
service
systemd
user
command
shell
uri
lineinfile
```

面试不要只说：

```text
command = run command
```

更好的回答：

> Prefer purpose-built modules when available because they are generally more declarative and idempotent.


#### `command` vs `shell`


**command**

```yaml
- name: Check uptime
  ansible.builtin.command:
    cmd: uptime
```

不会通过 shell 执行。因此 shell features，例如：

```text
|
>
&&
```

不能直接依赖。

**shell**

```yaml
- name: Get nginx processes
  ansible.builtin.shell:
    cmd: ps aux | grep nginx
```

通过 shell 执行。

但：

> Don't use `shell` when a dedicated Ansible module can do the job.



#### Idempotency

Ansible 最重要的概念之一。

例如：

```yaml
state: present
```

第一次：

```text
Nginx doesn't exist
       ↓
Install
```

第二次：

```text
Nginx already exists
       ↓
No change
```

这就是：

```text
Idempotency
```

面试回答：

> An idempotent automation can be executed repeatedly and converges to the same desired state without making unnecessary changes.



#### Variables

例如：

```yaml
nginx_version: "1.24.*"
```

Playbook：

```yaml
- name: Install Nginx
  ansible.builtin.apt:
    name: "nginx={{ nginx_version }}"
    state: present
```

使用：

```text
{{ variable_name }}
```


### Facts

Ansible 可以收集机器信息：

```bash
ansible all -m setup
```

例如：

```text
OS
CPU
Memory
IP
Hostname
Architecture
```

使用：

```yaml
{{ ansible_facts['distribution'] }}
```


#### Handlers

Handlers 通常用于：

> Restart/reload something only when configuration changes.

例如：

```yaml
- name: Configure Nginx
  hosts: web
  become: true

  tasks:
    - name: Install Nginx
      ansible.builtin.apt:
        name: nginx
        state: present

    - name: Deploy config
      ansible.builtin.template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: Restart Nginx

  handlers:
    - name: Restart Nginx
      ansible.builtin.service:
        name: nginx
        state: restarted
```

流程：

```text
Config changed
      ↓
notify
      ↓
handler
      ↓
restart
```

#### Loops

```yaml
- name: Install packages
  ansible.builtin.apt:
    name: "{{ item }}"
    state: present
  loop:
    - nginx
    - curl
    - git
```

#### Conditionals

```yaml
- name: Restart service
  ansible.builtin.service:
    name: nginx
    state: restarted
  when: ansible_facts['os_family'] == "Debian"
```

#### Roles

生产环境不要把所有东西塞进一个 playbook。

推荐：

```text
roles/
└── nginx/
    ├── tasks/
    │   └── main.yml
    ├── handlers/
    │   └── main.yml
    ├── templates/
    ├── files/
    ├── vars/
    ├── defaults/
    └── meta/
```

创建：

```bash
ansible-galaxy init nginx
```

使用：

```yaml
- hosts: web
  roles:
    - nginx
```


#### Ansible Debugging

Syntax：

```bash
ansible-playbook site.yml --syntax-check
```

Dry run：

```bash
ansible-playbook site.yml --check
```

Diff：

```bash
ansible-playbook site.yml --check --diff
```

Verbose：

```bash
ansible-playbook site.yml -vvv
```


#### `become` vs `become_user`

你的原资料这里有一个容易出错的地方。

正确理解：

```yaml
become: true
```

表示：

> **Elevate privileges.**

**通常变成 root。**

如果指定：

```yaml
become_user: nginx
```

表示：

> Become the nginx user.

例如：

```yaml
- name: Run command as nginx
  ansible.builtin.command:
    cmd: whoami
  become: true
  become_user: nginx
```


## Chef

Chef 也是 Configuration Management。

核心对象：

```text
Chef
 |
 ├── Cookbook
 │     |
 │     ├── Recipe
 │     ├── Template
 │     ├── File
 │     └── Attribute
 |
 └── Node
```


#### Chef Terminology

| Concept   | Meaning                                |
| --------- | -------------------------------------- |
| Recipe    | **Defines configuration resources**        |
| Cookbook  | **Collection of recipes/supporting files** |
| Resource  | Package/service/file etc.              |
| Node      | Managed machine                        |
| Run List  | Ordered list of recipes/roles          |
| Attribute | Configuration data                     |


#### Chef Recipe Example

```ruby
package 'nginx' do
  action :install
end

service 'nginx' do
  action [:enable, :start]
end

file '/var/www/html/index.html' do
  content '<h1>Welcome to Chef</h1>'
end
```

Chef 的思想仍然是：

```text
Desired State
      ↓
Chef Client
      ↓
Node
```


#### Puppet

Puppet 同样属于 Configuration Management。

核心：

```text
Manifest
Module
Class
Resource
Node
Fact
```

### 35. Puppet Example

```puppet
class nginx {
  package { 'nginx':
    ensure => installed,
  }

  service { 'nginx':
    ensure => running,
    enable => true,
  }

  file { '/var/www/html/index.html':
    content => '<h1>Welcome to Puppet</h1>',
    mode    => '0644',
  }
}
```

应用：

```bash
puppet apply my_manifest.pp
```

Agent：

```bash
puppet agent --test
```


###  Terraform vs Ansible vs Chef vs Puppet

这个表建议你直接背。

|                | Terraform            | Ansible                          | Chef                   | Puppet               |
| -------------- | -------------------- | -------------------------------- | ---------------------- | -------------------- |
| Main purpose   | IaC                  | Config/Automation                | Config Management      | Config Management    |
| Typical target | Cloud infrastructure | Servers/apps                     | Servers                | Servers              |
| Language       | HCL                  | YAML                             | Ruby DSL               | Puppet DSL           |
| Agent          | No                   | Usually no                       | Usually yes            | Usually yes          |
| State          | Terraform state      | Facts/remote state model         | Chef server/node state | Puppet catalog/facts |
| Style          | Declarative          | Declarative + procedural modules | Declarative DSL        | Declarative          |
| Common use     | VNet/VM/AKS          | Configure/deploy                 | Enterprise config      | Enterprise config    |


### 最重要的架构理解

如果面试官问：

> You need to provision a VM and configure Nginx. What would you use?

非常好的回答：

```text
Terraform
   |
   | Provision VM
   v
Cloud
   |
   v
Ansible
   |
   ├── Install Nginx
   ├── Configure Nginx
   ├── Deploy config
   └── Start service
```

也就是说：

```text
Terraform = Infrastructure
Ansible   = Configuration
```

### Terraform + Ansible CI/CD

生产架构可以这样：

```text
             Git
              |
              v
          CI Pipeline
              |
      ┌───────┴────────┐
      |                |
 Terraform          Ansible
      |                |
      v                v
 Cloud             VM Config
      |                |
      └───────┬────────┘
              v
          Application
```

例如 Azure：

```text
Terraform
   ↓
Resource Group
   ↓
VNet
   ↓
Subnet
   ↓
VM / AKS
   ↓
Ansible
   ↓
Nginx / packages / users / config
```


### Terraform vs Ansible 面试陷阱

**Question**

> Can Ansible create infrastructure?

答案：

**可以。**

Ansible 可以通过 cloud modules 创建 VM、network 等资源。

但是：

> Terraform is generally a better fit for lifecycle management of declarative infrastructure, while Ansible is commonly used for configuration and operational automation.

---

# 40. Terraform vs Ansible：State 的区别

Terraform：

```text
State is fundamental.
```

它需要 state 来知道：

```text
What resources are managed
What changed
What needs to be created/updated/destroyed
```

Ansible：

```text
does not use Terraform-style centralized state
```

它通常：

```text
Connect
 ↓
Inspect
 ↓
Apply desired configuration
```


###  Ansible Troubleshooting Scenario

面试题

> Ansible playbook works on one server but fails on another. How do you troubleshoot?

回答：

```text
1. Verify connectivity
2. Check inventory
3. Check SSH credentials
4. Check privilege escalation
5. Check OS/distribution
6. Check Python availability
7. Check variables
8. Run with -vvv
9. Use --check --diff
10. Test the failing module independently
```

例如：

```bash
ansible web1 -m ping
```

然后：

```bash
ansible web1 -m setup
```

最后：

```bash
ansible-playbook site.yml -vvv
```


### Terraform Troubleshooting Scenario


**`terraform plan` unexpectedly wants to recreate a resource.**

排查：

```text
1. terraform plan
2. terraform state list
3. terraform state show <resource>
4. Check configuration
5. Check lifecycle settings
6. Check provider version
7. Check resource changes
```

重点理解：

```text
Configuration
      +
State
      +
Actual infrastructure
      ↓
Terraform Plan
```


### Terraform `lifecycle`

非常适合面试。

例如：

```hcl
resource "aws_instance" "example" {
  # ...

  lifecycle {
    prevent_destroy = true
  }
}
```

防止：

```text
terraform destroy
```

意外删除重要资源。


**`ignore_changes`**

例如：

```hcl
lifecycle {
  ignore_changes = [
    tags
  ]
}
```

意思：

> Ignore selected externally-managed changes.

但是面试注意：

> `ignore_changes` should be used carefully because it can hide configuration drift that Terraform would otherwise report.


### Configuration Management 的核心关键词

Ansible / Chef / Puppet 都应该围绕这几个词回答：

```text
Desired State

Idempotency

Automation

Consistency

Repeatability

Configuration Drift

Standardization
```

### 最值得背的 20 个面试问题

#### **Terraform**

**Q1. What is Terraform?**

> Terraform is a declarative IaC tool used to provision and manage infrastructure through configuration files.

**Q2. What is Terraform state?**

> State maps Terraform configuration to real infrastructure and allows Terraform to determine what changes are required.

**Q3. Why remote state?**

```text
Centralized

Collaboration

Locking

Security

Backup
```

**Q4. What is `terraform plan`?**

> It previews the changes Terraform intends to make without applying them.

**Q5. What is a module?**

> A reusable Terraform configuration that encapsulates infrastructure resources.

**Q6. How do you manage secrets?**

> Use a secret management system rather **than hardcoding secrets, and protect the Terraform state because it may contain sensitive values**.

**Q7. How do you import existing infrastructure?**

```bash
terraform import ...
```

**Q8. What is state locking?**

> It prevents concurrent Terraform operations from modifying the same state simultaneously.

#### Ansible

**Q9. What is Ansible?**

> **An agentless automation and configuration management tool** commonly using SSH for Linux hosts.

**Q10. What is a Playbook?**

> **A YAML file that defines plays, tasks, variables, handlers, and desired configuration**.

**Q11. What is idempotency?**

> Repeated execution converges to the same desired state without unnecessary changes.

**Q12. What are handlers?**

> **Tasks triggered by notifications, commonly used for service restart/reload after configuration changes**.

**Q13. What are roles?**

> **A reusable structure for organizing Ansible tasks, handlers, templates, files, variables, and metadata.**

**Q14. `command` vs `shell`?**

```text
command → no shell interpretation

shell   → executes through shell
```

**Q15. How do you troubleshoot Ansible?**

```bash
ansible ... -m ping
ansible-playbook ... --syntax-check
ansible-playbook ... --check
ansible-playbook ... --check --diff
ansible-playbook ... -vvv
```

#### Architecture

**Q16. Terraform vs Ansible?**

```text
Terraform → Provision infrastructure

Ansible   → Configure/automate systems
```

**Q17. Can Terraform and Ansible be used together?**

> Yes. Terraform can provision infrastructure and Ansible can configure the resulting hosts.

**Q18. Chef vs Puppet vs Ansible?**

重点：

```text
All are configuration-management/automation tools,
but they differ in architecture, agent model,
DSL, ecosystem and operational model.
```

**Q19. How do you prevent configuration drift?**

```text
Git
+
IaC
+
Configuration Management
+
CI/CD
+
Policy
+
Monitoring
```

**Q20. How would you design production IaC?**

建议回答：

```text
Git
 ↓
PR
 ↓
Terraform fmt/validate
 ↓
Security / Policy checks
 ↓
terraform plan
 ↓
Review
 ↓
Approved apply
 ↓
Remote State + Locking
 ↓
Cloud
```


#### 最终你要形成这张 DevOps Mental Model

你目前已经整理了：

```text
Linux
   ↓
Git
   ↓
Jenkins
   ↓
OWASP / SonarQube / Trivy
   ↓
Docker
   ↓
Terraform
   ↓
Ansible
   ↓
Kubernetes
   ↓
Argo CD
```

把它串起来就是一个完整的企业 DevOps Platform：

```text
                         Developer
                             |
                             v
                           Git
                             |
                    ┌────────┴────────┐
                    |                 |
                    v                 v
                Jenkins           Terraform
                    |                 |
          ┌─────────┼─────────┐       v
          |         |         |     Cloud
        Build     Test      Scan      |
          |         |         |       v
          └─────────┼─────────┘    Compute
                    |                 |
                    v                 v
               Container          Ansible
                Registry              |
                    |          Configuration
                    |                 |
                    └────────┬────────┘
                             |
                             v
                       Deployment Repo
                             |
                             v
                          Argo CD
                             |
                       GitOps Reconcile
                             |
                             v
                        Kubernetes
                             |
                             v
                        Application
```

**如果你是准备 Platform Operations / DevOps 面试，这一层的“工具之间怎么协作”比单独背 `terraform apply`、`ansible-playbook`、`argocd app sync` 更重要。**

