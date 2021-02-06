---
layout: post
title: "Managing Azure App Service SSL Certificate with Azure Key Vault"
subtitle: "This article shows you how to manage Azure App Service SSL Certificate with the help Azure Key Vault service."
date: 2021-02-05 00:00:00
categories: [Azure,AppService,Azure KeyVault]
tags: [Azure,AppService,Azure KeyVault]
author: "Anuraj"
image: /assets/images/2021/02/azure_keyvault.png
---
In my last [blog post](https://dotnetthoughts.net/working-with-ssl-certificate-azure-app-service/) I wrote about working with SSL certificate in Azure App Service. In this article I will explain how to manage Azure App Service SSL certificates with Azure Key Vault Service. If you're running SAAS applications on Azure App Service with custom domains and SSL certificates it is quite complicated. Here is a screenshot of an App Service running a SAAS app with custom domain and SSL certificates.

![Multiple custom domains and SSL Certificates]({{ site.url }}/assets/images/2021/02/multiple-tls-certificate.png)

In this scenario updating the SSL is quite complicated because you need to upload the SSL certificate, and manually change all the SSL/TLS bindings for all the custom domains. The alternate option is to use Azure Key Vault. So instead of uploading the SSL certificate to the app service directly, you will be uploading the certificate to the Azure Key Vault and access the certificate from App service via Key Vault. Incase of SSL certificate renewal you will be able to upload the latest certificate to the KeyVault and it internally manage and provide the latest certificate to the app service.

To use Azure Key Vault, you need to create an Azure Vault service. In Azure portal click on New Resource and search for Azure Key Vault. You need to select a resource group and provide a name of the Key Vault and click on the `Review and Create` button. The name should be unique and using this name you will be able to interact with key vault using REST API.

![Create Azure Key Vault]({{ site.url }}/assets/images/2021/02/create_azure_keyvault.png)

Once you create the Key Vault service, you can import the SSL certificate to Azure Key Vault with the help to import option. Similar to App Service you will be able to import PFX file in Azure Key vault as well. To get this option, you can choose the `Certificates` option from the Key Vault, and you can click on the `Generate / Import`, this will display a screen like this, after you choose `Import` from the list. 

![Import SSL certificate]({{ site.url }}/assets/images/2021/02/create_certificate.png)

Similar to App Service you need to provide the PFX file password in this screen. Once it is imported, you may need to assign an identity to manage this SSL from Web App. First you need to enable `Identity` in the App Service.

![Enable App Service Identity]({{ site.url }}/assets/images/2021/02/appservice_enable_identity.png)

Once you enable App Service identity, you will be able to assign Azure Key Vault permissions to the identity. To do this, you need to select `Access policies` from the Azure Key Vault, and click on the `Add Access Policy` option.

![Access Policies]({{ site.url }}/assets/images/2021/02/access_policy.png)

In the `Add access policy` option, choose `Get` option in Key permissions, Secret permissions, and Certificate permissions. And for the Select Principal, click on the `None Selected`. And search for the web application on which you have enabled the Identity.

![Add Access Policy]({{ site.url }}/assets/images/2021/02/add_access_policy.png)

Once selected, click on the Select button and click on the Add button. After it got added to the Key Vault. In the Web Application, select `TLS/SSL settings` and select the `Private key certificates (.pfx)` option. And click on the `Import Key Vault Certificate` option.

![Import Key Vault Certificate]({{ site.url }}/assets/images/2021/02/import_ssl_keyvault.png)

Now you can bind the SSL certificate to the custom domains. As I mentioned earlier if you're using SSL certificate from Azure Key Vault - renewal of SSL certificate can be automated. If you're using SSL certificate from Azure Key Vault integrated provider the creation and renewal can be automated. You can use Azure Key Vault to store your application secrets and keys as well. And you will be able to manage it with Azure Key Vault Client SDKs.

### Reference Links

1. [About Azure Key Vault](https://docs.microsoft.com/en-us/azure/key-vault/general/overview?WT.mc_id=AZ-MVP-5002040)
2. [Add a TLS/SSL certificate in Azure App Service - Import a certificate from Key Vault](https://docs.microsoft.com/en-us/azure/app-service/configure-ssl-certificate#import-a-certificate-from-key-vault?WT.mc_id=AZ-MVP-5002040)
3. [Assign a Key Vault access policy using the Azure portal](https://docs.microsoft.com/en-us/azure/key-vault/general/assign-access-policy-portal?WT.mc_id=AZ-MVP-5002040)
4. [Tutorial: Use a managed identity to connect Key Vault to an Azure web app in .NET](https://docs.microsoft.com/en-us/azure/key-vault/general/tutorial-net-create-vault-azure-web-app?WT.mc_id=AZ-MVP-5002040)

Happy Programming :)