• Tags in Ansible are used to run only specific tasks or parts of a playbook instead of running everything
• They help you save time, debug faster, and control 

• What tags do
    • Attach a label to a task, role, or block
    • Run only tasks with matching tags
    • Skip unwanted tasks during execution

```yaml
- name: Install Apache
  yum:
    name: httpd
    state: present
  tags:
    - lamp
    - apache
```

```bash
ansible-playbook lamp.yml --tags lamp # Use --tags to run tasks with specific tags

ansible-playbook lamp.yml --tags "apache,php" #Runs tasks tagged with apache OR php

ansible-playbook lamp.yml --skip-tags never #Skip tasks with that label -> Tasks tagged with "never" will not run

ansible-playbook lamp.yml --list-tags #Shows all tags available in the playbook
```
```yaml
# Tasks tagged with "never" do not run by default
#
# Example:
#
- name: Drop database
  mysql_db:
    name: appdb
    state: absent
  tags:
    - never
#
# This task will run ONLY if explicitly called:
# ansible-playbook lamp.yml --tags never
```

```yaml
# --------------------------------
# Using tags with roles
# --------------------------------
# roles:
#   - role: apache
#     tags: apache
#   - role: mysql
#     tags: mysql
#   - role: php
#     tags: php
#
# Run only MySQL role:
# ansible-playbook lamp.yml --tags mysql
```