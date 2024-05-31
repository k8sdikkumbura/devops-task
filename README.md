## Note:- I'm using ubuntu 22.04 desktop machine.I have done the test using this machine.


### Step 1: Set Up the Development Environment

First, make sure you have the necessary tools installed on your Ubuntu 22.04 machine:

.NET 5 SDK
Docker
Git

### Install .NET 5 SDK:

wget https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
sudo apt-get update
sudo apt-get install -y apt-transport-https
sudo apt-get update
sudo apt-get install -y dotnet-sdk-5.0

### Install Docker:

sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get install -y docker-ce
sudo usermod -aG docker ${USER}


## Step 2: Update DB Connection String


a) Update the connection string in appsettings.Development.json

Before that i have deployed the MS SQL database using docker container & create the database as well

## Start SQL Server with Docker

docker run -e 'ACCEPT_EULA=Y' -e 'SA_PASSWORD=your_password' -p 1433:1433 -d mcr.microsoft.com/mssql/server:2019-latest

docker exec -it condescending_goldberg /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P your_password

## Create the SalesDb Database

CREATE DATABASE SalesDb;
GO

## Verify the Database Creation

SELECT name FROM sys.databases;
GO

Exit the SQL Command Prompt
EXIT


### Step 03- Update the appsettings.json
vim src/DevOpsChallenge.SalesApi/appsettings.json

{
  "Logging": {
    "LogLevel": {
      "Default": "Warning",
      "DevOpsChallenge": "Information",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost,1433;Database=SalesDb;User Id=sa;Password=your_password;"
  }
}

### Step -04 Update the Startup.cs

vim src/DevOpsChallenge.SalesApi/Startup.cs

using DevOpsChallenge.SalesApi.Business;
using DevOpsChallenge.SalesApi.Database;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Mvc.Controllers;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Microsoft.OpenApi.Models;
using System;
using System.IO;
using System.Reflection;
using System.Text.Json.Serialization;

namespace DevOpsChallenge.SalesApi
{
    /// <summary>
    /// Start the web application.
    /// </summary>
    public class Startup
    {
        /// <summary>
        /// Initializes a new instance of the <see cref="Startup"/> class.
        /// </summary>
        /// <param name="configuration">The application configuration.</param>
        public Startup(IConfiguration configuration)
        {
            Configuration = configuration;
        }

        /// <summary>
        /// Gets the application configuration.
        /// </summary>
        public IConfiguration Configuration { get; }

        /// <summary>
        /// Configure the dependency injection container.
        /// </summary>
        /// <param name="services">The dependency injection service collection.</param>
        public void ConfigureServices(IServiceCollection services)
        {
            // --------------------------------------------------------------------------------
            // DOMAIN
            // --------------------------------------------------------------------------------

            // Business
            services.AddBusiness();

            // --------------------------------------------------------------------------------
            // DATABASE
            // --------------------------------------------------------------------------------

            // Database Context Options
            void DbContextOptionsBuilder(DbContextOptionsBuilder builder) =>
                builder.UseSqlServer(Configuration.GetConnectionString("DefaultConnection"), o =>
                    o.MigrationsAssembly(typeof(DatabaseContext).Assembly.FullName));

            // Databases
            services.AddDatabase(DbContextOptionsBuilder);

            // --------------------------------------------------------------------------------
            // HTTP PIPELINE
            // --------------------------------------------------------------------------------

            // MVC
            services
                .AddControllers()
                .AddJsonOptions(options =>
                {
                    options.JsonSerializerOptions.IgnoreNullValues = true;
                    options.JsonSerializerOptions.Converters.Add(new JsonStringEnumConverter());
                });

            // Swagger
            services.AddSwaggerGen(c =>
            {
                // Information
                c.SwaggerDoc("v1", new OpenApiInfo { Title = "sales-api", Version = "v1" });

                // Comments
                string xmlCommentsFilePath = Path.Combine(AppContext.BaseDirectory, $"{Assembly.GetExecutingAssembly().GetName().Name}.xml");
                if (File.Exists(xmlCommentsFilePath))
                {
                    c.IncludeXmlComments(xmlCommentsFilePath);
                }

                // Operation IDs - required for some client code generation tools
                c.CustomOperationIds(x => (x.ActionDescriptor as ControllerActionDescriptor)?.ActionName);
            });
        }

        /// <summary>
        /// Configure the HTTP pipeline and some services.
        /// </summary>
        /// <param name="app">The application pipeline builder.</param>
        /// <param name="env">The hosting environment.</param>
        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            // Development mode
            if (env.IsDevelopment())
            {
                // Show exceptions
                app.UseDeveloperExceptionPage();

                // Swagger JSON
                app.UseSwagger();

                // Swagger UI
                app.UseSwaggerUI(c =>
                {
                    c.SwaggerEndpoint($"/swagger/v1/swagger.json", "sales-api" + ' ' + "v1");
                });
            }

            // Endpoint routing
            app.UseRouting();

            // Endpoints
            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllers();
            });
        }
    }
}

### Step 05 Create Dockerfile

vim Dockerfile

FROM mcr.microsoft.com/dotnet/aspnet:5.0 AS base
WORKDIR /app
EXPOSE 80

FROM mcr.microsoft.com/dotnet/sdk:5.0 AS build
WORKDIR /src
COPY ["src/DevOpsChallenge.SalesApi/DevOpsChallenge.SalesApi.csproj", "src/DevOpsChallenge.SalesApi/"]
RUN dotnet restore "src/DevOpsChallenge.SalesApi/DevOpsChallenge.SalesApi.csproj"
COPY . .
WORKDIR "/src/src/DevOpsChallenge.SalesApi"
RUN dotnet build "DevOpsChallenge.SalesApi.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "DevOpsChallenge.SalesApi.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "DevOpsChallenge.SalesApi.dll"]

### Step 05 Verify the app container status

Step -04 Check the logs in the app container
milan@sl-mdikkumburage:~/workspace/mytask/devops-task/devops-challenge-dotnet(main)$  docker logs -f sales-api-container
info: Microsoft.Hosting.Lifetime[0]
      Now listening on: http://[::]:80
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Production
info: Microsoft.Hosting.Lifetime[0]
      Content root path: /app

### Step 06 Access the Sales API 

http://localhost:8080/api/sales

 


