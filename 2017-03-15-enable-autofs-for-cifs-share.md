---
layout: post
title:  "Mount samba share with autofs"
date:   2017-03-15 19:56:00 +0100
tags: linux,filesystem,configuration,fedora
---

This post describes how to configure autofs to automatically mount a CIFS volume in Fedora 25. This is useful when having a network share that you access regularly close by.

## Prerequsites
* Computer running Fedora 25
* Some Linux knowledge
* A existing CIFS (Samba) share on a remote server with a preconfigured user and password. 
* A password configuration in the file `/home/user/.smbpasswd` with the following contents:
..* username=myuser
..* password=yoursecretpassword

## Configuration
1.  Install autofs with dnf install
`# dnf install autofs`

2. Configure the main configuration file `/etc/auto.master`to create the master mountpoint where the mounts later configured in `/etc/auto.cifs`.
Add the root directory where the mounts will be created in.
`# vim /etc/auto.master`

3. Add the following line that reflects where you want your mounts to be located in your filesystem`/mnt/shares`, and where your mount file is located `/etc/auto.cifs`. More about the options later.
`/mnt/shares     /etc/auto.cifs --timeout=60 --ghost`
..* The option `--timeout=60` declares how long the automount program should try to mount the directories.
..* The `--ghost`-option creates the directories even if the mount is not currently mounted. If this is not set the directories will be destoryed/removed when the share is unmounted.

4. Now it is time to configure the actual share that we want to have available in the filesystem
..* Open up the configuration file with an editor
`# vim /etc/auto.cifs`
..* Add the following line and change the mandatory settings `credentials` `uid` `gid` and `://address.to.cifs.server/share` to reflect your configuration
`directory -fstype=cifs,rw,noperm,credentials=/home/myuser/.smbpasswd,sec=ntlm,iocharset=utf8,uid=myuid,gid=mygid ://address.to.cifs.server/share`

5. Start the service and enable autostart on boot
`# systemctl start autofs`
`# systemctl enable autofs`

6. Now you should be able to browse your share and modify files in the share
`$ cd /mnt/shares/share`
`$ ls`

## Troubleshooting
If something goes wrong you might be able to find something in the system logs
`# journalctl -u autofs`

## References
<https://docs.fedoraproject.org/en-US/Fedora/14/html/Storage_Administration_Guide/s1-nfs-client-config-autofs.html>
<https://www.unixmen.com/how-to-mount-a-smbcifs-share-as-an-automount-on-centosfedorarhel/>

