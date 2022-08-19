## Managing Files

- https://docs.ansible.com/ansible/2.9/modules/copy_module.html

1. This will go over the concept of copying a file or files to a server using Ansible. So what we will do is use Ansible playbook to copy files and sync it to a specific place on one of nodes

2. I created a new directory called `/files` and created a simple html file inside called `default_site.html` 

```
<html>
    <title>Web-site test</title>

    <body>
        <p> Ansible is awesome!</p>
    </body>
</html>
```


3. Then we will add a play in the playbook `site.yml` to copy the file to the servers.

- Under the plays for `web_servers` we will add a new play that will use the `copy` module to copy the `default_site.html` file I created to the destination on the servers in the path `/var/www/html/index.html` 

```
  - name: copy default html file for site
    tags: apache,apache2,httpd
    copy:
      src: default_site.html
      dest: /var/www/html/index.html
      owner: root
      group: root
      mode: 0644
```

Example: This is the output when running the ansible playbook without any tags

```
jonathan@dockerhost-01:~/ansible_tutorial$ ansible-playbook --ask-become-pass site.yml
BECOME password:

PLAY [all] ***********************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************ok: [192.168.1.74]
ok: [192.168.1.77]
ok: [192.168.1.75]
ok: [192.168.1.76]

TASK [install updates (AlmaLinux)] ***********************************************************************************skipping: [192.168.1.74]
skipping: [192.168.1.75]
skipping: [192.168.1.76]
ok: [192.168.1.77]

TASK [install updates (Ubuntu)] **************************************************************************************skipping: [192.168.1.77]
ok: [192.168.1.74]
ok: [192.168.1.75]
ok: [192.168.1.76]

PLAY [web_servers] ***************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************ok: [192.168.1.74]
ok: [192.168.1.77]

TASK [install apache2 and php packages] ******************************************************************************skipping: [192.168.1.77]
ok: [192.168.1.74]

TASK [install apache and php packages] *******************************************************************************skipping: [192.168.1.74]
ok: [192.168.1.77]

TASK [copy default html file for site] *******************************************************************************changed: [192.168.1.74]
changed: [192.168.1.77]

PLAY [db_servers] ****************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************ok: [192.168.1.75]

TASK [install mariadb package (AlmaLinux)] ***************************************************************************skipping: [192.168.1.75]

TASK [install mariadb package (Ubuntu)] ******************************************************************************ok: [192.168.1.75]

PLAY [file_servers] **************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************ok: [192.168.1.76]

TASK [install samba package] *****************************************************************************************ok: [192.168.1.76]

PLAY RECAP ***********************************************************************************************************192.168.1.74               : ok=5    changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
192.168.1.75               : ok=4    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
192.168.1.76               : ok=4    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
192.168.1.77               : ok=5    changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
```

Proof of concept: The file has now been copied to one of the Ansible lab servers in the `web_servers` group.

```
jonathan@ansiblelab-01:~$ cat /var/www/html/index.html
<html>
    <title>Web-site test</title>

    <body>
        <p> Ansible is awesome!</p>
    </body>
</html>

jonathan@ansiblelab-01:~$ stat /var/www/html/index.html
  File: /var/www/html/index.html
  Size: 101             Blocks: 1          IO Block: 512    regular file
Device: 63h/99d Inode: 45842       Links: 1
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2022-08-13 16:58:00.622018719 +0000
Modify: 2022-08-13 16:37:30.164778438 +0000
Change: 2022-08-13 16:37:30.372780457 +0000
 Birth: 2022-08-13 16:37:30.164778438 +0000
```

4. We can also use Ansible to configure and install packages to our local workstation or control host. 

- This play creates a new `[workstations]` group that will install and unzip Terraform to the local dockerhost server I'm using to run plays. The following was added to `site.yml`: 

```
- hosts: workstations
  become: true
  tasks:

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

- Then, we will add the local IP to the `inventory` file and create a `[workstations]` group

```
[workstations]
192.168.1.71
```

- Then we can copy the SSH ID for the ansible SSH pub key to our local authorized hosts 

```
ssh-copy-id -i ~/.ssh/ansible_key.pub 192.168.1.71
```

- After running the playbook `ansible-playbook --ask-become-pass site.yml` we can see that Terraform is now installed locally

```
jonathan@dockerhost-01:~/ansible_tutorial$ which terraform
/usr/local/bin/terraform
```
