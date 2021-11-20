# Devops Tooling Website Solution
- The task of the project is to implete a tooling website solution which makes access to Devops tools within the corporate infrastructure easily accessible.
- The project will implement a solution that consist of the following components:
- i. Infrastructure: AWS
- ii. Webserver Linux: Redhat Enterprise Linux 8
- iii. Database Server: Unbuntu 20.04 + MySQL
- iv. Storage Server: Redhat Enterprise Linux 8 + NFS Server
- v. Programming Language: PHP
- vi. Code Repository: Github
![p7](https://user-images.githubusercontent.com/50557587/142696895-e018359e-2c9b-4a6c-848f-e22457c833e8.PNG)

- The diagram above shows 3 stateless Web Servers sharing a common database and also accessing same files using  Network File System as a shared filed storage, which can also be used for back up in case a server crashes as result the content of the server is secured and safe.

## Prepare the NFS Server
- Spin up a new EC2 instance with RHEL Linux Linux 8 Operating System named NFS.
- Create Volumes for the NFS with the availability zone same as the instance type and attach volume to the volume created for NFS ensuring that in the instance slot, you select the instance associated with NFS.
- Open the linux terminal to begin configuration of LVM on the server.
- Use gdisk to create single partition `sudo gdisk /dev/xvdf`.
- Install lvm2 `sudo yum install lvm2 -y`.
- Run pvcreate on the disk to create a physical volume `sudo pvcreate /dev/xvdf1`.
- Verify physical volume has been created successfully `sudo pvs`.
- Run vgcreate command on the physical volume to create a volume group webdata-vg` sudo vgcreate webdata-vg /dev/xvdf1`.
![p8](https://user-images.githubusercontent.com/50557587/142699476-3d120828-6b0c-429d-8045-10d08e32bcc8.PNG)

- Create 3 logical volumes lv-apps, lv-logs and lv-opt sudo lvcreate - n lv-apps -L 5G webdate-vg, lv-opt sudo lvcreate - n lv-logs -L 5G webdate-vg, lv-opt sudo lvcreate - n lv-opt -l 100%FREE webdate-vg
![p9](https://user-images.githubusercontent.com/50557587/142700058-4234f0fe-5fba-47b8-a408-3b80ee85c3f2.PNG)

- Format the disks as xfs `sudo mkfs.xfs /dev/webdata-vg/lv-apps` and the others.

- Create mount point on /mnt directory , for the logical volumes  as follows:
- Mount lv-apps on /mnt/apps – To be used by webservers `sudo mount /dev/webdata-vg/lv-apps /mnt/apps`.
- Mount lv-logs on /mnt/logs – To be used by webserver logs `sudo mount /dev/webdata-vg/lv-logs /mnt/logs`.
- Mount lv-opt on /mnt/opt – To be used by Jenkins server `sudo mount /dev/webdata-vg/lv-opt /mnt/opt`.
- Confirm mount `df -h`.
- Get the UUID of the device, run `sudo blkid`.
- Update fstab with the UUID `sudo vi /etc/fstab`.
![p1](https://user-images.githubusercontent.com/50557587/142701102-06d34092-1881-40f7-8621-57cbaa77f738.PNG)

- Run `sudo mount -a` to confirm if the mount was ok and reload daemon `sudo systemctl daemon-reload`.
- Install NFS Server, configure it to start on reboot and make sure its up and running.
![p10](https://user-images.githubusercontent.com/50557587/142701394-5612f51c-c945-46de-bf3d-5a6ea78a9f31.PNG)

- Set up permission that will allow the Web servers to read, write and execute file on the NFS.
![p11](https://user-images.githubusercontent.com/50557587/142701671-2677477b-4741-4927-b433-e25ac5fdc60d.PNG)

- Get the subnet cidr for NFS on the EC2, locate 'Networking' tab and open Subnet link.
- ![p12](https://user-images.githubusercontent.com/50557587/142702495-d8a05ab6-73e4-44b6-adc4-489c0224a88d.PNG)

- Configure access to NFS for clients with the same subnet "172.31.0.0/20" and run `sudo exportfs -arv`.
![p13](https://user-images.githubusercontent.com/50557587/142702712-9193799d-fe54-4a12-9248-b54dc4bc3290.PNG)
![p14](https://user-images.githubusercontent.com/50557587/142702885-71409361-7298-4fe1-9ed4-2e7339f2a5c5.PNG)

- Check which port is used by the NFS 
![p15](https://user-images.githubusercontent.com/50557587/142703026-d7d3ed55-9ebe-49ba-9a5f-d4948a591743.PNG)

- In order for NFS server to be accessible from your client, you must also open following ports: TCP 111, UDP 111, UDP 2049

## Configure Database
- Install mysql-server `sudo apt install mysql-server`, `sudo systemctl start mysql`, `sudo systemctl enable mysql`, `sudo systemctl status mysql`.
- sudo mysql_secure_installation.
- Create database for tooling and other set up
![p17](https://user-images.githubusercontent.com/50557587/142710997-998642e1-0e67-4377-911b-ea0c4af35a09.PNG)

- Set the bind address `sudo vi  /etc/mysql/mysql.conf.d/mysqld.cnf`
- 
- 

## Configure Web Servers
- Launch 3 new EC2 instance with RHEL 8 Operating System.
- Install NFS client and Apache `sudo yum install nfs-utils nfs4-acl-tools -y`, `sudo yum install httpd -y`.
- Create directory  `mkdir /var/www` and mount `sudo mount -t nfs -o rw,nosuid 172.31.9.79:/mnt/apps /var/www`.
- Run df -h to confirm that NFS was mounted successfully.
- To make sure changes persist after reboot run `sudo vi /etc/fstab` and the following line `172.31.9.79:/mnt/apps /var/www nfs defaults 0 0`.
- Repeat the steps above on the 2 Web Servers.
- Verify that the Apache files and directories are available on the Web servers in /var/www are same on the NFS in /mnt/apps, if same files are showing, it means NFS mounted correctly.
- Locate the log folder for Apache on the Web Server and mount it to NFS server's  export for log.
- Since the log folder already contains some content we back-up that because if we mount on that directory we lose the content inside.
- Run command `sudo mv /var/log/httpd /var/log/httpd.bak`, this command renames the folder httpd to httpd.bak with the content still intact.
- Create a new httpd folder `sudo mkdir /var/log/http` and mount `sudo mount -t nfs -o rw,nosuid 172.31.9.79:/mnt/logs /var/log/http`.
- To make sure changes persist after reboot run `sudo vi /etc/fstab` and the following line `172.31.9.79:/mnt/logs /var/log/http nfs defaults 0 0`.
![p16](https://user-images.githubusercontent.com/50557587/142706687-aadec479-7a90-4146-8932-7f0b3df159ab.PNG)

- Run `sudo systemctl daemon-reload` to update.
- Copy the content of httpd.bak into httpd folder since mount has already taken place `sudo cp -R /var/log/httpd.bak/. /var/log/httpd`.
- Fork the tooling source code from https://github.com/darey-io/tooling to your Github account
- Install git` sudo yum install git`.
- Clone the directory `git clone https://github.com/darey.io/tooling.git`, this will create a folder named tooling.
- cd into tooling `cd tooling/html/`.
- Deploy the content of the html folder to /var/www/html  sudo cp -R html/. /var/www/html.
- Disable Apache default page `sudo mv /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.conf_backup` and restart httpd `sudo systemctl restart httpd`.
- Edit the file functions.php.
- Install mysql server sudo apt install mysql-server.
- cd into html folder from tooling .
- 















- 

