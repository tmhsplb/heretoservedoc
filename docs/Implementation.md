# Implementation
Application OPIDDaily is implemented as an ASP.NET Framework application using the ASP.NET MVC 5 project template provided by Visual Studio 2019
(Community Edition). OPIDDaily is implemented as an ASP.NET Foundation application and as such is hosted by IIS. Since cross platform availability
is not a goal of OPIDDaily, there is no need to port it to ASP.NET Core. However, it may be useful to provide an alternative to AppHarbor as a hosting
service.

OPIDDaily uses ASP.NET Identity 2.0 to define a set of user roles. Each user role is associated with a separate MVC controller.
A key feature is the use of controller inheritance (see the SharedController section below) to share editing functionality across user roles.
Most of the OPIDaily interface is built around the JavaScript component jqGrid. This component provides a powerful user interface for CRUD operations,
relieving the application of the burden of providing such an interface. (See the 2 sections on the jqGrid and the dashboard below.)

## Bootstrap
The graphical user interface of OPIDDaily is styled using Bootstrap 4.5.2 (as shown in packages.config).  
The technique for placing the Register and Login links at the right of the menubar using Bootstrap 4 comes from
[this StackOverflow page](https://stackoverflow.com/questions/41513463/bootstrap-align-navbar-items-to-the-right).

## Architecture
Application OPIDDaily uses the well-defined architecture of an MVC application. The presentation layer is defined by the collection of controllers
and corresponding views. Controllers make calls to the data access layer to retrieve requested data from the database. The data access layer communicates
with the database using Entity Framework.

#### Models
The Models folder contains the models used by application OPIDDaily. Each model name with suffix ViewModel is a view model used to pass data from
a controller to a view.

#### Data Access Layer
The collection of classes in the DAL folder comprise the data access layer. Controllers retrieve database data by invoking
methods of the classes in the data access layer. The data access layer contains many methods that convert back and forth between
entities and view models. For example, method ClientEntityToClientViewModel returns a ClientViewModel representing the ClientEntity
which it receives as an argument. And method ClientViewModelToClientEntity converts a ClientViewModel passsed as its first argument
to a Client entity passed as its second argument. This back and forth conversion enforces a separation of concerns.

#### Database Entities
The Entities folder contains classes representing the database entities. There is one database entity corresponding to each application
database in the OPIDDaily database.

#### Data Contexts
Application OPIDDDaily uses 2 data contexts: IdentityDB and OpidDailyDB. These contexts are defined as classes in the DataContexts folder. The
OpidDailyDB class creates the OpidDailyDB context and Entity Framework Code First uses it to create tables in the database corresponding to the
entity classes in the Entities folder. Thus, for example, the class variable declaration
````
   public DbSet<Client> Clients { get; set; }
````
causes Entity Framework to create table **Clients** in the database.

#### Scripts
The Scripts folder contains the JavaScript scripts used by application OPIDDaily. The MVC application template provisions the application
with a set of generally useful scripts in an MVC application. Other scripts in the Scripts folder are added by adding packages through the
NuGet package manager. The scripts in the subfolders

* ClientHistory
* Clients
* Conversation
* Dashboards
* PocketChecks
* ReviewClients

all define jqGrids used by views in the application. The large number of jqGrids is evidence of the importance of this component to application
OPIDDaily.
## Users
OPIDDaily is a role-based system. Each registered user will be assigned a user role by the OPIDDaily administrator.
The role that a user is assigned will determine the OPIDDaily features available to the user. A user's assigned role will depend upon
wheterh thr user volunteers at the front desk, is a volunteer interviewer or is a back office volunteer. There will also be a SuperAdmin
role. The single user in this role, the Superadmin, will have access to features necessary for the maintenance of application OPIDDaily.
The credentials for this account will only be available to a select few individuals.

## The SuperAdmin User
OPIDDaily defines a pre-registered SuperAdmin user who has privileges to

* create new roles
* invite new users to register in a pre-determined role
* add new agencies

The credentials for the SuperAdmin user are configured on file Startup.cs. File Startup.cs is where the Superadmin role is created. There is only a
single user, the SuperAdmin, with role of Superadmin.

## MVC Routing
Application OPIDDaily uses only the default routing rule supplied by the Visual Studio MVC 5 template. This default routing rule is found in
````
`.../App_Start/RouteConfig.cs`.

    routes.MapRoute(
      name: "Default",
      url: "{controller}/{action}/{id}",
      defaults: new { controller = "Users", action = "Index", id = UrlParameter.Optional }
    );
````
For the sake of simplicity, future development of application OPIDDaily should strive to keep this as the one and only routing rule.

## Role Controllers
There is an MVC controller defined for each role defined by the SuperAdmin user. Each controller defined for a role inherits from SharedController to
implement shared functionality. The role controllers manage the views of application OPIDDaily. The implementation of each role controller
defines methods that are accessible through the menubar defined on the layout file for the role. For example, the FrontDeskController - which implements
the FrontDesk role - contains methods ExpressClient and ExistingClient (found on the SharedController) invoked from the menubar defined on file

    ~/Shared/_FrontDesk.cshtml.

This is the layout file for the FrontDesk role. Each view returned by the FrontDeskController includes this layout file, thereby ensuring that a
user in the role of FrontDesk will only invoke methods defined by the FrontDeskController.

Each other role controllers is implemented the same way: each has a defined layout file that is included in each view returned by the controller.
The layout file defines a menubar that specifies the methods that users in the role can invoke.

As protection against unauthorized access to methods of a role controller, use of each role controller is limited to users in the role associated with
the controller. For example, the FrontDeskController is protected by the annotation

    [Authorize(Roles = "FrontDesk")]

Access is then restricted by the functionality of ASP.NET Identity to authenticated users in role FrontDesk.

A quirk of role-based authorization of controllers occurs when a new user is registered in a role. The AccountController register method redirects
a newly registered user to the Index method of the UsersController. This method then in turn attempts to redirect the user to the Home method of
the appropriate controller, based on the newly registered users role. For example, a newly registered user in the role BackOffice will be redirected
to the Home method of the BackOfficeController. Here lies the rub. The BackOfficeController is decorated with the attribute

    [Authorize(Roles = "BackOffice")]

but the newly registered user, even though he/she is in role BackOffice as determined by the UsersController is blocked by the attribute! The
attribute causes the user to be redirected to the Login method of the AccountController. This may be by design and logging in with the same
credentials just supplied in the registration process works. But it would seem that the act of registering should cause the new user to be logged in
upon registration. When a new session is started, a user registered in role BackOffice is not blocked by the Authorize attribute even though the login
workflow is basically the same as the registration workflow. Go figure!

In any event, registration of new users for application OPIDDaily has been handled by the SuperAdmin user who therefore knows
the initial password of each user. There is a Change Password feature which allows a user to change his/her password.
After a password has been changed it will no longer be known to the SuperAdmin.

At this time there is no Forgot Password feature, since this would require access to an email server and this access is
not available. A user who forgets his/her password will have to ask the SuperAdmin to reset it.

## The SharedController
Each role controller derives from SharedController. The SharedController implements the shared editor functionality available to the different roles.
The FrontDeskController, the BackOfficeController, the InterviewerController and the CaseManagerController all inherit from the SharedController.
Deriving role controllers from a shared controller is an extremely useful technique for building a role-based editor.

## The UsersController
The UsersController controls access to the ASP.NET Identity tables used to store registered users and the roles they are in. The method
UsersController.Index is the entry point for an authenticated user. The role an authenticated user is in determines the method the user will be
redirected to from this entry point.

## jqGrid
The entire implementation of application OPIDDaily is structured around instances of jqGrid appearing in MVC Views.
Instructions for installng jqGrid are given on the Infrastructure tab.

Each jqGrid is initially populatedby a call to an MVC action made through the url property of the grid. For example, the
clientsGrid on view FrontDesk/Clients.cshtml is initially populated by the call

    "@Url.Action("GetClients", "FrontDesk")"

which is the value of the url argument to grid clientsGrid. (Method GetClients is found on the SharedController.) The clientsGrid is updated by
SingalR without requiring a page refresh! This enhances the interactivity of the OPIDDaily application.

Each instance of a jqGrid defines a *pager*, which defines the CRUD operations supported by the grid. Each CRUD operation is
implemented by an MVC action of the role controller associated with the grid.

There is a collection of [jqGrid Demos](http://trirand.com/blog/jqgrid/jqgrid.html) that was very helpful during the development of OPIDDaily.

## Dashboard
The jqGrid used to represent the dashboard of service requests supports filtered server side searching. Performing filtered searches on
the server side is necessary in order to preserve pagination, which will be needed if there are more than 25 service requests active
at any given time. The dashboard is sorted by ascending order of service request expiry. A service request rolls off the dashboard
once its expiry has passed.

The dashboard is made searchable via the configuration:

    jQuery("#dashboardGrid").jqGrid('filterToolbar', { searchOperators: true });

Each column which supports filtered searching (example Agency Name) includes the parameter

    search: true

in its column declaration. Filtered searching is implemented by the method SharedController/GetDashboard.

The parameter sps of type SearchParameters is populated by the POST that calls method GetDashboard. The
POST will include the search string as the value of the jqGrid column name, for example AgencyName or LastName.
The date of birth is searchable with a side-effect. All clients sharing a specified date of birth will be
retrieved, not just those on the current dashboard. This allows a client whose service request has rolled off
the dashboard to be retrieved. If the Service Ticket of a "rolled off" client is viewed, then as a side effect, the
service request for the client will be restored as the first service request on the dashboard, unless the client is
a dependent of another client and hence not the head of a household. A service request for a dependent client will
never appear at the top level of the dashboard..  

Method GetDashboard calls Clients.GetDashboardClients which in turn calls GetFilteredDashboardClients in case
a filtered search has been requested. This is determined by the boolean variable _search. The value of _search
will be true if a filtered search has been requested and false otherwise. The value of _search is false on the
initial call to GetDashboard and on each call to refresh the dashboard.

The parameter setting

    loadonce : false

as part of the jqGrid declaration enables the server side filtering.

## Gift Cards
The HereToServe application is an extension of the OPID Daily application created to handle the distribution of gift
cards to clients seeking help with identity services through Operation ID. The extended application is deployed as
the website HereToServe at Appharbor. See the section Before You Start on the Background tab.

There is a one-to-many relationship between a client and the number of gift cards the client may be issued. This
relationship is represented by the table **GiftCards** which maintains a foreign key to the **Clients** table.

There is also a table called **GiftCardInventories** that is part of the Gift Cards extension. This table maintains
a timestamped inventory of the number of gift cards of each available type that are in stock. There is a GiftCardsController
that is private to users in the role of Gift Card Admin. A Gift Card Admin user is responsible for maintaining the
inventory and setting the gift card budgets of each agency participating in the gift cards program.

## Pocket Checks
An uncashed check issued to a client is referred to as a *pocket check*. This name is intended to evoke the image of a client
putting the check in his/her pocket at the time it is issued. The disposition of a pocket check is not known until it
appears on a report from the bank. Until this time it may be thought of as being in the pocket of a client.

A pocket check is most often made out to either the DPS or the Bureau of Vital Statistics
and is intended to be cashed for a specific service. A pocket check may also expire before it is cashed. A cashed pocket
check will appear in a bank report as being Cleared and an expired check will appear as Voided.

Gift cards will be treated as pocket checks. When a Gift Card Admin reserves a gift card for a client, a pocket
check representing the gift card will be created in the **PocketChecks** table. When the gift card has been delivered
to a client, the corresponding pocket check should be deleted from the **PocketChecks** table.

The table **PocketChecks**, used to store pocket checks, is related table to the **Clients** table, since there is
a one-to-many relationship between a client and pocket checks issued for that client. However, instead of using a
foreign key to establish this relationship, the table **PocketChecks** has been implemented
as a standalone table with a ClientId data field which points back to the client owning any given pocket check.

Once the disposition of a pocket check
becomes known (either Cleared or Voided), the pocket check is removed from the table **PocketChecks**. Each pocket check is
represented by a corresponding research check, which is created at the time the pocket check is created. Service Tickets
retrieve a client's visit history from the Research Table, not from any pocket checks.

The table **PocketChecks** is updated by a *cross-load* but a cross-load will not cause pocket checks to be removed from the
table. A cross-load is a merge into the OPID Daily database of the OPID Daily Tracking Report. This report offers a means of
updating the OPID Daily database based on modifications performed on the Apricot database since a specified date. An Origen
Bank file which resolves a pocket check will cause the pocket check to be removed form table **PocketChecks**.

A cross-load is an undesired coupling between Apricot and OPID Daily. Ideally, OPID Daily alone should manage the database of
checks issued by Operation ID; Apricot should not need to store any check information. OPID Daily should be updated by
consuming the check reports issued by Origen Bank, not by a cross-load from Apricot. A check whose
disposition is settled by Origen Bank will be removed from the table **Pocket Checks**. The practice of
cross-loading will continue until OPID Daily is viewed as a reliable alternative to Apricot for check management.

## Styling a jqGrid using jquery-ui
The default jquery ui theme that comes with a Visual Studio 2019 project is referenced by
adding the lines

   "~/Content/themes/base/jquery-ui.css",
   "~/Content/themes/base/jquery.ui.theme.css"

to the Content/css bundle on AppStart/BundleConfig.cs and adding the link

    <link href="@Url.Content("~/Content/themes/base/jquery.ui.all.css")" rel="stylesheet" />

to each file that contains a jqGrid.

The file jquery.ui.all.css contains styling for all the jquery ui components. But this is not
necessary for the OPIDDaily application, because the only jquery component contained on any
page is the jqGrid component. Instead of linking in jquery.ui.all.css by the above link, replace
this link by the link

    <link href="@Url.Content("~/Content/jquery.jqGrid/copied.ui.jqgrid.css")" rel="stylesheet" />

on each page that uses a jqGrid. This links in file ui.jqgrid.css that comes with the jqGrid
download. This file has a problem with correctly displaying the caption of a jqGrid. A new
version was downloaded from the [jqDemos page](http://www.trirand.com/blog/jqgrid/themes/ui.jqgrid.css)
and copying the displayed stylesheet to the file copied.ui.jqgrid.css. When this is linked
in instead of the original ui.jqgrid.css the caption is displayed correctly. However, it still
uses the default theme referenced above.

To change the theme, go to the Download tab on jqueryui.com. Uncheck all components, since
only the jqGrid needs to be styled and this is done by ui.jqgrid.css which came with the
jqGrid download. At the bottom of the page, select a theme (the jqDemos page uses the Redmond
theme) and click the Download button to get a .zip file.

The .zip file includes files named jquery-ui.css and jquery-ui.theme.css. Rename them according
to the selected theme (example: redmond-jquery-ui.css and redmond-jquery-ui.theme.css) and
copy them to the project. The .zip file also contains an images folder which contains images
referenced by the theme. Rename the existing Content/themes/base/images folder to be old-images
and create a new images folder. Copy the images from the .zip file to the new images folder.  

Finally, on file AppStart/BundleConfig.cs, replace the lines providing the default theme (see
above) by the lines

     "~/Content/themes/base/redmond-jquery-ui.css",
     "~/Content/themes/base/redmond-jquery-ui.theme.css"

This will cause the styling on a page containing a jqGrid to be done according to the Redmond
theme.

## jquery Datatables
The Research Table is implemented as a server side datatable. It is dependent on 2 packages downloaded using the NuGet
package manager:

    jquery.datatables  v1.10.15
    datatables.mvc5    v0.1.0

The first of the 2 packages adds folders:

    ~/Content/DataTables
    ~/Scripts/DataTables

For more information see the [jquery DataTables website](https://www.datatables.net/).

## NowServing
By convention, wherever the code makes use of a variable with the name `nowServing`, its value is the id of a client that has been
selected from a jqGrid. Understanding this convention greatly simplifies understanding the code base.  When a row is selected from a
client grid or dashboard the id of the selected row is the id of the client referred to by the row. This id is stored in the session context
for ease access by any method in the code. This technique eliminates the need to pass the id as an argument to each method that needs it.
It is in effect a means of managing session state across pages in the OPID Daily website in the same way that cookies are designed to manage
state across web pages in a large website. See the **NowConversing** section for additional details.

## NowConversing
When a row in the clientsGrid defined on FrontDeskClients.js is selected, the JavaScript function that is the value of the
onSelectRow property of the grid is invoked. When the function is invoked, the value passed to its `nowServing` argument is the id
associated with the client represented by the selected row.  The function posts to the server side method NowConversing found on SharedController,
passing the JavaScript variable `nowServing` in the post as a result of the line

    postData: { nowServing: nowServing }

Method SharedController/NowConversing has an argument called nowServing. MVC data binding will cause this variable to be
bound to the JavaScript variable in the post.

The method NowConversing was originally called NowServing, but was changed when it was understood that it needs to return a
JsonResult to the grid in order to support pagination. The name NowConversing was chose, because the call frequently initiates
a conversation between two OPID Daily users. The JsonResult returned by NowConversing is the set of records that are the
current focus of the jqGrid. When the JsonResult is returned, the code
````
    .trigger('reloadGrid', { fromServer: true });
````
chained to the call to NowConversing then forces the grid to reload with the grid data records in the JsonResult.

It would be natural to follow this chained call by a call to set the selection:
````
    .trigger('reloadGrid', { fromServer: true }).jqGrid('setSelection', nowServing);
````

but this DOES NOT WORK. Instead, it is necessary to wait for the reload to complete before the selection is set.
````
    loadComplete: function () {
      jquery("#dashboardGrid").jqGrid('setSelection', lastServed);
    }
````
The global JavaScript variable `lastServed` referenced here is set to the value of `nowServing`.

There is a complicated interplay between the server side and the client side regarding the selection of a client
being served. This interplay is at the heart of how the OPIDDaily application works. For this reason it is important
to study method NowConversing and the several jqGrids which call it.

One thing that will be noticed is that the JsonResult returned by method NowConversing depends upon wheter it is being
called by an agency or by OPID. If it is being called by OPID, it returns records for a dashboard display, which may
include records from clients being served by several different agencies. When it is called by an agency, the JsonResult
includes only records from clients being served by that agency.

## Households
Table **Clients** is a list of clients and their dependents. Most clients represent only themselves but some have dependents. A client together with any
dependents he or she may have is referred to as a *household*. The head of a household is a top level member of the table **Clients**. The HH field of
each top level client C is set to 0, that is C.HH = 0. If client D is a dependent of client C then the HH field of client D points back to client C through
the setting D.HH = C.Id. In fact, dependent D belongs to a subgrid of the jqGrid used to
render the **Clients** table. Dependent D is added to the subgrid whose parent entry is the entry for client C. If client C is has dependents, then
C.HeadOfHousehold will be set to true and the row rendering client C in the jqGrid will indicate that client C is the head of a household.

## Conversations
The OPID Daily application supports conversations between users through a sequence of text messages. There is a one-to-many relationship
between each client and the messages concerning that client. Selecting a client from a client grid causes the conversation
of text messages to open. Each message is tagged with a date, a sender and a receiver. The entire conversation is stored
in the table **TextMsgs** which uses a client Id as a foreign key. (See the database diagram on the Database tab.). When a client
is selected from a grid and a text message is sent, the client's row in the grid turns red, indicating waiting on receipt of a reply.
The recipient of the message sees the row representing the client in the grid he/she is monitoring turn green, indicating that a
message is waiting to be read. This simple mechanism enables communication between users of OPID Daily without the need for time consuming
phone calls or difficult to manage emails.

## The AgencyId data field in table **AspNetUsers**
Application OPIDDaily is used by users at Operation ID and by users representing agencies that refer clients to
Operation ID. The AgencyId field in table **AspNetUsrs** will be set to 0 for Operation ID users (for example
the TicketMaster user) and will be set to the AgencyId of their referring agency for other users. The AgencyId
data field in the **Clients** table is set in the same way. So, the AgencyId of a client entered into the **Clients**
table at Operation ID on a day of operation will be set to 0. The AgencyId of a client entered by a user representing
a referring agency will be set to the AgencyId of the referring agency. Clients are tagged in this way to allow
Operation ID users to see all clients and to allow a users from a referring agency to see only the clients referred
by that agency.

## Date Management
Dates are treated differently at AppHarbor from the way they are treated on the desktop. In both environments, a date is
stored as the midnight hour. For example, March 26, 2020 is stored as 2020-03-26:00:00:00:000 in the desktop database as
well as in the AppHarbor database. But if this date is stored in a ClientViewModel object and reported as the value of a jqGrid
column using the date formatter, then the desktop reports it as 03/26/2020 whereas AppHarbor reports it as 03/25/2020. It is not
clear why this happens, but a way to make both environments report 03/26/2020 is to offset the value stored in the view model by
12 hours to noon (2020-03-26:12:00:00:000). This is for done several DateTime data fields in Clients.ClientEntityToClientViewModel.
(A 1 hour offset was tried first, but it did not work.) The same 12 hour offsetting was done with the Date field in a VisitViewModel.
This can be found on file DAL/Visits.cs.

When a jqGrid row containing a date is edited the date posted to the server is the midnight time, not the offset to noon time.
When the row is retrieved in a ClientViewModel, the time will again be offset by 12 hour to noon and will thus display as intended
in the jqGrid.

## The ServiceDate and Expiry data fields in table **Clients**
When the TicketMaster adds a new client to the **Clients* table, the date the client is entered is recorded in the
ServiceDate data field of table **Clients**. The same date is recorded in the Expiry data field. When a Case Manager
user enters a new client into the **Clients** table, the ServiceDate data field is set to the date the client is
entered, but the Expiry data field is set at least 30 days into the future. This expiry date will be printed
on the Case Manager Voucher given to a client by his/her Case Manager. Application OPIDDaily will preload the
Dashboard table seen by the TicketMaster with any clients whose Expiry date has not yet past. This will allow
a client with a Case Manager Voucher to be tracked on the day he/she appears at Operation ID. It is
the client's responsibility to appear at Operation ID on or before the expiry date printed on their voucher.  

## SessionHelper
The SessionHelper class found on fie DAL/SessionHelper.cs is the key method for managing state in application OPIDDaily. This class is
used to store key value pairs in the sesssion context private to each authenticated user.

Managing the value of NowServing is a key use of the SessionHelper. Instead of having methods called GetNowServing and SetNowServing, the
SharedController uses polymorphism to define two methods called NowServing with different signatures. The NowServing method with zero arguments invokes
method SessionHelper.Get and the NowServing method with optional argument nowServing invokes method SessionHelper.Set. These two NowServing methods are
invoked by many methods on SharedController to implement editing functionality private to an authenticated user.

## Service Tickets
A primary goal of application OPIDDaily is the production of *Service Tickets*. A service ticket is a single piece of paper that shows the services
requested by a given client together with the documents the client is supplying in support of his/her service request. In addition, a service ticket
provides a history of service requests from previous visits by the client, if any.

Service Tickets are produced by the TicketMaster at the front desk for same-day-service clients. In practice, the TicketMaster only creates a Service
Ticket for a client with a history of previous visits. This Service Ticket is attached to the referral letter for a same-day-service client and the
client is picked up by an interviewer. The Service Ticket contains a section of supporting documents which an interviewer can use as a worksheet
while interviewing a client. The supporting documents section is not filled out by the front desk admin. The interviewer will assist the client in
filling out paperwork in support of service(s) sought.

It has been suggested that a client's Service Ticket be forwarded to a back office admin before work on a client's paperwork begins. By passing
the Service Ticket to a back office admin, a client's service voucher(s) can be cut and recorded in Apricot while the client is busy with paperwork.
In effect a client's **BackOffice** stage can proceed in parallel with his/her **Interviewing** stage once the Service Ticket has been produced.

A Service Ticket is specified by a Case Manager before production of a Case Manager Voucher. See the Remote Operation ID section of the Background tab
for information about this. A Service Ticket for Back Office use is printed by an Operation ID employee monitoring the remote service requests that
appear on the Back Office service request dashboard display. See the Remote Operation ID section of the Background tab for information about this.

## SignalR
Application OPIDDaily uses version 2.4.1 of Microsoft's SignalR framework for real-time broadcast messaging.
SignalR is installed using the NuGet package manager.
SignalR is a push-technology that updates pages without requiring a page refresh.

The SignalR connection hub is defined on file DAL/DailyHub.cs. Invoking a method of DailyHub in the server side code causes a push-notification to
go out to all clients registered to the hub. For example, when a case manager adds a new client to the jqGrid managed by the CaseManagerController
the method CaseManagerController.AddMyClient will invoke method DailyHub.Refresh. This method executes

    hubContext.Clients.All.refreshPage();

which sends a push notification to all registered clients of the hub. The registered clients include all other users logged in any role. They will each
see their view dashbords update without the need to refresh the dashboard. In this way all grids are kept in synch with the grid
to which the client was added.

SignalR is also used as the means to update a progress bar display used when processing external Excel files containing check data.
The code for doing so is found in this excellent [Code Project article](https://www.codeproject.com/articles/1124691/signalr-progress-bar-simple-example-sending-live-d). Application OPIDDaily makes use of this code. On the client side, the progress bar is included as a partial view by the statement

     @Html.Partial("_ModalProgressBar")

See for example file ~/Views/BackOffice/Merge.cshtml. On the server side the progress bar is updated by executing a call like

     DailyHub.SendProgress("Merge in progress...", i, checkCount);

found on file DAL/Merger.cs.

SignalR is also used by both the special users **Client1** and **Client2** to refresh the tablet computer displays used by these users. (See the section
Managing Users on the Database tab.)

## ExcelDataReader
Application OPIDDaily uses version 3.60 of the ExcelDataReader package downloaded using the NuGet package manager. The package is used to read
externally generated Excel data files containing check data that is imported into OPIDDaily. The code that makes use of the package is found in
files Utils/ExcelData.cs and Utils/MyExcelDataReader.cs
