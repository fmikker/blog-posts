---
layout: post
title:  "Deduplicating secure backups with borg"
date:   2017-03-21 21:08:00 +0100
tags: linux,backup,server
---

[logo] https://borgbackup.readthedocs.io/en/stable/_static/logo.png

# About
I have been curious about the backup tool [borgbackup](https://github.com/borgbackup) for a while and I finally took the time to replace my [rsync](https://rsync.samba.org/) backup solution with something that has the feature set that I want in a backup tool and something that I think I can trust. I have trust issues with rsync, and I don't see it as a backup tool, but rather a file transfer application.
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

# Conclusion
This deployment isn't anything fancy, but I belive that it will do the job to backup my server and desktop computer(s) to my backup server in a efficient way.

I guess I'll come up with something more elaborate after a while, and I might post that here as well.

10-4




