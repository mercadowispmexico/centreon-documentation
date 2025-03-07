---
id: applications-mscs-nrpe
title: Microsoft Cluster Server NSClient++ NRPE
---
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';


> Hello community! We're looking for a contributor to help us to translate the 
content in french and provide a sample execution command. If it's you, let us 
know and ping us on [slack](https://centreon.slack.com)

## Overview

The Plugin Pack *Microsoft Cluster Server NRPE* works with the Centreon NSClient++ monitoring
agent to check the health and performance of Microsoft Clusters.

## Pack assets

### Monitored objects

* Microsoft Server with the MSCS extension

### Collected metrics

*Coming soon*

## Prerequisites

### NSClient++

To monitor a *Microsoft Cluster Server* through NRPE, install the Centreon packaged version 
of the NSClient++ agent. Please follow our [official documentation](../tutorials/centreon-nsclient-tutorial) 
and make sure that the **NRPE Server** configuration is correct. 

## Installation 

<Tabs groupId="sync">
<TabItem value="Online License" label="Online License">

1. Install the Centreon NRPE Client package on every Poller expected to monitor *Microsoft Cluster servers*:

```bash
yum install centreon-nrpe-plugin
```

2. On the Centreon Web interface, install the Centreon Pack *Microsoft Cluster servers* 
from the **Configuration > Plugin Packs > Manager** page

</TabItem>
<TabItem value="Offline License" label="Offline License">

1. Install the Centreon Plugin package on every Poller expected to monitor *Microsoft Cluster servers*:

```bash
yum install centreon-nrpe-plugin
```

2. Install the Centreon Pack RPM on the Centreon Central server:

```bash
yum install centreon-pack-applications-mscs-nrpe
```

3. On the Centreon Web interface, install the Centreon Pack *Exchange NRPE* 
from the **Configuration > Plugin Packs > Manager** page

</TabItem>
</Tabs>

## Host configuration

* Log into Centreon and add a new Host through **Configuration > Hosts**.
* Apply the *App-Mscs-NRPE-custom* template and configure all the mandatory Macros:

| Mandatory | Name             | Description                                                            |
| :-------- | :--------------- | :--------------------------------------------------------------------- |
| X         | NRPECLIENT       | NRPE Binary used to perform the check (default: 'check_centreon_nrpe)  | 
| X         | NRPEPORT         | Port used to reach the NRPE server (default: '5666')                   |
| X         | NRPETIMEOUT      | Timeout to connect to the NRPE Server (default: '10')                  |
| X         | NRPEEXTRAOPTIONS | Extraoptions to use with the NRPE binary (default: '-u -m 8192')       |

## Important information

* You cannot use `|` and `!` characters in your Centreon Macro values. 
Prefer using RestAPI version of the Pack if you really need it. 