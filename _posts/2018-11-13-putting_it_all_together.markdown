---
layout: post
title:      "Putting it all Together"
date:       2018-11-13 22:01:54 +0000
permalink:  putting_it_all_together
---


An interesting thing happened the day I decided I was ready to submit my React/Redux final project: I took a look at the final product and discovered it didn't look at all like something that should have taken over a month and a half to complete. There just wasn't that much to it. It freaked me out a little, I can't lie. A look at the [Github repo](https://github.com/lizbur10/nme-account-manager) revealed that I had made 144 commits, but I wasn't sure whether that should make me feel better or worse. Had it taken me a month and a half and 144 commits to do very little? It was not the happiest moment in my Flatiron School journey.

In hopes that it would make me feel a little better, I spent some time looking through my commits to remind myself of what I had been doing all that time. It made me feel a little better almost right away (although it probably shouldn't have) when I figured out that it was really more like a month of actual work days. But it didn't change the fact that I was looking at a final product that seemed pretty thin. However, in the silver lining department, the process of looking back through all the work I'd done turned out to be a valuable exercise: I got a nice overview of the whole process, and a reminder of a lot of things I'd forgotten. Looking at it from the 10,000 foot level made it seem a lot more orderly and organized than it felt while I was doing it. It also gave me a little sense of pride at everything I'd accomplished, which was a nice bonus.

The 10,000 foot level view can be divided into the following 6 steps:

1. Create the Rails backend: create the rails app, set up the migrations, models, associations, etc.
2. Set up the React app and create the connection that allows the front-end (React) and back-end (Rails) servers to communicate with each other
3. Create the Rails API: install active model serializers; create the serializers
4. Flesh out the React frontend (this was the big one, time-wise)
5. Install Redux and migrate state from the components to the application
6. Install Thunk and set up data persistence to the database

The steps aren't really as discrete as the list implies: a lot of things overlapped, and there were lots of occasions when I went back and modified things. But, overall, they give a reasonably accurate view of how the process proceeded.

## Step 1: Creating the Rails backend
This was the easy part. By the time I got to the final project, creating a Rails app felt familiar and comfortable. Setting up the models, controllers, routes, migrations, and associations made for a nice low-stress entry into the project. The hardest part was figuring out how I wanted everything set up. I did have to go back later to make adjustments, but I knew how to do that so it was pretty easy. 

## Step 2: Set up the React app
This was simply a matter of 1) following the instructions for using the [create-react-app](https://github.com/facebook/create-react-app) generator to set up the React framework, and 2) following the instructions [in this post](https://www.fullstackreact.com/articles/how-to-get-create-react-app-to-work-with-your-rails-api/) to make it possible to run both the front-end and back-end servers at the same time and for them to communicate with each other. There was a lot of futzy stuff involved in this step, but it wasn't really difficult: I just needed to read (and follow) the directions carefully.

## Step 3: Create the API
This was also quite straightforward: again, the hardest part was figuring out what the API needed to include and how it should be structured. Once I had that figured out (although here again I had to go back once or twice to adjust later), it was a simple matter of installing ActiveModel::Serializer and creating the serializers. 

The whole process of creating the Rails API was actually a whole lot of fun. Turns out, when you don't have to worry about things like views and layouts and partials and locals, creating a Rails app is (relatively speaking) a snap! But of course I *did* have to worry about creating the front end, which brings us to...

## Step 4: Create the React Front End
This was where the lion's share of the work came. My project is the start of a scheduling app for a company I'm working for, and some of the design decisions I made in an effort to make the app as useful, usable, and maintainable as possible for the employees had some significant development implications. Well, significant to a relative beginner, anyway. For example...

### The Sort Order Conundrum
The landing page for the app shows a list of all the corporate clients. Each client has a particular delivery day and a particular manager. The list of client companies is long so I wanted to display them separately by delivery day and then sorted by manager name:

![](http://burtonux.com/flatiron_blog/list_view_before.jpg)

So I needed to figure out two things: 1) how to get the records sorted by multiple values, and 2) how to separate the companies into tables by day of the week. 

The sorting issue turned out to be easy, but I spent a fair amount of time trying to figure out how to do it using JavaScript before it finally occurred to me that the sort should happen in the backend; specifically, in the AccountsController's index action:

```
def index
    @accounts = Account.all.sort_by do |account|
        [to_day_of_week(account.delivery_day), account.manager.name]
    end
    render json: @accounts
end
```

The sort calls a `to_day_of_week()` utility function so the days are sorted chronologically and not alphabetically:

```
def to_day_of_week(day)
    Date.parse(day,"%w")
end
```

Then, in the React Container for accounts, a `separateDays()` function uses the `JavaScript filter()` method to separate the accounts by day of the week:

