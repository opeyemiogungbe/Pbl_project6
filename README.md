# **WEB SOLUTION WITH WORDPRESS** 
WordPress is a free and open-source content management system written in PHP and paired with MySQL or MariaDB as its backend. In this project i will be preparing storage infrastructure on two Linux servers and implement a basic web solution using WordPress.

**This project will be of two parts:**


### Part 1. i'll be configuring storage subsystem for Web and Database servers based on Linux OS. i'll be dealing with disks, partitions and volumes in Linux

**What is storage subsystem**

A storage subsystem is a component of a computer system that is responsible for managing the storage of data. The storage subsystem is responsible for managing the storage of data on various types of storage devices, including hard drives, solid-state drives, and network-attached storage1.

The Linux storage subsystem includes several layers, including the block layer, the file system layer, and the virtual file system (VFS) layer1. The block layer is responsible for managing the low-level access to storage devices, while the file system layer is responsible for organizing data into files and directories. The VFS layer provides a common interface for accessing different file systems and allows multiple file systems to be mounted and accessed simultaneously1.

Linux supports a wide range of file systems, including ext4, XFS, Btrfs, and many others. Each file system has its own set of features and capabilities, and users can choose the file system that best meets their needs

In AWS we will set up an EC2 instance as a web server (This is where you will install WordPress) and another EC2 instance as a database (DB) server using RedHat OS for this project

![Screenshot 2023-08-14 070753](https://github.com/opeyemiogungbe/Pbl_project6/assets/136735745/2270f985-6342-4af1-8bfe-061e064268f1)

### Part 1 configuring webserver

1. We are going to launch an EC2 instance that will serve as “Web Server” and create 3 EBS (additional storage) volumes in the same Availability Zone) as our Web Server EC2, each of 10 GiB.

2. we will attach all three volumes one by one to the Web Server EC2 instance after which we'll open the linux terminal to start our configuration.

3. On our terminal i'll run the lsblk command to see how the attched EBS volume which is gong to be `xvdf xvdg xvdh`
```
lsblk
```

