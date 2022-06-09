## WEB SOLUTION WITH WORDPRESS

## our task:
* The task in this project is to design a storage infrastructure on two Linux servers and implement a basic web solution using WordPress. 
* WordPress is a free and open-source content management system written in PHP and paired with MySQL or MariaDB as its backend Relational Database Management System (RDBMS).

### Project consist of implementing a Three-tier Architecture in software development

Three-tier Architecture:
* Generally, web, or mobile solutions are implemented based on what is called the Three-tier Architecture.
* Three-tier Architecture is a client-server software architecture pattern that comprise of 3 separate layers.

![image](https://user-images.githubusercontent.com/58276505/172831496-0c28b6dc-ce5b-49b8-ad0d-740445394f5b.png)

#### 2 Part (I & II) 
* Part I: Configure storage subsystem for Web and Database servers based on Linux OS. The focus of this part is to give you practical experience of working with disks, partitions and volumes in Linux.

* Part II: Install WordPress and connect it to a remote MySQL database server. 

## Project requirement:
* Presentation Layer (PL): This is the user interface such as the client server or browser on your laptop.
* Business Layer (BL): This is the backend program that implements business logic. Application or Webserver
* Data Access or Management Layer (DAL): This is the layer for computer data storage and data access. Database Server or File System Server such as FTP server, or NFS Server

### Your 3-Tier Setup
A Laptop or PC to serve as a client
An EC2 Linux Server as a web server (This is where you will install WordPress)
An EC2 Linux server as a database (DB) server


### Project implementation:
* We will use RedHat’ (it has a fully compatible derivative)
* Launch a Redhat instance and Create 3 volumes in the same AZ as the Web Server EC2 where each of them is 10 GB.

![image](https://user-images.githubusercontent.com/58276505/172836484-14a12909-3eee-4dc6-afb1-cadc23e912ff.png)

![image](https://user-images.githubusercontent.com/58276505/172836674-777ff97e-c73a-4b9f-bca9-97f70ba40be5.png)

* View or list what block devices/volumes are attached to the server

```
lsblk
```

![image](https://user-images.githubusercontent.com/58276505/172836185-8552a564-c29c-481b-99da-86d3262944f9.png)

### To list all mounts and free space on your Server

```
df -h
```
### To create a new partition on each of the 3 disks (The operation is repeated for the other disks xvdg and xvdh)
```
sudo gdisk /dev/xvdf
sudo gdisk /dev/xvdg
sudo gdisk /dev/xvdh
```

sudo gdisk /dev/xvdf
![image](https://user-images.githubusercontent.com/58276505/172837184-0894d495-500e-4fc2-b000-d096c445e965.png)

View disk
lsblk
![image](https://user-images.githubusercontent.com/58276505/172837865-2dec026f-addf-477c-9839-0d066a77a8a1.png)

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

![image](https://user-images.githubusercontent.com/58276505/172838419-d7f320e5-8694-41d4-9c70-d6c8c2aa3600.png)

Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG **webdata-vg**
```
sudo vgcreate vg-webdata /dev/xvdh1 /dev/xvdg1 /dev/xvdf1
```
### To verify that your VG has been created successfully

```
sudo vgs
```

![image](https://user-images.githubusercontent.com/58276505/172838715-92a6d4ae-6807-4545-afc1-02460de48997.png)
Use lvcreate utility to create 2 logical volumes. **apps-lv (Use half of the PV size)**, and **logs-lv Use the remaining space of the PV size**. 
**NOTE:** apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.
```
sudo lvcreate -n apps-lv -L 14G vg-webdata
sudo lvcreate -n logs-lv -L 14G vg-webdata
```

To Verify that your Logical Volume has been created successfully
```
sudo lvs
```

![image](https://user-images.githubusercontent.com/58276505/172839250-0a6b35b4-72e3-423e-9f5c-e7ed3fa58504.png)

### Verify the entire setup
```
sudo vgdisplay -v #view complete setup - VG, PV, and LV
sudo lsblk 
```

![image](https://user-images.githubusercontent.com/58276505/172839353-9c6c77b0-d828-4594-942c-4b298937de56.png)

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
![image](https://user-images.githubusercontent.com/58276505/172839680-e8087173-c33f-4450-8310-d969f30e26b7.png)

![image](https://user-images.githubusercontent.com/58276505/172839824-0e159346-4e46-45e9-baa5-cfc452c2ec0b.png)

### Update /etc/fstab

### Test the configuration and reload the daemon
```
sudo mount -a
sudo systemctl daemon-reload
df -h
```

![image](https://user-images.githubusercontent.com/58276505/172839939-e3692c10-7b86-4569-9337-0060f6c44389.png)

## PartII:
Launch a second EC2 Instance for Database Server and create 3 volumes in the same AZ & make partition as it was done for Web Server EC2
**Repeat steps above to create partition for DB-server too**

### create db-lv and mount it to /db directory
```
### Install WordPress on Web Server EC2
sudo yum -y update

### Install wget, Apache and it’s dependencies**
sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json

### Start & enable Apache
sudo systemctl start httpd
sudo systemctl enable httpd

### To install PHP and it’s depemdencies
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
setsebool -P httpd_execmem 1

### Restart Apache
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

![image](https://user-images.githubusercontent.com/58276505/172843008-86427b0d-4882-46fe-a956-fb91b28fd1d1.png)

### Install MySQL client and test that connectiveity from Web Server to DB server by using mysql-client
```
sudo yum install mysql
sudo mysql -u admin -p -h 172.31.39.40

# Verify if you can successfully execute
SHOW DATABASES;
```
**Change permissions and configuration so Apache could use WordPress:**
* change ownwer & permit execution
### Enable TCP port 80 in Inbound Rules configuration for Web Server EC2 (enable from everywhere 0.0.0.0/0)

Try to access from your browser the link to your WordPress http://3.17.204.32/wordpress/
![image](https://user-images.githubusercontent.com/41236641/127143778-5166df95-d1bf-4bd5-8a57-b72e502dcfcb.png)
