### SSH Key Copy

##### Copy a custom SSH Key
- ssh-copy-id command
```
ssh-copy-id -i ~/.ssh/ansible.pub vagrant@192.168.29.109

Default copy id_rsa.pub
```
- scp command
```
scp /path/to/your/custom_key.pub username@remote_host:~/.ssh/custom_key.pub
ssh username@remote_host 'cat ~/.ssh/custom_key.pub >> ~/.ssh/authorized_keys && rm ~/.ssh/custom_key.pub'
```

### Ansible-inventory command
- To Validate an Ansible inventory file and output the result in JSON format(Inventory listing): 
```
ansible-inventory -i inventory --list

{
   "_meta": {
       "hostvars": {
           "db": {
               "ansible_host": "10.0.2.4"
           }
       }
   },
   "all": {
       "children": [
           "ungrouped",
           "dev_web",
           "prod_web",
           "dev_database",
           "prod_database"
       ]
   },
   "dev_database": {
       "hosts": [
           "db"
       ]
   },
   "dev_web": {
       "hosts": [
           "10.0.2.2"
       ]
   },
   "prod_database": {
       "hosts": [
           "10.0.2.4"
       ]
   },
   "prod_web": {
       "hosts": [
           "10.0.2.3"
       ]
   }
}
```
- Save output to file
```
ansible-inventory -i inventory.ini --list --output output.json
```
- Display your inventory in YAML format:
```
ansible-inventory -i inventory.ini --list --yaml

all:
 children:
   dev_database:
     hosts:
       db:
         ansible_host: 10.0.2.4
   dev_web:
     hosts:
       10.0.2.2: {}
   prod_database:
     hosts:
       10.0.2.4: {}
   prod_web:
     hosts:
       10.0.2.3: {}
```
- To see all hosts 
```
ansible-inventory -i hosts --host all

{
    "ansible_host": "192.168.29.43",
    "ansible_user": "vagrant"
}
```

- Show variables assigned to a host:
```
ansible-inventory -i inventory.ini --host db  

{
   "ansible_host": "10.0.2.4"
}
```
- Show an inventory graph:
```
ansible-inventory -i inventory.ini --graph

@all:
 |--@ungrouped:
 |--@dev_web:
 |  |--10.0.2.2
 |--@prod_web:
 |  |--10.0.2.3
 |--@dev_database:
 |  |--db
 |--@prod_database:
 |  |--10.0.2.4
```

### List a host in a group of default inventory
```
ansible <group_name> --list-hosts
```

### Check is specific server is in default inventory or not
```
ansible <host_name> --list-hosts
```

### List all grouped host in default inventory
```
ansible all --list-hosts
```

### List all ungrouped host in default inventory
```
ansible ungrouped --list-hosts
```

### To gather a raw information about remote systems(ansible_facts)
```
ansible <host> -m setup 
ansible <host> -m setup > facts.txt

- name:
  setup: 
  register: result
- name:
  debug: 
    msg: result
```

### USING A CUSTOM INVENTORY
```
ansible all -i inventory -m ping
ansible-playbook -i inventory.ini playbook.yml
ansible-playbook -i inventory.yml update-packages.yml
```

### Check the syntax of playbook
```
ansible-playbook --syntax file.yaml
ansible-playbook --syntax-check file.yaml
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


#### To generate a template config file with all default settings:
```
ansible-config init --disabled > ansible.cfg
```

#### To include all available plugin settings:
```
ansible-config init --disabled -t all > ansible.cfg

```

#### Test the connection:
```ansible 192.168.25.16 -m ping
ansible hostname -m ping

SUCCESS => {"ping": "pong"}
```

#### Ping multiple hosts with wildcards
```
ansible webserver* -m ping -i inventory2.txt

