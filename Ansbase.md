Ansible Playbooks
~~~~~~~~~~~~~~~~~
As ad-hoc commands are to bash commands, playbooks are to bash scripts. Playbooks run using 'ansible-playbook' command
and not the ansible command. Playbooks are written in YAML. It contain different elements called plays. Plays contain 
list of host and at minimum one task. 

You can use default host inventory or you can create a new inventory file.

>>  sudo su - ansible
>>  pwd  ==> /home/ansible

Create user defined inventory file
----------------------------------
>>  vim inv
localhost

[remote]                --> this defines a group of hosts in ansible called 'webservers'
vsathyak2 ansible_host=vsathyak5c.mylabserver.com
vsathyak3 ansible_host=vsathyak6c.mylabserver.com

[centos]             
localhost
vsathyak2 ansible_host=vsathyak5c.mylabserver.com
vsathyak3 ansible_host=vsathyak6c.mylabserver.com

[ubuntu]             
vsathyak4 ansible_host=vsathyak6c.mylabserver.com

>> Save it.

_______________________________________________________________________________________________________________________
Playbook Basics
~~~~~~~~~~~~~~~
Using YAML for Ansible Playbooks
--------------------------------
>>  vim sample.yml
    
user:                       ==> represents a list
- bob
- john                     
- ram

user:                       ==> represents a dictionary containing key value pairs.
  name: bob
  job: engineer            
  salary: 100000

include_newlines: |         ==> Multi line value using '|' and is interpreted as multi line.
  type some text           
  even more text
  the last of the text

fold_newlines: >            ==> Multi line value using '>' and is interpreted single line.
  type some text           
  even more text
  the last of the text  

"{{ var_name }}"            ==> Always wrap the variable names in double quote to prevent interpreting as dictonary.

become: yes                 ==> Boolean values in ansible are yes and no instead of true and false.

version: "1.0"              ==> Floating point numbers are taken as numbers by default.

_______________________________________________________________________________________________________________________
Creating an Ansible Play
~~~~~~~~~~~~~~~~~~~~~~~~
What is a Play in Ansible?
--------------------------
. A set of instruction expressed in YAML
. It targets a host of group of hosts
. It provide a series of tasks, which are executed in sequence

. Plays are kept in a file called Playbooks, which may contain one or more plays
. Each play may have a number of options configured such as remote_user, become and gather_facts

. Next section of play is called task section
  - This section contains a list of modules that will be executed against the target
  - Each task maps to the use of an Ansible module

Best Practices
--------------
. Comments are important
. Break up plays with white space


>>  vim play.yml

--- # play example
- hosts: vsathyak2                         -----|
  become: yes                                   |
  tasks:                                        |
    - name: install elinkss software            |    
      yum:                                      |
        name: elinks                            |---> Eg. of a 'Play' 
        state: latest                           |     The file where we store our Plays is called as a         
    - name: install nmap-ncat software          |     'Playbook' which contain more than one 'Plays'.
      yum:                                      |
        name: nmap-ncat                         |    
        state: latest                      -----|  

- hosts: vsathyak3
  become: yes
  tasks: 
    - name: install httpd software           
      yum:                                
        name: httpd                            
        state: latest 

This is a sample Playbook with multiple Plays in it.

_______________________________________________________________________________________________________________________
The `ansible-playbook` Command
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

. Playbooks must be executed using the command 'ansible-playbook'
. The ansible-playbook commands take few basic arguments.
  - The inventory file to use      
  - The playbook to execute
  
Notable Options
---------------
-K  ask for the become password      (This is used when you havent configured ansible users to have sudo privileges)
-k  ask for the connect password     (This is used when you havent configured the pre share key)
-C  run in check mode which is an effective dry run of the provided playbook.

>>  ansible-playbook -i inv play.yml
>>  ansible-playbook -i inv play.yml -C
    This command is doesnot execute the playbook instructions on the host. Instead it will do a check run
    Means it will let us know whether package mentioned in the playbook is already installed on the host or not.

_______________________________________________________________________________________________________________________
Understanding Playbook Tasks
~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.  Tasks are presented in list form begining with the name property. Each list elements start with a hyphen. '-'
.  The name property is simply a plain english statement describing what the task does.
.  The module that will be used to provide on the next line is followed by a colon ':'
.  If applicable, each argument to be provided to the module follows line by line in the following format
   argument: value
   
