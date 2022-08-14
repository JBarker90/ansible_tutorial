# Ansible Ping Inventory

## 1. This will test the connection

```
ansible all --key-file ~/.ssh/ansible_key -i inventory -m ping
```

Example:

```
jonathan@dockerhost-01:~/ansible_tutorial$ ansible all --key-file ~/.ssh/ansible_key -i inventory -m ping
192.168.1.74 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
192.168.1.76 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
192.168.1.7cts": {
        "discovered_inte5 | SUCCESS => {
    "ansible_farpreter_python": "/usr/bin/python3"
    },
    "changed": false,u can automate t
    "ping": "pong"
}
```

## 2. You can automate this by creating an ansible config file called `ansible.cfg`

NOTE: This file will be read by ansible everytime you run it.

```
jonathan@dockerhost-01:~/ansible_tutorial$ cat ansible.cfg
# This Ansible config file will automate processes in the inventory file

[defaults]

inventory = inventory
private_key_file = ~/.ssh/ansible_key
```

## 3. Then you can shorten the command and ping all again using the ansible config file

```
ansible all -m ping
```

Example:

```
jonathan@dockerhost-01:~/ansible_tutorial$ ansibl all -m pineg
192.168.1.74 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
192.168.1.76 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
192.168.1.75 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

## 4. You can list hosts using ansible

```
ansible all --list-hosts
```

Example: What the output would look like

```
jonathan@dockerhost-01:~/ansible_tutorial$ ansible all --list-hosts
  hosts (3):
    192.168.1.74
    192.168.1.75
    192.168.1.76
```


## 5. You can use this ansible command to gather all of the info about the hosts. It will be useful. You can output a ton of info but you can pipe it through `less` or some other method to avoid a data dump on the terminal.

```
ansible all -m gather_facts
```

NOTE: You can limit this to one specific host

```
ansible all -m gather_facts --limit 192.168.1.74
```
