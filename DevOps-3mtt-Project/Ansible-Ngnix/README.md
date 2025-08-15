# DEPLOY AND CONFIGURE NGINX WEB SERVER USING ANSIBLE.


# Introduction

Nginx is a powerful and widely used web server known for it performance and flexibility. Deploying and configuring Nginx manually an multiple servers can be time-consuming, but with ansible, this process becomes automated and efficient. This project will teach me how to use Ansible to deploy and configure an Nginx server on a Linux machine.


# Objectives

1. Understand how Ansible simplifies the deployment and configuration of applications.
2. Set up an Ansible environment for managing Linux servers.
3. Create and execute an Ansible playbook to install Nginx.
4. Configure a basic Nginx website using Ansible.
5. Verify the Nginx deployment.


# Prerequisites

**Linux Servers**: At least one server to act as the target machine and an optional control machine for ansible.

**Ansible Installed**: Ansible is installed on the control machine. you can check this guide [out](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

**SSH Access**: SSH access between the control machine and target servers with public key authentication.

**Tools**: A text editor to create and edit Ansible playbooks.


# Project Tasks

## Task 1 -Install and Configure Ansible

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

![](./Images/1.%20Ping%20(1).png)


# Task 2 - Set Up the Ansible Inventory FIle

1. Create an inventory file to define the target server.

``` bash
cat inventory.ini
```

![](./Images/2.%20Inventory.png)


# Task 3 - Create an Ansible Playbook to Install Nginx

1. Create a playbook file for installing Nginx.

``` bash
nano install_nginx.yml
```

2. Add the following playbook content.

``` bash
# install_nginx.yml

- name: Install Nginx on the server
  hosts: linux_servers
  become: yes
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Ensure Nginx is running
      service:
        name: nginx
        state: started
        enabled: yes
```

3. save the file


# Task 4 - Configure a Custom Nginx Website Using Ansible

1. Create a playbook for Nginx website configuration.


``` bash
nano configure_nginx.yml
```

2. Add the following playbook content.

``` bash
# configure_nginx.yml

- name: Configure Nginx website
  hosts: linux_servers
  become: yes
  tasks:

    - name: Create website root directory
      file:
        path: /var/www/mywebsite
        state: directory
        mode: '0755'

    - name: Deploy HTML content
      copy:
        content: |
          <html>
            <head><title>Welcome to My Website</title></head>
            <body>
              <h1>Hello from Nginx!</h1>
            </body>
          </html>
        dest: /var/www/mywebsite/index.html

    - name: Configure Nginx server block
      copy:
        content: |
          server {
              listen 80;
              server_name _;
              root /var/www/mywebsite;
              index index.html;

              location / {
                  try_files $uri $uri/ =404;
              }
          }
        dest: /etc/nginx/conf.d/mywebsite.conf

    - name: Test Nginx configuration
      command: nginx -t
      register: nginx_test
      ignore_errors: true

    - name: Fail if Nginx config test fails
      fail:
        msg: "Nginx configuration test failed. Check your server block syntax."
      when: nginx_test.rc != 0

    - name: Reload Nginx
      service:
        name: nginx
        state: reloaded
```


# Task 5 - Verify the Nginx Deployment

1. Run the playbook to install and configure Nginx.

``` bash
ansible-playbook -i inventory.ini install_nginx.yml --ask-become-pass

ansible-playbook -i inventory.ini configure_nginx.yml --ask-become-pass
```

![](./Images/3.%20Nginx%20Install.png)


![](./Images/4.%20Nginx%20Configure.png)


2. Verify Nginx is running on the target server.

``` bash
curl http://<target-server-ip>
```

![](./Images/5.%20CURL.png)


3. Open the target server's IP address in a web browser to access the custom website.


``` bash
Open the target server's IP address in a web browser to access the custom website.
```

# Conclusion

In this project, i used Ansible to automate the deployment and configuration of the Nginx web server on a Linux machine. I created reusable playbook for installing Nginx and deploying a custom website.

With these skills, I can manage multiple web servers efficiently, customize configuration further, and scale my deployment processes.