Best Practices
--------------
. Name you tasks
. The provided name is displayed on task execution which provide insights to those running the plays
. The name also serves as basic documentation within the playbook

_______________________________________________________________________________________________________________________
Task
~~~~~
Create a playbook called /home/ansible/bootstrap.yml to fulfill the following boot strap requirements:
All servers
-----------
. Edit /etc/hosts to include the following entry:
  - ansible.xyzcorp.com  169.168.0.1
   
. Install elinks
. Create the user xyzcorp_audit
. Copy the files /home/ansible/motd and /home/ansible/issue to /etc/

Network servers
---------------
. Install nmap-ncat
. Create the user xyzcorp_network

SysAdmin servers
----------------
. Copy /home/ansible/scripts.tgz from the control node to /mnt/storage

Solution
--------
vim bootstarp.yml

---
- hosts: all
  become: yes
  tasks:
    - name: Edit /etc/host file
      lineinfile:
        path: /etc/hosts
        line: "169.168.0.1 anzible.xyzcorp.com"   
    - name: Install elinks
      package:
        name: elinks
        state: latest
    - name: Create user xyzcorp_audit
      user:
        name: xyzcorp_audit
        state: present
    - name: Copy /home/ansible/motd to /etc/
      copy:
        src: /home/ansible/motd
        dest: /etc/motd
    - name: Copy /home/ansible/issue to /etc/
      copy:
        src: /home/ansible/issue
        dest: /etc/issue

- hosts: network
  become: yes
  tasks:
    - name: Install netcat
      yum:
        name: nmap-ncat 
        state: latest
    - name: Create user xyzcorp_network
      user:
        name: xyzcorp_network
        state: present

- hosts: sysadmin
  become: yes
  tasks:
    - name: Copy /home/ansible/scripts.tgz to /mnt/storage
      copy:
        src: /home/ansible/scripts.tgz
        dest: /mnt/storage

_______________________________________________________________________________________________________________________
Using Variables in Playbooks
~~~~~~~~~~~~~~~~~~~~~~~~~~~~
. Variable names should be letter, numbers and underscores
. Variables should always start with a letter
. Variables can be used to store
   -  a simple text or numeric value
   -  simple lists
   -  python style dictionaries
. A dictionarry is a list of key value pairs   
. Variables may be defined in a number of ways
   -  via command line argument
   -  within a variables file
   -  within a playbook
   -  within an inventory file
. It is a good practice to wrap variable names in weak quotes
. The register module is used to store task output in a dictionary variable


>>  vim vars.yml
working_dir: /home/ansible/working

service_list: 
- httpd
- nfs
- mariadb

share_paths:
  nfs: /mnt/nfs
  cifs: /mnt/cifsfs
  iscsi: /mnt/iscsi

>>  Save

>>  mkdir group_vars  and host_vars
>>  cp vars.yml group_vars/remote

This means we created a variable file for the group 'remote' and this file need to be in same place as we created our 
'inv' file. Then all the variables defined in remote.yml will be defined in all the hosts that somes under the host 
group 'remote' define in the 'inv' file.

Same is the case with host_vars, where files in the host_vars is for that particular hosts and so the variable defined
in the host_var dir file is avaiable to the particular host based on filename match.

In group_vars and host_vars, when we create file to define variables for hosts or groups, the variableas and its values 
are stored as key value pair like in a dictionary.

>>  vim vars_play.yml
---
- hosts: vsathyak2
  tasks:
    - name: Create working directory
      file:
        name: "{{ working_dir }}"
        state: directory
    - name: write service list
      lineinfile:
        path: "{{ working_dir }}/services.txt"
        create: yes
        line: "{{ service_list }}"
    - name: view share property
      command: ls -la "{{ share_paths['nfs'] }}"    >>  register module is used to store task output in a variable.
      register: cmd_output                          >>  here o/p of the task 'command' is stored in cmd_output
    - name: write command module output to file     
      copy:
        content: "{{ cmd_output }}"
        dest: "{{ working_dir }}/cmd_output.txt"
        
Running vars_play.yml and using vars.ymls for variable definitions
------------------------------------------------------------------
>>  ansible-playbook vars_play.yml -e @vars.yml
>>  ansible-playbook vars_play.yml -e @group_vars/remote
>>  ansible-playbook vars_play.yml -e @./group_vars/remote

