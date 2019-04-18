update and upgrade

```

# apt install nut

# ls -l /etc/nut/
total 44
-rw-r----- 1 root nut  1538 Jan 25  2017 nut.conf
-rw-r----- 1 root nut  5522 Jan 25  2017 ups.conf
-rw-r----- 1 root nut  4578 Jan 25  2017 upsd.conf
-rw-r----- 1 root nut  2131 Jan 25  2017 upsd.users
-rw-r----- 1 root nut 15308 Jan 25  2017 upsmon.conf
-rw-r----- 1 root nut  3887 Jan 25  2017 upssched.conf

# mkdir /etc/nut/templates

# mv /etc/nut/* /etc/nut/templates/
mv: cannot move '/etc/nut/templates' to a subdirectory of itself, '/etc/nut/templates/templates'

# nano /etc/nut/nut.conf
# nano /etc/nut/ups.conf
# nano /etc/nut/upsd.conf
# nano /etc/nut/upsd.users
# nano /etc/nut/upsmon.conf
# nano /etc/nut/upssched.conf

# chmod 755
# chmod 640
# chown root:nut

~# ls -l /etc/nut/
total 28
-rw-r----- 1 root nut   15 Apr 14 11:04 nut.conf
drwxr-xr-x 2 root nut 4096 Apr 14 11:00 templates
-rw-r----- 1 root nut  109 Apr 14 11:05 ups.conf
-rw-r----- 1 root nut   20 Apr 14 11:06 upsd.conf
-rw-r----- 1 root nut  203 Apr 14 11:08 upsd.users
-rw-r----- 1 root nut 1100 Apr 14 11:10 upsmon.conf
-rw-r----- 1 root nut  207 Apr 14 11:12 upssched.conf

# ls -l /etc/nut/templates/
total 44
-rw-r----- 1 root nut  1538 Jan 25  2017 nut.conf
-rw-r----- 1 root nut  5522 Jan 25  2017 ups.conf
-rw-r----- 1 root nut  4578 Jan 25  2017 upsd.conf
-rw-r----- 1 root nut  2131 Jan 25  2017 upsd.users
-rw-r----- 1 root nut 15308 Jan 25  2017 upsmon.conf
-rw-r----- 1 root nut  3887 Jan 25  2017 upssched.conf

# cp /bin/upssched-cmd /bin/upssched-cmd.template
# nano /bin/upssched-cmd

#! /bin/sh
#
# This script should be called by upssched via the CMDSCRIPT directive.
#
# The first argument passed to your CMDSCRIPT is the name of the timer
# from your AT lines.

case $1 in
        ups-on-batt)
                logger -t upssched-cmd "Oh no! The UPS is on wall power."
                ;;
        ups-on-line)
                logger -t upssched-cmd "Oh yes! The UPS is on Battery Power"
                ;;
        *)
                logger -t upssched-cmd "Unrecognized command: $1"
                ;;
esac
```


## Testing

```
upsdrvctl -t shutdown
Network UPS Tools - UPS driver controller 2.7.4
*** Testing mode: not calling exec/kill
   0.000000
If you're not a NUT core developer, chances are that you're told to enable debugging
to see why a driver isn't working for you. We're sorry for the confusion, but this is
the 'upsdrvctl' wrapper, not the driver you're interested in.

Below you'll find one or more lines starting with 'exec:' followed by an absolute
path to the driver binary and some command line option. This is what the driver
starts and you need to copy and paste that line and append the debug flags to that
line (less the 'exec:' prefix).

   0.000080     Shutdown UPS: old-smart-1500
   0.000090     exec:  /lib/nut/usbhid-ups -a old-smart-1500 -k

upsmon -c fsd
power plugged in
after about 20 seconds
computer completes shut off
hear a clunk

upsmon -c fsd
on battery
after about 20 seconds
computer completes shut off
nothing happens with UPS

/lib/nut/usbhid-ups -a old-smart-1500 -k
On wall power - hear clunk
on battery - nothing
```

The apparent result is that the APC UPS does not implement the USB HID specification to the letter.
Possible next step would be to get a serial connection going, or to update the firmware.
Ignoring for now, as the only major drawback here is wear and tear on the batteries



#### Notable Differences from Master

Overall, there are less configuration files. In terms of the contents:

- nut.conf:
  - `netclient` vs `netserver`
- upsmon.conf
  - `MONITOR` directive
  - no `POWERDOWNFLAG`
- upssched.conf
  - the number of seconds for turn-off timer
- /bin/upssched-cmd
  - exactly the same












```




```