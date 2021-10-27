# Infrastructure
The infrastructure of project OPIDDaily refers to the tools and technologies used to develop it, exclusive of the implementation itself. This
section will be useful to a developer wanting to maintain and further develop OPIDDaily. It describes both the desktop development environment and the
AppHarbor deployment environment for the web application OPIDDaily.

## Hosting Environments
There are 4 hosting environments for OPIDDaily: desktop, training, staging and production. They differ in the database connection string used by each.
The connection string is configured as the value of variable **OpidDailyConnectionString** in the `<connectionStrings>` section of
Web.config. The static value configured there is used by the desktop environment. The static value is overwritten by injection (at AppHarbor)
when OPIDDaily is deployed to create a training, staging or production release. The transformation files Web.Staging.config and Web.Release.config
play a role in these deployments. The staging deployment at AppHarbor (called stagedaily) has its Environment variable set to Staging to force
Web.Staging.config to be used upon deployment. This is done in the Settings section of the deployed application at AppHarbor. (When Web.Staging.config
was created, it was necessary to set its build action to Content in Visual Studio to include it in the build at AppHarbor. The same thing happened
when Web.Training.config was created.)  The production deployment at AppHarbor has its Environment variable set to Release by default. This causes
Web.Release.config to be used upon deployment.

The training release was the last release created. It was created by use of the Configuration Manager under the Visual Studio Build menu. In the
`Active solution configuration` dropdown `<New>` was selected and Training was created with settings copied from the Staging configuration. Then
`Add Config Transform` was selected from the context menu of Web.config. (This item is grayed out until a new configuration has been created.) This
automatically added file Web.Training.config. The `<appSettings>` section of Web.Staging.config was copied to Web.Training.config.

## SEO
OPIDDaily is a password protected application that is not intended to be discoverable by search engines. To prevent this it includes
the following robots.txt file

     User-agent:*
     Disallow:/

This directs all search engines to ignore the entire site.

## Visual Studio Project
To check out application OPIDDaily from GitHub into Visual Studio 2019 (Community Edition), use the Visual
Studio capability to clone the repository

     https://tmhsplb@appharbor.com/heretoserve.git

into a local folder after being made a collaborator on the project. Being a collaborator is necessary in order to commit changes to the GitHub repository.