>>  ls /ansible/home/working
     services.txt   cmd_output.txt

>>  cat /ansible/home/working/services.txt
    ['httpd', 'nfs', 'mariadb']

>>  cat /ansible/home/working/cmd_output.txt

_______________________________________________________________________________________________________________________
Working with Templates
~~~~~~~~~~~~~~~~~~~~~~
. Template module is capable of taking a file that is you have pre configured and deploying that file in a destination
   of your choosing.
. Templates are files with Ansible variables inside that are substituted on play execution
. Template uses the template module
. Notes about template file
   -  They are essentially text files that have variable references.
   -  They uses Jinja 2 templating
   -  They use the file extension .j2
   - A typical use case is a skeleton configuration file where variables may be used tfor simple customizations
     (such as IP addresses or host names)

>>  vim config.j2
name={{ code_name }}        ===> Eg. for ansible template
version={{ version }}

>>  vim template.yml
--- # template example
- hosts: vsathyak2
  gather_facts: no
  become: yes
  vars:
    code_name: whisky
    version: 4.2
  tasks:
    - name: Deploy config file
      template:
        src: config.j2
        dest: /opt/config

>>  ansible-playbook template.yml
>>  cat /opt/config
name=whisky
version=4.2

Note : There is 'validate' command to run before copying into place. 

_______________________________________________________________________________________________________________________
Using Ansible Facts
~~~~~~~~~~~~~~~~~~~
Ansible facts are simple various properties and information regarding a given remote system                                                            
- Setup module can retrieve facts
- Facts may be filtered by passing a value for the 'filter' paramater
Facts are gathered by default in ansible playbook execution
-  Setting gather_facts: no in playbook would change the default fact_gather behaviour

>>  ansible -i inv vsathyak2 -m setup -a 'filter=ansible_all_ipv4_addresses' 
>>  ansible -i inv vsathyak2 -m setup -a filter=*ipv4*

>>  vim facts.yml
---  # Ansible facts example
- hosts: vsathyak2
  tasks:
    - name: create a file
      lineinfile: 
        path: /home/ansible/hostname
        create: yes
        line: "{{ ansible_hostname }}"
    - name: access magic variables
      lineinfile:
        path: /home/ansible/hostname
        line: ""{{ hostvars['vsathyak2']['ansible_default_ipv4']['address'] }}""
        # Other magic variables are groups, group_name and inventory_hostname

>>  ansible-playbook -i inv facts.yml
>>  ssh cloud_user@vsathyak2c.mylabserver.com
>>  su - ansible
>>  cat /home/ansible/hostname
vsathyak2c
172.31.124.106

Handling fact_caching
---------------------
>>  vim /etc/ansible/ansible.cfg
>>  Find key fact_caching and fact_caching_connection and set the following

#fact_caching = memory              --->  fact_caching = json_file
#fact_caching_connection = /tmp     --->  fact_caching_connection = /tmp
:wq

>>  ansible -i inv all -m setup     --->  Running this will gather facts for all hosts and create a fact_caching and 
                                          store it in fact_caching location as a json file.

>>  ansible-playbook -i inv facts.yml
    Running this after fact caching, will set the info on all host based on the hosts/group you choose on the .yml file.
    
_______________________________________________________________________________________________________________________
Conditional Execution in Playbooks 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
. Ansible playbooks are capable of making actions conditional
. 'when' keyword is used to test the condition within a playbook
. Jinja 2 expressions are used for confitional evaluation

>>  vim conditional.yml
--- # ansible conditional example
- hosts: remote
  vars:
    target_file: /home/ansible/hostname
  tasks:
    - name: Gather file information
      stat:                                                     --> stat module used to retrieve file or file s/m status
        path: "{{ target_file }}"
      register: hostname                                        --> hostname going to hold value of path variable
    - name: Rename hostname when found
      command: mv "{{ target_file }}" /home/ansible/net-info    --> command module used since no mv module available
      when: hostname.stat.exists                                --> same as hostname['stat']['exists']

>>  ansible-playbook -i inv conditional.yml
>>  Running this script will check for the filename : hostname under /home/ansible and when found move or rename it
    to net-info
_______________________________________________________________________________________________________________________
Using Loops in Ansible
~~~~~~~~~~~~~~~~~~~~~~
. loop keyword is used to express a repeated action.
. loop is also operate with a list variable
. it is possible to combine loops and conditionals