```

#### Specify the Configuration File:
- Using the --config-file option: You can specify the path to your custom ansible.cfg file when running Ansible commands.
```
ansible-playbook --config-file /path/to/your/ansible.cfg playbook.yml
```

- Using the ANSIBLE_CONFIG environment variable: Set the ANSIBLE_CONFIG variable to the path of your custom ansible.cfg.
```
export ANSIBLE_CONFIG=/path/to/your/ansible.cfg
ansible-playbook playbook.yml
```

#### Here's an example of a basic ansible.cfg:
```
[defaults]
inventory = /path/to/your/inventory  # Replace with your inventoryfile path
remote_user = your_remote_user      # Replace with your remote user
host_key_checking = False          # Disable host key checking(use with caution)
stdout_callback = yaml             # Use YAML for output

[privilege_escalation]
become = True                      # Enable privilege escalation
become_method = sudo               # Use sudo for privilegeescalation
become_user = root                 # Become root user
become_ask_pass = False            # Don't ask for password (usewith keys)
```

#### Passing variable value from command line
```
ansible-playbook demo.yaml -e "user=mahin"
ansible-playbook demo.yaml -e "file=@vars.yaml"
```

#### Passing remote user from command line
```
ansible-playbook demo.yaml -u mahin
```
'-u ---> command line when running ansible or ansible-playbook, Overrides any user specified in inventory or playbook'

'ansible_user --> Set in inventory file, host vars, group vars, or inside the playbook, Used for per-host or per-group user setting'


#### verbos output of ansible command
```
ansible all -m ping -v to -vvvvv
ansible ungroupd -m ping -v to -vvvvv
ansible-playbook demo.yaml -v to -vvvvv
```

#### Find out which config file ansible uses
```
ansible --version
ansible-config dump | grep CONFIG_FILE
ansible all -m ping -v
```

#### Set & unset ansible config file in ENVIRONMENT variable
```
export ANSIBLE_CONFIG=./ansible.cfg
unset ANSIBLE_CONFIG
```

### Ansible Doc
##### List all ansible plugin
```
ansible-doc -l
ansible-doc -F
```

##### Get detail about any plugin
```
ansible-doc -t <plugin_type> <plugin_name>
```

##### Get detail information about specific module
```
ansible-doc <module>
ansible-doc ping
```


#### Different ways to user variable
1. vars:
```
vars:
    name: mahin
```
2. vars_files
```
vars_files:
    - file.yaml

#file.yaml
name: mahin

```
#### Referencing Variables
```
{{ var_name }} 
"{{ var_name }}"

In conditions
var_name ---> only by name
```

### host_vars and group_vars

##### Key Differences and Usage: `host_vars` vs `group_vars`

###### `host_vars`
- Directory that contains YAML files named after **individual hosts** in your inventory.
- Each file stores variables **specific to that host**.
- Example: `host_vars/webserver1.example.com.yml`

##### `group_vars`
- Directory that contains YAML files named after **groups** defined in your inventory.
- Each file stores variables **common to all hosts in that group**.
- Example: `group_vars/webservers.yml`

---

#### How They Work

##### 1. Inventory
- Ansible uses an inventory file (e.g., `hosts`) to define the hosts and groups to manage.

##### 2. Variable Files
- You create `host_vars/` and `group_vars/` directories.
- Inside them, place YAML files with variables for specific hosts or groups.

##### 3. Variable Precedence
- If a variable is defined in both `host_vars` and `group_vars`,  
  the value in `host_vars` **takes precedence** for that specific host.

##### 4. Playbook Usage
- Variables can be accessed using Jinja2 syntax:
  ```jinja
  {{ my_variable }}


### Ansible variabel Precedence

#### Ansible Variable Precedence

In Ansible, when the same variable is defined in multiple places, the one with **higher precedence** overrides the others.  
Below is the **order of precedence** from **lowest to highest**:

---

##### 🔽 Lowest Precedence

1. **Role Defaults**  
   - Defined in: `defaults/main.yml` inside a role.  
   - Purpose: Provide default values that can be easily overridden.

---

##### 🗂️ Inventory Variables

