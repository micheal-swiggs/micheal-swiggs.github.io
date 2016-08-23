---
layout: post
title: sFTP server on ubuntu 14.04
---

This post is for setting up sFTP on ubuntu 14.04. The following configuration disables FTP – so the user must connect via sFTP.

Let’s install VsFTPD

    sudo apt-get update
    sudo apt-get install vsftpd 

Next open the config file for VsFTPD and make the following changes

    # Uncomment the following lines
    write_enable=YES
    local_umask=022
    chroot_local_user=YES
    allow_writeable_chroot=YES

    # The following line restricts access to the localhost
    listen_address 127.0.0.1

    # Add the following lines to the file
    pasv_enable=Yes
    pasv_min_port=50000
    pasv_max_port=40100

Then restart the service:

    sudo service vsftpd restart 

The FTP server will be listening on port 21. However, it will only accept connections originating from the localhost.

## Securing FTP with SSH

First create a ftpaccess group

    sudo groupadd ftpaccess 

Next make changes to ssh-server

    # Replace the following line
    Subsystem sftp /usr/lib/openssh/sftp-server

    # with
    Subsystem sftp internal-sftp
    Match group ftpaccess
    ChrootDirectory %h
    X11Forwarding no
    AllowTcpForwarding no
    ForceCommand internal-sftp

    # Comment the following line
    UsePAM yes 

Then restart the sshd service

    sudo service ssh restart 

## Adding FTP users

For ftp users we don’t want to allow them access to ssh and telnet. So make sure the following exists in /etc/shells

    /usr/sbin/nologin 

Lets create the user alex

    sudo useradd -m alex -g ftpaccess -s /usr/sbin/nologin
    sudo passwd alex 

Make the root user the owner of the home directory, but create a directory specifically for alex to read and write to:

    sudo chown root /home/alex
    sudo mkdir /home/alex/ftp
    sudo chown alex:ftpaccess /home/alex/ftp 

*Links*

http://www.krizna.com/ubuntu/setup-ftp-server-on-ubuntu-14-04-vsftpd/

http://serverfault.com/questions/358211/can-i-configure-vsftpd-to-listen-only-to-localhost

http://www.cyberciti.biz/tips/howto-linux-shell-restricting-access.html
