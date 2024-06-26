# Changelog
Newest changes listed first. 
The goal of this is to capture the changes needed to get a workable solution up and running. 
This will help me remember what changes to make in future backend projects. For now it will be extremely verbose.

## 2024.06.14.04 - PgAdmin support and persisting data between runs
Create a space on disk to persist settings, and add PgAdmin

We will use the .WithDataVolume() method on the dbServer. See [Persist .NET Aspire app data using volumes.](https://learn.microsoft.com/en-us/dotnet/aspire/fundamentals/persist-data-volumes)

Named volumes require a consisten password between app launches. See [Create a persisten password.](https://learn.microsoft.com/en-us/dotnet/aspire/fundamentals/persist-data-volumes) 

We will use the AddParameter method to create a parameter that can be used as a password. The password will be stored in the appSettings.development.json file. We will create a Parameters section as follows.
```
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "Parameters": {
    "DbServerPassword": "Db$3rv3r"
  }
}
```

Finally we will now add support for PgAdmin have a web based user interface for postgres administration.

There is an issue in Aspire [Set read permissions to allow pgadmin container to read the generated config file](https://github.com/dotnet/aspire/pull/4191) that is causing a problem with loading the Database Server in PgAdmin. Checking to see if it made it into a release yet.

So need to Add New Server Manually
```
dbServer
host.docker.internal
modify port according to dashboard
use paramter password value
```

## 2024.06.14.03 - Add PostgreSQL Support
Add a PostgreSQL database server to the solution
1. Add Aspire.Hosting.PostgreSQL package to AppHost project
`
dotnet add package aspire.hosting.postgresql
`
2. Add Postgres Database Server and Places Db and create a reference to it in teh API project. (see code changes)

The .WithReference(PlacesDb) allows for the creation of an environment variable "ConnectionStrings_PlacesDb" that has the connection infromation to the PlacesDb on the dbServer.



## 2024.06.14.02 - Https - Attempted Option B But Unsuccessful
I attempted to follow instructions for Linux and WSL, however I was unable to get to work. For now we are going forward with Option A. 
  * Filed Issue [#1](https://github.com/mikelor/tripperist-srv/issues/1)

## 2024.06.14.01 - Implemented Option A
Allow Unsecured Transport via Settings File
**ASPIRE_ALLOW_UNSECURED_TRANSPORT** environment variable in the AppHost  */properties/launchSettings.json* file to **"true"**

## 2024.06.14.01 - Add solution to github
1. Create tripperist-srv on github
2. git clohne https://github.com/mikelor/tripperist-srv
3. dotnet new aspire-starter -n Tripperist
4. git add .
5. git commit -m "Initial Project Version
6. dotnet build
7. dotnet run

```
warn: Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServer[8]
      The ASP.NET Core developer certificate is not trusted. For information about trusting the ASP.NET Core developer certificate, see https://aka.ms/aspnet/https-trust-dev-cert.
```
[Cause and Fix Described Here](https://learn.microsoft.com/en-us/aspnet/core/security/enforcing-ssl?view=aspnetcore-8.0&tabs=visual-studio%2Clinux-ubuntu#trust-the-aspnet-core-https-development-certificate-on-windows-and-macos)

### Option A - Allow Unsecured Transport via Settings File
**ASPIRE_ALLOW_UNSECURED_TRANSPORT** environment variable in the AppHost  */properties/launchSettings.json* file to **"true"**
```
    "http": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": true,
      "applicationUrl": "http://localhost:15234",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development",
        "DOTNET_ENVIRONMENT": "Development",
        "DOTNET_DASHBOARD_OTLP_ENDPOINT_URL": "http://localhost:19172",
        "DOTNET_RESOURCE_SERVICE_ENDPOINT_URL": "http://localhost:20185",
        "ASPIRE_ALLOW_UNSECURED_TRANSPORT": "true"
      }
```

### Option B - Trust Dev Cert
```
dotnet dev-certs https --trust
```
If running on Linux, follow [these instructions](https://learn.microsoft.com/en-us/aspnet/core/security/enforcing-ssl?view=aspnetcore-8.0&tabs=visual-studio%2Clinux-ubuntu#ssl-linux).