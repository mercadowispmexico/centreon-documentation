---
id: cloud-azure-management-discover
title: Azure Discover
---
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';


## Overview

The Centreon Plugin Pack *Azure Discover* is a *super* Pack allowing to discover a whole Azure infrastructure for a given
subscription.
This Pack relies on the Azure Monitor API to fetch the resources of the Azure infrastructure and on all of the Centreon Plugin Packs
for Azure to set templates and proper monitoring indicators for each type of resource.

> This Plugin Pack is only compatible with the 'api' custom-mode. 'azcli' is not supported for this usage.

## Pack Assets

> The Centreon Plugin Pack *Azure Discover* is only a *discovery* pack. It doesn't natively provide any templates nor
> indicators to monitor Azure resources

### Discovery rules

The Centreon Plugin Pack *Azure Discover* includes a Host Discovery *provider* to automatically discover all of the Azure resources
of a given subscription and add them to the Centreon configuration using the dedicated Host Templates.
This provider is named **Microsoft Azure Management Discover**:

![image](../../../assets/integrations/plugin-packs/procedures/cloud-azure-management-discover-provider.png)

> This discovery feature is only compatible with the 'api' custom mode.

More information about the Host Discovery module is available in the Centreon documentation:
[Host Discovery](../../../monitoring/discovery/hosts-discovery.md)

## Prerequisites

### Azure credentials

To use the 'api' custom mode, make sure to obtain the required information using the 
how-to below. Keep it safe until including it in the Autodiscovery job settings form.

* Create an *application* in Azure Active Directory:
    - Log in to your Azure account.
    - Select *Azure Active directory* in the left sidebar.
    - Click on *App registrations*.
    - Click on *+ Add*.
    - Enter Centreon as the application name (or any name of your choice), select application type(api) and sign-on-url.
    - Click on the *Create* button.

* Get *Subscription ID*
    - Log in to your Azure account.
    - Select *Subscriptions* in the left sidebar.
    - Select whichever subscription is needed.
    - Click on *Overview*.
    - **Copy the Subscription ID.**

* Get *Tenant ID*
    - Log in to your Azure account.
    - Select *Azure Active directory* in the left sidebar.
    - Click on *Properties*.
    - **Copy the directory ID.**

* Get *Client ID*
    - Log in to your Azure account.
    - Select *Azure Active directory* in the left sidebar.
    - Click on *Enterprise applications*.
    - Click on *All applications*.
    - Select the application previously created.
    - Click on *Properties*.
    - **Copy the Application ID.**

* Get *Client secret*
    - Log in to your Azure account.
    - Select *Azure Active directory* in the left sidebar.
    - Click on *App registrations*.
    - Select the application previously created.
    - Click on *All settings*.
    - Click on *Keys*.
    - Enter the key description and select the duration.
    - Click on *Save*.
    - **Copy and store the key value. You won't be able to retrieve it after you leave this page.**

## Setup 

<Tabs groupId="sync">
<TabItem value="Online License" label="Online License">

1. Install the Plugin on every Centreon Poller expected to discover Azure resources:

```bash
yum install centreon-plugin-Cloud-Azure-Management-Discover-Api
```

2. On the Centreon Web interface, install the *Azure Discover* Centreon Plugin Pack on the "Configuration > Plugin Packs > Manager" page
You'll be prompted to install several other Azure Plugin Packs as dependencies (they will be used to set the proper templates/indicators
on the discovered elements).

</TabItem>
<TabItem value="Offline License" label="Offline License">

1. Install the Plugin on every Centreon Poller expected to discover Azure resources:

```bash
yum install centreon-plugin-Cloud-Azure-Management-Discover-Api
```

2. Install the Centreon Plugin Pack RPM on the Centreon Central server, install all of the Centreon Plugin Packs for Azure, in order
to make all the dependencies available:

```bash
yum install centreon-pack-cloud-azure\*
```

3. On the Centreon Web interface, install the *Azure Discover* Centreon Plugin Pack on the "Configuration > Plugin Packs > Manager" page
You'll be prompted to install several other Azure Plugin Packs as dependencies (they will be used to set the proper templates/indicators
on the discovered elements).

