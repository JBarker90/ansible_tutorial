# Managing Services

### 1. With some services they sometimes need to be started manually after installing them. So this will go over ways we can manage services with a playbook.

### 2. This play will set the service state to "started". 

```
  - name: start httpd (AlmaLinux)
    tags: apache,almalinux,httpd
    service:
      name: httpd
      state: started
    when: ansible_distribution == "AlmaLinux"
```

- To summarize, this play can be add after the first play to install Apache and the it will run after installed to start the service

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

    # This section will update repo cache then install apache for AlmaLinux
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
    when: ansible_distribution == "AlmaLinux"
```

Proof of Concept: I manually disabled Apache on the AlmaLinux box because it was previously active. But we can see that the service was inactive and then after running the playbook the service was restarted. 

- Step 1: Check status

```
[jonathan@ansiblelab-04 ~]$ sudo systemctl stop httpd
[jonathan@ansiblelab-04 ~]$ sudo systemctl status httpd
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
  Drop-In: /usr/lib/systemd/system/httpd.service.d
           └─php-fpm.conf
           /run/systemd/system/httpd.service.d
           └─zzz-lxc-service.conf
   Active: inactive (dead)
     Docs: man:httpd.service(8)
```

- Step 2: Run playbook

```
ansible-playbook --ask-become-pass site.yml
```

Example: I pulled some of the output of the play to show changes.

```
TASK [start httpd (AlmaLinux)] ***************************************************************************************
skipping: [192.168.1.74]
changed: [192.168.1.77]

PLAY RECAP ***********************************************************************************************************
192.168.1.71               : ok=5    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
192.168.1.74               : ok=5    changed=0    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0
192.168.1.75               : ok=4    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
192.168.1.76               : ok=4    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
192.168.1.77               : ok=6    changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
```

- Step 3: Check Apache service status

```
[jonathan@ansiblelab-04 ~]$ sudo systemctl status httpd
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
  Drop-In: /usr/lib/systemd/system/httpd.service.d
           └─php-fpm.conf
           /run/systemd/system/httpd.service.d
           └─zzz-lxc-service.conf
   Active: active (running) since Wed 2022-08-17 15:44:07 UTC; 45s ago
     Docs: man:httpd.service(8)
 Main PID: 18642 (httpd)
   Status: "Running, listening on: port 80"
    Tasks: 213 (limit: 202889)
   Memory: 19.5M
   CGroup: /system.slice/httpd.service
           ├─18642 /usr/sbin/httpd -DFOREGROUND
           ├─18663 /usr/sbin/httpd -DFOREGROUND
           ├─18730 /usr/sbin/httpd -DFOREGROUND
           ├─18731 /usr/sbin/httpd -DFOREGROUND
           └─18732 /usr/sbin/httpd -DFOREGROUND
```
### 3. Now, there is one more issue with the Apache service. From the output it is showing that the service is "disabled". So the play will install apache package and then start the service. However, if this server were to be rebooted for whatever reason the service will not automatically start. 

- We can fix the play by adding "enabled: yes" to the current play

```
  - name: start httpd (AlmaLinux)
    tags: apache,almalinux,httpd
    service:
      name: httpd
      state: started
      enabled: yes
    when: ansible_distribution == "AlmaLinux"
```

Proof of Concept: After running the playbook we can see that it worked.

- This is a summarized version of output

```
jonathan@dockerhost-01:~/ansible_tutorial$ ansible-playbook --ask-become-pass site.yml
BECOME password:

TASK [start httpd (AlmaLinux)] ***************************************************************************************
skipping: [192.168.1.74]
changed: [192.168.1.77]

PLAY RECAP ***********************************************************************************************************
192.168.1.71               : ok=5    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
192.168.1.74               : ok=5    changed=0    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0
192.168.1.75               : ok=4    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
192.168.1.76               : ok=4    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
192.168.1.77               : ok=6    changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
```

- Checked service status. Now it showed enabled in the loaded field

```
[jonathan@ansiblelab-04 ~]$ sudo systemctl status httpd
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
  Drop-In: /usr/lib/systemd/system/httpd.service.d
           └─php-fpm.conf
           /run/systemd/system/httpd.service.d
           └─zzz-lxc-service.conf
   Active: active (running) since Wed 2022-08-17 15:51:13 UTC; 14min ago
     Docs: man:httpd.service(8)
 Main PID: 19826 (httpd)
   Status: "Total requests: 2; Idle/Busy workers 100/0;Requests/sec: 0.0023; Bytes served/sec:   1 B/sec"
    Tasks: 278 (limit: 202889)
   Memory: 20.7M
   CGroup: /system.slice/httpd.service
           ├─19826 /usr/sbin/httpd -DFOREGROUND
           ├─19827 /usr/sbin/httpd -DFOREGROUND
           ├─19828 /usr/sbin/httpd -DFOREGROUND
           ├─19829 /usr/sbin/httpd -DFOREGROUND
           ├─19830 /usr/sbin/httpd -DFOREGROUND
           └─20265 /usr/sbin/httpd -DFOREGROUND
```

### 4. Now, we can also restart services using Ansible after a change or update has been made to service. Typically, this will need to happen after a modification to a configuration or some other change. 

- This new play will use the `lineinfile` module which will allow us to change a line in whichever file.

```
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
```

Example: We can see after running the playbook the change was made and if we cat the `httpd.conf` file it is now showing the email somebody@somewhere.net

```
TASK [change e-mail address for admin] *******************************************************************************
skipping: [192.168.1.74]
changed: [192.168.1.77]

TASK [restart httpd (AlmaLinux)] *************************************************************************************
skipping: [192.168.1.74]
changed: [192.168.1.77]

PLAY RECAP ***********************************************************************************************************
192.168.1.71               : ok=5    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
192.168.1.74               : ok=5    changed=0    unreachable=0    failed=0    skipped=5    rescued=0    ignored=0
192.168.1.75               : ok=4    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
192.168.1.76               : ok=4    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
192.168.1.77               : ok=8    changed=2    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
```

```
[jonathan@ansiblelab-04 ~]$ cat /etc/httpd/conf/httpd.conf | grep 'ServerAdmin'
# ServerAdmin: Your address, where problems with the server should be
ServerAdmin somebody@somewhere.net
```

NOTE: When using the `lineinfile` module you may have duplicate changes if there are any spelling errors or typos. It's very common. However, you can simply run the playbook again and if there were no changes made the second time then it should be good to go. 

### 5. If you are making multiple changes to a file one after another, you can run into problems if you use the same register variable. 

- If you have two plays that make different changes but both are using the same `register: <variable>` variable then the second modification may never run. 
- Or other things will not run properly.  The reason is because the first variable will be stored as "changed" but when it gets to the second it will see the variable as already changed not updates necessary.

Example:

```
  - name: second change
    tags: apache,almalinux,httpd
    lineinfile:
      path: /etc/httpd/conf/httpd.conf
      regexp: '^ServerAdmin'
      line: ServerAdmin somebody@somewhere.net
    when: ansible_distribution == "AlmaLinux"
    register: httpd
	
	  - name: change e-mail address for admin
    tags: apache,almalinux,httpd
    lineinfile:
      path: /etc/httpd/conf/httpd.conf
      regexp: '^ServerAdmin'
      line: ServerAdmin somebody@somewhere.net
    when: ansible_distribution == "AlmaLinux"
    register: httpd
```