2. **Inventory `group_vars/` and `host_vars/`**  
   - Includes files like:  
     - `group_vars/all.yml`  
     - `group_vars/webservers.yml`  
     - `host_vars/hostname.yml`  
   - Used to define variables at the host or group level.

---

##### 📘 Playbook Variables

3. **Playbook `group_vars/` and `host_vars/`**  
   - Similar to inventory ones, but defined in the playbook directory.

4. **`vars_files`**  
   - Example:
     ```yaml
     vars_files:
       - vars/main.yml
     ```

5. **`vars_prompt`**  
   - Prompts user for input at runtime.

6. **`vars` in Play**  
   - Defined directly inside a playbook:
     ```yaml
     vars:
       app_port: 8080
     ```

---

##### ⚙️ Task-Level Variables

7. **Task and Block-level `vars:`**  
   - Used inside individual tasks or blocks of tasks.

8. **`set_fact` and `register`**  
   - `set_fact` creates persistent facts.
   - `register` stores the output of a task:
     ```yaml
     - name: Register output
       command: whoami
       register: result
     ```

---

##### 🧩 Include and Role Variables

9. **Role or Include Parameters**  
   - Passed when including a role:
     ```yaml
     - include_role:
         name: myrole
       vars:
         role_var: "value"
     ```

10. **`include_vars` Module**  
    - Dynamically loads variable files:
      ```yaml
      - include_vars: vars/app_vars.yml
      ```

---

##### 🔼 Highest Precedence

11. **Extra Variables (Command Line)**  
    - Passed using `-e` or `--extra-vars`:
      ```bash
      ansible-playbook site.yml -e "user=admin port=8080"
      ```
    - **Overrides everything**, including `set_fact`.

---

###### 📌 Note:
- Ansible always chooses the **highest-precedence definition** of a variable.
- Use lower-precedence levels for defaults and higher ones for overrides.

#### Force Handler
```
Adding --force-handlers to your command line
Adding force_handlers: True in your playbook
Adding force_handlers = True in ansible.cfg
```

