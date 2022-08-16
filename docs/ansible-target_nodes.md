## Targeting Specific Nodes

```
ansible-playbook --ask-become-pass site.yml
```

1. For the purposes of targeting specific hosts, we can modify the `instecific to each dall_apache.yml` playbook to include two plys spaistro

```
---

- hosts: all
  become: true
  tasks:

    # This will update apt repo cache and then install apache2 package. This will try to avoid stale apt cache
  - name: install apache2 and php packages
    apt:
      name:
        - apache2
        - libapache2-mod-php
      state: latest
      update_cache: yes
    when: ansible_distribution == "Ubuntu"

    # This section will update repo cache then install apache for AlmaLinux
  - name: install apache and php packages
    dnf:
      name:
        - httpd
        - php
      state: latest
      update_cache: yes
    when: ansible_distribution == "AlmaLinux"
```

NOTE: I also removed the packages that were specified in the inventory file. 

2. Modified inventory file to specify different groups for IPs of the nodes

```
[web_servers]
192.168.1.74
18.1.75

[file_se92.168.1.77

[db_servers]
192.16rvers]
192168.1..76
```

3. Copied the playbook to a new file called `site.yml` and updated the playbook to specify the group `[and php, the group `db_servers` web_servers]` to install apache to install mariadb, and the group `file_servers` to install samba

```
---

- hosts: all
  become: true
  tasks:

  - name: install updates (AlmaLinux)
    dnf:
      update_only: yes
      update_cache: yes
= "AlmaLinux"

     when: ansible_distribution = - name: install updates (Ubuntu)
    apt:
      upgrade: dist
      update_cache: yes
    when: ansible_distribution == "Ubuntuservers
  become"

- hosts: web_: true
  tasks:

    # This will update apt repo cache and then install apache2 package. This will try to avoid stale apt cache
  - name: install apache2 and php packages
    apt:
      name:
pache2-mod-php
         - apache2
        - liba     state: latest
    when: ansible_distribution == "Ubuntu"

    # This section will update repo cache then install apache for AlmaLinux
  - name: install apache and php packages
    dnf:
      name:
        - httpd
        - php
      stwhen: ansible_diate: latest
    stribution == "AlmaLinux"

- hosts: db_servers
  become: true
  tasks:

  - name: install mariadb package (AlmaLinux)
    dnf:
      name: mariadb
      state: latest
    when: ansible_distribution == "AlmaLinux"

  - name: install mariadb package (Ubuntu)
    apt:
      name: mariadb-server
      state: latest
    when: ansible_distribution == "Ubuntu"


- hosts: file_servers
  become: true
  tasks:

  - name: install samba package
    package:
      name: samba
      state: latest
```