>>  vim loop.yml
--- # loop example
- hosts: vsathyak2
  become: yes
  tasks:
    - name: Install a list of softwares
      yum:
        name: "{{ item }}"                   -> item is the variable that work with loops.
        state: present
      loop:
        - elinks
        - nmap-ncat
        - bind-utils

>>  ansible-playbook -i inv loop.yml


>>  vim loop2.yml
--- # loop example
- hosts: vsathyak2
  become: yes
  vars_files:
    - vars.yml
  tasks:
    - name: Check services
      service:
        name: "{{ item }}"                   -> item is the variable that work with loops.
        state: started
      loop: "{{ service_list }}"
        
>>  ansible-playbook -i inv loop2.yml

_______________________________________________________________________________________________________________________
Working with Handlers in Ansible
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
. Ansible provide a mechanism that allows an action to be flagged for execution when a tasks performs a change. 
. By only executing certain tasks during a change, plays are more efficient. This mechanism is known as Handler.
. A handler may be called using 'notify' keyword.
. Also even if the handler is flagged multiple times in a play, it only runs during a plays final phase.
. 'notify' will only flag a handler if a tasks make a change.

>>  vim handler.yml
--- # handler example
- hosts: vsathyak2
  become: yes
  vars:
    httpd_log_level: error
  tasks:
    - name: install httpd
      yum:
        name: httpd
        state: latest
    - name: update configuration
      template:
        src: /home/ansible/httpd/conf.j2
        dest: /etc/httpd/conf/httpd.conf
      notify: httpd service
  handlers:
    - name: restart httpd
      service: 
        name: httpd
        state: restarted
      listen: httpd service
     
>>  ansible-playbook -i inv handler.yml  
     
     
>>  vim web_handler.yml
--- # Bootstrap webservers
- hosts: remote
  become: yes
  gather_facts: yes
  tasks:
    - name: install latest httpd
      yum:
        name: "{{ target_service }}"
        state: latest
      notify:
        - restart httpd                  # A change in task yum will call the handler restart httpd
    - name: create index.html file
      file:
        name: /var/www/html/index.html
        state: touch
    - name: add web content
      lineinfile:
        line: "{{ ansible_hostname }}"
        path: /var/www/html/index.html
      notify:
        - restart httpd                 # A change in task yum will call the handler restart httpd
  handlers:
    - name: Attempt to restart httpd
      service:
        name: httpd
        state: restarted
      listen: restart httpd

>>  ansible-playbook -i inv web_handler.yml -e "target_service=httpd" 

_______________________________________________________________________________________________________________________
Task
~~~~~
You have been tasked with creating the build for a common NFS server. Write a playbook that satisfies the following 
requirements:

On nfs:
-------
. Make sure nfs-utils is installed.
. Configure /etc/exports via an Ansible template stored in /home/ansible/exports.j2. 
  Deploy the template so that /mnt/nfsroot is exported with read and write to all hosts.
  . Note: Your template file should contain the following with variable {{ share_path }} being defined within your playbook:
    {{ share_path }} *(rw)
  . Note: The file /etc/exports on nfs should have the following content once deployed:
    /mnt/nfsroot *(rw)
. Create a handler that runs the command exportfs -a if the file /etc/exports is modified in a playbook task.
. You can assume all necessary firewall rules have been deployed.

On remote:
----------
. Configure /etc/hosts from a template file stored on control at /home/ansible/etc.hosts.j2 with the following entries:
  127.0.0.1 localhost, {{ ansible_hostname }}
  {{ nfs_ip }}  {{ nfs_hostname }}
   . Note: You should populate the variables {{ nfs_ip }} and {{ nfs_hostname }} using magic variables in your playbook.
. Create users from file stored on control at /home/ansible/user-list.txt only if the remote host has the file 
   /opt/user-agreement.txt.
   
The Ansible control node has been configured for you and all other servers have already been configured for use with 
Ansible. The default inventory has been configured to include the group remote and server nfs.

Also add user-list.txt with the following values on control node under /home/ansible.

users:
- frank
- judy
- joe
- sarah
- sam
- carry


TASK1 : Create the necessary template files on the Ansible control node.
------------------------------------------------------------------------
>>  vim exports.j2       --> /home/ansible/exports.j2
{{ share_path }} *(rw)

