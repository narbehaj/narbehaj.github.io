# Check Server’s Disk Health Behind Hardware RAID

![](assets/img/drive.jpg)

There are many disk monitoring tools available but what I use for almost everywhere is the SMART tool (`smartmontools`) which has a [support of NVMe storages](http://support of NVMe storages) too.

Installation is easy, use your package manager to find it. For Debian/Ubuntu:

```
sudo apt install smartmontools
```

SMART has support for hardware RAID controllers, that means if your  disks are behind a hardware RAID controller, SMART can also detect and  extract information from the disks.

```
narbeh@server1:~$ sudo fdisk -l
Disk /dev/sda: 893.8 GiB, 959656755200 bytes, 1874329600 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 262144 bytes / 262144 bytes
Disklabel type: dos
Disk identifier: 0x45f30bb0

Device     Boot Start        End    Sectors   Size Id Type
/dev/sda1        2048 1874329566 1874327519 893.8G 83 Linux
```

Now if I check the health of the `/dev/sda` I’ll get an error:

```
narbeh@server1:~$ sudo smartctl -H /dev/sda
smartctl 6.6 2016-05-31 r4324 [x86_64-linux-5.3.0-28-generic] (local build)
Copyright (C) 2002-16, Bruce Allen, Christian Franke, www.smartmontools.org

Smartctl open device: /dev/sda failed: DELL or MegaRaid controller, please try adding '-d megaraid,N'
```

There is a hint that smartctl that pointed is, we might use MegaRaid controller, so let’s use it as our identifier. The `-i` here provides information about the disk:

```
narbeh@server1:~$ sudo smartctl -i -d megaraid,0 /dev/sda
smartctl 6.6 2016-05-31 r4324 [x86_64-linux-5.3.0-28-generic] (local build)
Copyright (C) 2002-16, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF INFORMATION SECTION ===
Device Model:     SSDSC2KG960G8R
Serial Number:    BTYG92320BR4960GWA
LU WWN Device Id: 5 5cd2e4 150acebac
Add. Product Id:  DELL(tm)
Firmware Version: XCV1DL63
User Capacity:    960,197,124,096 bytes [960 GB]
Sector Sizes:     512 bytes logical, 4096 bytes physical
Rotation Rate:    Solid State Device
Form Factor:      2.5 inches
Device is:        Not in smartctl database [for details use: -P showall]
ATA Version is:   ACS-3 T13/2161-D revision 5
SATA Version is:  SATA 3.2, 6.0 Gb/s (current: 6.0 Gb/s)
Local Time is:    Fri Mar 27 14:22:36 2020 CET
SMART support is: Available - device has SMART capability.
SMART support is: Enabled
```

 The reason I used `megaraid,0` because the MegaRaid has 2 slots for my 2 disks. I have 2 disks so the second slot will be `megaraid,1`:

```
narbeh@server1:~$ sudo smartctl -i -d megaraid,1 /dev/sda
smartctl 6.6 2016-05-31 r4324 [x86_64-linux-5.3.0-28-generic] (local build)
Copyright (C) 2002-16, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF INFORMATION SECTION ===
Device Model:     SSDSC2KG960G8R
Serial Number:    BTYG92320BDX960CGN
LU WWN Device Id: 5 5cd2e4 150acec9f
Add. Product Id:  DELL(tm)
Firmware Version: XCV1DL63
User Capacity:    960,197,124,096 bytes [960 GB]
Sector Sizes:     512 bytes logical, 4096 bytes physical
Rotation Rate:    Solid State Device
Form Factor:      2.5 inches
Device is:        Not in smartctl database [for details use: -P showall]
ATA Version is:   ACS-3 T13/2161-D revision 5
SATA Version is:  SATA 3.2, 6.0 Gb/s (current: 6.0 Gb/s)
Local Time is:    Fri Mar 27 14:24:19 2020 CET
SMART support is: Available - device has SMART capability.
SMART support is: Enabled
```

#### Getting The Health Status

There are many ways to check the health status, first, let’s check  the overall details about our disks. I will only share the first slot of the RAID controller, this can be your JBOD disks so no need to use `-d` argument.

The `-a` argument shows all the information regarding the disk:

```
narbeh@server1:~$ sudo smartctl -a -d megaraid,0 /dev/sda
smartctl 6.6 2016-05-31 r4324 [x86_64-linux-5.3.0-28-generic] (local build)
Copyright (C) 2002-16, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF INFORMATION SECTION ===
Device Model:     SSDSC2KG960G8R
Serial Number:    BTYG92320RT4960CGN
LU WWN Device Id: 5 5cd2e4 150acebac
Add. Product Id:  DELL(tm)
Firmware Version: XCV1DL63
User Capacity:    960,197,124,096 bytes [960 GB]
Sector Sizes:     512 bytes logical, 4096 bytes physical
Rotation Rate:    Solid State Device
Form Factor:      2.5 inches
Device is:        Not in smartctl database [for details use: -P showall]
ATA Version is:   ACS-3 T13/2161-D revision 5
SATA Version is:  SATA 3.2, 6.0 Gb/s (current: 6.0 Gb/s)
Local Time is:    Fri Mar 27 14:36:32 2020 CET
SMART support is: Available - device has SMART capability.
SMART support is: Enabled

=== START OF READ SMART DATA SECTION ===
SMART Status not supported: ATA return descriptor not supported by controller firmware
SMART overall-health self-assessment test result: PASSED
Warning: This result is based on an Attribute check.

General SMART Values:
Offline data collection status:  (0x02)	Offline data collection activity
					was completed without error.
					Auto Offline Data Collection: Disabled.
Self-test execution status:      (   0)	The previous self-test routine completed
					without error or no self-test has ever 
					been run.
Total time to complete Offline 
data collection: 		(    2) seconds.
Offline data collection
capabilities: 			 (0x79) SMART execute Offline immediate.
					No Auto Offline data collection support.
					Suspend Offline collection upon new
					command.
					Offline surface scan supported.
					Self-test supported.
					Conveyance Self-test supported.
					Selective Self-test supported.
SMART capabilities:            (0x0003)	Saves SMART data before entering
					power-saving mode.
					Supports SMART auto save timer.
Error logging capability:        (0x01)	Error logging supported.
					General Purpose Logging supported.
Short self-test routine 
recommended polling time: 	 (   1) minutes.
Extended self-test routine
recommended polling time: 	 (  60) minutes.
Conveyance self-test routine
recommended polling time: 	 (  60) minutes.
SCT capabilities: 	       (0x003d)	SCT Status supported.
					SCT Error Recovery Control supported.
					SCT Feature Control supported.
					SCT Data Table supported.

SMART Attributes Data Structure revision number: 1
Vendor Specific SMART Attributes with Thresholds:
ID# ATTRIBUTE_NAME          FLAG     VALUE WORST THRESH TYPE      UPDATED  WHEN_FAILED RAW_VALUE
  1 Raw_Read_Error_Rate     0x000e   130   130   039    Old_age   Always       -       4294967295
  5 Reallocated_Sector_Ct   0x0033   100   100   001    Pre-fail  Always       -       0
  9 Power_On_Hours          0x0032   100   100   000    Old_age   Always       -       4322
 12 Power_Cycle_Count       0x0032   100   100   000    Old_age   Always       -       37
 13 Read_Soft_Error_Rate    0x001e   090   089   000    Old_age   Always       -       56173877264383
170 Unknown_Attribute       0x0033   100   100   010    Pre-fail  Always       -       0
174 Unknown_Attribute       0x0032   100   100   000    Old_age   Always       -       31
179 Used_Rsvd_Blk_Cnt_Tot   0x0033   100   100   010    Pre-fail  Always       -       0
180 Unused_Rsvd_Blk_Cnt_Tot 0x0032   100   100   000    Old_age   Always       -       12313
181 Program_Fail_Cnt_Total  0x003a   100   100   000    Old_age   Always       -       0
182 Erase_Fail_Count_Total  0x003a   100   100   000    Old_age   Always       -       0
184 End-to-End_Error        0x0032   100   100   000    Old_age   Always       -       0
194 Temperature_Celsius     0x0022   100   100   000    Old_age   Always       -       24
195 Hardware_ECC_Recovered  0x0032   100   100   000    Old_age   Always       -       0
197 Current_Pending_Sector  0x0012   100   100   000    Old_age   Always       -       0
198 Offline_Uncorrectable   0x0010   100   100   000    Old_age   Offline      -       0
199 UDMA_CRC_Error_Count    0x003e   100   100   000    Old_age   Always       -       0
201 Unknown_SSD_Attribute   0x0033   100   100   010    Pre-fail  Always       -       219052181922
202 Unknown_SSD_Attribute   0x0027   100   100   000    Pre-fail  Always       -       0
225 Unknown_SSD_Attribute   0x0032   100   100   000    Old_age   Always       -       469625
226 Unknown_SSD_Attribute   0x0032   100   100   000    Old_age   Always       -       143
227 Unknown_SSD_Attribute   0x0032   100   100   000    Old_age   Always       -       42
228 Power-off_Retract_Count 0x0032   100   100   000    Old_age   Always       -       259013
232 Available_Reservd_Space 0x0033   100   100   010    Pre-fail  Always       -       0
233 Media_Wearout_Indicator 0x0032   100   100   000    Old_age   Always       -       469625
234 Unknown_Attribute       0x0032   100   100   000    Old_age   Always       -       0
241 Total_LBAs_Written      0x0032   100   100   000    Old_age   Always       -       469625
242 Total_LBAs_Read         0x0032   100   100   000    Old_age   Always       -       388310
245 Unknown_Attribute       0x0032   100   100   000    Old_age   Always       -       100

SMART Error Log not supported

SMART Self-test log structure revision number 1
Num  Test_Description    Status                  Remaining  LifeTime(hours)  LBA_of_first_error
# 1  Short offline       Completed without error       00%      4319         -
# 2  Short offline       Completed without error       00%      4319         -
# 3  Short offline       Completed without error       00%      4319         -
# 4  Extended offline    Completed without error       00%         5         -
# 5  Short offline       Completed without error       00%         4         -
# 6  Extended offline    Completed without error       00%         1         -
# 7  Short offline       Completed without error       00%         1         -

Read SMART Selective Self-test Log failed: megasas_cmd result: 0.0 = 0/45
```

There is also `-H` argument which reports the overall health status:

```
narbeh@server1:~$ sudo smartctl -H -d megaraid,0 /dev/sda
smartctl 6.6 2016-05-31 r4324 [x86_64-linux-5.3.0-28-generic] (local build)
Copyright (C) 2002-16, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF READ SMART DATA SECTION ===
SMART Status not supported: ATA return descriptor not supported by controller firmware
SMART overall-health self-assessment test result: PASSED
Warning: This result is based on an Attribute check.
```

And also there is an argument which you can see the self-test as below:

```
narbeh@server1:~$ sudo smartctl -l selftest -d megaraid,0 /dev/sda
smartctl 6.6 2016-05-31 r4324 [x86_64-linux-5.3.0-28-generic] (local build)
Copyright (C) 2002-16, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF READ SMART DATA SECTION ===
SMART Self-test log structure revision number 1
Num  Test_Description    Status                  Remaining  LifeTime(hours)  LBA_of_first_error
# 1  Short offline       Completed without error       00%      4319         -
# 2  Short offline       Completed without error       00%      4319         -
# 3  Short offline       Completed without error       00%      4319         -
# 4  Extended offline    Completed without error       00%         5         -
# 5  Short offline       Completed without error       00%         4         -
# 6  Extended offline    Completed without error       00%         1         -
# 7  Short offline       Completed without error       00%         1         -
```

#### Adding Reports To Zabbix

If you are using Zabbix monitoring system, you can also use the plugin which developed for it. 

> https://github.com/v-zhuravlev/zbx-smartctl 