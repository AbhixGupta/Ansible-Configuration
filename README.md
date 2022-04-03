# Ansible-Configuration
This repository is good to get hands on Ansible server using AWS ec2. Learn and deploy the webserver using Modules, playbooks or specifying roles.


# About Ansible

* Configuration management tool.
* All work are done through YAML scripting
* Work on the PUSH Mechanism.
* It is maintained by the Ansible
* Ansible Tower for GUI Interface.


## Basic Installation

Create 3 AWS instances. One for ansible server and two for nodes. Run the following commands in the ansibel server node.

```bash
  sudo su 
  yum update -y
  wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm -y
  yum install epel-release-latest-7.noarch
  yum update -y
  yum install git python python-level python-pip opensssl ansible -y
  ansible --version
```

Create one group in which consist of Private Ip's remaining nodes. To do this edit the host file to create one group.
```bash
  vi /etc/ansible/hosts
```
Under the example1 write:
```javascript
[demo]
Private Ip1
Private Ip2

esc :wq -->enter
```
In this way you can create multiple groups.
Now make some changes in ansible.cfg group.
```bash
  vi /etc/ansible/ansible.cfg
```
Uncomment those sudo_user and inventory for starting the ansible services. Add one ansible user in the ansible server.
```bash
  adduser ansible
  passwd ansible 
```
Similarly add ansible user in remaining two nodes.
Now add ansible user into the sudoers file so that it can have have sudo priveleges.
```bash
  visudo
```
Find the "Allow Root to run any commands anywhere" and write write same for ansible user like for root user.
```bash
  ansible ALL=(ALL) NOPASSWD: ALL
```
Do the same for all remaining Nodes. After that login through ansible user in all the nodes. Before that make approprite conguration in ssh of ansible server node.
```bash
  # vi /etc/ssh/sshd_config
  # service sshd restart
```
uncomment "PermitRootLogin" and  "PasswordAuthentication yes", Comment "PasswordAuthentication no". Do this for all nodes.

Everytime we access the node machine thorugh we have give password, so tackle this we will generate trust-relationship, ssh-keygen. We will do this ssh work thorugh ansible user.

```bash
  ssh-keygen
  cd .ssh/
  ssh-copy-id ansible@PrivateIpOfNode
```

# Ad-hoc Commands and Modules
### Ad-hoc commands
These commands are also known as temperary commands. These commands are simple Linux commands. These commands uses /usr/bin/ansible command line tool to run indiviually to perform single tasks.
There is no idempotency in ad-hoc commands that why they are not used in Configuration, Management and Deployment and that is the reason ansible palybook came into picture.
```bash
  su -ansible
  ansibel demo -a "ls"
  ansible demo[0] -a "touch file1"
  ansible all -a "touch file"
  ansible demo -a "ls -al"
  ansible demo -a "sudo yum install httpd -y"
  ansible demo -ba "yum install httpd -y"
  ansible demo -ba "yum remove httpd -y"
```

### Modules
Ansible Modules are directly executed on the nodes. The major advantage of Modules over ad-hoc command is that it follow the rule of idempotency. The default location for inventory fiel is /etc/ansible/hosts.
```bash
  ansible demo -b -m yum -a "pkg=httpd state=present"
  ansible demo -b -m yum -a "pkg=httpd state=latest"
  ansible demo -b -m -a "pkg=httpd state=absent"
  ansible demo -b -m service -a "pkg=httpd state=started"
  ansible demo -b -m user -a "name=raj"
  ansible demo -b -m copy -a "src=file1 dest=/tmp"
```
Ansible "setup" is responsible to deal with the concurrency in the nodes and the server.
```bash
  ansible demo -m setup
  ansible demo -m setup -a "filter=*pv4*"
```
# Ansible Playbooks, Variables, Handlers and Loops
### Playbook
More than one module is known as playbook. It is written in YAML (Yet Another MarkUp Language) format, which is human readable data serielization langauge being commonly used for configuration of ansible files. Playbook consist of Vars (Variable), Handler, Files templates and roles. Playbook is divided into three sections:

* Target Section: It contains the information regarding the tasks which has to be executed in the host node.
* Variable Section: This section defines variable.
* Task Section: This contains list of all modules that are mentioned by the user which is exexuted in the serial order.
In YAML, all codes section starts with "---", and each item in the list is the list of key-value pairs commonly known as the Dictionary. Ex:
```bash
  --- # A list of fruits
  - mango
    guava
    grapes
    apple
```
A Dictionary is represented in the form of the key-value pairs. Ex:
```bash
  --- # Details of Customers
  - customer:
        - name: Rajput
          job: engineering
          skills: HTML
          Exp: 8
```
The extension of YAML file is ".yml".

