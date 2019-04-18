### NUT Configuration in ProxMox

#### Install NUT
```
#apt install nut
```
#### Configure NUT as a whole
```
# nano /etc/nut/nut.conf
MODE=netserver
```
#### Configure UPS
The [USBHID-UPS Man Page](https://networkupstools.org/docs/man/usbhid-ups.html) explicitly states that *"The driver ignores the 'port' value in ups.conf"*.
```
[old-smart-1500]
    driver = usbhid-ups
    port = auto 
    desc = "The APC UPS at the bottom of the rack."
```
#### Configure UPSD
The line for listeing on localhost was **commented** by default.
```
# nano /etc/nut/upsd.conf
 LISTEN 127.0.0.1 3493
```
#### Start Driver
```
# upsdrvctl start
Network UPS Tools - UPS driver controller 2.7.4
Network UPS Tools - Generic HID driver 0.41 (2.7.4)
USB communication driver 0.33
Using subdriver: APC HID 0.96
```
#### Start UPSD
```
# upsd
Network UPS Tools upsd 2.7.4
fopen /var/run/nut/upsd.pid: No such file or directory
listening on 127.0.0.1 port 3493
Connected to UPS [old-smart-1500]: usbhid-ups-old-smart-1500
```
#### Get UPS Details
```
# upsc old-smart-1500
Init SSL without certificate database
battery.charge: 100
battery.charge.low: 10
battery.charge.warning: 50
battery.mfr.date: 2006/10/17
battery.runtime: 15120
battery.runtime.low: 120
battery.temperature: 29.7
battery.type: PbAc
battery.voltage: 27.5
battery.voltage.nominal: 24.0
device.mfr: American Power Conversion
device.model: Smart-UPS 1500 RM
device.serial: AS0643130730
device.type: ups
driver.name: usbhid-ups
driver.parameter.pollfreq: 30
driver.parameter.pollinterval: 2
driver.parameter.port: /dev/ugen0.5
driver.parameter.synchronous: no
driver.version: 2.7.4
driver.version.data: APC HID 0.96
driver.version.internal: 0.41
input.sensitivity: high
input.transfer.high: 127
input.transfer.low: 106
input.voltage: 122.4
output.frequency: 60.0
output.voltage: 123.1
output.voltage.nominal: 120.0
ups.beeper.status: enabled
ups.delay.shutdown: 20
ups.delay.start: 30
ups.firmware: 617.3.D
ups.firmware.aux: 8.1
ups.load: 0.0
ups.mfr: American Power Conversion
ups.mfr.date: 2006/10/17
ups.model: Smart-UPS 1500 RM
ups.productid: 0002
ups.serial: AS0643130730
ups.status: OL
ups.test.result: No test initiated
ups.timer.reboot: -1
ups.timer.shutdown: -1
ups.timer.start: -1
ups.vendorid: 051d
```
#### Reboot
To see if everything is loaded automatically. **Yes, Apaprently So!**
```
# upsc old-smart-1500
[expected output]
```
#### Create NUT User
```
# nano /etc/nut/upsd.users 
[upsmon]
    password = fixmepass
    upsmon master
```
#### Reload UPSD
```
# upsd -c reload
Network UPS Tools upsd 2.7.4
```
#### Configure UPSMON
Figure out the default configuation
```
# cat /etc/nut/upsmon.conf  | grep -v '#.*'
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
``` 
#### Add Monitor Directive
```
# nano /etc/nut/upsmon.conf
MONITOR old-smart-1500 1 upsmon fixmepass master
```
#### Start UPSMON
```
# upsmon
Network UPS Tools upsmon 2.7.4
fopen /var/run/nut/upsmon.pid: No such file or directory
Using power down flag file /etc/killpower
UPS: old-smart-1500 (master) (power value 1)
```
#### Test Shutdown
```
# upsdrvctl -t shutdown
[...]
   0.000259     Shutdown UPS: old-smart-1500
   0.000277     exec:  /lib/nut/usbhid-ups -a old-smart-1500 -k

# upsmon -c fsd
```
#### Result
Works, **and** resets the UPS!