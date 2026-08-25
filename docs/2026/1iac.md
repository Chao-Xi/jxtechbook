# 1- 2026 IAC + Configuration Management


## 1 Terraform

**1-1 Main Terraform Configuration: `main.tf`**

**Terraform Configuration Example:**


```
terraform { required_providers {

aws = {

source = "hashicorp/aws"

version = "~> 4.16"

	} 
} 

required_version = ">= 1.2.0" 

}

resource "aws_instance" "app_server" { 

	ami = ”ami-08d70e59c07c61a3a" 
	instance_type = "t2.micro"

tags = {
	Name = var.instance_name
	}
}
```


**Input Variables: variables.tf**


```
variable "instance_name" {
	description = "Value of the Name tag for the EC2 instance"
	type = string
	default = "ExampleAppServerInstance"
}
```

**Output Values: outputs.tf:**

```
output "instance_id" {
	description = "ID of the EC2 instance"
	value = aws_instance.app_server.id
	}

output "instance_public_ip" {
	description = "Public IP address of the EC2 instance"
	value = aws_instance.app_server.public_ip
}
```


**Running the Configuration:**

- **Initialize Terraform:**   `terraform init`
- Apply the Configuration: `terraform apply`
	- Confirm by typing yes when prompted.

**Inspect Output Values:** `terraform output`

**Destroy the Infrastructure:** `terraform destroy`


### Terraform Advanced Configuration Use Cases

**1.Provider Configuration:**

```
provider "aws" {
region = "us-west-2"
}
```

**2.Resource Creation:**

```
resource "aws_instance" "example" {
	ami = "ami-0c55b159cbfafe1f0"
	instance_type = "t2.micro"
	tags = {
		Name = "ExampleInstance"
	}
}
```

**3. Variable Management:**

```
variable "region" {
	default = "us-west-2"
	}
	
provider "aws" {
	region = var.region
}
```

**4. State Management:**

Example for using remote state in S3:

```
terraform {
	backend "s3" {
	bucket = "my-tfstate-bucket"
	key = "terraform/state"
	region = "us-west-2"
	encrypt = true
	dynamodb_table = "terraform-locks"
	}
}
```


**5. Modules:**


```
module "vpc" {
	source = "terraform-aws-modules/vpc/aws"
	name = "my-vpc"
	cidr = "10.0.0.0/16"

azs = ["us-west-2a", "us-west-2b"]
public_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
private_subnets = ["10.0.3.0/24", "10.0.4.0/24"]

```


* `terraform init`: Initializes the Terraform configuration.
* `terraform fmt`: Formats configuration files.
* `terraform validate`: Validates the configuration files.
* `terraform plan`: Previews changes to be applied.
* `terraform apply`: Applies the changes to reach the desired state.
* `terraform destroy`: Destroys the infrastructure and removes it from the
* state.
* `terraform show`: Displays the current state of resources.
* `terraform state list`: Lists resources in the current state.
* `terraform taint <resource>:` Marks a resource for recreation.
* `terraform import <resource> <resource_id>:` Imports existing resources
* into Terraform.
* `terraform providers`: Lists the providers used in the configuration



**Terraform Best Practices**


- Use `Version Control ` to manage your Terraform code.
- Break your code into Modules for reusability.
- Use Remote State (e.g., AWS S3, Terraform Cloud) to store state files.
- Always run terraform plan before terraform apply.
- Use **`terraform fmt & terraform validate`** to ensure code correctnes
- **Avoid hardcoding secrets**; use environment variables or secret management
tools.
Keep configurations modular and well-documented.


## Ansible (playbooks, roles, inventory)

### Ansible Basics

* Check version: `ansible --version`
* Check inventory: `ansible-inventory --list -y`
* Ping all hosts: `ansible all -m ping`
* Run command on all hosts: `ansible all -a "uptime"`

**Inventory & Configuration**

**Default inventory: `/etc/ansible/hosts`**

Custom inventory:


```
ansible -i inventory.ini all -m ping
```

**Define hosts in inventory.ini:**

```
[web]
web1 ansible_host=192.168.1.10 ansible_user=ubuntu

[db]
db1 ansible_host=192.168.1.20 ansible_user=root
```

**Ad-Hoc Commands**

Run as a specific user:   `ansible all -m ping -u ubuntu --become`

Copy file to remote host:  `ansible all -m copy -a "src=/etc/hosts dest=/tmp/hosts"`

