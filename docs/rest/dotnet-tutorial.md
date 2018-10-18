---
title: How to use Outlook REST APIs in an ASP.NET MVC app
description: Learn how to use Microsoft Graph in an ASP.NET web app to access the Outlook API. This tutorial goes step-by-step to authorize and list the inbox.
author: jasonjoh

ms.topic: get-started-article
ms.technology: ms-graph
ms.devlang: csharp
ms.date: 02/20/2018
ms.author: jasonjoh
---

# Write an ASP.NET MVC Web app to get Outlook mail, calendar, and contacts

The purpose of this guide is to walk through the process of creating a simple ASP.NET MVC C# app that retrieves messages in Office 365 or Outlook.com. The source code in this [repository](https://github.com/jasonjoh/dotnet-tutorial) is what you should end up with if you follow the steps outlined here.

This tutorial will use the [Microsoft Authentication Library (MSAL)](https://www.nuget.org/packages/Microsoft.Identity.Client) to make OAuth2 calls and the [Microsoft Graph Client Library](https://www.nuget.org/packages/Microsoft.Graph/) to call the Mail API. Microsoft recommends using Microsoft Graph to access Outlook mail, calendar, and contacts. You should use the Outlook APIs directly (via `https://outlook.office.com/api`) only if you require a feature that is not available on the Graph endpoints. For a version of this sample that uses the Outlook APIs, see [this branch](https://github.com/jasonjoh/dotnet-tutorial/tree/outlook-api).

This guide assumes that you already have Visual Studio 2013 or Visual Studio 2015 installed and working on your development machine.

## Create the app

Let's dive right in! In Visual Studio, create a new Visual C# ASP.NET Web Application using .NET Framework 4.5. Name the application `dotnet-tutorial`.

![The Visual Studio New Project window.](images/dotnet-tutorial/new-project.PNG)

Select the **MVC** template. Click the **Change Authentication** button and choose "No Authentication". Un-select the "Host in the cloud" checkbox. The dialog should look like the following.

![The Visual Studio Template Selection window.](images/dotnet-tutorial/template-selection.PNG)

Click OK to have Visual Studio create the project. Once that's done, run the project to make sure everything's working properly by pressing **F5** or choosing **Start Debugging** from the **Debug** menu. You should see a browser open displaying the stock ASP.NET home page. Close your browser.

Now that we've confirmed that the app is working, we're ready to do some real work.

## Designing the app

Our app will be very simple. When a user visits the site, they will see a button to log in and view their email. Clicking that button will take them to the Azure login page where they can login with their Office 365 or Outlook.com account and grant access to our app. Finally, they will be redirected back to our app, which will display a list of the most recent email in the user's inbox.

Let's begin by replacing the stock home page with a simpler one. Open the `./Views/Home/Index.cshtml` file. Replace the existing code with the following code.

#### Contents of the `./Views/Home/Index.cshtml` file

```csharp
@{
    ViewBag.Title = "Home Page";
}

<div class="jumbotron">
    <h1>ASP.NET MVC Tutorial</h1>
    <p class="lead">This sample app uses the Mail API to read messages in your inbox.</p>
    <p><a href="#" class="btn btn-primary btn-lg">Click here to login</a></p>
</div>
```

This is basically repurposing the `jumbotron` element from the stock home page, and removing all of the other elements. The button doesn't do anything yet, but the home page should now look like the following.

![The sample app's home page.](images/dotnet-tutorial/home-page.PNG)

Let's also modify the stock error page so that we can pass an error message and display it. Replace the contents of `./Views/Shared/Error.cshtml` with the following code.

#### Contents of the `./Views/Shared/Error.cshtml` file

```csharp
<!DOCTYPE html>
<html>
<head>
    <meta name="viewport" content="width=device-width" />
    <title>Error</title>
</head>
<body>
    <hgroup>
        <h1>Error.</h1>
        <h2>An error occurred while processing your request.</h2>
    </hgroup>
    <div class="alert alert-danger">@ViewBag.Message</div>
    @if (!string.IsNullOrEmpty(ViewBag.Debug))
    {
    <pre><code>@ViewBag.Debug</code></pre>
    }
</body>
</html>
```

Finally add an action to the `HomeController` class to invoke the error view.

#### The `Error` action in `./Controllers/HomeController.cs` ####

```csharp
public ActionResult Error(string message, string debug)
{
    ViewBag.Message = message;
    ViewBag.Debug = debug;
    return View("Error");
}
```

## Register the app

[!include[App Registration Intro](~/includes/rest/app-registration-intro.md)]

Head over to the [Application Registration Portal](https://apps.dev.microsoft.com/) to quickly get an application ID and secret. 

1. Using the **Sign in** link, sign in with either your Microsoft account (Outlook.com), or your work or school account (Office 365).
1. Click the **Add an app** button. Enter `dotnet-tutorial` for the name and click **Create application**. 
1. Locate the **Application Secrets** section, and click the **Generate New Password** button. Copy the password now and save it to a safe place. Once you've copied the password, click **Ok**.
1. Locate the **Platforms** section, and click **Add Platform**. Choose **Web**, then enter `http://localhost:<PORT>`, replacing `<PORT>` with the port number your project is using under **Redirect URIs**.

    > [!TIP]
    > You can find the port number your project is using by selecting the project in **Solution Explorer**, then checking the value of **URL** under **Development Server** in the **Properties** window.
1. Click **Save** to complete the registration. Copy the **Application Id** and save it along with the password you copied earlier. We'll need those values soon.

Here's what the details of your app registration should look like when you are done.

![A screenshot of the completed app registration in the Application Registration Portal](images/dotnet-tutorial/app-registration.PNG)

## Implementing OAuth2 ##

Our goal in this section is to make the link on our home page initiate the [OAuth2 Authorization Code Grant flow with Azure AD](https://msdn.microsoft.com/en-us/library/azure/dn645542.aspx). To make things easier, we'll use the [Microsoft Authentication Library (MSAL)](https://www.nuget.org/packages/Microsoft.Identity.Client) to handle our OAuth requests.

First, let's create a separate config file to hold the OAuth settings for the app. Right-click the **dotnet-tutorial** solution in **Solution Explorer**, choose **Add**, then **New Item...**. Select **Web Configuration File**, then enter `AzureOauth.config` in the **Name** field. Click **Add**.

Open the `AzureOauth.config` file and replace its entire contents with the following.

```xml
<appSettings>
    <add key="ida:AppID" value="YOUR APP ID" />
    <add key="ida:AppPassword" value="YOUR APP PASSWORD" />
    <add key="ida:RedirectUri" value="http://localhost:10800" />
    <add key="ida:AppScopes" value="User.Read Mail.Read" />
</appSettings>
```

Replace the value of the `ida:AppID` key with the application ID you generated above, and replace the value of the `ida:AppPassword` key with the password you generated above. If the value of your redirect URI is different, be sure to update the value of `ida:RedirectUri`.

Now open the `Web.config` file. Find the line with the `<appSettings>` element, and change it to the following.

```xml
<appSettings file="AzureOauth.config">
```

This will cause ASP.NET to add the keys from the `AzureOauth.config` file at runtime. By keeping these values in a separate file, we make it less likely that we'll accidentally commit them to source control.

The next step is to install the OWIN middleware, MSAL, and Graph libraries from NuGet. On the Visual Studio **Tools** menu, choose **NuGet Package Manager**, then **Package Manager Console**. To install the OWIN middleware libraries, enter the following commands in the Package Manager Console:

```Powershell
Install-Package Microsoft.Owin.Security.OpenIdConnect
Install-Package Microsoft.Owin.Security.Cookies
Install-Package Microsoft.Owin.Host.SystemWeb
```

Next install the **Microsoft Authentication Library** with the following command:

```Powershell
Install-Package Microsoft.Identity.Client -Pre
```

Finally install the **Microsoft Graph Client Library** with the following command:

```Powershell
Install-Package Microsoft.Graph
```

### Back to coding

Now we're all set to do the sign in. Let's start by wiring the OWIN middleware to our app. Right-click the **App_Start** folder in **Project Explorer** and choose **Add**, then **New Item**. Choose the **OWIN Startup Class** template, name the file `Startup.cs`, and click **Add**. Replace the entire contents of this file with the following code.

```csharp
using System;
using System.Configuration;
using System.IdentityModel.Claims;
using System.IdentityModel.Tokens;
using System.Threading.Tasks;
using System.Web;

using Microsoft.IdentityModel.Protocols;
using Microsoft.Owin;
using Microsoft.Owin.Security;
using Microsoft.Owin.Security.Cookies;
using Microsoft.Owin.Security.Notifications;
using Microsoft.Owin.Security.OpenIdConnect;

using Owin;


[assembly: OwinStartup(typeof(dotnet_tutorial.App_Start.Startup))]

namespace dotnet_tutorial.App_Start
{
    public class Startup
    {
        public static string appId = ConfigurationManager.AppSettings["ida:AppId"];
        public static string appPassword = ConfigurationManager.AppSettings["ida:AppPassword"];
        public static string redirectUri = ConfigurationManager.AppSettings["ida:RedirectUri"];
        public static string[] scopes = ConfigurationManager.AppSettings["ida:AppScopes"]
          .Replace(' ', ',').Split(new char[] { ',' }, StringSplitOptions.RemoveEmptyEntries);
          
        public void Configuration(IAppBuilder app)
        {
            app.SetDefaultSignInAsAuthenticationType(CookieAuthenticationDefaults.AuthenticationType);

            app.UseCookieAuthentication(new CookieAuthenticationOptions());

            app.UseOpenIdConnectAuthentication(
              new OpenIdConnectAuthenticationOptions
              {
                  ClientId = appId,
                  Authority = "https://login.microsoftonline.com/common/v2.0",
                  Scope = "openid offline_access profile email " + string.Join(" ", scopes),
                  RedirectUri = redirectUri,
                  PostLogoutRedirectUri = "/",
                  TokenValidationParameters = new TokenValidationParameters
                  {
                      // For demo purposes only, see below
                      ValidateIssuer = false

                      // In a real multitenant app, you would add logic to determine whether the
                      // issuer was from an authorized tenant
                      //ValidateIssuer = true,
                      //IssuerValidator = (issuer, token, tvp) =>
                      //{
                      //  if (MyCustomTenantValidation(issuer))
                      //  {
                      //    return issuer;
                      //  }
                      //  else
                      //  {
                      //    throw new SecurityTokenInvalidIssuerException("Invalid issuer");
                      //  }
                      //}
                  },
                  Notifications = new OpenIdConnectAuthenticationNotifications
                  {
                      AuthenticationFailed = OnAuthenticationFailed,
                      AuthorizationCodeReceived = OnAuthorizationCodeReceived
                  }
              }
            );
        }

        private Task OnAuthenticationFailed(AuthenticationFailedNotification<OpenIdConnectMessage,
          OpenIdConnectAuthenticationOptions> notification)
        {
            notification.HandleResponse();
            string redirect = "/Home/Error?message=" + notification.Exception.Message;
            if (notification.ProtocolMessage != null && !string.IsNullOrEmpty(notification.ProtocolMessage.ErrorDescription))
            {
                redirect += "&debug=" + notification.ProtocolMessage.ErrorDescription;
            }
            notification.Response.Redirect(redirect);
            return Task.FromResult(0);
        }

        private Task OnAuthorizationCodeReceived(AuthorizationCodeReceivedNotification notification)
        {
            notification.HandleResponse();
            notification.Response.Redirect("/Home/Error?message=See Auth Code Below&debug=" + notification.Code);
            return Task.FromResult(0);
        }
    }
}
```

Let's continue by adding a `SignIn` action to the `HomeController` class. Open the `.\Controllers\HomeController.cs` file. At the top of the file, add the following lines:

```csharp
using System.Security.Claims;
using System.Threading.Tasks;
using Microsoft.Identity.Client;
using Microsoft.Owin.Security;
using Microsoft.Owin.Security.Cookies;
using Microsoft.Owin.Security.OpenIdConnect;
using Microsoft.Graph;
```

Now add a new method called `SignIn` to the `HomeController` class.

#### `SignIn` action in `./Controllers/HomeController.cs`

```csharp
public void SignIn()
{
    if (!Request.IsAuthenticated)
    {
        // Signal OWIN to send an authorization request to Azure
        HttpContext.GetOwinContext().Authentication.Challenge(
            new AuthenticationProperties { RedirectUri = "/" },
            OpenIdConnectAuthenticationDefaults.AuthenticationType);
    }
}
```

Finally, let's update the home page so that the login button invokes the `SignIn` action.

#### Updated contents of the `./Views/Home/Index.cshtml` file

```csharp
@{
    ViewBag.Title = "Home Page";
}

<div class="jumbotron">
    <h1>ASP.NET MVC Tutorial</h1>
    <p class="lead">This sample app uses the Mail API to read messages in your inbox.</p>
    <p><a href="@Url.Action("SignIn", "Home", null, Request.Url.Scheme)" class="btn btn-primary btn-lg">Click here to login</a></p>
</div>
```

Save your work and run the app. Click on the button to sign in. After signing in, you should be redirected to the error page, which displays an authorization code. Now let's do something with it.

### Exchanging the code for a token

Now let's update the `OnAuthorizationCodeReceived` function to retrieve a token. In `./App_Start/Startup.cs`, add the following line to the top of the file:

```csharp
using Microsoft.Identity.Client;
```

Replace the current `OnAuthorizationCodeReceived` function with the following.

#### Updated `OnAuthorizationCodeReceived` action in `./App_Start/Startup.cs`

```csharp
// Note the function signature is changed!
private async Task OnAuthorizationCodeReceived(AuthorizationCodeReceivedNotification notification)
{
    ConfidentialClientApplication cca = new ConfidentialClientApplication(
        appId, redirectUri, new ClientCredential(appPassword), null, null);

    string message;
    string debug;

    try
    {
        var result = await cca.AcquireTokenByAuthorizationCodeAsync(notification.Code, scopes);
        message = "See access token below";
        debug = result.AccessToken;
    }
    catch (MsalException ex)
    {
        message = "AcquireTokenByAuthorizationCodeAsync threw an exception";
        debug = ex.Message;
    }

    notification.HandleResponse();
    notification.Response.Redirect("/Home/Error?message=" + message + "&debug=" + debug);
}
```

Save your changes and restart the app. This time, after you sign in, you should see an access token displayed. Now that we can retrieve the access token, we need a way to save the token so the app can use it. The MSAL library defines a `TokenCache` interface, which we can use to store the tokens. Since this is a tutorial, we'll just implement a basic cache stored in the session.

Create a new folder in the project called `TokenStorage`. Add a class in this folder named `SessionTokenCache`, and replace the contents of that file with the following code (borrowed from https://github.com/Azure-Samples/active-directory-dotnet-webapp-openidconnect-v2).

#### Contents of the `./TokenCache/SessionTokenCache.cs` file

```csharp
using System.Threading;
using System.Web;

using Microsoft.Identity.Client;

namespace dotnet_tutorial.TokenStorage
{
    // Adapted from https://github.com/Azure-Samples/active-directory-dotnet-webapp-openidconnect-v2
    public class SessionTokenCache
    {
        private static ReaderWriterLockSlim sessionLock = new ReaderWriterLockSlim(LockRecursionPolicy.NoRecursion);
        string userId = string.Empty;
        string cacheId = string.Empty;
        HttpContextBase httpContext = null;

        TokenCache tokenCache = new TokenCache();

        public SessionTokenCache(string userId, HttpContextBase httpContext)
        {
            this.userId = userId;
            cacheId = userId + "_TokenCache";
            this.httpContext = httpContext;
            Load();
        }

        public TokenCache GetMsalCacheInstance()
        {
            tokenCache.SetBeforeAccess(BeforeAccessNotification);
            tokenCache.SetAfterAccess(AfterAccessNotification);
            Load();
            return tokenCache;
        }

        public bool HasData()
        {
            return (httpContext.Session[cacheId] != null && ((byte[])httpContext.Session[cacheId]).Length > 0);
        }

        public void Clear()
        {
            httpContext.Session.Remove(cacheId);
        }

        private void Load()
        {
            sessionLock.EnterReadLock();
            tokenCache.Deserialize((byte[])httpContext.Session[cacheId]);
            sessionLock.ExitReadLock();
        }

        private void Persist()
        {
            sessionLock.EnterReadLock();

            // Optimistically set HasStateChanged to false. 
            // We need to do it early to avoid losing changes made by a concurrent thread.
            tokenCache.HasStateChanged = false;

            httpContext.Session[cacheId] = tokenCache.Serialize();
            sessionLock.ExitReadLock();
        }

        // Triggered right before ADAL needs to access the cache. 
        private void BeforeAccessNotification(TokenCacheNotificationArgs args)
        {
            // Reload the cache from the persistent store in case it changed since the last access. 
            Load();
        }

        // Triggered right after ADAL accessed the cache.
        private void AfterAccessNotification(TokenCacheNotificationArgs args)
        {
            // if the access operation resulted in a cache update
            if (tokenCache.HasStateChanged)
            {
                Persist();
            }
        }
    }
}
```

Now let's update the `OnAuthorizationCodeReceived` function to use our new cache. 

Add the following lines to the top of the `Startup.cs` file:

```csharp
using dotnet_tutorial.TokenStorage;
```

Replace the current `OnAuthorizationCodeReceived` function with this one.

#### New `OnAuthorizationCodeReceived` in `./App_Start/Startup.cs`

```csharp
private async Task OnAuthorizationCodeReceived(AuthorizationCodeReceivedNotification notification)
{
    // Get the signed in user's id and create a token cache
    string signedInUserId = notification.AuthenticationTicket.Identity.FindFirst(ClaimTypes.NameIdentifier).Value;
    SessionTokenCache tokenCache = new SessionTokenCache(signedInUserId, 
        notification.OwinContext.Environment["System.Web.HttpContextBase"] as HttpContextBase);

    ConfidentialClientApplication cca = new ConfidentialClientApplication(
        appId, redirectUri, new ClientCredential(appPassword), tokenCache.GetMsalCacheInstance(), null);

    try
    {
        var result = await cca.AcquireTokenByAuthorizationCodeAsync(notification.Code, scopes);
    }
    catch (MsalException ex)
    {
        string message = "AcquireTokenByAuthorizationCodeAsync threw an exception";
        string debug = ex.Message;
        notification.HandleResponse();
        notification.Response.Redirect("/Home/Error?message=" + message + "&debug=" + debug);
    }
}
```

Since we're saving stuff in the session, let's also add a `SignOut` action to the `HomeController` class. Add the following lines to the top of the `HomeController.cs` file:

```csharp
using dotnet_tutorial.TokenStorage;
```

Then add the `SignOut` action.

#### `SignOut` action in `./Controllers/HomeController.cs`

```csharp
public void SignOut()
{
    if (Request.IsAuthenticated)
    {
        string userId = ClaimsPrincipal.Current.FindFirst(ClaimTypes.NameIdentifier).Value;

        if (!string.IsNullOrEmpty(userId))
        {
            // Get the user's token cache and clear it
            SessionTokenCache tokenCache = new SessionTokenCache(userId, HttpContext);
            tokenCache.Clear();
        }
    }
    // Send an OpenID Connect sign-out request. 
    HttpContext.GetOwinContext().Authentication.SignOut(
        CookieAuthenticationDefaults.AuthenticationType);
    Response.Redirect("/");
}
```

Now if you restart the app and sign in, you'll notice that the app redirects back to the home page, with no visible result. Let's update the home page so that it changes based on if the user is signed in or not.

First let's update the `Index` action in `HomeController.cs` to get the user's display name. Replace the existing `Index` function with the following code.

#### Updated `Index` action in `./Controllers/HomeController.cs`

```csharp
public ActionResult Index()
{
    if (Request.IsAuthenticated)
    {
        string userName = ClaimsPrincipal.Current.FindFirst("name").Value;
        string userId = ClaimsPrincipal.Current.FindFirst(ClaimTypes.NameIdentifier).Value;
        if (string.IsNullOrEmpty(userName) || string.IsNullOrEmpty(userId))
        {
            // Invalid principal, sign out
            return RedirectToAction("SignOut");
        }

        // Since we cache tokens in the session, if the server restarts
        // but the browser still has a cached cookie, we may be
        // authenticated but not have a valid token cache. Check for this
        // and force signout.
        SessionTokenCache tokenCache = new SessionTokenCache(userId, HttpContext);
        if (!tokenCache.HasData())
        {
            // Cache is empty, sign out
            return RedirectToAction("SignOut");
        }

        ViewBag.UserName = userName;
    }
    return View();
}
```

Next, replace the existing `Index.cshtml` code with the following.

#### Updated contents of `./Views/Home/Index.cshtml`

```csharp
@{
    ViewBag.Title = "Home Page";
}

<div class="jumbotron">
    <h1>ASP.NET MVC Tutorial</h1>
    <p class="lead">This sample app uses the Mail API to read messages in your inbox.</p>
    @if (Request.IsAuthenticated)
    {
        <p>Welcome, @(ViewBag.UserName)!</p>
    }
    else
    {
        <p><a href="@Url.Action("SignIn", "Home", null, Request.Url.Scheme)" class="btn btn-primary btn-lg">Click here to login</a></p>
    }
</div>
```

Finally, let's add some new menu items to the navigation bar if the user is signed in. Open the `./Views/Shared/_Layout.cshtml` file. Locate the following lines of code:

```csharp
<ul class="nav navbar-nav">
    <li>@Html.ActionLink("Home", "Index", "Home")</li>
    <li>@Html.ActionLink("About", "About", "Home")</li>
    <li>@Html.ActionLink("Contact", "Contact", "Home")</li>
</ul>
```

Replace those lines with this code:

```csharp
<ul class="nav navbar-nav">
    <li>@Html.ActionLink("Home", "Index", "Home")</li>
    @if (Request.IsAuthenticated)
    {
        <li>@Html.ActionLink("Inbox", "Inbox", "Home")</li>
        <li>@Html.ActionLink("Sign Out", "SignOut", "Home")</li>
    }
</ul>
```

Now if you save your changes and restart the app, the home page should display the logged on user's name and show two new menu items in the top navigation bar after you sign in. Now that we've got the signed in user and an access token, we're ready to call the Mail API.

## Using the Mail API

Let's start by adding a new function to the `HomeController` class to get the user's access token. In this function we'll use MSAL and our token cache. If there is a valid non-expired token in the cache, MSAL will return it. If it is expired, MSAL will refresh the token for us.

Add the following `using` statements to `./Controllers/HomeController.cs`.

```csharp
using System.Configuration;
using System.Net.Http.Headers;
```

#### `GetAccessToken` function in `./Controllers/HomeController.cs`

```csharp
public async Task<string> GetAccessToken()
{
    string accessToken = null;

    // Load the app config from web.config
    string appId = ConfigurationManager.AppSettings["ida:AppId"];
    string appPassword = ConfigurationManager.AppSettings["ida:AppPassword"];
    string redirectUri = ConfigurationManager.AppSettings["ida:RedirectUri"];
    string[] scopes = ConfigurationManager.AppSettings["ida:AppScopes"]
        .Replace(' ', ',').Split(new char[] { ',' }, StringSplitOptions.RemoveEmptyEntries);

    // Get the current user's ID
    string userId = ClaimsPrincipal.Current.FindFirst(ClaimTypes.NameIdentifier).Value;

    if (!string.IsNullOrEmpty(userId))
    {
        // Get the user's token cache
        SessionTokenCache tokenCache = new SessionTokenCache(userId, HttpContext);

        ConfidentialClientApplication cca = new ConfidentialClientApplication(
            appId, redirectUri, new ClientCredential(appPassword), tokenCache.GetMsalCacheInstance(), null);

        // Call AcquireTokenSilentAsync, which will return the cached
        // access token if it has not expired. If it has expired, it will
        // handle using the refresh token to get a new one.
        AuthenticationResult result = await cca.AcquireTokenSilentAsync(scopes, cca.Users.First());

        accessToken = result.AccessToken;
    }

    return accessToken;
}
```

Now let's test our new function. Add a new function to the `HomeController` class called `Inbox`.

#### `Inbox` action in `./Controllers/HomeController.cs`

```csharp
public async Task<ActionResult> Inbox()
{
    string token = await GetAccessToken();
    if (string.IsNullOrEmpty(token))
    {
        // If there's no token in the session, redirect to Home
        return Redirect("/");
    }

    return Content(string.Format("Token: {0}", token));
}
```

If you run the app now and click the **Inbox** menu item after logging in, you should see the access token displayed in the browser. Let's put it to use.

Update the `Inbox` function with the following code.

#### Updated `Inbox` action in `./Controllers/HomeController.cs`

```csharp
public async Task<ActionResult> Inbox()
{
    string token = await GetAccessToken();
    if (string.IsNullOrEmpty(token))
    {
        // If there's no token in the session, redirect to Home
        return Redirect("/");
    }

    GraphServiceClient client = new GraphServiceClient(
        new DelegateAuthenticationProvider(
            (requestMessage) =>
            {
                requestMessage.Headers.Authorization =
                    new AuthenticationHeaderValue("Bearer", token);

                return Task.FromResult(0);
            }));

    try
    {
        var mailResults = await client.Me.MailFolders.Inbox.Messages.Request()
                            .OrderBy("receivedDateTime DESC")
                            .Select(m => new { m.Subject, m.ReceivedDateTime, m.From})
                            .Top(10)
                            .GetAsync();

        string content = "";

        foreach (var msg in mailResults.CurrentPage)
        {
            content += string.Format("Subject: {0}<br/>", msg.Subject);
        }

        return Content(content);
    }
    catch (ServiceException ex)
    {
        return RedirectToAction("Error", "Home", new { message = "ERROR retrieving messages", debug = ex.Message });
    }
}
```

To summarize the new code in the `Inbox` function:

- It creates a `GraphServiceClient` object.
- It modifies the outgoing request's headers in the `DelegateAuthenticationProvider` by adding the access token as an `Authorization` header.
- It issues a GET request to the URL for inbox messages, with the following characteristics:
    - It uses the `OrderBy()` function with a value of `receivedDateTime DESC` to sort the results by ReceivedDateTime.
    - It uses the `Select()` function with a LINQ expression to limit the fields returned to only those that we need.
- It loops over the results and prints out the subject.

If you restart the app now, you should get a very basic listing of email subjects. However, we can use the features of MVC to do better than that.

## Displaying the results

MVC can generate views based on a model. So let's start by creating a view based on the `Microsoft.Graph.Message` object. In Solution Explorer, right-click the `./Views/Home` folder and choose **Add**, then **View**. Enter `Inbox` for the **View name**. Change the **Template** field to `Empty (without model)`. Leave everything else as default values and click **Add**.

![The Add View dialog.](images/dotnet-tutorial/add-view.PNG)

Open the `Inbox.cshtml` file that gets created and replace the entire contents of the file with the following code.

```csharp
@model IEnumerable<Microsoft.Graph.Message>

@{
    ViewBag.Title = "Inbox";
}

<h2>Inbox</h2>

<table class="table">
    <tr>
        <th>
            Subject
        </th>
        <th>
            Received
        </th>
        <th>
            From
        </th>
    </tr>

    @foreach (var item in Model)
    {
        <tr>
            <td>
                @Html.DisplayFor(modelItem => item.Subject)
            </td>
            <td>
                @Html.DisplayFor(modelItem => item.ReceivedDateTime)
            </td>
            <td>
                @Html.DisplayFor(modelItem => item.From.EmailAddress.Name) &lt;@Html.DisplayFor(modelItem => item.From.EmailAddress.Address)&gt;
            </td>
        </tr>
    }

</table>
```

At this point Visual Studio will indicate a problem with `item.ReceivedDateTime`. We need to add a reference to `System.Runtime` in the `Web.config` file. Open `Web.config` and find the line:

```xml
<compilation debug="true" targetFramework="4.5" />
```

Replace that line with the following:

```xml
<compilation debug="true" targetFramework="4.5">
  <assemblies>
    <add assembly="System.Runtime, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" />
  </assemblies>
</compilation>
```

Just one more thing to do. Let's update the `Inbox` function to use our new view.

#### Updated `Inbox` action in `./Controllers/HomeController.cs`

```csharp
public async Task<ActionResult> Inbox()
{
    string token = await GetAccessToken();
    if (string.IsNullOrEmpty(token))
    {
        // If there's no token in the session, redirect to Home
        return Redirect("/");
    }

    GraphServiceClient client = new GraphServiceClient(
        new DelegateAuthenticationProvider(
            (requestMessage) =>
            {
                requestMessage.Headers.Authorization =
                    new AuthenticationHeaderValue("Bearer", token);

                return Task.FromResult(0);
            }));

    try
    {
        var mailResults = await client.Me.MailFolders.Inbox.Messages.Request()
                            .OrderBy("receivedDateTime DESC")
                            .Select("subject,receivedDateTime,from")
                            .Top(10)
                            .GetAsync();

        return View(mailResults.CurrentPage);
    }
    catch (ServiceException ex)
    {
        return RedirectToAction("Error", "Home", new { message = "ERROR retrieving messages", debug = ex.Message });
    }
}
```

The changes here are minimal. Instead of building a string with the results, we instead pass the `mailResults.CurrentPage` collection to the `View` method.

Save your changes and run the app. You should now get a list of messages that looks something like this.

![The sample app displaying a user's inbox.](images/dotnet-tutorial/inbox-display.PNG)

## Adding Calendar and Contacts APIs

Now that you've mastered calling the Outlook Mail API, doing the same for Calendar and Contacts APIs is similar and easy.

> [!TIP]
> If you've followed along with the tutorial, you probably have an access token saved in your session. That token will only be valid for the `Mail.Read` scope. In order to call the Calendar or Contacts API, we will need to add new scopes. Be sure sign out of the app to get rid of the saved tokens so that you can start the login process from the beginning to get a new access token.

### For Calendar API:

1. Update the `ida:AppScopes` value in `AzureOauth.config` to include the `Calendars.Read` scope.

    ```xml
    <add key="ida:AppScopes" value="User.Read Mail.Read Calendars.Read" />
    ```

1. Add a view for events called `Calendar` that uses the `Empty (without model)` template. Add the following code to `Calendar.cshtml`:

    ```csharp
    @model IEnumerable<Microsoft.Graph.Event>

    @{
        ViewBag.Title = "Calendar";
    }

    <h2>Calendar</h2>

    <table class="table">
        <tr>
            <th>
                Subject
            </th>
            <th>
                Start
            </th>
            <th>
                End
            </th>
        </tr>

    @foreach (var item in Model) {
        <tr>
            <td>
                @Html.DisplayFor(modelItem => item.Subject)
            </td>
            <td>
                @Html.DisplayFor(modelItem => item.Start.DateTime) (@Html.DisplayFor(modelItem => item.Start.TimeZone))
            </td>
            <td>
                @Html.DisplayFor(modelItem => item.End.DateTime) (@Html.DisplayFor(modelItem => item.End.TimeZone))
            </td>
        </tr>
    }

    </table>
    ```

1. Add a navigation item for the Calendar view to `./Views/Shared/_Layout.cshtml`.

    ```csharp
    @if (Request.IsAuthenticated)
    {
        <li>@Html.ActionLink("Inbox", "Inbox", "Home")</li>
        <li>@Html.ActionLink("Calendar", "Calendar", "Home")</li>
        <li>@Html.ActionLink("Sign Out", "SignOut", "Home")</li>
    }
    ```

1. Add a `Calendar` function to the `HomeController` class.

    ```csharp
    public async Task<ActionResult> Calendar()
    {
        string token = await GetAccessToken();
        if (string.IsNullOrEmpty(token))
        {
            // If there's no token in the session, redirect to Home
            return Redirect("/");
        }

        GraphServiceClient client = new GraphServiceClient(
            new DelegateAuthenticationProvider(
                (requestMessage) =>
                {
                    requestMessage.Headers.Authorization =
                        new AuthenticationHeaderValue("Bearer", token);

                    return Task.FromResult(0);
                }));

        try
        {
            var eventResults = await client.Me.Events.Request()
                                .OrderBy("start/dateTime DESC")
                                .Select("subject,start,end")
                                .Top(10)
                                .GetAsync();

            return View(eventResults.CurrentPage);
        }
        catch (ServiceException ex)
        {
            return RedirectToAction("Error", "Home", new { message = "ERROR retrieving events", debug = ex.Message });
        }
    }
    ```

1. After logging into the app, click the **Calendar** item in the top navigation menu.

### For Contacts API:

1. Update the `ida:AppScopes` value in `AzureOauth.config` to include the `Contacts.Read` scope.

    ```xml
    <add key="ida:AppScopes" value="User.Read Mail.Read Contacts.Read" />
    ```

1. Add a view for events called `Contacts` that uses the `Empty (without model)` template. Add the following code to `Contacts.cshtml`:

    ```csharp
    @model IEnumerable<Microsoft.Graph.Contact>

    @{
        ViewBag.Title = "Contacts";
    }

    <h2>Contacts</h2>

    <table class="table">
        <tr>
            <th>
                DisplayName
            </th>
            <th>
                Email Address
            </th>
            <th>
                Mobile Phone
            </th>
        </tr>

    @foreach (var item in Model) {
        <tr>
            <td>
                @Html.DisplayFor(modelItem => item.DisplayName)
            </td>
            <td>
                @Html.Partial("EmailAddresses", item.EmailAddresses)
            </td>
            <td>
                @Html.DisplayFor(modelItem => item.MobilePhone)
            </td>
        </tr>
    }

    </table>
    ```

1. Right-click the **Views\Shared** folder, choose **Add** then **View...**. Be sure to select the **Create as a partial view** option. Add  view for email addresses called `EmailAddresses` that uses the `Empty (without model)` template. Add the following code to `EmailAddresses.cshtml`:

    ```csharp
    @model IEnumerable<Microsoft.Graph.EmailAddress>

    @foreach(var item in Model)
    {
        <p>@Html.DisplayFor(modelItem => item.Address)</p>
    }
    ```

1. Add a navigation item for the Contacts view to `./Views/Shared/_Layout.cshtml`.

    ```csharp
    @if (Request.IsAuthenticated)
    {
        <li>@Html.ActionLink("Inbox", "Inbox", "Home")</li>
        <li>@Html.ActionLink("Contacts", "Contacts", "Home")</li>
        <li>@Html.ActionLink("Sign Out", "SignOut", "Home")</li>
    }
    ```

1. Add a `Contacts` function to the `HomeController` class.

    ```csharp
    public async Task<ActionResult> Contacts()
    {
        string token = await GetAccessToken();
        if (string.IsNullOrEmpty(token))
        {
            // If there's no token in the session, redirect to Home
            return Redirect("/");
        }

        GraphServiceClient client = new GraphServiceClient(
            new DelegateAuthenticationProvider(
                (requestMessage) =>
                {
                    requestMessage.Headers.Authorization =
                        new AuthenticationHeaderValue("Bearer", token);

                    return Task.FromResult(0);
                }));

        try
        {
            var contactResults = await client.Me.Contacts.Request()
                                .OrderBy("displayName")
                                .Select("displayName,emailAddresses,mobilePhone")
                                .Top(10)
                                .GetAsync();

            return View(contactResults.CurrentPage);
        }
        catch (ServiceException ex)
        {
            return RedirectToAction("Error", "Home", new { message = "ERROR retrieving contacts", debug = ex.Message });
        }
    }
    ```

1. After logging into the app, click the **Contacts** item in the top navigation menu.
