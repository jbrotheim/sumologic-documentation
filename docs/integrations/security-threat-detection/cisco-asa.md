---
id: cisco-asa
title: Cisco ASA App
sidebar_label: Cisco ASA
description: The Cisco ASA App gives you insight into website visitor patterns, monitors infrastructure operations, and provides easy access to threat monitoring. The App uses a predefined parser, searches, and Dashboards which provide visibility into your environment for analysis of overall usage and threats.
---

The Cisco ASA App gives you insight into website visitor patterns, monitors infrastructure operations, and provides easy access to threat monitoring. The App uses a predefined parser, searches, and Dashboards which provide visibility into your environment for analysis of overall usage and threats. The Cisco ASA App consists of the following main categories:

* Connection statistics
* Outbound connections
* Denied connections


##### Log Types
1


The Cisco ASA App assumes logs from the Cisco Adaptive Security Appliance (ASA) firewall product.


## Collect Logs for the Cisco ASA App

This page provides instructions for configuring log collection for the Cisco ASA App, as well as a sample log and field extraction rule.


### Prerequisites
2


Configure your ASA to send its logs to a syslog server. ASA sends syslog on UDP port 514 by default, but you can set the protocol and port.


### Configure log collection for the Cisco ASA App
3

To configure log collection, do the following:

1. Configure an [Installed Collector](https://help.sumologic.com/03Send-Data/Installed-Collectors) appropriate for right for your host environment.
2. Configure a [Syslog Source](https://help.sumologic.com/03Send-Data/Sources/01Sources-for-Installed-Collectors/Syslog-Source) to the same port and protocol used by your ASA.


### Sample Log
4

```
Tue Aug 15 23:30:09 %ASA-6-302016: Teardown UDP connection 40 for outside:44.44.4.4/500 to inside:44.44.2.2/500 duration 0:02:02 bytes 1416
```

### Field Extraction Rule

This Field Extraction Rule (FER) is provided as an example to help you reduce your overall parsing time. Note that not all parse operators are supported in FERs. For more information, see [Creating a Field Extraction Rule](https://help.sumologic.com/Manage/Field-Extractions/Create-a-Field-Extraction-Rule).


5
There is a 200 field name limit for Field Extraction Rules (FER) and once a field is persisted using a FER, it can’t be removed. You can assign different targets to the name, but do not create overlapping messages and source categories.


```
| parse regex "(?<protocol>TCP|tcp|UDP|udp|ICMP|icmp)" nodrop
| parse regex "%[\w-]+(?<log_level>\d)-(?<message_id>\d{6})" nodrop
| parse regex "bytes\s(?<bytes_in>\d*)" nodrop
| parse regex "(?<direction>[i|I]nbound|[O|o]utbound)" nodrop
| parse regex "(?:\(type\s(?<icmp_type>[^,]+),\scode\s(?<icmp_code>[^\)]+)\))?\s+by\s+access-group\s+\"\+(?<rule_name>[^\"]+)" nodrop
| parse regex "(?i)icmp\s*type=(?<icmp_type>\d+)" nodrop
| parse regex "\d{2}:\d{2}:\d{2}(?:\-\d{2}:\d{2})?\s(?<dvc>[^\s]+)" nodrop
| parse regex "(?<src_translated_ip>(?:[0-9A-Fa-f]{1,4}:(?:[0-9A-Fa-f]{1,4}:(?:[0-9A-Fa-f]{1,4}:(?:[0-9A-Fa-f]{1,4}:(?:[0-9A-Fa-f]{1,4}:[0-9A-Fa-f]{0,4}|:[0-9A-Fa-f]{1,4})?|(?::[0-9A-Fa-f]{1,4}){0,2})|(?::[0-9A-Fa-f]{1,4}){0,3})|(?::[0-9A-Fa-f]{1,4}){0,4})|:(?::[0-9A-Fa-f]{1,4}){0,5})(?:(?::[0-9A-Fa-f]{1,4}){2}|:(?:25[0-5]|(?:2[0-4]|1[0-9]|[1-9])?[0-9])(?:\.(?:25[0-5]|(?:2[0-4]|1[0-9]|[1-9])?[0-9])){3})|(?:(?:[0-9A-Fa-f]{1,4}:){1,6}|:):[0-9A-Fa-f]{0,4}|(?:[0-9A-Fa-f]{1,4}:){7}:)/(?:\d+)\)\s(?<src_translated_port>to|dst)\s" nodrop
| parse regex "\s+(?:to|dst(?! user)) (?:(?<dest_zone>\S+):)[\w-]*?(?<dest_ip>\d{1,3}\.\d{1,3}.\d{1,3}.\d{1,3})(?:\/(?<dest_port>\d+))?\s*(?:\(?(?<dest_translated_ip>\d{1,3}\.\d{1,3}.\d{1,3}.\d{1,3})?\/?(?<dest_translated_port>\d+)?\))?\s*(?:\((?:(?<dest_nt_domain>[\S^\\]+)\\)?(?<dest_user>[\w\-_]+)\))?" nodrop
| parse regex "\s(?:for|from|src)\s+(?:(?<src_zone>\S+):)?(?<src_ip>[\d\.]+)\/(?<src_port>\d+)\s+(?:to|dst)\s+(?:(?<dest_zone>[^:]+):)?(?<dest_ip>[\d\.]+)\/(?<dest_port>\w+)\s+" nodrop
| parse regex "\s+[Aa]ddress\s*(?<dest_ip>\d{1,3}\.\d{1,3}.\d{1,3}.\d{1,3})(?:/(?<dest_port>\d+))?\s*[Dd]iscovered\s*for\s*domain\s*(?<dest_nt_domain>[\S]+)" nodrop
| parse regex "\s+(?:to|dst(?! user)) (?:(?<dest_zone>[^\/]+)\/)?(?<dest_ipv6>(?:[0-9A-Fa-f]{1,4}:(?:[0-9A-Fa-f]{1,4}:(?:[0-9A-Fa-f]{1,4}:(?:[0-9A-Fa-f]{1,4}:(?:[0-9A-Fa-f]{1,4}:[0-9A-Fa-f]{0,4}|:[0-9A-Fa-f]{1,4})?|(?::[0-9A-Fa-f]{1,4}){0,2})|(?::[0-9A-Fa-f]{1,4}){0,3})|(?::[0-9A-Fa-f]{1,4}){0,4})|:(?::[0-9A-Fa-f]{1,4}){0,5})(?:(?::[0-9A-Fa-f]{1,4}){2}|:(?:25[0-5]|(?:2[0-4]|1[0-9]|[1-9])?[0-9])(?:\.(?:25[0-5]|(?:2[0-4]|1[0-9]|[1-9])?[0-9])){3})|(?:(?:[0-9A-Fa-f]{1,4}:){1,6}|:):[0-9A-Fa-f]{0,4}|(?:[0-9A-Fa-f]{1,4}:){7}:)(?:\/(?<dest_port>\S+))?\s*" nodrop
| parse regex "\s(?:to|dest)\s(?:[a-f|A-F|0-9|:]+)(?:\/\S+)?\s*\((?<dest_translated_ip>(?:[0-9A-Fa-f]{1,4}:(?:[0-9A-Fa-f]{1,4}:(?:[0-9A-Fa-f]{1,4}:(?:[0-9A-Fa-f]{1,4}:(?:[0-9A-Fa-f]{1,4}:[0-9A-Fa-f]{0,4}|:[0-9A-Fa-f]{1,4})?|(?::[0-9A-Fa-f]{1,4}){0,2})|(?::[0-9A-Fa-f]{1,4}){0,3})|(?::[0-9A-Fa-f]{1,4}){0,4})|:(?::[0-9A-Fa-f]{1,4}){0,5})(?:(?::[0-9A-Fa-f]{1,4}){2}|:(?:25[0-5]|(?:2[0-4]|1[0-9]|[1-9])?[0-9])(?:\.(?:25[0-5]|(?:2[0-4]|1[0-9]|[1-9])?[0-9])){3})|(?:(?:[0-9A-Fa-f]{1,4}:){1,6}|:):[0-9A-Fa-f]{0,4}|(?:[0-9A-Fa-f]{1,4}:){7}:)(?:\/(?<dest_translated_port>\S+))?\)" nodrop
| parse regex "\sfaddr (?:(?<dest_zone>\S+):)?(?<dest_ip>\d{1,3}\.\d{1,3}.\d{1,3}.\d{1,3})(?:\/(?<dest_port>\d+))?\s*(?:\((?:(?<dest_nt_domain>[^\\]+)\\)?(?<dest_user>[^\)]+)\))?" nodrop
| parse regex "\sfaddr\s(?:(?<dest_zone>[^\/]+)\/)?(?<dest_ipv6>(?:[0-9A-Fa-f]{1,4}:(?:[0-9A-Fa-f]{1,4}:(?:[0-9A-Fa-f]{1,4}:(?:[0-9A-Fa-f]{1,4}:(?:[0-9A-Fa-f]{1,4}:[0-9A-Fa-f]{0,4}|:[0-9A-Fa-f]{1,4})?|(?::[0-9A-Fa-f]{1,4}){0,2})|(?::[0-9A-Fa-f]{1,4}){0,3})|(?::[0-9A-Fa-f]{1,4}){0,4})|:(?::[0-9A-Fa-f]{1,4}){0,5})(?:(?::[0-9A-Fa-f]{1,4}){2}|:(?:25[0-5]|(?:2[0-4]|1[0-9]|[1-9])?[0-9])(?:\.(?:25[0-5]|(?:2[0-4]|1[0-9]|[1-9])?[0-9])){3})|(?:(?:[0-9A-Fa-f]{1,4}:){1,6}|:):[0-9A-Fa-f]{0,4}|(?:[0-9A-Fa-f]{1,4}:){7}:)\/?(?<dest_port>\d*)" nodrop
| parse regex "\sladdr (?:(?<src_zone>\S+):)?(?<src_ip>\d{1,3}\.\d{1,3}.\d{1,3}.\d{1,3})(?:\/(?<src_port>\S+))?\s*" nodrop
| parse regex "\sladdr\s(?:(?<src_zone>[^\/]+)\/)?(?<src_ipv6>(?:[0-9A-Fa-f]{1,4}:(?:[0-9A-Fa-f]{1,4}:(?:[0-9A-Fa-f]{1,4}:(?:[0-9A-Fa-f]{1,4}:(?:[0-9A-Fa-f]{1,4}:[0-9A-Fa-f]{0,4}|:[0-9A-Fa-f]{1,4})?|(?::[0-9A-Fa-f]{1,4}){0,2})|(?::[0-9A-Fa-f]{1,4}){0,3})|(?::[0-9A-Fa-f]{1,4}){0,4})|:(?::[0-9A-Fa-f]{1,4}){0,5})(?:(?::[0-9A-Fa-f]{1,4}){2}|:(?:25[0-5]|(?:2[0-4]|1[0-9]|[1-9])?[0-9])(?:\.(?:25[0-5]|(?:2[0-4]|1[0-9]|[1-9])?[0-9])){3})|(?:(?:[0-9A-Fa-f]{1,4}:){1,6}|:):[0-9A-Fa-f]{0,4}|(?:[0-9A-Fa-f]{1,4}:){7}:)\/?(?<src_port>\d*)" nodrop
| parse regex "\sgaddr (?<src_public_ip>\d{1,3}\.\d{1,3}.\d{1,3}.\d{1,3})\/?(?<src_public_port>\d*)" nodrop
| parse regex "\sgaddr (?<src_public_ip>(?:[0-9A-Fa-f]{1,4}:(?:[0-9A-Fa-f]{1,4}:(?:[0-9A-Fa-f]{1,4}:(?:[0-9A-Fa-f]{1,4}:(?:[0-9A-Fa-f]{1,4}:[0-9A-Fa-f]{0,4}|:[0-9A-Fa-f]{1,4})?|(?::[0-9A-Fa-f]{1,4}){0,2})|(?::[0-9A-Fa-f]{1,4}){0,3})|(?::[0-9A-Fa-f]{1,4}){0,4})|:(?::[0-9A-Fa-f]{1,4}){0,5})(?:(?::[0-9A-Fa-f]{1,4}){2}|:(?:25[0-5]|(?:2[0-4]|1[0-9]|[1-9])?[0-9])(?:\.(?:25[0-5]|(?:2[0-4]|1[0-9]|[1-9])?[0-9])){3})|(?:(?:[0-9A-Fa-f]{1,4}:){1,6}|:):[0-9A-Fa-f]{0,4}|(?:[0-9A-Fa-f]{1,4}:){7}:)\/?(?<src_public_port>\d*)" nodrop
| parse regex "(?:(?<src_zone>\S+)\/)?(?<src_ip>\d{1,3}.\d{1,3}.\d{1,3}.\d{1,3})\((?<src_port>\d*)\)\s\->\s(?:(?<dest_zone>\S+)\/)?(?<dest_ip>\d{1,3}.\d{1,3}.\d{1,3}.\d{1,3})\((?<dest_port>\d*)\)" nodrop
| parse regex "\s+(?:from|for|src(?! user)) (?:(?<src_zone>[^\/]+)\/)?(?<src_ipv6>(?:[0-9A-Fa-f]{1,4}:(?:[0-9A-Fa-f]{1,4}:(?:[0-9A-Fa-f]{1,4}:(?:[0-9A-Fa-f]{1,4}:(?:[0-9A-Fa-f]{1,4}:[0-9A-Fa-f]{0,4}|:[0-9A-Fa-f]{1,4})?|(?::[0-9A-Fa-f]{1,4}){0,2})|(?::[0-9A-Fa-f]{1,4}){0,3})|(?::[0-9A-Fa-f]{1,4}){0,4})|:(?::[0-9A-Fa-f]{1,4}){0,5})(?:(?::[0-9A-Fa-f]{1,4}){2}|:(?:25[0-5]|(?:2[0-4]|1[0-9]|[1-9])?[0-9])(?:\.(?:25[0-5]|(?:2[0-4]|1[0-9]|[1-9])?[0-9])){3})|(?:(?:[0-9A-Fa-f]{1,4}:){1,6}|:):[0-9A-Fa-f]{0,4}|(?:[0-9A-Fa-f]{1,4}:){7}:)(?:\/(?<src_port>\S+))?\s*" nodrop
| parse regex "\s\->\s(?:(?<dest_zone>\S+)\/)?(?<dest_ipv6>(?:[0-9A-Fa-f]{1,4}:(?:[0-9A-Fa-f]{1,4}:(?:[0-9A-Fa-f]{1,4}:(?:[0-9A-Fa-f]{1,4}:(?:[0-9A-Fa-f]{1,4}:[0-9A-Fa-f]{0,4}|:[0-9A-Fa-f]{1,4})?|(?::[0-9A-Fa-f]{1,4}){0,2})|(?::[0-9A-Fa-f]{1,4}){0,3})|(?::[0-9A-Fa-f]{1,4}){0,4})|:(?::[0-9A-Fa-f]{1,4}){0,5})(?:(?::[0-9A-Fa-f]{1,4}){2}|:(?:25[0-5]|(?:2[0-4]|1[0-9]|[1-9])?[0-9])(?:\.(?:25[0-5]|(?:2[0-4]|1[0-9]|[1-9])?[0-9])){3})|(?:(?:[0-9A-Fa-f]{1,4}:){1,6}|:):[0-9A-Fa-f]{0,4}|(?:[0-9A-Fa-f]{1,4}:){7}:)\((?<dest_port>\d*)\)" nodrop
| parse regex "%ASA-\d-400\d+\sIPS:(?<signature_id>\d{4})\s(?<signature>.*)\sfrom" nodrop
| parse regex "access[\s-]group\s[\(\"]?(?<acl>[^\s\"\)]+)" nodrop
| parse regex "(?<src_ip>\d{1,3}\.\d{1,3}.\d{1,3}.\d{1,3})\s+(?<vendor_action>\S+)\s+(?:url|URL)\s+(?<dest_ip>\d{1,3}\.\d{1,3}.\d{1,3}.\d{1,3})\s*:\s*(?<url>\S*)" nodrop
| parse regex "threat-level\s*:\s*(?<vendor_severity>[^\s,]+)[\s,]+category\s*:\s*(?<vendor_category>[^\s,]+)" nodrop
| parse regex "(?<action>[Aa]uthentication [Ss]ucceeded|[Aa]uthorization [Pp]ermitted|authentication Successful|passed authentication|Login permitted|Authentication failed|Authorization denied|Can't find authorization|Authentication Failed|authentication Rejected|credentials rejected|Authentication:Dropping|login warning|login failed|failed authentication|[Cc]onnection denied|Deny inbound|Deny|Terminating|action locally|Unable to Pre-allocate|denied\s[tcp|udp|icmp]+|access denied|access requested|access permitted|limit exceeded|Dropped|Dropping|[B|b]uilt|[pP]ermitted|whitelisted|Pre-allocated|Rebuilt|redirected|discarded)" nodrop
| parse regex "(?<action>Teardown\s[A-Z]{3,4})\sconnection" nodrop
| parse regex "%ASA-\d-\d+: (?<msg>.+)" nodrop
| parse regex "\sconnection (?<session_id>\d+)" nodrop
| parse regex "access-list (?<rule>[^\s]+)" nodrop
| parse regex "[Dd]uration:?\s*(?:(?<duration_day>\d+)[dD])?\s*(?<duration_hour>\d+)[Hh]?:(?<duration_minute>\d+)[Mm]?:(?<duration_second>\d+)[Ss]?" nodrop
| parse regex "[Gg]roup\s+(?:=\s+)?(?<group>[^\s,]+)" nodrop
| parse regex "User <(?<user>\S+)>" nodrop
| parse regex "IP <(?<src_ip>\d{1,3}\.\d{1,3}.\d{1,3}.\d{1,3})>" nodrop
| parse regex "[Tt]unnel[Gg]roup\s+(?:=\s+)?(?<tunnelgroup>[^\s,]+)" nodrop | parse regex "Bytes xmt: (?<bytes_out>\d+), Bytes rcv: (?<bytes_in>\d+), Reason: (?<reason>.+)" nodrop
| parse regex "Authentication: (?<action>\S+)," nodrop
| parse regex "Session Type: (?<type>\S+)," nodrop
| parse regex "user-identity: (?<method>Add|Delete) (?:\S+) mapping (?<src_ip>\d{1,3}\.\d{1,3}.\d{1,3}.\d{1,3}) - (?<user>\S+) (?<action>\S+) - (?<reason>.+)" nodrop
| parse regex "[Aa]ddress\s\<?(?<assigned_ip>\d{1,3}\.\d{1,3}.\d{1,3}.\d{1,3})\>?\s" nodrop
| parse regex "DAP: User (?<user>\S+), Addr (?<src_ip>\d{1,3}\.\d{1,3}.\d{1,3}.\d{1,3})" nodrop
| parse regex "Connection (?<type>\S+), (?<dap_message>.+)" nodrop
| parse regex "acl\s*=\s*(?<acl>[^,\s\)]+)" nodrop
| parse regex "[gG]roup\s*=\s*(?<group>[^,\s\)]+)" nodrop
| parse regex "(?:[uU]sername|[uU]ser)\s*=\s*(?<user>[^,\s\)]+)" nodrop
| parse regex "msgid\s*=\s*(?<msgid>[^,\s\)]+)" nodrop
| parse regex "[Oo]utbound\s+\S+\s+connection\s+\d+\s+for\s+\S+\s*:\s*(?<dest_ip>[^\s\/\(]+)(?:\/(?<dest_port>\w+))?(?:\((?<dest_user>\S+)\))?\s*\(?(?<dest_translated_ip>[^\s\/\(]+)?\/?(?<dest_translated_port>\d+)?\)?\s+to\s+[^:]+:\s*(?<src_ip>[^\s\/\(]+)(?:\/(?<src_port>\w+))?(?:\((?<src_user>\S+)\))?\s*\(?(?<src_translated_ip>[^\s\/\(]+)?\/?(?<src_translated_port>\d+)?\)?"

```


## Install the Cisco ASA App and view the Dashboards


This page provides instructions on how to install the Cisco ASA App, and examples of each of the dashboards. The App preconfigured searches and [Dashboards](https://help.sumologic.com/07Sumo-Logic-Apps/22Security_and_Threat_Detection/Cisco_ASA/Cisco-ASA-App-Dashboards#Dashboards) provide easy-to-access visual insights into your data.


### Install the Cisco ASA App  
6


This section shows you how to install the Cisco ASA App.

**To install the app, do the following:**

Locate and install the app you need from the **App Catalog**. If you want to see a preview of the dashboards included with the app before installing, click **Preview Dashboards**.



1. From the **App Catalog**, search for and select the app**.**
2. Select the version of the service you're using and click **Add to Library**.


7
Version selection is applicable only to a few apps currently. For more information, see the [Install the Apps from the Library.](https://help.sumologic.com/01Start-Here/Library/Apps-in-Sumo-Logic/Install-Apps-from-the-Library)



1. To install the app, complete the following fields.
    1. **App Name.** You can retain the existing name, or enter a name of your choice for the app. 
    2. **Data Source.** Select either of these options for the data source. 
        * Choose **Source Category**, and select a source category from the list. 
        * Choose **Enter a Custom Data Filter**, and enter a custom source category beginning with an underscore. Example: (_sourceCategory=MyCategory). 
    3. **Advanced**. Select the **Location in Library** (the default is the Personal folder in the library), or click **New Folder** to add a new folder.
2. Click **Add to Library**.

Once an app is installed, it will appear in your **Personal** folder, or other folder that you specified. From here, you can share it with your organization.

Panels will start to fill automatically. It's important to note that each panel slowly fills with data matching the time range query and received since the panel was created. Results won't immediately be available, but with a bit of time, you'll see full graphs and maps.


### Dashboard filters    
8


**Each dashboard has a set of filters** that you can apply to the entire dashboard, as shown in the following example. Click the funnel icon in the top dashboard menu bar to display a scrollable list of filters that are applied across the entire dashboard.


9
You can use filters to drill down and examine the data on a granular level.


10


**Each panel has a set of filters** that are applied to the results for that panel only, as shown in the following example. Click the funnel icon in the top panel menu bar to display a list of panel-specific filters.


11



### Cisco ASA - Overview Dashboard
12



13


**Outbound Destinations.** A geolocation query tracks the number of outbound connection and displays their destinations on a map of the world. Results are displayed for the last hour. Click **Show in Search** to see more details of the query results.

**Denied Connections by Location.** Uses a geolocation query to track the number of denied connections and displays their destinations on a map of the world. Results are displayed for the last hour. Click **Show in Search **to see more details of the query results.

**Total Denied Sources. **Displays the total number of denied Sources as a single value chart for the last hour.

**Bandwidth Served.** Shows the amount of bandwidth served as a single value chart for the last hour.


### Cisco ASA - Firewall Detail Dashboard
14



15


**Connections Over Time.** Provides information on inbound, outbound, and denied connections over the past hour in increments of five minutes in a stacked area chart.

**Protocol Distribution Over Time.** Displays data on the connection protocols being used, including GRE, UDP, and TCP, over the past hour in increments of five minutes. Displayed in a column chart.

**Bandwidth Served in MBytes.** Shows the bandwidth served in megabytes over the last hour in five-minute increments.

**Most Severe Messages.** Displays the top five most severe message alerts and their count over the last hour.

**Outbound Destinations.** A geolocation query tracks the number of outbound connection and displays their destinations on a map of the world. Results are displayed for the last hour. Click **Show in Search** to see more details of the query results.

**Outbound Connections Over Time.** Provides details on the number of outbound connections by country over the past hour in increments of five minutes.

**Outbound Connections to High Risk Countries.** Displays the number of outbound connections by country to countries considered high risk over the last hour in five-minute increments.

**Denied Connections by Location.** Uses a geolocation query to track the number of denied connections, and displays their destinations on a map of the world. Results are displayed for the last hour. Click **Show in Search **to see more details of the query results.

**Top Denied Sources.** Lists the top five denied sources by IP address over the last hour.

**Top Denying ACLs.** Displays the top ten denying ACL connections over the last hour in a bar chart.


### Cisco ASA - Logs Analytics Dashboard
16



17


**Count of ASA Logs by LogLevel.** Displays the logs by LogLevel and Severity.

**Count by Severity Code. **Displays the logs by Severity Code.

**Parameterized Search.** Log Details with counts.