---
layout: post
title:  "Hard drive crashes and learning from mistakes"
date:   2017-09-09 18:56:30 +0200
categories: hardware security crash zfs luks
---

Sometime it just happens, again.

I've been down this road a few times before and lost a lot of pictures, music, documents and other stuff that I would like to kept, but hard drives fail, it is just a matter of time before you loose some important data, if you don't follow these three simple steps:
1. Backup your data, preferably on a remote location
2. Use RAID or use a file system that prevents bit-rot and have auto correction features such as btrfs or ZFS. RAID/ZFS/btrfs is NOT a substitute for backups
3. See #1

So, did I follow these simple rules? Of course not.

However, since I monitor my hard drives with [smartmontools](https://en.wikipedia.org/wiki/Smartmontools) and sends the system logs to a remote logserver with notification possibilities I got a heads up pretty early when the drive started to crumble.

It started with a message about the drive was getting full:
_op5 Monitor

Service PROBLEM detected 2017-09-08 15:50:11.
'Disk usage /media/storage' on host 'zaphod' has passed the WARNING threshold.
https://mon/monitor/index.php/status/service/zaphod

Additional info;
DISK WARNING - free space: /media/storage 132860 MB (4.96% inode=100%):_

And just a minute later, the drive started to report errors:
_op5 Monitor

Service PROBLEM detected 2017-09-08 16:51:32.
'Zaphod logs warning or above' on host 'loghost' has passed the WARNING threshold.
https://mon/monitor/index.php/status/service/loghost

Additional info;
FILTER WARNING - zaphod (outside range 0:5) Device: /dev/sde [SAT], 352 Currently unreadable (pending) sectors (changed +192)_

Lets just say that this email gave me the chills down my spine, and thoughts rushed through my head somewhat like ***"oh no, not again!"***, and ***OH SHIII-!***

## Diagnosis
Ok, lets see what's going on here ***_cracks fingers_***:

```
# journalctl -u smartd -S "2017-09-08 15:00:00"
sep 08 16:20:51 zaphod smartd[10844]: Device: /dev/sde [SAT], 160 Currently unreadable (pending) sectors
sep 08 16:20:51 zaphod smartd[10844]: Device: /dev/sde [SAT], 160 Offline uncorrectable sectors
sep 08 16:50:50 zaphod smartd[10844]: Device: /dev/sdb [SAT], SMART Usage Attribute: 195 Hardware_ECC_Recovered changed from 25 to 24
sep 08 16:50:51 zaphod smartd[10844]: Device: /dev/sde [SAT], 352 Currently unreadable (pending) sectors (changed +192)
sep 08 16:50:51 zaphod smartd[10844]: Device: /dev/sde [SAT], 352 Offline uncorrectable sectors (changed +192)
sep 08 16:50:51 zaphod smartd[10844]: Device: /dev/sde [SAT], SMART Usage Attribute: 197 Current_Pending_Sector changed from 100 to 98
sep 08 16:50:51 zaphod smartd[10844]: Device: /dev/sde [SAT], SMART Usage Attribute: 198 Offline_Uncorrectable changed from 100 to 98
sep 08 17:20:52 zaphod smartd[10844]: Device: /dev/sde [SAT], 584 Currently unreadable (pending) sectors (changed +232)
sep 08 17:20:52 zaphod smartd[10844]: Device: /dev/sde [SAT], 584 Offline uncorrectable sectors (changed +232)
```
The issue started just about when I was saving some data from my laptop to the server which almost filled up the drive `/dev/sde1`, and this was the last drop that the drive could take.

### Drive information
This is the bugger that failed me. A 3TB Seagate desktop drive that has been along for about five years, so I'm not that surprised that it gave up. It has been spinning pretty much all its life, and it has preformed pretty well during its lifetime.

***Disk lifetime:***
_Hours:_38730
_Years:_ 4.418207


Verbose disk information:
```
=== START OF INFORMATION SECTION ===
Model Family:     Seagate Barracuda 7200.14 (AF)
Device Model:     ST3000DM001-1CH166
Serial Number:    Z1F0WN7R
LU WWN Device Id: 5 000c50 04df97b03
Firmware Version: CC29
User Capacity:    3 000 592 982 016 bytes [3,00 TB]
Sector Sizes:     512 bytes logical, 4096 bytes physical
Rotation Rate:    7200 rpm
Device is:        In smartctl database [for details use: -P show]
ATA Version is:   ACS-2, ACS-3 T13/2161-D revision 3b
SATA Version is:  SATA 3.1, 6.0 Gb/s (current: 3.0 Gb/s)
Local Time is:    Sat Sep  9 17:14:04 2017 CEST
```
The most recent error on the device:
```
=== START OF READ SMART DATA SECTION ===                                                                                                                               [217/377]
SMART Error Log Version: 1
ATA Error Count: 115 (device log contains only the most recent five errors)
        CR = Command Register [HEX]
        FR = Features Register [HEX]
        SC = Sector Count Register [HEX]
        SN = Sector Number Register [HEX]
        CL = Cylinder Low Register [HEX]
        CH = Cylinder High Register [HEX]
        DH = Device/Head Register [HEX]
        DC = Device Command Register [HEX]
        ER = Error register [HEX]
        ST = Status register [HEX]
Powered_Up_Time is measured from power on, and printed as
DDd+hh:mm:SS.sss where DD=days, hh=hours, mm=minutes,
SS=sec, and sss=millisec. It "wraps" after 49.710 days.

Error 115 occurred at disk power-on lifetime: 38730 hours (1613 days + 18 hours)
  When the command that caused the error occurred, the device was active or idle.

  After command completion occurred, registers were:
  ER ST SC SN CL CH DH
  -- -- -- -- -- -- --
  40 51 00 ff ff ff 0f  Error: UNC at LBA = 0x0fffffff = 268435455

  Commands leading to the command that caused the error were:
  CR FR SC SN CL CH DH DC   Powered_Up_Time  Command/Feature_Name
  -- -- -- -- -- -- -- --  ----------------  --------------------
  60 00 08 ff ff ff 4f 00  37d+01:07:21.451  READ FPDMA QUEUED
  60 00 08 ff ff ff 4f 00  37d+01:07:21.450  READ FPDMA QUEUED
  60 00 08 ff ff ff 4f 00  37d+01:07:21.450  READ FPDMA QUEUED
  60 00 08 ff ff ff 4f 00  37d+01:07:21.450  READ FPDMA QUEUED
  60 00 08 ff ff ff 4f 00  37d+01:07:21.450  READ FPDMA QUEUED
```

Since these are read-errors I don't know how serious they really are - but I'm not taking any chances. 

## Data rescue and mounting the drive read-only
To make as little additional damage as possible to the drive I remounted the drive in read-only mode as soon as I saw the error above. If I've had the storage space to create an image of the drive I would have done that, but unfortunately  I don't have any >3GB drive on the shelf, so I'll make do with what I have.
```
# mount -o remount,ro /dev/sde1
```

And start copying the data off the drive to multiple locations whereever I can fit the data:
```
# cd /mnt/storage
# rsync -avz --progress Pictures /mnt/external/
``` 

## tmux and mosh
Down the road when the ssh-connection started lagging I decided to give [mosh](https://mosh.org/) a shot. This in turn gave me an excuse to give the long neglected application called  [tmux](https://wiki.archlinux.org/index.php/Tmux) a shot due to a [bug in mosh](https://github.com/mobile-shell/mosh/issues/122). 
Tmux didn't really solve my issues, but it is a really nifty tool that is easier to user than Gnu Screen in my opinion. It also has a lot of nifty features such as [powerline support](https://github.com/powerline/powerline) and [advanced configuration possibilities](https://github.com/tmuxinator/tmuxinator) which gave me something new to fiddle with while I was waiting on extracting the data from the failed drive.

After a while I found another application that tries to solve the same problem called [Eternal Terminal](https://mistertea.github.io/EternalTCP/), but I haven't tried that yet, might mention it in an upcoming post if it works good.

## Wiping the failed drive

To make sure that all the data on the failed drive is impossible (or at least very hard) to recover, I will do my best to assure that nobody can access the data that has resided on the drive.
This will be done in three steps:
* Overwriting the drive with zeros from /dev/zero using dd (dd_rescue in this case)
* Do another pass with dd_rescue, this time with random data from /dev/urandom as the source

```
dd_rescue -A /dev/urandom /dev/sde -b 100M -B 512
dd_rescue: (info): Using softbs=102400.0kiB, hardbs=0.5kiB
dd_rescue: (info): ipos: 211353600.0k, opos: 211353600.0k, xferd: 211353600.0k
                   errs:      0, errxfer:         0.0k, succxfer: 211353600.0k
             +curr.rate:    10011kB/s, avg.rate:    10195kB/s, avg.load:011.+%

```
The throughput is about 10Mib/sec, from what I can tell it is because the cpu in the server is really slow. 
* CPU: AMD Turion(tm) II Neo N54L Dual-Core Processor



Use a binary file as keyfile: https://wiki.archlinux.org/index.php/Dm-crypt/Device_encryption#Keyfiles




## Final words
Hard drive crashes are not just evil, they gives you a chance to get a fresh start and clean out [Tumbleweed](https://en.wikipedia.org/wiki/Tumbleweed) from your hard drive and get some fresh data on that new shiny drive!
It can also be used to learn something about hardware, software, backup-routines and how avoid doing the same mistake again (3 hard drive failures and counting..), to create a fault-tolerant and stable storage solution that can withstand a hard drive crash with good backup routines and system monitoring.

ZFS has a lot to offer, and I'm just skimming on the surface to get a mirrored setup going for now, but I will continue to explore the possibility, caveats and quirks. 
I'll also be looking closer at [BTRFS](https://en.wikipedia.org/wiki/Btrfs) / [Archlinux wiki](https://wiki.archlinux.org/index.php/Btrfs) to see what it has to offer in the future.
Btrfs could handle a mirrored setup (raid1) without any major issues, but ZFS is more mature and has more to offer for now.


Do you remember how many uncorrectable sectors that was listed in the inital error message in the beginning? You don't?
Well, to save your wrists the pain to scroll to the top of the page I'll just spell it out: `sep 08 16:20:51 zaphod smartd[10844]: Device: /dev/sde [SAT], 160 Offline uncorrectable sectors`
Now when I've almost filled the drive with zeros, the count has gone up quite a bit when the writes comes closer to the sectors that produced the first error: 

<FIXME>

```
sep 09 15:20:52 zaphod smartd[10844]: Device: /dev/sde [SAT], 2640 Currently unreadable (pending) sectors
sep 09 15:20:52 zaphod smartd[10844]: Device: /dev/sde [SAT], 2640 Offline uncorrectable sectors
sep 09 15:20:52 zaphod smartd[10844]: Device: /dev/sde [SAT], SMART Prefailure Attribute: 1 Raw_Read_Error_Rate changed from 117 to 119
sep 09 15:50:50 zaphod smartd[10844]: Device: /dev/sda [SAT], SMART Usage Attribute: 189 Airflow_Temperature_Cel changed from 25 to 26
sep 09 15:50:50 zaphod smartd[10844]: Device: /dev/sda [SAT], SMART Usage Attribute: 194 Temperature_Celsius changed from 25 to 26
sep 09 15:50:52 zaphod smartd[10844]: Device: /dev/sde [SAT], 2640 Currently unreadable (pending) sectors
sep 09 15:50:52 zaphod smartd[10844]: Device: /dev/sde [SAT], 2640 Offline uncorrectable sectors
sep 09 15:50:52 zaphod smartd[10844]: Device: /dev/sde [SAT], SMART Prefailure Attribute: 1 Raw_Read_Error_Rate changed from 119 to 114
```


# How to setup a secure, encrypted, mirrored and self healing storage pool
In a later post I will go through how to setup a encrypted, mirrored zfs-pool with snapshots and a lot of other goodies, in the mean time, enjoy some reading:

## ZFS guides and documentation
This one I must recommend for everyone that are interested in ZFS on Linux or otherwise, it's the most torough guide to how ZFS works with all it bolts and nuts: [ZFS Administration, Part I- VDEVs](https://pthree.org/2012/12/04/zfs-administration-part-i-vdevs/)
[Install ZFS on CentOS](https://github.com/zfsonlinux/zfs/wiki/RHEL-%26-CentOS)
[The State of ZFS on Linux](https://clusterhq.com/2014/09/11/state-zfs-on-linux/) (2014)

## Cryptsetup documentation and considerations
[Archlinux dm-crypt documentation](https://wiki.archlinux.org/index.php/Dm-crypt)
[Considerations when encrypting a device](https://gitlab.com/cryptsetup/cryptsetup/wikis/FrequentlyAskedQuestions#5-security-aspects)

## Combinations of the above, btrfs and more
[Btrfs: Working with multiple devices](https://lwn.net/Articles/577961/)
[Encrypt the root partiotion, and decrypt it remotely](https://github.com/dracut-crypt-ssh/dracut-crypt-ssh)
[71 TiB DIY NAS Based on ZFS on Linux](http://louwrentius.com/74tb-diy-nas-based-on-zfs-on-linux.html)
[The shocking Truth about the current state of your Data: How to built a fully encrypted file server with ZFS and Linux to avoid data loss and corruption](https://www.skelleton.net/2015/06/14/how-to-built-a-fully-encrypted-file-server-with-zfs-and-linux/#zfsonlinux)
[Archlinux Tmux-guide](https://wiki.archlinux.org/index.php/Tmux)
[Eternal Terminal](https://mistertea.github.io/EternalTCP/)
https://seravo.fi/2015/using-raid-btrfs-recovering-broken-disks
