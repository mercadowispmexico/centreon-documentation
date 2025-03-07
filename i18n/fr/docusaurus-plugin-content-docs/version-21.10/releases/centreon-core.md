---
id: centreon-core
title: Centreon Core
---

## Introduction

Vous trouverez dans ce chapitre tout ce qui concerne les **Centreon Core**.

> Il est important de mettre à jour en utilisant la documentation adéquate de mise à jour et de lire attentivement les
> notes de mise à jour afin d'être au courant des changements qui pourraient impacter votre usage ou votre plateforme
> ou des développements spécifiques que vous auriez fait.

Pour faire des demandes d'évolutions ou reporter des bugs sur les extensions commerciales, vous pouvez vous rendre sur
notre [Github](https://github.com/centreon/centreon/issues/new/choose).

## Centreon Web

### 21.10.3

Release date: `January 26, 2022`

#### Bug Fixes

- [Graph] Fixed display of additional graph if it came from Resources Status
- [Install] Fixed SQL request syntax error for cron with MySQL 8
- [Resources Status] Fixed display of meta-services
- [Resources Status] Fixed graph unit displayed twice
- [Resources Status] Fixed saving a filter on an existing name
- [Resources Status] Take the default downtime options to set downtime
- [UX] Fixed random disconnection since update to Centreon 21.10

### 21.10.2

Release note: `December 24, 2021`

#### Enhancements

- [Administration] Display the name of the object that has been modified in the detail form of logs
- [Authentication] Removed token display in login debug file
- [UI] The top-counter menu for pollers is now refreshed immediately after enabling the "Export button" in the user's profile

#### Bug Fixes

- [API] Fixed the access to API is account doesn't have access to GUI
- [Authentication] Fixed LDAP OU quote connection breaking
- [CLAPI] Fixed an issue preventing ACLs from applying on services created with CLAPI
- [CLAPI] Fixed error with LDAP configuration ID
- [Configuration] Fixed SNMP Trap matching with service linked to multiple hosts
- [Configuration] Fixed an issue that caused the Anomaly Detection services to lose their graphs when they were renamed
- [Configuration] Fixed an issue that caused the loss of broker output configuration
- [Configuration] Fixed an issue that prevented from removing the SNMP community (and other fields) from the host form
- [Configuration] Fixed the wizard for adding a new server that did not add it
- [Configuration] Fixed unwanted writes into unexisting file when exporting Traps config at the same time as a trap arrives. Based on PR [#9973](https://github.com/centreon/centreon/pull/9973). Fixes issue [#4236](https://github.com/centreon/centreon/issues/4236).
- [MBI] Fixed CBIS process trying to get contact_js_effects column that no longer exists
- [Resource Status] Fixed graph tooltip

### 21.10.1

Release date: `November 29, 2021`

#### Bug Fixes

- [Authentication] Fixed PHP error when debug is enabled with OIDC authentication
- [Configuration] Fixed the list of host template that was not available if the database name was different from the default
- [UX] Non admin user do not have the same submenu subsections
- [UX] Remove "Animation effects" option

### 21.10.0

#### Enhancements

- [Authentication] Improve OIDC support (OpenId Connect)
  - Add Okta support
  - Add MS Azure AD / ADFS
  - Add possibility to define which claim is used for Centreon login
  - Add possibility to define complete URL for endpoints
  - Add possibility to use client_secret_basic as authentication. Based on PR
    [#9878](https://github.com/centreon/centreon/pull/9878)
  - Allow to define no redirect URL. Based on PR
    [#9877](https://github.com/centreon/centreon/pull/9877)
  - Add errors log in /var/log/centreon/login.log
  - Add possibility to display debug log in /var/log/centreon/login.log
  - Use proxy if defined
- [API] API versioning is now consistent with Centreon's major release number
- [CEIP] Product Adoption component integration
- [Configuration] The poller management actions are now only available via buttons:
  - "Add" now leads to the wizard.
  - "Add (advanced)" leads to the former "Add" action (for experts only).
  - "Delete" and "Duplicate" are converted into buttons.
  - "Delete" should normally not be confused with another action.
- [Configuration] The deprecated "Logger" tab of the "Broker configuration" menu has been removed
- [Resources Status] Revamp Search experience
- [Resources Status] Revamp Timeline
- [Resources Status] Add Sticky and Persistent options to ACK in Resource Status
- [Resources Status] Allow detail tiles to be re-ordered for each user
- [Resources Status] Add multi-select to Resources Status listing
- [Resources Status] Add "Last OK" tile within Details panel
- [Resources Status] Persist user selected number of rows displayed
- [Resources Status] Make "duration" as the default second sorting criteria
- [Resources Status] Add link to performance page in detail panel. Based on PR [#9822](https://github.com/centreon/centreon/issues/9822)
- [Resources Status] Add Graphs panel for Hosts
- [Resources Status] Add tooltip to explain grayed options
- [Resources Status] Improve Custom Columns Name Display
- [Resources Status] Move Shortcuts from dedicated panel to option within Header
- [Resources Status] Make configure resource icon always visible
- [Resources Status] Improve readability of command line displayed
- [UX] Add Feature Flipping for Resources Status vs Legacy Pages
- [UX] Downtimes can now be scheduled until 2100
- [UX] The poller management action buttons are now hidden on Remote Servers

#### Beta enhancements

- [Configuration] Administrators can toggle a new button in the Pollers top-counter menu that allows them to export and
  reload the configuration of all pollers from any page

#### Breaking changes

> Access to API v2 has been changed. All of the beta endpoints have been migrated to version 21.10. This must be
> modified by "latest" or by the version of your Centreon platform (v21.10 for example).

For example replace:
```shell
{protocol}://{server}:{port}/centreon/api/beta/login
```

By:
```shell
{protocol}://{server}:{port}/centreon/api/latest/login
```

or:
By:
```shell
{protocol}://{server}:{port}/centreon/api/v21.10/login
```

#### Performances

- Move to PHP 8.0
- Preparing Debian 11 support

##  Centreon Collect

> As of version 21.10.0, the components of Centreon Collect (Centreon Broker, Centreon Clib, Centreon Engine and Centreon Connectors)
> are released simultaneously. They are now grouped under this section.

### 21.10.1

Release date: `February 23, 2022`

#### Centreon Broker

##### Improvements
- Improved the multiplexing of events, which was a performance bottleneck. The processing speed of queued events should be significantly increased.
- Some TLS trace logs were logged as errors, flooding the log file.
- The SQL connections stats have been added to the broker stats available via gRPC.
- Added the SQL Manager statistics in gRPC and stabilized the stats provider.
- Added to LUA Stream Connectors' perfdata label parser the compatibility with centreon-plugins new metrics naming convention.

##### Bug fixes

- Fixed a regression due to the central broker's cache generation optimization, which was too thorough and prevented BAM from computing KPIs based on boolean rules
- The central broker's cache generation loaded too much data and took too much time when BAM was activated.
- Fixed an issue that could cause segmentation faults in centreon-engine when scheduling external commands
- Fixed a design issue to avoid trying to access variables of broker's new logger when the logger was stopped. This issue could cause segmentation faults.
- When a single metric is deleted, the corresponding RRD file is now actually removed.
- If the SQL stream took too long to initialize its connection, then the Perfdata stream timed out and the whole connection failed. To fix this, the timeout has been increased.
- In some circumstances, the `mysql_ping` function, which is used to test if the session is still active, could freeze. To fix this, the calls to `mysql_ping` have been spaced out, a timeout has been added, and the commit management has been consolidated.
- Fixed an issue causing BAM Business Activities (best status) to remain in an OK state when the OK KPIs were removed
- Refactored the BAM Business activities downtimes inheritance mechanism so that they are properly inherited and not duplicated anymore.

### 21.10.0

#### Centreon Engine

- Flapping now starts only on non-OK states. Based on PR [#523](https://github.com/centreon/centreon-engine/pull/523)
- Flapping now starts only for services of UP hosts or for hosts with UP parent. Based on PR [#524](https://github.com/centreon/centreon-engine/pull/524). Fixes Issue [#192](https://github.com/centreon/centreon-engine/issues/192)
- Provide feedback on gRPC client execution success/failure

#### Centreon Broker

- Queues (in memory and retention files) are now cleared for reversed broker flows without `peer retention` when the connection is reset. This is a change of behavior back to what it should have always been. It will prevent endless retention files for Centreon-Studio (Centreon-Map).
- [BETA] Centreon-broker is now able to use OpenSSL instead of GNUTLS and allows forcing TLS/SSL version and cipher suite
- Broker now only loads the modules that are necessary for its inputs and outputs
- Old broker log format has been removed

## Centreon Gorgone

### 21.10.2

Release date: `February 17, 2022`

#### Enhancements

- Added an "audit" module to Gorgone to provide an overview of the system status, package versions, + some Centreon metrics.
- Added a new "httpserverng" module to allow asynchronous API calls.

#### Bugfixes

- Fixed an issue that caused Service Discovery scans to fail because the wrong message was caught.
- Fixed an issue that could make Gorgone crash in pull mode.
- Fixed uninitialized values in Gorgone that could cause error log messages.
- Fixed an issue that prevented Gorgone from handling advanced [Service Discovery features](../monitoring/discovery/services-discovery.md#options-avanc%C3%A9es) correctly.
- Fixed an issue in the module management that could cause crashes.

### 21.10.1

Release date: `December 14, 2021`

#### Bugfixes

- Make Gorgone’s private key readable by centreon-gorgone only
- Gorgone was too long to restart, which could cause the service to reach the systemctl timeout. The time to stop has been thoroughly decreased.


### 21.10.0

- Add IPv6 support
