---
tags: [Angular2, ROPC, OAUTH2, IdentityServer4, ASP.NET Core, TypeScript]
title: OAuth2 ROPC Flow Authorization with Angular 2 
---

This post will cover the topic of authenticating an Angular 2 client using the ROPC flow, managing the token
and in brief how to setup IdentityServer4 for ROPC.
We will also create a simple API using ASP.NET Core MVC 6.
<!--more-->
Some basics first, what is ROPC?

> The OAuth 2.0 resource owner password grant allows a client to send username and password
> to the token service and get an access token back that represents that user.

ROPC flow should only be used for "trusted" clients,
what that means is that the app sending the username and password to the AUTH server should be
ideally written by you or a trusted 3rd party.
The reason being is that it will have the responsibility of transfering the users credentials securely
and not exploit them.

A simple diagram showing ROPC flow.

<pre>
+----------+
| Resource |
|  Owner   |
|          |
+----------+
     v
     |    Resource Owner
    (1) Password Credentials
     |
     v
+---------+                                  +---------------+
|         |&gt;--(2)---- Resource Owner -------&gt;|               |
|         |         Password Credentials     | Authorization |
| Client  |                                  |     Server    |
|         |&lt;--(3)---- Access Token ---------&lt;|               |
|         |    (w/ Optional Refresh Token)   |               |
+---------+                                  +---------------+
</pre>

The creators of IdentityServer recomend that ROPC should not be used, and instead advice the use
of "Implicit Flow" but that one involves an extra step of redirecting the user which i wanted to avoid
for my app for UX purposes and also i am the creator of both the API and the client Angular 2 app.

So let's get started with the IdentityServer.

## Identity Server Setup

Make sure you have **Node** installed and also **.Net Core**.

First we will need to install some NPM packages from the terminal, like Yeoman to create our .net core project.

 1. Install Yeoman and Bower globally : `npm install -g yo bower`
 2. Install the Asp.net generator for Yeoman : `npm install -g generator-aspnet`

If you are running on a Mac or Linux, you may get a permission error, if so run the commands under `sudo`.
{: .notice--info}

Great we are now ready to create the Authorization server project.
First create a folder to hold all the projects, i will name it "ROPCDemo" and inside it we will create the
IdentityServer project.

```
mkdir ROPCDemo
cd ROPCDemo
```

Now the terminal is at "ROPCDemo" folder and we will scaffold a .net core project here, run the Yeoman ASP.NET generator:
`yo aspnet`

Choose "Empty Web Application", next name the project "AuthServer"

![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/posts/OAuth2-ROPC-autentication-with-Angular2/yoAspNet.jpg){: .align-center}

Go to "AuthServer" folder and restore the needed .net core packages, then run the web app.

```
cd AuthServer
dotnet restore
dotnet run
```

Next, we add the IdentityServer4 nuget package by adding the following line to the project.json file in the "dependencies" property: `"IdentityServer4": "1.0.0"`

![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/posts/OAuth2-ROPC-autentication-with-Angular2/dep.jpg){: .align-center}

Run `dotnet restore` to fetch the IdentityServer4 package.

Let's now add the IdentityServer services to ASP.NET core HTTP pipeline, go to `Startup.cs`,
the `ConfigureServices` anf `Configure`  methods should like the code below.


``` csharp
public void ConfigureServices(IServiceCollection services)
{
   services.AddIdentityServer()
            .AddTemporarySigningCredential();
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
{
    loggerFactory.AddConsole(LogLevel.Debug);
    app.UseDeveloperExceptionPage();

    app.UseIdentityServer();
}
```

> The `AddTemporarySigningCredential` extension creates temporary RSA key pair for signing tokens on every startup.
> Useful to get started, but needs to be replaced by some persistent key material for production scenarios.

