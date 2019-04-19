# Proxmox UPS Master | HP Envy Laptop
## Role
There aren't any VMs on this server. It is connected to the UPS via USB cable, and helps facilitate clean shutdown of the VMs and iSCSI targets on other machines.
## Configuration
- OS: Proxmox 5.4-1
- Host Name: `pve0.vnet`
### NUT
This server is the last one to shut down. It is connected to the UPS via USB, and it broadcasts UPS information to the other servers on the network.
#### NUT Configuration Files
The files that came by default with the installation of the nut package are stored, for backup purposes, in the `templates` directory.

*NUT can be particular about file permissions. I suggest matching these carefully.*
```
# ls -l /etc/nut/
total 28
-rw-r----- 1 root nut   15 Apr 14 11:04 nut.conf
drwxr-xr-x 2 root nut 4096 Apr 14 11:00 templates
-rw-r----- 1 root nut  109 Apr 14 11:05 ups.conf
-rw-r----- 1 root nut   20 Apr 14 11:06 upsd.conf
-rw-r----- 1 root nut  203 Apr 14 11:08 upsd.users
-rw-r----- 1 root nut 1100 Apr 14 11:10 upsmon.conf
-rw-r----- 1 root nut  325 Apr 15 23:50 upssched.conf
```
```
# cat /etc/nut/nut.conf
MODE=netserver
```
```
# cat /etc/nut/ups.conf
[old-smart-1500]
    driver = usbhid-ups
    port = auto
    desc = "The APC UPS at the bottom of the rack."
```
```
# cat /etc/nut/upsd.conf
LISTEN 0.0.0.0 3493
```
```
# cat /etc/nut/upsd.users
[ups-master]
    password = YOURMASTERPW
    upsmon master

[ups-slave]
    password = YOURSLAVEPW
    upsmon slave

[admin]
    password = YOURADMINPW
    actions = set
    instcmds = all
```
```
# nano /etc/nut/upsmon.conf
root@pve0:~# cat /etc/nut/upsmon.conf
MONITOR old-smart-1500@pve0.vnet 1 ups-master YOURMASTERPW master
MINSUPPLIES 1

SHUTDOWNCMD "/sbin/shutdown -h +0"
NOTIFYCMD /sbin/upssched
POLLFREQ 5
POLLFREQALERT 5
HOSTSYNC 15
DEADTIME 15
POWERDOWNFLAG /etc/killpower

NOTIFYMSG ONLINE "UPS %s: On line power."
NOTIFYMSG ONBATT "UPS %s: On battery."
NOTIFYMSG LOWBATT "UPS %s: Battery is low."
NOTIFYMSG REPLBATT "UPS %s: Battery needs to be replaced."
NOTIFYMSG FSD "UPS %s: Forced shutdown in progress."
NOTIFYMSG SHUTDOWN "Auto logout and shutdown proceeding."
NOTIFYMSG COMMOK "UPS %s: Communications (re-)established."
NOTIFYMSG COMMBAD "UPS %s: Communications lost."
NOTIFYMSG NOCOMM "UPS %s: Not available."
NOTIFYMSG NOPARENT "upsmon parent dead, shutdown impossible."

NOTIFYFLAG ONLINE SYSLOG+WALL+EXEC
NOTIFYFLAG ONBATT SYSLOG+WALL+EXEC
NOTIFYFLAG LOWBATT SYSLOG+WALL
NOTIFYFLAG REPLBATT SYSLOG+WALL
NOTIFYFLAG FSD SYSLOG+WALL
NOTIFYFLAG SHUTDOWN SYSLOG+WALL
NOTIFYFLAG COMMOK SYSLOG+WALL
NOTIFYFLAG COMMBAD SYSLOG+WALL
NOTIFYFLAG NOCOMM SYSLOG+WALL
NOTIFYFLAG NOPARENT SYSLOG+WALL

RBWARNTIME 43200
NOCOMMWARNTIME 300
FINALDELAY 5
```
```
# cat /etc/nut/upssched.conf
CMDSCRIPT /bin/upssched-cmd
PIPEFN /var/run/nut/upssched.pipe
LOCKFN /var/run/nut/upssched.lock

AT ONBATT old-smart-1500@pve0.vnet EXECUTE ups-on-batt
AT ONLINE old-smart-1500@pve0.vnet EXECUTE ups-on-line

AT ONBATT old-smart-1500@pve0.vnet START-TIMER turn-off 900
AT ONLINE old-smart-1500@pve0.vnet CANCEL-TIMER turn-off
```
```
# cat /bin/upssched-cmd
#! /bin/sh
#
# This script should be called by upssched via the CMDSCRIPT directive.
#
# The first argument passed to your CMDSCRIPT is the name of the timer
# from your AT lines.

case $1 in
        ups-on-batt)
                logger -t upssched-cmd "Oh no! $1"
                ;;
        ups-on-line)
                logger -t upssched-cmd "Oh yes! $1"
                ;;
        turn-off)
                logger -t upssched-cmd "Powering off. $1"
                upsmon -c fsd
                ;;
        *)
                logger -t upssched-cmd "Unrecognized command: $1"
                ;;
esac
```

