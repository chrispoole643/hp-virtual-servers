---

copyright:
  years: 2019
lastupdated: "2019-02-01"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:note: .note}
{:important: .important}
{:tip: .tip}
{:pre: .pre}

# Getting started with {{site.data.keyword.cloud_notm}} Hyper Protect Virtual Servers
{: #getting-started-with-hp-virtual-servers}

{{site.data.keyword.cloud}} Hyper Protect Virtual Servers is in the **Experimental** phase and is for tryout and test purpose only. You need to sign-up via the [Hyper Protect info page](https://www.ibm.com/cloud/hyper-protect-services) and request to be added to the entitlement list before you can see this service in the Cloud Experimental catalog. The virtual server that you create with Hyper Protect Virtual Servers will be deleted after 30 days. To prevent data loss, use only test data in the current service. This restriction also applies to using Hyper Protect Virtual Servers with other {{site.data.keyword.cloud_notm}} services.
{:important}

<!-- {{site.data.keyword.cloud_notm}} Hyper Protect Virtual Servers is an {{site.data.keyword.cloud_notm}} service that provides highly secure virtual servers that can run Linux and Docker workloads on demand. It offers a flexible and scalable platform that allows you to quickly and easily provision and manage the virtual server of your choice, which allows for a range of capacity sizes to meet various demands of applications that run in the server.-->

This {{site.data.keyword.cloud_notm}} offering instantiates a base open source Ubuntu Linux image inside each virtual server, which can run a myriad of Linux applications including Docker. This tutorial guides you how to retrieve virtual server information using Hyper Protect Virtual Servers.
{:shortdesc}

## Prerequisite
{: #prerequisite}

Before you start, provision the Hyper Protect Virtual Servers service. For detailed steps, see [Provisioning Hyper Protect Virtual Servers](/docs/services/hp-virtual-servers/provision.html).

## Retrieving virtual server information
{: #retrieve-vs-info}

After you create an instance of Hyper Protect Virtual Servers, you can check the detailed information of your virtual server.

1. Go to the virtual server instance on {{site.data.keyword.cloud_notm}}.
2. Click the **Manage** tab from the left navigation.
3. Click **OPEN** to open the {{site.data.keyword.cloud_notm}} Hyper Protect Virtual Servers dashboard.

You can then find detailed information of your virtual server in the dashboard, including IP address, which you will use to connect to the virtual server.


## Next steps
{: #next-steps}

Now you have created your virtual server on {{site.data.keyword.cloud_notm}}. You can connect to your virtual server with the IP address of the service instance and your corresponding private key.