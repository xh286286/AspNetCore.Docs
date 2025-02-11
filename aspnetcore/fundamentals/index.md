---
title: ASP.NET Core fundamentals
author: rick-anderson
description: Learn the foundational concepts for building ASP.NET Core apps.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 12/15/2021
no-loc: [Home, Privacy, Kestrel, appsettings.json, "ASP.NET Core Identity", cookie, Cookie, Blazor, "Blazor Server", "Blazor WebAssembly", "Identity", "Let's Encrypt", Razor, SignalR]
uid: fundamentals/index
---
# ASP.NET Core fundamentals

::: moniker range=">= aspnetcore-6.0"

This article provides an overview of key topics for understanding how to develop ASP.NET Core apps.

## Program.cs

The `Startup` class is where:

* Services required by the app are configured.
* The app's request handling pipeline is defined, as a series of middleware components.

Here's a sample `Startup` class:

[!code-csharp[](index/samples_snapshot/3.x/Startup.cs?highlight=3,12)]

## Dependency injection (services)

ASP.NET Core includes a built-in dependency injection (DI) framework that makes configured services available throughout an app. For example, a logging component is a service.

Code to configure (or *register*) services is added to the `Startup.ConfigureServices` method. For example:

[!code-csharp[](index/samples_snapshot/3.x/ConfigureServices.cs)]

Services are typically resolved from DI using constructor injection. With constructor injection, a class declares a constructor parameter of either the required type or an interface. The DI framework provides an instance of this service at runtime.

The following example uses constructor injection to resolve a `RazorPagesMovieContext` from DI:

[!code-csharp[](index/samples_snapshot/3.x/Index.cshtml.cs?highlight=5)]

If the built-in Inversion of Control (IoC) container doesn't meet all of an app's needs, a third-party IoC container can be used instead.

For more information, see <xref:fundamentals/dependency-injection>.

## Middleware

The request handling pipeline is composed as a series of middleware components. Each component performs operations on an `HttpContext` and either invokes the next middleware in the pipeline or terminates the request.

By convention, a middleware component is added to the pipeline by invoking a `Use...` extension method in the `Startup.Configure` method. For example, to enable rendering of static files, call `UseStaticFiles`.

The following example configures a request handling pipeline:

[!code-csharp[](index/samples_snapshot/3.x/Configure.cs)]

ASP.NET Core includes a rich set of built-in middleware. Custom middleware components can also be written.

For more information, see <xref:fundamentals/middleware/index>.

## Host

On startup, an ASP.NET Core app builds a *host*. The host encapsulates all of the app's resources, such as:

* An HTTP server implementation
* Middleware components
* Logging
* Dependency injection (DI) services
* Configuration

There are two different hosts: 

* .NET Generic Host
* ASP.NET Core Web Host

The .NET Generic Host is recommended. The ASP.NET Core Web Host is available only for backwards compatibility.

The following example creates a .NET Generic Host:

[!code-csharp[](index/samples_snapshot/3.x/Program.cs)]

The `CreateDefaultBuilder` and `ConfigureWebHostDefaults` methods configure a host with a set of default options, such as:

