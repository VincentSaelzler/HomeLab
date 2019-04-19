# Proxmox | Dell PowerEdge R620
## Specifications
- 2x Intel Xeon E5 2620
  - 2.0GHz
  - 6 Core / 12 Thread
- 8GB DDR3 1333 ECC (2x4GB)
- Network Cards
  - 4x 1Gbit
- RAID / HBA Cards
  - Dell PERC H310 Mini Mono (Original IR Firmware)
- Hard Drives
  - 8x 2.5" Drive Backplane
  - 1x 146Gb 10K SAS
  - 7x Blanks
- 1x 495W power supply
## Configuration
- OS: Proxmox 5.4-1
- Host Name: `pve1.vnet`
### NUT
This server is the first one to shut down. It is a "slave" server to `pve0.vnet`
#### NUT Configuration Files
The files that came by default with the installation of the nut package are stored, for backup purposes, in the `templates` directory.

*NUT can be particular about file permissions. I suggest matching these carefully.*
```
# ls -l /etc/nut/
total 16
-rw-r--r-- 1 root nut   15 Apr 15 23:24 nut.conf
drwxr-xr-x 2 root nut 4096 Apr 15 23:22 templates
-rw-r--r-- 1 root nut 1069 Apr 15 23:31 upsmon.conf
-rw-r--r-- 1 root nut  324 Apr 16 00:18 upssched.conf
```
```
# cat /etc/nut/nut.conf
MODE=netclient
```
```
# cat /etc/nut/upsmon.conf
MONITOR old-smart-1500@pve0.vnet 1 ups-slave YOURSLAVEPW slave
MINSUPPLIES 1

SHUTDOWNCMD "/sbin/shutdown -h +0"
NOTIFYCMD /sbin/upssched
POLLFREQ 5
POLLFREQALERT 5
HOSTSYNC 15
DEADTIME 15

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

AT ONBATT old-smart-1500@pve0.vnet START-TIMER turn-off 30
AT ONLINE old-smart-1500@pve0.vnet CANCEL-TIMER turn-off
root@pve1:~# nano /etc/nut/upssched.conf
root@pve1:~# cat /etc/nut/upssched.conf
CMDSCRIPT /bin/upssched-cmd
PIPEFN /var/run/nut/upssched.pipe
LOCKFN /var/run/nut/upssched.lock

AT ONBATT old-smart-1500@pve0.vnet EXECUTE ups-on-batt
AT ONLINE old-smart-1500@pve0.vnet EXECUTE ups-on-line

AT ONBATT old-smart-1500@pve0.vnet START-TIMER turn-off 300
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
