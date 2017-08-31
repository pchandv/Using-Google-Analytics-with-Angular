Web application owners depend on the free services of google analytics to track the behaviour of visitors to their website.

For traditional web applications, setting up google analytics is as easy as copying and pasting the JavaScript tracking snippet provided on registration of a new property. The only setup necessary is adding the code to the `<head>` tag of the `HTML` files to be tracked and the entire navigation history of each visitor will be updated during real time, if nothing goes wrong.

Do not get lost in the comfort of pasting snippets around because the era of single page applications is forcing developers and non-developers alike to adopt new approaches towards implementing web analytics services on their applications.

# What is a Single Page Application (SPA)

A single page application is a web application or website, that only needs to load its resources the first time a page (usually the root page) is requested. Anymore data needed as the user interacts with the application is dynamically loaded during real time using XMLHttpRequest, and there is no actual refresh to the browser or full request for a new page from the server.

Though, the location of the browser may be updated in accordance with anchor clicks by the user, the web page isn't actually changed, content is just dynamically rendered to the view section of the SPA and no full page request is really ever made to the server.

This is futuristic, SPAs are even great for developing responsive applications, a popular example of a single page application would be Google's Gmail. While these type of applications deliver wonderful user experience and lots more, they require extra attention during development and maintenance in certain areas that would normally be overlooked in their traditional counterpart.

Back to web analytics services, the Javascript tracking snippet for google analytics is not well adapted to track user navigation history and user behaviour on SPAs yet, though google provides a [documentation](https://developers.google.com/analytics/devguides/collection/analyticsjs/single-page-applicationstp) on how to achieve this. 

Angular is an awesome framework of Javascript for developing single page applications; it has a robust routing framework called the [UI-Router](https://scotch.io/tutorials/angular-routing-using-ui-router) that is responsible for updating the view based on the state of the application, this article will mainly discuss how your AngularJS application can be intentionally written to support google analytics services.

The source code is available [here](https://github.com/Jordanirabor/Using-Google-Analytics-with-Angular) on github.

Below is a step by step guide:

# Setting up google analytics
We begin by setting up a google analytics account and registering a property [here ](https://analytics.google.com/analytics/). 

If you have Angular and other modules installed on your local machine, be sure to have the UI Router module also available because we will be injecting it as a module dependency when initializing our app's main module.

If you'd rather use AngularJS and other modules from another computer, you can get Angular on this [url](https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js) and the UI Router module on this [url](https://cdnjs.cloudflare.com/ajax/libs/angular-ui-router/1.0.3/angular-ui-router.js) (I will be referencing Angular and its modules from a CDN for this article).

# Setting up the `head` tag
Copy and paste the JavaScript tracking snippet with its unique tracking ID just above the closing `</head>` tag.

Now comment out the two lines of code that look like this: 


```
ga('create', 'UA-XXXXX-X', 'auto');

ga('send', 'pageview');
```


Where `UA-XXXXX-X` is your unique tracking ID.

We are commenting these lines out right now because we are building an SPA and not a traditional application, we'd later wrap them in the `run` block of our AngularJS script to ensure that when our app begins running, they will only be triggered based on certain changes in the application's state.

Following the above instructions, we should have a `</head>` tag looking similar to this:

```
<!DOCTYPE html>
<html>
<head>

<!-- script to load Angular -->
<script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>

<!-- script to load the UI Router module -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/angular-ui-router/1.0.3/angular-ui-router.js"></script>

<!-- Javascript tracking snippet begins here -->
<script>
  (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
  (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
  m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
  })(window,document,'script','https://www.google-analytics.com/analytics.js','ga');
</script>

</head>
</html>
```

# Setting up the body as a view container
Firstly, let's understand how the view section of an SPA works; since the entire application behaves as if it is a single page, Angular loads the content of other requested pages into the view section, we can think of this view section as a container within the main application. To specify this view section, we assign the `ui-view` directive of the UI-Router module to a div or any other suitable `HTML` tag, just like this:

```
<body ng-app="myApp">
<div ui-view>
<a ui-sref="first">This is the first page.</a>
<a ui-sref="second">This is the second page.</a>
<a ui-sref="third">This is the third page.</a>
<h1>{{ title }}</h1>
</div>
</body>
```

This is the body section for my application, it's very simple but it defines the basic structure for defining the AngularJS app and the view section.

# Configuring the application
We begin by initializing the main Angular module and injecting `ui.router` as a module dependency.

`var app = angular.module("myApp", ['ui.router']);`

Next we write the configuration block of our application and inject all the providers we will be needing for this application, the result looks like this:

```
app.config(['$urlRouterProvider', '$stateProvider', function($urlRouterProvider, $stateProvider){
	  $urlRouterProvider.otherwise('/');
	  $stateProvider
	  .state('index', {
			url: '/',
			templateUrl: 'index.html',
			controller: function($scope){
			$scope.title = 'I am the root page, ergo, submit thyself to me.'
    }
  })
```

Again, this is just a simple demonstration but it defines how an actual application should be set up.

Lastly, we write the `run` block of the AngularJS app to ensure that our application sends data to google analytics whenever a new page is loaded into the view section. We do this by injecting some services and using their methods during the run phase of our app's lifecycle. 

We inject the `$location` service and it will help us determine the current location of the browser using its `path()` method.

Remember the two lines of code we initially commented out above, we'd be needing them very soon since we've gotten to the run block we spoke about earlier.

Next we inject the global `$window` service and what this will do for us is:  **Trigger the ga's (google analytics object) methods we commented out in the `</head>` of our program. **

Next we inject the `$transitions` service that will help us determine when a user has made or is making a transition to a new state. 

In the end we combine all these services and their methods to jointly instruct AngularJS to send data (containing information of the browsers current location) to google analytics when a transition to a new state is successful.

Here is what the code for the `run` block looks like:

```
app.run(['$location', '$window','$transitions', function($location, $window, $transitions){
        $window.ga('create', 'UA-XXXXX-X', 'auto');
        $transitions.onSuccess({}, function(){
              $window.ga('send', 'pageview', $location.path());
        });
}])
```

Awesome, now for every new transition to a new state in our application, we'd get an update on our google analytics dashboard. On my web browser, I have visted my application and have navigated to a view that has a url of `/first`.

This is the immediate update on my google analytics dashboard:

//image go here

And just like that, our application is now being tracked properly by google analytics, we get real-time update about the navigation history of visitors to our web application.
