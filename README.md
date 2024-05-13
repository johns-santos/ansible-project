#ansible-project

# Ansible Robo Playground in AWS
Requirements:
AWS account
two web servers (app1, app2)
One loadbalancer (lb1)

Objective:
Become familiar with ansible configuration and playbooks.



# Steps
# ===========================
1. Create AWS key pair and download to workstation

2. move keypair and change permissions
   ➜  ~ mv Downloads/ansible-robo-pair.pem ~/.ssh 
  ➜  ~ chmod 400 ~/.ssh/ansible-robo-pair.pem

3. Go to AWS Cloud Formation and create new stack 
      ---- upload template to S3 and navigate to yaml file and create.
      ---- yml contains 3  EC2 with elastic IP.

4.  Log onto ansible controller and install ansible.
      --- ➜  project pip install ansible

5. Create a ansible INVENTORY file (this is a static list, can be dynamic)
       ---- Default location of file  = /etc/ansible/hosts (can use "-i" to use another.
       ----  FORMAT is as followed:
              [webserver]
              hostname1:<can include non-standard ssh port>
              hostname2=<12.34.55.77> ALIAS
             [loadbalancers]
              lb.hostname3
             [local]
             control ansible_connection=local  (This specifies no SSH)

6. run ansible command to list host in INVENTORY
       ------>  ansible -i <file name> --list-hosts all

7. No longer use "-i" option, specify location inventory file in ansible CONFIG 
     ---NOTE -- Order of CONFIG file search
                       - ANSIBLE_CONFIG (env variable set)
                       - ansible.cfg  (in the current directory)
                       - ~/.ansible.cfg   (in the home dir)
                       - /etc/ansible/ansible.cfg  (used globally)
       -- STEP 1 -- CREATE A NEW FILE  ansible.cfg
                       -- FORMAT EXAMPLE
                          [defaults]
                          inventory = ./hosts-dev
         -- STEP 2 -- TEST new ansible.cfg is referenced
                          ansible_files ansible --list-host all
                          ansible --list-hosts "webservers[1]"    <--- use index location :)


# Run ansible TASKS   ---- basic block of Ansible execution (simple commands)
# ===========================
FORMAT:  ansisble options <host-pattern>
EXAMPLE: ansible -m ping alll

1.  Update  ansible.cfg add ssh users - add following parameters
      remote_user = ec2-user
      private_key_file = ansible-robo-pair.pem
      host_jey_checking = False
      interpreter_python=auto_silent ; # HIDES WARNINGS

     ---- RUN commands below  ---
             ansible -m ping "webserver[0]"
              ansible -m shell -a "uname" webservers:loadbalancers


# PLAYBOOK (YAML syntax)
# ===========================
Example of playbook (Task are grouped  and are in logical order)
----------------------------------------------------------------------------------------- 
-UPDATE
     • update all packages
      • patching needed
- INSTALL SOMETHING
       • install  service 1
        • install service 2
-CONFIGURE
       • setup services
       • update config files
       • restart services
-CHECK STATUS
        • Ensure up status

Example of playbook yml format:
----------------------------------------------------------------------------------------- 
ping.yml (This line should be commented)
---
- host: all
  tasks:
  - name: Pinging all servers
     action: ping




# CONSTRUCT SYSTEM USING PLAYBOOKS
    1. Package Management
    2. Configure Infrastructure
    3. Service Handlers
# ===========================
1) Package Mgmt (defines what pkgs are needed)
    - patching 
    - pkg manager

Example of yml (Reference docs for more info)
        -hosts: loadbalancers
         tasks: 
         -name: Install apache
         yum: name=httpd state=latest

2) Configure Infrastructure (configure system and app conf files for env)
    - copy files
    - edit configuration files

-hosts: loadbalancers
 tasks:
 -name: copy config file
  copy: src=./config.cfg dest=/etc/config.cfg

-hosts: webservers
 tasks:
 -name : Synchronise folders
  synchoronize: src./app dest=/var/www/html/app

3) Service handlers (create handlers to start, stop, resetart systems or services)
    - command
    - service
    - handlers