> `AddIdentityServer` registers the IdentityServer services in DI.  
> It also registers an in-memory store for runtime state.
> This is useful for development scenarios.
> For production scenarios you need a persistent or shared store like a database or cache for that.
> See the [EntityFramework](https://identityserver4.readthedocs.io/en/release/quickstarts/8_entity_framework.html#refentityframeworkquickstart) quickstart for more information.

Next thing to do is configure/add client app settings and a resource owner aka the end user of the client app.

### Adding Users to IdentityServer4

We will use the `TestUser` class provided by IdentityServer and store it in-memory, ideal for the purposes of this blog post,
for production though you should store users information securely in a DataBase.

I would recommend you use "ASP.NET Identity",
you can take a look at how to tie the two together [here.](https://identityserver4.readthedocs.io/en/release/quickstarts/6_aspnet_identity.html)

Create a new file called `Config.cs` in the root of the "AuthServer" and create a method that returns a list of "TestUser" instances.

``` csharp
using IdentityServer4.Models;
using IdentityServer4.Test;
using System.Collections.Generic;

namespace AuthServer
{
    public class Config
    {
        public static List<TestUser> GetUsers()
        {
            return new List<TestUser>
            {
                new TestUser
                {
                    SubjectId = "1",
                    Username = "boban",
                    Password = "p@ssword"
                }
            };
        }
    }
}
```

We have a user now, moving on to adding the API information

```csharp
// scopes define the API resources in your system
public static IEnumerable<ApiResource> GetApiResources()
{
    return new List<ApiResource>
    {
        new ApiResource("SecureAPI", "API for testing purposes")
    };
}
```

We are missing the Client so let's add another method that returns a list of Clients.

The Client we will create will use the API we just added, so we will set the " AllowedScopes"
property to "SecureAPI", this will restrict access to other APIs  we might have.

``` csharp
// client want to access resources (aka scopes)
public static IEnumerable<Client> GetClients()
{
    // client credentials client
    return new List<Client>
    {
        // resource owner password grant client
        new Client
        {
            ClientId = "ro.ng2Client",
            AllowedGrantTypes = GrantTypes.ResourceOwnerPassword,
            ClientSecrets = { new Secret("secret".Sha256()) },
            AllowedScopes = { "SecureAPI" }
        }
    };
}
```

Now we need to tell IdentityServer about the Users, APIs and Clients, we do that in the "Startup.cs" file.
Let's revisit the "ConfigureServices" method.

``` csharp
public void ConfigureServices(IServiceCollection services)
{
    // configure identity server with in-memory stores, keys, clients and scopes
    services.AddIdentityServer()
        .AddTemporarySigningCredential()
        // Set API
        .AddInMemoryApiResources(Config.GetApiResources())
        // Set the Clients
        .AddInMemoryClients(Config.GetClients())
        // Set the Users
        .AddTestUsers(Config.GetUsers());
}
```

That's it, we are done setting up IdentityServer, start the AuthServer, type in the terminal `dotnet run` and go to [http://localhost:5000/.well-known/openid-configuration](http://localhost:5000/.well-known/openid-configuration)
if you see the JSON (so-called discovery document) then all went fine.

![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/posts/OAuth2-ROPC-autentication-with-Angular2/discoveryDocument.jpg){: .align-center}

## Creating the API

Open the terminal and go to the "ROPCDemo" folder, again we will use Yeoman to scaffold our API project, run `yo aspnet`.

Go down in the project options and select "Web API Application", name the project "API", after the project has been created
run `dotnet restore`.

Open the "Program.cs" file and configure "Kestrel" (the web server) to run on port 5001.

``` csharp
namespace API
{
    public class Program
    {
        public static void Main(string[] args)
        {
            var config = new ConfigurationBuilder()
                .AddCommandLine(args)
                .AddEnvironmentVariables(prefix: "ASPNETCORE_")
                .Build();

            var host = new WebHostBuilder()
                .UseConfiguration(config)
                .UseKestrel()
                .UseUrls("http://localhost:5001")
                .UseContentRoot(Directory.GetCurrentDirectory())
                .UseIISIntegration()
                .UseStartup<Startup>()
                .Build();

            host.Run();
        }
    }
}
```

Run the API and go to [http://localhost:5001/api/values](http://localhost:5001/api/values) you should
get a response that's an array `["value1","value2"]`, if you see that than the API is working correctly
and we can now move on to securing it.

Adding the Authorization/Authentication middleware is simple.

What will this middleware do:

* validate the incoming token to make sure it is coming from a trusted issuer
* validate that the token is valid to be used with this api (aka scope)

Add the following package to your project.json: `"IdentityServer4.AccessTokenValidation": "1.0.1"` 
and run `dotnet restore`.

After the packages has been downloaded, we can add it to the pipeline,
open "Startup.cs" and add the IdentityServer middleware as shown below.

``` csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
{
    loggerFactory.AddConsole(Configuration.GetSection("Logging"));
    loggerFactory.AddDebug();

    // IdentityServer middleware setup
    app.UseIdentityServerAuthentication(new IdentityServerAuthenticationOptions
    {
        Authority = "http://localhost:5000",
        RequireHttpsMetadata = false,

        ApiName = "ROPCApi"
    });

    app.UseMvc();
}
```

Let's test it out, open "ValuesController" at the top add `using Microsoft.AspNetCore.Authorization;` and then add
the `[Authorize]` attribute to the controller, save and run it.

Now if we go to open [http://localhost:5001/api/values](http://localhost:5001/api/values) we don't see the values returned
instead we got an error code "401 Unauthorized" (open the network tab on your browser dev tools).

![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/posts/OAuth2-ROPC-autentication-with-Angular2/401.jpg){: .align-center}

Yay the API is now secured.

## Creating the Angular Client