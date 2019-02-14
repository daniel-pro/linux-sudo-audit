# Auditing using sudo

[![N|Solid](https://upload.wikimedia.org/wikipedia/commons/d/d5/Webpage_icon-powered_by_linux.svg)](https://upload.wikimedia.org/wikipedia/commons/d/d5/Webpage_icon-powered_by_linux.svg)

This document shows how to configure ``` sudo ``` with the only purpose of auditing who did what using for example ```root```.

# Real Life Scenario
 Two teams (DBA and SA) need access to Proof of Concept systems where they will carry out their own installation and configuration activities.
 Every single team member will have his/her own nominal user on the systems and will be allowed to run commands, that will be logged, using privileged users - let's say ``` oracle```  or ``` mysql``` users for the DBA team and ```root``` for the SA team.
 


# Configuration
Be sure to have access to the system as root without ```sudo```. This is really important as any misconfiguration could potentially deny a future root access to the system.
Create two groups :
```sh
# groupadd sysadmin
# groupadd dbadmin
```
Edit sudo configuration :
```sh
# visudo
```
Comment out the Runas_Spec regarding wheel group if present, as we don't want anyone in wheel group to be able to run any commands:
```sh
## Allows people in group wheel to run all commands
# %wheel  ALL=(ALL)       ALL
```
Add the following lines :
```sh
Cmnd_Alias DANGEROUS_COMMANDS = /bin/passwd, /sbin/visudo, /bin/su, /bin/vi /etc/sudoers, /bin/vi /etc/shadow, /sbin/fdisk, /bin/bash, /bin/csh, /bin/sh, /bin/chsh, /bin/lchsh
%dbadmin ALL=(oracle:ALL) ALL
%dbadmin ALL=(mysql:ALL) ALL
%sysadmin  ALL=(root:ALL)   ALL, !DANGEROUS_COMMANDS
```
The first line defines a command alias which contains all potentially dangerous commands that won't be permitted via sudo.
The second and third lines allow ```dbadmin``` group members to run any commands as ```oracle``` and/or ```mysql``` user.
The last line allows ```sysadmin``` group members to run any commands as root with the exception of the ```DANGEROUS_COMMANDS``` that might be used to overcome the sudo auditing configuration.

Save the configuration and exit from ```visudo```

Now create all nominal users and assign ```sysadmin``` group to permit the user to run commands as ```root``` and/or ```dbadmin``` group to permit the user to run commands as ```oracle``` and ```mysql```.

```sh
# useradd dpro -G sysadmin,dbaadmin
# useradd jsmith -G dbadmin
# useradd jdoe -G sysadmin
```

#### Security Concerns
* Using CmndAlias ALL will never be safe. Blacklists can easily be overcome
* Whitelists are more secure than Blacklists
