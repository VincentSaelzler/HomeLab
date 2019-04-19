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
  - Dell PERC H310 (Original IR Firmware)
- Hard Drives
  - 24x 2.5" Drive Backplane (disconnected, missing cable)
  - 16x Empty Bays
  - 8x Blanks
- 2x 750W (redundant) Power Supplies
## Configuration
- OS: FreeNAS 11.2-U3
- Host Name: `freenas0.vnet`
