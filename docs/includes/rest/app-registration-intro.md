> [!IMPORTANT]
> New app registrations should be created and managed in the new [Application Registration Portal](https://apps.dev.microsoft.com/) to be compatible with Outlook.com. Only create new app registrations in the [Azure Management Portal](https://manage.windowsazure.com/) if your app:
> 
> - Uses the OAuth2 Client Credentials Grant Flow, or
> - Needs to access other Office 365 workloads besides Outlook (such as OneDrive for Business or SharePoint)
> 
> Bear in mind that apps registered using the Azure Management Portal will not be compatible with Outlook.com, and will not be able to dynamically request permissions scopes.
> Existing app registrations that were created in the the Azure Management Portal will continue to work for Office 365 only. These registrations do not show up in the Application Registration Portal and must be managed in the Azure Management Portal.
>
> ### Account requirements
> 
> In order to use the Application Registration Portal, you need either an Office 365 work or school account, or a Microsoft account. If you don't have either of these, you have a number of options:
> - Sign up for a new Microsoft account [here](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1). 
> - You can obtain an Office 365 subscription in a couple of different ways: 
>     - You can get a free one-year Office 365 Developer subscription by signing up for the [Office Developer program](http://dev.office.com/devprogram).
>     - You can signup for a [25-user free trial](https://portal.office.com/Signup/Signup.aspx?OfferId=467eab54-127b-42d3-b046-3844b860bebf&dl=O365_BUSINESS_PREMIUM&alo=1&lc=1033&ali=1#0) of the Office 365 Business subscription.
> 
> ### REST API availability
> 
> The REST API is currently enabled on all Office 365 accounts that have Exchange Online, and all Outlook.com accounts.