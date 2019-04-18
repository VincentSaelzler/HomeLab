### Note Original Files and Permissions
```
# ls -l /etc/nut/
total 52
-rw-r----- 1 root nut   1543 Apr  8 16:50 nut.conf
-rw-r----- 1 root nut   5649 Apr  8 16:57 ups.conf
-rw-r----- 1 root nut   4577 Apr  8 16:42 upsd.conf
-rw-r----- 1 root nut   2183 Apr  8 17:18 upsd.users
-rw-r----- 1 root nut    227 Apr  8 17:33 upsmon.conf
-rw-r----- 1 root nut   3887 Jan 24  2017 upssched.conf
```
#### Copy original as template
All future commands are done in `/etc/nut/`.
```
# cp nut.conf nut.conf.template
```
#### Clear out commented lines.
The first grep gets lines that **don't** start with `#`. The second one clears out blank lines.
```
# cat nut.conf.template | grep -v '#.*' | grep . > nut.conf
# cat nut.conf
MODE=netserver
```
#### Remove Extra Spaces
```
# nano nut.conf
[manually edit]
# cat nut.conf
MODE=netserver
```
#### Important Configs
```
# find /etc/nut/ -type f -not -name "*.template" -print -exec cat {} \;

/etc/nut/upssched.conf
CMDSCRIPT /bin/upssched-cmd

/etc/nut/upsd.users
[upsmon]
    password = fixmepass
    upsmon master

/etc/nut/nut.conf
MODE=netserver

/etc/nut/upsmon.conf
MINSUPPLIES 1
SHUTDOWNCMD "/sbin/shutdown -h +0"
POLLFREQ 5
POLLFREQALERT 5
HOSTSYNC 15
DEADTIME 15
POWERDOWNFLAG /etc/killpower
RBWARNTIME 43200
NOCOMMWARNTIME 300
FINALDELAY 5
MONITOR old-smart-1500 1 upsmon fixmepass master

/etc/nut/upsd.conf
LISTEN 0.0.0.0 3493

/etc/nut/ups.conf
maxretry = 3
[old-smart-1500]
    driver = usbhid-ups
    port = auto
    desc = "The APC UPS at the bottom of the rack."
```