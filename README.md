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

This guide will help you set up your EC2 instances, configure them for management with Ansible, and perform essential tasks. Let me know if you need more details or run into any issues!
