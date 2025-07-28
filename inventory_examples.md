## INVENTORY FILE

### TYPES OF INVENTORY
1. CUSTOM
2. DEFAULT

- Default inventory file /etc/ansible/hosts(created when ansible install)
- Custom inventory file specify using -i option in using CLI or other way is to specify in ansible.cfg file
- It is good practice to create a inventory based on:-
    - Project
    - What(type of server) (for example, database servers, web servers, and so on).
    - Where(location of server) (for example, east, west).
    - When(env of server) (for example, prod, test).

#### TWO FORMAT TO WIRTE INVENTORY FILE
1. INI FORMAT
```

```

2. YAML FORMAT
```
all:
  children:
    webservers:
      hosts:
        server1.example.com:
        server2.example.com:
      vars:
        http_port: 80
        max_clients: 200

    dbservers:
      hosts:
        db1.example.com:
        db2.example.com:
      vars:
        db_port: 5432
        db_name: myapp
```
- Managing a Multi-Tier Application(web servers, database servers, and caching servers)
```
all:
  children:
    webservers:
      hosts:
        web1.example.com:
        web2.example.com:
    dbservers:
      hosts:
        db1.example.com:
    cacheservers:
      hosts:
        cache1.example.com:
  vars:
    ansible_user: admin
```
- To update system packages on all servers, target them with a playbook:
```
---
- name: Update all servers
  hosts: all
  become: true
  tasks:
    - name: Ensure all packages are up to date
      ansible.builtin.yum:
        name: '*'
        state: latest
```


### SIMPLE INVENTORY FILE
```
203.0.113.111  #IP  
203.0.113.112  
203.0.113.113  
server_hostname
```

#### GROUP IN INVENTORY
- Host can be part of multiple group(host atleast part of 2 groups)
    - Default group ---> all, ungroupd
    - User-defined groups

```
[webservers]
203.0.113.111
203.0.113.112

[dbservers]
203.0.113.113
server_hostname

[development]
203.0.113.111
203.0.113.113

[production]
203.0.113.112
server_hostname
```

#### GROUPING PATTERNS
- To list a range of hosts with similar patterns
```
[databases]
hosts[01:10].mycompany.com
```

#### NESTED GROUPS
- Parent group ---> metagroup

```
[web_dev]
203.0.113.111

[web_prod]
203.0.113.112

[db_dev]
203.0.113.113

[db_prod]
server_hostname

[webservers:children]
web_dev
web_prod

[dbservers:children]
db_dev
db_prod

[development:children]
web_dev
db_dev

[production:children]
web_prod
db_prod
```

### ANSIBLE INVENTORY VARIABLE
- Variables allow you to customize configuration per host or group (e.g., app version, SSH user, connection address).

- Way to define variables
    - Host-Level Variables:
    - Group-Level Variables:

- Common Connection Variables:
    Examples:
    - ansible_user → SSH user
    - ansible_host → Real IP/hostname
    - These can also be set as inventory variables.

#### HOST ALIAS
- To use an alias, include a variable named ansible_host after the alias name, containing the corresponding IP address or hostname of the server that should respond to that alias:
```
server1 ansible_host=203.0.113.111
server2 ansible_host=203.0.113.112
server3 ansible_host=203.0.113.113
server4 ansible_host=server_hostname
```

#### ANSIBLE USER
- We can use the Ansible inventory file to set variables that change Ansible’s default behavior.
- Inventory variables can be set for `individual hosts` or groups== of hosts.
- These variables are also `available inside playbooks`, allowing you to customize tasks per host or group.
```
server1 ansible_host=203.0.113.111 ansible_user=sammy
server2 ansible_host=203.0.113.112 ansible_user=sammy
server3 ansible_host=203.0.113.113 ansible_user=myuser
server4 ansible_host=server_hostname ansible_user=myuser
```

