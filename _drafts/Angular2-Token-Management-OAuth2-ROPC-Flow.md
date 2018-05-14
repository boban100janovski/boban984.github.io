---
tags: [Angular2, ROPC, OAUTH2, IdentityServer4, ASP.NET Core, TypeScript]
title: Angular 2 Token Management, OAuth2 - ROPC Flow
---
{% include toc %}

This post will cover the topic of authenticating an Angular 2 client using the ROPC flow, managing the tokens (bearer and refresh token)
and in brief how to setup IdentityServer4 for the ROPC flow.
We will also create a simple API using ASP.NET Core MVC 6 to test out the tokens.
<!--more-->

If you have a the Authentication and API layer already done, feel free to jump to the Angular client part of this post,
the auth service class that holds the logic is not tied to a specific backend implementation so it will work for your project as well, just make sure you change the URLs.

I will assume you already have some basic knowledge of Angular and TypeScript.

Some basics first, what is ROPC authentication flow?

> The OAuth 2.0 resource owner password grant allows a client to send username and password
> to the token service and get an access token back that represents that user.

ROPC flow should only be used for "trusted" clients,
what that means is that the app sending the username and password to the AUTH server should be
written by you (ideally) or a trusted 3rd party.
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
for my app for UX purposes and also i will create both the API and the client app.

Before we start, make sure you have **Node** and **.Net Core** installed.

## Identity Server Setup

First we will need to install some NPM packages from the terminal, like Yeoman to create our .net core projects.

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

Go to "AuthServer" folder and restore the needed .net core packages.

```
cd AuthServer
dotnet restore
```

Next, we add the IdentityServer4 nuget package by adding the following line to the project.json file in the "dependencies" property: `"IdentityServer4": "1.0.0"`.

We should add one more package to Allow CORS since we will be calling the IdentityServer from a different port, add also `"Microsoft.AspNetCore.Cors": "1.1.0"`

Your "project.json" file should look similar to the image below.

![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/posts/OAuth2-ROPC-autentication-with-Angular2/dep.jpg){: .align-center}

Run `dotnet restore` to fetch the packages.

Let's now add the IdentityServer services to ASP.NET core HTTP pipeline, go to `Startup.cs`,
edit the `ConfigureServices` and `Configure` methods so they look like the code below.

``` csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddCors(options=>
    {
        // this defines a CORS policy called "default"
        options.AddPolicy("default", policy => {
            policy.WithOrigins("http://localhost:5002")
                .AllowAnyHeader()
                .AllowAnyMethod();
        });
    });

   services.AddIdentityServer()
           .AddTemporarySigningCredential();
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
{
    loggerFactory.AddConsole(LogLevel.Debug);
    app.UseDeveloperExceptionPage();

    // this uses the policy called "default"
    app.UseCors("default");

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

We will use the `TestUser` class provided by IdentityServer and store the user information in-memory,
ideal for the purposes of this blog post,
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
                    Password = "pass"
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

We have the user and the API defined, the Client is missing, so let's add another method that returns a list of Clients.

The Client will use the API we just added, so we will set the "AllowedScopes"
property to "SecureAPI", this will allow access to the named API, other APIs will not be accessible.
You can play with the roles and scopes to fine-tune which user/client can access your API methods.

``` csharp
// client want to access resources (aka scopes)
public static IEnumerable<Client> GetClients()
{
    // client credentials client
    return new List<Client>
    {
        // resource owner password grant client
         new Client {
            ClientId = "ng2Client",
            AllowedGrantTypes = GrantTypes.ResourceOwnerPassword,
            ClientSecrets = { new Secret("secret".Sha256()) },
            AllowOfflineAccess = true,
            AllowedScopes = { "SecureAPI" },
            AllowedCorsOrigins = { "http://localhost:5002" },

            AccessTokenLifetime = 60,
            AbsoluteRefreshTokenLifetime = 300
        }
    };
}
```

Now we need to tell IdentityServer about the Users, APIs and Clients we created, to do that open the "Startup.cs" file.
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

That's it, we are done setting up IdentityServer, let's start it, run `dotnet run` in the terminal and go to [http://localhost:5000/.well-known/openid-configuration](http://localhost:5000/.well-known/openid-configuration)
if you see the JSON (so-called discovery document) then all went fine.

![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/posts/OAuth2-ROPC-autentication-with-Angular2/discoveryDocument.jpg){: .align-center}

## Creating the API

With the IdentityServer all done, it's time to create a very simple API using ASP.NET Core MVC.
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

After the packages have been downloaded, we can add it to the pipeline,
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

## Angular 2 Token Management

With the API and the IdentityServer setup we can now focus on the Angular client.
Most of the work here will need to be done on managing the bearer and refresh token.
As before we will use Yeoman to generate our Angular 2 project, we will be using the generator from ["Steve Sanders"](http://blog.stevensanderson.com/).

Learn more about that [here](https://github.com/aspnet/JavaScriptServices).

I am using this generator because i've already used it a couple of times and i really like what it brings to the table,
feel free to use others such as Angular CLI or my preffered way ... create everything from scratch by hand.

The generator will create an Angular 2 Client app with MVC and WebPack already setup.

Fire up the terminal, let's install the generator first, run the following command: `npm install -g generator-aspnetcore-spa`.

After it's installed, create a new folder called "ngxClient" in the "ROPCDemo" folder and
navigate to it, then run `yo aspnetcore-spa` and select the "Angular 2" template.

On my laptop the generator is finished in about a minute or two, so be patient.

If everything went ok you should be able to run the project by using `dotnet run` in the "ngxClient" folder.

![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/posts/OAuth2-ROPC-autentication-with-Angular2/ngxClient1.jpg){: .align-center}

I will use the home component to build the UI for sending the username/password to the IdentityServer.

<!--[![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/posts/OAuth2-ROPC-autentication-with-Angular2/home.jpg)]({{ site.url }}{{ site.baseurl }}/assets/images/posts/OAuth2-ROPC-autentication-with-Angular2/home.jpg){: .align-center .image-popup}-->

![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/posts/OAuth2-ROPC-autentication-with-Angular2/home.jpg)

There are a couple of situations we need to care of :

* Store tokens
* Schedule a refresh of the token after expiration from a login (no tokens to begin with)
* Schedule the refresh after a page refresh (we already have the tokens)
* UnSchedule the refresh observer and revoke the token after user logs out

Part of the code is borrowed and sligtly modified from [angular2-jwt](https://github.com/auth0/angular2-jwt),
, we have some methods in there we can use for decoding and extracting the data from the token
, what's missing there is the part about refreshing the token
, we have some of the logic about refreshing at this [blog post](http://blog.ionic.io/ionic-2-and-auth0/).

I will combine the two so we will have a working Angular 2 app that handles the tokens and the Http calls to the API.

Let's start then, create an interface for the token response

``` ts
export interface IAuthInfo {
    access_token: string;
    expires_in: number;
    token_type: string;
    refresh_token: string;
}
```

i added the new file in `/ClientApp/app/common/models/` folder, we will use it
to set the response type, of the call to the IdentityServer.

Now we will create the Auth service which will handle all the main requierments for managing the tokens.

Create a new service in the `/ClientApp/app/common/services/auth.service.ts`.

We will put all the token endpoints at the top and also an observable for the token

``` ts
import { Injectable, EventEmitter } from '@angular/core';
import { Http, Headers, Response } from '@angular/http';
import { Observable } from 'rxjs/Observable';
import '../rxjs-extensions';

import { IAuthInfo } from '../models/IAuthInfo';

@Injectable()
export class AuthService {
    public tokenStream: Observable<string>;
    private _refreshSubscription: any;

    // Endpoints
    public static apiUrl: string = 'localhost:5001';
    public static baseAuthUrl = 'localhost:5000/connect/';
    public static tokenUrl = AuthService.baseAuthUrl + 'token';
    public static revokeUrl = AuthService.baseAuthUrl + 'revocation';
    public static infoUrl = AuthService.baseAuthUrl + 'userinfo';

    public static clientId: string = 'ng2Client';
    public static clientSecret = 'secret';
}
```

Ok, now let's add the constructor and a `authInfo` property which we will use to store and retrieve the Token information, using localStorage.
The constructor is where we tell our `tokenStream` observable where to get the value from.

``` ts
    constructor(private _http: Http) {
        this.tokenStream = new Observable<string>((obs: any) => {
            obs.next(this.authInfo.access_token);
        });
    }

    /**
     * The AuthInfo ( access token and refresh token)
     */
    private _authInfo: IAuthInfo = null;
    get authInfo(): IAuthInfo {
        return this._authInfo || JSON.parse(localStorage.getItem('authInfo'));
    }
    set authInfo(token: IAuthInfo) {
        if (token) {
            this._authInfo = token;
            localStorage.setItem('authInfo', JSON.stringify(token));
        } else {
            this._authInfo = null;
            localStorage.removeItem('authInfo');
        }
    }
```