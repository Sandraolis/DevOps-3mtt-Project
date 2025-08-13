# SETTING UP ANSIBLE ON A LINUX SERVER

# Introduction

Ansible is a powerful automation tool that simplifies the management of IT infrastructure. 

Setting up Ansible on a Linux server is the first step toward leverging its capabilities.This project guild me through installing and configuring Ansible on a Linux server, allowing you to automate tasks and manage servers effectively.

# Objective

- Understand what Ansible is and how it works.
- Install and configure Ansible on a Linux control node.
- Set up SSH key-based authentication for target nodes.
- Create an Ansible inventory file.
- Verify Ansible setup by running basic commands.

# Prerequisites

- **Linux Machine:** A Linux server or virtual machine to act as the controller node.
- **Target Machine(s):** At least one additional Linux server or virtual machine for ansible to manage.
- **SSH Access:** Access to target nodes with SSH
- **Tools:** basic knowledge of the Linux commands line and a tect editor.


# Project Tasks

## Task 1- Install Ansible on the Control Node

1. Update the package repository.

``` bash
sudo apt update
```

2. Install Ansible.

``` bash
sudo apt install ansible -y
```

3. Verify the installation

``` bash
ansible --version
```

![](./Images/1.%20ansible.png)

The output display the installed Ansible version.

# Task 2- Configure SSH Key-Based Authentication

1. Generate an SSH key pair on the control node:

``` bash
ssh-keygen -t rsa
```

Press `Enter` to accept the default path and passphrase.

![](./Images/2.%20keygen.png)


2. Copy the public key to the target machine(s).

``` bash
ssh-copy-id ubuntu@private ip of the server
```

[](./Images/3.%20SSH%20Copied.png)


3. Test SSH access without password


``` bash
ssh iyanu@192.168.0...
```

I was able to ssh into the account without password, which makes the ssh successful.

![](./Images/4.%20SSH.png)


# Task 3- Create an Inventory File

1. Create a directory for Ansible configuration.

``` bash
mkdir ~/ansible

cd ~/ansible
```

2. Create an inventory file

``` bash
nano inventory.ini
```

3. Add the target machine details to the inventory.

``` bash
[linux_servers]
target1 ansible_host=192.168.0... ansible_user=iemmanuel
target2 ansible_host=192.168.0... ansible_user=iyanu
```

![](./Images/5.%20Inventory%20File.png)


# Task 4 - Test Ansible Connectivity

1. Test Ansible connectivity to the target machine.


``` bash
ansible -i inventory.ini linux_servers -m ping
```

![](./Images/6.%20Ping.png)


# Task 5 - Run a Simple Ansible Ad-Hoc Command

1. Run a command to check the uptime of target machines

``` bash
ansible -i inventory.ini linux_servers -m command -a "uptime"
```

![](./Images/7.%20Uptime.png)


2. Run a command to check dick usage.

``` bash
ansible -i inventory.ini linux_servers -m shell -a "df -h"
```

![](./Images/8.%20Disk%20Usage.png)


# Conclusion

This project demostrated how to set up Ansible on a Linux server and configure it to manage target machines. I have installed Ansible, configured SSH access, created an inventory file, and verified connectivity using ad-hoc command. 

With this foundation, I am now prepared to explore more advanced Ansible functionalities like writing playbooks and managing complex infrastructures.
