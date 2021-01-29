---
layout: post
title: "A/B Testing with App Service"
subtitle: "A/B Testing feature helps you to test new website content, processes, workflows, etc. by routing the traffic into multiple slots. At a very high level, you route your users into different two deployments of code and measure the success of each version of the site based on your requirements. Azure App Service helps us to set up it very quickly with the help of Deployment Slots."
date: 2021-01-29 00:00:00
categories: [Azure,AppService,A/B Testing]
tags: [Azure,AppService,A/B Testing]
author: "Anuraj"
image: /assets/images/2021/01/appservice_abtesting.png
---
A/B Testing feature helps you to test new website content, processes, workflows, etc. by routing the traffic into multiple slots. At a very high level, you route your users into different two deployments of code and measure the success of each version of the site based on your requirements. Azure App Service helps us to set up it very quickly with the help of Deployment Slots.

To use this feature, you need to configure Deployment Slots for Azure App Service. Deployment slot feature is only available on Standard and Premium pricing SKU. If you're not running App Service on Standard or Premium SKU, you will see something like this.

![Standard and Premium pricing SKU]({{ site.url }}/assets/images/2021/01/appservice_abtesting.png)

You can click on the Upgrade button and change the SKU to Standard or Premium. For this blog post I am upgrading to Standard SKU.

![App Service Spec Picker]({{ site.url }}/assets/images/2021/01/appservice_specpicker.png)

It will take few seconds to upgrade SKU and once it is done, Deployment slot option will be enabled. Once enabled, the default slot is marked as production.

![App Service Deployment Slots enabled]({{ site.url }}/assets/images/2021/01/appservice_abtesting_slots.png)

You can add slot by clicking on Add Slot button. I am adding a staging slot. Since I don't have a configuration, I am selecting `Do not clone Settings` option, if you have any configuration, like App Settings or Connection Strings, you can clone from other slots. The slot is also an Azure app service. Most of the App service features are available for App Service slots as well.

Next I am creating an ASP.NET Core MVC app for deploying to this App Service slots. While creating the app service I choose the Runtime as .NET Core 3.1, you can run the following command to create an .NET Core 3.1 app - `dotnet new mvc --framework netcoreapp3.1 -o abtestingdemo`. Next I am deploying the app in Production Slot. I am using Visual Studio Code to deploy the app to the slot. 

![VS Code App Service Deployment]({{ site.url }}/assets/images/2021/01/vscode_app_service.png)

Once successfully deployed you can browse the application and verify whether it is working or not. I am using Swap slot to deploy the  Next, you need to modify code and deploy it to Staging slot. I am modified the Index view in the application like this and I am deploying it to staging slot.

{% highlight HTML %}
<div class="text-center">
    <h1 class="display-4">Welcome to v2 of the App Service</h1>
    <p>Learn about <a href="https://docs.microsoft.com/aspnet/core">building Web apps with ASP.NET Core</a>.</p>
    <button class="btn btn-primary">Send Feedback</button>
</div>
{% endhighlight %}

To deploy to a slot from VSCode, you need to right click on the Slot and choose the `Deploy to Slot` command.

![VS Code Deployment to slot]({{ site.url }}/assets/images/2021/01/vscode_deploy_slot.png)

Once the deployment is completed, you can open the Deployment slots option in Azure App Service and configure `Traffic %` to 30 to `Staging` slot. This will make sure 30% of traffic is redirected to staging slot and rest 70% to production slot.

![Traffic percentage configuration to deployment slots]({{ site.url }}/assets/images/2021/01/slots_traffic_percentage.png)

Even though the app service URL will remain the same, the traffic will be redirected to different slots. You can identify the slot with a cookie named - `x-ms-routing-name`. For staging slot it will be `staging`. And for production it will be `self`.

![Slots Cookies]({{ site.url }}/assets/images/2021/01/cookie_slots.png)

You can test this behavior by browsing the app and refreshing the browser. To verify it I wrote a C# console application which is hitting the server and reads the cookie and identify whether it is production slot or staging slot.

{% highlight CSharp %}
class Program
{
    static int productionHit = 0;
    static int stagingHit = 0;
    static void Main(string[] args)
    {
        var numberOfIterations = 10;
        for (int i = 0; i < numberOfIterations; i++)
        {
            HitUrl();
        }
        Console.WriteLine(
            string.Format("Number of iterations:{0} - Production Slot:{1} - Staging Slot:{2}", 
            numberOfIterations, productionHit, stagingHit));
    }

    static void HitUrl()
    {
        var appUrl = "https://abtestingdemo.azurewebsites.net/";
        var cookies = new CookieContainer();
        var handler = new HttpClientHandler();
        handler.CookieContainer = cookies;

        var client = new HttpClient(handler);
        var response = client.GetAsync(appUrl).Result;

        Uri uri = new Uri(appUrl);
        var responseCookies = cookies.GetCookies(uri).Cast<Cookie>();
        foreach (Cookie cookie in responseCookies)
        {
            if (cookie.Name == "x-ms-routing-name")
            {
                if (cookie.Value == "self")
                {
                    productionHit++;
                }
                else
                {
                    stagingHit++;
                }
            }
        }
    }
}
{% endhighlight %}

If you run this app, you will be able to see Production Slot 7 and Staging slot 3. Instead of automatically driving users to staging slot, you can implement a Beta link, so that user can opt in to Beta or staging version. To do this you need to create a link with the query string `x-ms-routing-name` and slot name as value. For staging slot or beta you can create something like `?x-ms-routing-name=staging`, if user clicks on this, will be redirected to staging slot. You can identify the slot by configuring an environment variable and selecting the slot setting option. And once the user switch to a slot, the app slot will be served in future. You can integrate Azure Application Insights in to the app service and identify the how users are using the features. Azure App service team posted a series on A/B Testing with App Service. I am including the links for your reference. You can configure Azure DevOps build pipeline to deploy artifacts to staging slot and once you completed the verification and you can use the swap option in deployment slots to swap between staging and production slots.

### Reference Links

1. [Part 1: Client-side configuration](https://azure.github.io/AppService/2020/08/03/ab_testing_app_service.html?WT.mc_id=AZ-MVP-5002040)
2. [Part 2: Server-side configuration](https://azure.github.io/AppService/2020/08/24/ab_testing_app_service2.html?WT.mc_id=AZ-MVP-5002040)
3. [Part 3: Analyzing the telemetry](https://azure.github.io/AppService/2020/08/24/ab_testing_app_service2.html?WT.mc_id=AZ-MVP-5002040)
4. [Set up staging environments in Azure App Service](https://docs.microsoft.com/en-us/azure/app-service/deploy-staging-slots?WT.mc_id=AZ-MVP-5002040)

Happy Programming :)