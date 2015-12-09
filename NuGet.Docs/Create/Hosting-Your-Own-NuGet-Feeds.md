# Hosting Your Own NuGet Feeds

Some companies restrict which third-party libraries their developers may use.
Therefore, they might not want developers to have access to everything in the official
NuGet feed, or they might have a set of proprietary libraries they want to make available
in addition to the official feed.

In these scenarios, you can set up a custom NuGet feed, and you can configure
Visual Studio to offer that feed instead of or in addition to the official feed.
A feed can be local (a folder on the local machine or a network folder), or remote
(an intranet or internet URL).

There are several third-party NuGet Servers available that make remote private feeds easy
to configure and set-up, including [MyGet](http://myget.org), 
[Inedo's ProGet](http://inedo.com/proget), 
[JFrog's Artifactory](http://www.jfrog.com/home/v_artifactorypro_overview),
[NuGet Server](http://nugetserver.net/), and 
[Sonatype's Nexus](http://www.sonatype.org/nexus/). See
[An Overview of the NuGet Ecosystem](/Contribute/Ecosystem) to learn more about these 
options. 

Otherwise, you can create a local feed on disk or build your own remote feed using NuGet's 
server components by following the instructions below.


## Creating Local Feeds

Begin by creating or getting the packages you want to include in the custom feed
and then putting them all into a folder. In the following example, a folder has
been created in the local *c:* drive.
The folder contains a single package (.nupkg file).

![LocalNuGetFeed-folder.png](/images/create/LocalNuGetFeed-folder.png)

Next, specify that folder as the location of a NuGet feed. In Visual Studio,
from the **Tools** menu select **Library Package Manager** and then click
**Package Manager Settings**.

![Package Manager Settings in menu](/images/create/Package-Manager-Settings-in-menu.png)

The **Package Sources** under **Package Manager** node in **Options** dialog box is displayed.

![Options dialog box](/images/create/Package-Sources.png)

In the **Name** box, enter a name for your feed.
In the **Source** box enter the path of your packages folder.

![Available Package Sources dialog box without new feed](/images/create/Package-Sources-With-Custom-Feed.png)

Click **Update**. Your local folder is now another NuGet feed source.

To install a package using the new feed, in the **Package Manager Console** window,
select the new feed in the **Package source** list.

![Selecting local feed in Package Manager Console](/images/create/Selecting-local-feed-in-Package-Manager-Console.png)

You can also select the new feed in the **Online** tab of the
**Manage NuGet Packages** dialog box.

![Selecting local feed in the Manage NuGet Packages dialog](/images/create/Selecting-local-feed-in-Add-Library-Package-Reference.png)

## Creating Remote Feeds

You can also host a remote (or internal) feed on a server that runs IIS. There are two alternatives from the NuGet team here
1. NuGet.Server
2. NuGet Gallery

For relatively small projects with a small set of packages go with NuGet.Server, it is basically a view of a network share or local folder through http, and as such is easy to setup and works quite well when the number of packages is relatively small.

<p class="info">
<strong>Note</strong><br />If the package count is high (> 10.000), have a look at <a href="https://github.com/NuGet/NuGetGallery/wiki/Hosting-the-NuGet-Gallery-Locally-in-IIS">the NuGet Gallery Project</a>. It is more complex to set up and host, but offers a lot more nuget.org like features. Other alternatives are available as well - check <a href="/Contribute/Ecosystem">the Overview of the NuGet Ecosystem</a> to learn more about these options. 
</p> 

Below are the instruction to host a NuGet.Server, the instructions for setting up NuGet Gallery are in the link above.

### Step 1: Create a new Empty Web Application in Visual Studio

Go to the **File** | **New** | **Project** menu option (or just hit CTRL + SHIFT + N)
which will bring up the new project dialog. Select **ASP.NET Web Application** as in the following screenshot. Please note that the steps here including NuGet.Server package are intended for use with C# project only.

![New Project dialog box](/images/create/New-Project-dialog-box.png)

Next, select the **ASP.NET 4.5 - Empty** template and create the project. This results in a very empty project.

![New project in Solution Explorer](/images/create/New-project-in-Solution-Explorer.png)

### Step 2: Install the NuGet.Server Package

Now right click on the project node and select **Manage NuGet Packages** to launch
the NuGet dialog (alternatively, you can use the Package Manager Console instead and
type `Install-Package NuGet.Server`).

Click the **Browse** tab and then type **NuGet.Server** in the top right search box.
Click **Install** on the **NuGet.Server** package as shown in the following image.

![NuGet.Server package](/images/create/NuGet.Server-package.png)

### Step 3: Configure the Packages folder

Starting with `NuGet.Server` 1.5, you can configure the folder which contains your packages. The web.config file contains a new `appSetting`, named `packagesPath`. When the key is omitted or left blank, the packages folder is the default ~/Packages. You can specify an **absolute path**, or a **virtual path**.

    <appSettings>
        <!-- Set the value here to specify your custom packages folder. -->
        <add key="packagesPath" value="C:\MyPackages" />
    </appSettings>


### Step 4: Add Packages to the Packages folder

That's it! The **NuGet.Server** package just converted your empty website into a site that's ready to serve up the OData package feed. Just add packages into the Packages folder and they'll show up.

In the following screenshot, you can see that a few packages that have been added manually to the default **Packages**
folder.  

![Adding packages to the packages folder](/images/create/Adding-packages-to-the-packages-folder.png)

<p class="info">If you want these packages to be published (such as when selecting Build -> Publish from
the application menu) you'll also need to include the .nupkg files in Solution Explorer
and change the Build Action property to "Content".</p>

You can also add and delete packages to the lightweight feed using 
NuGet.exe. After installing the package, the web.config file will contain a new `appSetting`, named 
`apiKey`. When the key is omitted or left blank, pushing packages to the feed is disabled. Setting the 
apiKey to a value (ideally a strong password) enables pushing packages using NuGet.exe.

    <appSettings>
         <!--
            Determines if an Api Key is required to push\delete packages from the server. 
        -->
        <add key="requireApiKey" value="true" />

        <!-- Set the value here to allow people to push/delete packages from the server.
             NOTE: This is a shared key (password) for all users. -->
        <add key="apiKey" value="" />
    </appSettings>

If however your server is already secured and \ or you do not require an api key to perform this operation, 
set the **requireApiKey** value to false.

### Step 5: Deploy and run your brand new Package Feed!

Hit CTRL + F5 to run the site and it'll provide some instructions on what to do next.

![Package feed home page](/images/create/Package-feed-home-page.png)

Clicking on "here" shows the OData over ATOM feed of packages.

![OData over ATOM package feed](/images/create/OData-over-ATOM-package-feed.png)

<p class="info">
<strong>Note</strong><br />After the first load of the feed, the Packages folder is restructured:<br />
<img src="/images/create/Adding-packages-to-the-packages-folder-expanded.png" title="Adding packages to the packages folder (expanded)" /><br /><br />
NuGet.Server uses the <a href="http://blog.nuget.org/20151118/nuget-3.3.html#folder-based-repository-commands">local storage layout introduced with NuGet 3.3</a> to store packages and will always copy packages that are added to the Packages folder manually to a subfolder that follows this new storage layout.
</p> 

Now all we need to do is deploy this website as we would any other site and then
we can click NuGet's Settings button and add this feed to my set of package sources.

Note that the URL you need to put in is <a href="http://yourdomain/nuget/">http://yourdomain/nuget/</a> depending on how you deploy the site.

Yes, it's that easy! An alternative way of publishing packages to this server is by simply placing the nupkg under the 
the **Packages** folder and they are automatically syndicated.