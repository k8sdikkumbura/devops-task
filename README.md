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


### Step -04 Update the Startup.cs

### Step 05 Create Dockerfile

### Step 06 Verify the app container status

docker logs -f sales-api-container

info: Microsoft.Hosting.Lifetime[0]
      Now listening on: http://[::]:80
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Production
info: Microsoft.Hosting.Lifetime[0]
      Content root path: /app

### Step 07 Access the Sales API 

http://localhost:8080/api/sales



 