Instructions for how to clone into Visual Studio were found on [this page]
(https://stackoverflow.com/questions/65082592/how-to-clone-a-specific-branch-from-new-git-visual-studio-2019-not-from-command). Note that the code base is stored in the master branch of the heretoserve repository, but
the cloning procedure seems to clone the main branch into Visual Studio instead. In fact, it cloned all
branches and is simply focused on the main branch. The point is that it is necessary to use the Visual Studio
interface to switch to the master branch. Read the StackOverflow page carefully! There is no guarantee that
the cloning procedure described will work in future release of Visual Studio. Just be reassured
that there is a way to clone a GitHub repository into Visual Studio.

The Visual Studio project representing application OPIDDaily defines a role-based system. It was developed using the ASP.NET
Identity 2.0 framework. A sample ASP.NET Identity 2.0 project was developed by Syed Shanu and described in the
[excellent CodeProject article ASP.NET MVC Security and Creating User Role](https://www.codeproject.com/Articles/1075134/ASP-NET-MVC-Security-And-Creating-User-Role).

The sample project uses the Visual Studio MVC5 project template and makes use of Katana OWIN middleware for user
authentication. The use of Katana is
built into the ASP.NET Identity 2.0 provider used by the project template, as is explained in the CodeProject article.

The OPIDDaily application was built with information taken from this article as well as the technique for maintaining 2 data
contexts described in  
[Scott Allen's Pluralsight video](https://app.pluralsight.com/player?author=scott-allen&name=aspdotnet-mvc5-fundamentals-m6-ef6&mode=live&clip=1&course=aspdotnet-mvc5-fundamentals).

On the Properties page of the Visual Studio project, remember to select Local IIS as the server and click the Create Virtual Directory button to set

     http://localhost/OpidDaily

as the Project Url. These two actions create an application called OpidDaily under the Default Web Site in IIS and enable project OPIDDaily to be run
in a desktop version of IIS under this Url. Without this, the desktop IIS cannot be used to host the application. See the section on configuring IIS
below.

New development in the OPIDDaily Visual Studio project will be done in the **staging** branch and deployed to the stagedaily
application at AppHarbor.
(See the section on Deployment.) After changes to the **staging** branch have been tested in the desktop environment, using
the Visual Studio GitHub
interface they will be committed and then pushed to the staging branch at GitHub. After changes have been tested, they will
be merged into the
**master** branch of the project and from there deployed to application OPIDDaily at Appharbor.

When the codebase is installed on a developer's Visual Studio instance on his/her machine by cloning the GitHub repository
**OPIDDaily**, the developer
must use Visual Studio to create a **staging** branch and then rebase this branch onto **origin/master**. This will cause the
remote changes to appear in
the local **staging** branch.

## SQL Server Express and SSMS
The desktop version of OPIDDaily makes use of SQL Server Express to store information about clients. The database is managed by v18.0 of SQL
Server Management Studio (SSMS). Visual Studio includes the ability to view an installed SQL Server Express database, but it is more convenient to have
SQL Server Management Studio available for this purpose.  SQL Server Express and SSMS require separate (lengthy) downloads.

The SQL Server Express database for OPIDDaily was created by executing the SQL query

    create database OPIDDailyDB  

executed inside of SSMS. With this database selected in SSMS, there are two SQL queries that need to be executed to enable IIS
to talk to SQL Server Express. The first query is
````
       CREATE USER [NT AUTHORITY\NETWORK SERVICE]
       FOR LOGIN [NT AUTHORITY\NETWORK SERVICE]
       WITH DEFAULT_SCHEMA = dbo;
````
This query creates the database user NT AUTHORITY\NETWORK SERVICE. The second query is
````
      EXEC sp_addrolemember 'db_owner', 'NT AUTHORITY\NETWORK SERVICE'
````
This query grants user NT AUTHORITY\NETWORK SERVICE the necessary permissions to communicate with IIS. Finally, go into the security
section of database OPIDDaily in SSMS. Expand Users and select the properties for user NT AUTHORITY\NETWORK SERVICE. Select Membership
from the Select a page section of the dialog box and tick the checkbox for db_owner. Performing this final change will permit user
NT AUTHORITY\NETWORK SERVICE to create tables in the database. This ability is necessary for the first the OPIDDaily application is
run to enable it to automatically create the ASP.NET Identity tables in the database.

Creating and configuring user NT AUTHORITY\NETWORK SERVICE does not need to be performed at AppHarbor in order to communicate with IIS.
See below for information about the AppHarbor deployment of OPIDDaily.

It is also necessary to change the application pool identity of application OPIDDaily running under IIS to NETWORKSERVICE. This is done
to the application pool itself, not the application. See the section on configuring IIS.

There is a bug in SSMS v18.0 that causes it to stop after launch; the splash screen will display and then SSMS will quit.
[The fix for this](https://dba.stackexchange.com/questions/238609/ssms-refuses-to-start) is to edit file ssms.exe.config found in folder

    C:\\Program Files (x86)\Microsoft SQL Server Management Studio 18\Common7\IDE

and remove (or comment out) the line which has the text:

    <NgenBind_OptimizeNonGac enabled="1" />

This should be around line 38. Then restart SSMS.

SSMS v18.0 does not have the capability to generate database diagrams. Previous versions of SSMS had this capability,
but it was removed from v18.0. The capability has been added back to newer version of SSMS.

SSMS can be used to connect to a remote database at AppHarbor. The credentials for the remote database can be discovered by
clicking on the SQL Server add-on at AppHarbor and then following the `Go to SQL Server` link on the page that appears.

When connecting through SSMS it is only necessary to use SQL Server Authentication and then supply the AppHarbor server name,
login and password. SSMS will discover the database associated with these credentials. Do not supply the database name
through the connection interface.

## Preliminaries

Before running the program for the first time, make sure the database OPIDDailyDB has been created in SQL Server and, if
running in the desktop environment, user NT AUTHORITY\NETWORK SERVICE has been created and configured. See the section SQL
Server Express and SSMS above for instructions on how to do this.

The CodeProject article (referenced in section Visual Studio Project) gives the Katana middleware code needed to cause the
ASP.NET Identity tables to be created and the first user to be entered into them. This middleware code is found on the
toplevel file Startup.cs. As specified on this file the first user is the SuperAdmin user, sa. When application OPIDDaily is
run for the first time the ASP.NET Identity tables will be created when the middleware is executed. The user sa will then be
found in table **AspNetUsesrs**. File Startup.cs is worth studying.

Since the password for user sa is retrieved by Startup.cs from Web.config where it appears in clear text,
this password should be changed in the deployed application as soon as possible. The Home page of the deployed application
contains a Change Password button that can be used to change the sa password. (The Home page for users in each role also
contains a Change Password button.)  

## Initializing the Database for a Clean Build

The Visual Studio project OPIDDaily is implemented using Version 6 of Enity Framework Code First. The project
maintains 2 data contexts called IdentityDB and OpidDailyDB. The technique for establishing a
single connection string over 2 data contexts is described in
[Scott Allen's Pluralsight video](https://app.pluralsight.com/player?author=scott-allen&name=aspdotnet-mvc5-fundamentals-m6-ef6&mode=live&clip=1&course=aspdotnet-mvc5-fundamentals).

Following the video, supporting 2 data contexts in application OPIDDaily was enabled by some manual scaffolding in the
codebase. This scaffolding in the Visual Studio project for OPIDDaily consisted of creating a folder called DataContexts with
two subfolders: IdentityMigrations and OPIDDailyMigrations. Also, a new folder called Entities was added to contain the
classes defining the entities used by the solution.

If you are building the database from a cloned copy then:

   1. Go to http://localhost/HereToServe
      This will cause the ASP.NET identity tables to be created. You will be able to login as sa.
	  But nothing else will work until all the other tables have been added. See step 2.

   2. Create the remaining tables from the Package Manager Console by running
       PM> update-database -ConfigurationTypeName OPIDDaily.DataContexts.OPIDDailyMigrations.Configuration
	  The cloned copy will already have DataContexts\OPIDDailyMigrations. So running the update-database
	  command will run all the migrations for context OPIDDaily.

Create the top level folder DataContexts and create class IdentityDB in this folder, as specified in Scott Allen's video.
Note that it was found necessary to move the class ApplicationDbContext from file Models\IdentityModels.cs on the video to
file DataContexts/IdentityDb.cs to make things work.

The class AppStart/IdentityConfig will need to be modified in order for the application to compile. Replace the line

     var manager = new ApplicationUserManager(new UserStore<ApplicationUser>(context.Get<ApplicationDbContext>()));

by

    var manager = new ApplicationUserManager(new UserStore<ApplicationUser>(context.Get<IdentityDB>()));

The class AppStart/StartupAuth will also need to be modified. Replace the line

    app.CreatePerOwinContext(ApplicationDbContext.Create);

by the line

     app.CreatePerOwinContext(IdentityDB.Create);

This will invoke the method IdentityDB.Create at the proper time.

Then run the PowerShell command

    PM> Enable-Migrations -ContextTypeName OPIDDaily.DataContexts.IdentityDB -MigrationsDirectory DataContexts\IdentityMigrations

This will create the file

    DataContexts\IdentityMigrations\Configuration.cs

Next in method Configuration on this file set

    AutomaticMigrationsEnabled = true;

to allow the ASP.NET Identity tables to be created when application OPIDDaily is run for the first time. Also add the line

    ContextKey = "OPIDDaily.DataContexts.IdentityDB";

to method Configuration. A similar ContextKey declaration will be made in the Configuration method for the other data
context. The ContextKey is used to distinguish between the two data contexts.

Next run the Powershell command

	   PM> Enable-Migrations -ContextTypeName OPIDDaily.DataContexts.OpidDailyDB -MigrationsDirectory DataContexts\OPIDDailyMigrations

This will create file OPIDDailyMigrations/Configuration.cs. Edit this file to include the lines

      AutomaticMigrationsEnabled = true;
      ContextKey = "OPIDDaily.DataContexts.OpidDailyDB";

in method Configuration. This will distinguish the OpidDaily data context from the IdentityDb data context defined
above. It will enable the add-migration command to find code first changes. Without it all Up() and Down() methods creted by
add-migration will be empty!

Run the Powershell command

      PM> add-migration -ConfigurationTypeName OPIDDaily.DataContexts.OPIDDailyMigrations.Configuration "InitialDB"

to create the migration containing all pre-defined tables. After running the commannd, inspect the InitialDB migration in
folder OPIDDailyMigrations to see that all pre-defined tables are present. If the migration does not look correct, simply
delete it before running the update-database. After verifying correctness, run the command

     PM> update-database -ConfigurationTypeName OPIDDaily.DataContexts.OPIDDailyMigrations.Configuration

to add these tables to the SQL database.

After application OPIDDaily is run for the first time the database will contain not only the ASP.NET Identity table but also
an entity framework table called _MigrationHistory. Entity Framework uses this table to record migrations. As a result of the
creation of the ASP.NET Identity tables, a migration with MigrationId 201906051504117_InitialCreate was created in the
_MigrationHistory table.

Once a migration has been applied to the database by running an update-database command, the code in the
Down() method of the migration must be run to back the migration out of the database. Read more about this on
the Database tab.

## The First New Migration for HereToServe
The HereToServe application is a deployment of the OPIDDaily application under the application name HereToServe
at AppHarbor. The HereToSeve application extends the OPIDDaily application by the inclusion of a gift cards feature.
When starting work on extending the OPIDDaily codebase to create the HereToServe application the new
table **GiftCards** was added to the OPIDDaily codebase via the commands

    PM> add-migration -ConfigurationTypeName OPIDDaily.DataContexts.OPIDDailyMigrations.Configuration "GiftCard"
    PM> update-database -ConfigurationTypeName OPIDDaily.DataContexts.OPIDDailyMigrations.Configuration

Running these two commands accomplished the following:

*  added a migration called GiftCard to folder DataContexts\OPIDDailyMigrations
*  updated the __MigrationHistory table
*  added table GiftCards to the desktop HereToServe database,

Migrations cannot be handled directly by EF Code First at AppHarbor. It is necessary
to use the Package Manager console (PM>) inside Visual Studio to create a script file for the migration and then execute this
script file in SSMS against the AppHarbor database. See the section Adding Migrations at AppHarbor via Script on the
Database tab to see how thisis done. Importantly, this script includes the migration record that will be inserted into the
__MigrationHistory table at AppHarbor. There is no other way to do this! Failure to do this will create an error at AppHarbor
which says  

    The model backing the OpidDailyDB context has changed since the database was created.

This is a fatal error reported by ELMAH.

## Configuring IIS
Development of the OPIDDaily application was performed under IIS in the development (desktop) environment. This
was done so that the development environment would match the deployment environment at AppHarbor as closely as possible.

The localhost application server, Internet Information Services (IIS), was not pre-installed on the localhost; however, it is
part of the operating system that can easily be activated. To activate IIS, go to the Programs section of the Control Panel
and turn on the IIS feature:

    Programs > Programs and Features > Turn Windows features on or off > Internet Information Services

After checking this box, expand it by clicking the plus sign (+) next to it and go to the section

    World Wide Web Services > Application Development Features

In this section, check the checkboxes for

        ASP
        ASP.NET 3.5
        ASP.NET 4.6

if they are not already checked. This will cause additional Application Pools to be made available to IIS.

The OPIDDaily application is installed as an application under the Default Web Site in IIS as described in the section
describing the Visual Studio Project. The Basic Settings dialog box for application OPIDDaily (accessible from the Actions
  pane of IIS), will give the physical path to the folder containing the source code as

    C:\VS2019Projects\OpidDaily\OpidDaily

This folder contains the project solution file, OPIDDaily.sln. Do not change it! Application OPIDDaily must be configured to
use the application pool .NET v4.5. in the Basic Settings dialog box. (This application pool became available by enabling the
features described above.)

Finally, change the application identity of the selected application pool (.NET v4.5) to NetworkService. To do this,
highlight Application Pools on the IIS Connections panel. This will cause the available application pools to appear in the
IIS body panel. Highlight the .NET v4.5 application pool and then select Set Application Pool Defaults… to display a dialog
box that will enable you to change the identity of the application pool. The dialog box
is reachable either from the context menu of the highlighted application pool (under right mouse click) or from the Actions
panel on the right of the IIS display.

The dialog box contains a section labeled Process Model which contains an entry labeled Identity. Selecting the Identity
entry adds an ellipsis next to the bold ApplicationPoolIdentity. Selecting the ellipsis brings up a dialog box with the pre-
selected radio button Built-in account. Select NetworkService from the dropdown menu associated with this radio button. After
approving this selection, the Identity column of the application pool .NET v4.5 will show NetworkService. See the section on
SQL Server and SSMS for how to establish user NetworkService.

## Git for Windows
Visual Studio 2019 (Community Edition) comes with built-in support for GitHub. A new project can be added to Git source control on the desktop by simply
selecting `Add to Source Control` from the context menu of the Solution file in the Solution Explorer. Once a project is under Git source control it can
be added to a remote GitHub repository by using tools available through Visual Studio. However, a technique preferred by many developers is to use [Git
for Windows](https://git-for-windows.github.io/). Git for Windows provides a BASH shell interface to GitHub which uses the same set of commands
available at GitHub itself. Git for Windows integrates with Windows Explorer to allow a BASH shell to be opened on a project that has been added to a
desktop Git repository. Simply point Windows Explorer at the folder containing the project solution file and select `Git BASH Here` from the context
menu of the folder to open a Git for Windows BASH shell. As a first git command, try entering

    git --version

in the shell. This will report the version of Git that has been installed by the download. After that simply execute Git commands from this shell window.
Git for Windows also offers Git GUI, a graphical version of most Git command line functions. To open Git GUI simply select `Git GUI Here` from Windows
Explorer.

## GitHub
Application OPIDDaily is stored at GitHub as a repository under an account with the email address peter3418@ymail.com and account name tmhsplb.

Only user tmhsplb can deploy directly to this repository. Any other user needing to deploy a version of OPIDDaily to this repository
must be declared a collaborator on repository OPIDDaily by user tmhsplb. A collaborator is a user associated with a different account established at
GitHub.  

Git for Windows was used to create a remote to save to this GitHub account. The remote was created in the Git BASH shell by opening the shell on the
folder which contains the OPIDDaily.sln file (folder `C:/VS2019Projects/OPIDDaily/OPIDDaily`) and issuing the command

    git remote add origin https://github.com/tmhsplb/opiddaily.git

Creating this remote only needs to be done once, because Git for Windows stores the remote.

To remove a remote use the command

     git remote rm myremote

The need for this may arise if there was a typo in the creation of myremote.

The HereToServe website was created by first cloning the codebase of OPID Daily and then saving the new code base in the HereToServe
repository at github. This required removing the original remote for origin as follows:

     git remote rm origin

After this, a new remote for origin was added:

    git remote add origin https://github.com/tmhsplb/HereToServe.git

The initial codebase was saved to github through the Sync command under the Team Explorer tab in Visual Studio. The initial Sync inherited
all the changes made to the cloned copy of opiddaily but this is not a problem.

## AppHarbor
AppHarbor (appharbor.com) is a Platform as a Service Provider which uses Amazon Web Services infrastructure for hosting applications and Git as a
versioning tool. When an application is defined at AppHarbor, a Git repository is created to manage versions of the application's deployment.
The OPIDDaily application is defined as an application at AppHarbor to create the production repository of the desktop application. The staging version
of the desktop application is defined by a repository called stagedaily.

The remote configured for OPIDDaily at AppHarbor is:

    https://tmhsplb@appharbor.com/opiddaily.git

This remote is configured from a Windows Git BASH shell by the command

    git remote add opiddaily https://tmhsplb@appharbor.com/opiddaily.git  

After the remote is configured in the Git BASH shell, issuing the command

    git push opiddaily master

will deploy the master branch of solution opiddaily to AppHarbor as application OPIDDaily, accessible through the URL

    https://opiddaily.apphb.com

If you reset your password at AppHarbor, the 'git push' command will no longer work from the Git BASH shell. You need to have Git prompt you for your
new password. To do this on a Windows 10 machine, go to

    Control Panel > User Accounts > Credential Manager > Windows Credentials

and remove the AppHarbor entry under Generic Credentials. The next time you push, you will be prompted for your repository password.

Application OPIDDaily is deployed using the free Canoe subscription level at AppHarbor. Under a Canoe subscription, the IIS application pool of
application OPIDDaily has a 20 minute timeout, which forces OPIIDDaily to spin up its resources again after 20 minutes of idle time. This has
not been a problem at Operation ID, because application OPIDDaily is in continuous use on the days Operation ID is open. However, the 20 minute
timeout for the free Canoe version at AppHarbor would become a problem if OPIDDaily were extended to add features suitable for use by agencies that
partner with Operation ID. These agencies would require that OPIDDaily be available on demand. On demand service would require the use of a paid
subscription level at AppHarbor.

The free Yocto version of SQL Server is used as an add-on to the OPIDDaily deployment. The Yocto version has a limit of 20MB of storage
space, which is adequate for many days of usage by Operation ID. However, the database usage must be monitored to avoid exceeding the 20MB limit.
See the Database Utilization section on the Database tab for how to do this. A paid subscription to a SQL Server at AppHarbor would alleviate this
problem.

## The Staging and Training Versions of OPIDDaily
A staging version of application OPIDDaily was created from a Visual Studio **staging** branch by creating an application called **stagedaily** at AppHarbor.
DO NOT CREATE A SEPARATE REPOSITORY FOR STAGEDAILY AT GITHUB.

The remote configured for stagedaily at AppHarbor is:

    https://tmhsplb@appharbor.com/stagedaily.git

This remote is configured from a Windows Git BASH shell by the command

    git remote add stagedaily https://tmhsplb@appharbor.com/stagedaily.git

After the remote is configured in the Git BASH shell, issuing the command

    git push stagedaily staging

will deploy the staging branch of OPIDDaily to AppHarbor as application stagedaily, accessible through the URL

    https://stagedaily.apphb.com

In the same way, a **traindaily** application at AppHarbor was created from a **training** branch in Visual Studio.

## Roslyn
On June 6, 2019 I ran into a problem when I first pushed my OPIDDaily solution from my laptop to its GitHub repository and tried to clone the resulting
repository onto my desktop computer. When I tried to run the solution from my desktop it complained about missing part of the path /bin/roslyn/csc.exe.
I found a fix that worked at StackOverflow

     https://stackoverflow.com/questions/32780315/could-not-find-a-part-of-the-path-bin-roslyn-csc-exe

The problem is that the \bin\roslyn folder is missing. This folder contains csc.exe which is needed.

There were many proposed fixes, but the one that looked easiest (score 205 points) to try was: clean solution
and then rebuild. It worked!

## Deployment
This section summarizes deployment to AppHarbor. Much of the information here can be found in the section on AppHarbor.

There are three applications at AppHarbor: **opiddaily** ,**stagedaily** and **traindaily**. Application **opiddaily** is the deployment of the Visual Studio
**master** branch of solution OPIDDaily. Application **stagedaily** is the deployment of the Visual Studio **staging** branch of solution OPIDDaily and
application **traindaily** is the deployment of branch **training** in Visual Studio.

After configuring the **opiddaily remote** the Visual Studio production branch can be deployed to AppHarbor by using
the Git BASH Shell command

    git push opiddaily master

AppHarbor will automatically deploy application OPIDDaily if the push results in a successful build. After AppHarbor finishes building and
deploying the code, application **opiddaily** can be viewed at

    https://opiddaily.apphb.com

 After configuring the staging remote (see above) the Visual Studio staging branch can be deployed to AppHarbor by using the Git BASH Shell command

    git push stagedaily staging

AppHarbor will not automatically deploy application **stagedaily** even if the build is successful. It is necessary to click on the Deploy button
at AppHarbor to deploy a successful build of application **stagedaily**. This may be by design if application **stagedaily** is recognized as a GitHub
branch of application OPIDDaily.

After clicking the Deploy button at AppHarbor to deploy a successful build of application **stagedaily**, the application can be viewed at

    https://stagedaily.apphb.com

After configuring the training remote (see above) the Visual Studio training branch can be deployed to AppHarbor by using the Git BASH Shell command

    git push traindaily training

Again, the Deploy button must be clicked to actually deploy application **traindaily** and run it through the URL

    https://traindaily.apphb.com

Although there are three applications at AppHarbor, there is only a single repository at GitHub. The name of this single repository is OPIDDaily. The
repository is by default focused on the **master** branch of the codebase but can be switched to the **staging** branch or **training** branch by using the
GitHub interface.

## jqGrid
Almost every page of the application OPIDDaily features a grid produced by the jQuery jqGrid component. It was installed into the OPIDDaily project by
using the Package Manager command:

    PM> Install-Package Trirand.jqGrid -Version 4.6.0

There is a collection of [jqGrid Demos](http://trirand.com/blog/jqgrid/jqgrid.html) that was very helpful during the development of OPIDDaily. There is a section about the jqGrid on the Implementation tab.

## ELMAH
Unhandled application errors are caught by ELMAH. Version 2.1.2 of Elamh.Mvc was installed in project OPIDDaily by using the Visual Studio NuGet package
manager. By default, the ELMAH log can only be viewed on the server that hosts the application in which ELMAH is installed. To make the ELMAH log
visible to a client remotely running the application, add

    <elmah>
       <security allowRemoteAccess="1" />
    </elmah>

to the `<configuration>` section of file Web.config.

To see ELMAH in action, modify the URL in the browser address bar to, for example,

    opiddaily.apphb.com/Admin/Foo

This will generate an unhandled error because the MVC routing system will not be able to resolve the URL. Then go to

    opiddaily.apphb.com/elmah.axd

to see that this error has been caught by ELMAH. On the localhost use

    localhost/opiddaily/elmah.axd

to see the list of ELMAH errors.

Installation of the Elmah.Mvc package adds the necessary DLL's and makes the necessary changes to Web.config to configure ELMAH for use. By default
ELMAH will write to a database table called ELMAH_Error. The DDL Script definition of this table is found in a
[separate download](https://elmah.github.io/downloads/). Download the DDL Script for MS SQL Server from the referenced web page. The script is a .SQL
file which may be executed as a query inside SSMS to create table ELMAH_Error and 3 stored procedures.

The ELMAH log is configured by the connection string named **OpidDailyConnectionString** on Web.config. The `<sytem.web>` section of Web.config must configure
```
    <httpHandlers>
      <add verb="POST,GET,HEAD" path="elmah.axd" type="Elmah.ErrorLogPageFactory, Elmah" />
    </httpHandlers>
```
 and the `<system.webServer>` section must configure
```
     <handlers>
       <add name="Elmah" verb="POST,GET,HEAD" path="elmah.axd" type="Elmah.ErrorLogPageFactory, Elmah" />
     </handlers>
```
in order for ELMAH to log both on the local IIS and on the remote server at AppHarbor.

## log4net
Application logging is handled by Version 2.0.8 of log4net by the Apache Software Foundation. Logging is used primarily for reporting of detected errors
during application execution. But logging is also very useful when trying to understand the execution of the code. Normally Visual Studio breakpoints
can be set in the code to interrupt execution and inspect the value of variables and parameters. But as the code grows larger it becomes increasingly
difficult to run it in Visual Studio debug mode. It is, however, always easy to insert logging statements to report on the values of variables and
parameters without interrupting execution.

The log4net package was installed using the Visual Studio NuGet package manager. The application log for project OPIDDaily is maintained as a database
table as described in [this article](https://logging.apache.org/log4net/release/config-examples.html) describing the AdoNetAppender for log4net. The
article includes a script for creating table Log (renamed AppLog in application OPIDDaily). The script must be executed as a query in SSMS to create
table AppLog in the database.

Table AppLog is created by the following script:

    CREATE TABLE[dbo].[AppLog] (
      [Id][int] IDENTITY(1, 1) NOT NULL,
      [Date][datetime] NOT NULL,
      [Thread][varchar](255) NOT NULL,
      [Level][varchar](50) NOT NULL,
      [Logger][varchar](255) NOT NULL,
      [Message][varchar](max) NOT NULL,
      [Exception][varchar](max) NULL)

The application log is configured by the connection string named OpidDailyConnectionString on Web.config. The value of this connection string is
overwritten when the application is deployed to AppHarbor. See the Connection String section of the Database tab.

log4net requires some additional configuration in Web.config. In the <configSections> section add:
```
   <section name="log4net" type="log4net.Config.Log4NetConfigurationSectionHandler, log4net" />
```
After the <configSections> section add the definition of the AdoNetAppender:
```
   <log4net debug="true">
      <appender name="AdoNetAppender" type="log4net.Appender.AdoNetAppender">
        <!--Change to 10 or MORE. This is critical, after 10 messages then log to database-->
        <bufferSize value="1" />
        <connectionType value="System.Data.SqlClient.SqlConnection, System.Data, Version=1.0.3300.0,
              Culture=neutral, PublicKeyToken=b77a5c561934e089" />
        <connectionStringName value="OpidDailyConnectionString" />
        <commandText value="INSERT INTO AppLog ([Date],[Thread],[Level],[Logger],[Message],[Exception])
               VALUES (@log_date, @thread, @log_level, @logger, @message, @exception)" />
        <commandType value="Text" />
        <!--<commmandText value="dbo.procLog_Insert"/> <commandType value="StoredProcedure"/>-->
        <parameter>
          <parameterName value="@log_date" />
          <dbType value="DateTime" />
           <layout type="log4net.Layout.RawTimeStampLayout" />
         </parameter>
         <parameter>
           <parameterName value="@thread" />
           <dbType value="String" />
           <size value="255" />
           <layout type="log4net.Layout.PatternLayout">
              <conversionPattern value="%thread" />
           </layout>
         </parameter>
         <parameter>
            <parameterName value="@log_level" />
            <dbType value="String" />
            <size value="50" />
            <layout type="log4net.Layout.PatternLayout">
               <conversionPattern value="%level" />
            </layout>
         </parameter>
         <parameter>
            <parameterName value="@logger" />
            <dbType value="String" />
            <size value="255" />
            <layout type="log4net.Layout.PatternLayout">
               <conversionPattern value="%logger" />
            </layout>
          </parameter>
          <parameter>
             <parameterName value="@message" />
               <dbType value="String" />
               <size value="4000" />
             <layout type="log4net.Layout.PatternLayout">
                <conversionPattern value="%message" />
             </layout>
           </parameter>
           <parameter>
              <parameterName value="@exception" />
              <dbType value="String" />
              <size value="2000" />
              <layout type="log4net.Layout.ExceptionLayout" />
          </parameter>
      </appender>
      <root>
         <level value="DEBUG" />
            <!-- <appender-ref ref="RollingLogFileAppender" /> -->
         <appender-ref ref="AdoNetAppender" />
      </root>
    </log4net>
```
Notice that the configured value of the bufferSize is 1, despite the comment above the configuration to use a value of 10 or more. The value
of 1 is chosen so that buffering of log statements will not occur; each log statement will be written to the log file at the time it is
generated. This is important because OPIDDaily does not include many log statements. If the buffer size were set to 10 or more, then log
statements would not appear in the log file until 10 log statements had accumulated. Setting the bufferSize to 1 is convenient for Log.Debug
statements which may be needed to trace the behavior of the website at AppHarbor where the Visual Studio debugger is not available. It is
also at times convenient when debugging in the desktop environment where the Visual Studio debugger is available.

Finally, log4net must be initialized on file Global.asax.cs by including the configuration statement

   log4net.Config.XmlConfigurator.Configure();

Without this statement nothing will work even if log4net is correctly configured on Web.config.

## MkDocs
This document was created using MkDocs as was the [MkDocs website](http://www.mkdocs.org/) itself. MkDocs
requires python and was installed following the guide
on [this page](https://www.sitepoint.com/building-product-documentation-mkdocs/). This guide is useful for setting up the environment; however,
the syntax for the file mkdocs.yml has changed from that described in the guide. The new syntax can be found in the User Guide section of
[this document](https://www.mkdocs.org/user-guide/writing-your-docs/#configure-pages-and-navigation).

An MkDocs document is a static website and can be hosted by any service that supports static sites. This MkDocs
document is hosted by
[GitHub Pages](https://pages.github.com/). The [Atom](http://atom.io/) open source text editor was used to
develop the document on the desktop.
An MkDocs document uses HTML Markdown for a desktop development version of a document. GitHub provides a
[cheatsheet for Markdown syntax](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet).

MkDocs provides a built-in preview server. To start this server, open a BASH Shell on the folder containing the
mkdoc.yml file of the project and execute

    mkdocs serve

Then go to

    http://127.0.0.1:8000

in a desktop browser. Pages can be edited and saved while in preview mode. The changes will be reflected in the browser document.

When it is time to publish a version of a document, in a Git BASH shell opened on the folder containing the mkdocs.yml file, issue the command

    mkdocs build

to expand the Markdown version of the document into an HTML version in the /site folder. Then open the Git GUI on the
folder containing the mkdocs.yml file and use the GUI to create a new Git repository on the local disk.

Next at GitHub create repository opiddailydoc to hold the documentation. Then create a repository on the desktop
machine
to associate with the GitHub repository. Issue the following command in the folder containing the mkdocs.yml file:

    git init

After this, in the folder containing the mkdocs.yml file, define a remote called origin for the document:

    git remote add origin https://github.com/tmhsplb/opiddailydoc.git

This command references the GitHub repository opiddailydoc. The remote only needs to be defined once. It will be remembered by the
Git BASH shell.

In the shell issue the following commands:

    git add -A

    git commit -a -m 'Initial commit'

    git push origin master

This will push the master branch of the document to the repository identified by the remote called origin. Then click on the Settings tab for the newly
created repository and scroll down to the GitHub Pages section. Select the master branch source and click on the Save button.

Finally, to view the published document go to:

    https://tmhsplb.github.io/opiddailydoc/site

Unless a new file is added to file `mkdocs.yml`, subsequent edits only require the commands

    mkdocs build

    git commit -a -m '<Comment for new commit>'

    git push origin master

to update repository opiddailydoc at GitHub.

If a new file is added to `mkdocs.yml` then

    git add -A

must be run before the `mkdocs build` command is run. This causes any new files to be added to the local git repository. In either case it may take
several minutes before edits are available.
