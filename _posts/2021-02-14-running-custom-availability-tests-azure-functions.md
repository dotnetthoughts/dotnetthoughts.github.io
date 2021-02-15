---
layout: post
title: "Running custom availability tests using Azure Functions"
subtitle: "This article shows you how create and run custom availability tests using Azure Functions. This is an update to the blog post about monitoring WebJobs using Application Insights, since Microsoft is deprecating the Load Test and Performance features, this is the recommended alternative for monitoring availability."
date: 2021-02-14 00:00:00
categories: [Azure,WebJobs,Monitoring,Serverless]
tags: [Azure,WebJobs,Monitoring,Serverless]
author: "Anuraj"
image: /assets/images/2021/02/application_insights_failures.png
---
This article shows you how create and run custom availability tests using Azure Functions. This is an update to the blog post about monitoring WebJobs using Application Insights, since Microsoft is deprecating the Load Test and Performance features, this is the recommended alternative for monitoring availability. To get started you need to create an Azure function in C# with Timer Trigger. You can choose the trigger interval based on your requirement. This is one of the main advantage compared to Application Insights availability test feature. Next you need to add following packages.

* Microsoft.ApplicationInsights - This package is for interacting with the Application Insights.
* Microsoft.Azure.Functions.Extensions - This package is for creating a Startup class for the Azure Function.
* Microsoft.Extensions.Http - This package is for creating the HttpClient which helps to interact with Web Jobs REST API.

This `Startup` class is required so that you can inject the `HttpClient` to the Azure Function - it is the recommended instead of creating the HttpClient instance every time in the Function code. Here is the Startup class code.

{% highlight CSharp %}
[assembly: FunctionsStartup(typeof(DotNetThoughts.Demo.Startup))]
namespace DotNetThoughts.Demo
{
    public class Startup : FunctionsStartup
    {
        public override void Configure(IFunctionsHostBuilder builder)
        {
            builder.Services.AddHttpClient();
        }
    }
}
{% endhighlight %}

And here is the Function implementation.

{% highlight CSharp %}
private readonly TelemetryClient _telemetryClient;
private readonly string instrumentationKey = Environment.GetEnvironmentVariable("APPINSIGHTS_INSTRUMENTATIONKEY");
private readonly string webJobAuthenticationToken = Environment.GetEnvironmentVariable("WEBJOB_AUTHORIZATION");
private readonly string webJobRESTEndpoint = Environment.GetEnvironmentVariable("WEBJOB_RESTENDPOINT");
private const string EndpointAddress = "https://dc.services.visualstudio.com/v2/track";
private readonly IHttpClientFactory _httpClientFactory;
public WebJobAvailability(IHttpClientFactory httpClientFactory)
{
    _httpClientFactory = httpClientFactory;
    _telemetryClient = new TelemetryClient(
            new TelemetryConfiguration(instrumentationKey, 
            new InMemoryChannel { EndpointAddress = EndpointAddress }));
}

[FunctionName("WebJobAvailability")]
public async Task Run([TimerTrigger("0 */1 * * * *")] TimerInfo myTimer, ILogger log)
{
    string testName = "WebJobStatusTest";
    string location = Environment.GetEnvironmentVariable("REGION_NAME");
    string operationId = Guid.NewGuid().ToString("N");
    var availabilityTelemetry = new AvailabilityTelemetry
    {
        Id = operationId,
        Name = testName,
        RunLocation = location,
        Success = false
    };

    try
    {
        await ExecuteTestAsync(log);
        availabilityTelemetry.Success = true;
    }
    catch (Exception ex)
    {
        availabilityTelemetry.Message = ex.Message;

        var exceptionTelemetry = new ExceptionTelemetry(ex);
        exceptionTelemetry.Context.Operation.Id = operationId;
        exceptionTelemetry.Properties.Add("TestName", testName);
        exceptionTelemetry.Properties.Add("TestLocation", location);
        _telemetryClient.TrackException(exceptionTelemetry);
    }
    finally
    {
        _telemetryClient.TrackAvailability(availabilityTelemetry);
        _telemetryClient.Flush();
    }
}
{% endhighlight %}

The `ExecuteTestAsync` execute the business logic or test implementation. In this you will be writing the code to interact with Kudu REST endpoint and verify the response status code.

{% highlight CSharp %}
public async Task ExecuteTestAsync(ILogger log)
{
    log.LogInformation("RunAvailabilityTestAsync - Started.");
    var httpClient = _httpClientFactory.CreateClient();
    httpClient.DefaultRequestHeaders.Authorization =
        new AuthenticationHeaderValue("Basic", webJobAuthenticationToken);
    var webJobResponseJson = await httpClient.GetStringAsync(webJobRESTEndpoint);
    var webJobResponse = JsonSerializer.Deserialize<WebJobResponse>(webJobResponseJson);
    if (!webJobResponse.LatestRun.Status.Equals("Success", StringComparison.OrdinalIgnoreCase))
    {
        throw new Exception("Latest Run status is not success.");
    }
}
{% endhighlight %}

`WebJobResponse` class is the POCO implementation of REST endpoint response. You can convert the JSON response to POCO using json2csharp.com or using Visual Studio. Now you can run the function and verify the status of the Web Job.

![Application Insights - Failures]({{ site.url }}/assets/images/2021/02/application_insights_failures.png)

This way you can create and run custom availability tests using Azure Functions. Similar to Application Insights availability tests you can configure alerts. As the location is null it is showing empty in the availability blade. If you host it on Azure Functions, the location variable will be auto populated.

### Reference Links

1. [Create and run custom availability tests using Azure Functions](https://docs.microsoft.com/en-us/azure/azure-monitor/app/availability-azure-functions?WT.mc_id=AZ-MVP-5002040)

Happy Programming :)