### Playbook syntax
```
---
- name: Ultimate Ansible Playbook with All Features and Become
  hosts: all
  gather_facts: flase    # disable facts
  become: true           # 🔹 Elevates privilege at the play level
  force_handler: true    # 🔁 Run handlers even on failure
  remote_user: deployuser         # 🔐 SSH user for connection

  ############################################
  # 🔸 PLAYBOOK-LEVEL VARIABLES
  ############################################
  vars:
    playbook_var: "This is a playbook-level variable"
    user_name: "deployuser"
    packages:
      - nginx
      - git
    config_file: "/etc/myapp/config.yml"
    server_port: 8080
    welcome_message: "Welcome to {{ inventory_hostname }}!"
    nested_var:
      key1: "value1"
      key2:
        - one
        - two

  ############################################
  # 🔸 EXTERNAL VARIABLES
  ############################################
  vars_files:
    - vars/global.yml
    - vars/secrets.yml

  ############################################
  # 🔸 PROMPTED VARIABLES
  ############################################
  vars_prompt:
    - name: "admin_password"
      prompt: "Enter admin password"
      private: yes

  ############################################
  # 🔸 ENVIRONMENT VARIABLES
  ############################################
  environment:
    GLOBAL_ENV_VAR: "Playbook level environment"

  ############################################
  # 🔸 PRE-TASKS
  ############################################
  pre_tasks:
    - name: Ensure Python is installed (for raw SSH)
      become: true
      raw: test -e /usr/bin/python3 || (apt update && apt install -y python3)
      changed_when: false

  ############################################
  # 🔸 ROLES
  ############################################
  roles:
    - common
    - { role: apache, port: 80 }

  ############################################
  # 🔸 TASKS
  ############################################
  tasks:

    - name: Show all host facts
      debug:
        var: hostvars[inventory_hostname]

    - name: Register a value from a command
      become: true
      shell: echo "Dynamic value from command"
      register: dynamic_output

    - name: Set a runtime fact from registered output
      set_fact:
        runtime_fact: "{{ dynamic_output.stdout }}"

    - name: Create a file using playbook and runtime variables
      become: true
      copy:
        content: |
          {{ welcome_message }}
          Runtime Value: {{ runtime_fact }}
        dest: /tmp/welcome.txt

    - name: Loop over packages and install (Debian)
      become: true
      apt:
        name: "{{ item }}"
        state: present
      loop: "{{ packages }}"
      when: ansible_os_family == "Debian"
      environment:
        APT_ENV_VAR: "Set inside task"

    - name: Add line using nested variables
      become: true
      lineinfile:
        path: /etc/environment
        line: "CUSTOM_KEY={{ nested_var.key1 }}"
        create: yes

    - name: Include task file with task-specific vars
      include_tasks: tasks/custom_task.yml
      vars:
        custom_message: "This comes from include_tasks"

    - name: Use a template with multiple variables
      become: true
      template:
        src: templates/config.j2
        dest: "{{ config_file }}"
      notify: restart nginx

    - name: Demonstrate block/rescue/always
      block:
        - name: Intentional failure
          become: true
          command: /bin/false
      rescue:
        - name: Report failure
          debug:
            msg: "Command failed, but rescued!"
      always:
        - name: Always run
          debug:
            msg: "This runs no matter what."

    - name: Conditional task with failed_when & changed_when
      become: true
      command: echo "testing"
      register: test_output
      failed_when: "'fail' in test_output.stdout"
      changed_when: "'changed' in test_output.stdout"
    
    - name: Retry until success
      shell: "curl -sf http://localhost:{{ server_port }}"
      register: curl_result
      until: curl_result.rc == 0
      retries: 5
      delay: 10
      become: true

  ############################################
  # 🔸 HANDLERS
  ############################################
  handlers:
    - name: restart nginx
      become: true
      service:
        name: nginx
        state: restarted

  ############################################
  # 🔸 POST-TASKS
  ############################################
  post_tasks:
    - name: Print final success message
      debug:
        msg: "Play completed on {{ inventory_hostname }}"

  ############################################
  # 🔸 STRATEGY & TAGS
  ############################################
  strategy: linear
  tags: [always, deploy]

```



### Ansible Predefine Variables
| Variable                       | Description                          | Example Output (varies per host)           |
| ------------------------------ | ------------------------------------ | ------------------------------------------ |
| `ansible_hostname`             | Host's system hostname               | `"web01"`                                  |
| `ansible_distribution`         | OS name (from facts)                 | `"Ubuntu"`                                 |
| `ansible_distribution_version` | OS version                           | `"20.04"`                                  |
| `ansible_facts`                | All collected system info            | `{...full dictionary...}`                  |
| `inventory_hostname`           | Hostname as defined in inventory     | `"web1"`                                   |
| `groups`                       | Inventory groups the host belongs to | `{ 'web': ['web1'], 'all': ['web1'] }`     |
| `hostvars`                     | Dict of all hosts and their vars     | `hostvars['web1']['ansible_hostname']`     |
| `play_hosts`                   | List of hosts in current play        | `['web1', 'web2']`                         |
| `ansible_env`                  | Remote host environment variables    | `ansible_env.PATH` = `"/usr/bin:/bin:..."` |
| `ansible_user`                 | Remote SSH user                      | `"deployuser"`                             |
| `ansible_connection`           | Connection method used               | `"ssh"` or `"local"`                       |
| `ansible_playbook_dir`         | Path to the current playbook         | `"/home/user/ansible"`                     |
| `inventory_dir`                | Directory of inventory               | `"/home/user/ansible/inventory"`           |