#### GROUP VARIABLES
```
[group_a]
server1 ansible_host=203.0.113.111 
server2 ansible_host=203.0.113.112

[group_b]
server3 ansible_host=203.0.113.113 
server4 ansible_host=server_hostname

[group_a:vars]
ansible_user=sammy
http_port=80
max_clients=200

[group_b:vars]
ansible_user=myuser
db_port=5432
db_name=myapp
```
```
[dev_web]
10.0.2.2

[prod_web]
10.0.2.3

[dev_database]
db ansible_host=10.0.2.4

[prod_database]
10.0.2.4

[all:vars]
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

#### Separate Variable Files
- Instead of defining variables directly in the inventory file, prefer using:
    - group_vars/ → For group-based variables
    - host_vars/ → For host-specific variables

    Example:
    group_vars/webservers
    ```
    http_port: 80
    ansible_user: automation_user
    ```
    Why?
    - Keeps things organized and clean
    - Allows hiding sensitive data (e.g., passwords, tokens)
    - Works better with version control systems

#### Multiple Inventory Sources
- Allows managing different environments or isolated setups together.
- Useful for scaling and centralizing inventory management.
- Mix Inventory Types:-
    - Static inventory files
    - Dynamic inventory scripts
    - Inventory plugins
    - Inventory directory
- For example, to target the development, testing, staging, and production hosts all at the same time:
```
ansible-playbook apply_updates.yml -i development -i testing -i staging -i production
```
#### Directory-Based Inventories
- We can combine multiple inventory sources in a directory and then manage them as one. This approach is useful when controlling static and dynamic hosts.
```
inventory/
  static-inventory-1
  static-inventory-2
  dynamic-inventory-1
  dynamic-inventory-1
```
- Then, we can run playbooks against all hosts in these inventories like this:
```
ansible-playbook apply_updates.yml -i inventory
```
- Ansible treats all files inside the inventory/ directory as part of the combined inventory.
- Managing both static and dynamic hosts (e.g., local servers + cloud instances).
- Simplifies maintenance when working with multiple sources or environments.

#### Security Best Practice
- Do not push your inventory file to Git if it contains secrets.
- Use a .gitignore file to exclude sensitive or unnecessary files.
```
inventory.ini
*.log
*.retry
*.swp
```

#### Setting Common Variables
```
[all:vars]
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

#### Ansible dynamic inventories
- Many modern environments are dynamic, cloud-based, possibly spread across multiple providers, and constantly changing.
- Automatically generate the list of managed nodes instead of using a static inventory.ini
- Why?: Useful in cloud environments (like AWS) where instances change frequently (created/destroyed dynamically).
- It will save time, manual update the list fo hosts, reducing errors.
- Two methods for tracking and targeting a dynamic set of hosts:
    1. Inventory Plugins (Recommended)
       - Preferred over scripts (more features, integrated with Ansible core).
       - View all available plugins(aws,azure,gcp etc):
        ```
        ansible-doc -t inventory -l
        ```
    2. Inventory Scripts (Older method)
       - Custom scripts that output inventory in JSON format.


#####  AWS EC2 Example Using Plugin
- Plugin: `amazon.aws.aws_ec2`
- To see plugin specific DOC: `ansible-doc -t inventory amazon.aws.aws_ec2`
- Install the Ansible Galaxy collection amazon.aws to work with the plugin: `ansible-galaxy collection install amazon.aws`

###### Example Configuration: dynamic_inventory_aws_ec2.yml
- To test created four EC2 instances on AWS.
- Added two tags, environment and role, on these EC2 instances, which we will use later to create inventory groups. 
```
plugin: amazon.aws.aws_ec2
regions:
  - us-east-1
  - us-east-2
  - us-west-2

hostnames: tag:Name

keyed_groups:
  - key: placement.region
    prefix: aws_region
  - key: tags['environment']
    prefix: env
  - key: tags['role']
    prefix: role

groups:
  private_only: "public_ip_address is not defined"

compose:
  ansible_host: public_ip_address | default(private_ip_address)

```
- View inventory structure:
`ansible-inventory -i dynamic_inventory_aws_ec2.yml --graph`

- Save inventory to a file:
`ansible-inventory -i dynamic_inventory_aws_ec2.yml --list --output inventory/dynamic_inventory_aws_ec2 -y`

#### Inventory file variables
```
server1 ansible_host=<IP_or_HOSTNAME> ansible_ssh_private_key_file=<public_key_file>
ansible_user=<username> ansible_ssh_pass=<password> ansible_connection=<connection_type> ansible_port=<port>
```
ansible_connection --> ssh,local,winrm