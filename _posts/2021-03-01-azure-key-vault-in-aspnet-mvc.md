---
layout: post
title: "Add Azure Key Vault to support to your ASP.NET application"
subtitle: "This article discuss about how to Azure Key Vault support to your ASP.NET MVC application."
date: 2021-03-01 00:00:00
categories: [Azure,KeyVault,AspNetMVC]
tags: [Azure,KeyVault,AspNetMVC]
author: "Anuraj"
image: /assets/images/2021/03/connect_to_azure_keyvault.png
---
This article will discuss about how to connect and use Azure Key Vault in your ASP.NET MVC application. 

### Azure Key Vault - Provisioning and Configuration

To do this first you need to create an Azure Key Vault. It is a straight forward process. Login to azure portal, search for Key Vault, and select the Key Vault option.

![Create Azure Key Vault]({{ site.url }}/assets/images/2021/03/create_key_vault.png)

You need to provide a resource group, unique name and location, similar to most of the Azure resources, and click on `Review + Create`. And in the review screen confirm the details and create it.

![Create Azure Key Vault]({{ site.url }}/assets/images/2021/03/create_key_vault_2.png)

Next select the Secrets blade and add your app settings and connection strings. You can click on the `Generate/Import` button and choose the `Upload options` as `Manual`. Then configure your app settings and connection strings - keys and values to the Name and Value options. And keep other options as default.

![Create Secret]({{ site.url }}/assets/images/2021/03/create_secret.png)

Now you have completed your Key Vault configuration, you can click on Secrets blade and you can see your configured secrets list.

![List of Secrets]({{ site.url }}/assets/images/2021/03/list_of_secrets.png)

In the next section, you will learn how to connect Visual Studio to Azure Key Vault and access the secret values in the code.

### Connecting to Azure Key Vault

To connect to Azure Key Vault from Visual Studio, you need to right click on the project and select Add &gt; Connected Service menu.

![Connected Service]({{ site.url }}/assets/images/2021/03/connect_to_azure_keyvault.png)

From the options, choose `Secure Secrets with Azure Key Vault` option.

![Secure Secrets with Azure Key Vault]({{ site.url }}/assets/images/2021/03/connect_to_azure_keyvault2.png)

If you're not signed in you might need to prompt to sign in to your account. Once you signed in, you can choose your Subscription and Key Vault - by default Visual Studio will prompt you with a new key vault, since you already created on you can select it from the list.

![Connect and Select Azure Key Vault]({{ site.url }}/assets/images/2021/03/select_keyvault.png)

And click on the Add button to add key vault reference to your application. This will add reference of the NuGet package `Microsoft.Configuration.ConfigurationBuilders.Azure` to the project. Also it will add some configuration in the `Web.Config` file.

{% highlight XML %}
<configSections>
  <section name="configBuilders" 
            type="System.Configuration.ConfigurationBuildersSection, System.Configuration, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" 
            restartOnExternalChanges="false" 
            requirePermission="false" />
</configSections>
<configBuilders>
  <builders>
    <add name="AzureKeyVault" 
          vaultName="dotnetthoughts" 
          type="Microsoft.Configuration.ConfigurationBuilders.AzureKeyVaultConfigBuilder, Microsoft.Configuration.ConfigurationBuilders.Azure, Version=1.0.0.0, Culture=neutral" 
          vaultUri="https://dotnetthoughts.vault.azure.net" />
  </builders>
</configBuilders>
{% endhighlight %}

Now the configuration is completed. Now you can use modify your `appsettings` and `connectionstrings` sections like this, so that the application can read from Azure Key Vault.

{% highlight XML %}
<appSettings configBuilders="AzureKeyVault">
  <add key="webpages:Version" value="3.0.0.0" />
  <add key="webpages:Enabled" value="false" />
  <add key="ClientValidationEnabled" value="true" />
  <add key="UnobtrusiveJavaScriptEnabled" value="true" />
  <add key="TextAnalyticsKey" value="from key vault" />
</appSettings>
<connectionStrings configBuilders="AzureKeyVault">
  <add name="DefaultConnection" connectionString="from key vault" providerName="System.Data.SqlClient" />
  <add key="StorageConnectionString" value="from key vault" />
</connectionStrings>
{% endhighlight %}

And you're completed the implementation. Now if you run the application, and put a break point on the configuration value, you will be able to see the application is reading from Azure Key Vault instead of the value provided in the configuration file. Here is the sample code - `var textAnalyticsKey = ConfigurationManager.AppSettings["TextAnalyticsKey"];`

![Debugging Code]({{ site.url }}/assets/images/2021/03/debugging_code.png)

This way you can connect and use Azure Key Vault in your classic ASP.NET MVC applications. If you're application is running is using .NET Framework 4.5 or lower version, you might need to upgrade to latest version of .NET Framework. You can use Azure Key Vault for App Service certificates as well.

### Reference Links

1. [Azure Key Vault Developer's Guide](https://docs.microsoft.com/en-us/azure/key-vault/general/developers-guide?WT.mc_id=AZ-MVP-5002040)
2. [Add Key Vault to your web application by using Visual Studio Connected Services](https://docs.microsoft.com/en-us/azure/key-vault/general/vs-key-vault-add-connected-service?WT.mc_id=AZ-MVP-5002040)

Happy Programming :)