```
---
- name: Demo Playbook – Using Predefined Ansible Variables
  hosts: all
  remote_user: deployuser
  become: true
  gather_facts: true
  force_handlers: true

  tasks:

    - name: 🔹 Print ansible_hostname (system's hostname)
      debug:
        msg: "Hostname: {{ ansible_hostname }}"

    - name: 🔹 Print OS distribution
      debug:
        msg: "Distribution: {{ ansible_distribution }} {{ ansible_distribution_version }}"

    - name: 🔹 Print full ansible_facts dictionary
      debug:
        var: ansible_facts

    - name: 🔹 Print inventory_hostname (from inventory)
      debug:
        msg: "Inventory hostname: {{ inventory_hostname }}"

    - name: 🔹 Show group membership
      debug:
        msg: "This host belongs to groups: {{ groups }}"

    - name: 🔹 Show variables from another host using hostvars
      debug:
        msg: "Hostname of first host in play: {{ hostvars[play_hosts[0]]['ansible_hostname'] }}"

    - name: 🔹 Print play_hosts list
      debug:
        msg: "All hosts in this play: {{ play_hosts }}"

    - name: 🔹 Print an environment variable from the remote machine
      debug:
        msg: "PATH env var: {{ ansible_env.PATH }}"

    - name: 🔹 Print SSH user being used
      debug:
        msg: "SSH user: {{ ansible_user }}"

    - name: 🔹 Show connection type to host
      debug:
        msg: "Connection type: {{ ansible_connection }}"

    - name: 🔹 Print the directory of the playbook
      debug:
        msg: "Playbook is running from: {{ ansible_playbook_dir }}"

    - name: 🔹 Print inventory directory
      debug:
        msg: "Inventory dir: {{ inventory_dir }}"

```

### Forks and Serial
##### Serial
- If you want to control how many hosts run the playbook at the same time, you can use serial.
- Serial: 1 means Ansible will:
    - Run all tasks on webserver1 first.
    - Then move to sqlserver1.
```
name: "strategy demo"
hosts: webserver1, sqlserver1
serial: 1
tasks:
- name: "first task"
apt: name='apache2' state='present'
- name: "second task"
command: touch /tmp/strategy_task2.txt
- name: "3rd task"
command: sleep 30
```

##### Forks
- Controlling How Many Hosts Run at Once
- Ansible doesn’t hit all hosts at the same time — the number of parallel connections is controlled by forks.
    • By default, Ansible uses 5 forks.
    • Defined in /etc/ansible/ansible.cfg.
    • You can increase this number, but make sure your machine has enough CPU and RAM.
```
Defining forks on the command line:
1. Use the --forks or -f option: When running an Ansible command or playbook,
use the --forks (or its short form -f) option followed by the desired number of forks. For example, to run a playbook with 50 forks, you would use: ansible- playbook -f 50 playbook.yml. @
```
```
Option 1: In ansible.cfg
[defaults]
forks = 20

Option 2: Via Command Line
ansible-playbook site.yml -f 10
```


#### Linear and Free strategy
- In Ansible, strategies control how tasks are run on multiple hosts. By default, Ansible uses the linear strategy.
- Linear Strategy (Default)
    • All hosts start the same task together.
    • No host can move to the next task until all hosts finish the current task.
```
- name: "strategy demo" 
  hosts: webserver1, sqlserver1 
  strategy: linear
  tasks:
  - name: "first task"
    apt: 
        name='apache2' 
        state='present' 
  - name: "second task"
        command: touch /tmp/strategy_task2.txt
```
-     If apache2 is already installed on webserver1 but sqlserver1 takes longer to install it, webserver1 will wait until sqlserver1 finishes before moving to the second task.


- Free Strategy
    • Hosts work independently.
    • No waiting! Each host moves to the next task as soon as it finishes the current one.
```
- name: "strategy demo" 
  hosts: webserver1, sqlserver1 
  strategy: free
  tasks:
  - name: "first task"
    apt:
        name='apache2' 
        state='present'
  - name: "second task"
        command: touch /tmp/strategy_task2.txt
```
- Here, webserver1 does not wait for sqlserver1 — both do their jobs at their own pace.


