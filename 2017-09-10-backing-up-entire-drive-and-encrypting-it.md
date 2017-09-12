# Backing up the root partition and encrypting the drive 

## Things to remember:

* Wipe the drive before encrypting
dd_rescue -A /dev/urandom /dev/sde -b 10M -B 512
or
cryptsetup open --type plain /dev/sdX foo --key-file /dev/random
pv /dev/zero > /dev/mapper/foo


* Partition sizes and names
* Re-install the boot loader

Flags to use when backing up the root partition
borg -x (only one filesystem, ignores /dev /proc etc, and all other filesystems)
Or manually exclude the unwanted paths

Hints from #borgbackup @ freenode
capi: /boot needs to be on a separate, unencrypted partition

ThomasWaldmann
Bind mount /dev to /rootfs, why? What 


Artefact2 
If using dm-crypt, backup the partition header, just in case.
This is to make sure that the data is accessible if the first sectors of the drive gets corrupt. If the disk header containing the encryption key is damaged, the data is 
impossible to access.

