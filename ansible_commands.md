
### VALIDATING A INVENTORY FILE AND GIVE OUTPUT IN JSON FORMAT
```ansible-inventory -i inventory --list```

### USING A CUSTOM INVENTORY
```
ansible all -i inventory -m ping
ansible-playbook -i inventory playbook.yml
ansible-playbook -i inventory.yml update-packages.yml
```
### RUN A PLAYBOOK AGAINST SPECIFIC GROUP
```
ansible-playbook -i inventory.yml -l webservers deploy-web.yml
```

### TARGET HOSTS WITH PATTERNS
- Ansible lets you use patterns to specify which hosts or groups to run commands/playbooks on.
- Hosts are organized into groups like [webservers], [dbservers], [development], and [production].
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
- Logical Pattern Operators:
```
: → OR (e.g. host1:host2 targets both)

\& → AND (e.g. dbservers:\&production targets hosts in both groups)

\! → NOT (e.g. dbservers:\!production targets dbservers not in production)

ansible dbservers:\&production -m ping
ansible dbservers:\!production -m ping

```
- Common Pattern Examples

| Pattern | Targets |
|---------|---------|
|all |All hosts|
|host1 |Only host1|
|host1:host2 |host1 OR host2|
|group1 |All hosts in group1|
|group1:group2 |Hosts in either group1 OR group2|
|group1:\\\&group2 |Hosts in BOTH group1 AND group2|
|group1:\\\!group2 |Hosts in group1 but NOT in group2|
