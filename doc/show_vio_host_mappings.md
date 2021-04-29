# show_vio_host_mappings

This script shows the mappings of virtual fibrechannel or scsi adapters to AIX LPARs. It needs to be run on a VIOS server as root. padmin permissions are not sufficient.

This script can be used to check
- if all necessary virtual host adapters are available. In usual setups there are at least 2 adapters, 1 for each SAN
- if the virtual adapter is mapped to the correct LPAR (name or ID) and adapter
- if the physical connection and SAN zoning is working. The SAN parameter needs to be LOGGED_IN
- if the mapping of the logical to physical adapter is correct. A convention is helpful here, e.g. C16 goes to SAN 1, C17 goes to SAN 2

```
Usage: show_vio_host_mappings [-h] | [-H ] | [ -d ]
          display vhost to LPAR mapping
  -h      display this help
  -d      display vhost to LPAR mapping with mapped disks and vtds (vscsi only)
  -H      don't display header line
```

Sample output on a VIOS server with NPIV adapters (vfchost) only:
```
#ADAPTER  STATE     SLOT HBA   WWPN             LPARID LPAR        LHBA  VSLOT LPARWWPN                    SAN
vfchost0  Available C10  fcs2  100000109B1DC5EF      6 aixlpar01   fcs2  C16   C0507604F7C300A6      LOGGED_IN
vfchost1  Available C11  fcs1  100000109B1DCA8E      6 aixlpar01   fcs3  C17   C0507604F7C300A8      LOGGED_IN
vfchost2  Available C12  fcs0  100000109B1DCA8D      7 aixlpar02   fcs2  C16   C0507604F859007E      LOGGED_IN
vfchost3  Available C13  fcs3  100000109B1DC5F0      7 aixlpar02   fcs3  C17   C0507604F8590080      LOGGED_IN
vfchost4  Available C14  fcs0  100000109B1DCA8D      8 aixlpar03   fcs2  C16   C0507604F8590086      LOGGED_IN
vfchost5  Available C15  fcs3  100000109B1DC5F0      8 aixlpar03   fcs3  C17   C0507604F8590088      LOGGED_IN
vfchost6  Available C31  fcs1  100000109B1DCA8E     16 aixlpar04   fcs3  C17   C0507604F85900D0      LOGGED_IN
vfchost7  Available C30  fcs2  100000109B1DC5EF     16 aixlpar04   fcs2  C16   C0507604F85900CE      LOGGED_IN
vfchost8  Available C20  fcs0  100000109B1DCA8D     14 aixlpar05   fcs2  C16   C0507604F85900F2      LOGGED_IN
vfchost9  Available C21  fcs3  100000109B1DC5F0     14 aixlpar05   fcs3  C17   C0507604F85900F4      LOGGED_IN
vfchost10 Available C26  fcs2  100000109B1DC5EF     13 aixlpar06   fcs2  C16   C0507604F7C301CC      LOGGED_IN
vfchost11 Available C27  fcs1  100000109B1DCA8E     13 aixlpar06   fcs3  C17   C0507604F7C301CE      LOGGED_IN
vfchost12 Available C29  fcs1  100000109B1DCA8E     15 aixlpar07   fcs3  C17   C0507604F8590098      LOGGED_IN
vfchost13 Available C28  fcs2  100000109B1DC5EF     15 aixlpar07   fcs2  C16   C0507604F8590096      LOGGED_IN
vfchost14 Available C17  fcs1  100000109B1DCA8E      9 aixlpar08   fcs3  C17   C0507604F8590116      LOGGED_IN
vfchost15 Available C16  fcs2  100000109B1DC5EF      9 aixlpar08   fcs2  C16   C0507604F8590114      LOGGED_IN
vfchost24 Available C23  -     -                    11 -           -     -     -                 NOT_LOGGED_IN
vfchost25 Available C22  fcs2  100000109B1DC5EF     11 aixlpar09   fcs2  C16   C0507604F7C300E7      LOGGED_IN
```
In this example vhost24 connected to LPAR 11 (aixlpar09) has a broken mapping.

Columns:
| Name | Purpose |
| ---      |  ------  |
| ADAPTER | Name of the virtual host adapter on the VIOS server|
| STATE | Virtual host adapter state, can be Available or Defined |
| SLOT | Adapter slot of the virtual host adapter as configured in the HMC profile |
| HBA | Name of the physical adapter the virtual adapter is attached to |
| WWPN | World Wide Port Number of the underlying physical fibrechannel adapter |
| LPARID | LPAR ID of the connected LPAR, see lparstat -i inside the LPAR |
| LPAR | Name of the LPAR as defined in the HMC profile, also visible with lparstat -i inside the LPAR |
| LHBA | Name of the virtual fibrechannel adapter inside the LPAR |
| VSLOT | Virtual slot number of the virtual LPAR HBA as configured in the HMC profile |
| LPARWWPN | Active LPAR WWPN, the inactive WWPN is only visible on the HMC! |
| SAN | State of the SAN connection of the virtual LPAR adapter (LOGGED_IN or NOT_LOGGED_IN) |

TODO: Add some example to the -d option
