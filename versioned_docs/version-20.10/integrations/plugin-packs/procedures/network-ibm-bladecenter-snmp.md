---
id: network-ibm-bladecenter-snmp
title: IBM Bladecenter Switch
---
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';


## Overview

IBM develops, manufactures and sells hardware, middleware, software and other
services and products. Created in 2002, IBM BladeCenter was IBM's blade server
architecture. 

## Plugin-Pack assests

### Monitored objects

* IBM Bladecenter switch module

### Collected Metrics

In addition to modes and metrics described here, it is also possible to monitor 
the following indicators:

* Ntp: Check time offset of server with ntp server
* Uptime: Elapsed time since the last reboot

<Tabs groupId="sync">
<TabItem value="Cpu" label="Cpu">

| Metric name                    | Description                              | Unit   |
| :----------------------------- | :--------------------------------------- | :------|
| cpu.utilization.1m.percentage  | CPU utilization for the last minute      | %      |
| cpu.utilization.5m.percentage  | CPU utilization for the last 5 minutes   | %      |

</TabItem>
<TabItem value="Storage" label="Storage">

| Metric name                         | Description                    | Unit   |
| :---------------------------------- | :----------------------------- |------- |
| storage.partitions.count            | Total number of partition      |        |
| partition#storage.space.usage.bytes | Used space on a disk partition | Bytes  |

</TabItem>
<TabItem value="Memory-Usage" label="Memory-Usage">

| Metric name             | Description                 | Unit   |
| :---------------------- | :---------------------------| :----- |
| memory.usage.bytes      | Total current memory usage  | Bytes  |
| memory.usage.percentage | Total current memory usage  |  %     |
| memory.free.bytes       | Current free memory         | Bytes  |

</TabItem>
<TabItem value="Traffic" label="Traffic">

| Metric name                              | Description                                                               | Unit        |
| :--------------------------------------- | :------------------------------------------------------------------------ | :---------- |
| status                                   | Interface status                                                          |             |
| interface.traffic.\*.bitspersecond       | \*in/out. Incoming/outgoing traffic going through the interface           | Bytes/s & % |
| interface.packets.\*.errors.percentage   | \*in/out. Incoming/outgoing errored packets going through an interface    | Count & %   |
| interface.packets.\*.discards.percentage | \*in/out. Incoming/outgoing discarded packets going through an interface  | Count & %   |

A regexp filter is available to target a specific interface identifier/ifName [```--interface='^my-interface-name$' --name```] 

</TabItem>
<TabItem value="Environment" label="Environment">

| Metric name                   | Description                      | Unit     |               
| :---------------------------- | :------------------------------- | :--------|
| hardware.temperature.celsius  | Temperature of the system        | Celsius  |
| faultled                      | Status of the fault LED (On/Off) |          |

You can use ```--no-component``` if you want to alert when a component is 
absent/removed. You can also overload the default status using the 
```--threshold-overload option```. 

</TabItem>
</Tabs>

## Prerequisites

To use this pack, the SNMP service must be properly configured on your 
IBM BladeCenter server. Lenovo provides an official documentation
to achieve this: 
* Throught BladeCenter Web Interface: https://bladecenter.lenovofiles.com/help/index.jsp?topic=%2Fcom.lenovo.bladecenter.mgtmod.doc%2Fkp1ag_bc_mmug_configsnmp
* Throught BladeCenter Command-Line Interface : https://bladecenter.lenovofiles.com/help/index.jsp?topic=%2Fcom.lenovo.bladecenter.advmgtmod.doc%2Fkp1bc_bc_cli_snmp

### Network flow

The target IBM BladeCenter server must be reachable from the Centreon
Poller on the UDP/161 SNMP port.

## Installation

<Tabs groupId="sync">
<TabItem value="Online License" label="Online License">

1. Install the Plugin package on every Centreon poller expected to monitor an IBM BladeCenter switch module:

```bash
yum install centreon-plugin-Network-Ibm-Bladecenter-Snmp
```

2. On the centreon Web interface, install the *IBM BladeCenter Switch* Centreon Plugin-Pack on the "Configuration > Plugin Packs > Manager" page

</TabItem>
<TabItem value="Offline License" label="Offline License">

1. Install the Plugin on every Centreon poller expected to monitor an IBM BladeCenter switch module:

```bash
yum install centreon-plugin-Network-Ibm-Bladecenter-Snmp
```

2. Install the Centreon Plugin-Pack RPM on the Centreon Central server:

```bash
yum install centreon-pack-network-ibm-bladecenter-snmp
```

3. On the centreon Web interface, install the *IBM BladeCenter Switch* Centreon Plugin-Pack on the "Configuration > Plugin Packs > Manager" page

</TabItem>
</Tabs>

## Configuration

Go to *Configuration* > *Host* > and click *Add*. Then fill the *SNMP Community*
and *SNMP Version* fields and apply the template 
*Net-Ibm-Bladecenter-SNMP-custom*.

If you are using SNMP Version 3, use the
*SNMPEXTRAOPTIONS* macro to configure your own SNMPv3 credentials combo.

| Mandatory   | Name             | Description                                    |
| :---------- | :--------------- | :--------------------------------------------- |
|             | SNMPEXTRAOPTIONS | Configure your own SNMPv3 credentials combo    |

## FAQ

### How do I run my plugin through the CLI and what do the main parameters stand for ?

Once you've installed the plugin, you can test it logging with *centreon-engine*
user:
 
```bash
/usr/lib/centreon/plugins/centreon_net_bladecenter_snmp.pl \
  --plugin=network::ibm::bladecenter::snmp::plugin \
  --mode=interfaces --hostname=10.30.2.138 \
  --snmp-community='public' \
  --snmp-version='2c' \
  --add-status \
  --add-traffic
```

Expected command output is shown below:

```
OK: All interfaces are ok | 'traffic_in_lo'=19795.35b/s;;;0;10000000 'traffic_out_lo'=19795.35b/s;;;0;10000000 'traffic_in_ens192'=26001.62b/s;;;0;10000000000 'traffic_out_ens192'=24438.38b/s;;;0;10000000000
```

All available modes with the plugin can be displayed with:

```bash
/usr/lib/centreon/plugins/centreon_net_bladecenter_snmp.pl \
  ---plugin=network::ibm::bladecenter::snmp::plugin \
  --list-mode
```

The available options for a mode can be displayed using the ```--help``` parameter:

```bash
/usr/lib/centreon/plugins/centreon_net_bladecenter_snmp.pl \
  --plugin=network::ibm::bladecenter::snmp::plugin \
  --mode=interfaces \
  --help
```

### UNKNOWN: SNMP GET Request : Timeout

If you get this message, you're probably facing one of theses issues:

* Your SNMP server isn't started or misconfigured
* An external device is blocking your request (firewall, ...)

### UNKNOWN: SNMP GET Request : Cant get a single value.

This message generally means that SNMP privileges are not wide enough for the
mode/plugin to work properly.
