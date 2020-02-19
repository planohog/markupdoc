# SSCNAS1 Virtual Install Guide for Use with CM02   



#### Table of Contents

1. [Overview](#overview)
2. [Requirements](#requirements)
3. [Start-Virtual-Manager](#start-virtual-manager)
4. [Setup-sscnas1-share](#setup-sscnas1-share)
5. [Setup-cm02-mount](#setup-cm02-mount)
6. [Installing-puppet-file](#installing-puppet-file)
7. [Puppet-Test-CM02](#puppet-test-cm02)

## Overview

This document is to cover the manual installation of SSCNAS1 and CM02 for creating a ZBook ISS Test Bed. This test bed can and should be used with puppet automation to create and test software implentation. Testing on SSC clients can be simulated with ssc-pluto-vm but final test on actual ssc client recommended.

## Requirements
* `ZBook Hypervisor with virtual servers sscnas1 and cm02 running.`
* `Zbook Hypervisor virtual-manager running.`
* `SSCNAS1 connectivity and read/write access to //sscnas1/puppet/ share.`
* `CM02 connectivity and read/write access to /etc/security/sscnas1.credentials.`

## Start-Virtual-Manager
___ 1. Login to ZBook as sscadmin

___ 2. Start the xwindows system.

* `[startx]`

___ 3. Start the virtual-manager

* `Right click in main screen to get dialog box.`
* `Move mouse down to Applications.`
* `Click on Run Program.`
* `Enter virt-manager`
* `Click Launch`
* `Click on the terminal icon at the bottom of screen`

___ 4. Start the following servers in Virtual Manager dialog box.

* `cm02`
* `dns01`
* `sscnas1`
* `usl-swa`
* `ntp01`

## Setup-sscnas1-share
___ 1. Login to sscnas1 as sscadmin and run following three commands:

* `Note: Use provided passwords for ground or flight environment.`
* `[sudo bash]`
* `[chown -R sscadmin:sscadmin /admin/pluto/sscadmin] `
* `[chown sscadmin:sscadmin /admin/pluto/sscadmin/.*] `

___ 2. Create smb user puppet-rw on SSCNAS1.

* `Note: Pass commands Will ask for password use the ground environment.`
* `[smbpasswd -a puppet-rw]`
* `[adduser puppet-rw]` (if fails, no-problem; user already exists)  
* `[passwd puppet-rw]`  (set to same password as the others)

___ 3. Verify Samba setup file SSCNAS1 VM.

* `Verify or Edit the samba setup file`
* `[cat /etc/samba/smb.conf]  or [vi /etc/samba/smb.conf]`
```
[puppet]
         comment = Puppet Folder
         read only = Yes
         browseable = No
         path = /mnt/storage/puppet
         valid users = puppet puppet-rw sscadmin oca
```
* ` If edited restart samba process `
* `[ /etc/init.d/smbd restart ]`

___ 3. Verify or Edit group file SSCNAS1 VM.

* `Verify that puppet-rw and puppet are in storage group.`
* `view [cat /etc/group ] or edit [ vi /etc/group ]`
```
   FROM: storage:x:1004:root,truser,sscadmin,oca
   TO:  storage:x:1004:root,truser,sscadmin,oca,puppet,puppet-rw
```
___ 4. Testing cifs mount access on SSCNAS1 VM.

* `Use Terminal  window from end of Start the virtual-manager section #3.`
* `[sudo bash]`
* `[mount -t cifs //192.168.65.51/puppet -o username=puppet-rw /mnt/cifs]`
* `The mount command will ask for password used at Section:Create smb user step #2`
* `The command [ df -h ] should show //sscnas1/puppet  /mnt/cifs at bottom of screen`

__ 5. Allow inbound ssh from Hypervisor on SSCNAS1 VM Lab Use Only.

* `Open ssh from hypervisor by replacing xx.xx.xx.xx with ZBook ip address.`
* `[iptables –A INPUT –i eth0 –p tcp –-dport 22 –s xx.xx.xx.xx –j ACCEPT ]`
* `[iptables-save > /etc/iptables/rules.v4]`

## Setup-cm02-mount

___ 1. Login to cm02 command line window
* `[sudo bash]`
* `Set The username = puppet-rw` and the password = Section Create smb user: Step #2`
* `[vi /etc/security/sscnas1.credientials]`
* `verify in the file /etc/fstab there is a line with //sscnas1`
* `[cat /etc/fstab]`
* `[mount -t]`
* `The command [ df -h ] should show //sscnas1/puppet/puppet4 ... /etc/puppet` 

## Installing-puppet-file. 

___ 1. From the ZBook mount the USB or device with puppet tar ball. 

* `Using the command window started in the Section: virtual-manager Step #3 `
* `[sudo bash]`
* `Use  [ blkid ] or [ fdisk -l ] to locate device.`
* `Use mount command example: [ mount /dev/sdd1 /mnt/usb ]`

___ 2. From the ZBook mount the cifs mount on sscnas1 vm.
 
* `Using the command window started in the Section: virtual-manager Step #3 `
* `Run [ df -h ] If there is no //sscnas1 mount run the following command`
* `[mount -t cifs //192.168.65.51/puppet -o username=puppet-rw /mnt/cifs ]`
* `Run [ df -h ] and verify you have a /mnt/usb and  /mnt/cifs .`

___ 3. Copy tar file onto the sscnas1 vm. 

* `[ cp /mnt/usb/PATH-TO-PUPPET-TAR  /mnt/cifs/puppet4/ ] `
* `[ cd /mnt/cifs/puppet4 ]`
* `[ tar xvzf PUPPET-TAR ]`
* `[find . –type f -exec chmod 644 {} \;]`
* `[find /etc/puppet –type d -exec chmod 755 {} \;]`

## Puppet-Test-CM02

___ 1. Using command line window on CM02 test puppet manifest.

```
This basic test will fail on ssc.amtexport . 
This fail is not important in most cases, please disreguard.
The test should show several lines on the screen starting with Notice:
The last line will show Notice: Applied Catalog in X seconds
```
* `[sudo bash]`
* `[puppet agent -t]`

___ 2. Other usefull CM02 puppet commands run as root or sudo. 

* `[puppet cert list --all]`
* `[puppet cert sign --all]`
* `[facter -p]`
* `[/etc/init.d/puppet-master restart]`
* `[ping anymachinename or ip]`


