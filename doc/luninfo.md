# luninfo

This script can be used to find information about nearly anything which is related to SAN storage in a virtulaized AIX environment:
- system serials and firmware versions
- disk sizes, number of paths, pvids, volume group and mirror pool names
- storage box types and data center locations
- fibre channel adapter types and firmware, WWPNs, configuration parameters
- virtual adapter mappings, VIOS server names and host adapters

This script needs root privileges on all LPARs. padmin on VIOS is not suffucient. There are no software prerequisites, just the installed perl.rte

AIX 5.3, 6.1, 7.1 and 7.2 are supported, VIOS 2.x and 3.x are known to work. Current versions probably don't run on AIX 5.3 anymore because of some used perl features. You should not run AIX 5.3 or 6.1 anywhere anymore because both are not supported anymore...

Currently supported storage boxes:
- DS 8000 (2107)
- SVC (2145), with and without SDDPCM, 2147 will probably also work with minimum code adjustments, just change the number in the source code
- Hitachi (htcvspgx00mpio, **only** with installed ODM fileset **devices.fcp.disk.Hitachi.array.mpio.rte**)
- Purestorage Flasharray (puredisk, **only** with installed ODM fileset **devices.fcp.disk.pure.flasharray.mpio.rte**)
- Nimble Storage (nimblevolume, **only** with installed ODM fileset **devices.disk.NimbleStorageMPIO.rte**)

More boxes can be added. Please supply information and I'll try my very best :-)

Following types of attachments are tested (not all combinations):
- direct attachment to physical or virtual NPIV fibrechannel adapter
- disks mapped "through" the VIOS server as vSCSI/VTD devices, disk UUIDs are visible through vSCSI mappings
- disk multipathing with and without SDDPCM, IBM boxes only
- generic AIX MPIO (mpioosdisk)

Fibrechannel adapter detection is limited, even though generic, it's possible, that your adapter might not be recognized. Please create an issue!

