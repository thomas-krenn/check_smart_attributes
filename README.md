# check_smart_attributes - Nagios/Icinga plugin to check SMART attributes

1. [Usage](#Usage)
2. [Synopsis](#Synopsis)
3. [Requirements](#Requirements)
4. [Installation](#Installation)
5. [Mailing List](#MailingList)
6. [Configuration](#Configuration)
    * 6.1. [Device(s) to check](#Devicestocheck)
		* 6.1.1. [NVMe devices](#NVMedevices)
		* 6.1.2. [Devices with LSI RAID controllers](#DeviceswithLSIRAIDcontrollers)
		* 6.1.3. [Devices with Adaptec RAID controllers](#DeviceswithAdaptecRAIDcontrollers)
		* 6.1.4. [Devices with hpsa/cciss based RAID controllers](#DeviceswithhpsaccissbasedRAIDcontrollers)
		* 6.1.5. [Devices with 3ware RAID controllers](#Deviceswith3wareRAIDcontrollers)
		* 6.1.6. [SAS devices](#SASdevices)
	* 6.2. [check_smartdb.json](#check_smartdb.json)
        * 6.2.1 [How to add a new drive to smartdb JSON file](#howtoadddrive)
    * 6.3 [check_smartcfg.json](#smartcfg)
    * 6.4 [sudo for Nagios user](#sudo)
7. [License](#license)

##  1. <a name='Usage'></a>Usage
```bash
sudo check_smart_values -dbj <smartdb json file> -d <device path> [-d <device path>]
[-r <regex pattern to find devices>] [-ucfgj <user config json file>]
[-p <path to smartctl>] [-nosudo] [-cu] [-ap] [-s]
[-O <extra options>][ -v|-vv|-vvv] [-h] [-V]
```

##  2. <a name='Synopsis'></a>Synopsis

```bash
$ sudo ./check_smart_attributes -d /dev/sda -dbj ./check_smartdb.json
Critical [sda_UDMA_CRC_Error_Count = Critical]|sda_Media_Wearout_Indicator=097;16;6
sda_Host_Writes_32MiB=517197 sda_Host_Reads_32MiB=395442
```

##  3. <a name='Requirements'></a>Requirements
The following software is required for `check_smart_attributes`:
* Perl
  * Getopt::Long
  * JSON
  * Storable qw(dclone);
* smartctl

##  4. <a name='Installation'></a>Installation
On Ubuntu use this command to install smartmontools including smartctl and the
Perl library for parsing JSON:
```bash
$ sudo apt-get install smartmontools libjson-perl
```

##  5. <a name='MailingList'></a>Mailing List
* Pleas add new issues via github at https://github.com/thomas-krenn/check_smart_attributes
* The thomas-krenn mailing list archive is at http://lists.thomas-krenn.com/pipermail/tk-monitoring-plugins-user/

##  6. <a name='Configuration'></a>Configuration

###  6.1. <a name='Devicestocheck'></a>Device(s) to check
The `-d` option specifies the devices to check. If multiple devices should be
checked, specify the option multiple times: `-d /dev/sda -d /dev/sdb`
Then multiple devices can be monitored with one check.

####  6.1.1. <a name='NVMedevices'></a>NVMe devices
NVMe devices are easier to check as they are using generic device attributes and don't require a specific smartdb entry.
You can check all NVMe of a system with the regex device check feature:
```bash
$ sudo ./check_smart_attributes -dbj check_smartdb.json -r '/dev/nvme[0-9]+'
OK (nvme0, nvme3, nvme2, nvme1) |'nvme0_Temperature'=32;50;60 'nvme0_Available_Spare'=100;16;11 'nvme0_Percentage_Used'=0;85;90 'nvme0_Data_Units_Read'=0.036864 'nvme0_Data_Units_Written'=0 'nvme0_Host_Read_Commands'=2181 'nvme0_Host_Write_Commands'=0 'nvme0_Power_Cycles'=2 'nvme0_Power_On_Hours'=3 'nvme3_Temperature'=32;50;60 'nvme3_Available_Spare'=100;16;11 'nvme3_Percentage_Used'=0;85;90 'nvme3_Data_Units_Read'=0.033792 'nvme3_Data_Units_Written'=0 'nvme3_Host_Read_Commands'=2973 'nvme3_Host_Write_Commands'=0 'nvme3_Power_Cycles'=3 'nvme3_Power_On_Hours'=3 'nvme2_Temperature'=32;50;60 'nvme2_Available_Spare'=100;16;11 'nvme2_Percentage_Used'=0;85;90 'nvme2_Data_Units_Read'=0.029696 'nvme2_Data_Units_Written'=0 'nvme2_Host_Read_Commands'=2129 'nvme2_Host_Write_Commands'=0 'nvme2_Power_Cycles'=4 'nvme2_Power_On_Hours'=3 'nvme1_Temperature'=32;50;60 'nvme1_Available_Spare'=100;16;11 'nvme1_Percentage_Used'=0;85;90 'nvme1_Data_Units_Read'=0.033792 'nvme1_Data_Units_Written'=0 'nvme1_Host_Read_Commands'=2097 'nvme1_Host_Write_Commands'=0 'nvme1_Power_Cycles'=2 'nvme1_Power_On_Hours'=3
```
**Attention:**

Be careful if when specifying the NVMe device, as smartcl presents different output if the device or the device namespace is used:
* Cf. https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=924892
* Smartctl 6.6 uses selected namespace to read SMART/Health and Error logs.
* Smartctl 7.0 always uses broadcast namespace.


####  6.1.2. <a name='DeviceswithLSIRAIDcontrollers'></a>Devices with LSI RAID controllers
For devices behind LSI controllers use the megaraid device string and the
corresponding device path. You can get the DID with
```bash
$ sudo storcli64 /c0 /eall/sall show
```
From this output use the number in the DID column. Then call the plugin:
```bash
$ sudo ./check_smart_attributes -dbj check_smartdb.json -d megaraid4,/dev/sda
```

####  6.1.3. <a name='DeviceswithAdaptecRAIDcontrollers'></a>Devices with Adaptec RAID controllers
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

####  6.1.4. <a name='DeviceswithhpsaccissbasedRAIDcontrollers'></a>Devices with hpsa/cciss based RAID controllers
For devices behind hpsa/cciss based RAID controller, you'll mostly find in
HP Hardware, you can use 'cciss,<N>_/dev/sg<X>' for hpsa or
'cciss,<N>_/dev/cciss/c<X>d<N>' for cciss where <N> is the number
of the device and <X> the RAID controller e.g.:
```bash
$ sudo ./check_smart_attributes -dbj check_smartdb.json -d cciss,1_/dev/sg2
```

####  6.1.5. <a name='Deviceswith3wareRAIDcontrollers'></a>Devices with 3ware RAID controllers
For devices behind 3ware RAID controllers the 3ware device ID must be
used with the corresponding tw device, e.g.:
```bash
$ sudo ./check_smart_attributes -dbj check_smartdb.json -d 3ware,8,/dev/twa0
```
####  6.1.6. <a name='SASdevices'></a>SAS devices
As attributes for SAS are not model specific the generic SAS entry in the smartdb JSON file
is used:
```bash
$ sudo ./check_smart_attributes -dbj check_smartdb.json -d /dev/sda
```

###  6.2. <a name='check_smartdb.json'></a>check_smartdb.json
The smartdb JSON file specifies the smart attribute configuration for a device.
It contains the configuration which value (VALUE or RAW_VALUE) must be taken for
a smart attribute ID. The config is checked on the basis of device and model
strings given by smartctl.

__If your device is not listed in the config please
study the device specification and add the id--attribute mapping.__

Attributes for NVMe and SAS devices are not model specific, therefore only one
generic NMVe and one SAS entry is present in the smartdb JSON file. If the device
is set up as 'nvme[a-z0-9]+' then NVMe specific parsing of attributes
is done. If the smart output contains "Transport protocol: SAS" then the SAS specific
parsing is done. 
The generic entry is necessary to enable default thresholds and
performance values for NVMe and SAS devices. Moreover now with the generic entry
users can still override the smartdb entry with the 'ucfgj' command line option.
This is way better than having hardcoded parsing of NVMe or SAS attributes in the plugin.

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

**Attention**
If "Threshs" is not specified in the smartdb JSON file then the corresponding SMART
attribute is not checked.

#### 6.2.1 <a name='howtoadddrive'></a>How to add a new drive to smartdb JSON file
1. The drive should be in the smartmontools database, because then you can rely on the output of smartctl.
Therefore it is important to check the information section of "smartctl -a" for "Device is:":
```
$ sudo smartctl -a /dev/sda | grep "Device is"
Device is:        In smartctl database [for details use: -P show]
```

2. Next add the new device to the check_smartdb.json file
- The "Device" key of the JSON entry has to match the "Device Model" of the smartctl output. This is how the plugin (check_smartdb.json) and smartmontools (smartctl) are connected.
- The "ID#" defines which value of smartctl ("VALUE" or "RAW_VALUE") has to be checked by the plugin. As the plugin was initially made for SSDs this is the key step to get a proper monitoring. Each SSD has it's own SMART specification therefore one must check the specification to get to know if "VALUE" or "RAW_VALUE" is relevant. To sum it up: without SMART specific it is not possible to form a correct entry in the check_smartdb.json file of the plugin.

3. After adding the device and checking the corresponding SMART specification, define in the check_smartdb.json file which attributes to monitor 
- The "Threshs" key of the JSON entry defines which attributes to monitor. "Threshs" defines thresholds for the values ("VALUE" or "RAW_VALUE") defined by the corresponding "ID#" entry.
Let's take an example: 
The following entry says:
    * For smart attribute "1" of smartctl take into account: VALUE
    * For smart attribute "3" of smartctl take into account: VALUE
    * For smart attribute "5" of smartctl take into account: RAW_VALUE
    * For smart attribute "194" of smartctl take into account: RAW_VALUE
```
            "Seagate Exos X" : {
                "Device" : ["Seagate Exos X","ST10000NM0156"],
                "ID#" : {
                    "1" : {"value": "VALUE", "comment": "Raw Read Error Rate"},
                    "3" : {"value": "VALUE", "comment": "Spin Up Time"},
                    "4" : {"value": "RAW_VALUE", "comment": "Start Stop Count"},
                    "5" : {"value": "RAW_VALUE", "comment": "Re-allocated Sector Count"},
                    "194" : {"value": "RAW_VALUE", "comment": "Temperature Celsius"}
                }
            }
```
- And the corresponding thresholds:
    * For smart attribute "1" of smartctl, if VALUE (see "ID#" above) is smaller than 62 -> WARNING, if smaller than 52 -> CRITICAL
    * For smart attribute "3" of smartctl, if VALUE (see "ID#" above) is smaller than 32 -> WARNING, if smaller than 22 -> CRITICAL
    * For smart attribute "5" of smartctl, if RAW_VALUE (see "ID#" above) is greater than 20 -> WARNING, if greater than 40 -> CRITICAL
    * For smart attribute "194" of smartctl, if RAW_VALUE (see "ID#" above) is greater than 54 -> WARNING, if greater than 60 -> CRITICAL
```
            "Threshs" : {
                "1" : ["62:","52:"],
                "3" : ["32:","22:"],
                "5" : ["20","40"],
                "194" : ["54","60"]
            }
```
**IMPORTANT**: As there is no "Threshs" entry for attribute number "4", this attribute will not be monitored by the plugin!

4.  Define in the check_smartdb.json file which attributes should be printed as performance data
- The "Perfs" key of the JSON entry defines which attributes to print as performance data.
Let's take an example: 
The following entry says:
    * For smart attribute "194" print "RAW_VALUE" as performance data (see "ID#" above)
```
            "Perfs" : ["194"]
```

### 6.3 <a name='smartcfg'></a>check_smartcfg.json (optional)
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

### 6.4 <a name='sudo'></a>sudo for nagios user
The smartctl executable requires root privileges to check a device. In order to
use the plugin with the nagios user, it is recommended to create a sudoers entry
for smartctl:
```
nagios ALL=(root) NOPASSWD:/usr/sbin/smartctl
```

## 7 <a name='license'></a>License
check_smart_attributes: Nagios/Icinga plugin to check smart attributes with
smartctl.

Copyright (C) 2020 Thomas-Krenn.AG,

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