>>  vim etc.hosts.j2       --> /home/ansible/etc.hosts.j2
127.0.0.1    localhost {{ ansible_hostname}}
{{ nfs_ip }}    {{ nfs_hostname }}

TASK2 : Create a playbook for the server 'nfs' in the Ansible inventory
TASK3 : Add a play for the remote host group
------------------------------------------------------------------------
>>  vim nfs.yml
---
- hosts: nfs
  become: yes
  vars:
    share_path: /mnt/nfsroot
  tasks:
  - name: Install nfs-util
    yum:
      name: nfs-utils
      state: latest
  - name: Start and enable nfs service
    service: 
      name: nfs-server
      enabled: yes
      state: started
  - name: Configure /etc/exports
    template:
      src: /home/ansible/exports.j2       # variable 'share_path' is used over here
      dest: /etc/exports
    notify: updated /etc/exports
  handlers:
  - name: update nfs exports
    command: exportfs -a
    listen: updated /etc/exports

- hosts: remote
  become: yes
  vars:
    nfs_ip: "{{ hostvars['nfs']['ansible_default_ipv4']['address'] }}"
    nfs_hostname: "{{ hostvars['nfs']['ansible_hostname'] }}" 
  vars_files:
    - /home/ansible/user-list.txt
  tasks:
  - name: Configure /etc/hosts
    template:
      src: /home/ansible/etc.hosts.j2                   # variables 'nfs_ip and nfs_hostname' is used over here
      dest: /etc/hosts
  - name: Check /opt/user-agreement.txt file status
    stat:
      path: /opt/user-agreement.txt
    register: filestat
  - name: Print file status
    debug:
      msg: File status "{{ filestat }}"
  - name: Create users using /home/ansible/user-list.txt
    user:
      name: "{{ item }}"
    when: filestat.stat.exists
    loop: "{{ users }}"

TASK4: Execute playbook to verify your playbook works correctly
---------------------------------------------------------------
>>  ansible-playbook /home/ansible/nfs.yml
>>  ssh nfs
>>  cat /etc/exports   -->   /mnt/nfsroot *(rw)
>>  systemctl status nfs-server
>>  logout

>>  ssh node2
>>  cat /etc/hosts
127.0.0.1      localhost   node2
10.0.1.48      nfs
>>  id judy
uid=1005(judy)   gid=1005(judy)   groups=1005(judy)

_______________________________________________________________________________________________________________________
Advanced Playbook Syntax
~~~~~~~~~~~~~~~~~~~~~~~~
Executing Selective Parts of a Playbook
---------------------------------------
. Running selective part of a playbook is through tags.
. Ansible allow both play and tasks to be tagged
. We can use tags to skip execution of certain parts
. We can specify which tags to run or not run via arguments to the ansible-playbook command

>>  vim tags.yml
--- # Tags example
- hosts: vsathyak2
  tasks: 
    - name: Install elinks
      become: yes                      --> sudo privilege can be provided on tasks level also
      yum: 
        name: elinks
        state: latest
      tags:
        - software
    - name: Add line to text file
      lineinfile: 
        path: /home/ansible/tag-test.txt
        create: yes
        line: "Tag called"
      tags:
        - files
        - config
    - name: Copy tag file
      copy:
        src: /home/ansible/tag-test.txt
        dest: /tmp/copied.txt
        remote_src: yes
      tags:
        - config

>>  ansible-playbook -i inv tags.yml -t software              --> Run tasks with tag : software
>>  ansible-playbook -i inv tags.yml --tags software          --> Run tasks with tag : software
>>  ansible-playbook -i inv tags.yml --tags software,files   --> Run tasks with tag : software and files    
>>  ansible-playbook -i inv tags.yml --skip-tags software     --> Skip tasks with tag : software

_______________________________________________________________________________________________________________________
Working with Sensitive Data Using Ansible Vault
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
. The ansible-vault command is used to encrypt files and work with those files
. It can take a number of sub commands
. A file encrypted with ansible-vault is called a vault
. The primary use case for vault is for encrypting variable files to protect sensitive information such as passwords
. It is also possible to encrypt task files or even arbitrary files such as binaries if desired
. The ansible-vault password file is simply a file that contain password
. The recommended way to provide a vault password from the CLI is to use --vault-id
. --vault-id may be passed as a vault file or a prompt flag(--vault-id@prompt) to collect credentials to unencrypt a
  target vault

Create a vault file
-------------------
>>  ansible-vault create vaultfile
New Vault password: test@123
Confirm New Vault password: test@123

