Here’s a concise guide to **Ansible** covering its key topics with examples for each:
 
---
 
### **1. What is Ansible?**
- **Definition**: Ansible is an open-source IT automation tool used for configuration management, application deployment, and task automation.
- **Features**: Agentless, simple YAML-based syntax, idempotent, and supports SSH.
 
---
 
### **2. Architecture**
- **Components**:
  - **Control Node**: Where Ansible is installed and commands are executed.
  - **Managed Nodes**: Servers managed by Ansible.
  - **Inventory**: A file listing managed nodes.
  - **Modules**: Pre-written scripts for performing tasks.
  - **Playbooks**: YAML files defining automation workflows.
  - **Plugins**: Extend Ansible's functionality (e.g., callback, connection plugins).
 
---
 
### **3. Inventory**
- **Definition**: A file defining hosts and groups of hosts to manage.
- **Format**: INI or YAML.
 
**Example** (inventory file in INI format):
```ini
[web]
web1.example.com
web2.example.com
 
[db]
db1.example.com ansible_user=root ansible_ssh_private_key_file=/path/to/key
```
 
---
 
### **4. Ad-Hoc Commands**
- **Definition**: Quick, one-line Ansible commands for immediate tasks without a playbook.
  
**Example** (Check disk space on all servers):
```bash
ansible all -m shell -a "df -h"
```
 
---
 
### **5. Playbooks**
- **Definition**: YAML files to define tasks in an ordered manner.
- **Structure**: Playbooks consist of plays, which target specific hosts/groups.
 
**Example** (Simple playbook to install Apache):
```yaml
- name: Install Apache
  hosts: web
  tasks:
    - name: Install Apache package
      apt:
        name: apache2
        state: present
```
 
---
 
### **6. Modules**
- **Definition**: Reusable, idempotent scripts for tasks (e.g., file, yum, apt, user, etc.).
  
**Example** (Create a file using the `file` module):
```yaml
- name: Create a file
  hosts: all
  tasks:
    - name: Create /tmp/testfile
      file:
        path: /tmp/testfile
        state: touch
```
 
---
 
### **7. Variables**
- **Definition**: Used to store values for reuse across tasks.
- **Declaration**: Directly in playbooks or external files.
 
**Example**:
```yaml
- name: Using variables
  hosts: all
  vars:
    package_name: nginx
  tasks:
    - name: Install a package
      apt:
        name: "{{ package_name }}"
        state: present
```
 
---
 
### **8. Loops**
- **Definition**: Perform repeated tasks.
 
**Example** (Install multiple packages):
```yaml
- name: Install packages
  hosts: all
  tasks:
    - name: Install packages
      apt:
        name: "{{ item }}"
        state: present
      loop:
        - git
        - curl
        - vim
```
 
---
 
### **9. Handlers**
- **Definition**: Triggered by tasks to perform actions (e.g., restart services).
  
**Example**:
```yaml
- name: Install Apache and restart
  hosts: all
  tasks:
    - name: Install Apache
      apt:
        name: apache2
        state: present
      notify: Restart Apache
 
  handlers:
    - name: Restart Apache
      service:
        name: apache2
        state: restarted
```
 
---
 
### **10. Roles**
- **Definition**: A way to organize playbooks into reusable components (tasks, handlers, variables, templates, etc.).
 
**Example** (Creating a role structure):
```bash
ansible-galaxy init myrole
```
 
Directory structure:
```
myrole/
├── tasks/
├── handlers/
├── templates/
├── files/
├── vars/
├── defaults/
└── meta/
```
 
---
 
### **11. Templates**
- **Definition**: Use Jinja2 templates to dynamically configure files.
  
**Example** (`nginx.conf.j2`):
```jinja
server {
    listen 80;
    server_name {{ server_name }};
}
```
 
Playbook:
```yaml
- name: Deploy NGINX config
  hosts: web
  vars:
server_name: example.com
  tasks:
    - name: Deploy configuration file
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
```
 
---
 
### **12. Conditionals**
- **Definition**: Execute tasks based on conditions.
  
**Example**:
```yaml
- name: Conditional task
  hosts: all
  tasks:
    - name: Install package only on Ubuntu
      apt:
        name: nginx
        state: present
      when: ansible_facts['os_family'] == "Debian"
```
 
---
 
### **13. Tags**
- **Definition**: Run specific tasks in a playbook.
 
**Example**:
```yaml
- name: Tagged tasks
  hosts: all
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
      tags: install
 
    - name: Start nginx
      service:
        name: nginx
        state: started
      tags: start
```
Run specific tasks:
```bash
ansible-playbook playbook.yml --tags "install"
```
 
---
 
### **14. Ansible Vault**
- **Definition**: Encrypt sensitive data (passwords, API keys).
  
**Example** (Encrypt a file):
```bash
ansible-vault encrypt secrets.yml
```
Decrypt a file:
```bash
ansible-vault decrypt secrets.yml
```
 
---
 
This covers the essentials of Ansible. Let me know if you need further elaboration!
