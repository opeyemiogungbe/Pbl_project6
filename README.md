# **WEB SOLUTION WITH WORDPRESS** 
In this project i will be preparing storage infrastructure on two Linux servers and implement a basic web solution using WordPress. WordPress is a free and open-source content management system written in PHP and paired with MySQL or MariaDB as its backend. 

### This project 6 consists of two parts:

**1. i'll be configuring storage subsystem for Web and Database servers based on Linux OS. i'll be dealing with disks, partitions and volumes in Linux.**

**2. Install WordPress and connect it to a remote MySQL database server.** 

In AWS i'll set up an EC2 instance as a web server (This is where you will install WordPress) and another EC2 instance as a database (DB) server using RedHat OS for this project
![Screenshot 2023-08-14 070753](https://github.com/opeyemiogungbe/Pbl_project6/assets/136735745/2270f985-6342-4af1-8bfe-061e064268f1)

### Part 1 configuring webserver

1. i'm going to launch an EC2 instance that will serve as “Web Server” and create 3 EBS (additional storage) volumes in the same Availability Zone) as my Web Server EC2, each of 10 GiB.

2. i'll attach all three volumes one by one to the Web Server EC2 instance after which i'll open the linux terminal to start my configuration.

3. On my terminal i'll run the lsblk command to see how the attched EBS volume which is gong to be `xvdf xvdg xvdh`
```
lsblk
```

![Screenshot 2023-07-28 064505](https://github.com/opeyemiogungbe/Pbl_project6/assets/136735745/e142dca6-8ca5-4d2d-8eb4-77f6d077bc61)

4. i'll use the df -h command to see all mounts and free space on my server
   ```
   df -h
   ```
   ![Screenshot 2023-07-28 065515](https://github.com/opeyemiogungbe/Pbl_project6/assets/136735745/e398cb31-3756-4d2d-ba41-bdbecff20471)
   

we can seee our attached volume is not mounted to any directory. we only have xvda2 and xvda3 mounted on the root directories.

5. i'm going to use gdisk utility to create a single partition on each of the 3 disks running the command

```
sudo gdisk /dev/xvdf
sudo gdisk /dev/xvdg
sudo gdisk /dev/xvdh
```

![Screenshot 2023-07-28 065552](https://github.com/opeyemiogungbe/Pbl_project6/assets/136735745/a0c076bf-86c7-4037-8aa2-fa1459f78d1c)

after this we are going to use lsblk to check if our partition was succesful.

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

![Screenshot 2023-07-28 075903](https://github.com/opeyemiogungbe/Pbl_project6/assets/136735745/da58de49-ac63-4d87-89d9-366e3221e218)

8. Verify that our Physical volume has been created successfully by runing
   ```
   sudo pvs
   ```

   ![Screenshot 2023-07-28 080117](https://github.com/opeyemiogungbe/Pbl_project6/assets/136735745/6094453d-5959-421b-a602-84e4b5e11be4)

```
sudo pvs
```

![Screenshot 2023-07-28 080033](https://github.com/opeyemiogungbe/Pbl_project6/assets/136735745/4ccaaefa-2241-4626-bab0-bc344e3982c9)

9. We need to use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg
```
sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1
```
10. we need to verify that our VG has been created successfully by running sudo vgs

![Screenshot 2023-07-28 154246](https://github.com/opeyemiogungbe/Pbl_project6/assets/136735745/21cd716d-cc98-4879-9c1f-cad96f07b500)