name: Logan
:wq

>>  cat vaultfile 
$ANSIBLE_VAULT;1.1;AES256
35326337663035616664396365616664663937656136336131346361323636633961323961303831
3034633039313638383263306131633437396465393330360a396134343539643634306332333630
64656163323433616337376634323633376536363134323665333038333439396438313437343563
3637393435366337390a623862363462383865653461366530316264313165336231656262656264
30323032643564393665356434373136313964633266346165633666303838393530

View ansible-vault
---------------------------------
>> ansible-vault view vaultfile
Vault password: test@123
name: Logan

Edit ansible-vault
---------------------------------
>> ansible-vault edit vaultfile
Vault password: test@123

name: Vitaly   (change from Logan)

>> ansible-vault view vaultfile
Vault password: test@123
name: Vitaly    (Now you can see the updated value)

Using encrypt_string command
-----------------------------
>>  ansible-vault encrypt_string  --vault-id prod@prompt 'new_var: new_value'
New vault password (prod): 
Confirm new vault password (prod): 

!vault |
          $ANSIBLE_VAULT;1.2;AES256;dev
          61323931353866666336306139373937316366366138656131323863373866376666353364373761
          3539633234313836346435323766306164626134376564330a373530313635343535343133316133
          36643666306434616266376434363239346433643238336464643566386135356334303736353136
          6565633133366366360a326566323363363936613664616364623437336130623133343530333739
          3039
Copy and stick this in vault section of the playbook   


Decrypt vault file
------------------
>>  ansible-vault decrypt vaultfile
Vault password: test@123

>>  cat vaultfile
name: Vitaly

Encrypt vault file
------------------
>>  ansible-vault encrypt vaultfile
New Vault password: test@123             --> You could use even a new password.
Confirm New Vault password: test@123

Encrypt vault Using vault-id 
----------------------------
The vault file should be created using the syntax 'ansible-vault create vaultfile' , before using the below syntaxes.

Syntax1 : ansible-vault encrypt --vault-id label@prompt vaultfile         -> this will ask you to enter password
Syntax2 : ansible-vault encrypt --vault-id label@password_file vaultfile  -> this will get pwd from password_file

Eg for Syntax1
--------------
>>  ansible-vault encrypt --vault-id prod@prompt vaultfile
New Vault password: test@123             --> You could use even a new password.
Confirm New Vault password: test@123

                        OR
Eg for Syntax2
--------------                        
>>  vim password_file
password: test@123

>>  ansible-vault encrypt --vault-id prod@password_file vaultfile
>>  ansible-vault decrypt --vault-id prod@password_file vaultfile
.   vaultfile is encrypted and decrypted using password in the password_file

Using a vault file in an ansible-playbook
-----------------------------------------
>>vim secret_playbook.yml
--- # Vault example
- hosts: vsathyak2
  vars_files: /home/ansible/vaultfile
  tasks:
  - name: password to open.txt
    lineinfile:
      path: /home/ansible/open.txt
      create: yes
      line: "{{ name }}"
    no_log: true                           ->  to avoid exposing protected information 

Executing the playbook when you encrypted the vaultfile using Syntax 1 ie using prompt
-------------------------------------------------------------------------------------
>>  ansible-playbook -i inv secret_playbook.yml  --vault-id prod@prompt
    This will prompt to enter password  to decrypt vault1 since 'vaultfile is refered as the var_file in playbook, 
    then use the password value in the 'vaultfile' file and write it into the open.txt

Executing the playbook when you encrypted the vaultfile using Syntax 2 ie using a password file
-----------------------------------------------------------------------------------------------
>>  ansible-playbook -i inv secret_playbook.yml --vault-id prod@password_file

Executing the playbook when you encrypted the vaultfile using the syntax 'ansible-vault encrypt vaultfile'
----------------------------------------------------------------------------------------------------------
>>  ansible-playbook -i inv secret_playbook.yml --vault-id @prompt
>>  ansible-playbook -i inv secret_playbook.yml --vault-id @password_file

_______________________________________________________________________________________________________________________
Error Handling in a Playbook - limit, ignore_errors, changed_when, and failed_when
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
. When playbook fail to execute on any host, a file is created containing names of each host where the playbook failed.
. This file may be used with --limit flag to execute only against hosts where the playbook failed.
. Ansible may be configured to continue execution even after an error occurs using the 'ignore_errors' keyword.
. Ansible allows failure conditions to be defined using the 'failed_when' keyword
. There is also 'changed_when' that allows an override of what Ansible considers changed
  
