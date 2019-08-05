# C-Live-Projects
Final Projects for the Tech Academy using C#

## Introduction
<br />

In the final 4 weeks of The Tech Academy Software Developer Bootcamp I participated in two sprints using C# and the .NET framework. The first project was a Construction Management portal for a construction company to keep track of jobs, jobsites, and employees. The second project was a more general Employee Management application. In the first sprint, I learned what it was like to join a project that was near completion. This involved making sense of what was already there and utilizing it in code of my own to make it work. It was more focused on fine tuning aspects of the site and meeting the client's needs. For the second sprint, I was part of the beginning of the project and was involved in laying some of the groundwork for the project. We utilized MVC to create the application and I was involved in creating some of the models, adding Google maps to a view, and adding an API to a controller to validate user input. Through these projects I learned much about MVC and how he three parts work together. These were great experiences to see what it is like to work on a project in the real-world and have to work with requirements from the client and having deadlines to meet. I have learned many valuable skills for future projects.

## Construction Management Portal
<br />

### Adding an Alert for Non-Standard Start Times
Each job has a default start time for employees, but some days the job required workers to arrive earlier or later and we wanted to alert employees to that when it happens. 

```
if (shiftTimeNotDefault.HasNonDefaultShiftTime(job.JobNumber))
{
    //Display an alert symbol to let the user know there is something different about this
    //schedule. When hovered over, it will give the user extra information about what kind
    //of changes there are.
    <a id="shiftAlert" href="#" title="Schedule Changes" data-html="true" data-toggle="popover" data-trigger="hover"
       data-placement="top" data-content="@shiftTimeNotDefault.daysWithoutDefaultStartTime(job)">
        <span class="glyphicon glyphicon-alert"></span>
    </a>
}
```

The alert also needed a message to inform them what days were different.

```
public string daysWithoutDefaultStartTime(Job job)
        {
            string message = "The following day(s) has/have different start time(s). <br />";
            if(job.ShiftTimes.Monday != null)
            {
                message += "Monday: " + job.ShiftTimes.Monday + "<br />";
            }
            if (job.ShiftTimes.Tuesday != null)
            {
                message += "Tuesday: " + job.ShiftTimes.Tuesday + "<br />";
            }
            if (job.ShiftTimes.Wednesday != null)
            {
                message += "Wednesday: " + job.ShiftTimes.Wednesday + "<br />";
            }
            if (job.ShiftTimes.Thursday != null)
            {
                message += "Thursday: " + job.ShiftTimes.Thursday + "<br />";
            }
            if (job.ShiftTimes.Friday != null)
            {
                message += "Friday: " + job.ShiftTimes.Friday;
            }
            return message;
        }
```

