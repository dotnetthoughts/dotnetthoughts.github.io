---
layout: post
title: "Scaffold an entire .NET 5 Web API using Wrapt"
subtitle: "This post is about scaffolding an entire .NET 5 Web API with a simple yaml or json file using Wrapt, which helps you can focus on the high value features in your web app."
date: 2021-01-01 00:00:00
categories: [WebApi,AspNetCore,Scaffolding]
tags: [WebApi,AspNetCore,Scaffolding]
author: "Anuraj"
image: /assets/images/2021/01/craftsman_command.png
---
This post is about scaffolding an entire .NET 5 Web API with a simple yaml or json file using Wrapt, which helps you can focus on the high value features in your web app. Recently I came across this tool which helps you to scaffold a Web API application in Clean architecture with Open API documentation, unit tests and integration tests by providing a YAML or JSON file.

To get started you need to install a project template and a dotnet tool. So first you need to run the command - `dotnet new -i Foundation.Api` - this command will install the Foundation API project template. Next you need to install the `craftsman` tool which helps you to scaffold the project. You can install it using `dotnet tool install -g craftsman` command. Once installation is successful, you can run the `craftsman` command which will display an output like this.

![Craftsman command]({{ site.url }}/assets/images/2021/01/craftsman_command.png)

Next you need to create a YAML or JSON file - it is the template from which `craftsman` is generating the solution and projects. For this demo I am building a todo application YAML template file, which will look like this.

{% highlight Javascript %}
SolutionName: TodoApi
DbContext:
 ContextName: TodoDbContext
 DatabaseName: TodoDb
 Provider: SqlServer
Entities:
- Name: TodoItem
  Properties:
  - Name: TodoId
    IsPrimaryKey: true
    Type: int
    CanFilter: true
    CanSort: true
  - Name: Name
    Type: string
    CanFilter: true
    CanSort: true
  - Name: IsCompleted
    Type: bool
    CanFilter: true
    CanSort: true
Environments:
  - EnvironmentName: Startup
    ConnectionString: "Data Source=.;Initial Catalog=TodoDb;Integrated Security=True;Encrypt=True;TrustServerCertificate=True;"
    ProfileName: Dev
SwaggerConfig:
  Title: Todo Api Service
  Description: API for managing todo items.
  ApiContact: 
    Name: Anuraj
    Email: support@dotnetthoughts.net
    Url: https://dotnetthoughts.net
{% endhighlight %}

The YAML file is simple and straight forward. Here is the detailed explanation of each element.

1. SolutionName - This element used to configure the application solution name.
2. DbContext - This element is about configuring Database Context associated to the application. This element got other properties like DbContext name, Database name and Provider - Currently SqlServer and PostgreSQL only supported.
3. Entities - This element helps you to configure the database entities in the application. For the todo list application I am configuring only one entity. You need to provide the name of the property, type and associated meta data like sort and filter support.
4. Environments - This element helps you to configure environment configuration - which helps you to configure the connection string and associated variables for a certain environment.
4. SwaggerConfig - This element helps you to build the Open API documentation for the Web API.

You can copy / paste the YAML file and save it as `todo-api.yaml`. Next let's build the web api. You can run the command - ` craftsman new:api .\todo-api.yaml`, which will parse the YAML file and scaffold the solution and projects.

![Craftsman New API]({{ site.url }}/assets/images/2021/01/craftsman_generate_api.png)

So we successfully scaffolded the application. Next change the directory to `TodoApi` and execute the command `dotnet run --project webapi` - the `webapi` folder contains the API project file. Once the application is running, browse the `/swagger` endpoint which will display the Open API documentation like this.

![Open API documentation]({{ site.url }}/assets/images/2021/01/openapi_todo.png)

And here the folder structure created by the application. 

![Solution Explorer]({{ site.url }}/assets/images/2021/01/solution_explorer.png)

This project is created using the Clean Architecture, because of that the files and project are structured in certain way. Long back I wrote a [blog post](https://dotnetthoughts.net/how-to-generate-angular-code-from-openapi-specifications/) on how to generate Angular code from OpenAPI, using those utilities we can build web api client application as well.

You can find more details about Wrapt from [https://wrapt.dev/](https://wrapt.dev/). And here is the [getting started](https://wrapt.dev/docs/how-it-works) and [tutorial](https://wrapt.dev/docs/tutorial). Explore it and let me know your thoughts.

Happy Programming :)