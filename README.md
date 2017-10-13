# angular-luminate-utils

**This library is currently in beta, and significant changes are likely in future versions.**

Luminate Online utilities for AngularJS 1.x apps. At its core, this library is a JavaScript wrapper around 
the [Luminate Online REST API](http://open.convio.com/api), with some helper functions and other magic 
sprinkled in. The library includes support for [all major modern browsers](#browser-support), and it can be 
used both within and outside of Luminate Online.

## Table of contents

- [Basic Setup](#basic-setup)
- [Including ngLuminateUtils In Your App](#including-ngLuminateUtils-in-your-app)
- [Configuration With $luminateUtilsConfig](#configuration-with-$luminateUtilsConfig)
- [API Requests With $luminateRest](#api-requests-with-$luminateRest)
- [Evaluating Template Tages With $luminateTemplateTag](#evaluating-template-tags-with-$luminateTemplateTag)
- [Managing Session Variables With $luminateSessionVar](#managing-session-variables-with-$luminateSessionVar)
- [Browser Support](#browser-support)
- [Reporting Issues](#reporting-issues)

## Basic Setup

Before getting started, there are a couple of basic steps you must follow:

 * Create an API Key
   
   In order to use the Luminate Online API, you must define an API Key for your organization's Luminate Online 
   website. If you haven't already done so, go to Setup -> Site Options -> Open API Configuration, and click 
   "Edit API Keys". The only option you need to worry about on this page is **1. Convio API Key**.
 
 * Whitelist your domain
   
   For security reasons, API requests are limited to a whitelist of domains defined by your organization. If you 
   haven't already done so, go to Setup -> Site Options -> Open API Configuration, and click 
   "Edit Javascript/Flash configuration". The only options you need to worry about on this page are 
   **1. Allow JavaScript/Flash API from these domains** and **2. Trust JavaScript/Flash API from these domains**. 
   Add any domains where you will use this library to these lists. As noted on the page, you can use an asterisk 
   as a wildcard if your website has multiple subdomains, e.g. "\*.myorganization.com".

## Including ngLuminateUtils In Your App

Once you've uploaded 
[angular-luminate-utils.min.js](https://github.com/noahcooper/angular-luminate-utils/blob/master/angular-luminate-utils.min.js) 
to your website, including the library is easy &mdash; just add it somewhere below Angular. 
(Change out the file path as needed, depending on where you uploaded the file on your site.)

```  html
<script src="../js/angular-luminate-utils.min.js"></script>
```

Then, using the library is as simple as injecting the `ngLuminateUtils` module as a dependency in your app.

``` js
angular.module('myApp', ['ngLuminateUtils']);
```

## Configuration With $luminateUtilsConfig

The library is instantiated using the `$luminateUtilsConfigProvider`. At a minimum, you must set your 
nonsecure and secure Luminate Online paths, as well as your API Key. **nonsecure** is the path for requests 
made over HTTP, e.g. "http://www.myorganization.com/site/". **secure** is the path for requests made over HTTPS, 
e.g. "https://secure2.convio.net/myorg/site/". Both the `setPath` and `setKey` methods return the provider, 
allowing for chaining.

``` js
angular.module('myApp').config(['$luminateUtilsConfigProvider', function($luminateUtilsConfigProvider) {
  $luminateUtilsConfigProvider.setPath({
    nonsecure: 'http://www.myorganization.com/site/', 
    secure: 'https://secure2.convio.net/myorg/site/'
  }).setKey('123456789');
}]);
```

Additionally, you can define a list of common parameters to be included in all API requests, e.g. source and 
sub-source codes, using the `setDefaultRequestData` method.

``` js
$luminateUtilsConfigProvider.setDefaultRequestData('source=MySourceCode');
```

## API Requests With $luminateRest

The `$luminateRest` service is the heart of the library. It provides methods for making AJAX requests to the 
Luminate Online REST API, with automatic handling of authentication tokens for methods that require it.

The `request` method accepts either a full, case-sensitive API servlet name, e.g. "CRConsAPI", or a 
case-insensitive shorthand with "CR" and "API" removed, e.g. "cons". The api_key, response_format, 
suppress_response_codes, and v parameters are automatically appended to the request data provided.

``` js
angular.module('myApp').controller('myCtrl', ['$scope', '$luminateRest', function($scope, $luminateRest) {
  $luminateRest.request('cons', 'method=loginTest').then(function(response) {
    if (response.data.loginResponse && response.data.loginResponse.cons_id && response.data.loginResponse.cons_id > 0) {
      $scope.loggedIn = true;
    } else {
      $scope.loggedIn = false;
    }
  });
}]);
```

The third argument passed to the request method is a Boolean indicating whether or not the API method being 
called requires authentication. If true, an auth token is automatically appended to the request data.

``` js
$luminateRest.request('cons', 'method=getUser', true);
```

By default, all API requests are made over HTTPS. Optionally, you can pass a fourth argument to force HTTP.

``` js
$luminateRest.request('cons', 'method=getUser', true, false);
```

Some API servlets (namely CRDonationAPI and CRTeamraiserAPI) must always be called over a secure channel, in 
which case this option is ignored. **Note that this option will be deprecated in a future version of this library.**

## Evaluating Template Tages With $luminateTemplateTag

For those occasions when the REST API does not provide a method for retrieving some information, but a Luminate Online 
template tag (e.g. S- or E-Tag) exists that meets the need, the `$luminateTemplateTag` service can be used to 
evaluate a tag client-side. 

``` js
angular.module('myApp').controller('myCtrl', ['$scope', '$luminateTemplateTag', function($scope, $luminateTemplateTag) {
  $luminateTemplateTag.parse('[[S42:1234:dollars]]').then(function(response) {
    $scope.amountRaised = response;
  });
}]);
```

## Managing Session Variables With $luminateSessionVar

The `$luminateSessionVar` service provides methods for setting and getting Luminate Online session variables.

``` js
angular.module('myApp').controller('myCtrl', ['$scope', '$luminateSessionVar', function($scope, $luminateSessionVar) {
  $luminateSessionVar.set('myVar', 'foo');
}]);
```

Both the `set` and `get` methods return a Promise, resolved with the value of the specified session variable.

``` js
$luminateSessionVar.get('myVar').then(function(response) {
  $scope.myVar = response;
});
```

## Browser Support

Browser support is largely dependent upon the version of AngularJS being used in your app, but for the most part, 
all major modern browsers are supported. See the [AngularJS FAQ](https://docs.angularjs.org/misc/faq#what-browsers-does-angularjs-work-with-) 
for more information. Note that cross-domain requests can only be made in those browsers with support for 
[Cross-Origin Resource Sharing (CORS)](http://www.w3.org/TR/cors/). For Internet Explorer specifically, this means IE10+.

## Reporting Issues

Should you encounter any issues when using this library, please report them here, using the 
["Issues"](https://github.com/noahcooper/angular-luminate-utils/issues) tab above. If you have general 
questions about the library, or about the API in general, the fastest way to get answers is to use 
the [Luminate Online](https://community.blackbaud.com/products/luminate) section on 
https://community.blackbaud.com.