# LinuxLearning
# Table of contents
1. [Introduction](#introduction)
2. [Configure SFTP for Web Server](#SFTP)
3. [Xrdp port and Clip copy issue](#XRDP)
4. [Scanning after adding new harddisk or increasing size](#ExtendHDD)
5. [Performance and System Utilization Report](#Performance)
6. [Add and remove the disk](#AddRemove)
7. [Fix the sudoers syntax error with root password](#SudoNonroot)
8. [Remove floppy disk](#rmfloppy)

## What is this? <a name="introduction"></a>
I have started as a habit to document anything that I will do on Linux. It is not only help my documentation but also a reference for me and others.
## Configure SFTP for Web Server <a name="SFTP"></a>
This configuration for configuring sftp users to uploade the files from computers to apached document root directory such as /var/www/html
  
  * Create a sftp user:  
      `useradd webtester`  
      `passwd webtester`  
  * Restrict user to the document root  
    edit `vi /etc/ssh/sshd_config` put following content:  
    ```
    Subsystem sftp internal-sftp
    Match User webtester
        ChrootDirectory /var/www/
        ForceCommand internal-sftp -u 0022
        X11Forwarding no
        AllowTcpForwarding no
        PasswordAuthentication yes ```  
  * Restart the sshd service:  
    ``` service sshd restart```  

  * Chaange the owner and gropu of html directory   
  `chown webtester:webtester /var/www/html`  

  That's is all. It is easy right?  
  you can test it uploading indext.html from your desktop any linux box using winscp.

  * Note: if you want allow multiple users to upload then you can create a group called `sftpuser` and add all the users to that group. you can change `Match User webtester` to `Match Group sftpusers` and `chown root:sftpuser /var/www/html` and `chmod 775 /var/www/html`


## Xrdp and session getting terminated once you select or copy anything <a name="XRDP"></a>

I recently faced an issue that user unable to take rdp session on Linux machine. The main issue I found that port 3389 wasn't listening  
* Edit xrdp.ini
   ```
   port: 3389
   ```
   `service xrdp restart`
 
* work around for copy paste issue.  
Edit .bashrc of user that facing the issue and add following:  
```
/usr/bin/vncconfig -get SendPrimary 2>/dev/null
if [ $? -eq 0 ];
then
        /usr/bin/vncconfig -set SendCutText=0
        /usr/bin/vncconfig -set SendPrimary=0
fi
```
`source /home/<user_name>/.bashrc`  
`service xrdp restart`

Hopefully these steps will fix the issue.

## Scanning after adding new harddisk or increasing size<a name="ExtendHDD"></a>
* Adding new disk: 
   you can run the below command to discover the newly added disk on host  
   ``` echo "- - -" >> /sys/class/scsi_host/host_$i/scan```  
   where $i is the host number
   #### Example:  
        echo "- - -" >> /sys/class/scsi_host/host0/scan   
        echo "- - -" >> /sys/class/scsi_host/host1/scan
        echo "- - -" >> /sys/class/scsi_host/host2/scan
  Note: If the added disk doest show up on the host after rescan, Please readd the disk with different scsi id.

* Adding more space to existing disk:
  If the added space doesn't show up on the host then you run the following command:  
  ``` echo 1 > /sys/class/scsi_device/0\:0\:0\:0/device/rescan ```  
  ``` echo 1 > /sys/class/scsi_device/0\:0\:1\:0/device/rescan ```  

  You can run the above command based on the disk ID. IF you added space then fist disk then run on 0:0:0:0

* Extending the mounted disk without Partition:  
You can create physical volume directly from disk without partitioning it such as:  
```pvcreate /dev/sdd```  

once you have creaed the pv then you can create VolumeGroup(VG) adn LVM.  

Whenever you want to extend the disk which is used to create PV without partition, you need to run following command after increasing the size and running the disk scan.  
```pvresize /dev/sdd```  
Automatically it increases the VG as well that is created from this disk.

## Performance and System Utilization Report<a name="Performance"></a>
* ```dstat``` is a excellent pefomance monitoring tool to provide the combined report that is provided by indivual command as ```vmstate```,  ```iostate```, ```netstat```, ```mpstate``` etc  
  * Installation  
```yum install dstat```  
``` man dstat``` # detail information and options available  

  * Examples:   
  ``` dstat ```  
  It gives following output:  
    * CPU stats:  
     cpu usage by a user (usr) processes, system (sys) processes, as well as the number of idle (idl) and waiting (wai) processes, hard interrupt (hiq) and soft interrupt (siq).
    * Disk stats:   
    total number of read (read) and write (writ) operations on disks.  
    * Network stats:  
    total amount of bytes received (recv) and sent (send) on network interfaces.  
    * Paging stats:  
    number of times information is copied into (in) and moved out (out) of memory.  
    * System stats:  
    number of interrupts (int) and context switches (csw)  
    ``` dstat -c ```  #CPU stats
    ``` dstat -d ``` #Disk read/write  
* Adding new disk:   ``` dstat -m ``` #memory stats  
   ``` dstat -n ``` # network stats  
    ``` dstat -g ```# paging stats  
    ``` dstat -y ``` # system stats   
    ``` dstat -p ``` # running process  
    ``` dstat -r ``` # read and write stats  
    ``` dstat -s ``` # swap details  
    ```dstat -tcdm  --output report.csv ``` #  it stores the time, cpu, disk and memory in report.csv file  
  
* ```systate```
 
## Add and remove the disk<a name="AddRemove"></a>
* Adding new disk:  
  Let say that you want to add a disk with 100GB, create a partition,create lvm with that partion and mount on /data directory.  
  once you attached the disk to vm from Vcenter/Vcloud, the disk mayn't be visible on OS  
  you can run the below command to discover the newly added disk on host
   ``` echo "- - -" >> /sys/class/scsi_host/host_$i/scan```
   where $i is the host number
   #### Example:
        echo "- - -" >> /sys/class/scsi_host/host0/scan
        echo "- - -" >> /sys/class/scsi_host/host1/scan
        echo "- - -" >> /sys/class/scsi_host/host2/scan
  Note: If the added disk doest show up on the host after rescan, Please readd the disk with different scsi id.  
  You can verify the disk by running `fdisk -l` command, You will see a disk such as /dev/sdb 100GB
  * Partition the disk:  
    The partition can be created by running `fdisk /dev/sdb` and provide following: # please note that /dev/sdb is the device name  
    `n` (new),  
    `p` (primary),  
    `enter` (default first block),  
    `enter` (default last block),  
    `enter` (default block size),  
    `t` (providing filetype),  
    `enter`  (default takes the 1st partition),  
    `8e`  (code for Linux LVM),  
    `w` (write changes),  
    Run `partprobe -s` to discover the newly created partioned that is #/dev/sdb1 
    verify the partition by running `fdisk -l`  

  * Create physical volume(PV)  from newly created partition  
    `pvcreate /dev/sdb1` and verify by `pvs` 
  * Create Volume Group(VG) 
    `vgcreate vg_data /dev/sdb1` vg_data is the name of the volume group and verify by running `vgs`  
  * Create Logical Volume Group(lvm)  
    `lvmcreate -n lv_data -l+100%FREE vg_data` # This creates lv_data lvm with all 100GB space and verify by running `lvs` or `lvdisplay`  
  *  We need to create the filesystem such xfs
    `mkfs.xfs /dev/vg_data/lv_data`
  * Create a directory `mkdir /data`  
  * Add the entry in /etc/fstab file
   `vi /etc/fstab`  
   /dev/mapper/vg_data-data /data xfs	defaults 0 0 	 
  * Finally mount it `mount -a`  
  * verify using `df -hT` # you can see the /data partition with 100GB
    
* Removing disk:  
  After creating the /data partition, you have reliased that you need to devide the 100GB to /data and /home.
  The main problem is here that /data and /home are from different volume group(vg) that vg_data and vg_root. The only solution we have to remove the partition and disk and  create 2 partition of each 50GB, creat pvs with those partion for differrent voulme group.   
  * Please note these steps will delte your data. so please backup and careful.
  Need to perform following steps:
    * `unmount /data` If it fails due to device busy then you have find the process using that /data using `lsof | grep "/data"` and kill that process  
    * Edit the fstab entry  
      `vi /etc/fstab`
      #/dev/mapper/vg_data-data /data xfs defaults 0 0
    * `lvremove /dev/mapper/vg_data-data` # Remove lv_data lvm  
    * `vgremove vg_data` # remove volume group vg_data
    * `pvremove /dev/sdb1` # Remove physical volume(PV)  
    * `echo "- - -" > /sys/class/scsi_host/host0/scan` # scan for change in fstab file  
    *  We are safe now to remove the disk from Vcenter/vcloud. 
    * `echo 1 > /sys/block/sdb/device/delete` # To Mark device is delete.  
    * `echo "- - -" > /sys/class/scsi_host/host0/scan` # scan for the changes and makes sure that disk removed completely from Memory.  
    *  Add the disk now from from Vcenter/vcloud and run the below command to discover the newly added disk on host.  
      `` echo "- - -" >> /sys/class/scsi_host/host_$i/scan```
      where $i is the host number
      #### Example:
        echo "- - -" >> /sys/class/scsi_host/host0/scan
        echo "- - -" >> /sys/class/scsi_host/host1/scan
        echo "- - -" >> /sys/class/scsi_host/host2/scan
      Note: If the added disk doest show up on the host after rescan, Please readd the disk with different scsi id.
      You can verify the disk by running `fdisk -l` command, You will see a disk such as /dev/sdb 100GB
     * Partition the disk:  
     The partition can be created by running `fdisk /dev/sdb` and provide following: # please note that /dev/sdb is the device name  
      `n` (new),  
      `p` (primary),  
      `enter` (default first block),  
      `enter` (+50GB),  
      `t` (providing filetype),  
      `enter`  (default takes the 1st partition),  
      `8e`  (code for Linux LVM),  
      `w` (write changes),  
      Run `partprobe -s` to discover the newly created partioned that is #/dev/sdb1  
      verify the partition by running `fdisk -l` and follow the steps to create pv, vg, lv , filesytem and mount as given the steps in adding hardisk section.  
      Now we have to create another partion such as /dev/sb2 for extending the /home partion by 50GB.  
      * Partition the disk:  
     The partition can be created by running `fdisk /dev/sdb` and provide following: # please note that /dev/sdb is the device name  
      `n` (new),  
      `p` (primary),  
      `enter` (default first block),  
      `enter` (defalut last sector),  
      `t` (providing filetype),  
      `enter`  (default takes the 1st partition),  
      `8e`  (code for Linux LVM),
      `w` (write changes),  
      Run `partprobe -s` to discover the newly created partioned that is #/dev/sdb2  
      * Extend the /home partition  
        Let assume that the /home partition has created using vg_root volume group and lv_home logical volume  
        
       * Create physical volume(PV)  from newly created partition /dev/sdb2  
         `pvcreate /dev/sdb2` and verify by `pvs`  
      * Extend vg_data volume group
        `vgextedn vg_data /dev/sdb2` vg_data is the name of the volume group and verify by running `vgs`. You will see 50GB free  
     * Extend vg_root Logical Volume Group(lvm)  
       `lvextend -l+100%FREE /dev/mapper/vg_data-lv_root`  
     *  Grow the xfs filesystem  
       `xfs_growfs /dev/mapper/vg_data-lv_root` 
     * Extend the ext4 filesystem
     `resiz2fs /dev/mapper/vg_data-lv_root`
     * verify by running `df -hT`  
    
    ## SudoFix<a name="SudoNonroot"></a>
      * Edit the /etc/sudoers file suing `pkexec visudo` it will prompt for which user you want to authenticate with.
      
    ## Remove floppy disk<a name="rmfloppy"></a>
      * `rmmod floppy`
      * `echo "blacklist floppy" | tee /etc/modprobe.d/floppy-blacklist.conf`
      * `update-initramfs -u`
