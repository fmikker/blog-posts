---
layout: post
title:  "Deduplicating secure backups with borg"
date:   2017-03-21 21:08:00 +0100
tags: linux,backup,server
---

[logo] https://borgbackup.readthedocs.io/en/stable/_static/logo.png

# About
I have been curious about the backup tool [borgbackup](https://github.com/borgbackup) for a while and I finally took the time to replace my [rsync](https://rsync.samba.org/) backup solution with something that has the feature set that I want in a backup tool and something that I think I can trust. I have trust issues with rsync, and it is not really a backup tool, just a file sync tool.
The main [feature set](https://borgbackup.readthedocs.io/en/stable/index.html?highlight=features) in borg that I'm interested in is:
* Compression (lz4, lzma and zlib)
* Encryption (AES-256)
* Deduplication (saves a lot of disk space!)
* Backups mountable as filesystems (Via SSH as well as local.)
..Oh, and did I mention that borg is free software?

# Deployment
I basically followed the [deployment](https://borgbackup.readthedocs.io/en/stable/deployment.html) documentation (not running Ansible, yet.) combined with a torough look at the entire documentation to find out as much as possible before starting the deployment.

It's nothing complicated, basically the installation consists of these tasks:
* Add a backup user on the backup server
* Copy the root user's public key from the _client host_ to the backup server
* Configure the _client's_ public ssh key on the backup server to contain the recommended options from the [documentation](https://borgbackup.readthedocs.io/en/stable/deployment.html#restrictions)
* Initialize the repository
* Create a [script](https://github.com/fmikker/backup-utils/blob/master/borgbackup/borgbackup-root.sh) that runs borg with the parameters that you desire
* Create a [exclude-file](https://github.com/fmikker/backup-utils/blob/master/borgbackup/.borgexcludes) for directiories that does not make sense to back up.
* Create a cron-job to run the backup daily `5 3 * * * root /root/bin/borgbackup-root.sh 2>&1`


## Example

Running a initial backup on the entire root filesystem on my vhost that utilizes ~60GB of disk took about one hour to complete via a 10mbit connection to my backup location.
I used the compression option `zlib,9` which is the highest compression for the compression type on a dual core VM.
Haven't done any benchmarking against the other compression techniques since they are not compatible, so I guess I'll stick with zlib for the time being until I have some more data to go on.

Paraphrazing one of the developers "lzma? That's like doing a burn-in of your cpu"

```
# time ./borgbackup-root.sh 
------------------------------------------------------------------------------
Archive name: hostname-2017-03-25
Archive fingerprint:<REDACTED> 
Time (start): Sat, 2017-03-25 20:48:10
Time (end):   Sat, 2017-03-25 22:01:52
Duration: 1 hours 13 minutes 41.47 seconds
Number of files: 115219
------------------------------------------------------------------------------
                       Original size      Compressed size    Deduplicated size
This archive:               62.63 GB             58.91 GB             34.11 GB
All archives:               62.63 GB             58.91 GB             34.11 GB

                       Unique chunks         Total chunks
Chunk index:                   90123               137002
------------------------------------------------------------------------------

real    73m52.061s
user    44m47.577s
sys     1m23.108s
```

# Conclusion
This deployment isn't anything fancy, but I belive that it will do the job to backup my server and desktop computer(s) to my backup server in a efficient way.

I guess I'll come up with something more elaborate after a while, and I might post that here as well.

10-4




