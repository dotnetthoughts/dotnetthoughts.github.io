---
layout: post
title: "Azure App Service - Enable the Health Check"
subtitle: "This article shows you what is App Service Health Check feature and how to enable it."
date: 2021-02-07 00:00:00
categories: [Azure,AppService]
tags: [Azure,AppService]
author: "Anuraj"
image: /assets/images/2021/02/healthcheck_in_appservice.png
---
This article shows you what is App Service Health Check feature and how to enable it. This feature will help you to improve the availability of your Azure App Service. You can increase the availability and throughput by scaling the app into multiple instances. But what will happen due to some exceptions one of your app becomes faulty and not responding? This feature helps you to configure an endpoint, in which system will ping on configured intervals, if an app service instance fails to respond to this ping, system remove the instance from your load balancer. This feature introduced in Azure App service in 2019 and now it is Generally Available and ready for production applications.

To enable health check, you can login into Azure Portal, Select the App Service and select the Health Check under Monitoring.

![Enable Health Check in Azure App Service]({{ site.url }}/assets/images/2021/02/healthcheck_in_appservice.png)

Once you specify the path, app service will start ping that URL in the configured intervals. If the URL responds with an error http status code or does not respond, then the instance is identified as unhealthy and it is removed from the load balancer rotation. And load balancer will no longer send the traffic to that app service. System will continuously ping the app service endpoint and if it becomes healthy, it will be added back to the load balancer and if it continues to respond error status code or not responding, App Service will restart the underlying virtual machine to bring back instance to a healthy state.

Once you configure your app service health check, you can monitor the health of your app service using Azure Monitor. From the Health check blade in the Portal, click Metrics. This will open a new blade where you can see the app service health status history and create a new alert rule.

![Azure Alert configuration]({{ site.url }}/assets/images/2021/02/configure_azure_monitor.png)

If you're using asp.net core application, you can configure Health Checks feature in ASP.NET Core - I wrote a [blog post](https://dotnetthoughts.net/implementing-health-check-aspnetcore/) about the Health check implementation. Check out it [here](https://dotnetthoughts.net/implementing-health-check-aspnetcore/). Because it is recommended that health endpoint is checking for the all the critical elements - and if any critical element is not responding the health endpoint should return an error status code. 

Here is some reference links which discuss about this feature in detail.

1. [Health Check is now Generally Available](https://azure.github.io/AppService/2020/08/24/healthcheck-on-app-service.html?WT.mc_id=AZ-MVP-5002040)
2. [Monitor App Service instances using Health check](https://docs.microsoft.com/en-us/azure/app-service/monitor-instances-health-check?WT.mc_id=AZ-MVP-5002040)

Happy Programming :)