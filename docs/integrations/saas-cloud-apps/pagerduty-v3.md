---
id: pagerduty-v3
title: Sumo Logic App for PagerDuty V3
sidebar_label: PagerDuty V3
---

The Sumo Logic App for PagerDuty V3 collects incident messages from your PagerDuty account via a webhook, and displays incident data in pre-configured Dashboards that allow you to monitor and analyze the activity of your PagerDuty account and Services. The Sumo Logic App for PagerDuty V3 uses Webhooks V3, to provide enhanced context for alert object models.


#### Event Types

The Sumo Logic App for PagerDuty V3 ingests PagerDuty incident Webhooks V3 messages, that are triggered by events that occur in your PagerDuty account and Services.

For more information on the incident messages supported in Webhooks V3, see the [PagerDuty documentation](https://developer.pagerduty.com/docs/ZG9jOjQ1MTg4ODQ0-overview).


## Collect Logs for PagerDuty V3

This page provides instructions for configuring a Sumo Logic Hosted Collector and HTTP Source to create a PagerDuty Webhook V3, to collect PagerDuty events. Click a link to jump to a topic:

* [Event types](https://help.sumologic.com/07Sumo-Logic-Apps/18SAAS_and_Cloud_Apps/PagerDuty_v3/Collect_Logs_for_PagerDuty_V3#Event_Types)
* [Log example](https://help.sumologic.com/07Sumo-Logic-Apps/18SAAS_and_Cloud_Apps/PagerDuty_v3/Collect_Logs_for_PagerDuty_V3#Log_Examples)
* [Query example](https://help.sumologic.com/07Sumo-Logic-Apps/18SAAS_and_Cloud_Apps/PagerDuty_v3/Collect_Logs_for_PagerDuty_V3#Query_example)
* [Configure a Sumo Logic Collector and Source](https://help.sumologic.com/07Sumo-Logic-Apps/18SAAS_and_Cloud_Apps/PagerDuty_v3/Collect_Logs_for_PagerDuty_V3#Configure_a_Sumo_Logic_Collector_and_Source)
* [Create a PagerDuty V3 Webhook](https://help.sumologic.com/07Sumo-Logic-Apps/18SAAS_and_Cloud_Apps/PagerDuty_v3/Collect_Logs_for_PagerDuty_V3#Create_a_PagerDuty_V3_Webhook)


1.png "image_tooltip")

Our new app install flow is now in Beta. It is only enabled for certain customers while we gather Beta customer feedback. If you can see the Add Integration button, you can follow the in-product instructions in Sumo Logic to set up the app.


#### Event Types

The Sumo Logic App for PagerDuty V3 ingests PagerDuty incident Webhooks V3 messages, caused by events that occur in your PagerDuty account and Services.

For more information on the incident messages supported in Webhooks V3, see the [PagerDuty documentation](https://developer.pagerduty.com/docs/ZG9jOjQ1MTg4ODQ0-overview#event-types).


#### Log example

For examples of incident.triggered, incident.acknowledged and incident.resolved log messages, see the [PagerDuty Webhooks V3 Examples](https://developer.pagerduty.com/docs/ZG9jOjQ1MTg4ODQ0-overview#webhook-payload) page.


#### Query example

The following Top Altering Services query is shown on the PagerDuty V3 - Overview dashboard.


```
_sourceCategory = Labs/pagerduty_v3 "incident.triggered"
| json "event.event_type","event.data","event.data.created_at" as event,incident,created_on nodrop
| json field=incident "id", "number", "escalation_policy.summary", "service.summary", "status", "title", "urgency", "teams[*].summary", "assignees[*]"  as incident_id, incident_number, escalation_policy_name, alertedBy_service, incident_status, incident_title, incident_urgency,  incident_team_involved, assignee nodrop
| parse regex field=assignee "summary\":\"(?<assigned_user>.+?)\"" multi nodrop
| alertedBy_service as impacted_service
| where event = "incident.triggered" and impacted_service matches "*" and incident_number matches "*" and incident_status matches "*" and incident_title matches "*" and incident_urgency matches "*"
| count by alertedBy_service
| order by _count
```

#### Configure a Sumo Logic Collector and Source

A** Hosted Collector **is not installed on a local system in your deployment. Instead, Sumo Logic hosts the Collector and its Sources in AWS. With a Hosted Collector, you can create Sources to collect data from various services. A single Hosted Collector can be configured with any number of  Sources.

An **HTTP Source** is an endpoint for receiving log and metric data uploaded to a unique URL generated for the Source. The URL securely encodes the Collector and Source information. You can add as many HTTP Logs and Metrics Sources as you'd like to a single Hosted Collector.

**To configure Hosted Collector and HTTP Source, do the following:**

1. Log in to Sumo Logic.
2. Follow the instructions for configuring a [Hosted Collector](https://help.sumologic.com/03Send-Data/Hosted-Collectors/Configure-a-Hosted-Collector).
3. Follow the instructions for configuring an [HTTP Source](https://help.sumologic.com/03Send-Data/Sources/02Sources-for-Hosted-Collectors/HTTP-Source).


2.png "image_tooltip")
Make sure to save the **HTTP Source Address URL**. You will be asked for this **Endpoint URL** when you configure the PagerDuty Webhook in the following procedure.


##### Create a PagerDuty V3 Webhook

Using PagerDuty with Webhooks V3, you receive HTTP callbacks when incident events occur in your PagerDuty account. Details about the events are then sent via HTTP to a URL that you specify.

**To create a PagerDuty V3 Webhook, do the following:**

1. Log in to your PagerDuty account.
2. Navigate to **Integrations Generic Webhooks (v3)**.
3. Click **New Webhook**.
4. Configure your webhook:
    1. Enter the **HTTP Source Address URL** as the Webhook URL.
    2. For** Scope Type**, select Service, Team or Account based on your preferences.
    3. For **Scope**, select the desired service or team.
    4. Enter a **Description**.
    5. For **Event Subscription**, select which events you want to send a webhook.
5. Click **Add Webhook**. \


For more information, see [https://support.pagerduty.com/docs/webhooks](https://support.pagerduty.com/docs/webhooks).

Continue with [installing the Sumo Logic App for PagerDuty V3](https://help.sumologic.com/07Sumo-Logic-Apps/18SAAS_and_Cloud_Apps/PagerDuty_v3/Install_the_PagerDuty_V3_App_and_view_the_Dashboards).


## Install the PagerDuty V3 App and view the Dashboards

This page provides instructions for installing the Sumo App for PagerDuty V3, as well as the descriptions of each of the app dashboards. Click a link to jump to a section:

* [Install the app](https://help.sumologic.com/07Sumo-Logic-Apps/18SAAS_and_Cloud_Apps/PagerDuty_v3/Install_the_PagerDuty_V3_App_and_view_the_Dashboards#Install_the_app)
* [Dashboard filters](https://help.sumologic.com/07Sumo-Logic-Apps/18SAAS_and_Cloud_Apps/PagerDuty_v3/Install_the_PagerDuty_V3_App_and_view_the_Dashboards#Dashboard_Filter_with_Template_Variables)
* [Overview Dashboard](https://help.sumologic.com/07Sumo-Logic-Apps/18SAAS_and_Cloud_Apps/PagerDuty_v3/Install_the_PagerDuty_V3_App_and_view_the_Dashboards#Overview_Dashboard)
* [Incidents Overview Dashboard](https://help.sumologic.com/07Sumo-Logic-Apps/18SAAS_and_Cloud_Apps/PagerDuty_v3/Install_the_PagerDuty_V3_App_and_view_the_Dashboards#Incidents_Overview_Dashboard)
* [Incidents by Urgency and Escalation Policy Dashboard](https://help.sumologic.com/07Sumo-Logic-Apps/18SAAS_and_Cloud_Apps/PagerDuty_v3/Install_the_PagerDuty_V3_App_and_view_the_Dashboards#Incidents_by_Urgency_and_Escalation_Policy_Dashboard)


#### Install the app

Now that you have set up a log and metric collection, you can install the Sumo Logic App for PagerDuty V3, and use its pre-configured searches and dashboards.

**To install the app, do the following:**

Locate and install the app you need from the **App Catalog**. If you want to see a preview of the dashboards included with the app before installing, click **Preview Dashboards**.

1. From the **App Catalog**, search for PagerDuty and select the app**.**
2. Select the version of the service you're using and click **Add to Library**.

3.png "image_tooltip")
Version selection is applicable only to a few apps currently. For more information, see the [Install the Apps from the Library.](https://help.sumologic.com/01Start-Here/Library/Apps-in-Sumo-Logic/Install-Apps-from-the-Library)


1. To install the app, complete the following fields.
    1. **App Name.** You can retain the existing name, or enter a name of your choice for the app. 
    2. **Data Source.** Select either of these options for the data source. 
        * Choose **Source Category**, and select a source category from the list. 
        * Choose **Enter a Custom Data Filter**, and enter a custom source category beginning with an underscore. Example: (_sourceCategory=MyCategory)
    3. **Advanced**. Select the **Location in Library** (the default is the Personal folder in the library), or click **New Folder** to add a new folder.
2. Click **Add to Library**.

Once an app is installed, it will appear in your **Personal** folder, or other folder that you specified. From here, you can share it with your organization.

Panels will start to fill automatically. It's important to note that each panel slowly fills with data matching the time range query and received since the panel was created. Results won't immediately be available, but with a bit of time, you'll see full graphs and maps.


#### Dashboard Filter with Template Variables      

Template variables provide dynamic dashboards that rescope data on the fly. As you apply variables to troubleshoot through your dashboard, you can view dynamic changes to the data for a fast resolution to the root cause. For more information, see the [Filter with template variables](https://help.sumologic.com/Visualizations-and-Alerts/Dashboard_(New)/Filter_with_template_variables) help page.


4.png "image_tooltip")
You can use template variables to drill down and examine the data on a granular level.


#### Overview Dashboard

**PagerDuty V3 - Overview Dashboard **provides a high-level view of top alerts, triggered incidents summary, as well as a breakdown of per-user incident assignments and comparisons over a specified time interval.

**Use this dashboard to:**

* Review the services, incidents and policies that are causing the most alerts.
* Analyze detailed information on incidents using the Incident Summary panel.
* Drill down to examine data on a granular level with filters.


5.png "image_tooltip")



#### Incidents Overview Dashboard

**PagerDuty V3 - Incidents Overview Dashboard** provides an at-a-glance analysis of triggered, acknowledged, escalated, assigned, and resolved incidents. This dashboard also provides a high-level view of incident trends and comparisons over specified time intervals.

**Use this dashboard to:**

* Review a high-level view of incident summaries.
* Compare the state of incidents with that of a previous time

6.png "image_tooltip")


#### Incidents by Urgency and Escalation Policy Dashboard

**PagerDuty V3 - Incidents by  Urgency and Escalation Policy Dashboard** provides an overview analysis of urgency events, from low to high, as well as a breakdown of the services impacted by high urgency events.

**Use this dashboard to:**



* Review see weekly incident summaries.
* Analyze incidents by severity and escalation policy.