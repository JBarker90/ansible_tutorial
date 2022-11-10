# Host Variables and Handlers

### 1. We can create a host variables file for each host in the `inventory` file. 

- Under the root directory we can create a new `host_vars` directory to store the files

```
jonathan@dockerhost-01:~/ansible_tutorial$ tree -d .
.
|-- docs
|-- files
|-- host_vars
`-- roles
    |-- base
    |   `-- tasks
    |-- db_servers
    |   `-- tasks
    |-- file_servers
    |   `-- tasks
    |-- web_servers
    |   |-- files
    |   `-- tasks
    `-- workstations
        `-- tasks

15 directories
```

- Then, under the `host_vars` directory we will create a file for each host (from inventory)

NOTE: The host files will need to be named either the IP or the hostname using the `.yml` extension

```
jonathan@dockerhost-01:~/ansible_tutorial/host_vars$ cat 192.168.1.74.yml
apache_package_name: apache2
apache_service: apache2
php_package_name: libapache2-mod-php
```

- The major benefit with using host variables is organizing and cleaning up taskbooks for each role that you create. 

### 2. Using these host variables, you can modify the taskbooks under each role with variables.

- Under `roles/web_servers/tasks/main.yml` you can replace the names with the variables that were created.

```
- name: install apache and php packages
  tags: apache,httpd,php
  package:
    name:
      - "{{ apache_package_name }}"
      - "{{ php_package_name }}"
    state: latest

- name: start and enable apache service
  tags: apache,httpd
  service:
    name: "{{ apache_service }}"
    state: started
    enabled: yes

- name: change e-mail address for admin
  tags: apache,almalinux,httpd
  lineinfile:
    path: /etc/httpd/conf/httpd.conf
    regexp: '^ServerAdmin'
    line: ServerAdmin somebody@somewhere.net
  when: ansible_distribution == "AlmaLinux"
  register: apache

- name: restart httpd
  tags: apache,httpd
  service:
    name: "{{ apache_service }}"
    state: restarted
  when: apache.changed

- name: copy default html file for site
  tags: apache,apache2,httpd
  copy:
    src: default_site.html
    dest: /var/www/html/index.html
    owner: root
    group: root
    mode: 0644
```

### 3. Now we can also use a concept called handlers

- A handler can be used to notify a service of a change more efficiently than our current task. 

- With our current taskbook for `web_servers` we have a task to restart Apache when `apache.changed`. But with a handler we can more efficiently run the task to make sure that the service is actually restarted even when there are multiple changes.

- In `roles/web_servers/tasks/main.yml` we can remove the task to restart Apache when changed and change register (for the task to modify email) to notify

```
- name: install apache and php packages
  tags: apache,httpd,php
  package:
    name:
      - "{{ apache_package_name }}"
      - "{{ php_package_name }}"
    state: latest

- name: start and enable apache service
  tags: apache,httpd
  service:
    name: "{{ apache_service }}"
    state: started
    enabled: yes

- name: change e-mail address for admin
  tags: apache,almalinux,httpd
  lineinfile:
    path: /etc/httpd/conf/httpd.conf
    regexp: '^ServerAdmin'
    line: ServerAdmin somebody@somewhere.net
  when: ansible_distribution == "AlmaLinux"
  notify: restart_apache

- name: copy default html file for site
  tags: apache,apache2,httpd
  copy:
    src: default_site.html
    dest: /var/www/html/index.html
    owner: root
    group: root
    mode: 0644
```

- Then in `roles/web_servers/` we can make a new directory called `handlers` and create a handler for `restart_apache` in `main.yml`

```
jonathan@dockerhost-01:~/ansible_tutorial/roles/web_servers/handlers$ cat main.yml
- name: restart_apache
  service:
    name: "{{ apache_service }}"
    state: restarted
```

- Now just to test that is runs, we can change the email address in the taskbook in `roles/web_servers/tasks/main.yml` from somebody@somewhere.net to somebody@somewhere.com

```
jonathan@dockerhost-01:~/ansible_tutorial$ ansible-playbook site.yml

PLAY [all] ***********************************************************************************************************

TASK [web_servers : install apache and php packages] *****************************************************************
ok: [192.168.1.74]
changed: [192.168.1.77]

TASK [web_servers : start and enable apache service] *****************************************************************
ok: [192.168.1.77]
ok: [192.168.1.74]

TASK [web_servers : change e-mail address for admin] *****************************************************************
skipping: [192.168.1.74]
changed: [192.168.1.77]

TASK [web_servers : copy default html file for site] *****************************************************************
ok: [192.168.1.74]
ok: [192.168.1.77]

RUNNING HANDLER [web_servers : restart_apache] ***********************************************************************
changed: [192.168.1.77]

PLAY RECAP ***********************************************************************************************************
192.168.1.71               : ok=7    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
192.168.1.74               : ok=8    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
192.168.1.75               : ok=6    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
192.168.1.76               : ok=6    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
192.168.1.77               : ok=10   changed=3    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
```

NOTE: This output is a snippet and doesn't include full playbook output