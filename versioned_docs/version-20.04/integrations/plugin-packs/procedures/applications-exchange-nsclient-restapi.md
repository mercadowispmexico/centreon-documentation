---
id: applications-exchange-nsclient-restapi
title: Exchange NSClient++ API
---
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';


## Overview

The Plugin Pack *Exchange NSClient API* works with the Centreon NSClient++ monitoring
agent and its built-in web server to run Powershell code to check the health and 
performance of Microsoft Exchange Servers.

## Pack assets

### Monitored objects

* From Exchange Server 2k10 to latest

### Collected metrics

*Coming soon ...*

## Prerequisites

### NSClient++

To monitor an *Exchange Server* through NSClient++ API, install the Centreon packaged version 
of the NSClient++ agent. Please follow our [official documentation](../tutorials/centreon-nsclient-tutorial) 
and make sure that the **Webserver / RESTApi** configuration is correct. 

### Powershell 

Powershell and the `Microsoft.Exchange.Management.PowerShell` snap-in must be installed
on the target Server.

## Installation 

<Tabs groupId="sync">
<TabItem value="Online License" label="Online License">

1. Install the Centreon NRPE Client package on every Poller expected to monitor *Microsoft Exchange servers*:

```bash
yum install centreon-plugin-Operatingsystems-Windows-Restapi
```

2. On the Centreon Web interface, install the Centreon Plugin-Pack *Exchange NSClient API* 
from the **Configuration > Plugin Packs > Manager** page

</TabItem>
<TabItem value="Offline License" label="Offline License">

1. Install the Centreon Plugin package on every Poller expected to monitor *Microsoft Exchange*:

```bash
yum install centreon-plugin-Operatingsystems-Windows-Restapi
```

2. Install the Centreon Plugin-Pack RPM on the Central server:

```bash
yum install centreon-pack-applications-exchange-nsclient-restapi
```

3. On the Centreon Web interface, install the Centreon Plugin-Pack *Exchange NSClient API* 
from the **Configuration > Plugin Packs > Manager** page

</TabItem>
</Tabs>

## Host configuration

* Log into Centreon and add a new Host through **Configuration > Hosts**.
* Apply the *App-Exchange-NRPE-custom* template and configure all the mandatory Macros:

| Mandatory | Name                      | Description                                                                |
|:----------|:--------------------------|:-------------------------------------------------------------------------- |
| X         | NSCPRESTAPIPORT           | NSClient++ RestAPI port (Default: '8443')                                  |
| X         | NSCPRESTAPIPROTO          | NSClient++ RestAPI protocol to use (Default: 'https')                      |
|           | NSCPRESTAPILEGACYPASSWORD | Password to authenticate against the API if relevant                       |
|           | NSCPRESTAPIEXTRAOPTIONS   | Any extra option you may want to add to the command (eg. a --verbose flag) |

## Important information

* The *Queue* monitoring Service only works on Exchange Server running with the 
hub/transport role
* Use this format to define MAILBOX macro at the service level: DOMAIN\\USER