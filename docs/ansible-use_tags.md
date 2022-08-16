## How to use Tags

https://docs.ansible.com/ansible/latest/user_guide/playbooks_tags.html tags in large p

1. You can uselaybooks to create searchable metadata for plays. Then run parts of the playbook (instead of entire playbook) based on tags

Example: 

```
- hosts: web_servers
  become: true
  tasks:

    # This will update apt repo cache and then install apache2 package. This will try to avoid stale apt cache
  - name: install apache2 and php packages
    tags: apache,apache2,ubuntu
    apt:
      name:
        - apache2
        - libapache2-mod-php
      state: latest
    when: ansible_distribution == "Ubuntu"
```

2. You can list tags in a playbook with the following command

```
ansible-playbook --list-tags <playbook>.yml
```

Example:

```
jonathan@dockerhost-01:~/ansible_tutorial$ ansible-playbook --list-tags site.yml

playbook: site.yml

  play #1 (all): all    TAGS: []
      TASK TAGS: [always]

  play #2 (web_servers): web_servers    TAGS: []
      TASK TAGS: [almalinux, apache, apache2, httpd, ubuntu]

  play #3 (db_servers): db_servers      TAGS: []
      TASK TAGS: [almalinux, db, mariadb, ubuntu]

  play #4 (file_servers): file_servers  TAGS: []
      TASK TAGS: [samba]
```

3. To run the ansible playbook using tags, you can run the following.

```
ansible-playbook --tags <nameoftag> --ask-become-pass <playbookname>.yml
```

Example:

```
jonathan@dockerhost-01:~/ansible_tutorial$ ansible-playbook --tags almalinux --ask-become-pass site.yml
BECOME password:

PLAY [all] ***********************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************ok: [192.168.1.76]
ok: [192.168.1.74]
ok: [192.168.1.75]
ok: [192.168.1.77]

TASK [install updates (AlmaLinux)] ***********************************************************************************skipping: [192.168.1.74]
skipping: [192.168.1.75]
skipping: [192.168.1.76]
ok: [192.168.1.77]

TASK [install updates (Ubuntu)] **************************************************************************************skipping: [192.168.1.77]
ok: [192.168.1.76]
ok: [192.168.1.75]
ok: [192.168.1.74]

PLAY [web_servers] ***************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************ok: [192.168.1.74]
ok: [192.168.1.77]

TASK [install apache and php packages] *******************************************************************************skipping: [192.168.1.74]
ok: [192.168.1.77]

PLAY [db_servers] ****************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************ok: [192.168.1.75]

TASK [install mariadb package (AlmaLinux)] ***************************************************************************skipping: [192.168.1.75]

PLAY [file_servers] **************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************ok: [192.168.1.76]

PLAY RECAP ***************************************************************************192.168.1.74      ********************************         : ok=3    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
192.168.1.75               : ok=3    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
192.168.1.76               : ok=3    changed=0    unreachabl   skipped=1    e=0    failed=0 rescued=0    ignored=0
192.168.1.77               : ok=4    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
```

4. If you want to run multiple tags you can run it like this:


```
ansible-playbook --tags "<nameoftag1>,<nameoftag2>" --ask-become-pass <playbookname>.yml
```

NOTE: Inside of the double quotes, do NOT put a space after the commas between the tags.

Example:

```
jonathan@dockerhost-01:~/ansible_tutorial$ ansible-playbook --tags "apache,db" --ask-become-pass site.yml
BECOME password:

PLAY [all] ***********************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************ok: [192.168.1.75]
ok: [192.168.1.74]
ok: [192.168.1.76]
ok: [192.168.1.77]

TASK [install updates (AlmaLinux)] ***********************************************************************************skipping: [192.168.1.74]
skipping: [192.168.1. [192.168.1.77]
76]
skipping: [192.168.1.75]
ok:
TASK [install updates (Ubuntu)] **************************************************************************************skipping: [192.168.1.77]
ok: [192.168.1.74]
changed: [192.168.1.76]
changed: [192.168.1.75]

PLAY [web_servers] ***************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************ok: [192.168.1.74]
ok: [192.16nstall apache2 a8.1.77]

TASK [ind php packages] ******************************************************************************skipping: [192.168.1.77]
ok: [192.168.1.74]

TASK [install apache and php packages] *******************************************************************************skipping: [192.168.1.74]
ok: [192.168.1.77]

PLAY [db_servers] ****************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************ok: [192.168.1.75]

TASK [install mariadb package (AlmaLinux)] ***************************************************************************skipping: [192.168.1.75]

TASK [install mariadb package (Ubuntu)] ******************************************************************************ok: [192.168.1.75]

PLAY [file_servers] **************************************************************************************************

TASK [Gatheri****************ng Facts] *******************************************************************************ok: [192.168.1.76]

PLAY RECAP ***********************************************************************************************************192.168.1.74               : ok=4    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
192.168.1.75               : ok=4    changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
192.168.1.76               : ok=3    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
192.168.1.77               : ok=4    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
```
