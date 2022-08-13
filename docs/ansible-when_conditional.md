# The 'when' Conditional

## 1. This will allow a conditional to specify plays when different distros are in the inventory. 

## 2. Since our current playbook specifies apt as the package manager, our playbook will fail when our newer AlmaLinux 8 server was added to the inventory list.

```
ansible-playbook --ask-become-pass install_apache.yml
```

Example: Our output will fail because of AlmaLinux not using `apt` as the package manager

```
jonathan@dockerhost-01:~/ansible_tutorial$ ansible-playbook --ask-become-pass install_apache.yml
BECOME password:

PLAY [all] ***********************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************ok: [192.168.1.76]
ok: [192.168.1.74]
ok: [192.168.1.75]
[WARNING]: Platform linux on host 192.168.1.77 is using the discovered Python interpreter at /usr/libexec/platform-
python, but future installation of another Python interpreter could change the meaning of that path. See
https://docs.ansible.com/ansible/2.10/reference_appendices/interpreter_discovery.html for more information.
ok: [192.168.1.77]

TASK [update repository index] ***************************************************************************************[WARNING]: Updating cache and auto-installing missing dependency: python3-apt
fatal: [192.168.1.77]: FAILED! => {"changed": false, "cmd": "apt-get update", "msg": "[Errno 2] No such file or directory: b'apt-get': b'apt-get'", "rc": 2}
ok: [192.168.1.74]
ok: [192.168.1.75]
ok: [192.168.1.76]

TASK [install apache2 package] ***************************************************************************************ok: [192.168.1.74]
ok: [192.168.1.75]
ok: [192.168.1.76]

TASK [add php support for apache] ************************************************************************************ok: [192.168.1.74]
ok: [192.168.1.76]
ok: [192.168.1.75]

PLAY RECAP ***********************************************************************************************************192.168.1.74               : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
192.168.1.75               : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
192.168.1.76               : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
192.168.1.77               : ok=1    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0
```

## 3. You can add a when statement to playbook. Below is a when statement specifying ubuntu distro

```
---

- hosts: all
  become: true
  tasks:

    # This will update apt repo cache and then install apache2 package. This will try to avoid stale apt cache
  - name: update repository index
    apt:
      update_cache: yes
    when: ansible_distribution == "Ubuntu"

  - name: install apache2 package
    apt:
      name: apache2
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: add php support for apache
    apt:
      name: libapache2-mod-php
      state: latest
    when: ansible_distribution == "Ubuntu"
```

- However, when running the playbook now it just skips over the newer AlmaLinux server

```
TASK [update repository index] ***************************************************************************************skipping: [192.168.1.77]
ok: [192.168.1.74]
ok: [192.168.1.75]
ok: [192.168.1.76]

TASK [install apache2 package] ***************************************************************************************skipping: [192.168.1.77]
ok: [192.168.1.74]
ok: [192.168.1.75]
ok: [192.168.1.76]

TASK [add php support for apache] ************************************************************************************skipping: [192.168.1.77]
ok: [192.168.1.74]
ok: [192.168.1.76]
ok: [192.168.1.75]

PLAY RECAP ***********************************************************************************************************192.168.1.74               : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
192.168.1.75               : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
192.168.1.76               : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
192.168.1.77               : ok=1    changed=0    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0
```


## 4. Now, we will need to add a when statement for the AlmaLinux server. So we can run `gather_facts` against the newer server to see distro information

```
ansible all -m gather_facts --limit 192.168.1.77 | grep 'ansible_distribution'
```

```
jonathan@dockerhost-01:~/ansible_tutorial$ ansible all -m gather_facts --limit 192.168.1.77 | grep 'ansible_distribution'
        "ansible_distribution": "AlmaLinux",
        "ansible_distribution_file_parsed": true,
        "ansible_distribution_file_path": "/etc/redhat-release",
        "ansible_distribution_file_variety": "RedHat",
        "ansible_distribution_major_version": "8",
        "ansible_distribution_release": "Sky Tiger",
        "ansible_distribution_version": "8.6",
```

- We could write a very specific when statement targeting a specific distribution for AlmaLinux like the following

```
when: ansible_distribution == "AlmaLinux"
```

- I added a newer section to the install playbook to specify dnf

```
    # This section will update repo cache then install apache for AlmaLinux
  - name: update repository index
    dnf:
      update_cache: yes
    when: ansible_distribution == "AlmaLinux"

  - name: install apache package
    dnf:
      name: httpd
      state: latest
    when: ansible_distribution == "AlmaLinux"

  - name: add php support for apache
    dnf:
      name: php
      state: latest
    when: ansible_distribution == "AlmaLinux"
```

- When re-running the playbook now it runs plays based on distro

```
TASK [update repository index] ***************************************************************************************skipping: [192.168.1.74]
skipping: [192.168.1.75]
skipping: [192.168.1.76]
ok: [192.168.1.77]

TASK [install apache package] ****************************************************************************************skipping: [192.168.1.74]
skipping: [192.168.1.75]
skipping: [192.168.1.76]
changed: [192.168.1.77]

TASK [add php support for apache] ************************************************************************************skipping: [192.168.1.74]
skipping: [192.168.1.75]
skipping: [192.168.1.76]
changed: [192.168.1.77]

PLAY RECAP ***********************************************************************************************************192.168.1.74               : ok=4    changed=0    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0
192.168.1.75               : ok=4    changed=0    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0
192.168.1.76               : ok=4    changed=0    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0
192.168.1.77               : ok=4    changed=2    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0
```
