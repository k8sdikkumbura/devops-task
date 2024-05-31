# DevOps Challenge: Setting Up the Development Environment

This guide will walk you through setting up your development environment for a .NET 5 application with a SQL Server database running in a Docker container.

## Prerequisites

- Ubuntu 22.04 Desktop machine
- .NET 5 SDK
- Docker
- Git

## Deploy the MS SQL db in docker

Before that i have deployed the MS SQL database using docker container & create the database as well

Start SQL Server with Docker

```bash
docker run -e 'ACCEPT_EULA=Y' -e 'SA_PASSWORD=your_password' -p 1433:1433 -d mcr.microsoft.com/mssql/server:2019-latest
```

## Create the database 

```bash
CREATE DATABASE SalesDb;
GO
SELECT name FROM sys.databases;
GO
EXIT
```

## Updated the appsettings.json

## Updated the Startup.cs

## Create Dockerfile

##  Verify the app container status

```bash
docker logs -f sales-api-container

info: Microsoft.Hosting.Lifetime[0]
      Now listening on: http://[::]:80
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Production
info: Microsoft.Hosting.Lifetime[0]
      Content root path: /app
```
## Access the Sales API 
http://localhost:8080/api/sales


## Set Up CI/CD Pipeline





## Introduce Best Practices

Add XML comments to your API controllers and configure Swagger to use them.
SalesController.cs

```bash
/// <summary>
/// Handles sales data.
/// </summary>
[ApiController]
[Route("api/[controller]")]
public class SalesController : ControllerBase
{
    /// <summary>
    /// Gets all sales.
    /// </summary>
    /// <returns>List of sales.</returns>
    [HttpGet]
    public IActionResult GetSales()
    {
        // implementation
    }

    // Other actions
}
```

## Remove Kestrel Server Header

In Program.cs, add the following code:
```bash
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.AddServerHeader = false;
});
```


