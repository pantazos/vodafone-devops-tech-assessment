# vodafone-devops-tech-assessment
Vodafone DevOps Tech Assessment

**Step 1:**

*Install the Apache package*

```bash
yum install -y httpd
```

```bash
firewall-cmd –permanent –add-service=http
```

```bash
firewall-cmd –reload
```

```bash
systemctl enable httpd
```

```bash
systemctl start httpd
```

**Step 2:**

*Extend the existing xfs file system to a total size of 200MB and add a label called myFS.*

```bash
lvextend –size 200M -r /dev/vg/lv_xfs
```

```bash
umount /xfs
```

```bash
xfs_admin -L “myFS” /dev/vg/lv_xfs
```

```bash
mount /xfs
```

**Step 3:**

*Create two users: john with uid/gid equal to 2000, password 12345678 and davis with uid/gid equal to 3000, password 87654321. Make davis‘ account validity stopping in one month.*

```bash
useradd -u 2000 john
```

```bash
passwd john
New password: 12345678
```

```bash
useradd -u 3000 davis
```

```bash
passwd davis
New password: 87654321
```

```bash
date -d “+1month”
```

```bash
usermod -e YYYY-MM-DD davis
```

```bash
chage -l davis
```

**Step 4:**

*Allow davis (and only davis) to get full access to john‘s home directory.*

```bash
setfacl -R -m u:davis:rwx /home/john
```

**Step 5:**

*Create a directory named /common. Allow john and davis to share documents in the /common directory using a group called team. Both of them can read, write and remove documents from the other in this directory but any user not member of the group can’t.*

```bash
mkdir /common
```

```bash
groupadd -g 50000 team
```

```bash
chgrp team /common
```

```bash
chmod 2770 /common
```

```bash
usermod -aG team john
```

```bash
usermod -aG team davis
```

**Step 6:**

*Validate the SELinux status and configure it temporarily to Permissive if not and make SElinux status permanent across reboot to Permissive.*

Edit **/etc/selinux/config**
Change the SELINUX value to “SELINUX=permissive”

```bash
# cat /etc/selinux/config
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=permissive
# SELINUXTYPE= can take one of these two values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

Reboot the server.

```bash
shutdown -r now
```

**Step 7:**

*Create a xfs file system on a new logical volume of 100MB called lv_xfs. Mount it permanently with uuid under /xfs.*

```bash
lvcreate –size 100M –name lv_xfs /dev/vg
```

```bash
mkfs.xfs /dev/vg/lv_xfs
```

```bash
mkdir /xfs
```

```bash
blkid | grep lv_xfs >> /etc/fstab
```

```bash
vi /etc/fstab
```

```UUID=… /xfs xfs defaults 1 2```

```bash
mount -a
```

**Step 8:**

*Create a logical volume of 200MB called lv_swap2 and add it permanently to the current swap space.*

```bash
lvcreate –size 200M –name lv_swap2 /dev/vg
```

```bash
mkswap /dev/vg/lv_swap2
```

```bash
swapon /dev/vg/lv_swap2
```

```vi /etc/fstab```

```/dev/vg/lv_swap2 swap swap defaults 0 0```

**Step 9:**

*Create a cron job running as root, starting at 11PM every day and writing a report on daily system resource consumption in the /var/log/consumption.log file.*

```bash
crontab -e
```

```00 23 * * * /usr/bin/sar -A > /var/log/consumption.log```

**Step 10:**

*Set the default target to boot into X Window level (previously level 5).*

```bash
systemctl set-default graphical.target
```

**Step 11:**

*Change the hostname to example.vodafone.com*

```bash
hostnamectl set-hostname example.vodafone.com
```

**Step 12:**

*Synchronize time with NTP source pool.ntp.org*

```bash
sudo ntpdate 1.ro.pool.ntp.org
```

```bash
sudo ntpdate -qu 1.ro.pool.ntp.org
```

```bash
sudo nano /etc/systemd/timesyncd.conf
```

```bash
sudo timedatectl set-ntp true 
timedatectl status
```

Note: OS may affect specifically minor problems.

**Step 13:**

*Install NFS and export directory /nfs/exports to client 192.168.1.10*

```bash
driftfile /var/lib/ntp/ntp.drift

server 0.pool.ntp.org
server 1.pool.ntp.org
server 2.pool.ntp.org
server 3.pool.ntp.org
```

**Step 14:**

*Create an Ansible playbook to install nginx and configure home page to a custom index.html page.*

Install Nginx on Ubuntu server

Create an Ansible Playbook with YAML file:

```nginx_install.yml```

```yml
- hosts: all
  tasks:
    - name: ensure nginx is at the latest version
      apt: name=nginx state=latest
    - name: start nginx
      service:
          name: nginx
          state: started
```

```
$ ansible-playbook -i inventory.cfg nginx_install.yml -b
```

**Configure nginx**

*simple_site.cfg*

```cfg
server {
        listen 80 default_server;
        listen [::]:80 default_server;
        root /home/foo/static-site;
        server_name _;
        location / {
                try_files $uri $uri/ =404;
        }
}
```

Edit *nginx.yml*

```yml
---
- hosts: all
  tasks:
    - name: ensure nginx is at the latest version
      apt: name=nginx state=latest
      become: yes
    - name: start nginx
      service:
          name: nginx
          state: started
      become: yes
    - name: copy the nginx config file and restart nginx
      copy:
        src: /home/foo/static_site.cfg
        dest: /etc/nginx/sites-available/static_site.cfg
      become: yes
    - name: create symlink
      file:
        src: /etc/nginx/sites-available/static_site.cfg
        dest: /etc/nginx/sites-enabled/default
        state: link
      become: yes
    - name: copy the content of the web site
      copy:
        src: /home/foo/static-site-src/
        dest: /home/foo/static-site
    - name: restart nginx
      service:
        name: nginx
        state: restarted
      become: yes
```

```bash
$ ansible-playbook -i inventory.cfg  --limit 192.168.56.11 nginx.yml
```

*Check the IP address that is already configured from DevOps team.*

> READY 
