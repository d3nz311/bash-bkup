# Bash Bkup
NAS BackUp Script between Windows &amp; Linux 

NAS Setup

Download Ubuntu Server 11.04
To make it easy install Xubuntu GUI interface via this link:  http://www.ubuntugeek.com/install-gui-in-ubuntu-server.html
Create NAS user and give him the password that is stated in the Password list
For accessing the attached 3TB drive follow the path: /media/backups/
Create three directories: DailyBackUp, WeeklyBackUp, MonthlyBackUp
Inside the DailyBackUp directory put and run the ResetDirectory.sh script to create the appropriate sub directories
Create the IncrementBackUp.sh script inside the DailyBackUp directory
Place the RotateWeeklyBackUp.sh Script inside the WeeklyBackUp directory



Windows Server Back Setup to NAS

Windows Active Directory Setup
Create a new AD User: NAS
Give password which is in Passwords list. For simplicity make passwords on both the Linux box and AD user the same
Give  NAS administration rights
Give NAS full control on all backup folders under the folder security settings
Use nano.exe in the terminal to write shell scripts
NOTE: When trying to access other drives in Cygwin, you must use the path: "/cygdrive/drive"
Cygwin Installation on Windows Server
Download Cygwin via this link http://ist.uwaterloo.ca/~kscully/CygwinSSHD_W2K3.html
Install these packages: Nano, Open SSH, RSync, All required packages, net-bash is selected
Install in the default 'C:' directory
When setting up SSH in Cygwin: ssh-user-config
Create SSH Passwordless Authentication from the Windows Server to NAS using this guide:http://linuxbites.wordpress.com/2010/06/28/configure-passwordless-ssh-login-in-linuxcygwin/
Copy and paste the three scripts below in the home directory of NAS: DoDailyBakUp.sh, DoWeeklyBackUp.sh, andDoMonthlyBackUp.sh
Windows Task Scheduler Setup
Create a new task (i.e. MyNAS Daily) in the task scheduler
Under the General tab select "Run whether user is logged on or not". Do not select "Do Not Store Password. ..." box
This option will prompt you for the NAS user password whenever a change is made to the specific task but will allow the task to execute regardless of whether NAS user is logged in or not.
Under the Triggers tab select the appropriate schedule for when the task will execute
Under the Actions tab (be careful with the slashes)
Program/Script: C:\cygwin\bin\bash.exe
Add Arguments: -l -c "C:/cygwin/home/nas/ScriptToExecute.sh"
Start in: C:\cygwin\bin\
BackUp Scripts

RotateWeeklyBackUp.sh Script on NAS
Path on NAS: /media/backups/WeeklyBackUp/

 
#!/bin/bash #rotate our backups around mv weekly.6 weekly.tmp mv weekly.5 weekly.6 mv weekly.4 weekly.5 mv weekly.3 weekly.4 mv weekly.2 weekly.3 mv weekly.1 weekly.2 mv weekly.0 weekly.1 rm -r weekly.tmp
 

IncrementBackUp.sh Script on NAS
Path on NAS: /media/backups/DailyBackUp/

 
#!/bin/sh
echo "Daily BackUp Starting"
#rotate backups
mv daily.10 daily.tmp
mv daily.9 daily.10
mv daily.8 daily.9
mv daily.7 daily.8
mv daily.6 daily.7
mv daily.5 daily.6
mv daily.4 daily.5
mv daily.3 daily.4
mv daily.2 daily.3
mv daily.1 daily.2
mv daily.0 daily.1
#now make a copy using links of the latest backup
cp -al daily.1/. daily.0
rsync -a --delete daily.0/ daily.1
rm -r daily.tmp
echo "Daily BackUp Done!"

DoDailyBackUp.sh Script on Windows Server

#! /bin/bash
date.exe >> rsync.log
echo "Rysnc BackUp Process Started ... " >> rsync.daily.log 
echo "...Daily BackUp of File(s) over to NAS ..." >> rsync.daily.log 
ssh.exe nas@192.168.1.33 "cd /media/backups/DailyBackUp; sh IncrementBackUp.sh"
# backup Data
rsync.exe -avzr --delete /cygdrive/y nas@192.168.1.33:/media/backups/DailyBackUp/daily.0 >> rsync.daily.log

echo "DONE!" >> rsync.daily.log 
echo "" >> rsync.daily.log

DoWeeklyBackUp.sh Script on Windows Server
 
#! /bin/bash #make temp directory ssh.exe nas@192.168.1.33 "mkdir /media/backups/WeeklyBackUp/weekly.0" scp.exe -r /cygdrive/c/Data nas@192.168.1.33:/media/backups/WeeklyBackUp/weekly.0 >> rsync.weekly.log scp.exe -r /cygdrive/y nas@192.168.1.33:/media/backups/WeeklyBackUp/weekly.0 >> rsync.weekly.log #execute directory roatation script ssh.exe nas@192.168.1.33 "sh /media/backups/WeeklyBackUp/RotateWeeklyBackUp.sh"
DoMonthlyBackUp.sh Script on Windows Server
 
#! /bin/bash
#Get Directory name appended by date
DIRNAME="backup_$(date +%b)_$(date +%d)_$(date +%Y)"
ssh.exe nas@192.168.1.33 "mkdir /media/backups/MonthlyBackUp/$DIRNAME"
scp.exe -r /cygdrive/c/Data nas@192.168.1.33:/media/backups/MonthlyBackUp/"$DIRNAME" >> rsync.monthly.log
scp.exe -r /cygdrive/y nas@192.168.1.33:/media/backups/MonthlyBackUp/"$DIRNAME" >> rsync.monthly.log

ResetDirectory.sh Script on NAS 
Path on NAS: /media/backups/DailyBackUp/

 
#! /bin/bash #clean directory
rm -r daily*
mkdir daily.0
mkdir daily.1
mkdir daily.2
mkdir daily.3
mkdir daily.4
mkdir daily.5
mkdir daily.6
mkdir daily.7
mkdir daily.8
mkdir daily.9
mkdir daily.10
