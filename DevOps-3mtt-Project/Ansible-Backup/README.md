# BACKUP AND RESTORE FILES ON A LINUX SERVER USING ANSIBLE


# Introduction

Date backup and restoration are essential practice for ensuring data safely and continuity in Linux server management. Ansible, an automation tools, simplifies these tasks by providing a scalable and repeatable solution. 

This project will guild me through creating Ansible playbooks to automate the backup and restore process for files on a Linux server.


# Objectives

1. Understand the basics of Ansible and its role in automation.
2. Set up an Ansible environment for managing Linyx servers.
3. Create a playbook to back up files to a remote or local directory.
4. Develop a playbook to restore files from a backup
5. Test and verify the backup and restore processes.


# Prerequisites

1. Linux Servers: At least one server to act as the target machine and an optional control machine for Ansible.
2. Ansible Installed: Ansible installed on the control machine.
3. SSH Access: SSH access between the control machine and target servers with public key authentication.
4. Tools: A text editor to create and edit Ansible playbooks.


# Project Tasks

Task 1- Install and Configure Ansible.

1. Task 1- Install and Configure Ansible.

2. Verify Ansible is installed on the control node/machine.


``` bash
ansible --version
```

2. Check SSH Authentication between the control machine and the target server.

Because I have already installed Ansible in our last project and work on the SSH authentication.


``` bash
cat inventory.ini

ansible -i inventory -m ping all
```

![](./Images/1.%20Ping.png)


# Task 2 - Set Up the Ansible Inventory FIle

1. Create an inventory file to define the target server.


``` bash
cat inventory.ini
```

![](./Images/2.%20Inventory.png)


# Task 3 - Create an Ansible Playbook to Back Up Files


1. Create a playbook file for backup.


``` bash
nano backup.yml
```

2. Add the following playbook content.


``` bash
- name: Backup files on the server
  hosts: linux_servers
  become: yes
  tasks:
    - name: Ensure script is present on all servers
      copy:
        src: /home/iemmanuel/install_k8s.sh  # Local path on Ansible controller
        dest: "{{ ansible_env.HOME }}/install_k8s.sh"  # Remote home dir of the user

    - name: Create backup directory
      file:
        path: /backup
        state: directory
        mode: '0755'

    - name: Copy files to backup directory
      copy:
        src: "{{ ansible_env.HOME }}/install_k8s.sh"
        dest: /backup/install_k8s.sh
        remote_src: yes
```

# Task 4 - Create an Ansible Playbook to Restore FIles

1. Create a playbook files for restoration.


``` bash
nano restore.yml
```

2. Add the following content.


``` bash
- name: Restore files from backup
  hosts: linux_servers
  become: yes
  tasks:
    - name: Restore script to user's home directory
      copy:
        src: /backup/install_k8s.sh
        dest: "{{ ansible_env.HOME }}/install_k8s.sh"
        remote_src: yes
```


# Task 5 - Test the Backup and Restore Functionality


1. Run the backup playbook.

``` bash
ansible-playbook -i inventory.ini backup.yml --ask-become-pass
```

![](./Images/3.%20BackUp.png)


2. Verify the backup directory and files on the target server.


``` bash
ls /backup
```

![](./Images/4.%20Ls%20BackUp.png)


3. Run the restore playbook


``` bash
ansible-playbook -i inventory.ini restore.yml --ask-become-pass
```

![](./Images/5.%20Restore%20Backup.png)


4. Verify the restored files in the original location on the target server.

``` bash
ls ~/install_k8s.sh
```

![](./Images/6.%20Confirm%20Restore.png)


# Conclusion

This project introducted me to automating file backup and restoration on a Linux server using Ansible. I set up an Ansible environment, created playbooks for backup and restoration, and verified the process. 

With these skills, I can extend the playbooks to include more servers, schedule regular backups, or integrate advanced options like compression or encryption.








