# Code-Summary-C-Sharp
##Introduction

After the two week live Python project at The Tech Academy, I got to jump into my next two week live project,
the C# live project. The development project was writing software to manage a collection of construction jobs. As 
with the live Python project, I had the opportunity to work with a team in an organized fashion to make and 
meet goals in an effective manner. The nature of the project differed from the Python project, which was a welcomed
change. I was able to hone my skills working with Visual Studio, ASP.Net and the entity framework. As with the
Python project, the C# project was already built out to some extent. This allowed me the ability to study the 
design of an already existing project. Stepping through each line of code is a fantastic way to learn, reproduce
and understand the design of a particular code.

Onto the contributions I made to the project, the challenges I faced and what I learned from them. 

###Week 1

#####The Problem #1

My first week and task started off on a very interesting note. For context, the software is set up in such a way that 
the admin would set and send out a user name, role in the company and registration code that the user would receive and 
use to sign up and register as a user. The user could not sign up or register without first receiving a registration 
code from an admin. My first task was to find the cause of a bug that was not throwing an error and had no apparent cause
The bug in question came about after the user successfully registered their account. They would be logged into the 
software with the username they were given, but without the main display dashboard or any of the privileges that came 
along with their company role.

As I touched on into the intro, the first thing I did was put a break point at the start of the program and step on 
though to the other side... so to speak. 

