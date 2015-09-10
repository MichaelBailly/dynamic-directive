# angular dynamic-directive

> inject directives dynamically to pre-defined anchor points

## Getting started

This module has been tested on angular `1.3.x`.

To use this module in your application, you have to:

* include the code in your main HTML file, after the angular.js inclusion

```xml
<script src='https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.3.18/angular.js'></script>
<script src='dynamic-directive/dist/dynamic-directive.min.js'></script>
```

* set `op.dynamicDirective` as a dependency of your angular module:

```javascript
angular.module('your-app', ['op.dynamicDirective'])...
```

now let the fun begin !

```javascript
angular.module('your-app', ['op.dynamicDirective'])
.directive('dir1', function() {
  return {
    restrict: 'E',
    template: '<div>Item X</div>'
  };
})
.directive('dir2', function() {
  return {
    restrict: 'E',
    template: '<div>Item Y</div>'
  };
})
.directive('addButton', ['dynamicDirectiveService', function(dynamicDirectiveService) {
  return {
    restrict: 'A',
    link: funtion(scope) {
      var dir2 = new dynamicDirectiveService.DynamicDirective(
        function(scope) {return true;},
        'dir2'
      );
      scope.add = function() {
        dynamicDirectiveService.addInjection('anchorPoint1', dir2);
      };
    }
  };
});
;
```

And the the HTML

```xml
<body ng-app='your-app'>
  <div class='items' dynamic-directive='anchorPoint1'>
    <dir1></dir1>
  </div>
  <div>
    <button add-button ng-click='add()'>Dynamically add the dir2 directive</button>
  </div>
</body>
```

## Goals

The objective of this library is to bring modularity to an Angular application. Let's say you are developing some complex Rich Internet Application, and you want lots of modularity, because that's best practice. Now, your application got a user menu. You want third party modules to be able to add entries to that menu. You use the `dynamic-directive` directive, and give it an anchor name :

```xml
<div dynamic-directive='userMenu'>
  <div>Profile</div>
</div>
```

Now, a third party module can inject entries to your user menu, by creating an Angular directive (let's say: 'avatar'), create a DynamicDirective out of it and then inject it using the dynamicDirectiveService.

```javascript
// third party directive
.directive('avatar', function() {
  return {
    restrict: 'E',
    template: '<div>avatar</div>',
    replace: true
  };
})

// directive injection can happen during the config phase,
// using the dynamicDirectiveServiceProvider
.config(['dynamicDirectiveServiceProvider', function(dynamicDirectiveServiceProvider) {
  // here "avatar" is the name of the angular directive
  var dd = new dynamicDirectiveServiceProvider.DynamicDirective(function() {return true;}, 'avatar');
  // here userMenu is the anchor name you gave as the dynamic-directive attribute value
  dynamicDirectiveServiceProvider.addInjection('userMenu', dd);
}])

// directive injection can also happen during the run phase,
// so anywhere in your code, using the dynamicDirectiveService service
.run(['dynamicDirectiveService', function(dynamicDirectiveService) {
  var dd = new dynamicDirectiveServiceProvider.DynamicDirective(function() {return true;}, 'avatar');
  dynamicDirectiveServiceProvider.addInjection('userMenu', dd);
}])
```
## API

### DynamicDirective class

    new DynamicDirective(filterFunction, angularDirectiveName, [attributes, [priority]]);

**filterFunction**

Required. This function receives the scope as an argument. Example:

```javascript
function(scope) {
  if (scope.isAdmin) {
    return true;
  } else {
    return false;
  }
}
```

The DynamicDirective contructor allows the boolean true as a shortcut for the filter function ```function() {return true;}```.

**angularDirectiveName**

Required. Will be used as the tagName to create the HTML tag of your directive. Example:

```javascript
"avatar"
```

**attributes**

Optional. Those attributes will be added to the HTML element of your injected directive.
This is really useful when using isolate scopes. Example:

```javascript
[
  {
    name: 'addressbook',
    value: 'ab'
  },
  {
    name: 'contact',
    value: 'currentContact'
  }
]
```

**priority**

Optional. In case you inject several directives in the same anchor, you can control the order of those directives in the DOM.
Default priority is 0. The highest priority will be put first in the DOM flow. Example:

```javascript
10
```

### dynamicDirectiveService service

**DynamicDirective**

```javascript
dynamicDirectiveService.DynamicDirective
```

Give access to the DynamicDirective object.

**addInjection(name, directive)**

    dynamicDirectiveService.addInjection(name, directive)

Add the injection of "directive" on anchor points named "name". Example:

```javascript
var dd = new dynamicDirectiveService.DynamicDirective(function() { return true;}, 'directivename');
dynamicDirectiveService.addInjection('anchorPointName', dd);
```

**getInjections(anchorName, scope)**

Get the injection of anchor point "anchorName" with scope context "scope". This method is used by the dynamic-directive directive. Example:

```javascript
var directivesToInject = dynamicDirectiveService.getInjection('anchorPointName', {});
```

### dynamicDirectiveServiceProvider provider

Allows third party modules to register the directives injection on angular configuration time.

**DynamicDirective**

    dynamicDirectiveService.DynamicDirective

Give access to the DynamicDirective object.

**addInjection(name, directive)**

```javascript
angular.config(function(dynamicDirectiveServiceProvider) {
  var dd = new dynamicDirectiveServiceProvider.DynamicDirective(function() { return true:}, 'thename');
  dynamicDirectiveServiceProvider.addInjection(name, directive);
});
```

## Developers ?

```bash
npm install
```

to install the development envronment. During development, use :

```bash
grunt watch
```

to have the linters, transpilers and tests, launched automatically on file change. Clone, patch, pull request, give love.