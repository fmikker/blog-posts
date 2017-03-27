---
layout: post
title:  "Installation of ElasticSearch on CentOS 7"
date:   2017-03-23 15:37:30 +0200
categories: logging
tags: centos7,logging,elasticsearch
---

#Installation
I basically followed the official [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/_installation.html "Elasticsearch documentation") With some minor additions such as installing the i[Oracle java](http://docs.oracle.com/javase/8/docs/technotes/guides/install/install_overview.html "Oracle Java") and loading the $JAVA_HOME in /etc/profile.d/java.sh:
```bash
export JAVA_HOME=/usr/java/jre1.8.0_121/
export PATH=$PATH:$JAVA_HOME
```
This will set the `$JAVA_HOME` variable when a shell starts so the application won't get confused.

The installation file `elasticsearch-5.2.2/bin/elasticsearch` will bail out if you try to run the installation as the root user. 
Create a separate user for ES:
```bash
# adduser elastic
# passwd elastic
New password: # insert your secret password
Retype new password:
passwd: all authentication tokens updated successfully.
```
Copy the elasticsearch installation package to the newly created user elastic's home directory and change the owner and change to that user:
```
# cp /root/elasticsearch*.tar.gz /home/elastic
# chown elastic:elastic /home/elastic/elasticsearch*.tar.gz
# su - elastic
```
And start over with the installation..

When starting the node it's recommended to change the cluster and the node name, this can be done at the command line as flags to the binary:
```java
./elasticsearch -Ecluster.name=my_cluster_name -Enode.name=my_node_name
```

#Firewall configuration

To enable communication with the ES instance we need to open the firewall to allow such connections. `firewall-cmd`makes this pretty easy:
```bash
# firewall-cmd --add-port 9200/tcp --permanent
# firewall-cmd --reload
```

