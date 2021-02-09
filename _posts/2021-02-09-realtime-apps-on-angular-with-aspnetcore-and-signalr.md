---
layout: post
title: "Building Realtime applications on Angular with ASPNET Core and SignalR"
subtitle: "This article shows you how to build realtime applications on Angular with ASPNET Core and SignalR."
date: 2021-02-09 00:00:00
categories: [AspNetCore,Angular,SignalR]
tags: [AspNetCore,Angular,SignalR]
author: "Anuraj"
image: /assets/images/2021/02/signalr_app_running.png
---
This article shows you how to build realtime applications on Angular with ASP.NET Core and SignalR. To get started you need to create an ASP.NET Core application and configure SignalR hub in that. I am using a Web API application. You can do this by `dotnet new webapi` command. Once it is created, you need to create a Hub - which the one of the core components in the SignalR framework. Here is the Hub implementation.

{% highlight CSharp %}
using Microsoft.AspNetCore.SignalR;

namespace Backend
{
    public class MessageHub : Hub
    {
    }
}
{% endhighlight %}

There is no methods implemented in the Hub. Next you need to configure the `Startup` class to use SignalR. You can do this by adding the following code.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddSignalR();
    services.AddControllers();
    //code removed for brevity
}

public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    //code removed for brevity
    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllers();
        endpoints.MapHub<MessageHub>("/messageHub");
    });
}
{% endhighlight %}

Now you're ready to accept SignalR connections. But you haven't implemented any methods to interact with clients. So instead of writing the code in the Hub, you can write the code in the controller - which is better since you can access all the other HTTP parameters in the controller and you can expose action methods to external systems so that they can send notifications to the client applications. You need to create a new API controller and in the constructor you can get `HubContext` using that you can interact with SignalR. Here is the implementation.

{% highlight CSharp %}
[ApiController]
[Route("[controller]")]
public class MessageController : ControllerBase
{
    private readonly IHubContext<MessageHub> _hubContext;

    public MessageController(IHubContext<MessageHub> hubContext)
    {
        _hubContext = hubContext;
    }

    [HttpPost]
    public async Task<IActionResult> SendMessage([FromBody]string message)
    {
        await _hubContext.Clients.All.SendAsync("MessageReceived", message);
        return Ok();
    }
}
{% endhighlight %}

In this code, you're exposing the SendMessage API endpoint using this method external systems can send message to the clients. Next let us create the Angular project and write code to interact with SignalR server. You can create a Angular project using `ng new --minimal` command - the `minimal` argument will not create the spec files - this is not recommended for production apps. Once the Angular app dependency packages are installed, you need to install the client package for SignalR to interact with SignalR server. You can do this with `npm i @microsoft/signalr` command. Once installed, you can modify the `app.component.ts` file like this.

{% highlight Javascript %}

import { Component } from '@angular/core';
import * as signalR from "@microsoft/signalr";

@Component({
  selector: 'app-root',
  template: '',
  styles: []
})
export class AppComponent {
  title = 'Frontend';
  connection = new signalR.HubConnectionBuilder()
    .withUrl("https://localhost:5001/messageHub")
    .build();
  ngOnInit() {
    this.connection.on("MessageReceived", (message) => {
      console.log(message);
    });
    this.connection.start().catch(err => document.write(err));
  }
}

{% endhighlight %}

In the above code you're establishing a connection with the SignalR server and subscribing for an event - `MessageReceived` which is the method server will be invoking. And the message from the server displaying in the console. Next you can run both applications and verify the output. From Angular app, you will get a message like this.

![Angular Client without CORS]({{ site.url }}/assets/images/2021/02/angular_client_app.png)

It is because SignalR requires CORS to work properly, so you need to configure CORS policy in the SignalR server. You can do like this.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddCors(options => options.AddPolicy("CorsPolicy", builder =>
        {
            builder.AllowAnyHeader()
                .AllowAnyMethod()
                .SetIsOriginAllowed((host) => true)
                .AllowCredentials();
        }));
    services.AddSignalR();
    services.AddControllers();
}

public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    //code removed for brevity
    app.UseCors("CorsPolicy");
    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllers();
        endpoints.MapHub<NotificationHub>("/NotificationHub");
    });
}
{% endhighlight %}

In this above code you have configured to accept any header, any method and with credentials. Again it is not recommended production apps. In production apps use proper domain names and allow only required methods. Next run both applications - you can run web api app using `dotnet run` command and angular app using `ng serve --watch` command. And from the Swagger UI try out the `/Message`endpoint and you will be able to the message text you're sending reaching the client apps. Now you have completed a basic SignalR and Angular app implementation. But in real world the requirements may change - for example if you're building a chat application, you want to welcome the user or you want to transfer some client information to the backend. To do this you can use query strings - you can append the information you need to transfer to the server as the query strings in the Angular app like the following. 

{% highlight Javascript %}
connection = new signalR.HubConnectionBuilder()
    .withUrl("https://localhost:5001/messageHub?userId=123456")
    .build();
{% endhighlight %}

And you will be able to get these values in the server - hub - `OnConnected()` method like this.

{% highlight CSharp %}
public class MessageHub : Hub
{
    public override Task OnConnectedAsync()
    {
        var userId = Context.GetHttpContext().Request.Query["userId"];
        return base.OnConnectedAsync();
    }
}
{% endhighlight %}

This way you can build real time applications in Angular with ASP.NET Core and SignalR. Right now you're accepting the message string from server and displaying it in the console, instead of this you can define JSON payload and implement display logic in the client side. You can make the application more interactive with the help of Notification API in browsers. You can find the source code for this blog post [here](https://github.com/anuraj/AspNetCoreSamples/tree/master/AngularSignalRDemo). 

Happy Programming :)