# AWS-Server-Configuration

This document describes how to setup a Linux server to host a Python web application. 

## Summary
1. IP address: [52.36.251.6][1]
2. URL: [http://ec2-52-36-251-6.us-west-2.compute.amazonaws.com][2]
3. SSH port: 2200
4. Password authentication is disabled on the sshd. Users must pass the private key to login the server through remote connections. 

## Installed sofwares

The following packages were installed in the sequency of their order. 

```
apt-get git
apt-get apache2
apt-get apache2-mod-wsgi
apt-get postgresql
apt-get git
apt-get python pip
pip Flask
```

## Additional Users and Groups
1. Group `catalog_db`: All users in this group is able to read and write the catalog application database.
2. User `grader`: this user is authorized to use sudo.
3. User `catalog` who is a member of the Group `catalog_db`: this user has limited permission to the catalog application database.

## firewall settings
The `ufw` is enabled, and we only allow ports HTTP (80), NTP (123), and SSH (2200). 
```
ufw default deny incoming
ufw default allow outgoing
ufw allow 80
ufw allow 123
ufw allow 2200
ufw enable
```

[1]:http://52.36.251.6
[2]:http://ec2-52-36-251-6.us-west-2.compute.amazonaws.com


