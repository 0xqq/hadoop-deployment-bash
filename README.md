# hadoop-deployment-bash

These are shell scripts to deploy Cloudera Manager and related Cloudera encryption products to a cluster.  The goal of these scripts are to be idempotent and to serve as a template for translation into other Configuration Management frameworks/languages.

* Works with RHEL/CentOS 6 or 7 x86_64.
* Works with Ubuntu Trusty 14.04 x86_64.
* Allows for installation of Oracle JDK 7 from Cloudera or Oracle JDK 7 or 8 from Oracle.

This is an example of some of the functionality.  Not everything is documented.  Some scripts have arguments that can be passed to them to change their internal operation.  Read the source to learn more.

# Prep

This is needed for both the Evaluation and Example sections below.

Set the GITREPO variable to the local directory where you have cloned this repository and create a file with SSH login and hostname.
```
GITREPO=~/git/teamclairvoyant/bash

cat <<EOF >HOSTLIST
centos@ip-10-2-6-62.ec2.internal
centos@ip-10-2-6-5.ec2.internal
centos@ip-10-2-5-22.ec2.internal
EOF
```

# Evaluation

Run the evaluation script to gather the configuration of all the nodes of the cluster.  Save the output in the directory "evaluate-pre".
```
mpssh -f HOSTLIST -r ${GITREPO}/evaluate.sh -o evaluate-pre
```

Or you can use SSH-in-a-for-loop and print to the screen.
```
mkdir evaluate-pre
for HOST in `cat HOSTLIST`; do
  echo "*** $HOST"
  scp -p ${GITREPO}/evaluate.sh ${HOST}:
  ssh -q $HOST './evaluate.sh' >evaluate-pre/${HOST}.out 2>evaluate-pre/${HOST}.err
done
```

Or you can use mussh and print to the screen.
```
mussh -H HOSTLIST -m -b -C ${GITREPO}/evaluate.sh
```

# Example

Copy several of the scripts to the nodes.
```
for HOST in `cat HOSTLIST`; do
  echo "*** $HOST"
  scp -p \
  ${GITREPO}/install_tools.sh \
  ${GITREPO}/change_swappiness.sh \
  ${GITREPO}/disable_iptables.sh \
  ${GITREPO}/disable_selinux.sh \
  ${GITREPO}/disable_thp.sh \
  ${GITREPO}/install_ntp.sh \
  ${GITREPO}/install_nscd.sh \
  ${GITREPO}/install_jdk.sh \
  ${GITREPO}/configure_javahome.sh \
  ${GITREPO}/install_jce.sh \
  ${GITREPO}/install_krb5.sh \
  ${GITREPO}/configure_tuned.sh \
  ${GITREPO}/link_openssl.sh \
  ${GITREPO}/install_entropy.sh \
  ${GITREPO}/install_jdbc.sh \
  ${GITREPO}/install_clouderamanageragent.sh \
  $HOST:
done
```

Run the scripts to prep the system for Cloudera Manager installation.  Pin the version of Cloudera Manager to the value in $CMVER.  Also deploy Oracle JDK 8.
```
#BOPT="-x"    # Turn on bash debugging.
#CMVER=5.9.1  # Set specific Cloudera Manager version, or ...
CMVER=5       # ... use major version 5.
for HOST in `cat HOSTLIST`; do
  echo "*** $HOST"
  ssh -t $HOST " \
  sudo bash $BOPT ./install_tools.sh; \
  sudo bash $BOPT ./change_swappiness.sh; \
  sudo bash $BOPT ./disable_iptables.sh; \
  sudo bash $BOPT ./disable_selinux.sh; \
  sudo bash $BOPT ./disable_thp.sh; \
  sudo bash $BOPT ./install_ntp.sh; \
  sudo bash $BOPT ./install_nscd.sh; \
  sudo bash $BOPT ./install_jdk.sh 8 $CMVER; \
  sudo bash $BOPT ./configure_javahome.sh; \
  sudo bash $BOPT ./install_jce.sh; \
  sudo bash $BOPT ./install_krb5.sh; \
  sudo bash $BOPT ./configure_tuned.sh; \
  sudo bash $BOPT ./link_openssl.sh; \
  sudo bash $BOPT ./install_entropy.sh"
done
```

Install the Cloudera Manager agent.
```
CMSERVER=ip-10-2-5-22.ec2.internal
for HOST in `cat HOSTLIST`; do
  echo "*** $HOST"
  ssh -t $HOST "sudo bash $BOPT ./install_clouderamanageragent.sh $CMSERVER $CMVER"
done
```

Install the Cloudera Manager server with the embedded PostgreSQL database.
```
scp -p ${GITREPO}/install_clouderamanagerserver.sh ${CMSERVER}:
ssh -t ${CMSERVER} "sudo bash $BOPT ./install_clouderamanagerserver.sh embedded $CMVER"
```
You can use the argument embedded, postgresql, mysql, or oracle.

# Post Evaluation

Run the evaluation script again to gather the new configuration of all the nodes of the cluster.  Save the output in the directory "evaluate-post".
```
mpssh -f HOSTLIST -r ${GITREPO}/evaluate.sh -o evaluate-post
```

# Contributing to this project

Everyone is welcome to contribute. Please take a moment to review the [guidelines for contributing](CONTRIBUTING.md).

# License

Copyright (C) 2015 [Clairvoyant, LLC.](http://clairvoyantsoft.com/)

Licensed under the Apache License, Version 2.0.