![Screenshot 2023-07-28 064505](https://github.com/opeyemiogungbe/Pbl_project6/assets/136735745/e142dca6-8ca5-4d2d-8eb4-77f6d077bc61)

5. We are going to use gdisk utility to create a single partition on each of the 3 disks running the command

```
sudo gdisk /dev/xvdf
sudo gdisk /dev/xvdg
sudo gdisk /dev/xvdh
```

![Screenshot 2023-08-15 051402](https://github.com/opeyemiogungbe/Pbl_project6/assets/136735745/ea420df2-51bc-4172-bda9-b2b95ec9c1f5)


From the image above we can see our partition to linux file system ( there are other file system we can partiton into). The letter "N" inpute is the code to create, while "p" means print and "w" means to write  GPT data. After this we are going to use lsblk to check if our partition was succesful.
```
lsblk
```
![Screenshot 2023-08-15 054537](https://github.com/opeyemiogungbe/Pbl_project6/assets/136735745/de15b19d-dc73-451a-ae9b-c989c442a5fb)

from the image above we can see chnages `xvdf1 xvdg1, xvdh1 ` this shows our partition was successful 

6. Install lvm2 package using and use lvmdiskscan command to check for available partitions.
 ```
  sudo yum install lvm2
  sudo lvmdiskscan
```

![Screenshot 2023-07-28 075842](https://github.com/opeyemiogungbe/Pbl_project6/assets/136735745/4dde568c-3c51-413e-bbe9-9b34cc96d414)

7. we will use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM
```
sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdg1
sudo pvcreate /dev/xvdh1
```
8. Verify that our Physical volume has been created successfully by runing
   ```
   sudo pvs
   ```
![Screenshot 2023-07-28 075903](https://github.com/opeyemiogungbe/Pbl_project6/assets/136735745/da58de49-ac63-4d87-89d9-366e3221e218)
 
9. We need to use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg
```
sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1
```
10. we need to verify that our VG has been created successfully by running
```
sudo vgs
```

![Screenshot 2023-07-28 080117](https://github.com/opeyemiogungbe/Pbl_project6/assets/136735745/242bbb9d-9a0c-4aa7-a27c-9f01f7237ec2)

11. i will use lvcreate utility to create 2 logical volumes. apps-lv (Using half of the PV size), and logs-lv using the remaining space of the PV size. NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.

```
sudo lvcreate -n apps-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-vg
```

Verify that your Logical Volume has been created successfully by running:

```
sudo lvs
```

![Screenshot 2023-08-15 052054](https://github.com/opeyemiogungbe/Pbl_project6/assets/136735745/71d6568e-3a4c-4cc1-8d03-003c8d897c27)

12. Verify the entire setup

```
sudo vgdisplay -v #view complete setup - VG, PV, and LV
```
![Screenshot 2023-07-28 154246](https://github.com/opeyemiogungbe/Pbl_project6/assets/136735745/21cd716d-cc98-4879-9c1f-cad96f07b500)

13. i'm going to use mkfs.ext4 to format the logical volumes with ext4 filesystem

```
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
```
![Screenshot 2023-07-28 162820](https://github.com/opeyemiogungbe/Pbl_project6/assets/136735745/89e1363b-b80b-4ff0-be16-54d87e9f1ced)

14. Now i'm going to create a folder /var/www/html directory to store website files

```
sudo mkdir -p /var/www/html
```

15. And create another folder /home/recovery/logs to store backup of log data

```
sudo mkdir -p /home/recovery/logs
```

16. i will Mount /var/www/html on apps-lv logical volume running the command

```
sudo mount /dev/webdata-vg/apps-lv /var/www/html/
```
17. Before mounting logs-lv on /var/log/ i will use the rysnc utilities to copy the log files from /var/logs nto /home/recovery/logs.
```
sudo rsync -av /var/log/. /home/recovery/logs/
```

![Screenshot 2023-07-28 163903](https://github.com/opeyemiogungbe/Pbl_project6/assets/136735745/e35e977c-b028-480f-b9c2-787260e078ef)

18. Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why step 17 above is very important). After mounting we will rysnc our log files back to /var/log

```
sudo mount /dev/webdata-vg/logs-lv /var/log
sudo rsync -av /home/recovery/logs/log/. /var/log
```

19. Now we will check our Block ID to get our UUID and use it to update /etc/fstab file so that the mount configuration will persist during restarting/rebooting server

```
sudo blkid
sudo vi /etc/fstab
```

![Screenshot 2023-08-15 055951](https://github.com/opeyemiogungbe/Pbl_project6/assets/136735745/b2cc3f78-c50e-412c-9d43-804f864f6b10)

![Screenshot 2023-08-15 060748](https://github.com/opeyemiogungbe/Pbl_project6/assets/136735745/92bc64c2-1f35-4843-9b62-cde2b85a32eb)

from the image above, we got our UUID: /dev/mapper/webdata--vg-apps--lv and /dev/mapper/webdata--vg-logs--lv: (image 1) and use it to update /etc/fstab

20. TWe will test our configuration and reload the daemon
```
sudo mount -a
sudo systemctl daemon-reload
```

21 We will verify our setup by running df -h, output must look like this:

![Screenshot 2023-07-28 180519](https://github.com/opeyemiogungbe/Pbl_project6/assets/136735745/20a3d42b-6659-47ba-9564-5aa2b4ca800e)


### Part 2. Install WordPress on Webserver and connect it to a remote MySQL database server

In this second part of this project we will be preparing our database and connect it to our wordpress webserver.

Step 1. we will be launching our second RedHat EC2 instance that will have a role – ‘DB Server’
Repeating the same steps as for the Web Server strogae configuration system, but instead of apps-lv create db-lv and mount it to /db directory instead of /var/www/html/. we will be showing the last part of our database storgae configuration for the database since we've done that in step one above. we will run the command bewlow to show that:

```
df -h
```

![Screenshot 2023-08-15 063347](https://github.com/opeyemiogungbe/Pbl_project6/assets/136735745/e0b7aade-ca42-42b3-82c1-adec9e20ce72)

We can see we configre our partitions, pv,vg and lv succesfully. we gave our db-lv 20g and it was mounted on the /db directory as the instructions we were given.

### Step3- Install WordPress on your Web Server EC2

1. We will update the repository, install wget, Apache and it’s dependencies,enable httpd and start Apache

```
sudo yum -y update
sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json
sudo systemctl enable httpd
sudo systemctl start httpd
```
2. We are going to install PHP and it’s depemdencies
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
```
3. Restart Apache
```
sudo systemctl restart httpd
```
![Screenshot 2023-08-02 104808](https://github.com/opeyemiogungbe/Pbl_project6/assets/136735745/c7c45093-e3c0-42e1-8e61-db9b9f771dc3)

![Screenshot 2023-08-03 084005](https://github.com/opeyemiogungbe/Pbl_project6/assets/136735745/a1e08714-cb96-403d-99e5-1beccfbdea17)


The image above show us that our apache and php is running as expected

4 Download wordpress and copy wordpress to var/www/html
 ```
 mkdir wordpress
 cd   wordpress
 sudo wget http://wordpress.org/latest.tar.gz
 sudo tar xzvf latest.tar.gz
 sudo rm -rf latest.tar.gz
 cp wordpress/wp-config-sample.php wordpress/wp-config.php
 cp -R wordpress /var/www/html/
```

![Screenshot 2023-08-08 233303](https://github.com/opeyemiogungbe/Pbl_project6/assets/136735745/2981c221-1f6f-48c1-831e-cae041f4b229)

5. Configure SELinux Policies
```
sudo chown -R apache:apache /var/www/html/wordpress
sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
sudo setsebool -P httpd_can_network_connect=1
```

### Step 4 — Install MySQL on your DB Server EC2
```
sudo yum update
sudo yum install mysql-server
```
It is best practice to always find out that the service is up and running by using sudo systemctl status mysqld, if it is not running, restart the service and enable it so it will be running even after reboot:
```
sudo systemctl restart mysqld
sudo systemctl enable mysqld
```
![Screenshot 2023-08-04 194054](https://github.com/opeyemiogungbe/Pbl_project6/assets/136735745/55c92468-7696-4bd5-9f69-6489fb9dc593)

### Step 5 — Configure DB to work with WordPress
```
sudo mysql
CREATE DATABASE wordpress;
CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit
```
![Screenshot 2023-08-04 093116](https://github.com/opeyemiogungbe/Pbl_project6/assets/136735745/6c482632-d4ff-4c94-be35-65c27bb41434)

![Screenshot 2023-08-04 194737](https://github.com/opeyemiogungbe/Pbl_project6/assets/136735745/3180ddb9-61e3-4da4-830b-b4d934ab966a)

### Step 6 — Configure WordPress to connect to remote database.
We are going to open MySQL port 3306 on DB Server EC2. For extra security, we shall allow access to the DB server ONLY from our Web Server’s IP address, so in the Inbound Rule configuration specify source as /32

### Step 6 - Install MySQL client and test that you can connect from your Web Server to your DB server by using mysql-client
```
sudo yum install mysql
sudo mysql -u admin -p -h <DB-Server-Private-IP-address>
```
![Screenshot 2023-08-11 071258](https://github.com/opeyemiogungbe/Pbl_project6/assets/136735745/5856ffd2-2909-4311-bd64-b1fa03d58095)

The image above shows our webserver connected successfully to our database.

### Final step
On our webswerver we will change permissions

Edit our wp-config.php file (vi/vim/nano) and input our database information (This will help us connect our webserver and Database)

We will also rename our apache welcom configuration so it can read our wordpress configuration.

Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2 (enable from everywhere 0.0.0.0/0 or from your workstation’s IP)

Now we will try to access from your browser the link to your WordPress http://Web-Server-Public-IP-Address

Try to access from your browser the link to your WordPress http://<Web-Server-Public-IP-Address>/wordpress/

![Screenshot 2023-08-11 081554](https://github.com/opeyemiogungbe/Pbl_project6/assets/136735745/cb652258-e670-4c55-9a3d-f5ccfcd57e95)

Our wordpress is live and active... 
