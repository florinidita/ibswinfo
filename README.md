# ibswinfo
Get information from unmanaged NVIDIA Infiniband switches.


## Description

`ibswinfo` is a simple script to get status and monitoring information from
unmanaged NVIDIA Infiniband switches.

NVIDIA Infiniband switches come in two flavors:

* managed switches have their own management controller, which allows
  monitoring fan speeds and temperatures, getting serial numbers, or updating
  firmwares over a variety of protocols (SSH, SNMP, HTTPs...)

* unmanaged switches are just that: unmanaged.

Some in-band management is possible for unmanaged switches with NVIDIA firmware
tools, but the only way to get their status is via the LEDs on their chassis:
they're either green (that's good), or red (that's bad). But you won't know
unless you physically take a look at them, which makes it difficult to get
notifications and alerts when problems occur.

`ibswinfo` helps solve this problem, by leveraging [NVIDIA Firmware Tools
(MFT)](https://network.nvidia.com/products/adapter-software/firmware-tools/) to
allow sysadmins to get more information about their unmanaged Infiniband
switches. It can be used to gather hardware vitals such as fan speeds or
temperatures, and monitor the switches more closely.

As of version [0.6](https://github.com/stanford-rc/ibswinfo/releases/tag/0.6),
`ibswinfo` also allows setting a user-friendly name (node description) for
unmanaged switches. No more endless lists of identical and unhelpful switch
names like "SwitchX - NVIDIA Networking", sysadmins can now customize each
switch by setting a descriptive name, using their model, physical location,
cluster name, etc. and make each switch more easily identifiable.


## Installation

### Dependencies

* [NVIDIA Firmware Tools
  (MFT)](https://network.nvidia.com/products/adapter-software/firmware-tools/)

    \>= 4.18.0

    \>= 4.22.0 for switch name setting support

* [`infiniband-diags`](https://github.com/linux-rdma/rdma-core)
* `bash`, `coreutils`, `awk` and `sed`

### Preparation

`ibswinfo` can address switches by using the virtual devices created by MST,
the NVIDIA Software Tools service, or by LID.

* to use the MST virtual devices, you can start the `mst` service and populate
  entries in `/dev/mst` with:

    ```
    # mst start
    # mst ib add
    ```

    Check that `/dev/mst` contains entries for your unmanaged switches (they
    should look like `/dev/mst/SW_*`), and you can run `ibswinfo` like this:
    ```
    # ./ibswinfo.sh -d /dev/mst/SW_model_node-desc_lid-0x0001
    ```
    or without the `/dev/mst` prefix, using the virtual device name directly:
    ```
    # ./ibswinfo.sh -d SW_model_node-desc_lid-0x0001
    ```

* to address switches by LID, starting the `mst` service is not required. You
  can get currently assigned LIDs with `ibswitches`, and run `ibswinfo`
  directly like this: `./ibswinfo.sh -d lid-16`


## Supported hardware

`ibswinfo` has been tested with the following unmanaged Infiniband switches:
* SB7790 Switch-IB (EDR) (_known limitation: QSFP module temperatures reported
  as 0_)
* SB7890 Switch-IB2 (EDR)
* QM8790 Quantum (HDR)

Limited support is also available for the managed version of those switches:
* SB7800 Switch-IB2 (EDR)
* QM8700 Quantum (HDR)

> [!NOTE]
> If you find other working models, please feel free to open an
[issue](https://github.com/stanford-rc/ibswinfo/issues/new) and
we'll complete the list.


### Available information

* Node description (switch name)
* Part number, serial number
* PSID, GUID, firmware version
* Uptime
* Power supply information (status, consumption, part and serial numbers)
* Temperatures (including QSFP modules temp.)
* Fan speeds and status


## Usage

```none
Usage: ibswinfo.sh -d <device> [-T] [-o <inventory|vitals|status>] [-S <description>]

  global options:
    -d <device>             MST device path ("mst status" shows devices list)
                            or LID (eg. "-d lid-44")
  get info:
    -o <output_category>    Only display inventory|vitals|status information
    -T                      get QSFP modules temperature

  set info:
    -S <description>        set device description (63 char max.)

```

### Default output

By default, `ibswinfo` presents all the available information for a switch in a
table-like output:

```none
# ./ibswinfo.sh -d <device>
=================================================
<node description>
=================================================
part number        | MQM8790-HS2F
serial number      | <redacted>
product name       | Jaguar Unmng IB 200
revision           | AC
ports              | 40
PSID               | MT_0000000063
GUID               | <redacted>
firmware version   | 27.2000.1886
CPLD               | 3
-------------------------------------------------
uptime (d-h:m:s)   | 196d-07:05:40
-------------------------------------------------
PSU0 status        | OK
     P/N           | MTEF-PSF-AC-C
     S/N           | <redacted>
     DC power      | OK
     fan status    | OK
     power (W)     | 64
PSU1 status        | OK
     P/N           | MTEF-PSF-AC-C
     S/N           | <redacted>
     DC power      | OK
     fan status    | OK
     power (W)     | 47
-------------------------------------------------
temperature (C)    | 34
max temp (C)       | 41
warn threshold (C) | 95/105 (low/high)
-------------------------------------------------
fan status         | OK
fan#1 (rpm)        | 5426
fan#2 (rpm)        | 4746
fan#3 (rpm)        | 5426
fan#4 (rpm)        | 4798
fan#5 (rpm)        | 5426
fan#6 (rpm)        | 4815
fan#7 (rpm)        | 5382
fan#8 (rpm)        | 4868
fan#9 (rpm)        | 5471
-------------------------------------------------
```

### Targeted outputs

Specific information can be targeted by choosing the appropriate output type:
`inventory`, `status` or `vitals`.

This can be particularly useful to quickly get a switch's serial number, check
its status to create alerts, or feed hardware metrics to a monitoring system.
Targeted outputs are designed to be parsed for input to other tools.

For instance, to only get hardware vitals, including QSFP temperatures:

```
# ./ibswinfo -d <device> -o vitals -T
uptime (sec)       : 16982312
psu0.power (W)     : 92
psu1.power (W)     : 102
cur.temp (C)       : 73
max.temp (C)       : 80
QSFP#01.temp (C)   : 44
QSFP#02.temp (C)   : 43
QSFP#03.temp (C)   : 43
QSFP#04.temp (C)   : 47
QSFP#05.temp (C)   : 42
QSFP#06.temp (C)   : 46
QSFP#07.temp (C)   : 44
QSFP#08.temp (C)   : 46
QSFP#09.temp (C)   : 44
QSFP#10.temp (C)   : 45
QSFP#11.temp (C)   : 44
QSFP#12.temp (C)   : 44
QSFP#13.temp (C)   : 48
QSFP#14.temp (C)   : 47
QSFP#15.temp (C)   : 51
QSFP#16.temp (C)   : 49
QSFP#17.temp (C)   : 53
QSFP#18.temp (C)   : 50
QSFP#19.temp (C)   : 52
QSFP#20.temp (C)   : 50
QSFP#21.temp (C)   : 51
QSFP#22.temp (C)   : 49
QSFP#23.temp (C)   : 47
QSFP#24.temp (C)   : 47
QSFP#25.temp (C)   : 44
QSFP#26.temp (C)   : 44
QSFP#27.temp (C)   : 42
QSFP#28.temp (C)   : 46
QSFP#29.temp (C)   : 41
QSFP#30.temp (C)   : 45
QSFP#31.temp (C)   : 40
QSFP#32.temp (C)   : 45
QSFP#33.temp (C)   : 39
QSFP#34.temp (C)   : 44
QSFP#35.temp (C)   : 37
QSFP#36.temp (C)   : 39
fan#1.speed (rpm)  : 6355
fan#2.speed (rpm)  : 5421
fan#3.speed (rpm)  : 6570
fan#4.speed (rpm)  : 5508
fan#5.speed (rpm)  : 6415
fan#6.speed (rpm)  : 5486
fan#7.speed (rpm)  : 6268
fan#8.speed (rpm)  : 5421
```

Or, to only get fans and power supplies' status:

```
# ./ibswinfo -d <device> -o status
psu0.status        : OK
psu0.dc            : OK
psu0.fan           : OK
psu1.status        : OK
psu1.dc            : OK
psu1.fan           : OK
fans               : OK
```


### Setting switch names
*requires `ibswinfo` >= 0.6 and MFT >= 4.22.0*

Unmanaged switches' node descriptions can be customized, for easier management
and better switch identification.

To update a switch's name to `<new_node description>`"
```
# ./ibswinfo -d <device> -S <new_node_description>
Device: <device>
  Current node description: <old_node_description>
  Set node description to : <new_node_description>
>> Confirm? (y/N) y
Setting new node description... done!
```

By default, unmanaged Infiniband switches all come with the same node
description, based on their model, and it can quickly become cumbersome to
determine which is which. Setting customized and descriptive switch names can
help solve that problem and allow better identification of switches in the
datacenter.

Before:
```
# ibswitches
Switch  : 0xXXXXXXXXXXXXXXXa ports 37 "SwitchX - Mellanox Technologies" base port 0 lid 1 lmc 0
Switch  : 0xXXXXXXXXXXXXXXXb ports 37 "SwitchX - Mellanox Technologies" base port 0 lid 2 lmc 0
Switch  : 0xXXXXXXXXXXXXXXXc ports 37 "SwitchX - Mellanox Technologies" base port 0 lid 3 lmc 0
Switch  : 0xXXXXXXXXXXXXXXXd ports 37 "SwitchX - Mellanox Technologies" base port 0 lid 4 lmc 0
```

After:
```
# ibswitches
Switch  : 0xXXXXXXXXXXXXXXXa ports 37 "switch1 in rack1 U1" base port 0 lid 1 lmc 0
Switch  : 0xXXXXXXXXXXXXXXXb ports 37 "switch2 in rack1 U3" base port 0 lid 2 lmc 0
Switch  : 0xXXXXXXXXXXXXXXXc ports 37 "switch3 in rack1 U5" base port 0 lid 3 lmc 0
Switch  : 0xXXXXXXXXXXXXXXXd ports 37 "switch4 in rack1 U7" base port 0 lid 4 lmc 0
```



