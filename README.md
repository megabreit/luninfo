# luninfo and friends for AIX

It is quite hard to collect all the necessary information to configure and debug fibrechannel storage setups in AIX environments. There are a lot of different tools which gather only a subset of information. Some information is only available via ODM commands. It's nearly impossible to get the whole picture easily and use this information to automate certain storage maintenance operations. This script collection can be used to improve storage operation, document system configurations and debug a lot of problems. This scripts are tested in AIX 5.3, 6.1, 7.1 and 7.2. Newer versions don't run on AIX 5.3 versions because of some used perl functions.

- [luninfo](doc/luninfo.md)
  - shows LUN information of fibre channel LUNs including multipathing, datacenter location, mirror pools, timeouts etc
  - shows physical and virtual fibrechannel adapter configuration including link state, speed, firmware, config parameters etc
  - shows connections to each VIOS server
  - shows disk configuration parameters including timeouts, queue depths, paths, disk sizes etc
  - shows datacenter location of the managed system when configured in luninfo.cfg

- [show_vio_host_mappings](doc/show_vio_host_mappings.md)
  - shows the virtual fibrechannel adapter mappings (NPIV) on VIOS servers

- [show_vio_disk_mappings](doc/show_vio_disk_mappings.md)
  - shows the virtual scsi disk mappings on VIOS servers (only raw disk mappings, no LVM or Storage pool luns)
  - can show the datacenter location of each disk when configured in luninfo.cfg 
  - supports the same storage boxes as luninfo
