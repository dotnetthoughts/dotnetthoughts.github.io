---
layout: post
title: "Getting started with Microsoft YARP"
subtitle: "This article discuss about YARP - A Reverse Proxy. YARP is a library to help create reverse proxy servers that are high-performance, production-ready, and highly customizable."
date: 2021-02-20 00:00:00
categories: [AspNetCore,YARP,ReverseProxy]
tags: [AspNetCore,YARP,ReverseProxy]
author: "Anuraj"
image: /assets/images/2021/02/reverse_proxy.png
---
This article discuss about YARP - A Reverse Proxy. YARP is a library to help create reverse proxy servers that are high-performance, production-ready, and highly customizable. So what is a reverse proxy? A reverse proxy is an intermediate connection point placed at a network's edge. It receives initial HTTP connection requests and behave like the actual endpoint based on the configuration. Reverse Proxy acts as gateway between your application and users.

![Reverse Proxy]({{ site.url }}/assets/images/2021/02/reverse_proxy.png)

YARP is built on .NET using the infrastructure from ASP.NET and .NET (.NET Core 3.1 and .NET 5.0). The key differentiator for YARP is that it's been designed to be easily customized and tweaked via .NET code to match the specific needs of each deployment scenario. YARP can support configuration from appsettings.json file or code. In this post, you will explore how to use YARP in an empty ASP.NET Core web application, and that application will act a front end for two ASP.NET Core MVC applications. To get started, you can create a empty web application using the command ` dotnet new web`. Next you need to add YARP package. You can do this with the following command - `dotnet add package Microsoft.ReverseProxy --version 1.0.0-preview.9.21116.1`. It is still preview. 

Once the package added, you can configure the Startup class to read the configuration and enable the Reverse proxy. You can do it like this.

{% highlight CSharp %}
public class Startup
{
    public IConfiguration Configuration { get; set; }
    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddReverseProxy()
            .LoadFromConfig(Configuration.GetSection("ReverseProxy"));
    }

    // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }

        app.UseRouting();

        app.UseEndpoints(endpoints =>
        {
            endpoints.MapReverseProxy();
        });
    }
}
{% endhighlight %}

In the `ConfigureServices` method Reverse Proxy middleware is added and the configuration is reading from the appsettings.json file. And the `Configure` method, routing to the reverse proxy configuration is mapped. Next you need to modify the configuration, you can do this by editing the appsettings.json file. Here is the reverse proxy configuration.

{% highlight CSharp %}
"ReverseProxy": {
  "Routes": [
    {
      "RouteId": "route1",
      "ClusterId": "cluster1",
      "Match": {
        "Path": "{**catch-all}"
      }
    }
  ],
  "Clusters": {
    "cluster1": {
        
      "Destinations": {
        "cluster1/destination1": {
          "Address": "https://localhost:11000"
        },
        "cluster1/destination2": {
          "Address": "https://localhost:12000"
        }
      }
    }
  }
}
{% endhighlight %}

This configuration has mainly two elements - Routes and Clusters. Routes is where you can configure the endpoint routes and URLs. The `Match` element is configured to execute the proxy for all the routes. `RouteId` is unique name for the route and `ClusterId` is used to identify the backend application servers or URLs. Inside the `Clusters`, there are two application urls configured. It is same application running in different ports. Now you're ready to run the proxy application and your other web apps. The other web apps you can run in different ports with following command - `dotnet run --urls="https://localhost:xxxxx"`. Now when you try to browse https://localhost:5001, you will be able to see the Index Page of the Web Application - the URL wont change. If you keep on refreshing, sometimes you will be able to see the second app as well. By default YARP will use the `PowerOfTwoChoices` algorithm for load balancing. There are other built in policies like.

* First - Select the first destination without considering load. This is useful for dual destination fail-over systems.
* Random - Select a destination randomly.
* PowerOfTwoChoices (default) - Select two random destinations and then select the one with the least assigned requests. This avoids the overhead of LeastRequests and the worst case for Random where it selects a busy destination.
* RoundRobin - Select a destination by cycling through them in order.
* LeastRequests - Select the destination with the least assigned requests. This requires examining all destinations.

To configure any other load balancing policy you can modify the configuration like this. Here `RoundRobin` algorithm is used.

{% highlight CSharp %}
"Clusters": {
  "cluster1": {
    "LoadBalancingPolicy": "RoundRobin",
    "Destinations": {
      "cluster1/destination1": {
        "Address": "https://localhost:11000"
      },
      "cluster1/destination2": {
        "Address": "https://localhost:12000"
      }
    }
  }
}
{% endhighlight %}

YARP also supports traffic routing by checking the health of the destination application and route client requests based on that. If you're using ASP.NET Core apps, you can enable ASP.NET Core health check option for this purposes. YARP coming with lot of features and improvements. Check out their documentation and home page for more details on the existing features and how to use it.

### Reference Links

1. [Introducing YARP Preview 1](https://devblogs.microsoft.com/dotnet/introducing-yarp-preview-1?WT.mc_id=AZ-MVP-5002040)
2. [YARP Home Page](https://microsoft.github.io/reverse-proxy?WT.mc_id=AZ-MVP-5002040)
3. [YARP GitHub Repository](https://github.com/microsoft/reverse-proxy?WT.mc_id=AZ-MVP-5002040)

Happy Programming :)