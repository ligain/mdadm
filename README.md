# mdadm  exercise
A sample script to work with `mdadm` tool.
What's going on in the script:
1) Init `Virtualbox` machine with 5 disks
2) Create RAID5 with all disks
3) Mark one drive as faulty and remove id
4) Add back removed drive
5) Generate `mdadm.conf` for the newly created RAID
6) Divide the RAID into 5 partitions 
7)  Format partitions with `ext4` file system
8) Mount each partition to separate mount point `/raid/part*`

# Run  

0) `vagrant`  should be installed on your system
```
$ vagrant -v
Vagrant 2.2.5
```
1) Clone this repository
```bash  
$ git clone https://github.com/ligain/mdadm.git  
``` 
2) Go to project folder
```bash  
$ cd mdadm/
```  
3) Run `Vagrantfile`
```bash  
$ vagrant up
```  

# Project Goals 
The code is written for educational purposes.