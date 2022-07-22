---
id: nginx-plus
title: Nginx Plus
sidebar_label: Nginx Plus
description: The Nginx Plus app is an unified logs and metrics app that helps you monitor the availability, performance, health and resource utilization of your Nginx Plus web servers.
---

Nginx Plus is a web server that can be used as a reverse proxy, load balancer, mail proxy, and HTTP cache.

1.png "image_tooltip")
The Sumo Logic App for Nginx Plus supports logs for Nginx Plus, as well as Metrics for Nginx Plus.

The Nginx Plus app is an unified logs and metrics app that helps you monitor the availability, performance, health and resource utilization of your Nginx Plus web servers. Preconfigured dashboards and searches provide insight into server status, location zones, server zones, upstreams, resolvers, visitor locations, visitor access types, traffic patterns, errors, web server operations and access from known malicious sources.


#### Log and Metrics Types
2.gif "image_tooltip")


The Sumo Logic App for Nginx Plus assumes the NCSA extended/combined log file format for Access logs and the default Nginx error log file format for error logs.

All Dashboards (except the Error logs Analysis dashboard) assume the Access log format. The Error logs Analysis Dashboard assumes both Access and Error log formats, so as to correlate information between the two.

For more details on Nginx/NginxPlus logs, see ([https://nginx.org/en/docs/http/ngx_http_log_module.html](https://nginx.org/en/docs/http/ngx_http_log_module.html)).

The Sumo Logic App for Nginx Plus assumes Prometheus format Metrics for Requests and Connections. For Nginx Plus Server metrics, API Module from Nginx Configuration is used.

For more details on Nginx Plus Metrics, see ([https://nginx.org/en/docs/http/ngx_http_api_module.html](https://nginx.org/en/docs/http/ngx_http_api_module.html)).


## Collect Logs and Metrics for Nginx Plus

This page provides instructions for configuring log and metric collection for the Sumo Logic App for Nginx Plus.

#### Collection Process Overview
3.gif "image_tooltip")

Sumo Logic supports a collection of logs and metrics data from Nginx Plus in both Kubernetes and non-Kubernetes environments.

Please click on the appropriate links below based on the environment where your Nginx Plus servers are hosted.

* [Collect Logs and Metrics for Kubernetes environments](https://help.sumologic.com/07Sumo-Logic-Apps/24Web_Servers/Nginx_Plus/Collect_Logs_and_Metrics_for_Nginx_Plus/Collect_Logs_and_Metrics_in_Kubernetes_environment)
* [Collect Logs and Metrics for Non-Kubernetes environments](https://help.sumologic.com/07Sumo-Logic-Apps/24Web_Servers/Nginx_Plus/Collect_Logs_and_Metrics_for_Nginx_Plus/Collect_Logs_and_Metrics_in_Non_Kubernetes_environment)


#### Sample Log Message  
4.gif "image_tooltip")



##### Non Kubernetes environment
5.gif "image_tooltip")


**Access Log Example**


```
50.1.1.1 - example [23/Sep/2016:19:00:00 +0000] "POST /api/is_individual HTTP/1.1" 200 58 "-"
"python-requests/2.7.0 CPython/2.7.6 Linux/3.13.0-36-generic"
```


**Error Log Example**


```
2016/09/23 19:00:00 [error] 1600#1600: *61413 open() "/srv/core/client/dist/client/favicon.ico"
failed (2: No such file or directory), client: 101.1.1.1, server: _, request: "GET /favicon.ico
HTTP/1.1", host: "example.com", referrer: "https://abc.example.com/"
```



##### Kubernetes environment
6.gif "image_tooltip")


**Access Log Example**


```
{"timestamp":1620821977736,"log":"10.244.0.132 - - [12/May/2021:12:19:28 +0000] \"GET //demo-index.html HTTP/1.1\" 200 8777 \"-\" \"curl/7.68.0\"","stream":"stdout","time":"2021-05-12T12:19:28.975861476Z"}
```


**Error Log Example**


```
{"timestamp":1620821977737,"log":"2021/05/12 12:19:36 [error] 7#7: *8192 upstream timed out (110: Connection timed out) while connecting to upstream, health check \"\" of peer 44.240.53.50:12345 in upstream \"stream_backend2\"","stream":"stderr","time":"2021-05-12T12:19:36.344706832Z"}
```



#### Create Field Extraction Rules
7.gif "image_tooltip")


Field Extraction Rules (FERs) tell Sumo Logic which fields to parse out automatically. For instructions, see [Create a Field Extraction Rule](https://help.sumologic.com/Manage/Field-Extractions/Create-a-Field-Extraction-Rule).

Nginx assumes the NCSA extended/combined log file format for Access logs and the default Nginx error log file format for error logs.

Both the parse expressions can be used for logs collected from Nginx Plus Server running on Local or container-based systems.

**FER for Access Logs**

Use the following Parse Expression:


```
| json field=_raw "log" as nginx_log_message nodrop
| if (isEmpty(nginx_log_message), _raw, nginx_log_message) as nginx_log_message
| parse regex field=nginx_log_message "(?<Client_Ip>\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})"
| parse regex field=nginx_log_message "(?<Method>[A-Z]+)\s(?<URL>\S+)\sHTTP/[\d\.]+\
"\s(?<Status_Code>\d+)\s(?<Size>[\d-]+)\s\"(?<Referrer>.*?)\"\s\"(?<User_Agent>.+?)\".*"
```


**FER for Error Logs**

Use the following Parse Expression:


```
| json field=_raw "log" as nginx_log_message nodrop
| if (isEmpty(nginx_log_message), _raw, nginx_log_message) as nginx_log_message
| parse regex field=nginx_log_message "\s\[(?<Log_Level>\S+)\]\s\d+#\d+:\s(?:\*\d+\s|)(?<Message>[A-Za-z][^,]+)(?:,|$)"
| parse field=nginx_log_message "client: *, server: *, request: \"* * HTTP/1.1\", host:
\"*\"" as Client_Ip, Server, Method, URL, Host nodrop
```



#### Query Samples
8.gif "image_tooltip")


This sample Query is from the **Responses Over Time** panel of the **Nginx Plus - Overview** dashboard.


```
_sourcecategory=Labs/Nginx/Logs
| json auto maxdepth 1 nodrop
| if (isEmpty(log), _raw, log) as nginx_log_message
| parse regex field=nginx_log_message "(?<Client_Ip>\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})"
| parse regex field=nginx_log_message "(?<Method>[A-Z]+)\s(?<URL>\S+)\sHTTP/[\d\.]+\"\s(?<Status_Code>\d+)\s(?<Size>[\d-]+)\s\"(?<Referrer>.*?)\"\s\"(?<User_Agent>.+?)\".*"
| where _sourceHost matches "{{Server}}" and Client_Ip matches "{{Client_Ip}}" and Method matches "{{Method}}" and URL matches "{{URL}}" and Status_Code matches "{{Status_Code}}"
| if(Status_Code matches "2*", 1, 0) as Successes
| if(Status_Code matches "3*", 1, 0) as Redirects
| if(status_code matches "4*", 1, 0) as Client_Errors
| if(Status_Code matches "5*", 1, 0) as Server_Errors
| timeslice by 5m
| sum(Successes) as Successes, sum(Client_Errors) as Client_Errors,  sum(Redirects) as Redirects, sum(Server_Errors) as Server_Errors by _timeslice
| sort by _timeslice asc
```



### Collect Logs and Metrics in Kubernetes environment

In a Kubernetes environment, we use the Telegraf Operator, which is packaged with our Kubernetes collection. You can learn more about it [here](https://help.sumologic.com/03Send-Data/Collect-from-Other-Data-Sources/Collect_Metrics_Using_Telegraf/01_Telegraf_Collection_Architecture).The diagram below illustrates how data is collected from Nginx Plus in Kubernetes environments. In the architecture shown below, there are four services that make up the metric collection pipeline: Telegraf, Prometheus, Fluentd and FluentBit.

The first service in the pipeline is Telegraf. Telegraf collects metrics from Nginx Plus. Note that we’re running Telegraf in each pod we want to collect metrics from as a sidecar deployment: i.e. Telegraf runs in the same pod as the containers it monitors. Telegraf uses the Nginx Plus input plugin to obtain metrics. (For simplicity, the diagram doesn’t show the input plugins.) The injection of the Telegraf sidecar container is done by the Telegraf Operator. We also have Fluentbit that collects logs written to standard out and forwards them to FluentD, which in turn sends all the logs and metrics data to a Sumo Logic HTTP Source.


9.png "image_tooltip")


Configuring log and metric collection for the Nginx Plus App includes the following tasks:

* Step 1: Collect Logs for Nginx Plus
* Step 2: Collect Metrics for Nginx Plus


#### Step 1: Collect Logs for Nginx Plus in Kubernetes environment
10.gif "image_tooltip")