Install a package (example: nginx): `ansible all -m apt -a "name=nginx state=present" --become`


**Playbook Structure**


```
- name: Install Nginx
	hosts: web
	become: yes
	tasks:
		- name: Install Nginx
			apt:
				name: nginx
				state: present
```


Run the playbook: `ansible-playbook install_nginx.yml`


**5.Variables & Facts**

Define variables in vars.yml:

`nginx_version: latest`

Use variables in playbook:

```
- name: Install Nginx
	apt:
	name: nginx={{ nginx_version }}
	state: present
```


Display all facts:  `ansible all -m setup`


**6. Handlers & Notifications**


```
- name: Restart Nginx
  hosts: web
  become: yes
  tasks:
   - name: Install Nginx
     apt:
       name: nginx
       state: present
     notify: Restart Nginx
     
  handlers:
	- name: Restart Nginx
	service:
		name: nginx
		state: restarted
```


**Loops & Conditionals**

```
- name: Install multiple packages
  apt:
	name: "{{ item }}"
	state: present
  loop:
	- nginx
	- curl
	- git
```


**Conditional execution:**


```
- name: Restart service only if Nginx is installed
  service:
   name: nginx
   state: restarted
  when: ansible_facts['pkg_mgr'] == 'apt'
```


**8. Roles & Reusability**

Create a role:

**`Ansible-galaxy init my_role`**

Run a role in a playbook:

```
- hosts: web
	roles:
		- my_role
```

**9. Debugging & Testing**

```
Debug a variable:
- debug:
msg: "The value of nginx_version is {{ nginx_version }}"
```

Check playbook syntax:

`ansible-playbook myplaybook.yml --syntax-check`

**Run in dry mode:**

`ansible-playbook myplaybook.yml --check`


**Playbook Structure**


```
- name: Example Playbook
	hosts: all
	become: yes
	tasks:
		- name: Print a message
	debug:
		msg: "Hello, Ansible!"
```

**Defining Hosts & Privilege Escalation**

```
- name: Install Nginx
	hosts: web
	become: yes
```


**Run as a specific user:**

```
- name: Install package
	apt:
	  name: nginx
	  state: present
	  become_user: root
```

### 3 **Tasks & Modules**

```
- name: Ensure Nginx is installed
  hosts: web
  become: yes
  tasks:
	- name: Install Nginx
		apt:
			name: nginx
			state: present
```


Common Modules

- command: Run shell commands
- copy: Copy files
- service: Manage services
- user: Manage users
- file: Set file permissions


## 3 Configuration Management - Chef (recipes, cookbooks)


### Basic Concepts


- **Recipe**: Defines a set of resources to configure a system.
- **Cookbook**: A collection of recipes, templates, and attributes.
- **Resource**: Represents system objects (e.g., package, service, file).
- **Node**: A machine managed by Chef.
- **Run List**: Specifies the order in which recipes are applied.
- **Attributes**: Variables used to customize recipes


Commands

* chef-client # Run Chef on a node
* knife cookbook create my_cookbook # Create a new cookbook
* knife node list # List all nodes
* knife role list # List all roles
* chef-solo -c solo.rb -j run_list.json # Run Chef in solo mode





Example Recipe

```
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


**Basic Concepts**

* **Manifest**: A file defining resources and configurations (.pp).
* **Module**: A collection of manifests, templates, and files.
* **Class**: A reusable block of Puppet code.
* **Node**: A system managed by Puppet.
* **Fact**: System information collected by Facter.
* **Resource**: The basic unit of configuration (e.g., package, service).


## 4  Puppet (manifests, modules)

Commands

* `puppet apply my_manifest.pp` # Apply a local manifest
* `puppet module install my_module` # Install a module
* `puppet agent --test` # Run Puppet on an agent node
* `puppet resource service nginx` # Check a resource state


Basic Concepts

* **Manifest**: A file defining resources and configurations (.pp).
* **Module**: A collection of manifests, templates, and files.
* **Class**: A reusable block of Puppet code.
* **Node**: A system managed by Puppet.
* **Fact**: System information collected by Facter.
* **Resource**: The basic unit of configuration (e.g., package, service).



Commands

* `puppet apply my_manifest.pp `# Apply a local manifest
* `puppet module install my_module` # Install a module
* `puppet agent --test` # Run Puppet on an agent node
* `puppet resource service nginx` # Check a resource state


```
puppet
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
	mode => '0644',
	}
}
```