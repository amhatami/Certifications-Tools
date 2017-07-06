# angularjs-login (Angular 1.6)
Techniques for authentication in AngularJS applications , A collection of ideas for authentication & access control

## Authentication
The most common form of authentication is logging in with a username (or email address) and password. This means implementing a login form where users can enter their credentials. Such a form could look like this:

```HTML
<form name="loginForm" ng-controller="LoginController"
      ng-submit="login(credentials)" novalidate>
  <label for="username">Username:</label>
  <input type="text" id="username"
         ng-model="credentials.username">
  <label for="password">Password:</label>
  <input type="password" id="password"
         ng-model="credentials.password">
  <button type="submit">Login</button>
</form>
```
(Note: there’s an extended version below, this is just a simple example)

Since this is an Angular-powered form, we use the ngSubmit directive to trigger a scope function on submit. Note that we’re passing the credentials as an argument rather than relying on $scope.credentials, this makes the function easier to unit-test and avoids coupling between the function and it’s surrounding scope. The corresponding controller could look like this:

```Javascript
.controller('LoginController', function ($scope, $rootScope, AUTH_EVENTS, AuthService) {
  $scope.credentials = {
    username: '',
    password: ''
  };
  $scope.login = function (credentials) {
    AuthService.login(credentials).then(function (user) {
      $rootScope.$broadcast(AUTH_EVENTS.loginSuccess);
      $scope.setCurrentUser(user);
    }, function () {
      $rootScope.$broadcast(AUTH_EVENTS.loginFailed);
    });
  };
})
```
The first thing to notice is the absence of any real logic. This was done deliberately so to decouple the form from the actual authentication logic. It’s usually a good idea to abstract away as much logic as possible from your controllers, by putting that stuff in services. `AngularJS controllers should only manage the $scope object (by watching and manipulating) and not do any heavy lifting.`

## Communicating session changes
Authenticating is one of those things that affect the state of the entire application. For this reason I prefer to use events (with $broadcast) to communicate changes in the user session. It’s a good practice to define all of the available event codes in a central place. I like to use constants for these things:

```javascript
.constant('AUTH_EVENTS', {
  loginSuccess: 'auth-login-success',
  loginFailed: 'auth-login-failed',
  logoutSuccess: 'auth-logout-success',
  sessionTimeout: 'auth-session-timeout',
  notAuthenticated: 'auth-not-authenticated',
  notAuthorized: 'auth-not-authorized'
})
```
A nice thing about constants is that they can be injected like services, which makes them easy to mock in your unit tests. It also allows you to easily rename them (change the values) later without having to change a bunch of files. The same trick is used for user roles:
```javascript
.constant('USER_ROLES', {
  all: '*',
  admin: 'admin',
  editor: 'editor',
  guest: 'guest'
})
```
If you ever want to give all editors the same rights as administrators, you can simply change the value of editor to ‘admin’.

## The AuthService
The logic related to authentication and authorization (access control) is best grouped together in a service:
```javascript
.factory('AuthService', function ($http, Session) {
  var authService = {};
 
  authService.login = function (credentials) {
    return $http
      .post('/login', credentials)
      .then(function (res) {
        Session.create(res.data.id, res.data.user.id,
                       res.data.user.role);
        return res.data.user;
      });
  };
 
  authService.isAuthenticated = function () {
    return !!Session.userId;
  };
 
  authService.isAuthorized = function (authorizedRoles) {
    if (!angular.isArray(authorizedRoles)) {
      authorizedRoles = [authorizedRoles];
    }
    return (authService.isAuthenticated() &&
      authorizedRoles.indexOf(Session.userRole) !== -1);
  };
 
  return authService;
})
```
To further separate concerns regarding authentication, I like to use another service (a singleton object, using the service style) to keep the user’s session information. The specifics of this object depends on your back-end implementation, but I’ve included a generic example below.
```javascript
.service('Session', function () {
  this.create = function (sessionId, userId, userRole) {
    this.id = sessionId;
    this.userId = userId;
    this.userRole = userRole;
  };
  this.destroy = function () {
    this.id = null;
    this.userId = null;
    this.userRole = null;
  };
})
```
Once a user is logged in, his information should probably be displayed somewhere (e.g. in the top-right corner). In order to do this, the user object must be referenced in the $scope object, preferably in a place that’s accessible to the entire application. While $rootScope would be an obvious first choice, I try to refrain from using $rootScope too much (actually I use it only for global event broadcasting). Instead my preference is to define a controller on the root node of the application, or at least somewhere high up in the DOM tree. The body tag is a good candidate:
```html
<body ng-controller="ApplicationController">
  ...
</body>
```
The ApplicationController is a container for a lot of global application logic, and an alternative to Angular’s run function. Since it’s at the root of the $scope tree, all other scopes will inherit from it (except isolate scopes). It’s a good place to define the currentUser object:
```javascript
.controller('ApplicationController', function ($scope,
                                               USER_ROLES,
                                               AuthService) {
  $scope.currentUser = null;
  $scope.userRoles = USER_ROLES;
  $scope.isAuthorized = AuthService.isAuthorized;
 
  $scope.setCurrentUser = function (user) {
    $scope.currentUser = user;
  };
})
```
We’re not actually assigning the currentUser object, we’re merely initializing the property on the scope so the currentUser can later be accessed throughout the application. Unfortunately we can’t simply assign a new value to it from a child scope, because that would result in a shadow property. It’s a consequence of primitive types (strings, numbers, booleans, undefined and null) being passed by value instead of by reference. To circumvent shadowing, we have to use a setter function. For a lot more on Angular scope and prototypal inheritance, read Understanding Scopes.
Besides initializing the currentUser property, I’ve also included some properties which allow easy access to USER_ROLES and the isAuthorized function. These should only be used in template expressions, not from other controllers, because doing so would complicate the controller’s testability. In the next chapter you’ll see how we use these properties.

