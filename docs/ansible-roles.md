## Ansible Roles

- https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html

1. The concept of roles will allow us to split up tasks in a way that makes more sense.

2. Using roles we can clean up the playbook quite a bit. We made a copy of the previous `site.yml` and then reconfigured `site.yml` to use the roles below

```
---

- hosts: all
  become: true
  pre_tasks:

  - name: update repo cache (AlmaLinux)
    tags: always
    dnf:
      update_cache: yes
    changed_when: false
    when: ansible_distribution == "AlmaLinux"

  - name: update repo cache (Ubuntu)
    tags: always
    apt:
      update_cache: yes
    changed_when: false
    when: ansible_distribution == "Ubuntu"

- hosts: all
  become: true
  roles:
    - base

- hosts: workstations
  become: true
  roles:
    - workstations

- hosts: web_servers
  become: true
  roles:
    - web_servers

- hosts: db_servers
  become: true
  roles:
    - db_servers

- hosts: file_servers
  become: true
  roles:
    - file_servers
```

NOTE: This is will fail because we haven't created the roles yet. 

2. In order to create these roles, we will need to create a new directory called `/roles` and then create a directory for each role. 

```
jonathan@dockerhost-01:~/ansible_tutorial/roles$ ll
total 27
drwxrwxr-x 7 jonathan jonathan  7 Sep  3 15:22 ./
drwxrwxr-x 6 jonathan jonathan 16 Sep  3 15:21 ../
drwxrwxr-x 2 jonathan jonathan  2 Sep  3 15:22 base/
drwxrwxr-x 2 jonathan jonathan  2 Sep  3 15:22 db_servers/
drwxrwxr-x 2 jonathan jonathan  2 Sep  3 15:22 file_servers/
drwxrwxr-x 2 jonathan jonathan  2 Sep  3 15:22 web_servers/
drwxrwxr-x 2 jonathan jonathan  2 Sep  3 15:22 workstations/
```

- Then, under each role directory create a new directory for `tasks`

```
jonathan@dockerhost-01:~/ansible_tutorial/roles$ mkdir base/tasks
jonathan@dockerhost-01:~/ansible_tutorial/roles$ mkdir db_servers/tasks
jonathan@dockerhost-01:~/ansible_tutorial/roles$ mkdir file_servers/tasks
jonathan@dockerhost-01:~/ansible_tutorial/roles$ mkdir web_servers/tasks
jonathan@dockerhost-01:~/ansible_tutorial/roles$ mkdir workstations/tasks
```

3. Now, in each `tasks` directory this will be where we will create our playbooks or task book. These will not be complete playbooks because they will only include tasks.

- Under the tasks directory we will create a `main.yml` file

```
jonathan@dockerhost-01:~/ansible_tutorial/roles$ tree
.
|-- base
|   `-- tasks
|       `-- main.yml
```

- Then, in the `main.yml` file we will add the task to add ssh key for our automated user Dave

```
- name: add ssh key for dave
  authorized_key:
    user: dave
    key: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGAhvL68zMlGEqZtBBiJskMpRgXRMX6N7/6BfWxZ8s7Q Ansible keys for Docker host"
```

- Then for `roles/db_servers/tasks` we create another `main.yml` file that will contain the tasks for installing packages for `db_servers`

```
- name: install mariadb package (AlmaLinux)
  tags: almalinux,db,mariadb
  dnf:
    name: mariadb
    state: latest
  when: ansible_distribution == "AlmaLinux"

- name: install mariadb package (Ubuntu)
  tags: ubuntu,db,mariadb
  apt: 
    name: mariadb-server
    state: latest
  when: ansible_distribution == "Ubuntu"
```

- Then, we will do the same with `roles/file_servers/tasks` and create a `main.yml` for installing samba

```
- name: install samba package
  tags: samba
  package: 
    name: samba
    state: latest
```

- Then, we will do the same for a `main.yml` for `web_servers` role.

```
- name: install apache2 and php packages
  tags: apache,apache2,ubuntu
  apt:
    name:
      - apache2
      - libapache2-mod-php
    state: latest
  when: ansible_distribution == "Ubuntu"

- name: install apache and php packages
  tags: apache,almalinux,httpd
  dnf:
    name:
      - httpd
      - php
    state: latest
  when: ansible_distribution == "AlmaLinux"

- name: start httpd (AlmaLinux)
  tags: apache,almalinux,httpd
  service:
    name: httpd
    state: started
    enabled: yes
  when: ansible_distribution== "AlmaLinux"

- name: change e-mail address for admin
  tags: apache,almalinux,httpd
  lineinfile:
    path: /etc/httpd/conf/httpd.conf
    regexp: '^ServerAdmin'
    line: ServerAdmin somebody@somewhere.net
  when: ansible_distribution == "AlmaLinux"
  register: httpd

- name: restart httpd (AlmaLinux)
  tags: apache,almalinux,httpd
  service:
    name: httpd
    state: restarted
  when: httpd.changed

- name: copy default html file for site
  tags: apache,apache2,httpd
  copy:
    src: default_site.html
    dest: /var/www/html/index.html
    owner: root
    group: root
    mode: 0644
```

- Then, we will do the same for a `main.yml` for `workstations` role.

```
- name: install unzip
  package:
    name: unzip

- name: install terraform
  unarchive:
    src: https://releases.hashicorp.com/terraform/0.12.28/terraform_0.12.28_linux_amd64.zip
    dest: /usr/local/bin
    remote_src: yes
    mode: 0755
    owner: root
    group: root
```

4. At this point the play will still fail, we need to create `files` directory located in `/roles/web_servers/files` so the `web_servers` roles has a place to copy source files from. 

NOTE: This is only needed if we are copying files from a source location to the destination. 

- I copied the `default_site.html` file from the earlier location to this `web_servers` role under the `files` directory

```
jonathan@dockerhost-01:~/ansible_tutorial/roles/web_servers$ cp -av ~/ansible_tutorial/files/default_site.html files/ '/home/jonathan/ansible_tutorial/files/default_site.html' -> 'files/default_site.html'
```

- When you run this playbook, the `web_servers` role will now copy the `default_site.html` file from our new files directory for the role.

```
jonathan@dockerhost-01:~/ansible_tutorial$ ansible-playbook site.yml

PLAY [all] ***********************************************************************************************************
PLAY [web_servers] ***************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************
ok: [192.168.1.74]
ok: [192.168.1.77]

TASK [web_servers : install apache2 and php packages] ****************************************************************
skipping: [192.168.1.77]
ok: [192.168.1.74]

TASK [web_servers : install apache and php packages] *****************************************************************
skipping: [192.168.1.74]
ok: [192.168.1.77]

TASK [web_servers : start httpd (AlmaLinux)] *************************************************************************
skipping: [192.168.1.74]
ok: [192.168.1.77]

TASK [web_servers : change e-mail address for admin] *****************************************************************
skipping: [192.168.1.74]
ok: [192.168.1.77]

TASK [web_servers : restart httpd (AlmaLinux)] ***********************************************************************
skipping: [192.168.1.74]
skipping: [192.168.1.77]

TASK [web_servers : copy default html file for site] *****************************************************************
ok: [192.168.1.74]
ok: [192.168.1.77]
```
