---
tags: [Angular2, ROPC, OAUTH2, IDENTITYSERVER4, ASP.NET Core, TypeScript]
title: OAuth2 ROPC autentication with Angular 2 
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

The default asp.net core web app should now be available at [http://localhost:5000](http://localhost:5000).

Next, we add the IdentityServer4 nuget package by adding the following line to the project.json file in the "dependencies" property: `"IdentityServer4": "1.0.0"`

Let's now add the IdentityServer services to ASP.NET core HTTP pipeline, go to `Startup.cs`,
we will remove MVC from the pipeline since we have no need for it,
the `ConfigureServices` anf `Configure`  methods should like the code below.


``` csharp
// This method gets called by the runtime. Use this method to add services to the container.
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

> AddIdentityServer registers the IdentityServer services in DI. It also registers an in-memory store for runtime state.
> This is useful for development scenarios.
> For production scenarios you need a persistent or shared store like a database or cache for that.
> See the [EntityFramework](https://identityserver4.readthedocs.io/en/release/quickstarts/8_entity_framework.html#refentityframeworkquickstart) quickstart for more information.

Next thing to do is configure/add client app settings and a resource owner aka the end user of the client app.

