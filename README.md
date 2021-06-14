# ArchivingData
  This project focuses on archiving and logging data using tar, journalctl, logrotate, and auditd in Linux systems.
  In an effort to mitigate network attacks and meet federal compliance, Credico Inc. developed an efficient log management program that performs:
  - Log size management using logrotate.
  - Log auditing with auditd to track events, record the events, detect abuse or unauthorized activity, and create custom reports.
  These tools, in addition to archives, backups, scripting, and task automation, contribute to a fully comprehensive log management system.
  
  Steps for Archiving
  - Step 1: Create, Extract, Compress, and Manage tar Backup Archives
  - Step 2: Create, Manage, and Automate Cron Jobs
  - Step 3: Write Basic Bash Scripts
  - Step 4: Manage Log File Sizes
  - Step 5: Check for Policy and File Violations

## Step 1: Create, Extract, Compress, and Manage tar Backup Archives
- Command to extract `TarDocs.tar` 
  -  ```tar -xf TarDocs.tar```
- Create a tar archive called `Javaless_Docs.tar` that excludes the Java directory from the newly extracted `TarDocs/Document/` directory.
  -  ```sudo tar -cvvf Javaless_Doc.tar --exclude Java ~/Projects/TarDocs/Documents```
- Verify that this new `Javaless_Docs.tar` archive does not contain the Java subdirectory by using tar to list the contents of `Javaless_Docs.tar` and then piping `grep` to       search for Java.
  -  ```tar -tvvf Javaless_Doc.tar | grep -i java```
- Why wouldn't you use the options `-x` and `-c` at the same time with tar?
  - The `-x` and `-c` both do different things.  `-x` extracts files from an archive and `-c` creates a new archive.   
  - To use the options simultaneously would not work since you cannot extract from a tar that has not been created.  
      
## Step 2: Create, Manage, and Automate Cron Jobs
- In response to a ransomware attack, you have been tasked with creating an archiving and backup scheme to mitigate against CryptoLocker malware. 
  This attack would encrypt the entire server’s hard disk and can only be unlocked using a 256-bit digital key after a Bitcoin payment is delivered.
- For this task, you'll need to create an archiving cron job using the following specifications:
  - This cron job should create an archive of the following file: /var/log/auth.log.
  - The filename and location of the archive should be: /auth_backup.tgz.
  - The archiving process should be scheduled to run every Wednesday at 6 a.m.
  - Use the correct archiving zip option to compress the archive using gzip.
  - ```0 6 * * 3 tar -czvf auth_backup.tgz /var/log/auth.log > /dev/null 2>&1```
    
## Step 3: Write Basic Bash Scripts
  - Portions of the Gramm-Leach-Bliley Act require organizations to maintain a regular backup regimen for the safe and secure storage of financial data.
  - You'll first need to set up multiple backup directories. Each directory will be dedicated to housing text files that you will create with different kinds of system information.
    - ```mkdir ~/backups/{freemen,diskuse,openlist,freedisk}```
  - Create the `system.sh` script file so that it that does the following:
    - Prints the amount of free memory on the system and saves it to ~/backups/freemem/free_mem.txt.
    - Prints disk usage and saves it to ~/backups/diskuse/disk_usage.txt.
    - Lists all open files and saves it to ~/backups/openlist/open_list.txt.
    - Prints file system disk space statistics and saves it to ~/backups/freedisk/free_disk.txt.
      -     system.sh
         ``` #!/bin/bash
             #Print out free memory on the system and save it to /freemem
             free -mh > ~/backups/freemem/free_mem.txt
             #Print disk usage and save it to /diskuse
             df -h > ~/backups/diskuse/disk_usage.txt
             #List all open files and save to /openlist
             lsof > ~/backups/openlist/open_list.txt
             #Print file system disk space stats
             df -h > ~/backups/freedisk/free_disk.txt```
             
    - Save this file and make sure to change or modify the system.sh file permissions so that it is executable.
     - `chmod +x system.sh`
    - Commands to test script (from /backups)
     -  `cat openlist/open_list.txt`
     -  `cat freemem/free_mem.txt`
     -  `cat freedisk/freedisk.txt`
     -  `cat diskuse/disk_use.txt`
    - Automate your script system.sh by adding it to the weekly system-wide cron directory.
     -  `sudo cp ~/backups/system.sh /etc/cron.weekly`
        
## Step 4: Manage Log File Sizes
  - You realize that the spam messages are making the size of the log files unmanageable.
  - You’ve decided to implement log rotation in order to preserve log entries and keep log file sizes more manageable. 
  - You’ve also chosen to compress logs during rotation to preserve disk space and lower costs.
  - Configure a log rotation scheme that backs up authentication messages to the /var/log/auth.log directory using the following settings:
    - Rotates weekly.
    - Rotates only the seven most recent logs.
    - Does not rotate empty logs.
    - Delays compression.
    - Skips error messages for missing logs and continues to next log.
    - logrotate.config edits
             ```/var/log/auth.log {
                    missingok
                    weekly
                    rotate 7
                    notifempty
                    delaycompress
                }
                                     ``` 
## Step 5: Check for Policy and File Violations
  - In an effort to help mitigate against future attacks, you have decided to create an event monitoring system that specifically generates reports whenever new accounts are created or modified, and when any modifications are made to authorization logs.
  - Verify the auditd service is active
    - `sudo systemctl status auditd`
  - Edit the auditd config file using the following parameters.
    - Number of retained logs is seven.
    - Maximum log file size is 35.
     - `sudo nano /etc/audit/auditd.conf`
    - Edit the rules for auditd. Create rules that watch the following paths:
     - For /etc/shadow, set wra for the permissions to monitor and set the keyname for this rule to hashpass_audit.
     - For /etc/passwd, set wra for the permissions to monitor and set the keyname for this rule to userpass_audit.
     - For /var/log/auth.log, set wra for the permissions to monitor and set the keyname for this rule to authlog_audit.
      - `sudo nano /etc/audit/rules.d/audit.rules`
         `-w /etc/shadow -p wra -k hashpass_audit`
         `-w /etc/passwd -p wra -k userpass_audit`
         `-w /var/log/auth.log -p wra -k authlog_audit`
    - Restart the auditd daemon.
        `sudo systemctl restart auditd`
    - Perform a listing that reveals all existing auditd rules.
        `sudo auditctl -l`
    - Produce an audit report that returns results for all user authentications.
        `sudo aureport -au`
    - Now you will shift into hacker mode. Create a user with sudo useradd attacker and produce an audit report that lists account modifications.
        `sudo aureport -m` 
    - Use auditctl to add another rule that watches the /var/log/cron directory.
        `sudo auditctl -w /var/log/cron -p war -k monitor_varcron`
    - Perform a listing that reveals changes to the auditd rules took affect.
        `sudo auditctl -l`
