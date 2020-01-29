# LinuxLearning
# Table of contents
1. [Introduction](#introduction)
2. [Configure SFTP for Web Server](#SFTP)

## What is this? <a name="introduction"></a>
I have started as a habit to document anything that I will do on Linux. It is not only help my documentation but also a reference for me and others.
## Configure SFTP for Web Server? <a name="SFTP"></a>
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



    