Take note on the cshtml page below. While stepping throught the program I noticed Each of the "@if (User.IsInRole)" 
conditional statements was not even being hit. These are the conditional statements that determine what html content to 
generate. 


    **@if (User.IsInRole("Admin") || User.IsInRole("Manager"))**
    {
    <div class="row ">
        <div class="panel-group top-buffer col-lg-9 " id="accordion" role="tablist" aria-multiselectable="true">
            <div class="panel panel-default ">
                <div class="panel-heading" role="tab" id="headingOne">
                    <h2 class="panel-title">
                        Schedule
                        **@if (User.IsInRole("Admin"))**
                        {

                            @Html.ActionLink("Create New", "Create", "Schedules", null, new { @class = "createNew" })

                        }
                    </h2>

                </div>
                <div id="collapseSchedule" class="panel-collapse collapse in" role="tabpanel" aria-labelledby="headingOne">
                    <div class="panel-body overflow-auto">
                        @{ Html.RenderAction("MasterSchedulePartial"); }
                    </div>
                </div>
            </div>
            <div class="panel panel-default">
                <div class="panel-heading" role="tab" id="headingThree">
                    <h2 class="panel-title">
                        Company News

                        **@if (User.IsInRole("Admin"))**
                        {
                            // calls create company news partial view modal
                            @Html.ActionLink("Create New", "CompanyNewsModal", null, new { @class = "createNew", data_target = "#myModal", data_toggle = "modal" })
                        }
                    </h2>
                </div>
                <div id="collapseCompanyNews" class="panel-collapse collapse show" role="tabpanel" aria-labelledby="headingThree">
                    <div class="panel-body">
                        @Html.Partial("_CompanyNews", new ApplicationDbContext().CompanyNews.AsEnumerable(), new ViewDataDictionary { { "UserList", ViewBag.UserList } })
                    </div>
                </div>
            </div>
            <div class="panel panel-default">
                <div class="panel-heading" role="tab" id="headingTwo">
                    <h2 class="panel-title">
                        User List
                        **@if (User.IsInRole("Admin"))**
                        {
                            @Html.ActionLink("Create New", "Create", "CreateUserRequests", null, new { @class = "createNew" })//update with add user function
                        }
                    </h2>
                </div>
                <div id="collapseUserList" class="panel-collapse collapse show" role="tabpanel" aria-labelledby="headingTwo">
                    <div class="panel-body">
                        @Html.Action("_UserListPartial", "Account")
                    </div>
                </div>
            </div>
        </div>
    </div>

#####The Solution #1

After noticing the user's role was not being utilized I dove into the corresponding controller to see where the issue was.
Below is the finished code after the problem was solved. I have used 3 asterisks to highlight the code relating to the 
problem and below is a screenshot of the side by side, before and after code changes. I noticed that the line of code 
that was calling the function to actually assigning the user role to the user was being called after the function that
signs the user in. Because of this, the UI that was being generated was being done so without any user role. The fix
was simple, to place "UserManager" above "SignInManager". Despite the solution being simple, the path to that solution
took an attention to detail that was not as straight forward. I find this to be a reoccurring theme in software development
and it is one that I rather enjoy examining.

     // POST: /Account/Register
        [HttpPost]
        [AllowAnonymous]
        [ValidateAntiForgeryToken]
        public async Task<ActionResult> Register(RegisterViewModel model)
        {
            if (ModelState.IsValid)
            {
                var user = new ApplicationUser
                {
                    UserName = model.UserName,
                    Email = model.Email,
                    PhoneNumber = model.PhoneNumber,
                    FName = model.FName,
                    LName = model.LName,
                    // Get user role
                    UserRole = model.UserRoles
                };
                var identityResult = await UserManager.CreateAsync(user, model.Password);
                if (identityResult.Succeeded)
                {
                    // assign user to role
                    ***await UserManager.AddToRoleAsync(user.Id, user.UserRole);***

                    ***await SignInManager.SignInAsync(user, false, false);***

                    // For more information on how to enable account confirmation and password reset please visit https://go.microsoft.com/fwlink/?LinkID=320771
                    // Send an email with this link
                    // string code = await UserManager.GenerateEmailConfirmationTokenAsync(user.Id);
                    // var callbackUrl = Url.Action("ConfirmEmail", "Account", new { userId = user.Id, code = code }, protocol: Request.Url.Scheme);
                    // await UserManager.SendEmailAsync(user.Id, "Confirm your account", "Please confirm your account by clicking <a href=\"" + callbackUrl + "\">here</a>");

                    // Delete CreateUserRequest
                    using (db)
                    {
                        var createUserRequest = db.CreateUserRequests.First(x => x.UserName == user.UserName);
                        if (createUserRequest != null) db.CreateUserRequests.Remove(createUserRequest);
                        db.SaveChanges();
                    }

                    return RedirectToAction("Index", "Home");
                }
            }
            // If we got this far, something failed, redisplay form
            return View(model);
        }

#####The Problem #2

The next story was straight forward one. I had to add authentication and functionality to restrict the views available
to non-registered users. In addition to this I also wanted to remove the views to be restricted from the navigation bar.

#####The Solution #2

First the nav bar! I got lucky here as the code was already set up in such a way where all the options I wanted to 
limit fell in the same code block. All I had to do was place a conditional statement that I already had experience with 
from the previous story. You'll notice this statement in the code below. I have place 3 asterisks around the code that I
added achieve this. The only limitation with this implementation is that it filters the nav bar options through user role,
so If there is a new user role added to the program the statement will have to be updated to include the new role.

    <div class="collapse navbar-collapse ei-navbar" id="nav-links bs-slide-dropdown">
                ***@if (User.IsInRole("Admin") || User.IsInRole("Manager") || User.IsInRole("Employee"))***
                {
                    <ul class="nav navbar-nav ml-auto">
                        <li>@Html.ActionLink("Dashboard", "Index", "Dashboard", null, new { Style = "color: White;" })</li>
                        @*Admin only*@
                        @if (User.IsInRole("Admin") || User.IsInRole("Manager"))
                        {
                            <li class="dropdown">
                                <a href="#" class="dropdown-toggle" data-toggle="dropdown" id="users">Users</a>
                                <ul class="dropdown-menu" role="menu">
                                    <li>
                                        <a class="dropdown-item">@Html.ActionLink("Unregistered Users", "Index", "CreateUserRequests")</a>
                                    </li>
                                    <li>
                                        <a class="dropdown-item">@Html.ActionLink("All Users", "AllUsers", "CreateUserRequests")</a>
                                    </li>
                                    <li>
                                        <a class="dropdown-item">@Html.ActionLink("Create New", "Create", "CreateUserRequests")</a>
                                    </li>
                                </ul>
                            </li>
                        }
                        <li class="dropdown">
                            <a href="#" class="dropdown-toggle" data-toggle="dropdown" id="jobs">Jobs</a>
                            <ul class="dropdown-menu" role="menu">
                                <li>
                                    <a class="dropdown-item">@Html.ActionLink("List", "Index", "Jobs")</a>
                                </li>
                                @*Admin only*@
                                @if (User.IsInRole("Admin") || User.IsInRole("Manager"))
                                {
                                    <li>
                                        <a class="dropdown-item">@Html.ActionLink("Create New", "Create", "Jobs")</a>
                                    </li>
                                }
                            </ul>
                        </li>
                        <li class="dropdown">
                            <a href="#" class="dropdown-toggle" data-toggle="dropdown" id="schedule">Schedule</a>
                            <ul class="dropdown-menu" role="menu">
                                @*Admin only*@
                                @if (User.IsInRole("Admin") || User.IsInRole("Manager"))
                                {
                                    @*May need this link later, but not for now
                                            <li>
                                            <a class="dropdown-item">@Html.ActionLink("Edit", "Edit", "Schedules")</a>
                                        </li>*@
                                    <li>
                                        <a class="dropdown-item">@Html.ActionLink("Add New Item", "Create", "Schedules")</a>
                                    </li>
                                    <li>
                                        <a class="dropdown-item">@Html.ActionLink("Create Weekly Schedule", "MasterScheduleEdit", "Schedules")</a>
                                    </li>
                                }
                                <li>
                                    <a class="dropdown-item">@Html.ActionLink("View All", "Index", "Schedules")</a>
                                </li>
                            </ul>
                        </li>
                        <li class="dropdown">
                            <a href="#" class="dropdown-toggle" data-toggle="dropdown" id="company">Company</a>
                            <ul class="dropdown-menu" role="menu">
                                <li>
                                    <a class="dropdown-item">@Html.ActionLink("News", "Index", "CompanyNews")</a>
                                </li>
                                @*Admin only*@
                                @if (User.IsInRole("Admin") || User.IsInRole("Manager"))
                                {
                                    <li>
                                        <a class="dropdown-item">@Html.ActionLink("Create New", "Create", "CompanyNews")</a>
                                    </li>
                                }

                            </ul>
                        </li>
                    </ul>
                }
                @Html.Partial("_LoginPartial")
                
            </div>
            
As I touched on earlier, the next part of the story had a simple solution in the end but getting to that place took me
on a bit of a wild goose chase. Even though an unregistered user could not see or click the path to the views on the nav 
bar, they could still type the url path in the browser and access the views that way. Attached to the story was one   
suggestion for how to do this. I learned in version 7 of Microsoft's IIS, you could add a Web.config file to a folder
that would invoke UrlAuthorizationModule on all views in that folder. Unfortunately, I was unable to complete the task
using this method. I was unable to find any implementation of this method anywhere else outside of the article that was
linked in the description for this story on Azure. I suspect this is because that feature is so new that people haven't
had the opportunity to work with it that much but I'd like to return to it in the future once there is a more thorough
description of it's implantation. Hitting a wall, I decided to return to the code that we had already writen for a lead.
In hindsight, the solution to this problem was humorously simple. While it might not be as efficient as the Web.config 
solution would have been, it was arguably more simple. I was able to solve this issue on the controller side of the 
program by finding an "[Authorize]" tag in the AccountController I have showed below.
 
      [Authorize]
    public class AccountController : Controller
    {
        private ApplicationDbContext db = new ApplicationDbContext();
        private ApplicationSignInManager _signInManager;
        private ApplicationUserManager _userManager;

This was the exact tag that I had to add to each individual controller method that returned an html page that I wanted to 
restrict unregistered users from entering. Although I have not included all of the changes I made to each controller,
I have included an example below of the "Jobs" controller that I modified. That code is highlighted with 3 asterisks.
 
        ***[Authorize] //used to restrict view of unregistered users***
    public ActionResult Index()
    {
        return View(db.Jobs.OrderBy(i => i.JobNumber).ToList());
    } 
 
Moral of the story, The solution to a problem is usually closer then you think it is. Such is life that you travel a 
great distance only to find the answer was right next to you the whole time.

#####The Problem #3

My next story didn't require as much detective work as the first two. The assignment was to add a view that encapsulated
both users who were already resisted and users who had a registration code sent to them, but had not completed the 
registration form on their end. This was a fairly simple task because all of the code that I needed was already writen
in other parts of the program. The only wall I hit was that I couldn't use two models to populate the same view page.

  

#####The Solution #3

Below is the view that I wound up creating. You will see that I highlighted the CreateUserRequest model at the top of the
page, with 3 asterisks, that I used for the users that are not fully registered. Below that, I highlighted the view
"_UserListPartial" that created the list of registered users. Both of these lists had to maintain all of the functionality
for removing, editing and creating current or new users. 

      ***@model IEnumerable<ConstructionNew.Models.CreateUserRequest>***
    @{
        ViewBag.Title = "AllUsers";
    }
    <h2>All Users</h2>
    <div class="panel panel-default">
        <div class="panel-heading" role="tab" id="headingTwo">
            <h2 class="panel-title">
                Registered User List
            </h2>
        </div>
        <div id="collapseUserList" class="panel-collapse collapse show" role="tabpanel" aria-labelledby="headingTwo">
            <div class="panel-body">
                ***@Html.Action("_UserListPartial", "Account")***
            </div>
        </div>
    </div>
    <div class="panel panel-default">
        <div class="panel-heading" role="tab" id="headingTwo">
            <h2 class="panel-title">
                Unregistered User List
            </h2>
        </div>
        <div id="collapseUserList" class="panel-collapse collapse show" role="tabpanel" aria-labelledby="headingTwo">
            <div class="panel-body">
                <table class="table">
                    <tr>
                        <th>
                            @Html.DisplayNameFor(model => model.UserName)
                        </th>
                        <th>
                            @Html.DisplayNameFor(model => model.ConfirmationCode)
                        </th>
                        <th>
                            @Html.DisplayNameFor(model => model.UserRoles)
                        </th>
                        <th></th>
                    </tr>
                   @foreach (var item in Model)
                    {
                    <tr>
                        <td>
                            @Html.DisplayFor(modelItem => item.UserName)
                        </td>
                        <td>
                            @Html.DisplayFor(modelItem => item.ConfirmationCode)
                        </td>
                        <td>
                            @Html.DisplayFor(modelItem => item.UserRoles)
                        </td>
                        <td>
                            @*@Html.ActionLink("Edit", "Edit", new { id=item.UserCreationRequestId }) |
                            @Html.ActionLink("Details", "Details", new { id=item.UserCreationRequestId }) |*@
                            @Html.ActionLink("Delete", "Delete", new { id = item.UserCreationRequestId })
                        </td>
                    </tr>
                    }
                </table>
            </div>
        </div>
    </div>

After this view was created I had to adjust both the "CreateUserRequestsController.cs" and the "_Layout.cshtml" page
to reflect the creation of this new view page. While this story was simple, it was very good for practicing getting into
a state of work flow. Having certain tasks become second nature allows me to balance time and energy when I hit a wall or
a task that requires more critical thinking. 

###Week 2

#####The Problem #4

My first story for the second week of the C# live project was an interesting one. The admin has the ability to edit the 
schedule for the week via the "edit" view in the schedule tab. My task was to have the current values of the selected 
schedule being edited to populate each field shown on the edit page as defaults. At first I tried solving this problem
on the view side of things. I found some limitations in the .Net framework that made this difficult. I had to turn my 
focus to the controller side to look for a solution. I noticed we were not passing the whole model to the view but rather
using 2 different functions to select job sites and the users. It is here where I found the path to success.

#####The Solution #4

In the code snippets below you will find both the aforementioned "GetJobSites" and "GetUsers" functions. After a bit of 
research and learning about SelectListItem, I found that I could use a the "Selected" variable to choose an item to
be selected as a variable. You'll notice in both functions below, I added a parameter to both the "GetJobSites" and the
"GetUser" functions. The are highlighted with 3 asterisks. This is so we can pass in the ID of the user and job from the
selected Schedule. You'll notice we use the argument passed in the parameters in the aforementioned "Selected" variable.
Both those selected statements also are highlighted with 3 asterisks. The default value of both of the arguments to be
passed into the function is set to "null" in the event we do not need a default because these functions are used elsewhere
in the program. You will also see the comments I made after the function was complete.

    // Added selectedJobId that is set to null to be passed in when a default field value is needed
    private IEnumerable<SelectListItem> GetJobSites(***string selectedJobId = null***)  
    {
        // Create a list of JobTitle/JobId pairs to pass to the view with viewbag.jobsite
        // The DropDownList will display the JobTitle list and
        // The POST data receives the corresponding JobId

        return db.Jobs.OrderBy(o => o.JobNumber).Select(m => new SelectListItem()
        {
            Value = m.JobId.ToString(),
            Text = m.JobTitle,
    // Add Select variable that holds the Id of the current selected item that gets passed when the corresponding edit 
    //button is clicked
            ***Selected = selectedJobId == m.JobId.ToString() 
        });
    }

    // Added selectedUserId that is set to null to be passed in when a default field value is needed
    private IEnumerable<SelectListItem> GetUsers(***string selectedUserId = null***)
    {
        // Create a list of UserName/Id pairs to pass to the view with viewbag.person
        // The DropDownList will display the UserName list and
        // The POST data receives the corresponding Id

        return db.Users.Where(x => x.UserName != "SiteAdmin").OrderBy(o => o.UserName).Select(m => new SelectListItem()
        {
            Value = m.Id,
            Text = (m.FName + " " + m.LName),
        // Add Select variable that holds the Id of the current selected item that gets passed when the corresponding 
        //edit button is clicked
            ***Selected = selectedUserId == m.Id*** 
        });
    }

After adding the parameters and "Selected" variable to both functions, the next step was to pass in the relative IDs to
both functions that we are calling to be passed into the viewbag. Again in the code below I have highlighted that with
3 asterisks. The specific schedule we are using to grab both the Person Id and Job Id is being passed in as an argument
in the "ActionResult Edit" function as a parameter. I have highlighted that with 3 plus signs. The argument that gets
passed in is a result of the option that gets clicked on the view side.

     // GET: Schedules/Edit/5
            [Authorize(Roles = "Admin")]
            public ActionResult Edit +++(Guid? id)+++
            {
                if (id == null)
                {
                    return new HttpStatusCodeResult(HttpStatusCode.BadRequest);
                }
                Schedule schedule = db.Schedules.Find(id);
                if (schedule == null)
                {
                    return HttpNotFound();
                }
            // Pass in Job Id as string to GetJobsites method to retrieve selected object to be used as default selection. 
                ***ViewBag.jobsite = GetJobSites(schedule.Job.JobId.ToString());*** 
            // Pass in Person Id to GetUsers method to retrieve selected object to be used as default selection. 
                ***ViewBag.person = GetUsers(schedule.Person.Id);***
                return View(schedule);
            }
           
That took care of setting the defaults for the job and user field but I still had to find a way to set the dates to the
default value of the selected schedule to be edited. This was a bit more simple then the other fields. Because the 
schedule in question was already being passed into the edit view, the only thing I had to do was change the display
format in the Schedule model. 

       [Display(Name = "Start Date")]        
            [DisplayFormat(ApplyFormatInEditMode = true, DataFormatString = "{0:yyyy-MM-dd}")]
            public DateTime StartDate { get; set; }
            [Display(Name = "End Date")]
            [DataType(DataType.Date)]        
            [DisplayFormat(ApplyFormatInEditMode = true, DataFormatString = "{0:yyyy-MM-dd}")]
            
Below I have pasted a snippet of the schedule edit view to wrap up the retrospective of this particular problem. You will
see where I have utilize the both ViewBag.person and ViewBag.jobsite to create the SelectList.            
            
      <div class="form-group">
                @Html.LabelFor(model => model.Person.UserName, htmlAttributes: new { @class = "control-label col-md-2" })
                <div class="col-md-10">
                    @Html.DropDownList("Person", ViewBag.person as SelectList, "--Select Person--", new { style = "height:40px; width:250px;" })
                </div>
            </div>
            <div class="form-group">
                @Html.LabelFor(model => model.Job.JobTitle, htmlAttributes: new { @class = "control-label col-md-2" })
                <div class="col-md-10">
                    @Html.DropDownList("Jobsite", ViewBag.jobsite as SelectList, "--Select Job--", new { style = "height:40px; width:250px;" })
                </div>
            </div>
            <div class="form-group">
                @Html.LabelFor(model => model.StartDate, htmlAttributes: new { @class = "control-label col-md-2" })
                <div class="col-md-10">
                    @Html.EditorFor(model => model.StartDate, new { htmlAttributes = new { @class = "datepicker" } })               
                </div>
            </div>
            <div class="form-group">
                @Html.LabelFor(model => model.EndDate, htmlAttributes: new { @class = "control-label col-md-2" })
                <div class="col-md-10">
                    @Html.EditorFor(model => model.EndDate, new { htmlAttributes = new { @class = "datepicker" } })
                </div>
            </div>
            
#####The Problem #5

This one was another fun one. The issue here was concerning the chat feature that our program has. Below is the 
ChatMessage model as it existed when we had the issue. You'll notice that the data type for the variable for "Sender" 
was set to the class "ApplicationUser". I have highlighted that with 3 asterisks below. The issue with having the
data type be set to a whole instantiation of the ApplicationUser class was that it created a dependency where a user
could not be deleted from the program without first deleting all of their messages. My task was to solve this.

    namespace ConstructionNew.Models
    {
        public class ChatMessage
        {
            [Display(Name = "ID")]
            public Guid ChatMessageId { get; set; }
            [Display(Name = "Sender")]
            ***public virtual ApplicationUser Sender { get; set; }***
            [Display(Name = "Date")]
            public DateTime Date { get; set; }
            [Display(Name = "Message")]
            public string Message { get; set; }
        }
    } 

#####The Solution #5

This was the first chance I got to work with making migrations and changing an actively utilized class. All we really
needed was the user name of the Application user for the chat to function. First things first. The code below reflects
the changes I made to the ChatMessage model. Again, highlighted with 3 asterisks, you'll notice I changed the data type
of the variable sender to a string. This allowed for the user name to be used instead of the whole Application user. 

    namespace ConstructionNew.Models
    {
        public class ChatMessage
        {
            [Display(Name = "ID")]
            public Guid ChatMessageId { get; set; }
            [Display(Name = "Sender")]
            ***public string Sender { get; set; }***
            [Display(Name = "Date")]
            public DateTime Date { get; set; }
            [Display(Name = "Message")]
            public string Message { get; set; }
        }
    } 
    
Next, below you'll see the code block from the chat hub controller as it existed before the fix. Highlighted with 3
asterisks is where we were setting the variable sender when a new message was created.

     public void Send(string name, string message) //This send method is called from Ajax
        {
            object timestamp = DateTime.Now;
            // Call the addNewMessageToPage method to update clients' message boxes.
            Clients.All.addNewMessageToPage($"{ DateTime.Now.ToString("h:mm tt")} {name} {message}");
            using (ApplicationDbContext db = new ApplicationDbContext())
            {
                ChatMessage chat = new ChatMessage();
                Guid ChatId = Guid.NewGuid();
                chat.Message = message;
                chat.ChatMessageId = ChatId;
                chat.Date = DateTime.Now;
                string currentUserId = HttpContext.Current.User.Identity.GetUserId(); //Get's ID of current user
                ApplicationUser currentUser = db.Users.FirstOrDefault(x => x.Id == currentUserId); //Gets name of current user from ID
                ***chat.Sender = currentUser; //Save message-sender to db instance***
                //System.Diagnostics.Debug.WriteLine(message);
                db.ChatMessages.Add(chat); //Adds a chatmessage entity to collection representing DB table
                db.SaveChanges(); //Apply changes to DB model to DB itself
            }
        }
        
Below is the same code block after I made the necessary changes. Again, highlighted with 3 asterisks, you'll see that
all I had to do was specifically select the "UserName" variable from the current user to be assigned to the variable
"chat.Sender" instead of just "current user", which before was setting the entire instance to "chat.Sender".
 
     public void Send(string name, string message) //This send method is called from Ajax
        {
            object timestamp = DateTime.Now;
            // Call the addNewMessageToPage method to update clients' message boxes.
            Clients.All.addNewMessageToPage($"{ DateTime.Now.ToString("h:mm tt")} {name} {message}");
            using (ApplicationDbContext db = new ApplicationDbContext())
            {
                ChatMessage chat = new ChatMessage();
                Guid ChatId = Guid.NewGuid();
                chat.Message = message;
                chat.ChatMessageId = ChatId;
                chat.Date = DateTime.Now;
                string currentUserId = HttpContext.Current.User.Identity.GetUserId(); //Get's ID of current user
                ApplicationUser currentUser = db.Users.FirstOrDefault(x => x.Id == currentUserId); //Gets name of current user from ID
                ***chat.Sender = currentUser.UserName; //Save message-sender to db instance***
                //System.Diagnostics.Debug.WriteLine(message);
                db.ChatMessages.Add(chat); //Adds a chatmessage entity to collection representing DB table
                db.SaveChanges(); //Apply changes to DB model to DB itself
            }
        }

After this was complete all I had to do was make the necessary migrations. In the end this was a simple fix on the logic
side of things. What it gave me the most practice with was making migrations properly as to protect the integrity.
As I will touch on in the upcoming story retrospective, migrations became a reoccurring theme.

#####The Problem #6

This story was super easy. The task was to change an enum in our "state" enum list.

#####The Solution #6

There's not too much to talk about here. Below is the before and after of the code. I had to change the Idaho enum to 
California. When I get stories like this that are very straight foreword, I like to see how quickly and efficiently I
can run through them. As I touched on earlier, it's about getting into that state of flow when I know I can do so to
make the times I need to critically think more balanced. As before, the relevant code is highlighted with 3 asterisks.

    public enum State
    {
        [Description("Oregon"), Display(Name = "Oregon")]
        OR,
        [Description("Washington"), Display(Name = "Washington")]
        WA,
        ***[Description("Idaho"), Display(Name = "Idaho")]***
        ***ID***
    }
    
     public enum State
    {
        [Description("Oregon"), Display(Name = "Oregon")]
        OR,
        [Description("Washington"), Display(Name = "Washington")]
        WA,
        ***[Description("California"), Display(Name = "California")]***
        ***CA***
    }
    
#####The Problem #7

This next story was another good one for practicing efficiency with straight forward tasks and making migrations.
My task here was to add an attribute to the "IdentityModel" Users class and make the necessary migrations.


#####The Solution #7

Below is the attributes from the "IdentityModel" class and as you may have guessed by now, the code I added is 
highlighted with 3 asterisks. This was another simple story however having to make new migrations gave my a little more
responsibility here. 

    [Display(Name = "First Name")]
    public string FName { get; set; }
    [Display(Name = "Last Name")]
    public string LName { get; set; }
    [Display(Name = "Work Category")]
    public WorkType WorkType { get; set; }
    [Display(Name = "User Role")]
    public string UserRole { get; internal set; }
    ***[Display(Name = "Suspended")]***
    ***public bool Suspended { get; set; }***
    

#####The Problem #8

This one was another migrations story but this time instead of editing an existing model, I was creating a new one and
prepping the rest of the project to accept the new class where necessary. I had to add the class "ShiftTime" and model
around our database schema. I learned a few important lessons here. While the implementation for this class was 
basically planned already, I was not given much context as to what that was going to look like or how it would interact
with the other classes and the rest of the program. This being the case, there were some times of confusion regarding
exactly how I would implement the code I was supposed to write. I had to get used to the fact that sometimes I'm not 
always going to know or have communicated the entirety of a project or a part of a project. I'm going to have to 
determine the best course of action on my own and be patient with the fact that I eventually will learn how things are
going to come together, even if it takes a little while.

#####The Solution #8
 
Below is the ShiftTime class I wrote based off of our database schema. 
 
     public class ShiftTime
        {
            [Key]
            [Display(Name = "ID")]
            public Guid ShiftTimeId { get; set; }
            [Display(Name = "Monday")]
            public string Monday { get; set; }
            [Display(Name = "Tuesday")]
            public string Tuesday { get; set; }
            [Display(Name = "Wednesday")]
            public string Wednesday { get; set; }
            [Display(Name = "Thursday")]
            public string Thursday { get; set; }
            [Display(Name = "Friday")]
            public string Friday { get; set; }
            [Display(Name = "Saturday")]
            public string Saturday { get; set; }
            [Display(Name = "Sunday")]
            public string Sunday { get; set; }
            [Display(Name = "Default")]
            public string Default { get; set; }
        }
        
I also had to add a DB set for "ShiftTime" to the IdentityModel. I have had a decent amount of experience during this 
project with DB sets and learning about how they function. Very often throughout this project I would learn about how
all of the code relates to itself just by having to solve a simple task. I had already had a solid understanding of MVC,
but learning all of the intricacies of asp.net and the entity framework really came together during this project in a 
way it did not have the chance to before.    
        
     public DbSet<CreateUserRequest> CreateUserRequests { get; set; }
        public DbSet<ChatMessage> ChatMessages { get; set; }
        public DbSet<CompanyNews> CompanyNews { get; set; }
        public DbSet<Schedule> Schedules { get; set; }
        ***public DbSet<ShiftTime> ShiftTimes { get; set; }***
        public object AspNetUsers { get; internal set; }
        
        
        
 