* Use [Kestrel](#servers) as the web server and enable IIS integration.
* Load configuration from *appsettings.json*, *appsettings.{Environment Name}.json*, environment variables, command line arguments, and other configuration sources.
* Send logging output to the console and debug providers.

For more information, see <xref:fundamentals/host/generic-host>.

### Non-web scenarios

The Generic Host allows other types of apps to use cross-cutting framework extensions, such as logging, dependency injection (DI), configuration, and app lifetime management. For more information, see <xref:fundamentals/host/generic-host> and <xref:fundamentals/host/hosted-services>.

## Servers

An ASP.NET Core app uses an HTTP server implementation to listen for HTTP requests. The server surfaces requests to the app as a set of [request features](xref:fundamentals/request-features) composed into an `HttpContext`.

# [Windows](#tab/windows)

ASP.NET Core provides the following server implementations:

* *Kestrel* is a cross-platform web server. Kestrel is often run in a reverse proxy configuration using [IIS](https://www.iis.net/). In ASP.NET Core 2.0 or later, Kestrel can be run as a public-facing edge server exposed directly to the Internet.
* *IIS HTTP Server* is a server for Windows that uses IIS. With this server, the ASP.NET Core app and IIS run in the same process.
* *HTTP.sys* is a server for Windows that isn't used with IIS.

# [macOS](#tab/macos)

ASP.NET Core provides the *Kestrel* cross-platform server implementation. In ASP.NET Core 2.0 or later, Kestrel can run as a public-facing edge server exposed directly to the Internet. Kestrel is often run in a reverse proxy configuration with [Nginx](https://nginx.org) or [Apache](https://httpd.apache.org/).

# [Linux](#tab/linux)

ASP.NET Core provides the *Kestrel* cross-platform server implementation. In ASP.NET Core 2.0 or later, Kestrel can run as a public-facing edge server exposed directly to the Internet. Kestrel is often run in a reverse proxy configuration with [Nginx](https://nginx.org) or [Apache](https://httpd.apache.org/).

---

For more information, see <xref:fundamentals/servers/index>.

## Configuration

ASP.NET Core provides a configuration framework that gets settings as name-value pairs from an ordered set of configuration providers. Built-in configuration providers are available for a variety of sources, such as *.json* files, *.xml* files, environment variables, and command-line arguments. Write custom configuration providers to support other sources.

By [default](xref:fundamentals/configuration/index#default), ASP.NET Core apps are configured to read from *appsettings.json*, environment variables, the command line, and more. When the app's configuration is loaded, values from environment variables override values from *appsettings.json*.

The preferred way to read related configuration values is using the [options pattern](xref:fundamentals/configuration/options). For more information, see [Bind hierarchical configuration data using the options pattern](xref:fundamentals/configuration/index#optpat).

For managing confidential configuration data such as passwords, .NET Core provides the [Secret Manager](xref:security/app-secrets#secret-manager). For production secrets, we recommend [Azure Key Vault](xref:security/key-vault-configuration).

For more information, see <xref:fundamentals/configuration/index>.

## Environments

Execution environments, such as `Development`, `Staging`, and `Production`, are a first-class notion in ASP.NET Core. Specify the environment an app is running in by setting the `ASPNETCORE_ENVIRONMENT` environment variable. ASP.NET Core reads that environment variable at app startup and stores the value in an `IWebHostEnvironment` implementation. This implementation is available anywhere in an app via dependency injection (DI).

The following example configures the app to provide detailed error information when running in the `Development` environment:

[!code-csharp[](index/samples_snapshot/3.x/StartupConfigure.cs?highlight=3-6)]

For more information, see <xref:fundamentals/environments>.

## Logging

ASP.NET Core supports a logging API that works with a variety of built-in and third-party logging providers. Available providers include:

* Console
* Debug
* Event Tracing on Windows
* Windows Event Log
* TraceSource
* Azure App Service
* Azure Application Insights

To create logs, resolve an <xref:Microsoft.Extensions.Logging.ILogger%601> service from dependency injection (DI) and call logging methods such as <xref:Microsoft.Extensions.Logging.LoggerExtensions.LogInformation*>. For example:

[!code-csharp[](index/samples_snapshot/3.x/TodoController.cs?highlight=5,13,19)]

Logging methods such as `LogInformation` support any number of fields. These fields are commonly used to construct a message `string`, but some logging providers send these to a data store as separate fields. This feature makes it possible for logging providers to implement [semantic logging, also known as structured logging](https://softwareengineering.stackexchange.com/questions/312197/benefits-of-structured-logging-vs-basic-logging).

For more information, see <xref:fundamentals/logging/index>.

## Routing

A *route* is a URL pattern that is mapped to a handler. The handler is typically a Razor page, an action method in an MVC controller, or a middleware. ASP.NET Core routing gives you control over the URLs used by your app.

For more information, see <xref:fundamentals/routing>.

## Error handling

ASP.NET Core has built-in features for handling errors, such as:

* A developer exception page
* Custom error pages
* Static status code pages
* Startup exception handling

For more information, see <xref:fundamentals/error-handling>.

## Make HTTP requests

An implementation of `IHttpClientFactory` is available for creating `HttpClient` instances. The factory:

* Provides a central location for naming and configuring logical `HttpClient` instances. For example, register and configure a *github* client for accessing GitHub. Register and configure a default client for other purposes.
* Supports registration and chaining of multiple delegating handlers to build an outgoing request middleware pipeline. This pattern is similar to ASP.NET Core's inbound middleware pipeline. The pattern provides a mechanism to manage cross-cutting concerns for HTTP requests, including caching, error handling, serialization, and logging.
* Integrates with *Polly*, a popular third-party library for transient fault handling.
* Manages the pooling and lifetime of underlying `HttpClientHandler` instances to avoid common DNS problems that occur when managing `HttpClient` lifetimes manually.
* Adds a configurable logging experience via <xref:Microsoft.Extensions.Logging.ILogger> for all requests sent through clients created by the factory.

For more information, see <xref:fundamentals/http-requests>.

## Content root

The content root is the base path for:

* The executable hosting the app (*.exe*).
* Compiled assemblies that make up the app (*.dll*).
* Content files used by the app, such as:
  * Razor files (*.cshtml*, *.razor*)
  * Configuration files (*.json*, *.xml*)
  * Data files (*.db*)
* The [Web root](#web-root), typically the *wwwroot* folder.

During development, the content root defaults to the project's root directory. This directory is also the base path for both the app's content files and the [Web root](#web-root). Specify a different content root by setting its path when [building the host](#host). For more information, see [Content root](xref:fundamentals/host/generic-host#contentroot).

## Web root

The web root is the base path for public, static resource files, such as:

* Stylesheets (*.css*)
* JavaScript (*.js*)
* Images (*.png*, *.jpg*)

By default, static files are served only from the web root directory and its sub-directories. The web root path defaults to *{content root}/wwwroot*. Specify a different web root by setting its path when [building the host](#host). For more information, see [Web root](xref:fundamentals/host/generic-host#webroot).

Prevent publishing files in *wwwroot* with the [\<Content> project item](/visualstudio/msbuild/common-msbuild-project-items#content) in the project file. The following example prevents publishing content in *wwwroot/local* and its sub-directories:

```xml
<ItemGroup>
  <Content Update="wwwroot\local\**\*.*" CopyToPublishDirectory="Never" />
</ItemGroup>
```

In Razor *.cshtml* files, tilde-slash (`~/`) points to the web root. A path beginning with `~/` is referred to as a *virtual path*.

For more information, see <xref:fundamentals/static-files>.

::: moniker-end

::: moniker range="< aspnetcore-6.0"

This article provides an overview of key topics for understanding how to develop ASP.NET Core apps.

## The Startup class

The `Startup` class is where:

* Services required by the app are configured.
* The app's request handling pipeline is defined, as a series of middleware components.

Here's a sample `Startup` class:

[!code-csharp[](index/samples_snapshot/3.x/Startup.cs?highlight=3,12)]

For more information, see <xref:fundamentals/startup>.

## Dependency injection (services)

ASP.NET Core includes a built-in dependency injection (DI) framework that makes configured services available throughout an app. For example, a logging component is a service.

Code to configure (or *register*) services is added to the `Startup.ConfigureServices` method. For example:

[!code-csharp[](index/samples_snapshot/3.x/ConfigureServices.cs)]

Services are typically resolved from DI using constructor injection. With constructor injection, a class declares a constructor parameter of either the required type or an interface. The DI framework provides an instance of this service at runtime.

The following example uses constructor injection to resolve a `RazorPagesMovieContext` from DI:

[!code-csharp[](index/samples_snapshot/3.x/Index.cshtml.cs?highlight=5)]

If the built-in Inversion of Control (IoC) container doesn't meet all of an app's needs, a third-party IoC container can be used instead.

For more information, see <xref:fundamentals/dependency-injection>.

## Middleware

The request handling pipeline is composed as a series of middleware components. Each component performs operations on an `HttpContext` and either invokes the next middleware in the pipeline or terminates the request.

By convention, a middleware component is added to the pipeline by invoking a `Use...` extension method in the `Startup.Configure` method. For example, to enable rendering of static files, call `UseStaticFiles`.

The following example configures a request handling pipeline:

[!code-csharp[](index/samples_snapshot/3.x/Configure.cs)]

ASP.NET Core includes a rich set of built-in middleware. Custom middleware components can also be written.

For more information, see <xref:fundamentals/middleware/index>.

## Host

On startup, an ASP.NET Core app builds a *host*. The host encapsulates all of the app's resources, such as:

* An HTTP server implementation
* Middleware components
* Logging
* Dependency injection (DI) services
* Configuration

There are two different hosts: 

* .NET Generic Host
* ASP.NET Core Web Host

The .NET Generic Host is recommended. The ASP.NET Core Web Host is available only for backwards compatibility.

The following example creates a .NET Generic Host:

[!code-csharp[](index/samples_snapshot/3.x/Program.cs)]

The `CreateDefaultBuilder` and `ConfigureWebHostDefaults` methods configure a host with a set of default options, such as:

* Use [Kestrel](#servers) as the web server and enable IIS integration.
* Load configuration from *appsettings.json*, *appsettings.{Environment Name}.json*, environment variables, command line arguments, and other configuration sources.
* Send logging output to the console and debug providers.

For more information, see <xref:fundamentals/host/generic-host>.

### Non-web scenarios

The Generic Host allows other types of apps to use cross-cutting framework extensions, such as logging, dependency injection (DI), configuration, and app lifetime management. For more information, see <xref:fundamentals/host/generic-host> and <xref:fundamentals/host/hosted-services>.

## Servers

An ASP.NET Core app uses an HTTP server implementation to listen for HTTP requests. The server surfaces requests to the app as a set of [request features](xref:fundamentals/request-features) composed into an `HttpContext`.

# [Windows](#tab/windows)

ASP.NET Core provides the following server implementations:

* *Kestrel* is a cross-platform web server. Kestrel is often run in a reverse proxy configuration using [IIS](https://www.iis.net/). In ASP.NET Core 2.0 or later, Kestrel can be run as a public-facing edge server exposed directly to the Internet.
* *IIS HTTP Server* is a server for Windows that uses IIS. With this server, the ASP.NET Core app and IIS run in the same process.
* *HTTP.sys* is a server for Windows that isn't used with IIS.

# [macOS](#tab/macos)

ASP.NET Core provides the *Kestrel* cross-platform server implementation. In ASP.NET Core 2.0 or later, Kestrel can run as a public-facing edge server exposed directly to the Internet. Kestrel is often run in a reverse proxy configuration with [Nginx](https://nginx.org) or [Apache](https://httpd.apache.org/).

# [Linux](#tab/linux)

ASP.NET Core provides the *Kestrel* cross-platform server implementation. In ASP.NET Core 2.0 or later, Kestrel can run as a public-facing edge server exposed directly to the Internet. Kestrel is often run in a reverse proxy configuration with [Nginx](https://nginx.org) or [Apache](https://httpd.apache.org/).

---

For more information, see <xref:fundamentals/servers/index>.

## Configuration

ASP.NET Core provides a configuration framework that gets settings as name-value pairs from an ordered set of configuration providers. Built-in configuration providers are available for a variety of sources, such as *.json* files, *.xml* files, environment variables, and command-line arguments. Write custom configuration providers to support other sources.

By [default](xref:fundamentals/configuration/index#default), ASP.NET Core apps are configured to read from *appsettings.json*, environment variables, the command line, and more. When the app's configuration is loaded, values from environment variables override values from *appsettings.json*.

The preferred way to read related configuration values is using the [options pattern](xref:fundamentals/configuration/options). For more information, see [Bind hierarchical configuration data using the options pattern](xref:fundamentals/configuration/index#optpat).

For managing confidential configuration data such as passwords, .NET Core provides the [Secret Manager](xref:security/app-secrets#secret-manager). For production secrets, we recommend [Azure Key Vault](xref:security/key-vault-configuration).

For more information, see <xref:fundamentals/configuration/index>.

## Environments

Execution environments, such as `Development`, `Staging`, and `Production`, are a first-class notion in ASP.NET Core. Specify the environment an app is running in by setting the `ASPNETCORE_ENVIRONMENT` environment variable. ASP.NET Core reads that environment variable at app startup and stores the value in an `IWebHostEnvironment` implementation. This implementation is available anywhere in an app via dependency injection (DI).

The following example configures the app to provide detailed error information when running in the `Development` environment:

[!code-csharp[](index/samples_snapshot/3.x/StartupConfigure.cs?highlight=3-6)]

For more information, see <xref:fundamentals/environments>.

## Logging

ASP.NET Core supports a logging API that works with a variety of built-in and third-party logging providers. Available providers include:

* Console
* Debug
* Event Tracing on Windows
* Windows Event Log
* TraceSource
* Azure App Service
* Azure Application Insights

To create logs, resolve an <xref:Microsoft.Extensions.Logging.ILogger%601> service from dependency injection (DI) and call logging methods such as <xref:Microsoft.Extensions.Logging.LoggerExtensions.LogInformation*>. For example:

[!code-csharp[](index/samples_snapshot/3.x/TodoController.cs?highlight=5,13,19)]

Logging methods such as `LogInformation` support any number of fields. These fields are commonly used to construct a message `string`, but some logging providers send these to a data store as separate fields. This feature makes it possible for logging providers to implement [semantic logging, also known as structured logging](https://softwareengineering.stackexchange.com/questions/312197/benefits-of-structured-logging-vs-basic-logging).

For more information, see <xref:fundamentals/logging/index>.

## Routing

A *route* is a URL pattern that is mapped to a handler. The handler is typically a Razor page, an action method in an MVC controller, or a middleware. ASP.NET Core routing gives you control over the URLs used by your app.

For more information, see <xref:fundamentals/routing>.

## Error handling

ASP.NET Core has built-in features for handling errors, such as:

* A developer exception page
* Custom error pages
* Static status code pages
* Startup exception handling

For more information, see <xref:fundamentals/error-handling>.

## Make HTTP requests

An implementation of `IHttpClientFactory` is available for creating `HttpClient` instances. The factory:

* Provides a central location for naming and configuring logical `HttpClient` instances. For example, register and configure a *github* client for accessing GitHub. Register and configure a default client for other purposes.
* Supports registration and chaining of multiple delegating handlers to build an outgoing request middleware pipeline. This pattern is similar to ASP.NET Core's inbound middleware pipeline. The pattern provides a mechanism to manage cross-cutting concerns for HTTP requests, including caching, error handling, serialization, and logging.
* Integrates with *Polly*, a popular third-party library for transient fault handling.
* Manages the pooling and lifetime of underlying `HttpClientHandler` instances to avoid common DNS problems that occur when managing `HttpClient` lifetimes manually.
* Adds a configurable logging experience via <xref:Microsoft.Extensions.Logging.ILogger> for all requests sent through clients created by the factory.

For more information, see <xref:fundamentals/http-requests>.

## Content root

The content root is the base path for:

* The executable hosting the app (*.exe*).
* Compiled assemblies that make up the app (*.dll*).
* Content files used by the app, such as:
  * Razor files (*.cshtml*, *.razor*)
  * Configuration files (*.json*, *.xml*)
  * Data files (*.db*)
* The [Web root](#web-root), typically the *wwwroot* folder.

During development, the content root defaults to the project's root directory. This directory is also the base path for both the app's content files and the [Web root](#web-root). Specify a different content root by setting its path when [building the host](#host). For more information, see [Content root](xref:fundamentals/host/generic-host#contentroot).

## Web root

The web root is the base path for public, static resource files, such as:

* Stylesheets (*.css*)
* JavaScript (*.js*)
* Images (*.png*, *.jpg*)

By default, static files are served only from the web root directory and its sub-directories. The web root path defaults to *{content root}/wwwroot*. Specify a different web root by setting its path when [building the host](#host). For more information, see [Web root](xref:fundamentals/host/generic-host#webroot).

Prevent publishing files in *wwwroot* with the [\<Content> project item](/visualstudio/msbuild/common-msbuild-project-items#content) in the project file. The following example prevents publishing content in *wwwroot/local* and its sub-directories:

```xml
<ItemGroup>
  <Content Update="wwwroot\local\**\*.*" CopyToPublishDirectory="Never" />
</ItemGroup>
```

In Razor *.cshtml* files, tilde-slash (`~/`) points to the web root. A path beginning with `~/` is referred to as a *virtual path*.

For more information, see <xref:fundamentals/static-files>.

::: moniker-end