>> vim error1.yml
---
- hosts: vsathyak2
  become: yes
  tasks:
    - name: Install Software
      yum: 
        name: broke
        state: latest
      ignore_errors: yes                          -> Setting this flag will continue execution even when there is error
    - name: Run utility
      command: /home/ansible/do-stuff.sh nothahaha    -> nothahaha is a param passed to do-stuff.sh file
      register: cmd_output
      changed_when: "'CHANGED' in cmd_output.stdout"
      failed_when: "'FAIL' in cmd_output.stdout"
      
      
>>  vim do-stuff.sh
#!/bin/bash

if [ $1 == "hahaha" ]; then             
  echo "I CHANGED SOMETHING"
  exit
else
  echo "I FAILED"
fi        
      
>>  ansible-playbook -i inv error1.yml
>>  In error1.yml, change the parameter from nothahaha to hahaha and run it again to pass the execution

To retry using limit flag
-------------------------
>>  ansible-playbook -i inv error1.yml --limit @error1.retry   (error1.retry is autogenerated)

_______________________________________________________________________________________________________________________
Error Handling in a Playbook - Block Groups and The Debug Module
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
. The debug module may be used to help troubleshoot plays
  - Use to print detail information about in-progress plays
  - Handy for troubleshooting
. Error handling may also be dealt with using block groups in Ansible. Tere are 3 key blocks that may be used to organize tasks
  - block  : Group tasks into a block 
  - rescue : A special block that is executed when the preceding blocks fail
  - always : A special block that is always executed after the preceding block
  
>>  vim error2.yml
---
- hosts: vsathyak2
  become: yes
  vars:
    target_service: httpd
  tasks:
    - name: Install httpd
      block:
        - service:
            name: "{{ target_service }}"
            state: started
            register: service_status
      rescue:
        - debug:
            var: service_status
      always:
        - debug:
            msg: "Tried to ensure service was running"
            
>>  ansible-playbook error2.yml -i inv error2.yml            

_______________________________________________________________________________________________________________________
Asynchronous Tasks within a Playbook
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
. Some operation requires significant amount of time to execute
. By default, all playbook block tasks run against a single host use a single SSH session
. Ansible provide the async feature to allow an operation to run asynchronously such that the status may be checked
. This can prevent interuption from SSH timeouts for long running operations
. You may a configure  a few key values for an asynchronous task
  - A timeout for an operation(default is unlimited)
  - A poll value for how often ansible should checkback. A pool value of 0 will have ansible not chekback on a task.
  
>> vim async.yml
--- # Async Task Example
- hosts: vsathyak2
  tasks:
    - name: Run sleep.sh
      command: /home/ansible/sleep.sh
      async: 60
      poll: 10
      
>>  vim sleep.sh
#!/bin/sh

sleep 10
echo "I'm done~"
exit      

>>  ansible-playbook -i inv async.yml

_______________________________________________________________________________________________________________________
Delegating Playbook Execution with `delegate_to` and `local_action`
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
. In ansible, delegation is running a task on a specific host
  - By delegating a task, the task will only run on the host or the group to which it was delegated
  - In order to delegate, use delegate_to keyword

>>  vim delegate.yml
--- # Delegation Example
- hosts: remote
tasks:
  - name: Run sleep.sh                    ---> this particular task delegated to run only on localhost
    command: /home/ansible/sleep.sh            while other tasks run on all the servers that comes under remote.
    async: 60
    poll: 10
    delegate_to: localhost
  - name: Install mariadb
    package:
      name: mariadb
      state: latest
    become: yes
    
>> ansible-playbook -i inv delegate.yml

_______________________________________________________________________________________________________________________
Parallelism in Playbooks
~~~~~~~~~~~~~~~~~~~~~~~~
. The 'serial' keyword is used to control forks in a playbook
  - You must provide an integer count or a percentage of the total hosts
  - You may provide a 'step-up' approach(can mix and match the count with the percentage)
  - It is possible to use 'max_fail_percentage' to alow a certain percentage to fail

>>  vim serial.yml
--- # Serial execution example
- hosts: all
  serial : 1                       --> This will result in installing the elinks only one host at a time.
  become: yes
  tasks: 
    - name: Install elinks
      package:
        name: elinks
        state: latest
        
