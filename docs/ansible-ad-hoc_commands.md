# Ansible Using Elevated Privileges with Ad-hoc commands

- https://www.learnlinux.tv/getting-started-with-ansible-05-running-elevated-commands/
- https://docs.ansible.com/ansible/2.9/modules/apt_module.html#apt-module
- https://docs.ansible.com/ansible/latest/user_guide/intro_adhoc.html

### 1. When making changes to remote hosts Ansible needs the equivalent of `sudo` privileges. 

- This command will use the `apt` module and update cache. Which is the equivalent of running `apt update` 
- But it will fail without having elevate privileges

```
ansible all -m apt -a update_cache=true
```

Example: They all failed. This is the same as running `apt update` without `sudo`

```
jonathan@dockerhost-01:~/ansible_tutorial$ ansible all -m apt -a update_cache=true
192.168.1.74 | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "msg": "Failed to lock apt for exclusive operation: Failed to lock directory /var/lib/apt/lists/: E:Could not open lock file /var/lib/apt/lists/lock - open (13: Permission denied)"
}
192.168.1.75 | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "msg": "Failed to lock apt for exclusive operation: Failed to lock directory /var/lib/apt/lists/: E:Could not open lock file /var/lib/apt/lists/lock - open (13: Permission denied)"
}
192.168.1.76 | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "msg": "Failed to lock apt for exclusive operation: Failed to lock directory /var/lib/apt/lists/: E:Could not open lock file /var/lib/apt/lists/lock - open (13: Permission denied)"
}
```

### 2. Ansible needs it's equivalent of `sudo` which is `become`

```
ansible all -m apt -a update_cache=true --become --ask-become-pass
```

- This will ask for the `BECOME` password which is the `sudo` password and then run

Example:

```
jonathan@dockerhost-01:~/ansible_tutorial$ ansible all -m apt -a update_cache=true --become --ask-become-pass
BECOME password:
192.168.1.76 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "cache_update_time": 1659544211,
    "cache_updated": true,
    "changed": true
}
192.168.1.75 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "cache_update_time": 1659544211,
    "cache_updated": true,
    "changed": true
}
192.168.1.74 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "cache_update_time": 1659544211,
    "cache_updated": true,
    "changed": true
}
```

### 3. How to install a package using elevated privileges

```
ansible all -m apt -a name=vim --become --ask-become-pass
```

Example: This is after it was installed so it will show "changed: false"

```
jonathan@dockerhost-01:~/ansible_tutorial$ ansible all -m apt -a name=vim --become --ask-become-pass
BECOME password:
192.168.1.74 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "cache_update_time": 1659544211,
    "cache_updated": false,
    "changed": false
}
192.168.1.75 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "cache_update_time": 1659544211,
    "cache_updated": false,
    "changed": false
}
192.168.1.76 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "cache_update_time": 1659544211,
    "cache_updated": false,
    "changed": false
}
```


### 4. You can upgrade existing packages using the following

```
ansible all -m apt -a "name=openssl state=latest" --become --ask-become-pass
```

Example: On one of the lab servers I saw that the openssl package needed to be upgraded. So this command will use apt to upgrade to the "latest state". There was a bunch of output so this in just the first few important lines

```
jonathan@dockerhost-01:~/ansible_tutorial$ ansible all -m apt -a "name=openssl state=latest" --become --ask-become-pass
BECOME password:
192.168.1.75 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "cache_update_time": 1659544211,
    "cache_updated": false,
    "changed": true,
    "stderr": "",
    "stderr_lines": [],
	(This continues on but too much data to include)
```


### 5. This will upgrade all of the packages on the servers using dist upgrade

```
ansible all -m apt -a "upgrade=dist" --become --ask-become-pass
```

Example: It took a really long time. But this is 

```
jonathan@dockerhost-01:~/ansible_tutorial$ ansible all -m apt -a "upgrade=dist" --become --ask-become-pass
BECOME password:
192.168.1.74 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": true,
	(This continues on but too much data to include. There was a bunch of msg output and packages that were upgraded)
```

Proof: This was the 01 ansible lab server

```
jonathan@ansiblelab-01:~$ sudo apt dist-upgrade
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Calculating upgrade... Done
The following packages have been kept back:
  python3-distupgrade ubuntu-release-upgrader-core
0 upgraded, 0 newly installed, 0 to remove and 2 not upgraded.
```
