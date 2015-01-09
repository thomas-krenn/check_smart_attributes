        = README =

    == Synopsis ==
$ sudo ./check_smart_attributes -d /dev/sda -dbj ./check_smartdb.json
Critical [sda_UDMA_CRC_Error_Count = Critical]|sda_Media_Wearout_Indicator=097;16;6
sda_Host_Writes_32MiB=517197 sda_Host_Reads_32MiB=395442

    == Requirements ==
The following software is required for check_smart_attributes:
* Perl
** Getopt::Long
** Config::JSON
** Storable qw(dclone);
* smartctl

    == Installation ==
On Ubuntu use
  :~$ sudo apt-get install smartmontools libconfig-json-perl
to install smartmontools including smartctl and the Perl library for parsing
JSON config files.

    == Configuration ==
  === Device(s) to check ===
The '-d' option specifies the device to check. If multiple devices should be
checke specify the option multiple times:
  -d /dev/sda -d /dev/sdb
Then multiple devices can be monitored with one check.
  ==== Devices with LSI RAID controllers ====
For devices behind LSI controllers use the megaraid device string and the 
corresponding device path. You can get the DID with
  :~$ sudo storcli64 /c0 /eall/sall show
From this output use the number in the DID column. Then call the plugin:
  :~$ sudo ./check_smart_attributes -dbj check_smartdb.json -d megaraid4,/dev/sda
  ==== Devices with Adaptec RAID controllers ====
For devices behind Adaptec controllers use the sg device string and an extra
option for the device interface:
  -d /dev/sg2 -O sat
  -d /dev/sg2 -O scsi
Please consider, that the extra option is mandatory.

  === check_smartdb.json ===
The smartdb JSON file specifies the smart attribute configuration for a device.
It contains the configuration which value (VALUE or RAW_VALUE) must be taken for
a smart attribute ID. The config is checked on the basis of device and model
strings given by smartctl. If your device is not listed in the config please
study the device specification and add the id--attribute mapping.
The smartdb JSON file also specifies default threshold and performance values.
These values state the Warning/Critical sensor thresholds and the performance
data sensors. The threshold and performance base config can be overwritten with
a user config JSON file (see below).

  === check_smartcfg.json (optional) ===
Specify the path at which the JSON user config file can be found.
The user config can be used to override the threshold and performance base
config. This can be useful if the thresholds for a specific device must
be changed (e.g. showing up a non-critical error).
Note that a given sensor in the user config overwrites the corresponding sensor
in the smartdb base JSON file.

  === sudo for nagios user ===
The smartctl executable requires root privileges to check a device. In order to
use the plugin with the nagios user, it is recommended to create a sudoers entry
for smartctl:
  nagios ALL=(root) NOPASSWD:/usr/sbin/smartctl

    == License ==
check_smart_attributes: Nagios/Icinga plugin to check smart attributes with
smartctl.

Copyright (C) 2015 Thomas-Krenn.AG,

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