### Ansible Configuration File
```
[defaults]
inventory = /root/Ansible/inventory
remote_user = admin
roles_path = /root/Ansible/roles
collections_path = /root/Ansible/mycollection
ask_pass = false  #enable remote user pass or not
host_key_checking = false
private_key_file = ~/.ssh/ansible
DEFAULT_POLL_INTERVAL = 15(default) #https://docs.ansible.com/ansible/latest/reference_appendices/config.html#default-poll-interval

[privilege_escalation]
become = true    #Enable a privilage escalation
become_method = sudo  #Escalation method
become_user = root #User to switch
become_ask_pass = false  #become root user without password

[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=30m #

# Why Use This?
- Improves Ansible performance by reducing repeated SSH handshakes.  
- Helpful for large playbooks or multiple tasks targeting the same host(s).

ControlMaster=auto
  Enables connection sharing by reusing existing SSH connections.
  If a master connection exists, it reuses it.
  This significantly reduces overhead for multiple tasks.
ControlPersist=30m
  Keeps the SSH master connection open for 30 minutes after the last command.
  Makes repeated SSH tasks faster within that time window.

When you run an Ansible playbook, it connects to the remote host using a specific user (like ubuntu or ec2-user) and runs tasks as that user.

But what if that user doesn't have permission to do something important — like editing /etc/resolv.conf, installing packages, or restarting services?

That’s where become comes in.
become lets Ansible temporarily switch to a more powerful user, like root, to run certain tasks.
It’s a safe and clean way to gain elevated privileges — much better than hardcoding usernames or passwords.
```

#### Configuration file presedence
```
ANSIBLE_CONFIG
./ansible.cfg
~/.ansible.cfg
/etc/ansible/ansible.cfg
```


| Method   | Description                                     | Notes                          |
| -------- | ----------------------------------------------- | ------------------------------ |
| `sudo`   | Uses the `sudo` command to escalate privileges  | ✅ Most commonly used (default) |
| `su`     | Switches user using `su` command                | Requires `su` access           |
| `pbrun`  | Uses PowerBroker's `pbrun` for escalation       | Enterprise environments        |
| `pfexec` | Uses Solaris `pfexec` utility                   | Solaris-specific               |
| `doas`   | Lightweight sudo alternative                    | Used in OpenBSD systems        |
| `dzdo`   | Uses Centrify’s `dzdo` for privilege escalation | Centrify-based systems         |
| `ksu`    | Kerberized `su`, for Kerberos auth              | Requires Kerberos              |
| `runas`  | Windows-only method to escalate privileges      | Windows target hosts           |

### Ansible Vault
- Encrypting Content with Ansible Vault 
- When you store sensitive info like IP addresses, passwords, private keys in plain text, it's a security risk.
- Ansible Vault helps solve this by letting you encrypt those files.

- Why use Vault?
  • Protect sensitive data (passwords, keys, configs).
  • Encrypt playbooks, variable files, and inventory files.
- Decrypt happens automatically when the password is provided during execution.

- Encrypt already created vault
```
ansible-vault encrypt inventory
```

- Example: Encrypt inventory.txt and save as enc_inven.txt:
```
ansible-vault encrypt inventory --output enc_inven.txt
```
- You’ll be asked to set a password.
- The output file will look like encrypted gibberish.

- Using a encrypted inventory in playbook
```
ansible-playbook -i inventory demo.yaml --ask-vault-pass
```

- Using file to store vault key
```
echo "vault_password" > password.txt
ansible-playbook -i inventory demo.yaml --vault-password-file=password.txt
```
- `--ask-vault-pass` - prompts you for the Vault password at runtime.
- `--vault-password-file=<password_file>` - to give password file instead of prompt.
- Ansible decrypts it on the fly when running the playbook.


- Viewing Encrypted Files:
```
ansible-vault view enc_inven.txt
```
- Enter your password.
- It will show the original, decrypted content.

- Create the new vault and also set the password
```
ansible-vault create var.yaml
```

