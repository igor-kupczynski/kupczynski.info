---
title: Diagnose SSD failures
tags:
- fail
aliases:
- /2018/11/02/ssd-failure.html
---
How to minitor health of SSD drives, case study on my failed Kingston SUV500MS480G.

I use my old Samsung Series 7 ultrabook for some personal tasks. It originally
had 120 GB SSD, but I've upgraded it recently to Kingston 480GB SSD.

I've first installed Ubuntu 18.04, but then upgraded to 18.10. Everything seem
fine, but after few hours of usage and a restart it started complaining about
invalid file permissions. I don't have logs, as at this point I though &#x2014;
ok, a botched upgrade &#x2014; we can reinstall, no big deal. Reinstalling is
not a big deal, because most of the data is in the cloud already, and I maintain
a collection of scripts to setup the software I need.

Then after a week, ubuntu failed to start. I've boot a live usb, mounted the
drive and `fsck.ext4` it. There were some errors, but not too many. After that,
ubuntu started again, but only in a text mode &#x2014; GDM failed to started.

At this point, I've started suspecting the new SSD. But how to check if your SSD
maybe faulty?

# Badblocks?<a id="sec-1"></a>

The SSDs use [wear leveling](https://en.wikipedia.org/wiki/Wear_leveling),
contain redundant blocks and employ various techniques to improve their
reliability and increase their live span. It means that the physical address and
the logical one exposed to the operating system is different. The translation
being done by the disk controller.

It also means that the traditional techniques, like using
[`badblocks`](https://en.wikipedia.org/wiki/Badblocks) are not really useful
&#x2014; the disk controller may detect a read failure and reroute the block
somewhere else.

Not only it doesn't help, it may even hurt the drive as writing and reading the
same block over and over again increases the wear of the drive.

There is not that much info on the internet on the usage of `badblocks` with
SSDs. [This
stackexchange](https://unix.stackexchange.com/questions/93686/can-i-prove-that-an-ssd-is-broken#comment475560_93691)
comment is a little gem

> `badblocks` is probably not the best tool to unleash on an SSD as internally,
> the a read-write cycle of a single (small 512 Byte) `badblocks` level-block
> will cause the SSD to reallocate/erase a large (512 KiB) SSD-level block again
> and again, leading to excessive wear and tear (See [The Anatomy of an
> SSD](https://www.anandtech.com/show/2738/5)). One should probably set the
> blocksize: `badblocks -b 524288`. A supersimple test is trying to read the
> entire SSD using `dd if=/dev/sda of=dev/null`. There may be vendor-specific
> tools too, check the Internet. My Samsung diagnostics didn't bark though.
> 
> David Tonhofer
> 

And an answer below

> SSDs manage their own bad blocks internally and also use wear levelling to
> distribute use; the block addresses sent to the system are virtual. Therefore,
> none of those blocks should test bad, and if they do, some functioning of the
> drive has failed.

# S.M.A.R.T<a id="sec-2"></a>

Most modern hard drives have a built-in monitoring system called
[S.M.A.R.T.](https://en.wikipedia.org/wiki/S.M.A.R.T.) It monitors the various
parameters of the drive and exposes various metrics about the drive. It also
allows to run a self-check test.

To work with S.M.A.R.T. on linux we use
[smartmontools](https://www.smartmontools.org/).

First we need to install them

```sh
sudo apt install smartmontools
```

Then we can use it to display basic info

```sh
sudo smartctl --info /dev/sda
```

More details are available with the `--all` (or `-a`).

## Testing the drive<a id="sec-2-1"></a>

S.M.A.R.T. allows to run a self-test on the drive. There are three types of the
test &#x2014; short, long and captive (details in the manual). I was interested
in the long one

```sh
sudo smartctl -t long /dev/sda
```

It gives you an estimation when the test will be done. Then you can query the
result of the test (and other attributes recorded by the drive).

```sh
sudo smartctl -a /dev/sda
```

I've run the test a couple of times with few hours between them. I've also
accessed the data between the runs (I was curious how critical the failure was).

Here is one of the early results:

```
smartctl 6.6 2017-11-05 r4594 [x86_64-linux-4.17.0-kali1-amd64] (local build)
Copyright (C) 2002-17, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF INFORMATION SECTION ===
Device Model:     KINGSTON SUV500MS480G
Serial Number:    50026B77821C2188
LU WWN Device Id: 5 0026b7 7821c2188
Firmware Version: 003056RA
User Capacity:    480,103,981,056 bytes [480 GB]
Sector Sizes:     512 bytes logical, 4096 bytes physical
Rotation Rate:    Solid State Device
Form Factor:      2.5 inches
Device is:        Not in smartctl database [for details use: -P showall]
ATA Version is:   ACS-4, ATA8-ACS T13/1699-D revision 6
SATA Version is:  SATA 3.1, 6.0 Gb/s (current: 6.0 Gb/s)
Local Time is:    Mon Oct 29 22:23:43 2018 UTC
SMART support is: Available - device has SMART capability.
SMART support is: Enabled

=== START OF READ SMART DATA SECTION ===
SMART overall-health self-assessment test result: PASSED

General SMART Values:
Offline data collection status:  (0x00)	Offline data collection activity
          was never started.
          Auto Offline Data Collection: Disabled.
Self-test execution status:      (   0)	The previous self-test routine completed
          without error or no self-test has ever 
          been run.
Total time to complete Offline 
data collection: 		(    5) seconds.
Offline data collection
capabilities: 			 (0x71) SMART execute Offline immediate.
          No Auto Offline data collection support.
          Suspend Offline collection upon new
          command.
          No Offline surface scan supported.
          Self-test supported.
          Conveyance Self-test supported.
          Selective Self-test supported.
SMART capabilities:            (0x0003)	Saves SMART data before entering
          power-saving mode.
          Supports SMART auto save timer.
Error logging capability:        (0x01)	Error logging supported.
          General Purpose Logging supported.
Short self-test routine 
recommended polling time: 	 (   2) minutes.
Extended self-test routine
recommended polling time: 	 (   5) minutes.
Conveyance self-test routine
recommended polling time: 	 (   0) minutes.
SCT capabilities: 	       (0x003d)	SCT Status supported.
          SCT Error Recovery Control supported.
          SCT Feature Control supported.
          SCT Data Table supported.

SMART Attributes Data Structure revision number: 48
Vendor Specific SMART Attributes with Thresholds:
ID# ATTRIBUTE_NAME          FLAG     VALUE WORST THRESH TYPE      UPDATED  WHEN_FAILED RAW_VALUE
  1 Raw_Read_Error_Rate     0x002f   100   100   000    Pre-fail  Always       -       14737
  5 Reallocated_Sector_Ct   0x0033   100   100   010    Pre-fail  Always       -       2
  9 Power_On_Hours          0x0032   100   100   000    Old_age   Always       -       81
 12 Power_Cycle_Count       0x0032   100   100   000    Old_age   Always       -       67
100 Unknown_Attribute       0x0032   100   100   000    Old_age   Always       -       116608
101 Unknown_Attribute       0x0032   100   100   000    Old_age   Always       -       18880
170 Unknown_Attribute       0x0032   100   100   000    Old_age   Always       -       2
171 Unknown_Attribute       0x0032   100   100   000    Old_age   Always       -       0
172 Unknown_Attribute       0x0032   100   100   000    Old_age   Always       -       0
174 Unknown_Attribute       0x0032   100   100   000    Old_age   Always       -       7
175 Program_Fail_Count_Chip 0x0032   100   100   000    Old_age   Always       -       0
176 Erase_Fail_Count_Chip   0x0032   100   100   000    Old_age   Always       -       0
177 Wear_Leveling_Count     0x0032   100   100   000    Old_age   Always       -       42
178 Used_Rsvd_Blk_Cnt_Chip  0x0002   100   100   000    Old_age   Always       -       1
180 Unused_Rsvd_Blk_Cnt_Tot 0x0002   100   100   000    Old_age   Always       -       1288
183 Runtime_Bad_Block       0x0032   100   100   000    Old_age   Always       -       0
187 Reported_Uncorrect      0x0033   100   100   000    Pre-fail  Always       -       2
194 Temperature_Celsius     0x0022   034   100   000    Old_age   Always       -       34 (Min/Max 22/38)
195 Hardware_ECC_Recovered  0x0032   100   100   000    Old_age   Always       -       14737
196 Reallocated_Event_Count 0x0032   100   100   000    Old_age   Always       -       2
197 Current_Pending_Sector  0x0032   100   100   000    Old_age   Always       -       0
199 UDMA_CRC_Error_Count    0x0012   100   100   000    Old_age   Always       -       0
201 Unknown_SSD_Attribute   0x0032   100   100   000    Old_age   Always       -       0
204 Soft_ECC_Correction     0x0032   100   100   000    Old_age   Always       -       14735
231 Temperature_Celsius     0x0032   100   100   000    Old_age   Always       -       0
233 Media_Wearout_Indicator 0x0032   100   100   000    Old_age   Always       -       377
234 Unknown_Attribute       0x0032   100   100   000    Old_age   Always       -       221
241 Total_LBAs_Written      0x0032   100   100   000    Old_age   Always       -       264
242 Total_LBAs_Read         0x0032   100   100   000    Old_age   Always       -       96
250 Read_Error_Retry_Rate   0x0032   100   100   000    Old_age   Always       -       14735

SMART Error Log Version: 0
No Errors Logged

SMART Self-test log structure revision number 0
Warning: ATA Specification requires self-test log structure revision number = 1
Num  Test_Description    Status                  Remaining  LifeTime(hours)  LBA_of_first_error
# 1  Extended offline    Completed without error       00%        80         -
# 2  Short offline       Completed without error       00%        80         -

SMART Selective self-test log data structure revision number 0
Note: revision number not 1 implies that no selective self-test has ever been run
 SPAN  MIN_LBA  MAX_LBA  CURRENT_TEST_STATUS
    1        0        0  Not_testing
    2        0        0  Not_testing
    3        0        0  Not_testing
    4        0        0  Not_testing
    5        0        0  Not_testing
Selective self-test flags (0x0):
  After scanning selected spans, do NOT read-scan remainder of disk.
If Selective self-test is pending on power-up, resume after 0 minute delay.
```

We're interested in the table with attributes. Each manufacturer may define the
attributes a bit differently, so it good to go to the producer website and check
the spec.

In my case I was interested in:

```
5 Reallocated_Sector_Ct   0x0033   100   100   010    Pre-fail  Always       -       2
```

Which tells us there are already two "bad blocks" which needed to be
reallocated. This is not good for such a young drive (81h).

Even more worrying is, if that count increases. This means the drive is failing.

Here is the output from a test I've taken few hours later:

```
smartctl 6.6 2017-11-05 r4594 [x86_64-linux-4.17.0-kali1-amd64] (local build)
Copyright (C) 2002-17, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF INFORMATION SECTION ===
Device Model:     KINGSTON SUV500MS480G
Serial Number:    50026B77821C2188
LU WWN Device Id: 5 0026b7 7821c2188
Firmware Version: 003056RA
User Capacity:    480,103,981,056 bytes [480 GB]
Sector Sizes:     512 bytes logical, 4096 bytes physical
Rotation Rate:    Solid State Device
Form Factor:      2.5 inches
Device is:        Not in smartctl database [for details use: -P showall]
ATA Version is:   ACS-4, ATA8-ACS T13/1699-D revision 6
SATA Version is:  SATA 3.1, 6.0 Gb/s (current: 6.0 Gb/s)
Local Time is:    Tue Oct 30 21:06:08 2018 UTC
SMART support is: Available - device has SMART capability.
SMART support is: Enabled

=== START OF READ SMART DATA SECTION ===
SMART overall-health self-assessment test result: PASSED

General SMART Values:
Offline data collection status:  (0x00)	Offline data collection activity
          was never started.
          Auto Offline Data Collection: Disabled.
Self-test execution status:      (   0)	The previous self-test routine completed
          without error or no self-test has ever 
          been run.
Total time to complete Offline 
data collection: 		(    5) seconds.
Offline data collection
capabilities: 			 (0x71) SMART execute Offline immediate.
          No Auto Offline data collection support.
          Suspend Offline collection upon new
          command.
          No Offline surface scan supported.
          Self-test supported.
          Conveyance Self-test supported.
          Selective Self-test supported.
SMART capabilities:            (0x0003)	Saves SMART data before entering
          power-saving mode.
          Supports SMART auto save timer.
Error logging capability:        (0x01)	Error logging supported.
          General Purpose Logging supported.
Short self-test routine 
recommended polling time: 	 (   2) minutes.
Extended self-test routine
recommended polling time: 	 (   5) minutes.
Conveyance self-test routine
recommended polling time: 	 (   0) minutes.
SCT capabilities: 	       (0x003d)	SCT Status supported.
          SCT Error Recovery Control supported.
          SCT Feature Control supported.
          SCT Data Table supported.

SMART Attributes Data Structure revision number: 48
Vendor Specific SMART Attributes with Thresholds:
ID# ATTRIBUTE_NAME          FLAG     VALUE WORST THRESH TYPE      UPDATED  WHEN_FAILED RAW_VALUE
  1 Raw_Read_Error_Rate     0x002f   100   100   000    Pre-fail  Always       -       15462
  5 Reallocated_Sector_Ct   0x0033   099   099   010    Pre-fail  Always       -       25
  9 Power_On_Hours          0x0032   100   100   000    Old_age   Always       -       93
 12 Power_Cycle_Count       0x0032   100   100   000    Old_age   Always       -       73
100 Unknown_Attribute       0x0032   100   100   000    Old_age   Always       -       119168
101 Unknown_Attribute       0x0032   100   100   000    Old_age   Always       -       18944
170 Unknown_Attribute       0x0032   100   100   000    Old_age   Always       -       25
171 Unknown_Attribute       0x0032   100   100   000    Old_age   Always       -       0
172 Unknown_Attribute       0x0032   100   100   000    Old_age   Always       -       0
174 Unknown_Attribute       0x0032   100   100   000    Old_age   Always       -       9
175 Program_Fail_Count_Chip 0x0032   100   100   000    Old_age   Always       -       0
176 Erase_Fail_Count_Chip   0x0032   100   100   000    Old_age   Always       -       0
177 Wear_Leveling_Count     0x0032   100   100   000    Old_age   Always       -       43
178 Used_Rsvd_Blk_Cnt_Chip  0x0002   100   100   000    Old_age   Always       -       3
180 Unused_Rsvd_Blk_Cnt_Tot 0x0002   100   100   000    Old_age   Always       -       1265
183 Runtime_Bad_Block       0x0032   100   100   000    Old_age   Always       -       0
187 Reported_Uncorrect      0x0033   100   100   000    Pre-fail  Always       -       104
194 Temperature_Celsius     0x0022   033   100   000    Old_age   Always       -       33 (Min/Max 22/38)
195 Hardware_ECC_Recovered  0x0032   100   100   000    Old_age   Always       -       15462
196 Reallocated_Event_Count 0x0032   100   100   000    Old_age   Always       -       25
197 Current_Pending_Sector  0x0032   100   100   000    Old_age   Always       -       0
199 UDMA_CRC_Error_Count    0x0012   100   100   000    Old_age   Always       -       0
201 Unknown_SSD_Attribute   0x0032   100   100   000    Old_age   Always       -       102
204 Soft_ECC_Correction     0x0032   100   100   000    Old_age   Always       -       15358
231 Temperature_Celsius     0x0032   100   100   000    Old_age   Always       -       0
233 Media_Wearout_Indicator 0x0032   100   100   000    Old_age   Always       -       381
234 Unknown_Attribute       0x0032   100   100   000    Old_age   Always       -       223
241 Total_LBAs_Written      0x0032   100   100   000    Old_age   Always       -       264
242 Total_LBAs_Read         0x0032   100   100   000    Old_age   Always       -       163
250 Read_Error_Retry_Rate   0x0032   100   100   000    Old_age   Always       -       15358

SMART Error Log Version: 0
ATA Error Count: 204 (device log contains only the most recent five errors)
  CR = Command Register [HEX]
  FR = Features Register [HEX]
  SC = Sector Count Register [HEX]
  SN = Sector Number Register [HEX]
  CL = Cylinder Low Register [HEX]
  CH = Cylinder High Register [HEX]
  DH = Device/Head Register [HEX]
  DC = Device Command Register [HEX]
  ER = Error register [HEX]
  ST = Status register [HEX]
Powered_Up_Time is measured from power on, and printed as
DDd+hh:mm:SS.sss where DD=days, hh=hours, mm=minutes,
SS=sec, and sss=millisec. It "wraps" after 49.710 days.

Error 204 occurred at disk power-on lifetime: 0 hours (0 days + 0 hours)
  When the command that caused the error occurred, the device was active or idle.

  After command completion occurred, registers were:
  ER ST SC SN CL CH DH
  -- -- -- -- -- -- --
  40 51 08 70 0a e8 40  Error: UNC at LBA = 0x00e80a70 = 15207024

  Commands leading to the command that caused the error were:
  CR FR SC SN CL CH DH DC   Powered_Up_Time  Command/Feature_Name
  -- -- -- -- -- -- -- --  ----------------  --------------------
  60 0c 08 70 0a e8 40 00      00:18:17.111  READ FPDMA QUEUED

Error 203 occurred at disk power-on lifetime: 0 hours (0 days + 0 hours)
  When the command that caused the error occurred, the device was active or idle.

  After command completion occurred, registers were:
  ER ST SC SN CL CH DH
  -- -- -- -- -- -- --
  40 51 08 70 0a e8 40  Error: UNC at LBA = 0x00e80a70 = 15207024

  Commands leading to the command that caused the error were:
  CR FR SC SN CL CH DH DC   Powered_Up_Time  Command/Feature_Name
  -- -- -- -- -- -- -- --  ----------------  --------------------
  60 09 08 70 0a e8 40 00      00:18:16.728  READ FPDMA QUEUED

Error 202 occurred at disk power-on lifetime: 0 hours (0 days + 0 hours)
  When the command that caused the error occurred, the device was active or idle.

  After command completion occurred, registers were:
  ER ST SC SN CL CH DH
  -- -- -- -- -- -- --
  40 51 08 58 0a e8 40  Error: UNC at LBA = 0x00e80a58 = 15207000

  Commands leading to the command that caused the error were:
  CR FR SC SN CL CH DH DC   Powered_Up_Time  Command/Feature_Name
  -- -- -- -- -- -- -- --  ----------------  --------------------
  60 01 08 58 0a e8 40 00      00:18:16.341  READ FPDMA QUEUED

Error 201 occurred at disk power-on lifetime: 0 hours (0 days + 0 hours)
  When the command that caused the error occurred, the device was active or idle.

  After command completion occurred, registers were:
  ER ST SC SN CL CH DH
  -- -- -- -- -- -- --
  40 51 08 58 0a e8 40  Error: UNC at LBA = 0x00e80a58 = 15207000

  Commands leading to the command that caused the error were:
  CR FR SC SN CL CH DH DC   Powered_Up_Time  Command/Feature_Name
  -- -- -- -- -- -- -- --  ----------------  --------------------
  60 12 08 58 0a e8 40 00      00:18:15.976  READ FPDMA QUEUED

Error 200 occurred at disk power-on lifetime: 0 hours (0 days + 0 hours)
  When the command that caused the error occurred, the device was active or idle.

  After command completion occurred, registers were:
  ER ST SC SN CL CH DH
  -- -- -- -- -- -- --
  40 51 08 40 0a e8 40  Error: UNC at LBA = 0x00e80a40 = 15206976

  Commands leading to the command that caused the error were:
  CR FR SC SN CL CH DH DC   Powered_Up_Time  Command/Feature_Name
  -- -- -- -- -- -- -- --  ----------------  --------------------
  60 12 08 40 0a e8 40 00      00:18:15.566  READ FPDMA QUEUED

SMART Self-test log structure revision number 0
Warning: ATA Specification requires self-test log structure revision number = 1
Num  Test_Description    Status                  Remaining  LifeTime(hours)  LBA_of_first_error
# 1  Extended offline    Completed without error       00%        92         -

SMART Selective self-test log data structure revision number 0
Note: revision number not 1 implies that no selective self-test has ever been run
 SPAN  MIN_LBA  MAX_LBA  CURRENT_TEST_STATUS
    1        0        0  Not_testing
    2        0        0  Not_testing
    3        0        0  Not_testing
    4        0        0  Not_testing
    5        0        0  Not_testing
Selective self-test flags (0x0):
  After scanning selected spans, do NOT read-scan remainder of disk.
If Selective self-test is pending on power-up, resume after 0 minute delay.
```

Now, we have 25 reallocated sectors

```
5 Reallocated_Sector_Ct   0x0033   099   099   010    Pre-fail  Always       -       25
```

Plus some scary looking error messages at the bottom of the output.

I think it is time to check the warranty --- the drive is just six weeks old.


**Update** few weeks later I've received a new drive from Kingston. So far so
good. I hope it will last a bit longer ;)


**Update 2021-03-07** Unfortunately the replacement unit I've got from Kingston now exhibits the same symptoms. I've filled a new ticket with the reseller, let's see if they are going to replace it.