<hr align="center" width="50%">
<hr align="center" width="60%">

# Access control

Authorization a.k.a. access control in AngularJS doesn’t really exist. Since we’re talking about a client-side application, all of the source code is in the client’s hands. There’s nothing preventing the user from tampering with that code to gain ‘access’ to certain views and interface elements. All we can really do is visibility control. If you need real authorization you’ll have to do it server-side, but that’s beyond the scope of this article.

## Restricting element visibility

AngularJS comes with several directives to show or hide an element based on some scope property or expression: ngShow, ngHide, ngIf and ngSwitch. The first two will use a style attribute to hide the element, while the last two will actually remove the element from the DOM.
The first solution (hiding it) is best used only when the expression changes frequently and the element doesn’t contain a lot of template logic and scope references. The reason for this is that any template logic within a hidden element will still be reevaluated on each digest cycle, slowing down the application. The second solution will remove the DOM element entirely, including any event handlers and scope bindings. Changing the DOM is a lot of work for the browser (hence the reason for using ngShow/ngHide in some cases), but worth the effort most of the time. Since user access doesn’t change often, using ngIf or ngSwitch is the best choice:

```html
<div ng-if="currentUser">Welcome, {{ currentUser.name }}</div>
<div ng-if="isAuthorized(userRoles.admin)">You're admin.</div>
<div ng-switch on="currentUser.role">
  <div ng-switch-when="userRoles.admin">You're admin.</div>
  <div ng-switch-when="userRoles.editor">You're editor.</div>
  <div ng-switch-default>You're something else.</div>
</div>
```
The switch example assumes a user can have only one role. I’m sure you can come up with something more flexible, but you get the idea.

## Restricting route access

Most of the time you will want to disallow access to an entire page rather than hide a single element. Using a custom data object on the route (or state, when using UI Router), we can specify which roles should be allowed access. This example uses the UI Router style, but the same will work for ngRoute.

```javascript
.config(function ($stateProvider, USER_ROLES) {
  $stateProvider.state('dashboard', {
    url: '/dashboard',
    templateUrl: 'dashboard/index.html',
    data: {
      authorizedRoles: [USER_ROLES.admin, USER_ROLES.editor]
    }
  });
})
```

Next, we need to check this property every time the route changes (i.e. the user navigates to another page). This involves listening to the $routeChangeStart (for ngRoute) or $stateChangeStart (for UI Router) event:

```javascripts
.run(function ($rootScope, AUTH_EVENTS, AuthService) {
  $rootScope.$on('$stateChangeStart', function (event, next) {
    var authorizedRoles = next.data.authorizedRoles;
    if (!AuthService.isAuthorized(authorizedRoles)) {
      event.preventDefault();
      if (AuthService.isAuthenticated()) {
        // user is not allowed
        $rootScope.$broadcast(AUTH_EVENTS.notAuthorized);
      } else {
        // user is not logged in
        $rootScope.$broadcast(AUTH_EVENTS.notAuthenticated);
      }
    }
  });
})
```

When a user is not authorized to access the page (because he’s not logged in or doesn’t have the right role), the transition to the next page will be prevented, so the user will stay on the current page. Next, we broadcast an event which other modules can listen to. I suggest including a loginDialog directive on the page which appears when the notAuthenticated event is fired, and an error message which should appear when the notAuthorized event occurs.

<hr align="center" width="50%">
<hr align="center" width="60%">

# Session expiration


