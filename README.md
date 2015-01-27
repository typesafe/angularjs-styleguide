# AngularJS Style Guide

## Naming Conventions

### File Names

- Use hyphenated file names: `app.ts`, `sales.ts`, `account.ts`, `reset-password.ts`
- Do not include pre- or suffixes in file names.

### Controllers

  - PascalCase e.g. ResetPasswordController

  *Why?*: Omitting the suffix is more succinct and the controller is often easily identifiable even without the suffix.

  - Use a `Controller` suffix (but not in file name)
  
*Why?*: The `Controller` suffix is more commonly used and is more explicitly descriptive.



### Services
  
- camelCase
- Prefix sevices with `$` (e.g. $sales, $session), this avoids name clashes with route parameters.
- 
### Directives

- use `vex` prefix, this makes them easier to recognize in templates code.

### Folder Structure

  - Declare modules in the `/js/src` folder
  - Declare module components in their respective folders in a subfolder matching the module name:
    - `/js/src/app/controllers`
    - `/js/src/app/directives`
    - `/js/src/app/filters`
    - `/js/src/app/services`

### Modules

Modules are declare in their own file in the `/js/src` folder. Besides their declaration, this file should only contain the module configuration (`module.config`) and initialization (`module.run`) code.

  ```javascript
  // /js/src/app.ts
  
  (function() {
    var appModule = angular.module('app', ['ngRoute', 'ui', 'account', 'sales']);
  
    appModule.config(['$routeProvider', ($routeProvider) => {
  
          $routeProvider.mapRoute('home', '/', {
              templateUrl: '/templates/app/home-page',
              controller: 'HomePageController',
              resolve: {
                  sales: ['$sales', $sales => $sales.getSales()]
              }
          });
      }]);
    
     appModule.run(['$rootScope', function ($rootScope) {
     // init code goes here
     }]);
   })();
  ```
  
### Module Components

- Define only a single component per file in their corresponding module folder.
- Declare components using an IIFE.

  ```javascript
  // /js/src/account/controllers/login-overlay.ts
  (function (accountModule) {

    accountModule.controller('LoginOverlayController', ['$scope', '$session', function ($scope, $session) {
        ...
    }]);

  })(angular.module("account"));
  ```

## IIFE

### JavaScript Closures

  - Wrap AngularJS components in an Immediately Invoked Function Expression (IIFE).
  - Pass the module to the IIFE.

  ```javascript
  (function (module) {

    module.controller('ComponentController', ['$scope', '$session', function ($scope, $session) {
        //...
    }]);

  })(angular.module("moduleName"));
  ```
  
  *Why?*: An IIFE removes variables from the global scope. This helps prevent variables and function declarations from living longer than expected in the global scope, which also helps avoid variable collisions.

  *Why?*: When your code is minified and bundled into a single file for deployment to a production server, you could have collisions of variables and many global variables. An IIFE protects you against both of these by providing variable scope for each file.

  - Note: IIFE's prevent test code from reaching private members like regular expressions or helper functions which are often good to unit test directly on their own. However you can test these through accessible members or by exposing them through their own component. For example placing helper functions, regular expressions or constants in their own factory or constant.

## Controllers

