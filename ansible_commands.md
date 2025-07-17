
### VALIDATING A INVENTORY FILE AND GIVE OUTPUT IN JSON FORMAT
ansible-inventory -i inventory --list

### USING A CUSTOM INVENTORY
ansible all -i inventory -m ping
ansible-playbook -i inventory playbook.yml