```
separateDays = () => {
  const daysArray = ['monday', 'tuesday', 'wednesday', 'thursday'];
  const dayArray = [];
  if (this.props.accounts) {
    for (let i=0; i < daysArray.length; i++) {
      dayArray[i] = this.props.accounts.filter(function (account) {
        return account.delivery_day.toLowerCase() === daysArray[i];
      })
    }
  }
  return dayArray;
}
```

The `separateDays()` function creates an array of arrays; each sub-array contains the list of companies for a given day. The function is called from the render method:

```
render() {
  return (
    <div>
      { this.separateDays().map( (day, i) =>
          <div key={i}>
            <AccountList 
              accounts={day} />
          </div> 
      )}
      <Link className="add-new-button" to="/accounts/new">Add New Account</Link>
    </div>
  )
}
```

In the render method, each of the sub-arrays (a list of companies for a given day) is passed as props to the AccountList component, which then renders a separate table for each day.

### The Activation Predicament
The list view of companies includes an edit button which, when clicked, renders the edit view:

![](http://burtonux.com/flatiron_blog/edit_view.jpg)

Here (logically enough) the details for the account can be edited. But there's one piece of information here that could change fairly frequently: the Active/Inactive status. Companies receive a delivery on the same day and at the same time each week, but it's not unusual for companies to skip a week. Therefore, I thought it was important to include the active/inactive control in the list view as well, so it's easy to both see and change a company's status. But if it's going to be on the list view, UX principles dictate it should look and act the same as on the edit page:

![](http://burtonux.com/flatiron_blog/list_view_after.jpg)

Making this work turned out to be quite the challenge. On the edit view, changes to the active status are captured in the change handler. (Note that the switch is actually a checkbox that is made to look like a toggle with css. I modified [code from w3schools.com](https://www.w3schools.com/howto/howto_css_switch.asp) to make my version.)

```
handleChange = event => {
    let value;
    if (event.target.type === 'checkbox') {  
        value = event.target.checked;
    } else {
        value = event.target.value;
    }
    this.setState({
        account: {
            ...this.state.account,
            [event.target.name]: value
        }
    })
}
```

However, the active switch on the list page is not part of a form. I spent a pretty big chunk of time trying to get this to work in Redux to no avail. I then tried to add local state to the container and write a function to update that, then update the application state through Redux, but I couldn't get that to work either. Turns out I was (once again) making things more complicated than they needed to be. All I needed to do was create a `toggleSwitch()` method that is passed as props to the AccountList component:

```
toggleSwitch = (accountInfo, active) => {
  const updatedAccount = Object.assign({}, accountInfo, { active });
  this.props.onSubmitUpdatedAccount(updatedAccount, false);
}
```


```
render() {
  return (
    <div>
      { this.separateDays().map( (day, i) =>
          <div key={i}>
            <AccountList 
              Accounts={day}
              toggleSwitch={this.toggleSwitch} />
          </div> 
      )}
      <Link className="add-new-button" to="/accounts/new">Add New Account</Link>
    </div>
  )
}
```

The `toggleSwitch()` method first updates the account's active status then calls the Redux action that 1) updates the application state, and 2) saves the updated account to the database. This in turn triggers a re-render that updates the active status on the page.

## Step 5: Implement Redux
I'm not going to lie: implementing Redux was a pain in the neck. For each set of container/components you're using, you have to:

1. Create the actions
2. Create the reducers
3. Initialize the application state (which requires hard-coded data until Thunk is implemented)
4. Create the MapStateToProps function
5. Import the actions into the container 
6. Create the MapDispatchToProps function
7. Update the associated method calls to access the props, and 
8. Whatever else I've forgotten

There are so many moving pieces that there was always something wired together wrong and, because you can't really check it until all the pieces are in place, it was sometimes very difficult to find where the problem was. Sadly, I have developed PTSD around certain error types that cropped up over and over and over...

## Step 6: Implement Thunk
Finally, once I tracked down and killed all the errors from implementing Redux, it was time to implement Thunk and get the database hooked up again. This process was similar to Step 5 -- it involved creating more actions and getting them hooked up correctly, and more rounds of tracking down and killing those same errors. But it was oh, so satisfying once it was done! 

## The Hidden Step: Refactoring
Throughout the process, I spent a fair amount of additional time restructuring, reorganizing, and refactoring. This also was very satisying. At first I couldn't remember where anything was in my project to save my life. But after spending a lot of time with the code, coding and recoding, streamlining and consolidating, restructuring and DRYing, now that only happens about half the time. 

## Final Thoughts
My project still feels pretty thin to me, but it's pretty far from being ready for prime time. There are a lot features that are still to be added and improvements to be made to what *is* there. But looking back over everything I'd done helped me realize that some of that thinness was actually the product of a whole lot of work: sometimes less on the page (whether it's the files containing your code or the browser window showing your app) is better.

