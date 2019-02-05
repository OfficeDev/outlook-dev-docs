---
title: Build an Outlook add-in
description: Learn how to build a simple Outlook add-in by using jQuery and the Office JS API and test it locally.
ms.topic: get-started-article
ms.date: 12/27/2018
localization_priority: Priority
---

# Build your first Outlook add-in

In this article, you'll walk through the process of building an Outlook add-in by using jQuery and the Office JS API.

## Create the add-in

You can create an Office Add-in by using Visual Studio or any other editor. Tell us what editor you'd like to use by choosing one of the following tabs:

# [Visual Studio](#tab/visual-studio)

### Prerequisites

- [Visual Studio 2017](https://www.visualstudio.com/vs/) with the **Office/SharePoint development** workload installed

    > [!NOTE]
    > If you've previously installed Visual Studio 2017, [use the Visual Studio Installer](https://docs.microsoft.com/visualstudio/install/modify-visual-studio) to ensure that the **Office/SharePoint development** workload is installed.

- Office 365

    > [!NOTE]
    > If you do not have an Office 365 subscription, you can get a free one by signing up for the [Office 365 developer program](https://developer.microsoft.com/office/dev-program).

### Create the add-in project

1. On the Visual Studio menu bar, choose **File** > **New** > **Project**.

1. In the list of project types under **Visual C#** or **Visual Basic**, expand **Office/SharePoint**, choose **Add-ins**, and then choose **Outlook Web Add-in** as the project type.

1. Name the project, and then choose **OK**.

1. Visual Studio creates a solution and its two projects appear in **Solution Explorer**. The **MessageRead.html** file opens in Visual Studio.

### Explore the Visual Studio solution

When you've completed the wizard, Visual Studio creates a solution that contains two projects.

|**Project**|**Description**|
|:-----|:-----|
|Add-in project|Contains only an XML manifest file, which contains all the settings that describe your add-in. These settings help the Office host determine when your add-in should be activated and where the add-in should appear. Visual Studio generates the contents of this file for you so that you can run the project and use your add-in immediately. You can change these settings any time by modifying the XML file.|
|Web application project|Contains the content pages of your add-in, including all the files and file references that you need to develop Office-aware HTML and JavaScript pages. While you develop your add-in, Visual Studio hosts the web application on your local IIS server. When you're ready to publish the add-in, you'll need to deploy this web application project to a web server.|

### Update the code

1. **MessageRead.html** specifies the HTML that will be rendered in the add-in's task pane. In **MessageRead.html**, replace the `<body>` element with the following markup and save the file.
 
    ```HTML
    <body class="ms-font-m ms-welcome">
        <div class="ms-Fabric content-main">
            <h1 class="ms-font-xxl">Message properties</h1>
            <table class="ms-Table ms-Table--selectable">
                <thead>
                    <tr>
                        <th>Property</th>
                        <th>Value</th>
                    </tr>
                </thead>
                <tbody>
                    <tr>
                        <td><strong>Id</strong></td>
                        <td class="prop-val"><code><label id="item-id"></label></code></td>
                    </tr>
                    <tr>
                        <td><strong>Subject</strong></td>
                        <td class="prop-val"><code><label id="item-subject"></label></code></td>
                    </tr>
                    <tr>
                        <td><strong>Message Id</strong></td>
                        <td class="prop-val"><code><label id="item-internetMessageId"></label></code></td>
                    </tr>
                    <tr>
                        <td><strong>From</strong></td>
                        <td class="prop-val"><code><label id="item-from"></label></code></td>
                    </tr>
                </tbody>
            </table>
        </div>
    </body>
    ```

1. Open the file **MessageRead.js** in the root of the web application project. This file specifies the script for the add-in. Replace the entire contents with the following code and save the file.

    ```js
    'use strict';

    (function () {

        Office.onReady(function () {
            // Office is ready
            $(document).ready(function () {
                // The document is ready
                loadItemProps(Office.context.mailbox.item);
            });
        });

        function loadItemProps(item) {
            // Write message property values to the task pane
            $('#item-id').text(item.itemId);
            $('#item-subject').text(item.subject);
            $('#item-internetMessageId').text(item.internetMessageId);
            $('#item-from').html(item.from.displayName + " &lt;" + item.from.emailAddress + "&gt;");
        }
    })();
    ```

1. Open the file **MessageRead.css** in the root of the web application project. This file specifies the custom styles for the add-in. Replace the entire contents with the following code and save the file.

    ```CSS
    html,
    body {
        width: 100%;
        height: 100%;
        margin: 0;
        padding: 0;
    }

    td.prop-val {
        word-break: break-all;
    }

    .content-main {
        margin: 10px;
    }
    ```

### Update the manifest

1. Open the XML manifest file in the Add-in project. This file defines the add-in's settings and capabilities.

1. The `ProviderName` element has a placeholder value. Replace it with your name.

1. The `DefaultValue` attribute of the `DisplayName` element has a placeholder. Replace it with `My Office Add-in`.

1. The `DefaultValue` attribute of the `Description` element has a placeholder. Replace it with `My First Outlook add-in`.

1. Save the file.

    ```xml
    ...
    <ProviderName>Jason Johnston</ProviderName>
    <DefaultLocale>en-US</DefaultLocale>
    <!-- The display name of your add-in. Used on the store and various places of the Office UI such as the add-ins dialog. -->
    <DisplayName DefaultValue="My Office Add-in" />
    <Description DefaultValue="My First Outlook add-in"/>
    ...
    ```

### Try it out

1. Using Visual Studio, test the newly created Outlook add-in by pressing F5 or choosing the **Start** button. The add-in will be hosted locally on IIS.

1. In the **Connect to Exchange email account** dialog box, enter the email address and password for your [Microsoft account](https://account.microsoft.com/account) and then choose **Connect**. When the Outlook.com login page opens in a browser, login to your email account with the same credentials as you entered previously.

    > [!NOTE]
    > If the **Connect to Exchange email account** dialog box repeatedly prompts you to login, Basic Auth may be disabled for accounts on your Office 365 tenant. To test this add-in, login using a [Microsoft account](https://account.microsoft.com/account) instead.

1. In Outlook on the web, select or open a message.

1. Within the message, locate the add-in's button.

    ![A screenshot of a message window in Outlook on the web with the add-in button highlighted](images/quick-start-button-owa.png)

1. Click the button to open the add-in's task pane.

    ![A screenshot of the add-in's task pane in Outlook on the web displaying message properties](images/quick-start-task-pane-owa.png)

    > [!NOTE]
    > If the task pane doesn't load, try to verify by opening it in a browser on the same machine.

# [Any editor](#tab/visual-studio-code)

### Prerequisites

- [Node.js](https://nodejs.org)

- Install the latest version of [Yeoman](https://github.com/yeoman/yo) and the [Yeoman generator for Office Add-ins](https://github.com/OfficeDev/generator-office) globally.

    ```powershell
    npm install -g yo generator-office
    ```

### Create the add-in project

1. Use the Yeoman generator to create an Outlook add-in project. Run the following command and then answer the prompts as follows:

    ```powershell
    yo office
    ```

    - **Choose a project type** - `Office Add-in project using Jquery framework`

    - **Choose a script type** - `Javascript`

    - **What do you want to name your add-in?** - `My Office Add-in`

    - **Which Office client application would you like to support?** - `Outlook`

    ![A screenshot of the prompts and answers for the Yeoman generator](images/quick-start-yo-prompts.png)
	
    After you complete the wizard, the generator will create the project and install supporting Node components.

1. Navigate to the root folder of the web application project.

    ```bash
    cd "My Office Add-in"
    ```

### Update the code

1. In your code editor, open **index.html** in the root of the project. This files contains the HTML that will be rendered in the add-in's task pane.

1. Replace the `<header>`, `<section>`, and `<main>` elements inside the `<body>` element with the following markup and save the file.

    ```HTML
    <div class="ms-Fabric content-main">
        <h1 class="ms-font-xxl">Message properties</h1>
        <table class="ms-Table ms-Table--selectable">
            <thead>
                <tr>
                    <th>Property</th>
                    <th>Value</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td><strong>Id</strong></td>
                    <td class="prop-val"><code><label id="item-id"></label></code></td>
                </tr>
                <tr>
                    <td><strong>Subject</strong></td>
                    <td class="prop-val"><code><label id="item-subject"></label></code></td>
                </tr>
                <tr>
                    <td><strong>Message Id</strong></td>
                    <td class="prop-val"><code><label id="item-internetMessageId"></label></code></td>
                </tr>
                <tr>
                    <td><strong>From</strong></td>
                    <td class="prop-val"><code><label id="item-from"></label></code></td>
                </tr>
            </tbody>
        </table>
    </div>
    ```

1. Open the file **src/index.js** to specify the script for the add-in. Replace the entire contents with the following code and save the file.

    ```js
    'use strict';

    (function () {

        Office.onReady(function () {
            // Office is ready
            $(document).ready(function () {
                // The document is ready
                loadItemProps(Office.context.mailbox.item);
            });
        });

        function loadItemProps(item) {
            // Write message property values to the task pane
            $('#item-id').text(item.itemId);
            $('#item-subject').text(item.subject);
            $('#item-internetMessageId').text(item.internetMessageId);
            $('#item-from').html(item.from.displayName + " &lt;" + item.from.emailAddress + "&gt;");
        }
    })();
    ```

1. Open **app.css** in the root of the project, replace the entire contents with the following code, and save the file.

    ```CSS
    html,
    body {
        width: 100%;
        height: 100%;
        margin: 0;
        padding: 0;
    }

    td.prop-val {
        word-break: break-all;
    }

    .content-main {
        margin: 10px;
    }
    ```

### Update the manifest

1. Open the **manifest.xml** file.

1. The `ProviderName` element has a placeholder value. Replace it with your name.

1. The `DefaultValue` attribute of the `Description` element has a placeholder. Replace it with `My First Outlook add-in`.

1. The `DefaultValue` attribute of the `SupportUrl` element has a placeholder. Replace it with `https://localhost:3000` and save the file.

    ```xml
    ...
    <ProviderName>Jason Johnston</ProviderName>
    <DefaultLocale>en-US</DefaultLocale>
    <!-- The display name of your add-in. Used on the store and various places of the Office UI such as the add-ins dialog. -->
    <DisplayName DefaultValue="My Office Add-in" />
    <Description DefaultValue="My First Outlook add-in"/>

    <!-- Icon for your add-in. Used on installation screens and the add-ins dialog. -->
    <IconUrl DefaultValue="https://localhost:3000/assets/icon-32.png" />
    <HighResolutionIconUrl DefaultValue="https://localhost:3000/assets/hi-res-icon.png"/>

    <!--If you plan to submit this add-in to the Office Store, uncomment the SupportUrl element below-->
    <SupportUrl DefaultValue="https://localhost:3000" />
    ...
    ```

### Start the dev server

1. Open a command prompt in the root directory of your project (**[...]/My Office Add-in**) and run the following command to start the web server at `https://localhost:3000`.

    ```bash
    npm start
    ```

1. Open either Internet Explorer or Microsoft Edge and navigate to `https://localhost:3000`. If the page loads without any certificate errors, proceed to the next section in this article (**Try it out**). If your browser indicates that the site's certificate is not trusted, proceed to the following step.

1. If your browser indicates that the site's certificate is not trusted, you will need to add the certificate as a trusted certificate. Outlook will not load add-ins if the site is not trusted. See [Adding Self-Signed Certificates as Trusted Root Certificate](https://github.com/OfficeDev/generator-office/blob/master/src/docs/ssl.md) for details.

    > [!NOTE]
    > Chrome (web browser) may continue to indicate the site's certificate is not trusted, even after you have completed the process described in [Adding Self-Signed Certificates as Trusted Root Certificate](https://github.com/OfficeDev/generator-office/blob/master/src/docs/ssl.md). Therefore, you should use either Internet Explorer or Microsoft Edge to verify that the certificate is trusted. 

### Try it out

1. Follow the instructions in [Sideload Outlook add-ins for testing](sideload-outlook-add-ins-for-testing.md) to sideload the add-in in Outlook.

1. In Outlook, select or open a message.

1. On the **Home** tab (**Message** tab if you opened the message in a new window), locate the add-in's **Display all properties** button.

    ![A screenshot of a message window in Outlook with the add-in button highlighted](images/quick-start-button.png)

1. Click the button to open the add-in's task pane.

    ![A screenshot of the add-in's task pane displaying message properties](images/quick-start-task-pane.png)

---

## Next steps

Congratulations, you've successfully created your first Outlook add-in! Next, learn more about the capabilities of an Outlook add-in and build a more complex add-in by following along with the Advanced Outlook add-in tutorial.

> [!div class="nextstepaction"]
> [Advanced Outlook add-in tutorial](addin-tutorial.md)
