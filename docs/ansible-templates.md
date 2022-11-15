## Templates

1. Using templates, you can create template of something like an SSH configuration file to modify and then automate changes

2. As an example, we can store a copy of the SSH config file from `/etc/ssh/sshd_config` (since the path is the same regardless of the distro) and store this in version control

- On our ansible controller we can copy a version of the ssh config file in a newer directory called `templates` located in the base role `roles/base/templates` and save it with a Jinja 2 extension

- Then we can rsync a copy of the ssh config from the AlmaLinux server

```
jonathan@dockerhost-01:~/ansible_tutorial/roles/base/templates$ cp -av /etc/ssh/sshd_config sshd_config_ubuntu.j2
'/etc/ssh/sshd_config' -> 'sshd_config_ubuntu.j2'

jonathan@dockerhost-01:~/ansible_tutorial/roles/base/templates$ rsync -av root@ansiblelab-04.jbarkersquared.net:/etc/ssh/sshd_config sshd_config_alma.j2
root@ansiblelab-04.jbarkersquared.net's password:
receiving incremental file list
sshd_config

sent 43 bytes  received 4,360 bytes  677.38 bytes/sec
total size is 4,269  speedup is 0.97
```

3. Now, we can turn the files into templates. 

- In both files we can add an `AllowUsers` variable 

```
jonathan@dockerhost-01:~/ansible_tutorial/roles/base/templates$ grep -i 'allowusers' sshd_config_alma.j2
AllowUsers {{ ssh_users }}

jonathan@dockerhost-01:~/ansible_tutorial/roles/base/templates$ grep -i 'allowusers' sshd_config_ubuntu.j2
AllowUsers {{ ssh_users }}
```

- Then we will need to edit the host variable files in `host_vars/` and add the variables to the host-specific variables files

```
jonathan@dockerhost-01:~/ansible_tutorial/host_vars$ cat 192.168.1.74.yml
apache_package_name: apache2
apache_service: apache2
php_package_name: libapache2-mod-php
ssh_users: "jonathan dave"
ssh_template_file: sshd_config_ubuntu.j2

jonathan@dockerhost-01:~/ansible_tutorial/host_vars$ cat 192.168.1.77.yml
apache_package_name: httpd
apache_service: httpd
php_package_name: php
ssh_users: "jonathan dave"
ssh_template_file: sshd_config_alma.j2
```

NOTE: Here are two examples. But I added `ssh_users` and `ssh_template_file` variables to all variable files

- Then I created a host variable file for my local workstation (because it will fail without this host variable)

```
jonathan@dockerhost-01:~/ansible_tutorial/host_vars$ cp -av 192.168.1.74.yml 192.168.1.71.yml
'192.168.1.74.yml' -> '192.168.1.71.yml'
```

4. After creating the templates, we can add a task in `roles/base/tasks/main.yml` to generate an sshd_config file from the template

- This task is added below the original one that was already there

```
jonathan@dockerhost-01:~/ansible_tutorial/roles/base/tasks$ tail -n9 main.yml
- name: generate sshd_config file from template
  tags: ssh
  template:
    src: "{{ ssh_template_file }}"
    dest: /etc/ssh/sshd_config
    owner: root
    group: root
    mode: 0644
  notify: restart_sshd
```

- Since we created a handler in the taskbook to notify and restart ssh service, we will need to create a handler directory for this under `roles/base/handlers`

```
jonathan@dockerhost-01:~/ansible_tutorial/roles/base/handlers$ cat main.yml
- name: restart_sshd
  service:
    name: sshd
    state: restarted
```

5. Finally, we can run the main playbook to verify everything works as expected. 

- This is a snippet of the output. But you can see that everything ran normally and the ssh templates plays ran successfully

```
jonathan@dockerhost-01:~/ansible_tutorial$ ansible-playbook site.yml

PLAY [all] ***********************************************************************************************************

TASK [base : generate sshd_config file from template] ****************************************************************
changed: [192.168.1.74]
changed: [192.168.1.75]
changed: [192.168.1.76]
changed: [192.168.1.77]
changed: [192.168.1.71]

RUNNING HANDLER [base : restart_sshd] ********************************************************************************
changed: [192.168.1.76]
changed: [192.168.1.74]
changed: [192.168.1.71]
changed: [192.168.1.77]
changed: [192.168.1.75]

PLAY RECAP ***********************************************************************************************************
192.168.1.71               : ok=9    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
192.168.1.74               : ok=10   changed=2    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
192.168.1.75               : ok=8    changed=2    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
192.168.1.76               : ok=8    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
192.168.1.77               : ok=11   changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0

```