11.png "image_tooltip")
Nginx Plus app supports the default access logs and error logs format.

1. Configure logging in Nginx Plus

Before you can configure Sumo Logic to ingest logs, you must configure the logging of errors and processed requests in NGINX Open Source and NGINX Plus. For instructions, refer to the following documentation:

[https://www.nginx.com/resources/admin-guide/logging-and-monitoring/](https://www.nginx.com/resources/admin-guide/logging-and-monitoring/)

2. Use the Sumologic-Kubernetes-Collection, to send the logs to Sumologic. For more information,[ visit](https://help.sumologic.com/Observability_Solution/Kubernetes_Solution/04Set_up_collection_for_Kubernetes).

3. Identifying the logs metadata: For example, to get **Logs** data from the pod, you can use the following source  **_sourcecategory = "kubernetes/default/nginx"** where “kubernetes” is Cluster name, “default” is Namespace, “nginx” is application.

4. To get log data from Nginx Pods - all nginx logs must be redirected to standard output “**stdout**” and standard error “**stderr**”.


#### Step 2: Collect Metrics for Nginx Plus in Kubernetes environment  
12.gif "image_tooltip")


13.png "image_tooltip")
Nginx Plus app supports the metrics for Nginx Plus.

The following steps assume you are collecting Nginx Plus metrics from a Kubernetes environment. In a Kubernetes environment, we use the Telegraf Operator, which is packaged with our Kubernetes collection.  You can learn more about this[ here](https://help.sumologic.com/03Send-Data/Collect-from-Other-Data-Sources/Collect_Metrics_Using_Telegraf/01_Telegraf_Collection_Architecture).


1. Before you can configure Sumo Logic to ingest metrics, you must enable the API module to expose metrics in NGINX Plus.
    * For instructions on Nginx Plus, refer to the following documentation [https://docs.nginx.com/nginx/admin-guide/monitoring/live-activity-monitoring/](https://docs.nginx.com/nginx/admin-guide/monitoring/live-activity-monitoring/).
    * Make a note of the URL where the API is exposed. It will match the format like [https://localhost:8080/api](https://localhost:8080/api).
2. [Set up Kubernetes Collection with the Telegraf Operator.](https://help.sumologic.com/03Send-Data/Collect-from-Other-Data-Sources/Collect_Metrics_Using_Telegraf/03_Install_Telegraf#Install_Telegraf_in_a_Kubernetes_environment)
3. On your Nginx Plus Pods, add the following annotations to configure Telegraf.


```
annotations:
        telegraf.influxdata.com/inputs: |+
        [[inputs.nginx_plus_api]]
           urls = ["http://localhost:8080/api"]
           response_timeout = "5s"
           api_version = 6
        telegraf.influxdata.com/class: sumologic-prometheus
        prometheus.io/scrape: "true"
        prometheus.io/port: "9273"

```



* telegraf.influxdata.com/inputs - This contains the required configuration for the Telegraf Nginx Plus Input plugin. Please refer[ to this doc](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/nginx_plus_api) for more information on configuring the Nginx input plugin for Telegraf. Note since telegraf will be run as a sidecar the host should always be localhost
* telegraf.influxdata.com/class: sumologic-prometheus - This instructs the Telegraf operator what output to use. This should not be changed.
* prometheus.io/scrape: "true" - This ensures our Prometheus will scrape the metrics.
* prometheus.io/port: "9273" - This tells Prometheus what ports to scrape on. This should not be changed.


### Collect Logs and Metrics in Non-Kubernetes environment

We use the Telegraf operator for Nginx Plus metric collection and Sumo Logic Installed Collector for collecting Nginx Plus logs. The diagram below illustrates the components of the Nginx Plus collection in a non-Kubernetes environment. Telegraf runs on the same system as Nginx Plus, and uses the [Nginx Plus input plugin](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/nginx_plus_api) to obtain Nginx Plus metrics, and the Sumo Logic output plugin to send the metrics to Sumo Logic. Logs from Nginx on the other hand are sent to either a Sumo Logic Local File source.


14.png "image_tooltip")


Configuring log and metric collection for the Nginx Plus App includes the following tasks:



* Step 1: Collect Logs for Nginx Plus
1. Configure logging in Nginx Plus
2. Configure a Collector
3. Configure a Source
* Step 2: Collect Metrics for Nginx Plus
1. Configure Metrics in Nginx Plus
2. Configure a Hosted Collector
3. Configure a Metrics Source
4. Install Telegraf
5. Configure Telegraf and Forward Metrics to Sumo Logic


#### Step 1: Collect Logs for Nginx Plus in Non Kubernetes environment
15.gif "image_tooltip")



16.png "image_tooltip")
Nginx Plus app supports the default access logs and error logs format.

This section provides instructions for configuring log collection for the Sumo Logic App for Nginx Plus. Follow the instructions below to set up the Log collection.


    1. Configure logging in Nginx


    Before you can configure Sumo Logic to ingest logs, you must configure the logging of errors and processed requests in NGINX Open Source and NGINX Plus. For instructions, refer to the following documentation:


    [https://www.nginx.com/resources/admin-guide/logging-and-monitoring/](https://www.nginx.com/resources/admin-guide/logging-and-monitoring/)


    2. Configure a Collector


    Use one of the following Sumo Logic Collector options:



1. To collect logs directly from the Nginx Plus machine, configure an[ Installed Collector](https://help.sumologic.com/03Send-Data/Installed-Collectors).
2. If you are using a service like Fluentd, or you would like to upload your logs manually, [Create a Hosted Collector](https://help.sumologic.com/03Send-Data/Hosted-Collectors#Create_a_Hosted_Collector).

    3. Configure a Source


    **For an Installed Collector**


    To collect logs directly from your Nginx Plus machine, use an Installed Collector and a Local File Source.

1. Add a[ Local File Source](https://help.sumologic.com/03Send-Data/Sources/01Sources-for-Installed-Collectors/Local-File-Source).
2. Configure the Local File Source fields as follows:
    * **Name.** (Required)
    * **Description.** (Optional)
    * **File Path (Required).** Enter the path to your error.log or access.log. The files are typically located in /var/log/nginx/*.log. If you are using a customized path, check the nginx.conf file for this information. If you are using Passenger, you may have instructed Passenger to log to a specific log using the passenger_log_file option.
    * **Source Host.** Sumo Logic uses the hostname assigned by the OS unless you enter a different hostname.
    * **Source Category.** Enter any string to tag the output collected from this Source, such as **Nginx/Access** or **Nginx/Error**. (The Source Category metadata field is a fundamental building block to organize and label Sources. For details see[ Best Practices](https://help.sumologic.com/03Send-Data/01-Design-Your-Deployment/Best-Practices%3A-Good-Source-Category%2C-Bad-Source-Category).)
3. Configure the **Advanced** section:
    * **Enable Timestamp Parsing.** Select Extract timestamp information from log file entries.
    * **Time Zone.** Automatically detect.
    * **Timestamp Format.** The timestamp format is automatically detected.
    * **Encoding. **Select** **UTF-8 (Default).
    * **Enable Multiline Processing.**
        * **Error** **logs. **Select **Detect messages spanning multiple lines** and **Infer Boundaries - Detect message boundaries automatically**.
        * **Access** **logs. **These are single-line logs, uncheck **Detect messages spanning multiple lines**.
4. Click **Save**.

    **For a Hosted Collector**


    If you are using a service like Fluentd, or you would like to upload your logs manually, use a Hosted Collector and an HTTP Source.

1. Add an[ HTTP Source](https://help.sumologic.com/03Send-Data/Sources/02Sources-for-Hosted-Collectors/HTTP-Source).
2. Configure the HTTP Source fields as follows:
    * **Name.** (Required)
    * **Description.** (Optional)
    * **Source Host.** Sumo Logic uses the hostname assigned by the OS unless you enter a different hostname.
    * **Source Category.** Enter any string to tag the output collected from this Source, such as **Nginx/Access** or **Nginx/Error**. (The Source Category metadata field is a fundamental building block to organize and label Sources. For details see[ Best Practices](https://help.sumologic.com/03Send-Data/01-Design-Your-Deployment/Best-Practices%3A-Good-Source-Category%2C-Bad-Source-Category).)
3. Configure the **Advanced** section:
    * **Enable Timestamp Parsing.** Select **Extract timestamp information from log file entries**.
    * **Time Zone.** For Access logs, use the time zone from the log file. For Error logs, make sure to select the correct time zone.
    * **Timestamp Format.** The timestamp format is automatically detected.
    * **Enable Multiline Processing.**
        * **Error** **logs**: Select **Detect messages spanning multiple lines** and **Infer Boundaries - Detect message boundaries automatically**.
        * **Access** **logs**: These are single-line logs, uncheck **Detect messages spanning multiple lines**.
4. Click **Save**.
5. When the URL associated with the HTTP Source is displayed, copy the URL so you can add it to the service you are using, such as Fluentd.


#### Step 2: Collect Metrics for Nginx Plus in Non Kubernetes environment
17.gif "image_tooltip")



18.png "image_tooltip")
Nginx Plus app supports the metrics for Nginx Plus.

This section provides instructions for configuring metrics collection for the Sumo Logic App for Nginx Plus. Follow the below instructions to set up the metric collection.


    1. Configure Metrics in Nginx Plus


    Before you can configure Sumo Logic to ingest metrics, you must enable API module to expose metrics in NGINX Plus.



* The live activity monitoring data is generated by the [NGINX Plus API](https://nginx.org/en/docs/http/ngx_http_api_module.html). Visit [live activity monitoring](https://www.nginx.com/products/nginx/live-activity-monitoring/) to configure the API module.
* Make a note of the URL where the API is exposed. It will match the format like[ http://localhost/api](https://127.0.0.1:8080/nginx_status).

    2. Configure a Hosted Collector


    To create a new Sumo Logic hosted collector, perform the steps in the[ Create a Hosted Collector](https://help.sumologic.com/03Send-Data/Hosted-Collectors/Configure-a-Hosted-Collector) section of the Sumo Logic documentation.


    3. Configure a Metrics Source


    Create a new HTTP Logs and Metrics Source in the hosted collector created above by following[ these instructions. ](https://help.sumologic.com/03Send-Data/Sources/02Sources-for-Hosted-Collectors/HTTP-Source)


    Make a note of the HTTP** Source URL**.


    4. Install Telegraf


    Use the[ following steps](https://help.sumologic.com/03Send-Data/Collect-from-Other-Data-Sources/Collect_Metrics_Using_Telegraf/03_Install_Telegraf#install-telegraf-in-a-non-kubernetes-environment) to install Telegraf.


    5. Configure and start Telegraf


    Create a file called telegraf.conf and add the appropriate configuration. The following is a basic example:



```
[agent]
  interval = "60s"
# Read Nginx Plus full API information (ngx_http_api_module)
[[inputs.nginx_plus_api]]
 # An array of Nginx Plus API URLs to gather stats.
 urls = ["http://localhost/api"]
 # HTTP response timeout (default: 5s)
 response_timeout = "5s"
  # Nginx Plus API version, default: 3
 api_version = 6
[[outputs.sumologic]]
  url = "<URL Created in Step 3>"
  data_format = "prometheus"

```



* interval - This is the frequency to send data to Sumo Logic, in this example, we will send the metrics every 60 seconds. Please refer to[ this doc](https://help.sumologic.com/03Send-Data/Collect-from-Other-Data-Sources/Collect_Metrics_Using_Telegraf/03_Install_Telegraf#Configuring_Telegraf) for more properties that can be configured in the Telegraf agent globally.
* urls - The url to the Nginx Plus server with the API enabled. This can be a comma-separated list to connect to multiple Nginx Plus servers. Please refer[ to this doc](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/nginx_plus_api) for more information on configuring the Nginx API input plugin for Telegraf.
* url - This is the HTTP source URL created in step 3. Please refer[ to this doc](https://help.sumologic.com/03Send-Data/Collect-from-Other-Data-Sources/Collect_Metrics_Using_Telegraf/05_Configure_Telegraf_Output_Plugin_for_Sumo_Logic) for more information on configuring the Sumo Logic Telegraf output plugin.
* data_format = The format to use when sending data to Sumo Logic. Please refer[ to this doc](https://help.sumologic.com/03Send-Data/Collect-from-Other-Data-Sources/Collect_Metrics_Using_Telegraf/05_Configure_Telegraf_Output_Plugin_for_Sumo_Logic) for more information on configuring the Sumo Logic Telegraf output plugin.

    Once you have finalized your telegraf.conf file, you can run the following command to start telegraf.


    telegraf --config /path/to/telegraf.conf



## Install the Nginx Plus App and View the Dashboards

This page has instructions for installing the Sumo Logic Monitors for Nginx Plus,  also the app and descriptions of each of the app dashboards. These instructions assume you have already set up collection as described in the [Collect Logs and Metrics for Nginx Plus](https://help.sumologic.com/07Sumo-Logic-Apps/24Web_Servers/Nginx_Plus/Collect_Logs_and_Metrics_for_Nginx_Plus) App page.


#### Installing Monitors
19.gif "image_tooltip")


To install these alerts, you need to have the [Manage Monitors](https://help.sumologic.com/Manage/Users-and-Roles/Manage-Roles/05-Role-Capabilities#Monitors_(New)) role capability.

Alerts can be installed by either importing them via a JSON or via a Terraform script.


##### Method 1: Install the alerts by importing a JSON file:
20.gif "image_tooltip")




1. Download the [JSON file](https://github.com/SumoLogic/terraform-sumologic-sumo-logic-monitor/blob/main/monitor_packages/nginx-plus/nginxplus.json) describing all the monitors.
2. Replace **$$logs_data_source** and** $$metric_data_source** with logs and metrics data sources respectively.
    1. For example, `_sourceCategory=Labs/Nginx/Plus/Logs`
3. Go to Manage Data > Alerts > Monitors.
4. Click **Add**:


21.png "image_tooltip")




1. Click **Import** to import monitors from the JSON above.

Note: The monitors are disabled by default. Once you have installed the alerts via this method, navigate to the **Nginx Plus** folder under **Monitors** to configure them. Refer [document](https://help.sumologic.com/Visualizations-and-Alerts/Alerts/Monitors#Add_a_monitor) to enable monitors, to configure each monitor, to send notifications to teams or connections.


##### Method 2: Install the alerts via a Terraform script
22.gif "image_tooltip")


**Step 1: Generate a Sumo Logic access key and ID**

Generate an [access key](https://help.sumologic.com/Manage/Security/Access-Keys#create-an-access-key%C2%A0on-preferences-page) and access ID for a user that has the [Manage Monitors](https://help.sumologic.com/Manage/Users-and-Roles/Manage-Roles/05-Role-Capabilities#Monitors_(New)) role capability in Sumo Logic using these instructions. Please identify your Sumo Logic [deployment](https://help.sumologic.com/APIs/General-API-Information/Sumo-Logic-Endpoints-by-Deployment-and-Firewall-Security).

**Step 2: **[Download and install Terraform 0.13](https://www.terraform.io/downloads.html) or later** **

**Step 3: Download the Sumo Logic Terraform package for Nginx Plus alerts**

The alerts package is available in the Sumo Logic github [repository](https://github.com/SumoLogic/terraform-sumologic-sumo-logic-monitor/tree/main/monitor_packages/nginx-plus). You can either download it via the “git clone” command or as a zip file.

**Step 4: Alert Configuration**

After the package has been extracted, navigate to the package directory **terraform-sumologic-sumo-logic-monitor/monitor_packages/nginx-plus/**

Edit the **nginxplus.auto.tfvars** file as per below instruction


1. Add the Sumo Logic Access Key, Access Id, Deployment from Step 1.
    1. **access_id**   = `"<YOUR SUMO ACCESS ID>"`
    2. **access_key**  = `"<YOUR SUMO ACCESS KEY>"`
    3. **environment** = `"<DEPLOYMENT>"`
2. Add the data source values.
    4. **Metric_data_source** - Sumo Logic data source for nginx plus metrics.
    5. **Logs_data_source** - Sumo Logic data source for logs.
3. All monitors are disabled by default on installation, if you would like to enable all the monitors, set the parameter **monitors_disabled** to **false**.
4. All monitors are configured in a monitor folder called “**Nginx Plus**”, if you would like to change the name of the folder, update the parameter **folder**.

**Step 5: Email and Connection Notification Configuration Examples**

Modify the file **nginxplus.auto.tfvars** and populate connection_notifications and email_notifications as per below examples.


###### Pagerduty Connection Example:
23.gif "image_tooltip")



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


Replace `<CONNECTION_ID>` with the connection id of the webhook connection. The webhook connection id can be retrieved via calling the [Monitors API](https://api.sumologic.com/docs/#operation/listConnections).

For overriding payload for different connection types, refer to this [document](https://help.sumologic.com/Manage/Connections-and-Integrations/Webhook-Connections/Set_Up_Webhook_Connections).


###### Email Notifications Example:
24.gif "image_tooltip")



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
}
```


**Step 6: Install the Alerts**

1. Navigate to the package directory **terraform-sumologic-sumo-logic-monitor/monitor_packages/nginx-plus/** and run **terraform init. **This will initialize Terraform and will download the required components.
2. Run **terraform plan **to view the monitors resources which will be created/modified by Terraform.
3. Run **terraform apply**.

**Step 7: Post Installation steps**

If you haven’t enabled alerts and/or configured notifications via the terraform procedure outlined above, we highly recommend enabling alerts of interest and configuring each enabled alert to send notifications to other people or services. This is detailed in [Step 4](https://help.sumologic.com/Visualizations-and-Alerts/Alerts/Monitors#Add_a_monitor).

Note: There are limits to how many alerts can be enabled - please see the Alerts FAQ (insert link)


#### Install the App
25.gif "image_tooltip")


**To install the app:**

Locate and install the app you need from the **App Catalog**. If you want to see a preview of the dashboards included with the app before installing, click **Preview Dashboards**.



1. From the **App Catalog**, search for and select the app**.**
2. Select the version of the service you're using and click **Add to Library**.


26.png "image_tooltip")
Version selection is applicable only to a few apps currently. For more information, see the[ Install the Apps from the Library](https://help.sumologic.com/01Start-Here/Library/Apps-in-Sumo-Logic/Install-Apps-from-the-Library).



1. To install the app, complete the following fields.
    1. **App Name.** You can retain the existing name, or enter a name of your choice for the app. 
    2. **Data Source.** Select either of these options for the data source. 
        * Choose **Source Category**, and select a source category from the list. 
        * Choose **Enter a Custom Data Filter**, and enter a custom source category beginning with an underscore. Example: (_sourceCategory=MyCategory). 
    3. **Advanced**. Select the **Location in Library** (the default is the Personal folder in the library), or click **New Folder** to add a new folder.
2. Click **Add to Library**.

Once an app is installed, it will appear in your **Personal** folder, or other folder that you specified. From here, you can share it with your organization.

Panels will start to fill automatically. It's important to note that each panel slowly fills with data matching the time range query and received since the panel was created. Results won't immediately be available, but with a bit of time, you'll see full graphs and maps.


#### Dashboard Filter with Template Variables
27.gif "image_tooltip")


Template variables provide dynamic dashboards that rescope data on the fly. As you apply variables to troubleshoot through your dashboard, you can view dynamic changes to the data for a fast resolution to the root cause. For more information, see the[ Filter with template variables](https://help.sumologic.com/Visualizations-and-Alerts/Dashboard_(New)/Filter_with_template_variables) help page.


28.png "image_tooltip")
You can use template variables to drill down and examine the data on a granular level.


#### Dashboards
29.gif "image_tooltip")



##### Nginx Plus - Overview
30.gif "image_tooltip")


The** Nginx Plus - Overview** dashboard provides an at-a-glance view of the Nginx Plus server access locations, error logs along with connection metrics.

**Use this dashboard to:**



* Gain insights into originated traffic location by region. This can help you allocate computer resources to different regions according to their needs.
* Gain insights into your Nginx health using Critical Errors and Status of Nginx Server.
* Get insights into Active and dropped connections.


31.png "image_tooltip")



##### Nginx Plus - Error Logs Analysis
32.gif "image_tooltip")


The **Nginx Plus - Error Logs Analysis** dashboard provides a high-level view of log level breakdowns, comparisons, and trends. The panels also show the geographic locations of clients and clients with critical messages, new connections and outliers, client requests, request trends, and request outliers.

**Use this dashboard to:**



* Track requests from clients. A request is a message asking for a resource, such as a page or an image.
* To track and view client geographic locations generating errors.
* Track critical alerts and emergency error alerts.


33.png "image_tooltip")



##### Nginx Plus - Logs Timeline Analysis
34.gif "image_tooltip")


The **Nginx Plus - Logs Timeline Analysis** dashboard provides a high-level view of the activity and health of Nginx servers on your network. Dashboard panels display visual graphs and detailed information on traffic volume and distribution, responses over time, as well as time comparisons for visitor locations and server hits.

**Use this dashboard to:**



* To understand the traffic distribution across servers, provide insights for resource planning by analyzing data volume and bytes served.
* Gain insights into originated traffic location by region. This can help you allocate compute resources to different regions according to their needs.


35.png "image_tooltip")



##### Nginx Plus - Outlier Analysis
36.gif "image_tooltip")


The **Nginx Plus -  Outlier Analysis** dashboard provides a high-level view of Nginx server outlier metrics for bytes served, number of visitors, and server errors. You can select the time interval over which outliers are aggregated, then hover the cursor over the graph to display detailed information for that point in time.

**Use this dashboard to:**



* Detects outliers in your infrastructure with Sumo Logic’s machine learning algorithm.
* To identify outliers in incoming traffic and the number of errors encountered by your servers.


37.png "image_tooltip")
You can use schedule searches to send alerts to yourself whenever there is an outlier detected by Sumo Logic.


38.png "image_tooltip")



##### Nginx Plus - Threat Intel
39.gif "image_tooltip")


The **Nginx Plus - Threat Inte**l dashboard provides an at-a-glance view of threats to Nginx servers on your network. Dashboard panels display the threat count over a selected time period, geographic locations where threats occurred, source breakdown, actors responsible for threats, severity, and a correlation of IP addresses, method, and status code of threats.

**Use this dashboard to:**



* To gain insights and understand threats in incoming traffic and discover potential IOCs. Incoming traffic requests are analyzed using the[ Sumo - Crowdstrikes](https://help.sumologic.com/07Sumo-Logic-Apps/22Security_and_Threat_Detection/Threat_Intel_Quick_Analysis/03_Threat-Intel-FAQ) threat feed.


40.png "image_tooltip")



##### Nginx Plus - Web Server Operations
41.gif "image_tooltip")


The **Nginx Plus - Web Server Operations** dashboard provides a high-level view combined with detailed information on the top ten bots, geographic locations, and data for clients with high error rates, server errors over time, and non 200 response code status codes. Dashboard panels also show information on server error logs, error log levels, error responses by a server, and the top URIs responsible for 404 responses.

**Use this dashboard to:**



* Gain insights into Client, Server Responses on Nginx Server. This helps you identify errors in Nginx Server.
* To identify geo locations of all Client errors. This helps you identify client location causing errors and helps you to block client IPs.


42.png "image_tooltip")



##### Nginx Plus - Visitor Access Types
43.gif "image_tooltip")


The **Nginx Plus - Visitor Access Types** dashboard provides insights into visitor platform types, browsers, and operating systems, as well as the most popular mobile devices, PC and Mac versions used.

**Use this dashboard to:**



* Understand which platform and browsers are used to gain access to your infrastructure. \
These insights can be useful for planning in which browsers, platforms, and operating systems (OS) should be supported by different software services.


44.png "image_tooltip")



##### Nginx Plus - Visitor Locations
45.gif "image_tooltip")


The **Nginx Plus - Visitor Locations** dashboard provides a high-level view of Nginx visitor geographic locations both worldwide and in the United States. Dashboard panels also show graphic trends for visits by country over time and visits by  US region over time.

**Use this dashboard to:**



* Gain insights into geographic locations of your user base.  This is useful for resource planning in different regions across the globe.


46.png "image_tooltip")



##### Nginx Plus - Visitor Traffic Insight
47.gif "image_tooltip")


The **Nginx Plus - Visitor Traffic Insight** dashboard provides detailed information on the top documents accessed, top referrers, top search terms from popular search engines, and the media types served.

**Use this dashboard to:**

* To understand the type of content that is frequently requested by users.
* It helps in allocating IT resources according to the content types.

48.png "image_tooltip")



##### Nginx Plus - Caches
49.gif "image_tooltip")


The Nginx Plus - Caches metrics dashboard provides insight into cache states, cache hit rate and cache disk usage over time.

**Use this dashboard to:**



* Gain information about the number of caches used, how many of them are in active (hot) state and what is the hit rate of the cache.
* Gain information about how much disk space is used for cache.


50.png "image_tooltip")



##### Nginx Plus - HTTP Location Zones
51.gif "image_tooltip")


The Nginx Plus - HTTP Location Zones metrics dashboard provides detailed statistics on the frontend performance, showing traffic speed, responses/requests count and various error responses.

**Use this dashboard to:**

* Gain information about Location http zones traffic: received and sent; speed, requires/responses amount, discarded traffic.
* Gain information about Location http zones error responses: percentage of responses by server, percentage of each type of error responses.


52.png "image_tooltip")



##### Nginx Plus - HTTP Server Zones
53.gif "image_tooltip")


The Nginx Plus - HTTP Server Zones metrics dashboard provides detailed statistics on the frontend performance, showing traffic speed, responses/requests count and various error responses.

**Use this dashboard to:**

* Gain information about Server http zones traffic: received and sent; speed, requires/responses amount, discarded traffic.
* Gain information about Server http zones error responses: percentage of responses by server, percentage of each type of error responses.


54.png "image_tooltip")



##### Nginx Plus - HTTP Upstreams
55.gif "image_tooltip")


The Nginx Plus - HTTP Upstreams metrics dashboard provides information about each upstream group for HTTP and HTTPS traffic, showing number of HTTP upstreams, servers, back-up servers, error responses and health monitoring.

**Use this dashboard to:**

* Gain information about HTTP upstreams, servers and back-up servers.
* Gain information about HTTP upstreams traffic: received and sent; speed, requires/responses amount, downtime and response time.
* Gain information about HTTP upstreams error responses: percentage of responses by server, percentage of each type of error responses.
* Gain information about HTTP upstreams health monitoring.


56.png "image_tooltip")



##### Nginx Plus - Resolvers
57.gif "image_tooltip")


The Nginx Plus - Resolvers metrics dashboard provides DNS server statistics of requests and responses per each DNS status zone.

**Use this dashboard to:**

* Gain information about the total number of zones, responses and requests speed.
* Gain information about error responses by each type of error.


58.png "image_tooltip")



##### Nginx Plus - TCP/UDP Upstreams
59.gif "image_tooltip")


The Nginx Plus - TCP/UDP Upstreams metrics dashboard provides information about each upstream group for TCP and UDP traffic, showing number of TCP and UDP upstreams, servers, back-up servers, error responses and health monitoring.

**Use this dashboard to:**

* Gain information about TCP and UDP upstreams, servers and back-up servers.
* Gain information about TCP and UDP upstreams traffic: received and sent; speed, requests/responses amount, downtime and response time.
* Gain information about TCP and UDP upstreams error responses: percentage of responses by server, percentage of each type of error responses.
* Gain information about TCP and UDP upstreams health monitoring.

60.png "image_tooltip")



##### Nginx Plus - TCP/UDP Zones
61.gif "image_tooltip")


The Nginx Plus - TCP/UDP Zones metrics dashboard provides TCP and UDP status zones with charts for connection limiting.

**Use this dashboard to:**



* Gain information about TCP and UDP traffic: received and sent; speed, requires/responses amount, discarded traffic.
* Gain information about TCP and UDP error responses: percentage of responses by server, percentage of each type of error responses.
*


## Nginx Plus Alerts

Sumo Logic has provided out of the box alerts available via[ Sumo Logic monitors](https://help.sumologic.com/Visualizations-and-Alerts/Alerts/Monitors) to help you quickly determine if the Nginx Plus server is available and performing as expected. These alerts are built based on logs and metrics datasets and have preset thresholds based on industry best practices and recommendations.

**Sumo Logic provides the following out-of-the-box alerts**:


<table>
  <tr>
   <td><strong>Name</strong>
   </td>
   <td><strong>Description</strong>
   </td>
   <td><strong>Alert Condition</strong>
   </td>
   <td><strong>Recover Condition</strong>
   </td>
  </tr>
  <tr>
   <td>Nginx Plus - Dropped Connections
   </td>
   <td>This alert fires when we detect dropped connections for a given Nginx Plus server.
   </td>
   <td> &#62; 0
   </td>
   <td> &#60; &#61; 0
   </td>
  </tr>
  <tr>
   <td>Nginx Plus - Critical Error Messages
   </td>
   <td>This alert fires when we detect critical error messages for a given Nginx Plus server.
   </td>
   <td> &#62; 0
   </td>
   <td> &#60; &#61; 0
   </td>
  </tr>
  <tr>
   <td>Nginx Plus - Access from Highly Malicious Sources
   </td>
   <td>This alert fires when an Nginx Plus is accessed from highly malicious IP addresses.
   </td>
   <td> &#62; 0
   </td>
   <td> &#60; &#61; 0
   </td>
  </tr>
  <tr>
   <td>Nginx Plus - High Client (HTTP 4xx) Error Rate
   </td>
   <td>This alert fires when there are too many HTTP requests (>5%) with a response status of 4xx.
   </td>
   <td> &#62; 0
   </td>
   <td> &#60; &#61; 0
   </td>
  </tr>
  <tr>
   <td>Nginx Plus - High Server (HTTP 5xx) Error Rate
   </td>
   <td>This alert fires when there are too many HTTP requests (>5%) with a response status of 5xx.
   </td>
   <td> &#62; 0
   </td>
   <td> &#60; &#61; 0
   </td>
  </tr>
</table>