# UPS Poweroff on FreeNAS
### References
http://rogerprice.org/NUT/ConfigExamples.A5.pdf


Looks like it's coming in 11.3
https://github.com/freenas/freenas/pull/1753/commits/992f66c2d07af2d690a9b146be673fa30feed1b3
HOSTSYNC exposed in Upsmon.conf

This commit exposes the variable 'HOSTSYNC' in upsmon.conf file. It makes sure that the option is exposed in legacy UI as well and is configurable in ups service configuration.
Ticket: #43932
