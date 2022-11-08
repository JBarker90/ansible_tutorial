## Adding Users and Bootstrapping

- https://docs.ansible.com/ansible/2.4/user_module.html
- https://docs.ansible.com/ansible/2.3/authorized_key_module.html

1. We added play that will create the user named `dave` using the `user` module in Ansible.

```
- hosts: all
  become: true
  tasks:

  - name: create dave user
    tags: always
    user:
      name: dave
      groups: root
```

Proof of Concept: 

- You can see that most recently added users in `/etc/passwd`

```
jonathan@ansiblelab-01:~$ cat /etc/passwd | tail -n3
uuidd:x:108:116::/run/uuidd:/usr/sbin/nologin
tcpdump:x:109:117::/nonexistent:/usr/sbin/nologin
jonathan:x:1000:1000:Jonathan Barker,,,:/home/jonathan:/bin/bash
```

- Run playbook

```
ansible-playbook --ask-become-pass site.yml
```

NOTE: Output summary (shortened)

```
TASK [create dave user] **********************************************************************************************
changed: [192.168.1.75]
changed: [192.168.1.77]
changed: [192.168.1.76]
changed: [192.168.1.71]
changed: [192.168.1.74]

PLAY RECAP ***********************************************************************************************************
192.168.1.71               : ok=7    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
192.168.1.74               : ok=7    changed=1    unreachable=0    failed=0    skipped=5    rescued=0    ignored=0
192.168.1.75               : ok=6    changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
192.168.1.76               : ok=6    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
192.168.1.77               : ok=9    changed=1    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0
```

- Verify the user was created in `/etc/passwd` on same node.

```
jonathan@ansiblelab-01:~$ cat /etc/passwd | tail -n3
tcpdump:x:109:117::/nonexistent:/usr/sbin/nologin
jonathan:x:1000:1000:Jonathan Barker,,,:/home/jonathan:/bin/bash
dave:x:1001:1001::/home/dave:/bin/sh
```

2. We can add another play that will add the SSH pub key for Ansible using `authorized_key` module for the new user that was created. Then a second play that will add sudo privileges

```
- hosts: all
  become: true
  tasks:

  - name: create dave user
    tags: always
    user:
      name: dave
      groups: root

  - name: add ssh key for dave
    tags: always
    authorized_key:
      user: dave
      key: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGAhvL68zMlGEqZtBBiJskMpRgXRMX6N7/6BfWxZ8s7Q Ansible keys for Docker host"
  - name: add sudoers files for dave
    tags: always
    copy:
      src: sudoer_dave
      dest: /etc/sudoers.d/dave
      owner: root
      group: root
      mode: 0440
```

3. Then, we can create a `sudoer_dave` file that will set the sudo privilege for this user.

```
jonathan@dockerhost-01:~/ansible_tutorial/files$ cat sudoer_dave
dave ALL=(ALL) NOPASSWD: ALL
```

4. Now, if we run the playbook it will add the SSH pub key as an authorized key for the user `dave` and then add the sudoers file using the `copy` module.

```
TASK [add ssh key for dave] ******************************************************************************************
changed: [192.168.1.74]
changed: [192.168.1.75]
changed: [192.168.1.77]
changed: [192.168.1.76]
changed: [192.168.1.71]

TASK [add sudoers files for dave] ************************************************************************************
changed: [192.168.1.71]
changed: [192.168.1.74]
changed: [192.168.1.75]
changed: [192.168.1.76]
changed: [192.168.1.77]

PLAY RECAP ***********************************************************************************************************
192.168.1.71               : ok=9    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
192.168.1.74               : ok=9    changed=2    unreachable=0    failed=0    skipped=5    rescued=0    ignored=0
192.168.1.75               : ok=8    changed=2    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
192.168.1.76               : ok=8    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
192.168.1.77               : ok=11   changed=2    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0
```

5. Now, we can use this background user to run Ansible play books so that we do NOT have to include `--ask-become-pass` everytime we run the playbook. 

- To do this, we can modify the `ansible.cfg` config file and add our new user.

```
jonathan@dockerhost-01:~/ansible_tutorial$ cat ansible.cfg
# This Ansible config file will automate processes in the inventory file

[defaults]

inventory = inventory
private_key_file = ~/.ssh/ansible_key
remote_user = dave
```

- Then, we can test that it works by running the following:

```
ansible-playbook site.yml
```

Output summary (this doesn't include full output to avoid dump):

```
jonathan@dockerhost-01:~/ansible_tutorial$ ansible-playbook site.yml

PLAY [all] ***********************************************************************************************************

PLAY RECAP ***********************************************************************************************************
192.168.1.71               : ok=9    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
192.168.1.74               : ok=9    changed=0    unreachable=0    failed=0    skipped=5    rescued=0    ignored=0
192.168.1.75               : ok=8    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
192.168.1.76               : ok=8    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
192.168.1.77               : ok=11   changed=0    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0
```


6. Typically when you first provision a fresh server, it is good to create a separate bootstrap playbook that will configure users and everything to configure the server initially. Then run a second playbook that will configure/install services

```
---

- hosts: all
  become: true
  pre_tasks:

  - name: install updates (AlmaLinux)
    tags: always
    dnf:
      update_only: yes
      update_cache: yes
    when: ansible_distribution == "AlmaLinux"

  - name: install updates (Ubuntu)
    tags: always
    apt:
      upgrade: dist
      update_cache: yes
    when: ansible_distribution == "Ubuntu"

- hosts: all
  become: true
  tasks:

  - name: create dave user
    tags: always
    user:
      name: dave
      groups: root

  - name: add ssh key for dave
    tags: always
    authorized_key:
      user: dave
      key: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGAhvL68zMlGEqZtBBiJskMpRgXRMX6N7/6BfWxZ8s7Q Ansible keys for Docker host"
  - name: add sudoers files for dave
    tags: always
    copy:
      src: sudoer_dave
      dest: /etc/sudoers.d/dave
      owner: root
      group: root
      mode: 0440
```

- On a fresh server Ansible will not already be run. So this play will initially create the privileged user Dave that will run in the background. When initially running play you do need the become password and then once sudoer file is created/set it will NOT require a password

```
ansible-playbook --ask-become-pass bootstrap.yml
```

- After you run initial bootstrap playbook that will create the user then you can run the `site.yml` playbook to configure/install things without the become pass

```
ansible-playbook site.yml
```