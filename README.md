# Installation of Oracle Cloud Logging Data Source for Grafana

## Introduction

Grafana is a popular technology that makes it easy to visualize metrics and logs. The OCI Logging Grafana Plugin can be used to extend Grafana by adding OCI Logging as a data source in Grafana. 
The plugin allows you to retrieve logs related to a number of OCI resources: Compute, Networking, Storage, and custom logs. 
Once these logs are in Grafana, they can be analysed along with metrics, giving you a single pane of glass for application monitoring. 

This walk-through is intended for those who want to deploy Grafana and the OCI Logging service as a data source.
We will specifically focus on installation on local Mac-OS and 
Make sure you have access to the Logging service configured for the resources you want to fetch logs for.

For custom logs from your application, see [Custom Logging on OCI](https://docs.cloud.oracle.com/en-us/iaas/Content/Logging/Concepts/custom_logs.htm).
## Grafana IAM configuration 

### For Mac-OS
#### Install the Oracle Cloud Infrastructure CLI 

The [Oracle Cloud Infrastructure CLI](https://docs.cloud.oracle.com/iaas/Content/API/Concepts/cliconcepts.htm) provides you with a way to perform tasks in OCI from your command line rather than the OCI Console. It does so by making REST calls to the [OCI APIs](https://docs.cloud.oracle.com/iaas/Content/API/Concepts/usingapi.htm). We will be using the CLI to authenticate between our local environment hosting Grafana and OCI in order to pull in metrics. The CLI is built on Python (version 2.7.5 or 3.5 or later), running on Mac, Windows, or Linux.

Begin by [installing the Oracle Cloud Infrastructure CLI](https://docs.cloud.oracle.com/iaas/Content/API/SDKDocs/cliinstall.htm). Follow the installation prompts to install the CLI on your local environment. After the installation is complete, use the `oci setup config` command to have the CLI walk you through the first-time setup process. If you haven't already uploaded your public API signing key through the console, follow the instructions [here](https://docs.us-phoenix-1.oraclecloud.com/Content/API/Concepts/apisigningkey.htm#How2) to do so. 

#### Configure OCI Identity Policies

In the OCI console under **Identity > Groups** click **Create Group** and create a new group called **GrafanaLoggingUserGroup**. Add the user configured in the OCI CLI to the newly-created group. 

![usrGp](images/usrGp.png)

Under the **Policy** tab switch to the root compartment and click **Create Policy**. Create a policy allowing the group to read tenancy metrics. Add the following policy statements:

- `allow group GrafanaLoggingUserGroup to read log-groups in tenancy`
- `allow group GrafanaLoggingUserGroup to read log-content in tenancy`

![usrPolicy](images/usrPolicy.png)

### For compute-instance/VM on OCI
#### Create Dynamic Group for your instance 
Provision an Oracle Linux [virtual machine](https://docs.cloud.oracle.com/iaas/Content/Compute/Concepts/computeoverview.htm) in OCI connected to a [Virtual Cloud Network](https://docs.cloud.oracle.com/iaas/Content/Network/Tasks/managingVCNs.htm) with access to the public internet. If you do not already have access to a Virtual Cloud Network with access to the public internet you can navigate to **Virtual Cloud Networks** under **Networking** and click **Create Virtual Cloud Network**. Choosing the `CREATE VIRTUAL CLOUD NETWORK PLUS RELATED RESOURCES` option will result in a VCN with an Internet Routing Gateway and Route Tables configured for access to the public internet. Three subnets will be created: one in each availability domain in the region.

After creating your VM, the next step is to create a [dynamic group](https://docs.cloud.oracle.com/iaas/Content/Identity/Tasks/managingdynamicgroups.htm) used to group virtual machine or bare metal compute instances as “principals” (similar to user groups).
You can define the dynamic group similar to below, where your instance is part of the compartment given in the definition of the dynamic group.
![dgGroup](images/dgGroup.png)
#### Create IAM policy for Dynamic Group for your instance 

Next, create a [policy](https://docs.cloud.oracle.com/iaas/Content/Identity/Concepts/policygetstarted.htm) named “grafana_policy” in the root compartment of your tenancy to permit instances in the dynamic group to make API calls against Oracle Cloud Infrastructure services. Add the following policy statements:

* `allow dynamicgroup DynamicGroupForGrafanaInstances to read log-groups in tenancy`
* `allow dynamicgroup DynamicGroupForGrafanaInstances to read log-content in tenancy`

![dgPolicy](images/dgPolicy.png)

### Install Grafana and then OCI Logging Data Source for Grafana Plugin 

To [install the data source](https://grafana.com/plugins/oci-datasource/installation) make sure you are running [Grafana 3.0](https://grafana.com/get) or later.
For installing Grafana
on Mac-OS run: `brew install grafana`
on Oracle Linux compatible distro run: `sudo yum install grafana`

After Grafana installation, use the [grafana-cli tool](http://docs.grafana.org/plugins/installation/) to install the Oracle Cloud Logging Data Source for Grafana from the command line:

```
grafana-cli plugins install oci-logging-datasource
```
**NOTE** Today the latest version of the plugin is available only with the manual installation

The plugin will be installed into your Grafana plugins directory, which by default located at /var/lib/grafana/plugins. [Here is more information on the CLI tool](http://docs.grafana.org/plugins/installation/).

#### Manually installation 
 Alternatively, you can manually download the .tar file and unpack it into your /grafana/plugins directory. To do so, change to the Grafana plugins directory: `cd /usr/local/var/lib/grafana/plugins`. Download the OCI Grafana Plugin: wget `https://github.com/oracle/oci-grafana-plugin/releases/download/v2.0.0/plugin.tar`. Create a directory and install the plugin: `mkdir oci && tar -C oci -xvf plugin.tar` and then remove the tarball: `rm plugin.tar`

>  **Additional step for Grafana 7**. Open the grafana configuration  *grafana.ini* file and add the `allow_loading_unsigned_plugins = "oci-logging-datasource"`in the *plugins* section.

*Example* 
```
    [plugins]
    ;enable_alpha = false
    ;app_tls_skip_verify_insecure = false
    allow_loading_unsigned_plugins = "oci-logging-datasource"
```


To start the Grafana server run,
- on MAC OS : `brew services start grafana`
- on OCI Instance/VM: `sudo systemctl start grafana-server`

Navigate to the Grafana homepage 
for local Mac-OS installation: `http://localhost:3000`  
for OCI Instance installation: `http://<OCI Instance IP>:3000`
Make sure OCI instance IP and port are accessible to you.
To find the IP address of the newly-created instance, in the OCI Console go to Compute > Instances > [Your Instance]. The Public IP address is listed under the Primary VNIC Information section. 
You can also connect to Grafana running on your VM, locally via port forwarding, by running:
`ssh opc@[OCI Instance Public IP] -L 3000:localhost:3000`



### Configure Grafana

![admin_admin](images/admin_admin.png)

Log in with the default username `admin` and the password `admin`. You will be prompted to change your password. Click **Skip** or **Save** to continue. 

![changePdAdmin](images/changePdAdmin.png)

On the Home Page of Grafana, click **Add data source**.

![adUrFirstDS](images/adUrFirstDS.png)

Here you will see **Search Box** as shown below
![searchDS](images/searchDS.png)

Search with **Oracle Cloud Infrastructure Logs** as your data source type.
![logds](images/logds.png)

Fill in your **Tenancy OCID**, **Default Region**, and **Environment**. For **Environment** choose **local**. 

Click **Save & Test** to return to the home dashboard. 

![ociLogsDSsetup](images/ociLogsDSsetup.png)

### Next Steps to use the OCI Logs as data source for your Grafana panel

As shown below in the image, follow the steps 
1. Select **Oracle Cloud Infrastructure Logs** as data source from the dropdown
2. Select **OCI region** from where you want your logs to be fetched and shown in the panel
3. Write the query as per [OCI Logging Query Language](https://docs.cloud.oracle.com/en-us/iaas/Content/Logging/Reference/query_language_specification.htm)
4. Choose visualization type as logs for the panel
5. You should see the logs flowing in the panel shown
![usePlugin](images/usePlugin.png)
