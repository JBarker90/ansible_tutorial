# Writing Our first Playbook

## All of these changes and files can be found on GitHub and YouTube for Learn Linux TV

- https://docs.ansible.com/ansible/2.4/apt_module.html 
- https://github.com/JBarker90/ansible_tutorial.git

## 1. We can create a simple yml file called `install_apache.yml` 

- This will install the apache2 package using apt package installer

```
---

- hosts: all
  become: true
  tasks:

  - name: install apache2 package
    apt:
      name: apache2
```

NOTE: To run the playbook you can use the `ansible-playbook` command

```
ansible-playbook --ask-become-pass install_apache.yml
```

Example:

```
jonathan@dockerhost-01:~/ansible_tutorial$ ansible-playbook --ask-become-pass install_apache.yml                      
BECOME password:                                                                                                      
                                                                                                                      
PLAY [all] ***********************************************************************************************************
                                                                                                                      
TASK [Gathering Facts] ***********************************************************************************************
ok: [192.168.1.75]                                                                                                    
ok: [192.168.1.76]                                                                                                    
ok: [192.168.1.74]                                                                                                    
                                                                                                                      
TASK [install apache2 package] ***************************************************************************************
changed: [192.168.1.75]                                                                                               
changed: [192.168.1.76]                                                                                               
changed: [192.168.1.74]                                                                                               
                                                                                                                      
PLAY RECAP ***********************************************************************************************************
192.168.1.74               : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0    
192.168.1.75               : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0    
192.168.1.76               : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0                                                      
```


## 2. Then added a simple line to playbook to update apt cache before installing

```
---

- hosts: all
  become: true
  tasks:

    # This will update apt repo cache and then install apache2 package. This will try to avoid stale apt cache
  - name: update repository index
    apt:
      update_cache: yes

  - name: install apache2 package
    apt:
      name: apache2
```

NOTE: It did not change anything other than the repo index for apt. 

```
jonathan@dockerhost-01:~/ansible_tutorial$ ansible-playbook --ask-become-pass install_apache.yml
BECOME password:

PLAY [all] ***********************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************ok: [192.168.1.75]
ok: [192.168.1.74]
ok: [192.168.1.76]

TASK [update repository index] ***************************************************************************************changed: [192.168.1.75]
changed: [192.168.1.74]
changed: [192.168.1.76]

TASK [install apache2 package] ***************************************************************************************ok: [192.168.1.74]
ok: [192.168.1.75]
ok: [192.168.1.76]

PLAY RECAP ***********************************************************************************************************192.168.1.74               : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
192.168.1.75               : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
192.168.1.76               : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

## 3. Then add a play to install PHP support

```
---

- hosts: all
  become: true
  tasks:

    # This will update apt repo cache and then install apache2 package. This will try to avoid stale apt cache
  - name: update repository index
    apt:
      update_cache: yes

  - name: install apache2 package
    apt:
      name: apache2

  - name: add php support for apache
    apt:
      name: libapache2-mod-php
```

NOTE: When running the playbook again nothing changed except for the installion of libapache2-mod-php package

```
TASK [add php support for apache] ************************************************************************************changed: [192.168.1.76]
changed: [192.168.1.74]
changed: [192.168.1.75]

PLAY RECAP ***********************************************************************************************************192.168.1.74               : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
192.168.1.75               : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
192.168.1.76               : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```


## 4. Then added state latest to the playbook.

```
---

- hosts: all
  become: true
  tasks:

    # This will update apt repo cache and then install apache2 package. This will try to avoid stale apt cache
  - name: update repository index
    apt:
      update_cache: yes

  - name: install apache2 package
    apt:
      name: apache2
      state: latest

  - name: add php support for apache
    apt:
      name: libapache2-mod-php
      state: latest
```

NOTE: When running the playbook nothing changed because the latest version was already installed and apt already up to date

## 5. Copied the `install_apache.yml` file to `remove_apache.yml` to create a playbook that will remove apache2 and libapache2-mod-php

```
---

- hosts: all
  become: true
  tasks:

  - name: install apache2 package
    apt:
      name: apache2
      state: absent

  - name: add php support for apache
    apt:
      name: libapache2-mod-php
      state: absent
```

NOTE: Run playbook using the file name `remove_apache.yml`

```
ansible-playbook --ask-become-pass remove_apache.yml
```