![alt text](https://github.com/adavidsmith5/C-Live-Projects/blob/master/C%23_alert_message.png)

<br />

## Employee Management Application

<br />

### Restricting account creation by email
We wanted to allow people with the same name to register, but with the way the database is set up was to check for 
username by default. This required a simple switch of using the email for the username and then creating a display
name to store the person's actual name.

```
//This is storing the email as the user name so that two people with the same name
//can still be registered. Instead, it will stop people from registering with the same
//email. The display name will now becomde the person's name who is registering.
var user = new ApplicationUser { UserName = model.Email, Email = model.Email, DisplayName = model.UserName };
var result = await UserManager.CreateAsync(user, model.Password);
```
 Then it was simply a matter of showing an error for when users used the same email.
 
 ```//This just shows the error for the email being duplicated instead of username
 ModelState.AddModelError("", result.Errors.Last());
 ```
 ![alt text](https://github.com/adavidsmith5/C-Live-Projects/blob/master/C%23_email_verification_error.png)
 
 
 ### Showing user name properly in the side menu when a user is logged in
 After making the changes for using the email as the username, the display had to be changed to show the user's display name 
 instead of their email once they were logged in.
 
 ```
 <li>
    @{
        var manager = new UserManager<ApplicationUser>(new UserStore<ApplicationUser>(new ApplicationDbContext()));
        var currentUser = manager.FindById(User.Identity.GetUserId());
     }
     @Html.ActionLink("Hello " + currentUser.DisplayName + "!", "Index", "Manage", routeValues: null, htmlAttributes: new { title = "Manage" }}
 </li>
 ```
 
 ![alt text](https://github.com/adavidsmith5/C-Live-Projects/blob/master/C%23_personalization_menu.png)

<br />

### Setting up a Google map for the location of a jobsite
The next task for me was to create a Google map to show where a jobsite is located for the employee.


```
<script type="text/javascript">

    $(document).ready(function () {
        Initialize();
    });
    var geocoder = new google.maps.Geocoder();
    // Where all the fun happens
    function Initialize() {

        var geocoder = new google.maps.Geocoder();
        var latlng = new google.maps.LatLng(0, 45);
        var mapOptions = {
            zoom: 12,
            center: latlng
        }
        google.maps.visualRefresh = true;
        var map = new google.maps.Map(document.getElementById('jobSite_map_canvas'), mapOptions);
        var address = '@Model.Address @Model.State  @Model.Zip'
        geocoder.geocode({ 'address': address}, function (results, status) {   
            if (status == 'OK') {
                map.setCenter(results[0].geomet```ry.location);
                var marker = new google.maps.Marker({
                    map: map,
                    position: results[0].geometry.location
                });
            } else {
                alert('Geocode was not successful for the following reason: ' + status);
            }
        }); 
    }
</script>
```

### Verifying state, city, and zipcode when an admin is creating a new jobsite
The final story I worked on in the sprint was checking that the admin was entering in correct information when setting up a new jobsite. First, I used the SmartyStreets API, but when we learned that it was very limited in the number of times we could call it per month, I had to find a new API to use for zipcode verification.
```
if (ModelState.IsValid)
            {
                string URL = "https://form-api.com";
                string urlParameters = "/api/geo/country/zip?key=[my-api-key]country=US&zipcode=" + jobsite.Zip;

                HttpClient client = new HttpClient();
                client.BaseAddress = new Uri(URL);

                // Add an Accept header for JSON format.
                client.DefaultRequestHeaders.Accept.Add(
                new MediaTypeWithQualityHeaderValue("application/json"));

                // List data response.
                HttpResponseMessage response = client.GetAsync(urlParameters).Result;  // Blocking call! Program will wait here until a response is received or a timeout occurs.
                if (response.IsSuccessStatusCode)
                {
                    // Take in the response as a string, use deserialize to make it a dynamic object so that the data can be accessed easily. Test to see that the
                    //given state from the form matches the city and state with the given zipcode.
                    var data = response.Content.ReadAsStringAsync().Result;
                    dynamic zipcodeResponse = JsonConvert.DeserializeObject(data);
                    string state = zipcodeResponse.result.state;
                    string city = zipcodeResponse.result.city;

                    //Checking for both city and state matching the given information from the form based on the zipcode.
                    //Because the API is taking the zipcode to get the information, this is how we can verify it.
                    //Two notes: 1. some zip codes can cross state lines. I checked and they usually choose one state over
                    //the other, so for very special circumstances this may need to be dealt with. 2. I am checking for both
                    //city and state, and while the state shouldn't be an issue because it's a dropdown, the city needs to be spelled
                    //correctly
                    if (state == jobsite.State && city.ToLower() == jobsite.Town.ToLower())
                    {
                        db.JobSites.Add(jobsite);
                        db.SaveChanges();
                        return RedirectToAction("Index");
                    }
                    else
                    {
                        ModelState.AddModelError("Zip", "Please double check your city, state, and zipcode.");
                    }

                }
                else
                {
                    Console.WriteLine("{0} ({1})", (int)response.StatusCode, response.ReasonPhrase);
                }
                //Dispose once all HttpClient calls are complete. This is not necessary if the containing object will be disposed of; for example in this case the HttpClient instance will be disposed automatically when the application terminates so the following call is superfluous.
                client.Dispose();
            }

```


