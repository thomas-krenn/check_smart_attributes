# check_smart_attributes - Nagios/Icinga plugin to check SMART attributes

## Usage
```bash
$ sudo check_smart_values -dbj <smartdb json file> -d <device path> [-d <device path>]
[-ucfgj <user config json file>] [-p <path to smartctl>] [-nosudo] [-cu] [-ap] [-s]
[-O <extra options>][ -v|-vv|-vvv] [-h] [-V]
```

## Synopsis

```bash
$ sudo ./check_smart_attributes -d /dev/sda -dbj ./check_smartdb.json
Critical [sda_UDMA_CRC_Error_Count = Critical]|sda_Media_Wearout_Indicator=097;16;6
sda_Host_Writes_32MiB=517197 sda_Host_Reads_32MiB=395442
```

## Requirements
The following software is required for `check_smart_attributes`:
* Perl
 * Getopt::Long
 * Config::JSON
 * Storable qw(dclone);
* smartctl

## Installation
On Ubuntu use this command to install smartmontools including smartctl and the
Perl library for parsing JSON config files:
```bash
$ sudo apt-get install smartmontools libconfig-json-perl
```

## Mailing List
* Pleas add new issues via github at https://github.com/thomas-krenn/check_smart_attributes
* The thomas-krenn mailing list archive is at http://lists.thomas-krenn.com/pipermail/tk-monitoring-plugins-user/

## Configuration

### Device(s) to check
The `-d` option specifies the devices to check. If multiple devices should be
checked, specify the option multiple times: `-d /dev/sda -d /dev/sdb`
Then multiple devices can be monitored with one check.

#### Devices with LSI RAID controllers
For devices behind LSI controllers use the megaraid device string and the
corresponding device path. You can get the DID with
```bash
$ sudo storcli64 /c0 /eall/sall show
```
From this output use the number in the DID column. Then call the plugin:
```bash
$ sudo ./check_smart_attributes -dbj check_smartdb.json -d megaraid4,/dev/sda
```

#### Devices with Adaptec RAID controllers
For devices behind Adaptec controllers use the sg device string and an extra
option for the device interface:
```bash
$ sudo ./check_smart_attributes -dbj check_smartdb.json -d /dev/sg2 -O sat
$ sudo ./check_smart_attributes -dbj check_smartdb.json -d /dev/sg2 -O scsi
```
Please consider, that the extra option is mandatory. To find your devices you
can use command line tools like `sg_scan`.
As an alternative for Adaptec RAID controllers on Windows or Linux
you can specify `aacraid,H,L,ID` where `H` is the Host number and `L`
is the LUN number.
The `ID` can be found by executing `arcconf getconfig CONTROLLER-ID`.
Sample output for `CONTROLLER-ID=1`:
`...Reported Channel,Device(T:L): 0,4(4:0)...`
Then the device configuration would be:
 ```bash
$ sudo ./check_smart_attributes -dbj check_smartdb.json -d aacraid,0,0,4
```

#### Devices with hpsa/cciss based RAID controllers
For devices behind hpsa/cciss based RAID controller, you'll mostly find in
HP Hardware, you can use 'cciss,<N>_/dev/sg<X>' for hpsa or
'cciss,<N>_/dev/cciss/c<X>d<N>' for cciss where <N> is the number
of the device and <X> the RAID controller e.g.:
```bash
$ sudo ./check_smart_attributes -dbj check_smartdb.json -d cciss,1_/dev/sg2
```

#### Devices with 3ware RAID controllers
For devices behind 3ware RAID controllers the 3ware device ID must be
used with the corresponding tw device, e.g.:
```bash
$ sudo ./check_smart_attributes -dbj check_smartdb.json -d 3ware,8,/dev/twa0
```
#### NVMe devices
For NVMe devices just use '/dev/nvme[a-z0-9]+'. As attributes with NVMe
are not model specific the generic NVMe entry in the smartdb JSON file
is used:
```bash
$ sudo ./check_smart_attributes -dbj check_smartdb.json -d /dev/nvme0
```

### check_smartdb.json
The smartdb JSON file specifies the smart attribute configuration for a device.
It contains the configuration which value (VALUE or RAW_VALUE) must be taken for
a smart attribute ID. The config is checked on the basis of device and model
strings given by smartctl.

__If your device is not listed in the config please
study the device specification and add the id--attribute mapping.__

Attributes for NVMe devives are not model specific, therefore only one
generic NMVe entry is present in the smart db. If the device is set up
as 'nvme[a-z0-9]+' then NVMe specific parsing of attributes
is done. The generic entry is necessary to enable default thresholds and
performance values for NVMe devices. Moreover now with the generic entry
users can still override the smart db entry with 'ucfgj'. This is way
better than having hardcoded parsing of NVMe attributes in the plugin.

The smartdb JSON file also specifies default threshold and performance values.
These values state the Warning/Critical sensor thresholds and the performance
data sensors. The threshold and performance base config can be overwritten with
a user config JSON file (see `check_smartcfg.json` below).

If a RAW_VALUE requires bit rotation, check_smartdb.json can take an additional
rotation pattern, e.g.:

```"230" : ["85","95","r8"]```

Then the value is shifted according to the third argument, left if "l" is
specified or right if "r" is used. The number suffix tells the plugin how many
times shift should be done.

### check_smartcfg.json (optional)
Specify the path at which the JSON user config file can be found.
The user config can be used to override the threshold and performance base
config. This can be useful if the thresholds for a specific device must
be changed (e.g. showing up a non-critical error).

Note that a given sensor in the user config overwrites the corresponding sensor
in the smartdb base JSON file.

Attention: if megaraid devices are used the path must be set as
'/dev/megaraidID'. This is necessary for the internal naming convention
the plugin uses.
Attention: if 3ware devices are used the path must be set as
'/dev/3wareID'. This is necessary for the internal naming convention
the plugin uses.

### sudo for nagios user
The smartctl executable requires root privileges to check a device. In order to
use the plugin with the nagios user, it is recommended to create a sudoers entry
for smartctl:
```
nagios ALL=(root) NOPASSWD:/usr/sbin/smartctl
```

## License
check_smart_attributes: Nagios/Icinga plugin to check smart attributes with
smartctl.

Copyright (C) 2019 Thomas-Krenn.AG,

This program is free software; you can redistribute it and/or modify it under
the terms of the GNU General Public License as published by the Free Software
Foundation; either version 3 of the License, or (at your option) any later
version.

This program is distributed in the hope that it will be useful, but WITHOUT
ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS
FOR A PARTICULAR PURPOSE. See the GNU General Public License for more
details.

You should have received a copy of the GNU General Public License along with
this program; if not, see <http://www.gnu.org/licenses/>.