- Reset the password of ansible vault
```
ansible-vault rekey var.yaml
```

- Edit the vault content
```
ansible-vault edit var.yaml
```

### 🔍 What Are Lookups in Ansible?

In real-world automation, important data like **server details**, **passwords**, or **configuration settings** often live in files like CSV, INI, or TXT.  
**Ansible lookups** help you fetch that data **at runtime**, directly into your playbooks.

---

#### 📌 Why Use Lookups?

- Pull passwords, IPs, and settings **dynamically** from files or external sources.
- Keep your inventory clean and secure.
- Separate data from logic — make your playbooks modular and maintainable.

---

#### 📁 Examples of Lookup Usage

##### 🔐 Example 1: CSV Lookup for Passwords

###### 📄 Inventory (without passwords)

<img src="https://github.com/Mahin556/Ansible/blob/3520652567076b61813b092ebaba3d8828ce7bf9/img/image.png" alt="Inventory Example" width="600"/>

---

###### 📑 Password File: `credentials.csv`

```csv
hostname,password
webserver1,MySuperSecret
webserver2,AnotherSecret
```

<img src="https://github.com/Mahin556/Ansible/blob/3520652567076b61813b092ebaba3d8828ce7bf9/img/image%20copy%202.png" alt="CSV File Example" width="600"/>

---

###### 📘 Playbook: CSV Lookup

```yaml
- name: Use CSV Lookup
  hosts: webservers
  vars:
    my_password: "{{ lookup('csvfile', inventory_hostname + ' file=credentials.csv delimiter=, col=1') }}"
  tasks:
    - name: Show password from CSV
      debug:
        msg: "Password is {{ my_password }}"
```

<img src="https://github.com/Mahin556/Ansible/blob/3520652567076b61813b092ebaba3d8828ce7bf9/img/image%20copy.png" alt="CSV Lookup Playbook" width="600"/>

> ✅ Ansible pulls the password for `webserver1` from `credentials.csv` dynamically.

---

##### 🔐 Example 2: INI Lookup for Passwords

###### 📑 INI File: `credentials.ini`

```ini
[webserver1]
password=MySuperSecret

[webserver2]
password=AnotherSecret
```

<img src="https://github.com/Mahin556/Ansible/blob/3520652567076b61813b092ebaba3d8828ce7bf9/img/image%20copy%203.png" alt="INI File Example" width="600"/>

---

###### 📘 Playbook: INI Lookup

```yaml
- name: Use INI Lookup
  hosts: webservers
  vars:
    my_password: "{{ lookup('ini', inventory_hostname + '.password file=credentials.ini') }}"
  tasks:
    - name: Show password from INI
      debug:
        msg: "Password is {{ my_password }}"
```

<img src="https://github.com/Mahin556/Ansible/blob/3520652567076b61813b092ebaba3d8828ce7bf9/img/image%20copy%204.png" alt="INI Lookup Playbook" width="600"/>

---

#### ⚙️ General Lookup Syntax

```yaml
{{ lookup('plugin_name', 'query string') }}
```

##### 🔤 Syntax References

<p float="left">
  <img src="https://github.com/Mahin556/Ansible/blob/3520652567076b61813b092ebaba3d8828ce7bf9/img/image%20copy%205.png" alt="Syntax Example 1" width="750"/>
   <br>
  <img src="https://github.com/Mahin556/Ansible/blob/3520652567076b61813b092ebaba3d8828ce7bf9/img/image%20copy%206.png" alt="Syntax Example 2" width="750"/>
  <br>
  <img src="https://github.com/Mahin556/Ansible/blob/3520652567076b61813b092ebaba3d8828ce7bf9/img/image%20copy%207.png" alt="Syntax Example 3" width="750"/>
</p>

---

#### 💡 Key Points About Lookups

