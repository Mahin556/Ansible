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
- Host can be part of multiple group
- Default group ---> all, ungroupd (host atleast part of 2 groups)

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
- Inventory variables can be set for ==individual hosts or groups== of hosts.
- These variables are also ==available inside playbooks==, allowing you to customize tasks per host or group.
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

