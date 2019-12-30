# Live Project
## Introduction
During the Live Project with The Tech Academy, I was part of a team of fellow students, lead by two Project Managers, working on a pre-existing, full scale MVC/MVVM Web Application in C#. This was an amazing opportunity to see all that I'd been learning (one step at a time) come together in a real-world setting. Adding to this, because it was a legacy codebase, I was inadvertently provided with additional learning experience with fixing bugs, cleaning up code, and adding requested features. 

The other component of this that I felt was extremely useful was getting far more familiar with the ebb and flow of a sprint/project. I learned how to use more features of Microsoft Visual Studio (that weren't covered in my initial schooling), the ins and outs of project management software (Azure) and working with a more remote team, how important good communication is both written and verbal.

Sprints were a week in length, I worked on the Live Project for three sprints. Below are descriptions of the stories I worked on, along with code snippets.

### **Story 5406 - Page Preloader Bug Fix (Front End)**

**Story Description:**
We've got a preloader added to the project, but for some unknown reason, it is not working. The code on the javaScript file site.js (line 402 -421) is the logic behind the preloader. Please make the code work or create a working code on your own. (Don't forget to clean up unnecessary leftover code too.)
Some tips if you decide to stick with the code we have:
1. The class .btnSubmit needs to be on all the buttons/links that would trigger the page-preloader.
2. The div with ID "divLoader" needs to be on the _Layout page in order for it to be accessible from any page on the site.

**Resolution:**
A helper class was created for buttons in specific. After some testing on my part, it was discovered that those buttons created using the helper class properly triggered the preloader. Buttons created through scaffolding did not trigger the preloader (regardless of what class was being referenced). I did not create any new code, more that I ensured that things were where we needed them to be to at least get it working in one area so that further fixes could be made.

**Code:**
Example of a Button w/Helper Class: 
```
<div class="row">
      <div class="col-3" align="left">
          @Html.AnchorButton(AnchorType.Create, Url.Action("Create"))
      </div>
</div>
```

### **Story 5355 - Mobile Friendly Product/Index (Back End/Front End)**

**Story Description:**
We have mobile-friendly pages for all our sites (through the branch AH-5151-MobileFriendlyDesign). Pages added after this branch lacks the mobile-friendly display of their content. One of these pages is the Product/AdminIndex -- a view page you see when logged in as a User (employee).

Refer to how the AH-5151-Mobile FriendlyDesign was implemented in the first place and apply the same mobile-friendly view implementation for the Product/AdminIndex page. Nb: Please include a screenshot of your final work on the comment section while making the pull request. 

**Resolution:**
I created a new mobile layout view for the Products page and created a new class for styling (which was added to the mobile style sheet). Given this was what was implemented for other pages (and worked), I was simply following suit by creating a new layout view. The one adjustment I'd made was using `<center>` tags for the menu items vs. `<p>` tags. It seemed a more straightforward way of centering things. I opted to create a class specific to this particular view vs. adding to pre-existing styling as each page had unique elements that styling needed to account for.

**Code:**

Views - Products - Mobile Layout View (_AdminIndexMobile.cshtml)
```
@using ManagementPortal.Common

@model IEnumerable<ManagementPortal.Models.Product>

@{
    Layout = "";
}

<!--Added new Header container -->
<div class="titleContainerProductsMobile">
    <h2>Products</h2>
</div>

<center>
    @Html.ActionLink("Create New", "Create") |
    @Html.ActionLink("View Cart", "Index", "CartItem")
</center>

<div class="container">
    <div class="row">
        @foreach (var item in Model)
        {
            <div class="col-sm-6 col-lg-3 py-2">
                <div class="card">
                    <img src="~/Content/images/Product/@(item.ImagePath)" class="card-img-top mx-auto" alt="Image" style="width:210px;height:200px;" />
                    <div class="card-body">
                        <h5 class="card-title">@(Html.DisplayFor(modelItem => item.ProductName))</h5>
                        <p class="card-text">$@(Html.DisplayFor(modelItem => item.UnitPrice))</p>
                        <p class="card-text"><small class="text-muted">@(Html.DisplayFor(modelItem => item.ProductDescription))</small></p>
                        <p class="card-text">

                            <a href="@Url.Action("AddToCart", "CartItem", new { id = item.ProductId }, null)" title="Add to Cart"><i class="btn btn-primary fa fa-cart-plus" style="height:29px; line-height:75%; border-radius:0;"></i></a> @*Line-height is used to center the icon within the button*@
                            @Html.Partial(AnchorButtonGroupHelper.PartialView, AnchorButtonGroupHelper.GetEditDetailsDelete(item.ProductId.ToString()))

                        </p>
                    </div>
                </div>
            </div>
        }
    </div>
</div>
```

Content - Mobile - Products (Site.Mobile.css):
```
/*Begin Mobile - Products page*/
.titleContainerProductsMobile {
    margin-top: 60px;
}
/*End Mobile - Products page*/
```

### **Story 5587 - On Schedule Check (Back End)**

**Story Description:**
We already have a method in the Create Schedule method of the Schedules controller to check if someone has vacation during the time they are being scheduled. Now we want to add a method to check if they are on the schedule already for that job, or for another job. 

Use Linq and logic statements to check for the existing schedule items and see if they overlap with the new schedule item being created. We only care about the schedule items for the employee we're trying to create a new schedule item for. Alert the user about any overlap. 

Optional Add-On: 
If the overlapping schedule item is for the same job, give the user the option to edit that schedule item, including a button that takes them to the edit page for that schedule item.

**Resolution:**
I added an additional method to the SchedulesController to check if an employee was available (essentially not already scheduled) during the time of the job the admin was trying schedule them for. I used similar logic to that already being used to checked whether or not an employee was on vacation when an admin was trying to schedule a job for them. I also created a shared view specific to this situation so that the admin would know why they couldn't schedule an employee for a particular job (if they were already working).

**Code:**

Controllers - Schedules Controller
```
//Checking to see if an employee is scheduled to work another job.
var user_jobs = db.Schedules.Where(u => u.Person.Id == PersonId);

foreach (var job in user_jobs)
{
    if (job.StartDate >= schedule.StartDate && job.EndDate <= schedule.EndDate)
    {
        return View("ScheduleOverlap", job);
    }
    else if (schedule.StartDate <= job.EndDate && job.EndDate >= schedule.StartDate)
    {
        if (schedule.StartDate <= job.StartDate && schedule.EndDate <= job.StartDate)
        {
        }
        else
        {
            return View("ScheduleOverlap", job);
        }
    }
}
```

Views - Shared - Schedule (ScheduleOverlap.cshtml)
```
@model ManagementPortal.Models.Schedule
@{
    ViewBag.Title = "ScheduleOverlap";
}
<div class="defaultContainer">
    <h2>Whoopsies  @Html.DisplayFor(modelItem => modelItem.Person.FullName) is already scheduled to work from @Html.DisplayFor(model => model.StartDate) until @Html.DisplayFor(model => model.EndDate)</h2>

    <div class="loginItemContainer col-md-6">
        @Html.ActionLink("Back", null, null, new { Href = Request.UrlReferrer })
    </div>
</div>
```
