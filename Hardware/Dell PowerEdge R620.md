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
This server is the first one to shut down. It is a "slave" server, despite being connected directly to the UPS.
#### NUT Configuration Files
*NUT can be particular about file permissions and ownership. I suggest matching these carefully.*

File Listing:
```
# ls -l /etc/nut/
total 24
-rw-r----- 1 root nut   15 Apr 24 19:27 nut.conf
-rw-r----- 1 root nut  109 Apr 24 19:27 ups.conf
-rw-r----- 1 root nut   20 Apr 24 19:28 upsd.conf
-rw-r----- 1 root nut  203 Apr 24 19:31 upsd.users
-rw-r----- 1 root nut 1069 Apr 24 20:40 upsmon.conf
-rw-r----- 1 root nut  326 Apr 24 21:19 upssched.conf

```
```
# ls -l /bin/upssched-cmd
-rwxr-xr-x 1 root root 601 Apr 24 19:37 /bin/upssched-cmd
```
File Contents:
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
*The admin and ups-master entries are not required.*

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
# cat /etc/nut/upsmon.conf
MONITOR old-smart-1500@pve1.vnet 1 ups-slave YOURSLAVEPW slave
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
```co
# cat /etc/nut/upssched.conf
CMDSCRIPT /bin/upssched-cmd
PIPEFN /var/run/nut/upssched.pipe
LOCKFN /var/run/nut/upssched.lock

AT ONBATT old-smart-1500@pve1.vnet EXECUTE ups-on-batt
AT ONLINE old-smart-1500@pve1.vnet EXECUTE ups-on-line

AT ONBATT old-smart-1500@pve1.vnet START-TIMER turn-off 300
AT ONLINE old-smart-1500@pve1.vnet CANCEL-TIMER turn-off
```
```shell
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