Sample Playbooks:
```bash
  vi target.yml
```
```bash
  --- # Target palybook
  - hosts: demo
    user: ansible
    become: yes
    connection: ssh
    gather_facts: yes
```
To execute the playbook.
```bash
  ansible-playbook target.yml
```
Sample 2.
```bash
  vi tasks.yml
```
```bash
  --- # Target palybook
  - hosts: demo
    user: ansible
    become: yes
    connection: ssh
    tasks:
            - name: Install httpd on Linux machine
              action: yum name=httpd state=installed
```
To execute the playbook
```bash
  ansible-playbook tasks.yml
```
### Variables (Vars)
Variables are used to build more flexiblity while writing the codes. It can be very hellpful while writing loops in the ansible playbooks. Ex:
```bash
  vi vars.yml
```
```bash
   --- # Target palybook
   - hosts: demo
     user: ansible
     become: yes
     connection: ssh
     vars:
              pkgname: httpd
     tasks:
            - name: Install httpd on Linux machine
              action: yum name='{{pkgname}}' state=installed 
```

To execute the playbook
```bash
  ansible-playbook vars.yml
```
### Handlers 
Handlers are similar to the tasks, just they are executed if the task contains a notify directive and also indicate that it changed something.
```bash
  vi handlers.yml
```
```bash
   --- # My handlers palybook
   - hosts: demo
     user: ansible
     become: yes
     connection: ssh
     tasks:
            - name: Install httpd on Linux machine
              action: yum name=httpd state=installed 
              notify: restart httpd
     handlers: 
            - name: restart httpd
              action: service name=httpd state=installed
```
To DRY-RUN the playbook:
```bash
  ansible-playbook handlers.yml --check
```

To execute the playbook
```bash
  ansible-playbook hadlers.yml
```
### Loops
Loops are generally used to repeatedly do some task. Ex:
```bash
  vi loops.yml
```
```bash
   --- # My loops palybook
   - hosts: demo
     user: ansible
     become: yes
     connection: ssh
     tasks:
            - name: Install httpd on Linux machine
              user: name='{{item}}}' state=present
              with_items:
                    - Bhupinder
                    - Rajput
                    - Rohit
                    - XYZ
```

To execute the playbook
```bash
  ansible-playbook loops.yml
```


# Condition, Vaults and Roles in Ansible

### Condition
Suppose you are dealing with different types operating system in the Ansible environment (like Debian, RedHat, Fedora etc.) and you want to install the any specific service but the command for various os id different. At this point we mention the condition using "when".
```bash
  vi condition.yml

  --- # Condition Playbook
  - hosts: demo
    user: ansible
    become: yes
    connection: ssh
    tasks:
            - name: install apache on Debian
              command: apt-get -y install apache2
              when: ansible_os_family == "Debian"
              name: install apache for RedHat
              command: yum -y install httpd
              when: ansible_os_family == "RedHat"
```
### Vaults
Vaults are used to encrypt the Playbook so that no other user can access it without your permission. Vaults add an aditional security layer by adding password to your Playbooks. So everytime you want to access your playbook you have give password.

* Creating a new encrypted playbook
```bash
  ansible-vault edit xyz.yml
```
* To change the existing encrypted playbook password
```bash
  ansible-vault rekey xyz.yml
```
* To encrypt an existing playbook
```bash
  ansible-vault encrypt target.yml
```
* To decrypt an encrypted playbook
```bash
  ansible-vault decrypt target.yml
```

  
  
### Roles
There are two ways to in which we run ansibel tasks. One is "include" and other is "role". When we working with big projects it becomes very difficult to maintain the all the functionality in management in single file. There we can organize the playbook in the directory structure called Roles. In this directory there each tasks are handled by specific files.
Following are the types of roles we deal in ansible.

* Deafults Role
* Files Role
* Handlers Role
* Meta Role
* Tasks Role
* Vars Role
 These Roles helps us to distribute the complexity of the Single file.

 ```bash
   sudo yum install tree -y
   mkdir -p playbook/roles/webserver/tasks
   tree
   touch roles/webserver/tasks/main.yml
   tree
   touch master.yml
   vi roles/webserver/tasks/main.yml
```
Write the follwing example commands:
```bash
  - name: install apache
    yum: pkg=httpd state=latest
```
Open the master.yml
```bash
  vi master.yml
```
Write the following commands:
```bash
  - hosts: demo
    user: ansible
    become: yes
    connection: ssh
    roles:
        - webserver
```
Now run the playbook by:
```bash
  ansible-playbook master.yml
```    
