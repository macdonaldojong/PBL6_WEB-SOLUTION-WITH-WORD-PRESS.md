## WEB SOLUTION WITH WORDPRESS
### Setup an EC2 Instance that serve as the web server
###  Create 3 volumes in the same AZ as the Web Server EC2 where each of them is 10 GB.
### lsblk inspect and list what block devices are attached to the server
```
lsblk
```
### To list all mounts and free space on your Server
```
df -h
```
### To create a new partition on each of the 3 disks
```
sudo gdisk /dev/xvdf
sudo gdisk /dev/xvdg
sudo gdisk /dev/xvdh
```
```
 GPT fdisk (gdisk) version 1.0.3

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: not present

Creating new GPT entries.

Command (? for help branch segun-edits: p
Disk /dev/xvdf: 20971520 sectors, 10.0 GiB
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): D936A35E-CE80-41A1-B87E-54D2044D160B
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 20971486
Partitions will be aligned on 2048-sector boundaries
Total free space is 2014 sectors (1007.0 KiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048        20971486   10.0 GiB    8E00  Linux LVM

Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): yes
OK; writing new GUID partition table (GPT) to /dev/xvdf.
The operation has completed successfully.
```
### The operation is repeated for the other disks xvdg and xvdh
### Install lvm2 to check for available partitions.

```
sudo yum install lvm2
```

### Run pvcreate to mark each of 3 disks as physical volumes (PVs) to be used by LVM
```
sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdg1
sudo pvcreate /dev/xvdh1
```
### To verify that the Physical volume has been created successfully
```
sudo pvs
```
### add all 3 PVs to a volume group (VG) while Naming the VG vg-webdata
```
sudo vgcreate vg-webdata /dev/xvdh1 /dev/xvdg1 /dev/xvdf1
```
### To verify that your VG has been created successfully

```
sudo vgs
```

### To work with lvcreate utility to create 2 logical volumes. apps-lv (where I worked with half of the PV size), and logs-lv while Using the remaining space of the PV size. 
### apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs

```
sudo lvcreate -n apps-lv -L 14G vg-webdata
sudo lvcreate -n logs-lv -L 14G vg-webdata
```

### To Verify that your Logical Volume has been created successfully
```
sudo lvs
```

### Verify the entire setup
```
sudo vgdisplay -v #view complete setup - VG, PV, and LV
sudo lsblk 
```

### To Use mkfs.ext4 to format the logical volumes with ext4 filesystem
```
sudo mkfs -t ext4 /dev/vg-webdata/apps-lv
sudo mkfs -t ext4 /dev/vg-webdata/logs-lv
```

### Create /var/www/html directory to store website files
```
sudo mkdir -p /var/www/html
```

### To Create /home/recovery/logs to store backup of log data
```
sudo mkdir -p /home/recovery/logs
```

### Mount /var/www/html on apps-lv logical volume
```
sudo mount /dev/webdata-vg/apps-lv /var/www/html/
```
### To backup all the files in the log directory /var/log into /home/recovery/logs we use the rsync utility
```
sudo rsync -av /var/log/. /home/recovery/logs/
```
### Mount /var/log on logs-lv logical volume
```
sudo mount /dev/webdata-vg/logs-lv /var/log
```

### Restore log files back into /var/log directory
```
sudo rsync -av /home/recovery/logs/log/. /var/log
```

### Update /etc/fstab file so that the mount configuration will persist after restart of the server where the UUID of the device will be used to update the /etc/fstab file
```
sudo blkid
sudo vi /etc/fstab
```
### Update /etc/fstab

### Test the configuration and reload the daemon
```
sudo mount -a
sudo systemctl daemon-reload
df -h
```

### Launch a second EC2 Instance as a Database Server and create 3 volumes in the same AZ as the Web Server EC2 where each of them is 10 GB.
### create db-lv and mount it to /db directory
### Install WordPress on Web Server EC2

### Update the repository
```
sudo yum -y update
```
### Install wget, Apache and it’s dependencies
```
sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json
```
### Start Apache
```
sudo systemctl enable httpd
sudo systemctl start httpd
```
```
sudo systemctl enable httpd
sudo systemctl start httpd
```
### To install PHP and it’s depemdencies
```
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
setsebool -P httpd_execmem 1
Restart Apache

sudo systemctl restart httpd
```
### Download wordpress and copy wordpress to var/www/html
```
  mkdir wordpress
  cd   wordpress
  sudo wget http://wordpress.org/latest.tar.gz
  sudo tar xzvf latest.tar.gz
  sudo rm -rf latest.tar.gz
  cp wordpress/wp-config-sample.php wordpress/wp-config.php
  cp -R wordpress /var/www/html/
  ```
### Configure SELinux Policies
```
  sudo chown -R apache:apache /var/www/html/wordpress
  sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
  sudo setsebool -P httpd_can_network_connect=1
```
### Install MySQL on the DB Server EC2
```
sudo yum update
sudo yum install mysql-server
```
### To Verify that the service is up and running by using sudo systemctl status mysqld
```
sudo systemctl restart mysqld
sudo systemctl enable mysqld
```
### Configure DB to work with WordPress
```
sudo mysql
CREATE DATABASE wordpress;
CREATE USER `myuser`@`172.31.39.41` IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'172.31.39.41';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit
```
### Configure WordPress to connect to remote database while ensuring that we open MySQL port 3306 on DB Server EC2.
### Install MySQL client and test that connectiveity from Web Server to DB server by using mysql-client
```
sudo yum install mysql
sudo mysql -u admin -p -h 172.31.39.40
```

### Change permissions and configuration so Apache could use WordPress:

### Enable TCP port 80 in Inbound Rules configuration for Web Server EC2 (enable from everywhere 0.0.0.0/0)

Try to access from your browser the link to your WordPress http://3.17.204.32/wordpress/
![image](https://user-images.githubusercontent.com/41236641/127143778-5166df95-d1bf-4bd5-8a57-b72e502dcfcb.png)