## Table of Contents
- [Syntax](#syntax)
- [Disk summary](#show-disk-summary)
- [Fibrechannel adapter details](#show-fibrechannel-hba-detail)
- [Show disk details](#show-disk-details)
- [Show paths and priorities](#show-paths-and-priorities)
- [Show virtual adapter details and VIOS adapter mappings](#show-virtual-adapter-details-and-vios-adapter-mappings)
- [Show managed system information](#show-managed-system-information)
- [Show known datacenter locations and serial numbers](#show-known-datacenter-locations-and-serial-numbers)
- [luninfo.cfg layout and example configuration](#config-file-layout-and-example-configuration)

## Syntax
```
Usage: luninfo [-h] [-V] [-L] [-H | [ -d [ -l ] | -f [ -l ] | -v | -p | -m | -i [ -l ] [ -g ] ] ]
  -h      display this help
  -V      display version
  -d      list disk detail
  -f      list fibre channel hba detail
  -g      display disk size in GiB
  -H      don't display header line, only useful in combination with other options
  -i      display disk info summary (default when there is no -i, -d, -f, -m, -p or -v)
  -l      long list; display more information like uuid, mirror pool detail, locations, firmware versions
  -m      show managed system info
  -p      list disk paths and priority
  -v      list virtual scsi and fibre channel hba detail
  -L      list all known dc locations and serial numbers
```

## Show disk summary

Shows a summary with disk related information. Using no command line option defaults to "-i".

The option -l adds the complete UUID of each LUN and the mirror pool name of each disk.

The -g option can be used with -i and -l to print the storage size in GiB instead of MiB.

With -H the header line will not be displayed.

Only SAN disks are shown. Locally attached SAS or SCSI disks, DVD drives etc. are ignored. When the script finds a disk device which is unknown, it will print "Unknown/unsupported disk type \<name of the storage type\> for disk \<hdisk name\> found!" In these cases add an identification subroutine or create an issue.

The disk location information (LOC) is derived from the bos serial from disk UUID and the name is configured in luninfo.cfg (see below)

Example outputs:
```
luninfo
#DISK         SIZE LOC              BOX              UID SCSI PATHS      STATE PVID             VG             VGSTATE   TYPE
hdisk2       40960 DC1            12345            00:5D    0  4/4   Available 00f6a07c1dc6f18a altinst_rootvg       -   2145
hdisk3       40960 DC1            12345            07:08    1  4/4   Available 00f7403d3cc9f4b2 cache3vg        active   2145
hdisk4     1454080 DC1            12345            00:5E    3  4/4   Available 00f6a07c1dc6feb3 nimvg           active   2145
hdisk7       40960 DC1            12345            06:FB    6  4/4   Available 00f7403d797d9d0a cache1vg        active   2145
hdisk8       40960 DC2            12346            00:4D    0  4/4   Available 00f6a07c1dc6e3c6 altinst_rootvg       -   2145
hdisk19      39936 DC2 8A5000CD6D99406F            11570    2  4/4   Available 00cb7a716cee1eff None                 -   PURE
hdisk14  133169152 DC1      9004c7b9374 f59a8a2f4c8ac318    4 16/16  Available 00fb2fd399d35fa8 stovg1          active   NIMB
hdisk15  133169152 DC1      9004c7b9374 24b0c462a17e9aa7    5 16/16  Available 00fb2fd399d36f5f stovg2          active   NIMB

luninfo -g
#DISK       SIZE LOC              BOX   UID SCSI PATHS      STATE PVID             VG             VGSTATE   TYPE
hdisk2        40 DC1            12345 00:5D    0  4/4   Available 00f6a07c1dc6f18a altinst_rootvg       -   2145
hdisk3        40 DC1            12345 07:08    1  4/4   Available 00f7403d3cc9f4b2 cache3vg        active   2145

luninfo -l
#DISK         SIZE LOC              BOX                               UUID              UID SCSI PATHS      STATE PVID             VG               VGSTATE   TYPE MPOOL
hdisk2       40960 DC1            12345   6005076801808123450000000000005D            00:5D    0  4/4   Available 00f6a07c1dc6f18a altinst_rootvg         -   2145 None
hdisk3       40960 DC1            12345   60050768018081234500000000000708            07:08    1  4/4   Available 00f7403d3cc9f4b2 cache3vg          active   2145 None
hdisk4     1454080 DC1            12345   6005076801808123450000000000005E            00:5E    3  4/4   Available 00f6a07c1dc6feb3 nim1vg            active   2145 DC1
hdisk7       40960 DC1            12345   600507680180812345000000000006FB            06:FB    6  4/4   Available 00f7403d797d9d0a cache1vg          active   2145 None
hdisk8       40960 DC2            12346   6005076801818123460000000000004D            00:4D    0  4/4   Available 00f6a07c1dc6e3c6 altinst_rootvg         -   2145 None
hdisk9     1454080 DC2            12346   6005076801818123460000000000004E            00:4E    3  4/4   Available 00f6a07c64a181fa nim1vg            active   2145 DC2
hdisk12      40960 DC2            12346   600507680181812346000000000005DD            05:DD    6  4/4   Available 00f7403d797d9f10 cache2vg          active   2145 None
hdisk13      40960 DC2            12346   60050768018181234600000000000605            06:05    7  4/4   Available 00f7403d3ccdc3e5 cache4vg          active   2145 None
hdisk19      39936 DC2 8A5000CD6D99406F           8A5000CD6D99406F00011570            11570    2  4/8   Available 00cb7a716cee1eff None                   -   PURE None
hdisk0       10240 DC1             FFFF                          FFFF_0134            01:34   10  4/4   Available 00cf3f46f3debc32 caavg_private     active    HTC None
hdisk1      102400 DC1             FFFF                          FFFF_0128            01:28   11  4/4   Available 00cf3f46f3deb920 appvg             active    HTC DC1
hdisk2      286720 DC1             FFFF                          FFFF_0000            00:00   12  4/4   Available 00cf3f46d5f0a751 appvg1db          concur    HTC DC1
hdisk60      10240 DC2             FFAA                          FFAA_0134            01:34   13  4/4   Available 00cf3f46f3deb9e7 caavg_backup           -    HTC None
hdisk14  133169152 DC1      9004c7abc71        9004c7abc71f59a8a2f4c8ac318 f59a8a2f4c8ac318    4 16/16  Available 00fb2fd399d35fa8 storvg1           active   NIMB None
hdisk15  133169152 DC1      9004c7abc71        9004c7abc7124b0c462a17e9aa7 24b0c462a17e9aa7    5 16/16  Available 00fb2fd399d36f5f storvg2           active   NIMB None

```
|Column name | Description |
| --- | --- |
| DISK | name of the LUN |
| SIZE | size in MiB, with -g in GiB |
| LOC | Location name of this LUN, see luninfo.cfg |
| BOX | Serial number of the box where this LUN is located on |
| UUID | Complete UUID of the LUN, when there is no real UUID (like with Hitachi or DS8000), then one is generated as "box serial number" + "_" + UID |
| UID | Short UID of the LUN, leave out box serials or leading zeros is possible |
| SCSI | SCSI id of the LUN, in AIX known as lun_id. Storage admins almost always reference this as SCSI id |
| PATHS | Number of working paths / Number of configured paths to the LUN |
| STATE | AIX state of the LUN, either Available or Defined |
| PVID | PVID of the LUN |
| VG | Name of the volume group this LUN is member of, only visible, when the VG is known to AIX, e.g. the VG is imported<br>PowerHA 7 cluster repository disks are shown as caavg_private and caavg_backup, the backup disk name is a fake name to display anything, as long as it is a backup disk, it has no VG name |
| VGSTATE | State of the volume group, can be "active", "concur" (concurrent) or "-" for not activated |
| TYPE | Storage box type of this LUN <ul><li>2145 -> IBM SVC</li><li>2107 -> IBM DS8000</li><li>HTC -> Hitachi GX00</li><li>PURE -> PureStorage Flasharray</li><li>NIMB -> Nimble Storage</li></ul>|
| MPOOL | Name of the configured mirror pool |

When luninfo.cfg is configured correctly, the luninfo LOC column can be used to easily generate the mirror pool names e.g.
```
luninfo -H | awk '^/rootvg/ {print $3}'|xargs -n 1 chpv -M
```

## Show fibrechannel hba detail

Shows a summary with fibrechannel hba related information. Command line option is -f.

The option -l adds the adapter firmware, FRU, part number and feature code of the adapter.

With -H the header line will not be displayed.

```
luninfo -f
#HBA  STATE     SDD    ATTACH LINK SPEED HW_PATH                    WWPN             FSCSI   DYNTRK FC_ERR_REC  MAX_XFER NUMCMD
fcs0  Available -      switch   up    16 U78CB.001.WZA1T45-P1-C6-T1 100000109B1B364A fscsi0  yes    fast_fail   0x100000 1024  
fcs1  Available -      switch   up    16 U78CB.001.WZA1T45-P1-C6-T2 100000109B1B364B fscsi1  yes    fast_fail   0x100000 1024  

luninfo -fl
#HBA  STATE     SDD    ATTACH LINK SPEED HW_PATH                    FW          FRU     PN      FC   WWPN             FSCSI   DYNTRK FC_ERR_REC  MAX_XFER NUMCMD
fcs0  Available -      switch   up    16 U78CB.001.WZA1T45-P1-C6-T1 11.4.415.10 00E3496 00E3495 577F 100000109B1B364A fscsi0  yes    fast_fail   0x100000 1024  
fcs1  Available -      switch   up    16 U78CB.001.WZA1T45-P1-C6-T2 11.4.415.10 00E3496 00E3495 577F 100000109B1B364B fscsi1  yes    fast_fail   0x100000 1024  

luninfo -f
#HBA  STATE     SDD    ATTACH LINK SPEED HW_PATH                      WWPN             FSCSI   DYNTRK FC_ERR_REC  MAX_XFER NUMCMD
fcs0  Available NORMAL switch UNKN     8 U8408.44E.1234568-V10-C14-T1 C0507604F7C30148 fscsi0  yes    fast_fail   0x100000 200   
fcs1  Available NORMAL switch UNKN     8 U8408.44E.1234568-V10-C15-T1 C0507604F7C3014A fscsi1  yes    fast_fail   0x100000 200   
fcs2  Available NORMAL switch UNKN     8 U8408.44E.1234568-V10-C16-T1 C0507604F7C3014C fscsi2  yes    fast_fail   0x100000 200   
fcs3  Available NORMAL switch UNKN     8 U8408.44E.1234568-V10-C17-T1 C0507604F7C3014E fscsi3  yes    fast_fail   0x100000 200   

luninfo -fl
#HBA  STATE     SDD    ATTACH LINK SPEED HW_PATH                      FW   FRU  PN   FC   WWPN             FSCSI   DYNTRK FC_ERR_REC  MAX_XFER NUMCMD
fcs0  Available NORMAL switch UNKN     8 U8408.44E.1234568-V10-C14-T1 VIRT VIRT VIRT VIRT C0507604F7C30148 fscsi0  yes    fast_fail   0x100000 200   
fcs1  Available NORMAL switch UNKN     8 U8408.44E.1234568-V10-C15-T1 VIRT VIRT VIRT VIRT C0507604F7C3014A fscsi1  yes    fast_fail   0x100000 200   
fcs2  Available NORMAL switch UNKN     8 U8408.44E.1234568-V10-C16-T1 VIRT VIRT VIRT VIRT C0507604F7C3014C fscsi2  yes    fast_fail   0x100000 200   
fcs3  Available NORMAL switch UNKN     8 U8408.44E.1234568-V10-C17-T1 VIRT VIRT VIRT VIRT C0507604F7C3014E fscsi3  yes    fast_fail   0x100000 200   
```
|Column name | Description |
| --- | --- |
| HBA | name of the HBA port |
| STATE | AIX port state, either Available or Defined |
| SDD | Port state under SDDPCM (NORMAL, DEGRAD, FAILED), "-" when SDDPCM is not used |
| ATTACH | Attachment type of the port (switch, loop, none) as reported by the adapter port |
| LINK | Link state of the port (up, down), always UNKN inside of an LPAR, because fcstat does not propagate this information |
| SPEED | Link speed of the port, 0 when port is down |
| HW_PATH | Hardware path of the port |
| FW | Firmware version of the adapter |
| PN | Part number of the adapter |
| FC | Feature Code of the adapter |
| WWPN | World Wide Port Number of the physical port or *active* WWPN of the virtual port, *inactive* WWPNs on an LPAR are only visible in the HMC profile |
| FSCSI | FSCSI device which is associated with this port |
| DYNTRK | Adapter setting "dyntrk" for this port |
| FC_ERR_REC | Adapter setting "fc_err_recov" for this port |
| MAX_XFER | Adapter setting "max_xfer_size" for this port |
| NUMCMD | Adapter setting "num_cmd_elems" for this port |

Firmware, FRU, PN and FC are not visible from inside of an LPAR, "VIRT" is shown in this case.

## Show disk details
```
luninfo -d
#DISK      QD  WR PATHS PSTATE ALGORITHM         RESERVE      HCHECK_C      HCHECK_I  HCHECK_M  PR_KEY_V
hdisk2     20  30  4/4  OK     shortest_queue    no_reserve   test_unit_rdy 60        nonactive none    
hdisk3     20  30  4/4  OK     shortest_queue    no_reserve   test_unit_rdy 60        nonactive none    
hdisk4     20  30  4/4  OK     shortest_queue    no_reserve   test_unit_rdy 60        nonactive none    
hdisk5     20  30  4/4  OK     shortest_queue    no_reserve   test_unit_rdy 60        nonactive none    
hdisk6     20  30  4/4  OK     shortest_queue    no_reserve   test_unit_rdy 60        nonactive none    
hdisk7     20  30  4/4  OK     shortest_queue    no_reserve   test_unit_rdy 60        nonactive none    
hdisk8     20  30  4/4  OK     shortest_queue    no_reserve   test_unit_rdy 60        nonactive none    
hdisk9     20  30  4/4  OK     shortest_queue    no_reserve   test_unit_rdy 60        nonactive none    
hdisk10    20  30  4/4  OK     shortest_queue    no_reserve   test_unit_rdy 60        nonactive none    
hdisk11    20  30  4/4  OK     shortest_queue    no_reserve   test_unit_rdy 60        nonactive none    
hdisk12    20  30  4/4  OK     shortest_queue    no_reserve   test_unit_rdy 60        nonactive none    
hdisk13    20  30  4/4  OK     shortest_queue    no_reserve   test_unit_rdy 60        nonactive none    

luninfo -dl
#DISK      QD QDK  WR PATHS PSTATE ALGORITHM         RESERVE      HCHECK_C      HCHECK_I  HCHECK_M  PR_KEY_V
hdisk2     20  20  30  4/4  OK     shortest_queue    no_reserve   test_unit_rdy 60        nonactive none    
hdisk3     20  20  30  4/4  OK     shortest_queue    no_reserve   test_unit_rdy 60        nonactive none    
hdisk4     20  20  30  4/4  OK     shortest_queue    no_reserve   test_unit_rdy 60        nonactive none    
hdisk5     20  20  30  4/4  OK     shortest_queue    no_reserve   test_unit_rdy 60        nonactive none    
hdisk6     20  20  30  4/4  OK     shortest_queue    no_reserve   test_unit_rdy 60        nonactive none    
hdisk7     20  20  30  4/4  OK     shortest_queue    no_reserve   test_unit_rdy 60        nonactive none    
hdisk8     20  20  30  4/4  OK     shortest_queue    no_reserve   test_unit_rdy 60        nonactive none    
hdisk9     20  20  30  4/4  OK     shortest_queue    no_reserve   test_unit_rdy 60        nonactive none    
hdisk10    20  20  30  4/4  OK     shortest_queue    no_reserve   test_unit_rdy 60        nonactive none    
hdisk11    20  20  30  4/4  OK     shortest_queue    no_reserve   test_unit_rdy 60        nonactive none    
hdisk12    20  20  30  4/4  OK     shortest_queue    no_reserve   test_unit_rdy 60        nonactive none    
hdisk13    20  20  30  4/4  OK     shortest_queue    no_reserve   test_unit_rdy 60        nonactive none    
```
|Column name | Description |
| --- | --- |

## Show paths and priorities

|Column name | Description |
| --- | --- |

## Show virtual adapter details and VIOS adapter mappings
```
luninfo -v
#HBA    STATE     HW_PATH                        SLOT VIOS       VHOST      ERR_RECOV    PATH_TO
fcs0    Available U8408.44E.1234568-V10-C14-T1   C14  aixvios1   vfchost6   fast_fail    -      
fcs1    Available U8408.44E.1234568-V10-C15-T1   C15  aixvios1   vfchost7   fast_fail    -      
fcs2    Available U8408.44E.1234568-V10-C16-T1   C16  aixvios2   vfchost6   fast_fail    -      
fcs3    Available U8408.44E.1234568-V10-C17-T1   C17  aixvios2   vfchost7   fast_fail    -      
```
|Column name | Description |
| --- | --- |

## Show managed system information

```
luninfo -m
#MODELNAME      SYSTEMID   FIRMWARE   MSYSNAME  
8284-21A        1234567    SV860_226  p812dc1   

luninfo -ml
#MODELNAME      SYSTEMID   FIRMWARE    FIRMWARE(t) FIRMWARE(p) MSYSNAME   LOCATION RACK    
8284-21A        1234567    SV860_226   SV860_226   SV860_215   p812dc1    DC1      A0  
```

|Column name | Description |
| --- | --- |

## Show known datacenter locations and serial numbers

## config file layout and example configuration
