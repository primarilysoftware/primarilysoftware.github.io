---
layout: post
title: "The Minimalist Guide to ASP.NET Core: Getting Started"
comments: true
description: "An minimalistic overview of how to setup a new ASP.NET Core app"
keywords: "asp.net core, asp.net core 2.1, csharp, web"
---

ASP.NET Core has really arrived.  With the release of ASP.NET Core 2.1, the framework has reached a level
of maturity and adoption that it cannot be overlooked when starting a new project.  It has been a bit of 
a bumpy road getting to this point.  The API surface for ASP.NET Core apps has evolved rapidly.  Early
adopters of ASP.NET Core were unfortunately caught in the shuffle while the ASP.NET Core team refined
and improved the framework.  Now that 2.1 is out, it is starting to feel like the framework has reached
a point of stability.  So, I think this is the perfect time to kick the tires on Microsoft's new hot rod
and take a closer look at what ASP.NET Core has to offer.

### The Minimalist Guide
Some developers are happy to follow whatever recipe a framework may give you
(File > New Project > ASP.NET Web Application > With MVC, With Identity Auth, With Bootstrap, With...).  While
there is nothing wrong with that approach, in fact that is often a great place to start, that is not what this guide is
about.  In this guide, I want to take a minimalist approach to a variety of topics.  Cut out all the cruft,
and try and find the simplest possible implementation for various features.  The hope is that exploring
ASP.NET Core in this way I (and you) will gain a better understanding of how the bits and bytes that make up
the framework fit together, leading to a better intuition for how to best use the framework to build web apps.

### Getting Started
To start, I am going to create a new ASP.NET Core Web Application, cutting out everything but the essentials.
Using Visual Studio, you can
File > New Project > ASP.NET Core Web Application, selecting the Empty project template.
You can acheive the same thing via the command line with:

```
dotnet new web
```

That is going to generate 3 files:

* EmptyWebApp.csproj - the project file
* Program.cs - contains the entry point for the app (Main)
* Startup.cs - contains startup code to configure/bootstrap the app

Lets take a look at each of those files in a little more detail.

### The Project File
Compared to a full framework app, the project file for a ASP.NET Core app is incredibly sparse.

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>netcoreapp2.1</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <Folder Include="wwwroot\" />
  </ItemGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore.App" />
  </ItemGroup>

</Project>
```

The root `Project` element has a single 
[Sdk attribute](https://docs.microsoft.com/en-us/dotnet/core/tools/csproj#sdk-attribute)
specifying which SDK should be used to build the app.  In this case, we are using the Web SDK
(Microsoft.NET.Sdk.Web).

> There is also a non-web SDK that you will see used for .NET Core console
> apps or class libraries (Microsoft.NET.Sdk).

This `Sdk` attribute points the build tools to the appropriate MSBuild targets file to use when
building the app.  Yes, MSBuild is still being used under the covers to build the app (if it ain't
broke, don't fix it).  You can actually take a look at the targets file if you feel so inclined.
You can find the targets file for the Web SDK at:

```
C:\Program Files\dotnet\sdk\2.1.301\Sdks\Microsoft.NET.Sdk.Web\Sdk\Sdk.targets
```

> Note that the specific SDK version in the path above is going to change over time.
>
> If you are really curious about what MSBuild is doing, try running the following command:
> `dotnet msbuild /pp:fullproject.xml`

Under the project element, you will notice a `PropertyGroup` specifying the `TargetFramework`,
`netcoreapp2.1` in this case.  That makes sense, given we are building a ASP.NET Core 2.1
app.

> There is a bit of magic here that is worth pointing out.  When specifying the `TargetFramework`,
> there is an implicit reference taken to an associated
> [metapackage](https://github.com/dotnet/core/blob/master/release-notes/1.0/sdk/1.0-rc3-implicit-package-refs.md)
> for the targeted framework version. Making this reference implicit rather than explicit helps to avoid issues that
> can arise if the `TargetFramework` version and the metapackage version of framework libraries gets out of sync.

Next up, we see an `ItemGroup` that includes the `wwwroot` folder.  I suppose that is a reasonable
thing to include in most projects, but is not necessary, and can be removed.

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>netcoreapp2.1</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore.App" />
  </ItemGroup>

</Project>
```

Better.  Finally, there is a `PackageReference` to `Microsoft.AspNetCore.App`.  This is
[another metapackage](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/metapackage-app?view=aspnetcore-2.1)
that brings in all the libraries for the ASP.NET part of ASP.NET Core.  Notice that there are no version numbers here.
If one has not been specified, one is determined implicitly by the Web SDK.  Again, this helps to avoid conflicts when
upgrading package/framework versions.

In the name of minimalism, we should take a closer look at what this package contains.
[This list of dependencies](https://www.nuget.org/packages/Microsoft.AspNetCore.App/2.1.1)
is not for the feint of heart.  There is a lot in there that you may or may not need, depending on the app
(Entity Framework, Razor, Authentication, etc.).  To slim things down a bit, you can reference the
`Microsoft.AspNetCore` package instead.  This package will only inlcude the essentials.

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>netcoreapp2.1</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore" Version="2.1.1" />
  </ItemGroup>

</Project>
```

> Note that in this case, a `Version` is required.

Looking even better.

Wait a second, the app has a Startup.cs and Program.cs, yet those file references are missing from the project file.
What is going on here?  Well the SDK is doing
[a little extrac work for us](https://docs.microsoft.com/en-us/dotnet/core/tools/csproj#default-compilation-includes-in-net-core-projects).
By convention, the SDK (remember the SDK is just an MSBuild target) is going to inlcude all files that match the glob
`**/*.cs` for compilation.

### Program.cs

The `Program.cs` file defines the `Main` method, the entry point for the app.

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        CreateWebHostBuilder(args).Build().Run();
    }

    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
            .UseStartup<Startup>();
}
```

There is not much interesting going on here.  Most of the configuration has been placed in the `Startup.cs` class.

### Startup.cs

This is where all the interesting startup would live for the app.

```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
    }

    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }

        app.Run(async (context) =>
        {
            await context.Response.WriteAsync("Hello World!");
        });
    }
}
```

There is not much going on here, which is to be expected since this is an empty app.  I am not a fan of how
`Program.cs` and `Startup.cs` are split up.  It makes it harder to see how the pieces fit together.  That approach
might make more sense as the app startup logic gets more complicated, but for now, lets combine these files,
and cutout the parts that aren't necessary.

```csharp
public static void Main(string[] args)
{
    WebHost.CreateDefaultBuilder(args)
        .Configure(app =>
        {
            app.Run(async (context) =>
            {
                await context.Response.WriteAsync("Hello World!");
            });
        })
        .Build()
        .Run();
}
```

Note that the bit about showing a developer exception page has been removed, and there is no need to configure
any services just yet, so that is gone too.  What is left is fairly comprehensible.  Use the `WebHostBuilder`
to configure the `WebHost`, our app.  For now, any request will return a static response.  Once the `WebHost`
has been built, call `Run` to start it up.

### Wrapping Up

At this point, the app will start, and navigating to the site will return the response "Hello World!".  Not
the most impressive use of technology, but it is a start.  What is impressive is this is a functioning web app
written in ~10 lines of code.

Next time we will build on this start, adding some more useful features.
