---
id: process-metrics
title: Host and Process Metrics
---

The Sumo Logic App for Host and Process Metrics allows you to monitor the performance and resource utilization of hosts and processes that your mission critical applications are dependent upon. Preconfigured dashboards provide insight into CPU, memory, network, file descriptors, page faults, and TCP connectors. This app uses Telegraf, an open-source, plugin-based collector for the collection of both host and process metrics data.

1.png "image_tooltip")

This app uses Telegraf and associated input plugins to collect both host and process metrics. To use the installed collector to collect and analyze host metrics, please see the [Host Metrics app](https://help.sumologic.com/07Sumo-Logic-Apps/14Hosts_and_Operating_Systems/Host_Metrics).

This app has been validated on Linux(Ubuntu 20.04.2 LTS) and  Windows (Microsoft Windows Server 2019) and higher using Telegraf 1.18.2. This app is not recommended in Kubernetes environments; instead please use the [Kubernetes app](https://help.sumologic.com/07Sumo-Logic-Apps/10Containers_and_Orchestration/Kubernetes).


#### Sample Queries

**CPU Utilization by Host** Panel in **Host Metrics - CPU** Dashboard


```
host.name=*  cpu=cpu-total  metric=(host_cpu_usage_user OR host_cpu_usage_system OR host_cpu_usage_iowait OR host_cpu_usage_steal OR host_cpu_usage_softirq OR host_cpu_usage_irq OR host_cpu_usage_nice) | sum by host.name
```


**CPU Usage** Panel in **Process Metrics Details** Dashboard


```
metric=procstat_cpu_usage host.name=*  process.executable.name=* | avg by host.name, process.executable.name | outlier
```



## Collect Metrics for Host and Processes

We use the Telegraf agent for Host and Process metrics collection. Telegraf runs on the same system and uses the input plugins to obtain host and process metrics, and the Sumo Logic output plugin to send the metrics to Sumo Logic.

The following input plugins are used by Sumo Logic

For Linux:

[net](https://github.com/influxdata/telegraf/blob/master/plugins/inputs/net/NET_README.md)

[Diskio](https://github.com/influxdata/telegraf/tree/release-1.15/plugins/inputs/diskio)

[Disk](https://github.com/influxdata/telegraf/tree/release-1.15/plugins/inputs/disk)

[Netstat](https://github.com/influxdata/telegraf/blob/master/plugins/inputs/net/NETSTAT_README.md)

[Mem](https://github.com/influxdata/telegraf/tree/release-1.15/plugins/inputs/mem)

[CPU](https://github.com/influxdata/telegraf/blob/release-1.15/plugins/inputs/cpu/README.md)

[System](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/system)

[Procstat](https://github.com/influxdata/telegraf/blob/release-1.15/plugins/inputs/procstat/README.md)

[Process](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/processes)

For Windows:

[net](https://github.com/influxdata/telegraf/blob/master/plugins/inputs/net/NET_README.md)

[Disk](https://github.com/influxdata/telegraf/tree/release-1.15/plugins/inputs/disk)

[Netstat](https://github.com/influxdata/telegraf/blob/master/plugins/inputs/net/NETSTAT_README.md)

[Mem](https://github.com/influxdata/telegraf/tree/release-1.15/plugins/inputs/mem)

[CPU](https://github.com/influxdata/telegraf/blob/release-1.15/plugins/inputs/cpu/README.md)

[System](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/system)

[Procstat](https://github.com/influxdata/telegraf/blob/release-1.15/plugins/inputs/procstat/README.md)

[Win_Perf_counters](https://github.com/influxdata/telegraf/blob/master/plugins/inputs/win_perf_counters/README.md)


#### Configuring Metrics Collection

This section provides instructions for configuring metrics collection for the Sumo Logic App for Host and Process metrics. Follow the below instructions to set up the metric collection for a given machine:



1. Configure a Hosted Collector
2. Configure an HTTP Logs and Metrics Source
3. Install Telegraf
4. Configure and start Telegraf

    1. **Configure a Hosted Collector**


    To create a new Sumo Logic hosted collector, perform the steps in the[ Configure a Hosted Collector](https://help.sumologic.com/03Send-Data/Hosted-Collectors/Configure-a-Hosted-Collector)<span style="text-decoration:underline;"> </span>section of the Sumo Logic documentation.


    2. **Configure an HTTP Logs and Metrics Source**


    Create a new HTTP Logs and Metrics Source in the hosted collector created above by following [these instructions](https://help.sumologic.com/03Send-Data/Sources/02Sources-for-Hosted-Collectors/HTTP-Source). Suggestions for setting your source category:

1. For identifying a specific cluster or a group of hosts: `<clustername>/metrics`
2. For identifying a group of hosts within a given deployment: `<environment name>/<clustername>/metrics`

    Make a note of the HTTP Source URL and source category



2.png "image_tooltip")



    3. **Install Telegraf**


    Use the[ following steps](https://help.sumologic.com/03Send-Data/Collect-from-Other-Data-Sources/Collect_Metrics_Using_Telegraf/03_Install_Telegraf) to install Telegraf on each host machine.

* For [Windows](https://help.sumologic.com/03Send-Data/Collect-from-Other-Data-Sources/Collect_Metrics_Using_Telegraf/03_Install_Telegraf#install-telegraf-on-windows)
* For [Ubuntu](https://help.sumologic.com/03Send-Data/Collect-from-Other-Data-Sources/Collect_Metrics_Using_Telegraf/03_Install_Telegraf#install-telegraf-on-ubuntu-or-debian-with-apt-get)

    4. **Configure and start Telegraf**


    As part of collecting metrics data from Telegraf, we will use the input plugins (described earlier) to get data from Telegraf and the [Sumo Logic output plugin](https://github.com/SumoLogic/fluentd-output-sumologic) to send data to Sumo Logic.


    Create or modify telegraf.conf (in linux it’s located in `/etc/telegraf/telegraf.d/` and on Windows, it’s located in `C:\Program Files\InfluxData\Telegraf\`). Copy and paste the inputs, outputs and processors section  from one of the below files


    for Linux - [linux_sample_telegraf.conf](https://github.com/SumoLogic/terraform-sumologic-sumo-logic-monitor/blob/main/monitor_packages/host_process_metrics/host_process_metrics_linux_sample.conf)


    for Windows: [windows_sample_telegraf.conf](https://github.com/SumoLogic/terraform-sumologic-sumo-logic-monitor/blob/main/monitor_packages/host_process_metrics/host_process_metrics_windows_sample.conf)


    Please enter values for the following parameters (marked with `CHANGE_ME`) in the downloaded file:

* In the output plugins section which is `[[outputs.sumologic]]`:
    * URL - This is the HTTP source URL created in step 3. Please see [this doc](https://help.sumologic.com/03Send-Data/Collect-from-Other-Data-Sources/Collect_Metrics_Using_Telegraf/05_Configure_Telegraf_Output_Plugin_for_Sumo_Logic) for more information on additional parameters for configuring the Sumo Logic Telegraf output plugin.

    Here’s an explanation for additional values set by this Telegraf configuration that we request you to please **do not** modify these values as they will cause the Sumo Logic apps to not function correctly.

* data_format = “carbon2” In the output plugins section which is `[[outputs.sumologic]]`  This indicates that metrics should be sent in the carbon2 format to Sumo Logic

    For other optional parameters refer to [the respective plugin ](https://help.sumologic.com/07Sumo-Logic-Apps/14Hosts_and_Operating_Systems/Host_and_Process_Metrics/Collect_Metrics_for_Host_and_Processes#input+plugins)documentation for configuring the input plugins for Telegraf.


    For all other parameters please see [this doc](https://github.com/influxdata/telegraf/blob/master/docs/CONFIGURATION.md#agent) for more properties that can be configured in the Telegraf agent globally.


    Once you have finalized your `telegraf.conf` file, you can start or reload the telegraf service using instructions from the [doc](https://docs.influxdata.com/telegraf/v1.17/introduction/getting-started/#start-telegraf-service).


    At this point, host and process metrics should start flowing into Sumo Logic.



3.png "image_tooltip")
Please see Telegraf’s [Metric filtering capabilities](https://github.com/influxdata/telegraf/blob/master/docs/CONFIGURATION.md#metric-filtering) to exclude metrics that you do not need from being sent to Sumo Logic.


#### Monitoring processes with certain pattern

**exe:** Selects the processes that have process names that match the string that you specify

**pattern:** Selects the processes that have command lines (including parameters and options used with the command) matching the specified string using regular expression matching rules.

**For Linux**

On Linux servers, the strings that you specify in an exe or pattern section are evaluated as regular expressions.

Example

For filtering executable name containing nginx (i.e., p`grep <exe>`)


```
[[inputs.procstat]]
    pid_tag=false
    exe = "nginx"
```


For filtering command lines containing config (ie, `pgrep -f <pattern>`)


```
[[inputs.procstat]]
    pid_tag=false
        pattern = "config"
```


**For Windows**

On servers running Windows Server, these strings are evaluated as WMI queries. (Ex `pattern: "%apache%"`). For more information, see [LIKE Operator](https://docs.microsoft.com/en-us/windows/desktop/WmiSdk/like-operator).

Example

For filtering executable name containing nginx


```
[[inputs.procstat]]
    pid_finder = "native"
    pid_tag=false
    exe = "%nginx%"
```



For filtering command lines containing config


```
[[inputs.procstat]]
    pid_finder = "native"
    pid_tag=false
    pattern = "%config%"
```


For defining multiple patterns for multiple processes you can use the plugin multiple times


```
[[inputs.procstat]]
   pid_tag=false
   exe = "nginx"

[[inputs.procstat]]
   exe = "tomcat"
   pid_tag=false
```



#### Troubleshooting Section



1. To identify the operating system  version and name
    1. For Windows machines, run the command in Powershell to get the OS Version


```
[System.Environment]::OSVersion.Version
    (Get-WmiObject -class Win32_OperatingSystem).Caption
```


2. For Linux run below command in terminal


```
uname -a
    lsb_release -a

```



1. To enable debug logs, set `“debug = true”` flag in telegraf.conf and run the command, it will output error in stdout
```
    telegraf --config telegraf.conf --test
```
1. If the telegraf conf changes are not reflecting make sure to restart Telegraf using the command
    1. Windows - ​​ `./telegraf.exe --service restart`
    2. Linux - `sudo service telegraf restart`
2. If certain metrics are not coming you may have to run the telegraf agent as root. Check [the respective plugin](https://help.sumologic.com/07Sumo-Logic-Apps/14Hosts_and_Operating_Systems/Host_and_Process_Metrics/Collect_Metrics_for_Host_and_Processes#input+plugins) documentation for more information.


#### Sample Queries

**CPU Utilization by Host** Panel in **Host Metrics - CPU** Dashboard


```
host.name=*  cpu=cpu-total  metric=(host_cpu_usage_user OR host_cpu_usage_system OR host_cpu_usage_iowait OR host_cpu_usage_steal OR host_cpu_usage_softirq OR host_cpu_usage_irq OR host_cpu_usage_nice) | sum by host.name
```


**CPU Usage **Panel in ** Process Metrics Details** Dashboard


```
metric=procstat_cpu_usage host.name=*  process.executable.name=* | avg by host.name, process.executable.name | outlier

```



## Host and Process Metrics Alerts


#### Host Metrics Alerts

Sumo Logic provides out of the box alerts available via [Sumo Logic monitors](https://help.sumologic.com/Visualizations-and-Alerts/Alerts/Monitors). These alerts are built based on metrics datasets and have preset thresholds based on industry best practices and recommendations.

**insert table**


## Process Metrics Alerts

Sumo Logic provides out of the box alerts available via [Sumo Logic monitors](https://help.sumologic.com/Visualizations-and-Alerts/Alerts/Monitors). These alerts are built based on metrics datasets and have preset thresholds based on industry best practices and recommendations.



**insert table**


## Install the Host and Process Metrics app, Alerts, and view the Dashboards

This page provides instructions for installing the Sumo App and Alerts for hosts and processes, as well as the descriptions of each of the app dashboards. These instructions assume you have already set up a collection as described in the [Collect Metrics from Host and Processes](https://help.sumologic.com/07Sumo-Logic-Apps/14Hosts_and_Operating_Systems/Host_and_Process_Metrics/Collect_Metrics_for_Host_and_Processes) App page.


#### Pre-Packaged Alerts

Sumo Logic has provided out of the box alerts available through [Sumo Logic monitors](https://help.sumologic.com/Visualizations-and-Alerts/Alerts/Monitors) to help you monitor your hosts and processes. These alerts are built based on metrics and logs datasets and include preset thresholds based on industry best practices and recommendations.

For details on the individual alerts,  please see this [page](https://help.sumologic.com/07Sumo-Logic-Apps/14Hosts_and_Operating_Systems/Host_and_Process_Metrics/Host_and_Process_Metrics_Alerts).


#### Installing Alerts



* To install these alerts, you need to have the Manage Monitors role capability.
* Alerts can be installed by either importing them a JSON or a Terraform script.


4.png "image_tooltip")
There are limits to how many alerts can be enabled - please see the [Alerts FAQ](https://help.sumologic.com/Visualizations-and-Alerts/Alerts/Monitors/Monitor_FAQ) for details.


##### Method 1: Install the alerts by importing a JSON file



1. Download the [JSON file](https://github.com/SumoLogic/terraform-sumologic-sumo-logic-monitor/blob/main/monitor_packages/host_process_metrics/host_process_metrics.json) describing all the monitors.
    1. The JSON contains the alerts that are based on Sumo Logic searches that do not have any scope filters and therefore will be applicable to all hosts, the data for which has been collected via the instructions in the previous sections. However, if you would like to restrict these alerts to specific hosts or environments, update the JSON file by replacing the text **$$hostmetrics_data_source** with <your sourceCategory>. \
 \
SourceCategory examples:
        1. For alerts applicable only to a specific cluster of hosts, your custom filter could be:  **<code>'_sourceCategory=yourclustername/metrics'</code>.</strong>
        2. For alerts applicable to all hosts that start with ec2hosts-prod, your custom filter could be: <strong><code>'_sourceCategory=ec2hosts-prod*/metrics'</code></strong>.
        3. For alerts applicable to a specific cluster within a production environment, your custom filter could be: <strong><code>'_sourceCategory=prod/yourclustername/metrics'</code></strong>
2. Go to Manage Data > Alerts > Monitors.
3. Click <strong>Add</strong>: \

5.png "image_tooltip")

4. Click Import to import monitors from the JSON above.


6.png "image_tooltip")
The monitors are disabled by default. Once you have installed the alerts using this method, navigate to the Host and Process Metrics folder under Monitors to configure them. See [this](https://help.sumologic.com/Visualizations-and-Alerts/Alerts/Monitors) document to enable monitors, to configure each monitor, to send notifications to teams or connections please see the instructions detailed in [Alert Configuration](https://help.sumologic.com/07Sumo-Logic-Apps/14Hosts_and_Operating_Systems/Host_and_Process_Metrics/Install_the_Host_and_Process_Metrics_app%2C_Alerts%2C_and_view_the_Dashboards#Alert_Configuration) of this [document](https://help.sumologic.com/Visualizations-and-Alerts/Alerts/Monitors#Add_a_monitor).


##### Method 2: Install the alerts using a Terraform script


###### Generate a Sumo Logic access key and ID

Generate an access key and access ID for a user that has the Manage Monitors role capability in Sumo Logic using these[ instructions](https://help.sumologic.com/Manage/Security/Access-Keys#manage-your-access-keys-on-preferences-page). Please identify which deployment your Sumo Logic account is in, using this[ link](https://help.sumologic.com/APIs/General-API-Information/Sumo-Logic-Endpoints-by-Deployment-and-Firewall-Security).


###### [Download and install Terraform 0.13](https://www.terraform.io/downloads.html) or later


###### Download the Sumo Logic Terraform package for Host and Process alerts

The alerts package is available in the Sumo Logic GitHub [repository](https://github.com/SumoLogic/terraform-sumologic-sumo-logic-monitor/tree/main/monitor_packages/postgresql). You can either download it through the “git clone” command or as a zip file.


###### Alert Configuration

After the package has been extracted, navigate to the package directory t`erraform-sumologic-sumo-logic-monitor/monitor_packages/host_process_metrics/`

Edit the `host_and_processes.auto.tfvars` file and add the Sumo Logic Access Key, Access Id, and Deployment from [Generate a Sumo Logic access key and ID](https://help.sumologic.com/07Sumo-Logic-Apps/14Hosts_and_Operating_Systems/Host_and_Process_Metrics/Install_the_Host_and_Process_Metrics_app%2C_Alerts%2C_and_view_the_Dashboards#Generate_a_Sumo_Logic_access_key_and_ID).


```
access_id   = "<SUMOLOGIC ACCESS ID>"
access_key  = "<SUMOLOGIC ACCESS KEY>"
environment = "<SUMOLOGIC DEPLOYMENT>"
```


Update the variable `host_and_processes_data_source` with your source category:



1. SourceCategory examples:
    1. For alerts applicable only to a specific cluster of hosts, your custom filter could be: `_sourceCategory=yourclustername/metrics`.
    2. For alerts applicable to all hosts that start with ec2hosts-prod, your custom filter could be:`_sourceCategory=ec2hosts-prod*/metrics`.
    3. For alerts applicable to a specific cluster within a production environment, your custom filter could be: `_sourceCategory=prod/yourclustername/metrics`.

All monitors are disabled by default on installation. If you would like to enable all the monitors, set the parameter `monitors_disabled` to false in this file.

By default, the monitors are configured in a monitor folder called “Host and “Process Metrics”, if you would like to change the name of the folder, update the monitor folder name in this file.

If you would like the alerts to send email or connection notifications, configure these in the file `host_process_metrics_notifications.auto.tfvars`. For configuration examples, refer to the next section.


###### Email and Connection Notification Configuration Examples

To** configure notifications, m**odify the file `host_process_metrics_notifications.auto.tfvars` file and fill in the `connection_notifications` and `email_notifications` sections. See the examples for PagerDuty and email notifications below. See this [document](https://help.sumologic.com/Manage/Connections-and-Integrations/Webhook-Connections/Set_Up_Webhook_Connections) for creating payloads with other connection types.




###### Pagerduty Connection Example:


```
connection_notifications = [
    {
      connection_type       = "PagerDuty",
      connection_id         = "<CONNECTION_ID>",
      payload_override      = "{\"service_key\": \"your_pagerduty_api_integration_key\",\"event_type\": \"trigger\",\"description\": \"Alert: Triggered {{TriggerType}} for Monitor {{Name}}\",\"client\": \"Sumo Logic\",\"client_url\": \"{{QueryUrl}}\"}",
      run_for_trigger_types = ["Critical", "ResolvedCritical"]
    },
    {
      connection_type       = "Webhook",
      connection_id         = "<CONNECTION_ID>",
      payload_override      = "",
      run_for_trigger_types = ["Critical", "ResolvedCritical"]
    }
  ]
```


Replace `<CONNECTION_ID>` with the connection id of the webhook connection. The webhook connection id can be retrieved by calling the [Monitors API](https://api.sumologic.com/docs/#operation/listConnections).




###### Email Notifications Example:


```
email_notifications = [
    {
      connection_type       = "Email",
      recipients            = ["abc@example.com"],
      subject               = "Monitor Alert: {{TriggerType}} on {{Name}}",
      time_zone             = "PST",
      message_body          = "Triggered {{TriggerType}} Alert on {{Name}}: {{QueryURL}}",
      run_for_trigger_types = ["Critical", "ResolvedCritical"]
    }
  ]
```



###### Install the Alerts

1. Navigate to the package directory `terraform-sumologic-sumo-logic-monitor/monitor_packages/host_process_metrics/` and run **terraform init. **This will initialize Terraform and will download the required components.
2. Run **terraform plan **to view the monitors which will be created/modified by Terraform.
3. Run **terraform apply**.


###### Post Installation

If you haven’t enabled alerts or configured notifications through the Terraform procedure outlined above, we highly recommend enabling alerts of interest and configuring each enabled alert to send notifications to other people or services. This is detailed in Step 4 of [this document](https://help.sumologic.com/Visualizations-and-Alerts/Alerts/Monitors#Add_a_monitor).


#### Install the app

This section demonstrates how to install the Host and Process Metrics App.

Now that you have set up a log and metric collection for the Host and Process Metrics App, you can install the Sumo Logic App for Host and Processes to use the pre-configured searches and [Dashboards](https://help.sumologic.com/07Sumo-Logic-Apps/14Hosts_and_Operating_Systems/Host_and_Process_Metrics/Install_the_Host_and_Process_Metrics_app%2C_Alerts%2C_and_view_the_Dashboards#dashboard-filters-with-template-variables).

**To install the app, do the following:**

Locate and install the app you need from the **App Catalog**. If you want to see a preview of the dashboards included with the app before installing, click **Preview Dashboards**.



1. From the **App Catalog**, search for and select the app**.**
2. Select the version of the service you're using and click **Add to Library**. \

7.png "image_tooltip")
Version selection is applicable only to a few apps currently. For more information, see the [Install the Apps from the Library](https://help.sumologic.com/01Start-Here/Library/Apps-in-Sumo-Logic/Install-Apps-from-the-Library).
3. To install the app, complete the following fields.
    1. **App Name.** You can retain the existing name, or enter a name of your choice for the app. 
    2. **Data Source.** Select either of these options for the data source. 
* Choose **Source Category**, and select a source category from the list. 
    * Choose **Enter a Custom Data Filter**, and enter a custom source category beginning with an underscore. Example: (`_sourceCategory=MyCategory`). 
1. **Advanced**. Select the **Location in Library** (the default is the Personal folder in the library), or click **New Folder** to add a new folder.
2. Click **Add to Library.**

Once an app is installed, it will appear in your **Personal** folder, or another folder that you specified. From here, you can share it with your organization.

Panels will start to fill automatically. It's important to note that each panel slowly fills with data matching the time range query and received since the panel was created. Results won't immediately be available, but with a bit of time, you'll see full graphs and maps.


#### Filters with template variables   

Template variables provide dynamic dashboards that can rescope data on the fly. As you apply variables to troubleshoot through your dashboard, you view dynamic changes to the data for a quicker resolution to the root cause.** **For more information, see the [Filter with template variables](https://help.sumologic.com/Visualizations-and-Alerts/Dashboard_(New)/Filter_with_template_variables) help page.


8.png "image_tooltip")
You can use template variables to drill down and examine the data on a granular level.


#### Dashboards


##### Host Metrics - Overview Dashboard


9.png "image_tooltip")


The **Host Metrics - Overview** dashboard gives you an at-a-glance view of the key metrics like CPU, memory, disk, network, and TCP connections of all your hosts. You can drill down from this dashboard to the Host Metrics - CPU/Disk/Memory/Network/TCP dashboard by using the honeycombs or line charts in all the panels.

**Use this dashboard to:**



* Identify hosts with high CPU, disk, memory utilization, and identify anomalies over time.


##### Host Metrics - CPU


10.png "image_tooltip")


The **Host Metrics - CPU** dashboard provides a detailed analysis based on CPU metrics. You can drill down from this dashboard to the **Process Metrics - Details** dashboard by using the honeycombs or line charts in all the panels.

**Use this dashboard to:**



* Identify hosts and processes with high CPU utilization.
* Examine CPU usage by type and identify anomalies over time.


##### Host Metrics - Disk


11.png "image_tooltip")


The **Host Metrics - Disk** dashboard provides detailed information about on disk utilization and disk IO operations.You can drill down from this dashboard to the **Process Metrics - Details** dashboard by using the honeycombs or line charts in all the panels.

**Use this dashboard to:**



* Identify hosts with high disk utilization and disk IO operations.
* Monitor abnormal spikes in read/write rates.
* Compare disk throughput across storage devices of a host.


##### Host Metrics - Memory


12.png "image_tooltip")


The **Host Metrics - Memory** dashboard provides detailed information on host memory usage, memory distribution, and swap space utilization. You can drill down from this dashboard to the **Process Metrics - Details** dashboard by using the honeycombs or line charts in all the panels.

**Use this dashboard to:**



* Identify hosts with high memory utilization.
* Examine memory distribution (free, buffered-cache, used, total) for a given host.
* Monitor abnormal spikes in memory and swap utilization.


##### Host Metrics - Network


13.png "image_tooltip")


The **Host Metrics - Network** dashboard provides detailed information on host network errors, throughput, and packets sent and received.

**Use this dashboard to:**



* Determine top hosts with network errors and dropped packets.
* Monitor abnormal spikes in incoming/outgoing packets and bytes sent and received.
* Use dashboard filters to compare throughput across the interface of a host.


##### Host Metrics - TCP


14.png "image_tooltip")


The **Host Metrics - TCP** dashboard provides detailed information around inbound, outbound, open, and established TCP connections.

**Use this dashboard to:**



* Identify abnormal spikes in inbound, outbound, open, or established connections.


##### Process Metrics - Overview Dashboard


15.png "image_tooltip")


The **Process Metrics - Overview** dashboard gives you an at-a-glance view of all the processes by open file descriptors,  CPU usage, memory usage, disk read/write operations and thread count.You can drill down from this dashboard to the **Process Metrics - Details** dashboard by using the honeycombs or line charts in all the panels.

**Use this dashboard to:**



* Identify top processes by CPU, memory usage, and open file descriptors.
* Determine the longest running processes and users that have spawned the most number of processes.


##### Process Metrics - Details Dashboard


16.png "image_tooltip")


The **Process Metrics - Details** dashboard gives you a detailed view of key process related metrics such as CPU and memory utilization, disk read/write throughput, and major/minor page faults.

**Use this dashboard to:**



* Determine the number of open file descriptors in all hosts. If the number of open file descriptors reaches the maximum file descriptor limits,, it can cause IOException errors.
* Identify anomalies in CPU usage, memory usage,  major/minor page faults and reads/writes over time.
* Troubleshoot memory leaks using the resident set memory trend chart.


##### Process Metrics - Trends Dashboard


17.png "image_tooltip")


The **Process Metrics - Trend** dashboard gives you insight into the state of your processes over time.

**Use this dashboard to:**



* Analyze the current state of all the processes (sleeping, dead, idle, stopped, total, paging)
* Identify anomalies over time in the number of threads, zombie processes, and total processes