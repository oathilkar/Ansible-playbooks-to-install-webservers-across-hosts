# Guide-for-Ansible-playbooks-to-install-webservers-across-hosts

Installing web servers across multiple hosts using Ansible involves creating playbooks that define the tasks and configurations required for each web server type. 
Here’s a guide on how to create Ansible playbooks to install Nginx and Apache HTTP Server on multiple hosts.

### Prerequisites

1. **Ansible Installation**: Ensure Ansible is installed on your control node. You can follow the [Ansible Installation Guide](https://docs.ansible.com/ansible/latest/installation_guide/index.html).

2. **Inventory File**: Create an inventory file (`inventory.ini`) that lists the IP addresses or hostnames of your target servers.

   ```ini
   [webservers]
   server1 ansible_host=192.168.1.10
   server2 ansible_host=192.168.1.11
   ```

### 1. Installing Nginx

#### Ansible Playbook for Nginx (`nginx-install.yml`)

Create a playbook (`nginx-install.yml`) to install and configure Nginx across the hosts:

```yaml
---
- name: Install and configure Nginx
  hosts: webservers
  become: true  # Run tasks with root privileges

  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present  # Ensure Nginx is installed
      tags:
        - nginx

    - name: Copy Nginx configuration file
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: restart nginx
      tags:
        - nginx

  handlers:
    - name: restart nginx
      service:
        name: nginx
        state: restarted
```

#### Nginx Configuration Template (`templates/nginx.conf.j2`)

Create a Jinja2 template for the Nginx configuration (`templates/nginx.conf.j2`):

```nginx
# Nginx configuration file
user www-data;
worker_processes auto;
pid /run/nginx.pid;

events {
    worker_connections 768;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;

    keepalive_timeout 65;
    types_hash_max_size 2048;

    include /etc/nginx/conf.d/*.conf;

    server {
        listen 80;
        server_name {{ ansible_hostname }};

        location / {
            root /usr/share/nginx/html;
            index index.html index.htm;
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
            root /usr/share/nginx/html;
        }
    }
}
```

### 2. Installing Apache HTTP Server

#### Ansible Playbook for Apache (`apache-install.yml`)

Create a playbook (`apache-install.yml`) to install and configure Apache HTTP Server across the hosts:

```yaml
---
- name: Install and configure Apache HTTP Server
  hosts: webservers
  become: true  # Run tasks with root privileges

  tasks:
    - name: Install Apache
      apt:
        name: apache2
        state: present  # Ensure Apache is installed
      tags:
        - apache

    - name: Copy Apache configuration file
      template:
        src: templates/httpd.conf.j2
        dest: /etc/apache2/apache2.conf
      notify: restart apache
      tags:
        - apache

  handlers:
    - name: restart apache
      service:
        name: apache2
        state: restarted
```

#### Apache Configuration Template (`templates/httpd.conf.j2`)

Create a Jinja2 template for the Apache configuration (`templates/httpd.conf.j2`):

```apacheconf
# Apache configuration file
ServerName {{ ansible_hostname }}
```

### Running the Playbooks

To run the playbooks, execute the following commands from the directory where your playbooks (`nginx-install.yml`, `apache-install.yml`) and `inventory.ini` file are located:

```bash
# Run Nginx playbook
ansible-playbook -i inventory.ini nginx-install.yml

# Run Apache playbook
ansible-playbook -i inventory.ini apache-install.yml
```

### Summary

- **Nginx Installation**:
  - Use the `apt` module to install Nginx.
  - Use the `template` module to copy a Jinja2 template for the Nginx configuration.
  - Use a handler to restart Nginx after configuration changes.

- **Apache Installation**:
  - Use the `apt` module to install Apache HTTP Server.
  - Use the `template` module to copy a Jinja2 template for the Apache configuration.
  - Use a handler to restart Apache after configuration changes.

Adjust the paths and configurations in the playbooks according to your setup and requirements. This setup assumes Ubuntu/Debian hosts. For other Linux distributions, adjust the package manager and service management tasks accordingly (e.g., `yum` for CentOS/RHEL, `dnf` for Fedora).

### Additional Tips

- **Variables**: Use Ansible variables to make your playbooks more flexible and reusable.
- **Error Handling**: Implement error handling and retries for critical tasks.
- **Security**: Ensure you follow security best practices, especially when managing web servers.

This guide provides a solid foundation for automating the installation and configuration of web servers across multiple hosts using Ansible.
