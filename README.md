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

![Screenshot 2023-07-28 065537](https://github.com/opeyemiogungbe/Pbl_project6/assets/136735745/6304bf13-a9a9-43ad-802e-f822b6256bdc)