</TabItem>
</Tabs>

## Set up a discovery job

> The general specifications and mechanics of the *Host Discovery* feature is available [here](../../../monitoring/discovery/hosts-discovery.md)

### Access parameters

Create a new discovery job and select **Azure Management Discover** as the provider. Click on *next* and set the authentication parameters
as well as optional access parameters:

![image](../../../assets/integrations/plugin-packs/procedures/cloud-azure-management-discover-accessparameters.png)

- Select the **Centreon Poller** from where the discovery job will be launched
- If necessary, add an entreprise **proxy URL and port** to use to reach the Azure API
- Select the **Azure credentials profile** linked to the subscription to be used

The first time, a new credentials profile has to be created. You can do so by clicking the '+' button and set the proper Azure
authentication parameters:

![image](../../../assets/integrations/plugin-packs/procedures/cloud-azure-management-discover-credentials.png)

> All of the fields of the *credentials* form must be filled.

Click on *confirm* then *next* to go to the next step of the wizard and adjust the discovery parameters.

### Discovery parameters

If necessary, adjust the following settings:

![image](../../../assets/integrations/plugin-packs/procedures/cloud-azure-management-discover-discoparameters.png)

> All the fields of this form are optional

- Azure Location/Resource Group: allows to filter the discovery on a specific *Location* or *Resource Group*
- Filter on namespace/type: only discovers elements of a given namespace/type relative to Azure resources, for example:
    - *Resource namepsace*: 'Microsoft.Compute'
    - *Resource type*: 'virtualMachines'
    > ** Warning ** To use this filter, it's mandatory to fill **both** *Resource namespace* and *Resource type* fields

### Run the discovery job and display results

The step 4 of the wizard allows to adjust and set **mappers** if necessary; the Plugin Pack comes along with predefined **mappers** that
don't typically need to be changed. If you have a specific need and want to edit the **mappers** section, refer to 
[this documentation](../../../monitoring/discovery/hosts-discovery.md#how-to-use-the-mappers) to do so.

Final steps 5 & 6 will allow you to define a specific policy about the data modeling of the discovered results. Although the default configuration
is usually enough to proceed, [this documentation](../../../monitoring/discovery/hosts-discovery.md#define-analysis-and-update-policies) 
will help you to customize it if needed. Coming to step 6, just click on *finish* to launch the discovery job.

Once the discovery job complete, you can display the results by clicking on *job results*. All the available Host Templates
corresponding to the discovered Azure resources will be automatically set, like in the example below:

![image](../../../assets/integrations/plugin-packs/procedures/cloud-azure-management-discover-results.png)

> Some discovered elements may come up without any predefined Host Template; this is usually due to the following reasons:
> - These elements are not supposed to be monitored (Azure *technical* assets or dependencies of other assets)
> - These ressources cannot yet be monitored using the Centreon Plugin Packs

Just select the elements you want to add to the Centreon configuration and click on *save*. And... you're done !

## Troubleshooting

### The Azure credentials have changed and the Plugin does not work anymore

The Plugin is using a cache file to keep connection information and avoid an authentication at each call. 
If some of the authentication parameters change, you must delete the cache file. 

The cache file can be found within  ```/var/lib/centreon/centplugins/``` folder with a name similar to azure_api_`<md5>_<md5>_<md5>_<md5>`.

### ```UNKNOWN: Login endpoint API returns error code 'ERROR_NAME' (add --debug option for detailed message)```

When I run my command I obtain the following error message:
```UNKNOWN: Login endpoint API returns error code 'ERROR_NAME' (add --debug option for detailed message)```.

It means that some parameters used to authenticate the API request are wrong. The 'ERROR_NAME' string gives 
some hints about where the problem stands. 

### ```UNKNOWN: 500 Can't connect to login.microsoftonline.com:443```

This error message means that the Centreon Plugin couldn't successfully connect to the Azure Login API. Check that no third party
device (such as a firewall) is blocking the request. A proxy connection may also be necessary to connect to the API.
