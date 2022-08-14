# Improving Your Playbook

- https://docs.ansible.com/ansible/2.8/modules/package_module.html

```
ansible-playbook --ask-become-pass install_apache.yml
```

### 1. We can consolidate the installation plays a little bit. Instead of having two plays (one to install apache and the other to install php), we can combine the two in a list.

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

  - name: install apache2 and php packages
    apt:
      name:
        - apache2
        - libapache2-mod-php
      state: latest
    when: ansible_distribution == "Ubuntu"
```

### 2. We can do the same consolidation with the AlmaLinux play. 

```
    # This section will update repo cache then install apache for AlmaLinux
  - name: update repository index
    dnf:
      update_cache: yes
    when: ansible_distribution == "AlmaLinux"

  - name: install apache and php packages
    dnf:
      name:
        - httpd
        - php
      state: latest
    when: ansible_distribution == "AlmaLinux"
```

### 3. We can clean this up even more and consolidate it down to 2 plays. Instead of having 1 play to update repo cache and 1 to install packages, you can install packages and update cache in 1 play.

NOTE: I removed the play to update cache and added the line below state saying `update_cache: yes`

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

### 4. We can consolidate it even more down to 1 play in the playbook. 

- Step 1: We can declare variables in the inventory file to specify which php and apache packages.

Example: This is in `~/ansible_tutorial/inventory`
```
# These are internal IPs for Lab servers that Ansible automates

192.168.1.74 apache_package=apache2 php_package=libapache2-mod-php
192.168.1.75 apache_package=apache2 php_package=libapache2-mod-php
192.168.1.76 apache_package=apache2 php_package=libapache2-mod-php
192.168.1.77 apache_package=httpd php_package=php
```

- Step 2: In the `install_apache.yml` file, we can specify general variables for the package names and then change the installer to `package`.

Example: These variables are declared in the inventory in step 1. Also, `package` is a generic installer in ansible that will automatically use appropriate package manager based on distro (apt on Ubuntu and Debian-based, dnf on RHEL and AlmaLinux, pacman on Arch).
```
---

- hosts: all
  become: true
  tasks:

    # This will update apt repo cache and then install apache2 package. This will try to avoid stale apt cache
  - name: install apache and php packages
    package:
      name:
        - "{{ apache_package }}"
        - "{{ php_package }}"
      state: latest
      update_cache: yes
```