### controllerAs View Syntax

  - Consider using the [`controllerAs`](http://www.johnpapa.net/do-you-like-your-angular-controllers-with-or-without-sugar/) syntax over the `classic controller with $scope` syntax.
  - Alternatively, add a "view model" (e.g. `$scope.account = {}`) variable to `$scope` that aggregates all model values.
  - Avoid adding simple values to `$scope` directly (e.g. `$scope.email = 'foo@bar.com';`)

  *Why?*: It promotes the use of binding to a "dotted" object in the View (e.g. `customer.name` instead of `name`), which is more contextual, easier to read, and avoids any reference issues that may occur without "dotting".

  *Why?*: Helps avoid using `$parent` calls in Views with nested controllers.

  ```html
  <div ng-controller="CustomerController as customer">
      {{ customer.name }}
  </div>
  ```

### controllerAs Controller Syntax

  - Use the `controllerAs` syntax over the `classic controller with $scope` syntax. 

  - The `controllerAs` syntax uses `this` inside controllers which gets bound to `$scope`

  *Why?*: `controllerAs` is syntactic sugar over `$scope`. You can still bind to the View and still access `$scope` methods.

  *Why?*: Helps avoid the temptation of using `$scope` methods inside a controller when it may otherwise be better to avoid them or move them to a factory. Consider using `$scope` in a factory, or if in a controller just when needed. For example when publishing and subscribing events using [`$emit`](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$emit), [`$broadcast`](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$broadcast), or [`$on`](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$on) consider moving these uses to a factory and invoke from the controller. 

  - Use a capture variable for `this` when using the `controllerAs` syntax. Choose a consistent variable name such as `vm`, which stands for ViewModel.
  
  *Why?*: The `this` keyword is contextual and when used within a function inside a controller may change its context. Capturing the context of `this` avoids encountering this problem.

  ```javascript
  function Customer() {
      var vm = this;
      vm.name = {};
      vm.sendMessage = function() { };
  }
  ```

  Note: When creating watches in a controller using `controller as`, you can watch the `vm.*` member using the following syntax. (Create watches with caution as they add more load to the digest cycle.)

  ```html
  <input ng-model="vm.title"/>
  ```

  ```javascript
  function SomeController($scope, $log) {
      var vm = this;
      vm.title = 'Some Title';

      $scope.$watch('vm.title', function(current, original) {
          $log.info('vm.title was %s', original);
          $log.info('vm.title is now %s', current);
      });
  }
  ```

### Bindable Members Up Top

  - Place bindable members at the top of the controller, and not spread through the controller code.

    *Why?*: Placing bindable members at the top makes it easy to read and helps you instantly identify which members of the controller can be bound and used in the View. 

    *Why?*: Setting anonymous functions in-line can be easy, but when those functions are more than 1 line of code they can reduce the readability. Defining the functions below the bindable members (the functions will be hoisted) moves the implementation details down, keeps the bindable members up top, and makes it easier to read. 

  ```javascript
  function Sessions() {
      var vm = this;

      vm.gotoSession = gotoSession;
      vm.refresh = refresh;
      vm.search = search;
      vm.sessions = [];
      vm.title = 'Sessions';

      function gotoSession() {
        /* */
      }

      function refresh() {
        /* */
      }

      function search() {
        /* */
      }
  ```

  Note: If the function is a 1 liner consider keeping it right up top, as long as readability is not affected.

  ```javascript
  /* recommended */
  function Sessions(dataservice) {
      var vm = this;

      vm.gotoSession = gotoSession;
      vm.refresh = dataservice.refresh; // 1 liner is OK
      vm.search = search;
      vm.sessions = [];
      vm.title = 'Sessions';
  ```


### Defer Controller Logic

  - Defer logic in a controller by delegating to services and factories.

    *Why?*: Logic may be reused by multiple controllers when placed within a service and exposed via a function.

    *Why?*: Logic in a service can more easily be isolated in a unit test, while the calling logic in the controller can be easily mocked.

    *Why?*: Removes dependencies and hides implementation details from the controller.


  ```javascript
  function Order($credit) {
      var vm = this;
      vm.checkCredit = checkCredit;
      vm.isCreditOk;
      vm.total = 0;

      function checkCredit() { 
         return $credit.isOrderTotalOk(vm.total);
      .then(function(isOk) { vm.isCreditOk = isOk; })
            .catch(showServiceError);
      };
  }
  ```

### Keep Controllers Focused

  - Define a controller for a view, and try not to reuse the controller for other views. Instead, move reusable logic to factories and keep the controller simple and focused on its view. 

    *Why?*: Reusing controllers with several views is brittle and good end to end (e2e) test coverage is required to ensure stability across large applications.

### Assigning Controllers

  - When a controller must be paired with a view and either component may be re-used by other controllers or views, define controllers along with their routes. 

    Note: If a View is loaded via another means besides a route, then use the `ng-controller="Avengers as vm"` syntax. 

    *Why?*: Pairing the controller in the route allows different routes to invoke different pairs of controllers and views. When controllers are assigned in the view using [`ng-controller`](https://docs.angularjs.org/api/ng/directive/ngController), that view is always associated with the same controller.

  ```javascript
  /* recommended */

  // route-config.js
  angular
      .module('app')
      .config(config);

  function config($routeProvider) {
      $routeProvider
          .when('/avengers', {
              templateUrl: 'avengers.html',
              controller: 'Avengers',
              controllerAs: 'vm'
          });
  }
  ```

  ```html
  <!-- avengers.html -->
  <div>
  </div>
  ```

## Services

### Singletons

  - Services are instantiated with the `new` keyword, use `this` for public methods and variables.

    Note: [All AngularJS services are singletons](https://docs.angularjs.org/guide/services). This means that there is only one instance of a given service per injector.

## Factories

### Single Responsibility

  - Factories should have a [single responsibility](http://en.wikipedia.org/wiki/Single_responsibility_principle), that is encapsulated by its context. Once a factory begins to exceed that singular purpose, a new factory should be created.

### Singletons

  - Factories are singletons and return an object that contains the members of the service.

    Note: [All AngularJS services are singletons](https://docs.angularjs.org/guide/services).

## Data Services

### Separate Data Calls

  - Refactor logic for making data operations and interacting with data to a factory. Make data services responsible for XHR calls, local storage, stashing in memory, or any other data operations.

    *Why?*: The controller's responsibility is for the presentation and gathering of information for the view. It should not care how it gets the data, just that it knows who to ask for it. Separating the data services moves the logic on how to get it to the data service, and lets the controller be simpler and more focused on the view.

    *Why?*: This makes it easier to test (mock or real) the data calls when testing a controller that uses a data service.

    *Why?*: Data service implementation may have very specific code to handle the data repository. This may include headers, how to talk to the data, or other services such as $http. Separating the logic into a data service encapsulates this logic in a single place hiding the implementation from the outside consumers (perhaps a controller), also making it easier to change the implementation.

### Return a Promise from Data Calls

  - When calling a data service that returns a promise such as $http, return a promise in your calling function too.

    *Why?*: You can chain the promises together and take further action after the data call completes and resolves or rejects the promise.

  ```javascript

  function activate() {
      /**
       * Step 1
       * Ask the getAvengers function for the
       * avenger data and wait for the promise
       */
      return getAvengers().then(function() {
          /**
           * Step 4
           * Perform an action on resolve of final promise
           */
          logger.info('Activated Avengers View');
      });
  }

  function getAvengers() {
        /**
         * Step 2
         * Ask the data service for the data and wait
         * for the promise
         */
        return dataservice.getAvengers()
            .then(function(data) {
                /**
                 * Step 3
                 * set the data and resolve the promise
                 */
                vm.avengers = data;
                return vm.avengers;
        });
  }
  ```

## Directives

### Manipulate DOM in a Directive

  - When manipulating the DOM directly, use a directive. If alternative ways can be used such as using CSS to set styles or the [animation services](https://docs.angularjs.org/api/ngAnimate), Angular templating, [`ngShow`](https://docs.angularjs.org/api/ng/directive/ngShow) or [`ngHide`](https://docs.angularjs.org/api/ng/directive/ngHide), then use those instead. For example, if the directive simply hides and shows, use ngHide/ngShow. 

    *Why?*: DOM manipulation can be difficult to test, debug, and there are often better ways (e.g. CSS, animations, templates)

### Prefix directives with `vex`

  - Provide a short, unique and descriptive directive prefix such as `acmeSalesCustomerInfo` which is declared in HTML as `acme-sales-customer-info`.

    *Why?*: The unique short prefix identifies the directive's context and origin. For example a prefix of `cc-` may indicate that the directive is part of a CodeCamper app while `acme-` may indicate a directive for the Acme company. 

    Note: Avoid `ng-` as these are reserved for AngularJS directives. Research widely used directives to avoid naming conflicts, such as `ion-` for the [Ionic Framework](http://ionicframework.com/). 

### Restrict to Attributes

  - Restrict directives to `A` (custom attribute), this makes directives consistently recognizable and also allows for passing data (e.g. `vex-product-list='products'`)


  ```html
  
  <div vex-product-list='products'></div>
  ```
  
  ```javascript
  (function(salesModule){
  
    salesModule.directive('vexProductList', [function(){
      return {
        restrict: 'A',
        templateUrl: '...',
        link: function(scope, element, attrs){
          ...
        }
      };
    }]);
  })(angular.module('sales'));
  ```

## Resolving Promises for a Controller

### Route Resolve Promises

  - When a controller depends on a promise to be resolved before the controller is activated, resolve those dependencies in the `$routeProvider` before the controller logic is executed. If you need to conditionally cancel a route before the controller is activated, use a route resolver.

  - Use a route resolve when you want to decide to cancel the route before ever transitioning to the View.

    *Why?*: A controller may require data before it loads. That data may come from a promise via a custom factory or [$http](https://docs.angularjs.org/api/ng/service/$http). Using a [route resolve](https://docs.angularjs.org/api/ngRoute/provider/$routeProvider) allows the promise to resolve before the controller logic executes, so it might take action based on that data from the promise.

    *Why?*: The code executes after the route and in the controller’s activate function. The View starts to load right away. Data binding kicks in when the activate promise resolves. A “busy” animation can be shown during the view transition (via ng-view or ui-view)


  ```javascript
  
  // route-config.js
  angular
      .module('app')
      .config(config);

  function config($routeProvider) {
      $routeProvider
          .when('/avengers', {
              templateUrl: 'avengers.html',
              controller: 'Avengers',
              controllerAs: 'vm',
              resolve: {
                  moviesPrepService: function(movieService) {
                      return movieService.getMovies();
                  }
              }
          });
  }

  // avengers.js
  angular
      .module('app')
      .controller('Avengers', Avengers);

  Avengers.$inject = ['moviesPrepService'];
  function Avengers(moviesPrepService) {
        var vm = this;
        vm.movies = moviesPrepService.movies;
  }
  ```

## Exception Handling

### decorators

  - Use a [decorator](https://docs.angularjs.org/api/auto/service/$provide#decorator), at config time using the [`$provide`](https://docs.angularjs.org/api/auto/service/$provide) service, on the [`$exceptionHandler`](https://docs.angularjs.org/api/ng/service/$exceptionHandler) service to perform custom actions when exceptions occur.
  
    *Why?*: Provides a consistent way to handle uncaught AngularJS exceptions for development-time or run-time.

    Note: Another option is to override the service instead of using a decorator. This is a fine option, but if you want to keep the default behavior and extend it a decorator is recommended.

    ```javascript
    /* recommended */
    angular
        .module('blocks.exception')
        .config(exceptionConfig);

    exceptionConfig.$inject = ['$provide'];

    function exceptionConfig($provide) {
        $provide.decorator('$exceptionHandler', extendExceptionHandler);
    }

    extendExceptionHandler.$inject = ['$delegate', 'toastr'];

    function extendExceptionHandler($delegate, toastr) {
        return function(exception, cause) {
            $delegate(exception, cause);
            var errorData = { 
                exception: exception, 
                cause: cause 
            };
            /**
             * Could add the error to a service's collection,
             * add errors to $rootScope, log errors to remote web server,
             * or log locally. Or throw hard. It is entirely up to you.
             * throw exception;
             */
            toastr.error(exception.msg, errorData);
        };
    }
    ```

### Exception Catchers

  - Create a factory that exposes an interface to catch and gracefully handle exceptions.

    *Why?*: Provides a consistent way to catch exceptions that may be thrown in your code (e.g. during XHR calls or promise failures).

    Note: The exception catcher is good for catching and reacting to specific exceptions from calls that you know may throw one. For example, when making an XHR call to retrieve data from a remote web service and you want to catch any exceptions from that service and react uniquely.

    ```javascript
    /* recommended */
    angular
        .module('blocks.exception')
        .factory('exception', exception);

    exception.$inject = ['logger'];

    function exception(logger) {
        var service = {
            catcher: catcher
        };
        return service;

        function catcher(message) {
            return function(reason) {
                logger.error(message, reason);
            };
        }
    }
    ```

### Route Errors

  - Handle and log all routing errors using [`$routeChangeError`](https://docs.angularjs.org/api/ngRoute/service/$route#$routeChangeError).

    *Why?*: Provides a consistent way handle all routing errors.

    *Why?*: Potentially provides a better user experience if a routing error occurs and you route them to a friendly screen with more details or recovery options.

    ```javascript
    /* recommended */
    function handleRoutingErrors() {
        /**
         * Route cancellation:
         * On routing error, go to the dashboard.
         * Provide an exit clause if it tries to do it twice.
         */
        $rootScope.$on('$routeChangeError',
            function(event, current, previous, rejection) {
                var destination = (current && (current.title || current.name || current.loadedTemplateUrl)) ||
                    'unknown target';
                var msg = 'Error routing to ' + destination + '. ' + (rejection.msg || '');
                /**
                 * Optionally log using a custom service or $log.
                 * (Don't forget to inject custom service)
                 */
                logger.warning(msg, [current]);
            }
        );
    }
    ```


## File Templates and Snippets
Use file templates or snippets to help follow consistent styles and patterns. Here are templates and/or snippets for some of the web development editors and IDEs.

### Sublime Text
###### [Style [Y250](#style-y250)]

  - AngularJS snippets that follow these styles and guidelines. 

    - Download the [Sublime Angular snippets](assets/sublime-angular-snippets.zip?raw=true) 
    - Place it in your Packages folder
    - Restart Sublime 
    - In a JavaScript file type these commands followed by a `TAB`
 
    ```javascript
    ngcontroller // creates an Angular controller
    ngdirective // creates an Angular directive
    ngfactory // creates an Angular factory
    ngmodule // creates an Angular module
    ```
