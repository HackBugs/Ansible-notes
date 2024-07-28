# Ansible-notes

### Step-by-Step Guide for EC2 Setup with Ansible

#### 1. **Set Up EC2 Instances**

1. **Launch EC2 Instances:**
   - **Master Instance**: This instance will manage the other instances using Ansible.
   - **Server Instances**: Three server instances to be managed.

   **EC2 Console Steps:**
   - Go to the [EC2 Management Console](https://console.aws.amazon.com/ec2/).
   - Click on “Launch Instance”.
   - Select an Amazon Machine Image (AMI) (e.g., Ubuntu Server).
   - Choose an instance type (e.g., t2.micro for testing).
   - Configure instance details, add storage, and add tags as needed.
   - Configure security group rules to allow SSH access (port 22) and any other required ports.
   - Review and launch instances.

2. **Get Public IPs:**
   - After launching, note down the public IP addresses of the instances from the EC2 console.

#### 2. **Connect to the Master Instance to your local computer**

Use the SSH key to connect to the master instance:
```bash
ssh -i "ansible-master-key.pem" ubuntu@ec2-13-200-215-59.ap-south-1.compute.amazonaws.com
```

#### 3. **Install Ansible on Master Instance**

Once connected to the master instance, install Ansible:
```bash
sudo apt-add-repository ppa:ansible/ansible
sudo apt update
sudo apt install ansible -y
```

#### 4. **Configure Ansible Inventory**

Create an inventory file on the master instance:
```bash
sudo nano /etc/ansible/hosts
```

Add the following configuration to the inventory file:
```ini
[servers]
server1 ansible_host=<server1-public-ip>
server2 ansible_host=<server2-public-ip>
server3 ansible_host=<server3-public-ip>

[all:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_user=ubuntu
ansible_ssh_private_key_file=/path/to/your/private-key.pem
```

Replace `<server1-public-ip>`, `<server2-public-ip>`, `<server3-public-ip>`, and `/path/to/your/private-key.pem` with the actual IPs and path to your private key file.

#### 5. **Transfer SSH Key to Server Instances**

To transfer the SSH key to the server instances:
```bash
scp -i "ansible-master-key.pem" ansible-master-key.pem ubuntu@ec2-13-200-215-59.ap-south-1.compute.amazonaws.com:/home/ubuntu/keys/
```

#### 6. **Set Permissions for SSH Key**

On each server instance, set the correct permissions for the SSH key:
```bash
chmod 600 /home/ubuntu/keys/ansible-master-key.pem
```

#### 7. **Test Ansible Configuration**

Test the Ansible configuration to ensure everything is set up correctly:
```bash
ansible-inventory --list
ansible all -m ping
```

- `ansible-inventory --list`: Lists all the hosts and groups defined in your Ansible inventory.
- `ansible all -m ping`: Pings all the hosts to check connectivity.

#### 8. **Basic Ansible Commands**

- **update all server with master server:**
  ```sh
  ansible servers -a "sudo apt update"
  ```
  
- **Run a Command on All Servers:**
  ```
  ```bash
  ansible all -m shell -a "uptime"
  ```

- **Apply a Playbook:**
  ```bash
  ansible-playbook /path/to/your/playbook.yml
  ```

- **Check Syntax of a Playbook:**
  ```bash
  ansible-playbook /path/to/your/playbook.yml --syntax-check
  ```

### Summary of Important Commands

- **SSH to EC2 Instances:**
  ```bash
  ssh -i "your-key.pem" ubuntu@<public-ip>
  ```

- **Transfer Files Using SCP:**
  ```bash
  scp -i "your-key.pem" local-file ubuntu@<public-ip>:/remote/path
  ```

- **Set SSH Key Permissions:**
  ```bash
  chmod 600 /path/to/your/private-key.pem
  ```

- **Ansible Commands:**
  - **List Inventory:**
    ```bash
    ansible-inventory --list
    ```
  - **Ping All Hosts:**
    ```bash
    ansible all -m ping
    ```
  - **Run Playbook:**
    ```bash
    ansible-playbook /path/to/your/playbook.yml
    ```
---------------------------------------------------------------------------------------------------------------
To update the three server instances from the master instance using Ansible, you can follow these steps:

### Step-by-Step Guide to Update Server Instances Using Ansible

#### 1. **Prepare Ansible Playbook**

Create an Ansible playbook to handle the updates. This playbook will include tasks to update the server instances.

1. **Create the Playbook File**

On the master instance, create a playbook file, e.g., `update-servers.yml`:

```bash
nano /home/ubuntu/update-servers.yml
```

2. **Add the Following Content**

```yaml
---
- name: Update all servers
  hosts: servers
  become: yes
  tasks:
    - name: Update the package index
      apt:
        update_cache: yes

    - name: Upgrade all packages
      apt:
        upgrade: dist

    - name: Remove unused packages
      apt:
        autoremove: yes
```

This playbook does the following:
- Updates the package index.
- Upgrades all installed packages.
- Removes unused packages.

#### 2. **Run the Ansible Playbook**

Execute the playbook from the master instance to update the servers:

```bash
ansible-playbook /home/ubuntu/update-servers.yml
```

This command will run the `update-servers.yml` playbook on all servers listed in your Ansible inventory.

### Additional Information

#### **1. Verify Inventory Configuration**

Ensure that your inventory file (`/etc/ansible/hosts`) is properly configured to include the servers:
- INI Configuration
```ini
[servers]
server1 ansible_host=<server1-public-ip>
server2 ansible_host=<server2-public-ip>
server3 ansible_host=<server3-public-ip>

[all:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/ubuntu/private-key.pem
```

#### **2. Check for Connectivity**

Before running the playbook, you can check if Ansible can connect to all servers:

```bash
ansible all -m ping
```

#### **3. Run a Dry-Run**

To ensure your playbook is correct without making changes, you can use the `--check` option:

```bash
ansible-playbook /home/ubuntu/update-servers.yml --check
```

This will show what changes would be made without actually applying them.

### Summary of Commands

- **Create/Modify Playbook:**
  ```bash
  nano /home/ubuntu/update-servers.yml
  ```

- **Run Ansible Playbook:**
  ```bash
  ansible-playbook /home/ubuntu/update-servers.yml
  ```

- **Check Connectivity:**
  ```bash
  ansible all -m ping
  ```

- **Dry-Run Playbook:**
  ```bash
  ansible-playbook /home/ubuntu/update-servers.yml --check
  ```

- With this setup, you can efficiently manage updates across your server instances from the master instance using Ansible.
------------------------------------------------------------------------------------------------------------------------

# (i) - To manage servers at a company level, you typically need a more comprehensive and structured approach. This includes:

1. **Role-Based Playbooks**: Using Ansible roles to organize tasks into reusable units.
2. **Advanced Configuration Management**: Managing more complex configurations, such as user accounts, firewall rules, and application deployments.
3. **Security and Compliance**: Ensuring servers meet security standards and compliance requirements.
4. **Monitoring and Reporting**: Implementing monitoring and reporting tools to track the health and performance of the servers.

Here's a more advanced example that includes these elements:

### 1. **Directory Structure**

Organize your Ansible project with the following structure:

```
/home/ubuntu/ansible/
├── ansible.cfg
├── inventory
│   └── hosts
├── playbooks
│   └── site.yml
└── roles
    ├── common
    │   └── tasks
    │       └── main.yml
    ├── nginx
    │   └── tasks
    │       └── main.yml
    └── security
        └── tasks
            └── main.yml
```

### 2. **Ansible Configuration File**

Create an `ansible.cfg` file to configure Ansible settings:

```ini
[defaults]
inventory = ./inventory/hosts
remote_user = ubuntu
private_key_file = /home/ubuntu/private-key.pem
host_key_checking = False
```

### 3. **Inventory File**

Define your inventory in `inventory/hosts`:

```ini
[servers]
server1 ansible_host=<server1-public-ip>
server2 ansible_host=<server2-public-ip>
server3 ansible_host=<server3-public-ip>

[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

### 4. **Site Playbook**

Create a playbook that includes roles in `playbooks/site.yml`:

```yaml
---
- name: Configure all servers
  hosts: servers
  become: yes
  roles:
    - role: common
    - role: security
    - role: nginx
```

### 5. **Common Role**

Define common tasks in `roles/common/tasks/main.yml`:

```yaml
---
- name: Update the package index
  apt:
    update_cache: yes

- name: Upgrade all packages
  apt:
    upgrade: dist

- name: Install common packages
  apt:
    name: "{{ item }}"
    state: present
  with_items:
    - curl
    - git
    - htop
```

### 6. **Security Role**

Define security-related tasks in `roles/security/tasks/main.yml`:

```yaml
---
- name: Ensure UFW is installed
  apt:
    name: ufw
    state: present

- name: Allow OpenSSH
  ufw:
    rule: allow
    name: OpenSSH

- name: Enable UFW
  ufw:
    state: enabled
    policy: deny
    direction: incoming

- name: Disable root login
  lineinfile:
    path: /etc/ssh/sshd_config
    regexp: '^PermitRootLogin'
    line: 'PermitRootLogin no'
    state: present

- name: Restart SSH service
  service:
    name: ssh
    state: restarted
```

### 7. **Nginx Role**

Define Nginx-related tasks in `roles/nginx/tasks/main.yml`:

```yaml
---
- name: Install nginx
  apt:
    name: nginx
    state: present

- name: Ensure nginx is running
  service:
    name: nginx
    state: started
    enabled: yes

- name: Deploy a simple HTML page
  copy:
    src: /home/ubuntu/index.html
    dest: /var/www/html/index.html
    owner: www-data
    group: www-data
    mode: '0644'
```

### 8. **Execute the Playbook**

Run the playbook to configure all servers:

```bash
ansible-playbook /home/ubuntu/ansible/playbooks/site.yml
```

### Summary of Commands and Tasks

- **Inventory File**: Defines the servers and variables.
- **Ansible Configuration File**: Sets Ansible configurations.
- **Site Playbook**: Includes all roles and tasks to be applied.
- **Common Role**: Updates packages and installs common tools.
- **Security Role**: Configures security settings like UFW and SSH.
- **Nginx Role**: Installs and configures Nginx, deploys a web page.

- This structure allows for scalable and maintainable configuration management across multiple servers, addressing common company-level requirements such as security, compliance, and modularity.
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# (ii) - To create a more comprehensive and production-level Ansible configuration for managing servers, we'll need to incorporate several best practices:

1. **Role-Based Architecture**: Organize tasks into roles.
2. **Variables**: Use variables for configuration settings.
3. **Handlers**: Use handlers to manage service restarts.
4. **Templates**: Use templates for configuration files.
5. **Security and Compliance**: Ensure secure practices and compliance checks.
6. **Modularity**: Break down tasks into reusable components.

Here is a more advanced setup:

### Directory Structure

Organize your Ansible configuration with a directory structure:

```
ansible/
├── playbooks/
│   └── site.yml
├── roles/
│   ├── common/
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   └── handlers/
│   │       └── main.yml
│   ├── webserver/
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   ├── handlers/
│   │   │   └── main.yml
│   │   ├── templates/
│   │   │   └── index.html.j2
│   │   └── vars/
│   │       └── main.yml
├── inventory/
│   └── hosts
└── ansible.cfg
```

### 1. **Inventory File**

#### Inventory File: `ansible/inventory/hosts`

```ini
[servers]
server1 ansible_host=<server1-public-ip>
server2 ansible_host=<server2-public-ip>
server3 ansible_host=<server3-public-ip>

[all:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/ubuntu/private-key.pem
```

### 2. **Ansible Configuration**

#### Configuration File: `ansible/ansible.cfg`

```ini
[defaults]
inventory = inventory/hosts
remote_user = ubuntu
private_key_file = /home/ubuntu/private-key.pem
host_key_checking = False
retry_files_enabled = False
```

### 3. **Playbook File**

#### Playbook File: `ansible/playbooks/site.yml`

```yaml
---
- name: Apply common configuration to all servers
  hosts: servers
  become: yes
  roles:
    - common
    - webserver
```

### 4. **Roles**

#### Role: Common

##### Tasks File: `ansible/roles/common/tasks/main.yml`

```yaml
---
- name: Update the package index
  apt:
    update_cache: yes

- name: Upgrade all packages
  apt:
    upgrade: dist

- name: Ensure Python is installed
  apt:
    name: python3
    state: present
```

##### Handlers File: `ansible/roles/common/handlers/main.yml`

```yaml
---
# No handlers for common role
```

#### Role: Webserver

##### Tasks File: `ansible/roles/webserver/tasks/main.yml`

```yaml
---
- name: Install nginx
  apt:
    name: nginx
    state: present
  notify:
    - Restart nginx

- name: Deploy a simple HTML page
  template:
    src: index.html.j2
    dest: /var/www/html/index.html
    owner: www-data
    group: www-data
    mode: '0644'

- name: Ensure nginx is running
  service:
    name: nginx
    state: started
    enabled: yes
```

##### Handlers File: `ansible/roles/webserver/handlers/main.yml`

```yaml
---
- name: Restart nginx
  service:
    name: nginx
    state: restarted
```

##### Variables File: `ansible/roles/webserver/vars/main.yml`

```yaml
---
web_content: |
  <!DOCTYPE html>
  <html>
  <head>
      <title>Welcome to My Server</title>
  </head>
  <body>
      <h1>Hello, Ansible World!</h1>
  </body>
  </html>
```

##### Template File: `ansible/roles/webserver/templates/index.html.j2`

```html
{{ web_content }}
```

### Running the Playbook

1. **Connect to the Master Instance:**

   ```bash
   ssh -i "ansible-master-key.pem" ubuntu@<master-instance-public-ip>
   ```

2. **Navigate to the Ansible Directory:**

   ```bash
   cd /home/ubuntu/ansible
   ```

3. **Run the Ansible Playbook:**

   ```bash
   ansible-playbook playbooks/site.yml
   ```

### Additional Commands

- **Check Inventory:**

  ```bash
  ansible-inventory --list
  ```

- **Ping All Hosts:**

  ```bash
  ansible all -m ping
  ```

- **Run Playbook with Check Mode:**

  ```bash
  ansible-playbook playbooks/site.yml --check
  ```

- **Test Configuration:**

  ```bash
  ansible all -m shell -a "curl http://localhost"
  ```
- This setup provides a more scalable and maintainable way to manage configurations using Ansible, suitable for more complex environments and real-world use cases.
---------------------------------------------------------------------------------------------------------------------------------------------------------------------
# (iii) - Here's a step-by-step guide to set up the specified directory structure and files for managing three servers using Ansible. 

### 1. **Create the Directory Structure**

First, let's create the required directories and files.

```bash
mkdir -p ansible/{playbooks,roles/{common/{tasks,handlers},webserver/{tasks,handlers,templates,vars}},inventory}
touch ansible/{ansible.cfg,playbooks/site.yml,inventory/hosts,roles/common/tasks/main.yml,roles/common/handlers/main.yml,roles/webserver/tasks/main.yml,roles/webserver/handlers/main.yml,roles/webserver/templates/index.html.j2,roles/webserver/vars/main.yml}
```
-
```sh
mkdir -p /u01/app/oracle/product/19.0.0/dbhome_1
mkdir -p /u02/oradata
chown -R oracle:oinstall /u01 /u02
chmod -R 775 /u01 /u02
```

### 2. **Configure Inventory File**

Edit the inventory file to define your servers.

#### Inventory File: `ansible/inventory/hosts`

```ini
[servers]
server1 ansible_host=<server1-public-ip>
server2 ansible_host=<server2-public-ip>
server3 ansible_host=<server3-public-ip>

[all:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/ubuntu/private-key.pem
```

### 3. **Create Ansible Configuration File**

Configure the Ansible settings.

#### Configuration File: `ansible/ansible.cfg`

```ini
[defaults]
inventory = inventory/hosts
remote_user = ubuntu
private_key_file = /home/ubuntu/private-key.pem
host_key_checking = False
retry_files_enabled = False
```

### 4. **Create Playbook**

Create the main playbook to apply configuration tasks.

#### Playbook File: `ansible/playbooks/site.yml`

```yaml
---
- name: Apply common configuration to all servers
  hosts: servers
  become: yes
  roles:
    - common
    - webserver
```

### 5. **Create Roles**

#### Role: Common

##### Tasks File: `ansible/roles/common/tasks/main.yml`

```yaml
---
- name: Update the package index
  apt:
    update_cache: yes

- name: Upgrade all packages
  apt:
    upgrade: dist

- name: Ensure Python is installed
  apt:
    name: python3
    state: present
```

##### Handlers File: `ansible/roles/common/handlers/main.yml`

```yaml
---
# No handlers for common role
```

#### Role: Webserver

##### Tasks File: `ansible/roles/webserver/tasks/main.yml`

```yaml
---
- name: Install nginx
  apt:
    name: nginx
    state: present
  notify:
    - Restart nginx

- name: Deploy a simple HTML page
  template:
    src: index.html.j2
    dest: /var/www/html/index.html
    owner: www-data
    group: www-data
    mode: '0644'

- name: Ensure nginx is running
  service:
    name: nginx
    state: started
    enabled: yes
```

##### Handlers File: `ansible/roles/webserver/handlers/main.yml`

```yaml
---
- name: Restart nginx
  service:
    name: nginx
    state: restarted
```

##### Variables File: `ansible/roles/webserver/vars/main.yml`

```yaml
---
web_content: |
  <!DOCTYPE html>
  <html>
  <head>
      <title>Welcome to My Server</title>
  </head>
  <body>
      <h1>Hello, Ansible World!</h1>
  </body>
  </html>
```

##### Template File: `ansible/roles/webserver/templates/index.html.j2`

```html
{{ web_content }}
```

### Running the Playbook

1. **Connect to the Master Instance:**

   ```bash
   ssh -i "ansible-master-key.pem" ubuntu@<master-instance-public-ip>
   ```

2. **Navigate to the Ansible Directory:**

   ```bash
   cd /home/ubuntu/ansible
   ```

3. **Run the Ansible Playbook:**

   ```bash
   ansible-playbook playbooks/site.yml
   ```

### Additional Commands

- **Check Inventory:**

  ```bash
  ansible-inventory --list
  ```

- **Ping All Hosts:**

  ```bash
  ansible all -m ping
  ```

- **Run Playbook with Check Mode:**

  ```bash
  ansible-playbook playbooks/site.yml --check
  ```

- **Test Configuration:**

  ```bash
  ansible all -m shell -a "curl http://localhost"
  ```

This setup provides a scalable and maintainable way to manage configurations using Ansible, suitable for more complex environments and real-world use cases.
--------------------------------------------------------------------------------------------------------------------------------------------------------------

☑️ # New Method to install ansible from starting
- https://youtu.be/kE-6KDyf-0o?si=P5glRR-WToQzdyWC
### Download on your EC2 Master machine with wget cmd
- https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm   - Download latest
- yum install epel/epel-release-latest-9.noarch.rpm
- yum update -y
- yum install git python python-level python-pip openssl ansible -y
- vi /etc/ansible/hosts
- inside "hosts" file make group with [] like [host] paste ip of node, like [developer] paste ip of node, like [Tester] paste ip of node,
- vi /etc/ansible/ansible.cfg - write YAML code what you want configure on host node pc
- inside of "ansible.cfg" delete # this is for comment as per your requrement
- cmd - visudo
- vi /etc/ssh/sshd.config -- delete # this is for comment as per your requrement
- cmd - service sshd restart
- ssh "node-ips-paste"
- ssh-keygen - generate the key on host-master machine that will not ask everytime to node machine for password
- ls -a
- .ssh
- cd .ssh/
- ssh-copy-id node1@ip_address - add key on node machine that will not ask everytime to host machine for password

### This all in ad-hoc commands
- cmd - ansible all --list-host
- cmd - ansible demo --list-host
- cmd - ansible demo[0] --list-host -- get single node info
- cmd - ansible demo[1] --list-host -- get single node info
- cmd - ansible demo[0:10] --list-host -- get multipe node info
- cmd - ansible demo -a "sudo yum install httpd -y" -- install anyting on node machine
- cmd - ansible all -a "sudo yum install httpd -y"install anyting on all machine

### This is module where we write yml file or script run one time to configure on every node
- Ansible moudule --location /etc/ansible/hosts
- cmd - ansible all -b -m "sudo yum -a "pkg = httpd state = present" - m means module
- cmd - ansible all -b -m "sudo yum -a "pkg = nmap state = present" - m means module
- cmd ansible demo -m setup
  
- install = present
- Uninstall = absent
- update = latest

To create an Ansible playbook that adds a group, adds a user to that group, and installs a package, you can follow this structure. This playbook will use the appropriate Ansible modules to perform these tasks.

### Ansible Playbook
- follow indentation
- https://youtu.be/uyFrrKju4Es?si=vN26RbUmsyBdLCYn

```yaml
---
- name: Manage groups, users, and install packages
  hosts: all
  become: true

  tasks:
    - name: Add a group
      group:
        name: mygroup
        state: present

    - name: Add a user and add to group
      user:
        name: myuser
        groups: mygroup
        state: present

    - name: Install nmap package
      yum:
        name: nmap
        state: present
```

### Running the Playbook

Save the above content to a file named `playbook.yml`. Then, run the playbook using the following command:

```bash
ansible-playbook -i inventory playbook.yml
```

Where `inventory` is your inventory file specifying the hosts you want to run this playbook against.

### Ansible Command Example

If you prefer to run tasks directly using the Ansible command line, here’s how you can do it:

1. **Add a group**:
    ```bash
    ansible all -b -m group -a "name=mygroup state=present"
    ```

2. **Add a user and add to group**:
    ```bash
    ansible all -b -m user -a "name=myuser groups=mygroup state=present"
    ```

3. **Install nmap package**:
    ```bash
    ansible all -b -m yum -a "name=nmap state=present"
    ```

### Box Version (for Copy-Pasting)

```yaml
---
- name: Manage groups, users, and install packages
  hosts: all
  become: true

  tasks:
    - name: Add a group
      group:
        name: mygroup
        state: present

    - name: Add a user and add to group
      user:
        name: myuser
        groups: mygroup
        state: present

    - name: Install nmap package
      yum:
        name: nmap
        state: present
```

```bash
# Add a group
ansible all -b -m group -a "name=mygroup state=present"

# Add a user and add to group
ansible all -b -m user -a "name=myuser groups=mygroup state=present"

# Install nmap package
ansible all -b -m yum -a "name=nmap state=present"
```

This should help you manage groups, users, and packages using Ansible effectively.