- ✅ Lookups **run on the control node**, not the managed node.
- 🔐 Sensitive files (like passwords) **stay on the control machine** — not copied to the remote.
- 🔄 Useful for **CSV**, **INI**, **YAML**, **Vault**, **API**, and more.
- ⚠️ If a file doesn’t exist on the control node, **lookup will fail**.
- 🧼 Keeps your playbooks **agentless** and **clean** — no extra software needed on targets.


### Ansible Asynchronous Actions and Polling
- By default, Ansible runs tasks synchronously — meaning it waits for one task to finish before moving on to the next. This can be a problem if a task takes too long, like over 10 minutes, because the SSH connection between the Ansible controller and the remote machine might time out before the task finishes.

- To handle long-running tasks, Ansible offers Asynchronous Mode, which allows tasks to run in the background so the playbook doesn’t have to wait, and it can check their status later using polling.

#### Synchronous Example:

- In a normal setup, if you write a task like this:
```
- name: this is our 1st play.
  hosts: webserver1
  tasks:
  - name: "sleep for 120 sec"
    command: sleep 120
  - name: "second task"
    command: touch /tmp/second_task.txt
```
- The playbook will wait for 120 seconds before moving to the second task. If the connection drops during this time, the task fails.

#### Asynchronous Example:
- You can avoid that by using async and poll. Here’s how:
```
- name: this is our 1st play.
  hosts: webserver1
  tasks:
  - name: "sleep for 60 sec"
    command: sleep 60
    async: 70
    poll: 35
  - name: "second task"
    command: touch /tmp/second_task.txt
```
- async: 70 means the task is allowed to run for up to 70 seconds.
- poll: 35 means Ansible will check the task status every 35 seconds.

#### Fire-and-Forget Mode (poll: 0):
- If you don’t want Ansible to wait at all, you can set poll: 0. Ansible will start the task and immediately move to the next one without checking if the first task finished.

```
- name: this is our 1st play. hosts: webserver1
  tasks:
  - name: "sleep for 120 sec"
    command: sleep 120
    async: 60
    poll: e
  - name: "second task"
    command: touch /tmp/second_task.txt
```

- In this case, the sleep command starts and Ansible jumps to the next task without waiting.
- If you don’t set async, the task runs normally (synchronously).
- The default polling interval is 15 seconds, which can be changed in the Ansible config (DEFAULT_POLL_INTERVAL).
- async and poll are helpful for long or non-critical tasks, especially when you want to save time or avoid connection problems.


### Roles
#### Create a Role
```
ansible-galaxy init dummy
```
Default Roles directory in ansible host ---> /etc/ansible/roles

- including role in playbook
```
---
- name:
  roles:
  - role: <role_name>
    vars:
      role_var:<value>

- hosts: all
  roles:
    - common
    - webserver

# import_role is used to statically import a role.
- hosts: all
  tasks:
  - name: Run some other task
    import_role:
      name: common
    
    import_role:
      name: webserver

# include_role is used to dynamically include a role.
- hosts: all
  tasks:
  - name: Run some other task
    include_role:
      name: common
    
    include_role:
      name: webserver
```

### Specify the config file in ansible CLI Command
```
ansible-playbook --config /path/to/your/custom_ansible.cfg your_playbook.yml
```
```
Explanation:
ansible-playbook: The Ansible command being executed (e.g., ansible, ansible-playbook, ansible-galaxy).
--config or -c: The option to specify the path to your custom configuration file.
/path/to/your/custom_ansible.cfg: The absolute or relative path to your custom Ansible configuration file.
your_playbook.yml: The playbook you are executing (or other arguments relevant to the specific Ansible command).
```
- Precedence:
- It is important to note the order of precedence for Ansible configuration files:
  - ANSIBLE_CONFIG environment variable: If this environment variable is set, it takes the highest precedence.
  - ansible.cfg in the current working directory: If found, this file is used.
  - ~/.ansible.cfg in the user's home directory: If the above are not found, Ansible looks here.
  - /etc/ansible/ansible.cfg: The default system-wide configuration file.
- Using the --config option directly on the command line overrides all other configuration file locations for that specific command execution.
