# show_vio_disk_mappings

This script shows the mappings of locally visible scsi disk devices (hdisk*) to AIX LPARs. This is achieved with the help of virtual scsi host adapters (vhost) and virtual target disks (VTDs). Only mappings of raw disk devices are shown. No LVM volumes or other mappings, CDROMs etc. including SSP (Shared Storage Pools) are supported. vfchost adapter setups will not show any output since the disk mappings are not visible on the VIOS but only inside the LPAR.

It needs to be run on a VIOS server as root. padmin permissions are not sufficient.

This script can be used to check
- if all necessary virtual host adapters are available.
- if each disk device is mapped once per VIOS server
- if there are stale devices not visible to any LPAR
- if the virtual adapter is mapped to the correct LPAR (name or ID) and adapter

show_vio_disk_mappings has no command line options.

Example output:
```
```
