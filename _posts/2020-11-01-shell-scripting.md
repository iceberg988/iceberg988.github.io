---
layout:     post
title:      "Shell, sed and awk scripting"
author:     "Iceberg"
header-img: "assets/images/yellowstone5.jpg"
catalog:    true
tags:
  - Shell
---

## Shell

Special variables in Shell:

* $0 - The filename of the current script.
* $# - The number of arguments supplied to a script.
* $* - All the arguments are double quoted. If a script receives two arguments, $* is equivalent to $1 $2.
* $@ - All the arguments are individually double quoted. If a script receives two arguments, $@ is equivalent to $1 $2.
* $? - The exit status of the last command executed.
* $$ - The process number of the current shell. For shell scripts, this is the process ID under which they are executing.
* $! - The process number of the last background command.


Enable ssh passwordless login for multiple servers:

```shell
$ rpm -ivh sshpass-1.06-1.el7.x86_64.rpm

$ for i in `seq 0 239`
do
    sshpass -p "password" ssh-copy-id -i /root/.ssh/id_rsa.pub -o StrictHostKeyChecking=no server$i
done
```

## Sed

Print specific line from a file:

```shell
$ sed -n 5p <file>
```

Replace specific line with new string:

```shell
sed -i '2s/.*/aa/' /file/path
```

Add line after string match:

```shell
sed -i '/SERVER = str1/a SERVER = "str2"' /file/path
```

Remove strings from beginning of lines:

```shell
# Remove 40 characters from beginning of lines
$ cat file | sed -r 's/.{40}//' 
```

##  Misc

wget directory:

```shell
wget -r -np -R "index.html*" http://example.com/configs/.vim/
```

Sync Mac Address during boot:

```shell
$ cat /etc/rc.d/rc.local
new_mac=`cat /sys/class/net/eno16780032/address`
sed -i "s/HWADDR=.*/HWADDR=$new_mac/" /etc/sysconfig/network-scripts/ifcfg-eno16780032
service network restart
```