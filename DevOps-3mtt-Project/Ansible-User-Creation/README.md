# AUTOMATE USER CREATION ON LINUX SERVER USING ANSIBLE

# Introduction

Managing user creation is a common admistrative task for Linux servers. Manually creating and managing user accounts can become tedious, especially on multiple servers. Ansible simplifies this process by automating user creation with playbooks. This project will guild you in creating an Ansible playbook to automate user creation on a Linux server.

# Objectives

1. Understading the basic of Ansible and its automation capabilities.
2. Set up an Ansible environment to manage Linux servers.
3. Create an Ansible playbook to automate user creation.
4. Configure additional setting like home directory, groups, and SSH access.
5. Verify the user creation process and test access.


# Prerequisites

1. `Linux Servers`: At least one Linux server to act as the target machine and an optional control machine for Ansible.

2. `Ansible Installed`: Ansible installed on the control mahcine.

3. `SSH Access`: SSH access between the control machine and target servers with public key authentication.

4. `Tools`: A text editor to create and edit Ansible playbooks.


# Project 


### Task 1- Install and Configure Ansible

1. Verify Ansible is installed on the control node/machine.

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


# Task 3- Create an Ansible Playbook to Automate User Creation

1. Create a playbook file for user creation.

``` bash
nano create_users.yml
```

2. Add the following playbook content.

``` bash
- name: Automate user creation
  hosts: linux_servers
  become: yes
  tasks:
    - name: Create a new user
      user:
        name: "{"{ item.username "}}"
        state: present
        shell: /bin/bash
        create_home: yes
      with_items:
        - {" username: \"user1\" "}
        - {" username: \"user2\" "}
```

# Task 4: Configure Additional User Settings

1. Update the playbook to include group and SSH key configuration.

``` bash
# cat >> create_users.yml

- name: Automate user creation
  hosts: linux_servers
  become: yes
  tasks:
    - name: Create a new user with additional settings
      user:
        name: "{"{ item.username "}}"
        state: present
        shell: /bin/bash
        create_home: yes
        groups: "{"{ item.groups "}}"
      with_items:
        - {" username: \"user1\", groups: \"sudo\" "}
        - {" username: \"user2\", groups: \"docker\" "}

    - name: Add SSH key for the users
      authorized_key:
        user: "{"{ item.username "}}"
        state: present
        key: "{"{ lookup('file', item.ssh_key) "}}"
      with_items:
        - {" username: \"user1\", ssh_key: \"/path/to/user1.pub\" "}
        - {" username: \"user2\", ssh_key: \"/path/to/user2.pub\" "}
```

# Task 5 - Verify User Creation and Test Login

1. Run the playbook to create users.

``` bash
ansible-playbook -i inventory.ini create_users.yml --ask-become-pass
```

![](./Images/3.%20Ansible%20Run.png)


2. Verify the users were created on the target server.

``` bash
cat /etc/passwd
ls /home
```

![](./Images/4.%20Verified.png)


3. Test SSH access for the newly created users.

``` bash
ssh user1@<target-server-ip>
ssh user2@<target-server-ip>
```

![](./Images/5.%20SSH%20Access.png)


# Conclusion

In this project, I automated the creation of user account on a Linux server using Ansible. I have learnt how to write an Ansible playbook for user creation, configure additional settings like groups and SSH access, and verify the process. With these skills, I can manage user accounts efficiently across multiple servers and ectend the playbook for advanced configuration like password policies and user deletion.

