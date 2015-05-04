ms.TocTitle: Calendar tasks in Visual Studio
Title: Common calendar tasks using an Office 365 client library
Description: Use the Office 365 API client libraries to authenticate, get a client object, and interact with events, calendars, and calendar groups.
ms.ContentId: b48ff767-153e-414f-86c8-f89fe0f132be
ms.topic: article (how-tos)

[!INCLUDE [Add the O365API repo styles](../includes/controls/addo365apistyles.xml)]



# Common calendar tasks using an Office 365 client library

**Last modified:** April 29, 2015
    
_**Applies to:** Exchange Online | Office 365_

The Office 365 APIs .NET and Cordova JavaScript client libraries can make it easier interact with the REST APIs. They help manage authentication tokens, simplify the code needed to query
 and consume data, and perform discovery. You can get the libraries in the latest version of [Office 365 API Tools for Visual Studio](http://aka.ms/clientlibrary). Or you can dive right
 in and [try out a starter project for Windows Store Apps](#http://aka.ms/o365-apis-start-windows).

Alternatively, you can use the [Calendar REST APIs](..\api\calendar-rest-operations.md) to interact with Office 365 calendar data.

**Note** If you develop apps for Office 365 in China, see [API endpoints of Office 365 for China](..\api\o365-china-endpoints.md) for the specification of the required URLs.


## Use a client library to access the Calendar API

To access the Calendar API by using the .NET or JavaScript client library, you need to acquire an access token and get the Outlook Services client. Then, you can send async queries to
 interact with calendar data.

[Get an access token](#GetAuthToken) | [Get the Outlook Services client](#GetClient) 


[Get events](#GetEvents) | [Create events](#CreateEvents) | [Update events](#UpdateEvents) | [Delete events](#DeleteEvents) 


[Get calendars](#GetCalendars) | [Create calendars](#CreateCalendars) | [Update calendars](#UpdateCalendars) | [Delete calendars](#DeleteCalendars) 


[Get calendar groups](#GetCalendarGroups) | [Create calendar groups](#CreateCalendarGroups) | [Update calendar groups](#UpdateCalendarGroups) | [Delete calendar groups](#DeleteCalendarGroups) 



<a name="GetAuthToken"> </a>
## Get an access token

Acquire the access token used for authentication. The client ID and authorization URI are assigned when you register your app with Microsoft Azure Active Directory. 

**Note** This pattern applies only to code that runs on Windows devices. In a Windows Store App, values for ClientID and AuthorizationUri are added to your project's App.xaml file when you register your application.
 AuthorizationUri is used as the host name for the CommonAuthority variable.

<!-- BEGINSECTION class="tabbedCodeSnippets" -->

```cs
// Properties of the native client app. Get the ClientId from the resources section of the App.xaml file.
public const string ClientID = Application.Current.Resources["ida:ClientID"] as String; 
// Get the _returnUri from app settings.
public static Uri _returnUri = Windows.Security.Authentication.Web.WebAuthenticationBroker.GetCurrentApplicationCallbackUri();

// Properties used to communicate with a Windows Azure AD tenant.  
public const string CommonAuthority = "https://login.windows.net/Common"; 
public const string DiscoveryResourceId = "https://api.office.com/discovery/"; 

//Store authority in application data so that it isn't tied to the lifetime of the access token.
private static ApplicationDataContainer _settings = ApplicationData.Current.LocalSettings;
private static string LastAuthority
{
    get
    {
        if (_settings.Values.ContainsKey("LastAuthority") && _settings.Values["LastAuthority"] != null)
        {
            return _settings.Values["LastAuthority"].ToString();
        }
        else
        {
            return string.Empty;
        }

    }

    set
    {
        _settings.Values["LastAuthority"] = value;
    }
}

public static AuthenticationContext _authenticationContext { get; set; } 
private static async Task<string> GetTokenHelperAsync(AuthenticationContext context, string resourceId)
{
    string accessToken = null;
    AuthenticationResult result = null;

    result = await context.AcquireTokenAsync(resourceId, ClientID, _returnUri);

    accessToken = result.AccessToken;
    //Store authority in application data.
    _settings.Values["LastAuthority"] = context.Authority;

    return accessToken;
}
```
```javascript 
var authContext;
var authToken; // for use with creating an outlookClient later.
authContext = new O365Auth.Context();
authContext.getIdToken("https://outlook.office365.com/")
   .then((function (token) {
       authToken = token;
	   // The auth token also carries additional information. For example:	
       userName = token.givenName + " " + token.familyName;
   }).bind(this), function (reason) {
       console.log('Failed to login. Error = ' + reason.message);
   });

```
<!-- ENDSECTION -->

<a name="GetClient"> </a>
## Get the Outlook Services client


Get the **OutlookServicesClient** object. You can call this code from other methods that use the Outlook Services client.

<!-- BEGINSECTION class="tabbedCodeSnippets" -->

```cs
public static async Task<OutlookServicesClient> CreateOutlookClientAsync(string capability)
{
    try
    {
        //First, look for the authority used during the last authentication.
        //If that value is not populated, use CommonAuthority.
        string authority = null;
        if (String.IsNullOrEmpty(LastAuthority))
        {
            authority = CommonAuthority;
        }
        else
        {
            authority = LastAuthority;
        }
        // Create an AuthenticationContext using this authority.
        _authenticationContext = new AuthenticationContext(authority);
        
        //See the Discovery Service Sample (https://github.com/OfficeDev/Office365-Discovery-Service-Sample)
        //for an approach that improves performance by storing the discovery service information in a cache.
        DiscoveryClient discoveryClient = new DiscoveryClient(
            async () => await GetTokenHelperAsync(_authenticationContext, DiscoveryResourceId));

        // Get the specified capability ("Calendar").
        CapabilityDiscoveryResult result = 
            await discoveryClient.DiscoverCapabilityAsync(capability);
        var client = new OutlookServicesClient(
            result.ServiceEndpointUri,
            async () => 
                await GetTokenHelperAsync(_authenticationContext, result.ServiceResourceId));
        return client;
    }
    catch (Exception)
    {
        if (_authenticationContext != null && _authenticationContext.TokenCache != null)
        _authenticationContext.TokenCache.Clear();
        return null;
    }
}
```
```javascript 
// Once the authToken has been acquired, create an outlookClient. One place to do this is inside of the
//    ".then" function callback of authContext.getIdToken(...) above.
var outlookClient = new Microsoft.OutlookServices.Client('https://outlook.office365.com/api/v1.0', authToken.getAccessTokenFn('https://outlook.office365.com'));
```
<!-- ENDSECTION -->


<a name="GetEvents"></a>
## Get events from the user's calendar

Get the events from the user's default calendar. To get the events from a different calendar, call the calendar's **Events** property.

Example: `outlookClient.Me.Calendars[calendarId].Events.ExecuteAsync()`

To get a particular event, you can specify the event ID as the index of the **Events** collection or use the **GetById** method.

**Note** Event collections support query expressions such as **Select**, **OrderBy**, and **Take**.

This example calls the method that [creates the Outlook Services client](#GetClient).

<!-- BEGINSECTION class="tabbedCodeSnippets" data-resources="OutlookServices.Calendar" -->

```cs-i
var outlookClient = await CreateOutlookClientAsync("Calendar");
var events = await outlookClient.Me.Events
  .Take(10)
  .ExecuteAsync();
 
foreach(var calendarEvent in events.CurrentPage)
{
  System.Diagnostics.Debug.WriteLine("Event '{0}'.", calendarEvent.Subject);
}
 
```

```javascript-i
outlookClient.me.events.getEvents().fetch().then(function (result) {
    result.currentPage.forEach(function (event) {
console.log('Event "' + event.subject + '"')
    });
}, function(error) {
    console.log(error);
});
```

<!-- ENDSECTION -->


This call returns the event series, not the individual expanded instances for recurring events (such as a weekly team meeting).

Querying event instances is currently not supported in the client library. You can use the REST API to query the **CalendarView** property on the
 [Calendar](..\api\calendar-rest-operations.md#CalendarResource) resource or the **Instances** property on the [Event](..\api\calendar-rest-operations.md#EventResource) resource:
 
```httprequest
GET ../me/events/{event_id}/instances?startDateTime=â€™2014-01-01T00:00:00Zâ€™&endDateTime=â€™2014-12-31T00:00:00Zâ€™
```
 
<!--Update c# example to get instance-->
<!--Update js example and remove note when this works in js-->


<a name="CreateEvents"></a>
## Create calendar events

Create an event. To add an event to a different calendar, use the **Events** property of the destination calendar.

Example: `await client.Me.Calendars["AQMkADE3..."].Events.AddEventAsync(newEvent);`

This example assumes you already [got the Outlook Services client](#GetClient).

<!-- BEGINSECTION class="tabbedCodeSnippets" -->

```cs
// Create a location for the event
Location location = new Location 
{ 
    DisplayName = "Water cooler" 
}; 

// Create a description for the event	
ItemBody body = new ItemBody 
{ 
    Content = "Status updates, blocking issues, and next steps", 
    ContentType = BodyType.Text 
}; 
	
// Create an attendee for the event	
Attendee[] attendees =  
{  
    new Attendee  
    {  
        Type = AttendeeType.Required,  
        EmailAddress = new EmailAddress  
        {  
            Address = "katiej@a830edad9050849NDA1.onmicrosoft.com"  
        },  
    },  
}; 
	
// Create the event object
Event newEvent = new Event 
{ 
    Subject = "Sync up", 
    Location = location, 
    Attendees = attendees, 
    Start = new DateTimeOffset(new DateTime(2014, 12, 1, 9, 30, 0)), 
    End = new DateTimeOffset(new DateTime(2014, 12, 1, 10, 0, 0)), 
    Body = body 
}; 
	
// Add the event to the default calendar
await client.Me.Events.AddEventAsync(newEvent);

// Get the event ID.
string eventId = newEvent.Id;
```
```javascript
        var event = new Microsoft.OutlookServices.Event();
        event.subject = 'Your Subject';
        event.start = new Date("October 30, 2014 11:13:00").toISOString();
        event.end = new Date("October 30, 2014 12:13:00").toISOString();

        // Body
        event.body = new Microsoft.OutlookServices.ItemBody();
        event.body.content = 'Body Content';
        event.body.contentType = Microsoft.OutlookServices.BodyType.Text;

        // Location
        event.location = new Microsoft.OutlookServices.Location();
        event.location.displayName = 'Location';

        // Attendee
        var attendee1 = new Microsoft.OutlookServices.Attendee();
        var emailAddress1 = new Microsoft.OutlookServices.EmailAddress();
        emailAddress1.name = "Katie Jordan";
        emailAddress1.address = "katiej@a830edad9050849NDA1.onmicrosoft.com";

        attendee1.emailAddress = emailAddress1;

        event.attendees.push(attendee1);
        
        outlookClient.me.calendar.events.addEvent(event)
        .then(function (response) {
            console.log(response._Id);
        });    

```
<!-- ENDSECTION -->


<a name="UpdateEvents"></a>
## Update calendar events

Change an event.

This example assumes you already [got the Outlook Services client](#GetClient) and [got the event ID](#GetEvents).

<!-- BEGINSECTION class="tabbedCodeSnippets" -->

```cs
// Get an existing event by ID
IEvent eventToUpdate = await client.Me.Events[eventId].ExecuteAsync();

// Add attendees
eventToUpdate.Attendees.Add(new Attendee
{
    Type = AttendeeType.Required,
    EmailAddress = new EmailAddress
    {
        Address = "garthf@a830edad9050849NDA1.onmicrosoft.com",
    },
});

// Make other changes
eventToUpdate.Start = new DateTimeOffset(new DateTime(2014, 12, 1, 14, 30, 0));
eventToUpdate.End = new DateTimeOffset(new DateTime(2014, 12, 1, 15, 0, 0));
eventToUpdate.Subject = "New event name";
	
// Commit all changes to the event
await eventToUpdate.UpdateAsync();

// Get an updated property.
string newEventName = eventToUpdate.Subject;
```

<!-- ENDSECTION -->


You can define multiple updates client-side and send the requests all at once (batch them) by using the following pattern:
1. Call `UpdateAsync(true)` for each entity you want to update. Specifying `true` registers the updates locally on the client but doesn't post them to the server.
2. Call `client.Context.SaveChangesAsync()` to post all updates that are registered locally.


<a name="DeleteEvents"></a>
## Delete calendar events

Move an event to the Deleted Items folder.

This example assumes you already [got the Outlook Services client](#GetClient) and [got the event ID](#GetEvents).

<!-- BEGINSECTION class="tabbedCodeSnippets" -->

```cs
// Get an existing event by ID
IEvent eventToDelete = await client.Me.Events[eventId].ExecuteAsync();

//Delete the event
await eventToDelete.DeleteAsync();
```

<!-- ENDSECTION -->


<a name="GetCalendars"> </a>
## Get calendars

Get the user's calendars. To get the user's default calendar, use the `client.Me.Calendar` shortcut property. To get a different calendar, specify the calendar ID
 as the index of the  **Calendars** collection or use the **GetById** method.

Example: `client.Me.Calendars[calendarId].ExecuteAsync()`

**Note** Calendar collections support query expressions such as **Select**, **OrderBy**, and **Take**.

This example calls the method that [gets the Outlook Services client](#GetClient).

<!-- BEGINSECTION class="tabbedCodeSnippets" data-resources="OutlookServices.Calendar" -->

```cs-i
var outlookClient = await CreateOutlookClientAsync("Calendar");
var calendars = await outlookClient.Me.Calendars
  .Take(10)
  .ExecuteAsync();

foreach(var calendar in calendars.CurrentPage)
{
  System.Diagnostics.Debug.WriteLine("Calendar '{0}'.", calendar.Name);
}

```
```javascript-i
outlookClient.me.calendars.getCalendars().fetchAll(100).then(function(result) {
    result.forEach(function (calendar) {
        console.log('Calendar "' + calendar.name + '", URL ' + calendar.path)
    });
}, function(error) {
    console.log(error);
});
```

<!-- ENDSECTION -->


<a name="CreateCalendars"> </a>
## Create calendars

Create a calendar. See [Create events](#CreateEvents) for an example of how to create an event.

This example assumes you already [got the Outlook Services client](#GetClient).

<!-- BEGINSECTION class="tabbedCodeSnippets" -->

```cs
Calendar newCal = new Calendar
{
    Name = "Personal"
};

// Add the calendar to the Calendars collection
await client.Me.Calendars.AddCalendarAsync(newCal);

// Get the ID of the calendar
string calendarId = newCal.Id;
```

<!-- ENDSECTION -->


<a name="UpdateCalendars"> </a>
## Update calendars

Change the name of a calendar. **Name** is the only writeable property for a calendar.

This example assumes you already [got the Outlook Services client](#GetClient) and [got the calendar ID](#GetCalendars).

<!-- BEGINSECTION class="tabbedCodeSnippets" -->

```cs
// Get an existing calendar by ID
ICalendar calendarToUpdate = await client.Me.Calendars[calendarId].ExecuteAsync();
calendarToUpdate.Name = "Family";

// Commit the change
await calendarToUpdate.UpdateAsync();

// Get the updated property
string newCalendarName = calendarToUpdate.Name;
```

<!-- ENDSECTION -->


You can define multiple updates client-side and send the requests all at once (batch them) by using the following pattern:
1. Call `UpdateAsync(true)` for each entity you want to update. Specifying `true` registers the updates locally on the client but doesn't post them to the server.
2. Call `client.Context.SaveChangesAsync()` to post all updates that are registered locally.


<a name="DeleteCalendars"> </a>
## Delete calendars

Delete a calendar.

This example assumes you already [got the Outlook Services client](#GetClient) and [got the calendar ID](#GetCalendars).

<!-- BEGINSECTION class="tabbedCodeSnippets" -->

```cs
// Get an existing calendar by ID
ICalendar calendarToDelete = await client.Me.Calendars[calendarId].ExecuteAsync();
	
// Delete the calendar
await calendarToDelete.DeleteAsync(false);
```

<!-- ENDSECTION -->


<a name="GetCalendarGroups"> </a>
## Get calendar groups

Get a user's calendar groups. To get a different calendar group, specify the calendar group ID as the index of the **CalendarGroups** collection or use the **GetById** method.

Example: `client.Me.CalendarGroups[calendarGroupId].ExecuteAsync()`

**Note** Calendar group collections support query expressions such as **Select**, **OrderBy**, and **Take**.

This example assumes you already [got the Outlook Services client](#GetClient).

<!-- BEGINSECTION class="tabbedCodeSnippets" -->

```cs
IPagedCollection<ICalendarGroup> calendarGroupsResults = await client.Me.CalendarGroups.ExecuteAsync();

// Get the ID of the first calendar group
string groupId = calendarGroupsResults.CurrentPage[0].Id;
```

<!-- ENDSECTION -->


<a name="CreateCalendarGroups"> </a>
## Create calendar groups

Create a calendar group.

This example assumes you already [got the Outlook Services client](#GetClient).

<!-- BEGINSECTION class="tabbedCodeSnippets" -->

```cs
CalendarGroup newCalendarGroup = new CalendarGroup
{
    Name = "Business"
};

// Add it to the CalendarGroups collection
await client.Me.CalendarGroups.AddCalendarGroupAsync(newCalendarGroup);

// Get the ID of the calendar group
string calendarGroupId = newCalendarGroup.Id;
```

<!-- ENDSECTION -->


<a name="UpdateCalendarGroups"> </a>
## Update calendar groups

Change the name of a calendar group. **Name** is the only writeable property for a calendar group.

This example assumes you already [got the Outlook Services client](#GetClient) and [got the calendar group ID](#GetCalendarGroups).

<!-- BEGINSECTION class="tabbedCodeSnippets" -->

```cs
// Get an existing calendar group by ID
ICalendarGroup groupToUpdate = await client.Me.CalendarGroups[groupId].ExecuteAsync();
groupToUpdate.Name = "Contoso";

// Commit the change
await groupToUpdate.UpdateAsync();

// Get the updated property
string newCalendarGroupName = groupToUpdate.Name;
```

<!-- ENDSECTION -->


You can define multiple updates client-side and send the requests all at once (batch them) by using the following pattern:
1. Call `UpdateAsync(true)` for each entity you want to update. Specifying `true` registers the updates locally on the client but doesn't post them to the server.
2. Call `client.Context.SaveChangesAsync()` to post all updates that are registered locally.


<a name="DeleteCalendarGroups"> </a>
## Delete calendar groups

Delete a calendar group.

This example assumes you already [got the Outlook Services client](#GetClient) and [got the calendar group ID](#GetCalendarGroups).

<!-- BEGINSECTION class="tabbedCodeSnippets" -->

```cs
// Get an existing calendar group by ID
ICalendarGroup groupToDelete = await client.Me.CalendarGroups[groupId].ExecuteAsync();

// Delete the group
await groupToDelete.DeleteAsync(); 
```

<!-- ENDSECTION -->


## Additional resources

- [Overview of developing on the Office 365 platform](..\howto\platform-development-overview.md)
    
- [Office 365 app authentication and resource authorization](..\howto\common-app-authentication-tasks.md)

- [Office 365 API starter projects, code samples, and videos](..\howto\starter-projects-and-code-samples.md)
  
- [Common mail tasks using an Office 365 client library](..\howto\common-mail-tasks-client-library.md)

- [Common contacts tasks using an Office 365 client library](..\howto\common-contacts-tasks-client-library.md)

- [Common file tasks using an Office 365 client library](..\howto\common-file-tasks-client-library.md)

- [Calendar REST APIs operations reference](..\api\calendar-rest-operations.md)
