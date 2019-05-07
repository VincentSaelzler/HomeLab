# FreeNAS | Dell PowerEdge R720xd
## Role
This server presents iSCSI targets for the Proxmox machines.
## Specifications
- 1x Intel Xeon E5 2620 v2
  - 2.1 GHz
  - 6 Core / 12 Thread
- 32GB DDR3 1333 ECC (4x8GB)
- Network Cards
  - 4x 1Gbit
- RAID / HBA Cards
  - Dell PERC H310 (Flashed IT Firmware)
- Hard Drives
  - 24x 2.5" Drive Backplane
  - 9x 1 TB Laptop Drives (Mixed Vendors, some 5400 RPM, some 7200 RPM)
  - 1x 20 GB HDD (Originally from Xbox 360)
  - 2x 500GB Laptop HDDs. 
  - 4x Empty Bays
  - 8x Blanks
- 2x 750W (redundant) Power Supplies
## Configuration
- OS: FreeNAS 11.2-U3
- Host Name: `freenas0.vnet`
### Storage Pools
iSCSI is backed by a 4 TB RAID 10 array. Has 4 stripes of mirrored drive pairs. Total of 8 active 1 TB drives. There is also an additional hot spare.

OS runs on a 20 GB single drive. No RAID or anything. FreeNAS is often run on a flash drive, but having 24 2.5" slots on the front of the server, I was fine using a drive for the OS.

NFS backup storage is one mirrored pair of 500 GB HDDs.
### UPS (NUT)
The configuration is done in the GUI. It auto generates the NUT configuration files. *Changing these files manually is discouraged, because any changes will be overwritten by FreeNAS.* Including here just for reference.

File Listing:
```
# ls -l /etc/local/nut
total 8
-r--r-----  1 root  uucp  480 Apr 24 16:50 upsmon.conf
-r--r-----  1 root  uucp  430 Apr 24 16:50 upssched.conf
```
```
# ls -l /usr/local/bin/custom-upssched-cmd
-rwxr-xr-x  1 root  wheel  1065 Apr 15 19:46 /usr/local/bin/custom-upssched-cmd
```
File Contents:
```
# cat /etc/local/nut/upsmon.conf
MONITOR old-smart-1500@pve1.vnet:3493 1 ups-slave YOURSLAVEPW slave
NOTIFYCMD "/usr/local/sbin/upssched"
NOTIFYFLAG ONBATT SYSLOG+WALL+EXEC
NOTIFYFLAG LOWBATT SYSLOG+WALL+EXEC
NOTIFYFLAG ONLINE SYSLOG+WALL+EXEC
NOTIFYFLAG COMMBAD SYSLOG+WALL+EXEC
NOTIFYFLAG COMMOK SYSLOG+WALL+EXEC
NOTIFYFLAG REPLBATT SYSLOG+WALL+EXEC
NOTIFYFLAG NOCOMM SYSLOG+EXEC
NOTIFYFLAG FSD SYSLOG+EXEC
NOTIFYFLAG SHUTDOWN SYSLOG+EXEC
SHUTDOWNCMD "/sbin/shutdown -p now"
POWERDOWNFLAG /etc/nokillpower
```
```
# cat /etc/local/nut/upssched.conf
CMDSCRIPT   /usr/local/bin/custom-upssched-cmd
PIPEFN      /var/db/nut/upssched.pipe
LOCKFN      /var/db/nut/upssched.lock

AT NOCOMM   * EXECUTE EMAIL
AT COMMBAD  * START-TIMER COMMBAD 10
AT COMMOK   * CANCEL-TIMER COMMBAD COMMOK
AT FSD      * EXECUTE EMAIL
AT LOWBATT  * EXECUTE EMAIL
AT ONBATT   * EXECUTE EMAIL
AT ONLINE   * EXECUTE EMAIL
AT REPLBATT * EXECUTE EMAIL
AT SHUTDOWN * EXECUTE EMAIL
AT LOWBATT  * EXECUTE SHUTDOWN
```
```shell
# cat /usr/local/bin/custom-upssched-cmd
#! /bin/sh
#
# This script should be called by upssched via the CMDSCRIPT directive.
#
# Here is a quick example to show how to handle a bunch of possible
# timer names with the help of the case structure.
#
# This script may be replaced with another program without harm.
#
# The first argument passed to your CMDSCRIPT is the name of the timer
# from your AT lines.

. /etc/rc.freenas

PATH=/sbin:/usr/sbin:/bin:/usr/bin:/usr/local/sbin:/usr/local/bin

IFS=\|

f="ups_emailnotify ups_toemail ups_subject ups_shutdown"
sf=$(echo $f | sed -e 's/ /, /g')
${FREENAS_SQLITE_CMD} ${FREENAS_CONFIG} \
"SELECT $sf FROM services_ups ORDER BY -id LIMIT 1" | \
while eval read $f; do

case $1 in
        "SHUTDOWN")
                logger -t upssched-cmd "issuing shutdown"
                /usr/local/sbin/upsmon -c fsd
                ;;
        "EMAIL"|"COMMBAD"|"COMMOK")
                if [ "${ups_emailnotify}" -eq 1 ]; then
                        echo "$NOTIFYTYPE - $UPSNAME" | mail -s "$(echo "${ups_subject}"|sed "s/%d/$(date)/"|sed "s/%h/$(hostname)/")" "${ups_toemail}"
                fi
                ;;
        *)
                logger -t upssched-cmd "Unrecognized command: $1"
                ;;
esac

done
```
