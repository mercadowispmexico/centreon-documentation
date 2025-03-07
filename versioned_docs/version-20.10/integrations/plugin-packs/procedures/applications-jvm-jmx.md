---
id: applications-jvm-jmx
title: JVM JMX
---
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';


## Plugin Pack Assets

### Monitored Objects

The Pack JVM JMX collects metrics for:
* Class
* Gc
* Memory
* System (cpu, load-average, fd)
* Threads

### Collected Metrics

<Tabs groupId="sync">
<TabItem value="Class-count" label="Class-count">

| Metric name                | Description                      | Unit |
| :------------------------- | :------------------------------- | :--- |
| class.loaded.current.count | Current number of classes loaded |      |
| class.loaded.count         | Number of classes loaded         |      |
| class.unloaded.count       | Number of classes unloaded       |      |

</TabItem>
<TabItem value="Cpu-load" label="Cpu-load">

| Metric name                 | Description                  | Unit |
| :-------------------------- | :--------------------------  | :--- |
| system.cpu.load.percentage  | Cpu load of the machine      | %    |
| process.cpu.load.percentage | Cpu load of the jvm instance | %    |

</TabItem>
<TabItem value="Fd-usage" label="Fd-usage">

| Metric name                | Description                                   | Unit |
| :------------------------- | :-------------------------------------------- | :--- |
| fd.opened.usage.count      | Number of used file descriptors               |      |
| fd.opened.free.count       | Number of free file descriptors               |      |
| fd.opened.usage.percentage | Number of used file descriptors in percentage | %    |

</TabItem>
<TabItem value="Gc-usage" label="Gc-usage">

| Metric name                             | Description                              | Unit |
| :-------------------------------------- | :--------------------------------------- | :--- |
| gc.collection.time.elapsed.milliseconds | Accumulated collection elapsed time      | ms   |
| gc.collection.count                     | Number of collections that have occurred |      |

</TabItem>
<TabItem value="Load-average" label="Load-average">

| Metric name          | Description                             | Unit |
| :------------------- | :-------------------------------------- | :--- |
| system.load.1m.count | System load average for the last minute |      |

</TabItem>
<TabItem value="Memory" label="Memory">

| Metric name                | Description                           | Unit |
| :------------------------- | :------------------------------------ | :--- |
| memory.heap.usage.bytes    | Current heap memory usage             | B    |
| memory.nonheap.usage.bytes | Current memory usage outside the heap | B    |

</TabItem>
<TabItem value="Memory-detailed" label="Memory-detailed">

| Metric name                  | Description                    | Unit |
| :--------------------------- | :----------------------------- | :--- |
| memory.eden.usage.bytes      | Current eden memory usage      | B    |
| memory.tenured.usage.bytes   | Current tenured memory usage   | B    |
| memory.survivor.usage.bytes  | Current survivor memory usage  | B    |
| memory.permanent.usage.bytes | Current permanent memory usage | B    |
| memory.code.usage.bytes      | Current code memory usage      | B    |

</TabItem>
<TabItem value="Threads" label="Threads">

| Metric name           | Description                        | Unit |
| :-------------------- | :--------------------------------- | :--- |
| threads.active.count  | Number of active threads           |      |
| threads.started.count | Number of threads started          |      |
| threads.daemon.count  | Count of threads marked as daemons |      |

</TabItem>
</Tabs>

## Prerequisites

Please install jolokia agent on your JVM [Jolokia download page](https://jolokia.org/download). Ask to your admin to deploy it and give you the URL.

## Setup

<Tabs groupId="sync">
<TabItem value="Online License" label="Online License">

1. Install the Centreon Plugin on every Poller:

```bash
yum install centreon-plugin-Applications-Jvm-Jmx
```

2. On the Centreon Web interface in **Configuration > Plugin Packs > Manager**, install the *JVM JMX* Pack

</TabItem>
<TabItem value="Offline License" label="Offline License">

1. Install the Centreon Plugin on every Poller:

```bash
yum install centreon-plugin-Applications-Jvm-Jmx
```

2. On the Centreon Central server, install the Centreon Pack from the RPM:

```bash
yum install centreon-pack-applications-jvm-jmx
```

3. On the Centreon Web interface in **Configuration > Plugin Packs > Manager**, install the *JVM JMX* Pack

</TabItem>
</Tabs>

## Host configuration 

* Add a new Host and apply the *App-Jvm-JMX-custom* Host Template

> Once the template applied, some Macros have to be configured:

| Mandatory   | Name                | Description                                                                |
| :---------- | :------------------ | :------------------------------------------------------------------------- |
| X           | JOLOKIAURL          | Jolokia URL (eg: http://jvm.centreon.com:8080/jolokia)                     |
|             | JOLOKIAUSERNAME     | Jolokia user name                                                          |
|             | JOLOKIAPASSWORD     | Jolokia password                                                           |
|             | JOLOKIAEXTRAOPTIONS | Any extra option you may want to add to the command (eg. a --verbose flag) |

## How to test the Plugin and what are the main options for?

Once the Plugin installed, log into your Poller using the *centreon-engine* user account and test by running the following command
(Parameters such as ```url``` have to be adjusted):

```bash
/usr/lib/centreon/plugins/centreon_jvm_jmx.pl \
    --plugin=apps::java::jvm::jmx::plugin \
    --mode=class-count \
    --url='http://jvm.centreon.com:8080/jolokia' \
    --verbose
```

Output example:
```
OK: Class current: 3009, loaded: 0, unloaded: 0 | 'class.loaded.current.count'=3009;;;0; 'class.loaded.count'=0;;;0; 'class.unloaded.count'=0;;;0;
```

The command above monitors JVM class usage (```--mode=class-count```).
It uses the URL (```--url='http://jvm.centreon.com:8080/jolokia'```) to connect.

All the options that can be used with this plugin can be found over the ```--help``` command:

```bash
/usr/lib/centreon/plugins/centreon_jvm_jmx.pl \
    --plugin=apps::java::jvm::jmx::plugin \
    --mode=class-count \
    --help
```

## Troubleshooting

[Troubleshooting plugins](../tutorials/troubleshooting-plugins.md)
