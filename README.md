# Fix for CrowdStrike customers

This tutorial is designed for customers with numerous stations that are not protected with BitLocker. It will guide you through creating an ISO image for deploying over the network to every machine. This WinPE will automatically remove the offending file from CrowdStrike and reboot the machine back to normal.

## Set Up the WinPE Environment:

- Install the Windows Assessment and Deployment Kit (ADK) on your computer (both packages must be installed) - https://go.microsoft.com/fwlink/?linkid=2271337 & https://go.microsoft.com/fwlink/?linkid=2271338
- Launch the Deployment and Imaging Tools Environment as an administrator.

## Create a WinPE Image:

`copype amd64 C:\WinPE_amd64`

This command creates a working directory for your WinPE image.

## Mount the WinPE Image:

`dism /Mount-Image /ImageFile:C:\WinPE_amd64\media\sources\boot.wim /index:1 /MountDir:C:\WinPE_amd64\mount`

## Create a Diskpart Script:

Create a file called `mount_partition.txt` with the following contents (please choose the disc and partition for your systems wisely, in my case it was disc 0 and partition 1):

`select disk 0`  
`select partition 1`  
`assign letter=C`  
`exit`

## Add the Diskpart Script and a Batch File to Run It:

Place the `mount_partition.txt` file in the `C:\WinPE_amd64\mount\Windows\System32` directory.

Create a batch file named `crowdstrike_fix.bat` with the following contents and place it in the same directory:

`diskpart /s X:\Windows\System32\mount_partition.txt`  
`timeout t /5`  
`REM Navigate to the directory`  
`cd /d C:\Windows\System32\drivers\CrowdStrike`  
`REM Delete files matching the pattern`  
`del C-00000291*.sys`  
`REM Reboot the system`  
`wpeutil reboot`

## Modify the Startnet.cmd File:

Edit the `startnet.cmd` file located in `C:\WinPE_amd64\mount\Windows\System32`.

## Add a line to run crowdstrike_fix.bat:

`wpeinit`  
`start crowdstrike_fix.bat`

## Save and Unmount the Image:

`dism /Unmount-Image /MountDir:C:\WinPE_amd64\mount /Commit`

## Create the ISO:

`MakeWinPEMedia /ISO C:\WinPE_amd64 C:\WinPE_amd64\WinPE.iso`

Deploy the ISO to your workstations.