>> ansible-playbook -i inv serial.yml        


step-up approach and max_fail_percentage
----------------------------------------
>>  vim serial.yml
--- # Serial execution example
- hosts: all    
  max_fail_percentage: 10    -> This means if 10% of total hosts faile, execution will be stiooed
  serial :                   -> This will result in installing the elinks in a step up approach. first on single host, 
  - 1                           when its done, it will be installed on three hosts and then five hosts.
  - 3
  - 5
  become: yes  
  tasks: 
    - name: Install elinks
      package:
        name: elinks
        state: latest
        
Also instead of integers we can pass percentage of hosts to be executed on each batch like below
serial :                                serial :          
- "10%"                                 - 5  
- "30%"               OR                - "30%"  
- "50%"                                 - "50%"

_______________________________________________________________________________________________________________________
Using `run_once`
~~~~~~~~~~~~~~~~
. There are scenarios where a specific tasks needs to run only a single time in a given playbook and not on each host
  - This may be achieved using run_once keyword
  - This may be leveraged with delegate_to fro greater control over which host execute the command
  - It should be noted that when used with serial that run_once will execute for each serial batch
  
>>  vim runonce.yml
--- # run_once Example
- hosts: centos
tasks:
  - name: Run sleep.sh                      --> This task will run only once for the entire group of hosts. 
    command: /home/ansible/sleep.sh             So if u have a group of hosts and u havent specified about where
    async: 60                                   to run the tasks, it willrun only once in one of the hosts.
    poll: 0
    run_once: yes                  
  - name: Install mariadb
    package:
      name: mariadb
      state: latest
    become: yes
    
>> ansible-playbook -i inv runonce.yml  
  
_______________________________________________________________________________________________________________________
Overview of Ansible Roles
~~~~~~~~~~~~~~~~~~~~~~~~~
. Roles provide a way of automatically loading certain var_files, tasks and handlers based on a known file structure
. Roles also make sharing of configuration templates easier   
. Roles require a particular directory structure
  - Atleast one of the directories must exist in the role definition. However, unused directories are not required
  - Most file uses a file main.yml as entry point
. The role keyword is used to invoke a role
. When a role is called in a playbook, Ansible looks for the role definition in ${PWD}/roles/role_name
  - If ${PWD}/roles doesnot contain the sought role, then /etc/ansible/roles is checked
  - The default role location may be changed in: ansible.cfg
  - The full path to a role may also be specified with the role keyword to use a non-default path
. A role with a given set of parameter will only be applied once, even if called multiple times in a play  
  - If a role is called with different parameters, it will be rerun
  - A role may have allow_duplicates: true defined in meta/main.yml with in the role.
  
>>  man ansible-galaxy
  
Search roles using ansible-galaxy
---------------------------------
>>  ansible-galaxy search apache
  
Install roles using ansible-galaxy
-----------------------------------
>>  ansible-galaxy install a1max1.spadeDocker    ( spadeDocker : role installed  , a1max1 : owner )
    - get installed in /home/ansible/.ansible/roles/a1max1.spadeDocker
>>  cd .ansible/roles/a1max1.spadeDocker
>>  ls -la   will show the directory structure

_______________________________________________________________________________________________________________________
Ansible Role Demo
~~~~~~~~~~~~~~~~~ 
Creating a role shell
---------------------
>>  ansible-galaxy init /home/ansible/common
>>  cd common
>>  ls -la

drwxrwxr-x.  2 ansible ansible   21 Sep 23 23:07 defaults
drwxrwxr-x.  2 ansible ansible    6 Sep 23 23:07 files
drwxrwxr-x.  2 ansible ansible   21 Sep 23 23:07 handlers
drwxrwxr-x.  2 ansible ansible   21 Sep 23 23:07 meta
-rw-rw-r--.  1 ansible ansible 1328 Sep 23 23:07 README.md
drwxrwxr-x.  2 ansible ansible   21 Sep 23 23:07 tasks
drwxrwxr-x.  2 ansible ansible    6 Sep 23 23:07 templates
drwxrwxr-x.  2 ansible ansible   37 Sep 23 23:07 tests
drwxrwxr-x.  2 ansible ansible   21 Sep 23 23:07 vars

<<Not Completed>>
_______________________________________________________________________________________________________________________
