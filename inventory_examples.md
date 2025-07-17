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

### SIMPLE INVENTORY FILE
`
203.0.113.111 #IP
203.0.113.112
203.0.113.113
server_hostname #Hostname
`

#### GROUP IN INVENTORY
- Default group ---> all, ungroupd (host atleast part of 2 groups)

#### TWO FORMAT TO WIRTE INVENTORY FILE
