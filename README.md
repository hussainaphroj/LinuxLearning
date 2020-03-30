# LinuxLearning
# Table of contents
1. [Introduction](#introduction)
2. [Configure SFTP for Web Server](#SFTP)
3. [Xrdp port and Clip copy issue](#XRDP)
4. [Scanning after adding new harddisk or increasing size](#ExtendHDD)
5. [Performance and System Utilization Report](#Performance)

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
   ``` dstat -m ``` #memory stats  
   ``` dstat -n ``` # network stats  
    ``` dstat -g ```# paging stats  
    ``` dstat -y ``` # system stats   
    ``` dstat -p ``` # running process  
    ``` dstat -r ``` # read and write stats  
    ``` dstat -s ``` # swap details  
    ```dstat -tcdm  --output report.csv ``` #  it stores the time, cpu, disk and memory in report.csv file  
  
* ```systate```
 
