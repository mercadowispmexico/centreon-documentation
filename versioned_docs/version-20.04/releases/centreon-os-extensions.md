---
id: centreon-os-extensions
title: Open Source Extensions
---

## Introduction

You can find in this chapter all changelogs concerning **Centreon Open Source
extensions**.

> It is very important when you update your system to refer to this
> section in order to learn about behavior changes or major changes that
> have been made on this version. This will let you know the impact of
> the installation of these versions on the features you use or the
> specific developments that you have built on your platform (modules,
> widgets, plugins).

If you have feature requests or want to report a bug, please go to our
[Github](https://github.com/centreon/centreon/issues/new/choose)

## Centreon High-Availability

### 20.04.3

`November 30, 2020`

#### Bugfixes

- [Sync] An error in the script prevented the files to be synchronized
  correctly

#### Enhancements

- [RPM] All the configuration files (except for logrotate.d and sudoers.d)
  are now preserved by the update process

### 20.04.2

`November 3, 2020`

#### Bugfixes

- [Sync] Removing `-a` option to rsync

#### Enhancements

- [Sync] Protect paths with quotes to manage directories with spaces

### 20.04.1

`July 8, 2020`

#### Bugfixes

- [Cron] gorgoned manages all scheduled tasks
- [Sync] centreon_central_sync bugfixes (realtime sync)
- [Sync] fix(centreon_central_sync)fix-HUP-signal-and-bugs
- [Documentation] Revoke SUPER right from Centreon user. Add SELECT right to Replication user
- [Documentation] Add specific chmod associated to /var/log/centreon-engine/ and /tmp/centreon-autodisco/ shared directories
- [Documentation] Missing cache regeneration in update procedure

### 20.04.0

`April 22, 2020`

- [Release] First release of [centreon-ha](https://github.com/centreon/centreon-ha)

## Centreon DSM

### 20.04.0

`April 22, 2020`

- Manage compatibility with Centreon 20.04

#### Bugfixes

- Manage hosts disabled: host change instance or instance is not
  running (commit a99b068)
- Applied patch for MariaDB 10.2.x (commit a567991)

## Centreon Open Ticket

### 20.04.2

`May 31, 2021`

#### Bugfixes

- [Widget] Compatibility with jQuery 3.5

### 20.04.1

`February 4, 2021`

#### Bugfixes

- Manage pagination for GLPI webservice

### 20.04.0

`April 22, 2020`

- Manage compatibility with Centreon 20.04

#### Bugfixes

- [API] Better exception handling (PR #111)
- [Core] Fix test button (PR #114)
- [Install] Limit index size (InnoDB limit) (PR #125)

#### Security fixes

- [Security] Remove session id parameter (PR #118)

#### New features

- [Provider] Add a revamped version of the glpi provider (PR #113)
- [Provider] Add request tracker api v2 provider (commit 3d88c9)
- [Provider] Add iTop provider (PR #115)
