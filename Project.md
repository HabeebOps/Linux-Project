# Linux Project - Automated Backup via SSH connection betwenn servers using Rsync and Crontab

## Prerequisites
Below are prerequisites needed for the project:

- Two Ubuntu servers (Server1 and Server2)
- SSH access to both servers
- Basic knowledge of Linux command line
​

## Step by Step Implementation

The following step by step guide is used to implement the project

### Step 1 : Setup Server1 and Server2 on AWS

​Setup the ubuntu servers on AWS. Ensure SSH traffic is enabled on both servers.

![alt text](Images/servers.png)

Update the servers and install Rsync.
​
```
sudo apt update
```

```​
sudo apt install rsync
```
​

### Step 2 : Configure SSH Key-based Authentication

We will set up SSH key-based authentication between the 2 servers to allow secure communication without password prompts.
​

Generate an SSH key pair on server2:
​
```
ssh-keygen -t rsa
```

​![alt text](Images/keypair%20server2.png)

​Run the command below on server2 and copy the displayed text
​
```
cat ~/.ssh/id_rsa.pub
```
​
Run the command below in server1
​
```
echo "your_copied_public_key_here" >> ~/.ssh/authorized_keys
```
​![alt text](Images/server1%20echo.png)

Change permission of authorized_keys in server1
​
```
chmod 700 ~/.ssh/authorized_keys
```
Once the public key from server2 has been copied into the server1, try connecting to the server1 using SSH from the server2:
​
```
ssh username@target_server_ip_address
```

![alt text](Images/ssh%20server2%20-%20server1.png)

​Run `exit` to close the connection

Once all the above is complete, carry out the same above steps to allow server1 to ssh to server2:

​![alt text](Images/keypair%20server1.png)

​![alt text](Images/ssh%20server1%20-%20server2%20test.png)

### Step 3 : Create test users and user data to backup

On server2, create a directory named backup then SSH connect from server2 into server1 and create test directory (workplace) inside server1

`mkdir workplace`

Add test files using touch command and configuration data using nano editor

![alt_text](Images/test%20dir.png)

![alt_text](Images/test%20file1.png)

![alt_text](Images/test%20file2.png)

![alt_text](Images/test%20file3.png)

Check on server1 directly if test directory and files created via ssh connection of server2 to server are present 

![alt_text](Images/test%20check%20server1.png)

### Step 4 : Create Backup Script

While still connected via ssh from server2 to server1, create a backup script in server1, then open it with nano editor

```
touch backup.sh
```

```
nano backup.sh
```
​
Copy the below into the backup.sh and modify as required
​
```
#!/bin/bash
​
set -x
set -e
​
​
# Define source and destination directories
SRC_DIR="/path/to/source"
DEST_HOST="username@destination_server_ip"
DEST_DIR="/path/to/destination"
​
# Rsync command to synchronize files and directories
rsync -avz --delete $SRC_DIR $DEST_HOST:$DEST_DIR
​
# Check rsync exit status
if [ $? -eq 0 ]; then
    echo "Backup completed successfully."
    
else
    echo "Backup failed!"
   
fi
​
```

Save the edited script and exit. 

![alt_text](Images/backup-script.png)
​

Make the script executable:
​
```
chmod +x backup.sh
```
​Check backup to server2 from server1 works by running

`./backup.sh`

Backup to server2 can also be carried out using the rsync command
```
rsync -a ~/source_directory username@remote_host:destination_directory
```

![alt-text](Images/backup-success.png)

If successful, seerver2 backup directory will now contain the test directory and file data from the server1

![alt_text](Images/backup%20check.png)

### Step 5 : Schedule Backup with Cron
Create a new test file in the test directory "workplace" in server1 and edit as required with nano. 

![alt_text](Images/new%20test%20file.png)

![alt_text](Images/new%20file2.png)

Edit the crontab file to schedule the backups. Do this via server2 ssh connection to server1:
​
```
crontab -e
```
Add a cron job to execute the backup script 2 minute intervals
​
```
*/2 * * * * /path/to/backup.sh
```
![alt_text](Images/cronjob.png)

Check status of backup after 2 mins in server2 backup directory

![alt_text](Images/cronjob%20check.png)