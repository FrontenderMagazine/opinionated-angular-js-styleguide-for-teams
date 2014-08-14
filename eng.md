<article class="post clear">
##### Official styleguide repo [now on GitHub][1], all future styleguide
updates will be here!

After reading [Google's AngularJS guidelines][2], I felt they were a little too
incomplete and also guided towards using the Closure library. They
[also state][3] *"We don't think this makes sense for all projects that use
AngularJS, and we'd love to see our community of developers come up with a more 
general Style that's applicable to AngularJS projects large and small
"*, so here goes.

From my experience with Angular, [several talks][4] and working in teams, here'
s my opinionated styleguide for syntax, building and structuring Angular 
applications.

### Module definitions

Angular modules can be declared in various ways, either stored in a variable or
using the getter syntax. Use the getter syntax at all times
([angular recommended][5]).

###### Bad:

    <span class="kd">var</span> <span class="nx">app</span> <span class="o">=</span> <span class="nx">angular</span><span class="p">.</span><span class="nx">module</span><span class="p">(</span><span class="s1">'app'</span><span class="p">,</span> <span class="p">[]);</span>
    <span class="nx">app</span><span class="p">.</span><span class="nx">controller</span><span class="p">();</span>
    <span class="nx">app</span><span class="p">.</span><span class="nx">factory</span><span class="p">();</span>

###### Good:

    <span class="nx">angular</span>
      <span class="p">.</span><span class="nx">module</span><span class="p">(</span><span class="s1">'app'</span><span class="p">,</span> <span class="p">[])</span>
      <span class="p">.</span><span class="nx">controller</span><span class="p">()</span>
      <span class="p">.</span><span class="nx">factory</span><span class="p">();</span>

From these modules we can pass in function references.

### Module method functions

Angular modules have a lot of methods, such as `controller`, `factory`, 
`directive`, `service` and more. There are many syntaxes for these modules when
it comes to dependency injection and formatting your code. Use a named function 
definition and pass it into the relevant module method, this aids in stack 
traces as functions aren't anonymous (this could be solved by naming the 
anonymous function but this method is far cleaner
).

###### Bad:

    <span class="kd">var</span> <span class="nx">app</span> <span class="o">=</span> <span class="nx">angular</span><span class="p">.</span><span class="nx">module</span><span class="p">(</span><span class="s1">'app'</span><span class="p">,</span> <span class="p">[]);</span>
    <span class="nx">app</span><span class="p">.</span><span class="nx">controller</span><span class="p">(</span><span class="s1">'MyCtrl'</span><span class="p">,</span> <span class="kd">function</span> <span class="p">()</span> <span class="p">{</span>
      
    <span class="p">});</span>

###### Good:

    <span class="kd">function</span> <span class="nx">MainCtrl</span> <span class="p">()</span> <span class="p">{</span>
      
    <span class="p">}</span>
    <span class="nx">angular</span>
      <span class="p">.</span><span class="nx">module</span><span class="p">(</span><span class="s1">'app'</span><span class="p">,</span> <span class="p">[])</span>
      <span class="p">.</span><span class="nx">controller</span><span class="p">(</span><span class="s1">'MainCtrl'</span><span class="p">,</span> <span class="nx">MainCtrl</span><span class="p">);</span>

Define a module once using `angular.module('app', [])` setter, then use the 
`angular.module('app')` getter elsewhere (such as other files).

To avoid polluting the global namespace, wrap all your functions during *
compilation/concatenation* inside an IIFE which will produce something like
this:

###### Best:

    <span class="p">(</span><span class="kd">function</span> <span class="p">()</span> <span class="p">{</span>
      <span class="nx">angular</span><span class="p">.</span><span class="nx">module</span><span class="p">(</span><span class="s1">'app'</span><span class="p">,</span> <span class="p">[]);</span>
      
      <span class="c1">// MainCtrl.js</span>
      <span class="kd">function</span> <span class="nx">MainCtrl</span> <span class="p">()</span> <span class="p">{</span>
      
      <span class="p">}</span>
      
      <span class="nx">angular</span>
        <span class="p">.</span><span class="nx">module</span><span class="p">(</span><span class="s1">'app'</span><span class="p">)</span>
        <span class="p">.</span><span class="nx">controller</span><span class="p">(</span><span class="s1">'MainCtrl'</span><span class="p">,</span> <span class="nx">MainCtrl</span><span class="p">);</span>
        
      <span class="c1">// AnotherCtrl.js</span>
      <span class="kd">function</span> <span class="nx">AnotherCtrl</span> <span class="p">()</span> <span class="p">{</span>
      
      <span class="p">}</span>
      
      <span class="nx">angular</span>
        <span class="p">.</span><span class="nx">module</span><span class="p">(</span><span class="s1">'app'</span><span class="p">)</span>
        <span class="p">.</span><span class="nx">controller</span><span class="p">(</span><span class="s1">'AnotherCtrl'</span><span class="p">,</span> <span class="nx">AnotherCtrl</span><span class="p">);</span>
        
      <span class="c1">// and so on...</span>
        
    <span class="p">})();</span>

### Controllers

Controllers are classes and can use a `controllerAs` syntax or generic 
`controller` syntax. Use the `controllerAs` syntax always as it aids in nested
scoping and controller instance reference.

##### controllerAs DOM bindings

###### Bad:

    <span class="nt"><div</span> <span class="na">ng-controller=</span><span class="s">"MainCtrl"</span><span class="nt">></span>
      {{ someObject }}
    <span class="nt"></div></span>

###### Good:

    <span class="nt"><div</span> <span class="na">ng-controller=</span><span class="s">"MainCtrl as main"</span><span class="nt">></span>
      {{ main.someObject }}
    <span class="nt"></div></span>

Binding these `ng-controller` attributes couples the declarations tightly with
our DOM, and also means we can only use that controller for that specific view (
there are rare cases we might use the same view with different controllers). Use
the router to couple the controller declarations with the relevant views by 
telling each`route` what controller to instantiate.

###### Best:

    <span class="c"><!-- main.html --></span>
    <span class="nt"><div></span>
      {{ main.someObject }}
    <span class="nt"></div></span>
    <span class="c"><!-- main.html --></span>
    
    <span class="nt"><script></span>
    <span class="c1">// ...</span>
    <span class="kd">function</span> <span class="nx">config</span> <span class="p">(</span><span class="nx">$routeProvider</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">$routeProvider</span>
      <span class="p">.</span><span class="nx">when</span><span class="p">(</span><span class="s1">'/'</span><span class="p">,</span> <span class="p">{</span>
        <span class="nx">templateUrl</span><span class="o">:</span> <span class="s1">'views/main.html'</span><span class="p">,</span>
        <span class="nx">controller</span><span class="o">:</span> <span class="s1">'MainCtrl'</span><span class="p">,</span>
        <span class="nx">controllerAs</span><span class="o">:</span> <span class="s1">'main'</span>
      <span class="p">});</span>
    <span class="p">}</span>
    <span class="nx">angular</span>
      <span class="p">.</span><span class="nx">module</span><span class="p">(</span><span class="s1">'app'</span><span class="p">)</span>
      <span class="p">.</span><span class="nx">config</span><span class="p">(</span><span class="nx">config</span><span class="p">);</span>
    <span class="c1">//...</span>
    <span class="nt"></script></span>

This avoids using `$parent` to access any parent controllers from a child
controller, simple hit the`main` reference and you've got it. This could avoid
things such as`$parent.$parent` calls.

##### controllerAs this keyword

The `controllerAs` syntax uses the `this` keyword inside controllers instead of
`$scope`. When using `controllerAs`, the controller is infact *bound* to 
`$scope`, there is a degree of separation.

###### Bad:

    <span class="kd">function</span> <span class="nx">MainCtrl</span> <span class="p">(</span><span class="nx">$scope</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">$scope</span><span class="p">.</span><span class="nx">someObject</span> <span class="o">=</span> <span class="p">{};</span>
      <span class="nx">$scope</span><span class="p">.</span><span class="nx">doSomething</span> <span class="o">=</span> <span class="kd">function</span> <span class="p">()</span> <span class="p">{</span>
      
      <span class="p">};</span>
    <span class="p">}</span>
    <span class="nx">angular</span>
      <span class="p">.</span><span class="nx">module</span><span class="p">(</span><span class="s1">'app'</span><span class="p">)</span>
      <span class="p">.</span><span class="nx">controller</span><span class="p">(</span><span class="s1">'MainCtrl'</span><span class="p">,</span> <span class="nx">MainCtrl</span><span class="p">);</span>

You can also use the `prototype` Object to create controller classes, but this
becomes messy very quickly as each dependency injected provider needs a 
reference bound to the`constructor` Object.

###### Bad and Good:

Good for inheritance, bad (verbose) for general use.

    <span class="kd">function</span> <span class="nx">MainCtrl</span> <span class="p">(</span><span class="nx">$scope</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">this</span><span class="p">.</span><span class="nx">someObject</span> <span class="o">=</span> <span class="p">{};</span>
      <span class="k">this</span><span class="p">.</span><span class="nx">_$scope</span> <span class="o">=</span> <span class="nx">$scope</span><span class="p">;</span>
    <span class="p">}</span>
    <span class="nx">MainCtrl</span><span class="p">.</span><span class="nx">prototype</span><span class="p">.</span><span class="nx">doSomething</span> <span class="o">=</span> <span class="kd">function</span> <span class="p">()</span> <span class="p">{</span>
      <span class="c1">// use this._$scope</span>
    <span class="p">};</span>
    <span class="nx">angular</span>
      <span class="p">.</span><span class="nx">module</span><span class="p">(</span><span class="s1">'app'</span><span class="p">)</span>
      <span class="p">.</span><span class="nx">controller</span><span class="p">(</span><span class="s1">'MainCtrl'</span><span class="p">,</span> <span class="nx">MainCtrl</span><span class="p">);</span>

If you're using `prototype` and don't know why, then it's bad. If you are using
`prototype` to inherit from other controllers, then that's good. For general
use, the`prototype` pattern can be verbose.

###### Good:

    <span class="kd">function</span> <span class="nx">MainCtrl</span> <span class="p">()</span> <span class="p">{</span>
      <span class="k">this</span><span class="p">.</span><span class="nx">someObject</span> <span class="o">=</span> <span class="p">{};</span>
      <span class="k">this</span><span class="p">.</span><span class="nx">doSomething</span> <span class="o">=</span> <span class="kd">function</span> <span class="p">()</span> <span class="p">{</span>
      
      <span class="p">};</span>
    <span class="p">}</span>
    <span class="nx">angular</span>
      <span class="p">.</span><span class="nx">module</span><span class="p">(</span><span class="s1">'app'</span><span class="p">)</span>
      <span class="p">.</span><span class="nx">controller</span><span class="p">(</span><span class="s1">'MainCtrl'</span><span class="p">,</span> <span class="nx">MainCtrl</span><span class="p">);</span>

These just show examples of Objects/functions inside Controllers, however we
don't want to put logic in controllers.
..

##### Avoid controller logic

Avoid writing logic in Controllers, delegate to Factories/Services.

###### Bad:

    <span class="kd">function</span> <span class="nx">MainCtrl</span> <span class="p">()</span> <span class="p">{</span>
      <span class="k">this</span><span class="p">.</span><span class="nx">doSomething</span> <span class="o">=</span> <span class="kd">function</span> <span class="p">()</span> <span class="p">{</span>
    
      <span class="p">};</span>
    <span class="p">}</span>
    <span class="nx">angular</span>
      <span class="p">.</span><span class="nx">module</span><span class="p">(</span><span class="s1">'app'</span><span class="p">)</span>
      <span class="p">.</span><span class="nx">controller</span><span class="p">(</span><span class="s1">'MainCtrl'</span><span class="p">,</span> <span class="nx">MainCtrl</span><span class="p">);</span>

###### Good:

    <span class="kd">function</span> <span class="nx">MainCtrl</span> <span class="p">(</span><span class="nx">SomeService</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">this</span><span class="p">.</span><span class="nx">doSomething</span> <span class="o">=</span> <span class="nx">SomeService</span><span class="p">.</span><span class="nx">doSomething</span><span class="p">;</span>
    <span class="p">}</span>
    <span class="nx">angular</span>
      <span class="p">.</span><span class="nx">module</span><span class="p">(</span><span class="s1">'app'</span><span class="p">)</span>
      <span class="p">.</span><span class="nx">controller</span><span class="p">(</span><span class="s1">'MainCtrl'</span><span class="p">,</span> <span class="nx">MainCtrl</span><span class="p">);</span>

This maximises reusability, encapsulated functionality and makes testing far
easier and persistent.

### Services

Services are instantiated and should be class-like also and reference the 
`this` keyword, keep function style consistent with everything else.

###### Good:

    <span class="kd">function</span> <span class="nx">SomeService</span> <span class="p">()</span> <span class="p">{</span>
      <span class="k">this</span><span class="p">.</span><span class="nx">someMethod</span> <span class="o">=</span> <span class="kd">function</span> <span class="p">()</span> <span class="p">{</span>
    
      <span class="p">};</span>
    <span class="p">}</span>
    <span class="nx">angular</span>
      <span class="p">.</span><span class="nx">module</span><span class="p">(</span><span class="s1">'app'</span><span class="p">)</span>
      <span class="p">.</span><span class="nx">service</span><span class="p">(</span><span class="s1">'SomeService'</span><span class="p">,</span> <span class="nx">SomeService</span><span class="p">);</span>

### Factory

Factories give us a singleton module for creating service methods (such as
communicating with a server over REST endpoints). Creating and returning a bound
Object keeps controller bindings up to date and avoids pitfalls of binding 
primitive values.

Important: A "factory" is in fact a pattern/implementation, and shouldn't be
part of the provider's name. All factories and services should be called "
services
".

###### Bad:

    <span class="kd">function</span> <span class="nx">AnotherService</span> <span class="p">()</span> <span class="p">{</span>
    
      <span class="kd">var</span> <span class="nx">someValue</span> <span class="o">=</span> <span class="s1">''</span><span class="p">;</span>
    
      <span class="kd">var</span> <span class="nx">someMethod</span> <span class="o">=</span> <span class="kd">function</span> <span class="p">()</span> <span class="p">{</span>
    
      <span class="p">};</span>
      
      <span class="k">return</span> <span class="p">{</span>
        <span class="nx">someValue</span><span class="o">:</span> <span class="nx">someValue</span><span class="p">,</span>
        <span class="nx">someMethod</span><span class="o">:</span> <span class="nx">someMethod</span>
      <span class="p">};</span>
    
    <span class="p">}</span>
    <span class="nx">angular</span>
      <span class="p">.</span><span class="nx">module</span><span class="p">(</span><span class="s1">'app'</span><span class="p">)</span>
      <span class="p">.</span><span class="nx">factory</span><span class="p">(</span><span class="s1">'AnotherService'</span><span class="p">,</span> <span class="nx">AnotherService</span><span class="p">);</span>

###### Good:

We create an Object with the same name inside the function. This can aid
documentation as well for comment-generated docs.

    <span class="kd">function</span> <span class="nx">AnotherService</span> <span class="p">()</span> <span class="p">{</span>
    
      <span class="kd">var</span> <span class="nx">AnotherService</span> <span class="o">=</span> <span class="p">{};</span>
      
      <span class="nx">AnotherService</span><span class="p">.</span><span class="nx">someValue</span> <span class="o">=</span> <span class="s1">''</span><span class="p">;</span>
    
      <span class="nx">AnotherService</span><span class="p">.</span><span class="nx">someMethod</span> <span class="o">=</span> <span class="kd">function</span> <span class="p">()</span> <span class="p">{</span>
    
      <span class="p">};</span>
      
      <span class="k">return</span> <span class="nx">AnotherService</span><span class="p">;</span>
    <span class="p">}</span>
    <span class="nx">angular</span>
      <span class="p">.</span><span class="nx">module</span><span class="p">(</span><span class="s1">'app'</span><span class="p">)</span>
      <span class="p">.</span><span class="nx">factory</span><span class="p">(</span><span class="s1">'AnotherService'</span><span class="p">,</span> <span class="nx">AnotherService</span><span class="p">);</span>

Any bindings to primitives are kept up to date, and it makes internal module
namespacing a little easier, we can easily see any private methods and variables.

### Directives

Any DOM manipulation should take place inside a directive, and only directives
. Any code reusability should be encapsulated (behavioural and markup related) 
too.

##### DOM manipulation

DOM manipulation should be done inside the `link` method of a directive.

###### Bad:

    <span class="c1">// do not use a controller</span>
    <span class="kd">function</span> <span class="nx">MainCtrl</span> <span class="p">(</span><span class="nx">SomeService</span><span class="p">)</span> <span class="p">{</span>
    
      <span class="k">this</span><span class="p">.</span><span class="nx">makeActive</span> <span class="o">=</span> <span class="kd">function</span> <span class="p">(</span><span class="nx">elem</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">elem</span><span class="p">.</span><span class="nx">addClass</span><span class="p">(</span><span class="s1">'test'</span><span class="p">);</span>
      <span class="p">};</span>
    
    <span class="p">}</span>
    <span class="nx">angular</span>
      <span class="p">.</span><span class="nx">module</span><span class="p">(</span><span class="s1">'app'</span><span class="p">)</span>
      <span class="p">.</span><span class="nx">controller</span><span class="p">(</span><span class="s1">'MainCtrl'</span><span class="p">,</span> <span class="nx">MainCtrl</span><span class="p">);</span>

###### Good:

    <span class="c1">// use a directive</span>
    <span class="kd">function</span> <span class="nx">SomeDirective</span> <span class="p">(</span><span class="nx">SomeService</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">return</span> <span class="p">{</span>
        <span class="nx">restrict</span><span class="o">:</span> <span class="s1">'EA'</span><span class="p">,</span>
        <span class="nx">template</span><span class="o">:</span> <span class="p">[</span>
          <span class="s1">'<a href="" class="myawesomebutton" ng-transclude>'</span><span class="p">,</span>
            <span class="s1">'<i class="icon-ok-sign"></i>'</span><span class="p">,</span>
          <span class="s1">'</a>'</span>
        <span class="p">].</span><span class="nx">join</span><span class="p">(</span><span class="s1">''</span><span class="p">),</span>
        <span class="nx">link</span><span class="o">:</span> <span class="kd">function</span> <span class="p">(</span><span class="nx">$scope</span><span class="p">,</span> <span class="nx">$element</span><span class="p">,</span> <span class="nx">$attrs</span><span class="p">)</span> <span class="p">{</span>
          <span class="c1">// DOM manipulation/events here!</span>
          <span class="nx">$element</span><span class="p">.</span><span class="nx">on</span><span class="p">(</span><span class="s1">'click'</span><span class="p">,</span> <span class="kd">function</span> <span class="p">()</span> <span class="p">{</span>
            <span class="nx">$</span><span class="p">(</span><span class="k">this</span><span class="p">).</span><span class="nx">addClass</span><span class="p">(</span><span class="s1">'test'</span><span class="p">);</span>
          <span class="p">});</span>
        <span class="p">}</span>
      <span class="p">};</span>
    <span class="p">}</span>
    <span class="nx">angular</span>
      <span class="p">.</span><span class="nx">module</span><span class="p">(</span><span class="s1">'app'</span><span class="p">)</span>
      <span class="p">.</span><span class="nx">directive</span><span class="p">(</span><span class="s1">'SomeDirective'</span><span class="p">,</span> <span class="nx">SomeDirective</span><span class="p">);</span>

Any DOM manipulation should take place inside a directive, and only directives
. Any code reusability should be encapsulated (behavioural and markup related) 
too.

##### Naming conventions

Custom directives should *not* be `ng-*` prefixed to prevent future core
overrides if your directive name happens to land in Angular (such as when
`ng-focus` landed, there were many custom directives called this beforehand).
It also makes it more confusing to know which are core directives and which are 
custom.

###### Bad:

    <span class="kd">function</span> <span class="nx">ngFocus</span> <span class="p">(</span><span class="nx">SomeService</span><span class="p">)</span> <span class="p">{</span>
    
      <span class="k">return</span> <span class="p">{};</span>
    
    <span class="p">}</span>
    <span class="nx">angular</span>
      <span class="p">.</span><span class="nx">module</span><span class="p">(</span><span class="s1">'app'</span><span class="p">)</span>
      <span class="p">.</span><span class="nx">directive</span><span class="p">(</span><span class="s1">'ngFocus'</span><span class="p">,</span> <span class="nx">ngFocus</span><span class="p">);</span>

###### Good:

    <span class="kd">function</span> <span class="nx">focusFire</span> <span class="p">(</span><span class="nx">SomeService</span><span class="p">)</span> <span class="p">{</span>
    
      <span class="k">return</span> <span class="p">{};</span>
    
    <span class="p">}</span>
    <span class="nx">angular</span>
      <span class="p">.</span><span class="nx">module</span><span class="p">(</span><span class="s1">'app'</span><span class="p">)</span>
      <span class="p">.</span><span class="nx">directive</span><span class="p">(</span><span class="s1">'focusFire'</span><span class="p">,</span> <span class="nx">focusFire</span><span class="p">);</span>

Directives are the *only* providers that we have the first letter as lowercase
, this is due to strict naming conventions in the way Angular translates
`camelCase` to hyphenated, so `focusFire` will become 
`<input focus-fire>` when used on an element.

##### Usage restriction

If you need to support IE8, you'll want to avoid using the comments syntax for
declaring where a directive will sit. Really, this syntax should be avoided 
anyway - there are no real benefits of using it - it just adds confusion of what
is a comment and what isn't.

###### Bad:

These are terribly confusing.

    <span class="c"><!-- directive: my-directive --></span>
    <span class="nt"><div</span> <span class="na">class=</span><span class="s">"my-directive"</span><span class="nt">></div></span>

###### Good:

Declarative custom elements and attributes are clearest.

    <span class="nt"><my-directive></my-directive></span>
    <span class="nt"><div</span> <span class="na">my-directive</span><span class="nt">></div></span>

You can restrict usage using the `restrict` property inside each directive's
Object. Use`E` for `element`, `A` for `attribute`, `M` for `comment` (avoid)
and`C` for `className` (avoid this too as it's even more confusing, but plays
better with IE). You can have multiple restrictions, such as`restrict: 'EA'`.

### Resolve promises in router, defer controllers

After creating services, we will likely inject them into a controller, call
them and bind any new data that comes in. This becomes problematic of keeping 
controllers tidy and resolving the right data.

Thankfully, using `angular-route.js` (or a third party such as `ui-router.js`)
we can use a`resolve` property to resolve the next view's promises before the
page is served to us. This means our controllers are instantiated when all data 
is available, which means zero function calls.

###### Bad:

    <span class="kd">function</span> <span class="nx">MainCtrl</span> <span class="p">(</span><span class="nx">SomeService</span><span class="p">)</span> <span class="p">{</span>
    
      <span class="kd">var</span> <span class="nx">self</span> <span class="o">=</span> <span class="k">this</span><span class="p">;</span>
    
      <span class="c1">// unresolved</span>
      <span class="nx">self</span><span class="p">.</span><span class="nx">something</span><span class="p">;</span>
    
      <span class="c1">// resolved asynchronously</span>
      <span class="nx">SomeService</span><span class="p">.</span><span class="nx">doSomething</span><span class="p">().</span><span class="nx">then</span><span class="p">(</span><span class="kd">function</span> <span class="p">(</span><span class="nx">response</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">self</span><span class="p">.</span><span class="nx">something</span> <span class="o">=</span> <span class="nx">response</span><span class="p">;</span>
      <span class="p">});</span>
    
    <span class="p">}</span>
    <span class="nx">angular</span>
      <span class="p">.</span><span class="nx">module</span><span class="p">(</span><span class="s1">'app'</span><span class="p">)</span>
      <span class="p">.</span><span class="nx">controller</span><span class="p">(</span><span class="s1">'MainCtrl'</span><span class="p">,</span> <span class="nx">MainCtrl</span><span class="p">);</span>

###### Good:

    <span class="kd">function</span> <span class="nx">config</span> <span class="p">(</span><span class="nx">$routeProvider</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">$routeProvider</span>
      <span class="p">.</span><span class="nx">when</span><span class="p">(</span><span class="s1">'/'</span><span class="p">,</span> <span class="p">{</span>
        <span class="nx">templateUrl</span><span class="o">:</span> <span class="s1">'views/main.html'</span><span class="p">,</span>
        <span class="nx">resolve</span><span class="o">:</span> <span class="p">{</span>
          <span class="nx">doSomething</span><span class="o">:</span> <span class="kd">function</span> <span class="p">(</span><span class="nx">SomeService</span><span class="p">)</span> <span class="p">{</span>
            <span class="k">return</span> <span class="nx">SomeService</span><span class="p">.</span><span class="nx">doSomething</span><span class="p">();</span>
          <span class="p">}</span>
        <span class="p">}</span>
      <span class="p">});</span>
    <span class="p">}</span>
    <span class="nx">angular</span>
      <span class="p">.</span><span class="nx">module</span><span class="p">(</span><span class="s1">'app'</span><span class="p">)</span>
      <span class="p">.</span><span class="nx">config</span><span class="p">(</span><span class="nx">config</span><span class="p">);</span>

At this point, our service will internally bind the response of the promise to
another Object which we can reference in our "deferred-instantiated" controller:

###### Good:

    <span class="kd">function</span> <span class="nx">MainCtrl</span> <span class="p">(</span><span class="nx">SomeService</span><span class="p">)</span> <span class="p">{</span>
      <span class="c1">// resolved!</span>
      <span class="k">this</span><span class="p">.</span><span class="nx">something</span> <span class="o">=</span> <span class="nx">SomeService</span><span class="p">.</span><span class="nx">something</span><span class="p">;</span>
    <span class="p">}</span>
    <span class="nx">angular</span>
      <span class="p">.</span><span class="nx">module</span><span class="p">(</span><span class="s1">'app'</span><span class="p">)</span>
      <span class="p">.</span><span class="nx">controller</span><span class="p">(</span><span class="s1">'MainCtrl'</span><span class="p">,</span> <span class="nx">MainCtrl</span><span class="p">);</span>

We can go one better, however and create a `resolve` property on our own
Controllers to couple the resolves with the Controllers and avoid logic in the 
router.

###### Best:

    <span class="c1">// config with resolve pointing to relevant controller</span>
    <span class="kd">function</span> <span class="nx">config</span> <span class="p">(</span><span class="nx">$routeProvider</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">$routeProvider</span>
      <span class="p">.</span><span class="nx">when</span><span class="p">(</span><span class="s1">'/'</span><span class="p">,</span> <span class="p">{</span>
        <span class="nx">templateUrl</span><span class="o">:</span> <span class="s1">'views/main.html'</span><span class="p">,</span>
        <span class="nx">controller</span><span class="o">:</span> <span class="s1">'MainCtrl'</span><span class="p">,</span>
        <span class="nx">controllerAs</span><span class="o">:</span> <span class="s1">'main'</span><span class="p">,</span>
        <span class="nx">resolve</span><span class="o">:</span> <span class="nx">MainCtrl</span><span class="p">.</span><span class="nx">resolve</span>
      <span class="p">});</span>
    <span class="p">}</span>
    <span class="c1">// controller as usual</span>
    <span class="kd">function</span> <span class="nx">MainCtrl</span> <span class="p">(</span><span class="nx">SomeService</span><span class="p">)</span> <span class="p">{</span>
      <span class="c1">// resolved!</span>
      <span class="k">this</span><span class="p">.</span><span class="nx">something</span> <span class="o">=</span> <span class="nx">SomeService</span><span class="p">.</span><span class="nx">something</span><span class="p">;</span>
    <span class="p">}</span>
    <span class="c1">// create the resolved property</span>
    <span class="nx">MainCtrl</span><span class="p">.</span><span class="nx">resolve</span> <span class="o">=</span> <span class="p">{</span>
      <span class="nx">doSomething</span><span class="o">:</span> <span class="kd">function</span> <span class="p">(</span><span class="nx">SomeService</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">return</span> <span class="nx">SomeService</span><span class="p">.</span><span class="nx">doSomething</span><span class="p">();</span>
      <span class="p">}</span>
    <span class="p">};</span>
    
    <span class="nx">angular</span>
      <span class="p">.</span><span class="nx">module</span><span class="p">(</span><span class="s1">'app'</span><span class="p">)</span>
      <span class="p">.</span><span class="nx">controller</span><span class="p">(</span><span class="s1">'MainCtrl'</span><span class="p">,</span> <span class="nx">MainCtrl</span><span class="p">)</span>
      <span class="p">.</span><span class="nx">config</span><span class="p">(</span><span class="nx">config</span><span class="p">);</span>

##### Route changes and ajax spinners

While the routes are being resolved we want to show the user something to
indicate progress. Angular will fire the`$routeChangeStart` event as we
navigate away from the page, which we can show some form of loading and ajax 
spinner, which can then be removed on the`$routeChangeSuccess` event (
[see docs][6]).

### Avoid $scope.$watch

Using `$scope.$watch` should be avoided unless there are no others options. It'
s less performant than binding an expression to something like`ng-change`, a
list of supported events are in the Angular docs.

###### Bad:

    <span class="nt"><input</span> <span class="na">ng-model=</span><span class="s">"myModel"</span><span class="nt">></span>
    <span class="nt"><script></span>
    <span class="nx">$scope</span><span class="p">.</span><span class="nx">$watch</span><span class="p">(</span><span class="s1">'myModel'</span><span class="p">,</span> <span class="nx">callback</span><span class="p">);</span>
    <span class="nt"></script></span>

###### Good:

    <span class="nt"><input</span> <span class="na">ng-model=</span><span class="s">"myModel"</span> <span class="na">ng-change=</span><span class="s">"callback"</span><span class="nt">></span>
    <span class="c"><!--</span>
    <span class="c">  $scope.callback = function () {</span>
    <span class="c">    // go</span>
    <span class="c">  };</span>
    <span class="c">--></span>

### Project/file structure

One role, one file, rule. Separate all controllers, services/factories,
directives into individual files. Don't add all controllers in one file, you 
will end up with a huge file that is very difficult to navigate, keeps things 
encapsulated and bitesize.

###### Bad:

    |-- app.js
    |-- controllers.js
    |-- filters.js
    |-- services.js
    |-- directives.js

Keep naming conventions for files consistent, don't invent fancy names for
things, you'll just forget them.

###### Good:

    |-- app.js
    |-- controllers/
    |   |-- MainCtrl.js
    |   |-- AnotherCtrl.js
    |-- filters/
    |   |-- MainFilter.js
    |   |-- AnotherFilter.js
    |-- services/
    |   |-- MainService.js
    |   |-- AnotherService.js
    |-- directives/
    |   |-- MainDirective.js
    |   |-- AnotherDirective.js

Depending on the size of your code base, a "feature-driven" approach may be
better to split into functionality chunks.

###### Good:

    |-- app.js
    |-- dashboard/
    |   |-- DashboardService.js
    |   |-- DashboardCtrl.js
    |-- login/
    |   |-- LoginService.js
    |   |-- LoginCtrl.js
    |-- inbox/
    |   |-- InboxService.js
    |   |-- InboxCtrl.js

### Naming conventions and conflicts

Angular provides us many Objects such as `$scope` and `$rootScope` that are
prefixed with`$`. This incites they're public and can be used. We also get
shipped with things such as`$$listeners`, which are available on the Object but
are considered private methods.

Avoid using `$` or `$$` when creating your own services/directives/providers/
factories.

###### Bad:

Here we create `$$SomeService` as the definition, not the function name.

    <span class="kd">function</span> <span class="nx">SomeService</span> <span class="p">()</span> <span class="p">{</span>
    
    <span class="p">}</span>
    <span class="nx">angular</span>
      <span class="p">.</span><span class="nx">module</span><span class="p">(</span><span class="s1">'app'</span><span class="p">)</span>
      <span class="p">.</span><span class="nx">factory</span><span class="p">(</span><span class="s1">'$$SomeService'</span><span class="p">,</span> <span class="nx">SomeService</span><span class="p">);</span>

###### Good:

Here we create `SomeService` as the definition, *and* the function name for
consistency/stack traces.

    <span class="kd">function</span> <span class="nx">SomeService</span> <span class="p">()</span> <span class="p">{</span>
    
    <span class="p">}</span>
    <span class="nx">angular</span>
      <span class="p">.</span><span class="nx">module</span><span class="p">(</span><span class="s1">'app'</span><span class="p">)</span>
      <span class="p">.</span><span class="nx">factory</span><span class="p">(</span><span class="s1">'SomeService'</span><span class="p">,</span> <span class="nx">SomeService</span><span class="p">);</span>

### Minification and annotation

##### Annotation order

It's considered good practice to dependency inject Angular's providers in
before our own custom ones.

###### Bad:

    <span class="c1">// randomly ordered dependencies</span>
    <span class="kd">function</span> <span class="nx">SomeCtrl</span> <span class="p">(</span><span class="nx">MyService</span><span class="p">,</span> <span class="nx">$scope</span><span class="p">,</span> <span class="nx">AnotherService</span><span class="p">,</span> <span class="nx">$rootScope</span><span class="p">)</span> <span class="p">{</span>
    
    <span class="p">}</span>

###### Good:

    <span class="c1">// ordered Angular -> custom</span>
    <span class="kd">function</span> <span class="nx">SomeCtrl</span> <span class="p">(</span><span class="nx">$scope</span><span class="p">,</span> <span class="nx">$rootScope</span><span class="p">,</span> <span class="nx">MyService</span><span class="p">,</span> <span class="nx">AnotherService</span><span class="p">)</span> <span class="p">{</span>
    
    <span class="p">}</span>

##### Minification methods, automate it

Use `ng-annotate` for automated dependency injection annotation, as `ng-min` is
[deprecated][7]. You can find `ng-annotate` [here][8].

With our function declarations outside of the module references, we need to use
the`@ngInject` comment to explicitly tell `ng-annotate` where to inject our
dependencies. This method uses`$inject` which is faster than the Array syntax
.

Manually specifiying the dependency injection arrays costs too much time.

###### Bad:

    <span class="kd">function</span> <span class="nx">SomeService</span> <span class="p">(</span><span class="nx">$scope</span><span class="p">)</span> <span class="p">{</span>
    
    <span class="p">}</span>
    <span class="c1">// manually declaring is time wasting</span>
    <span class="nx">SomeService</span><span class="p">.</span><span class="nx">$inject</span> <span class="o">=</span> <span class="p">[</span><span class="s1">'$scope'</span><span class="p">];</span>
    <span class="nx">angular</span>
      <span class="p">.</span><span class="nx">module</span><span class="p">(</span><span class="s1">'app'</span><span class="p">)</span>
      <span class="p">.</span><span class="nx">factory</span><span class="p">(</span><span class="s1">'SomeService'</span><span class="p">,</span> <span class="nx">SomeService</span><span class="p">);</span>

###### Good:

Using the `ng-annotate` keyword `@ngInject` to instruct things that need
annotating:

    <span class="cm">/**</span>
    <span class="cm"> * @ngInject</span>
    <span class="cm"> */</span>
    <span class="kd">function</span> <span class="nx">SomeService</span> <span class="p">(</span><span class="nx">$scope</span><span class="p">)</span> <span class="p">{</span>
    
    <span class="p">}</span>
    <span class="nx">angular</span>
      <span class="p">.</span><span class="nx">module</span><span class="p">(</span><span class="s1">'app'</span><span class="p">)</span>
      <span class="p">.</span><span class="nx">factory</span><span class="p">(</span><span class="s1">'SomeService'</span><span class="p">,</span> <span class="nx">SomeService</span><span class="p">);</span>

Will produce:

    <span class="cm">/**</span>
    <span class="cm"> * @ngInject</span>
    <span class="cm"> */</span>
    <span class="kd">function</span> <span class="nx">SomeService</span> <span class="p">(</span><span class="nx">$scope</span><span class="p">)</span> <span class="p">{</span>
    
    <span class="p">}</span>
    <span class="c1">// automated</span>
    <span class="nx">SomeService</span><span class="p">.</span><span class="nx">$inject</span> <span class="o">=</span> <span class="p">[</span><span class="s1">'$scope'</span><span class="p">];</span>
    <span class="nx">angular</span>
      <span class="p">.</span><span class="nx">module</span><span class="p">(</span><span class="s1">'app'</span><span class="p">)</span>
      <span class="p">.</span><span class="nx">factory</span><span class="p">(</span><span class="s1">'SomeService'</span><span class="p">,</span> <span class="nx">SomeService</span><span class="p">);</span>

 [1]: http://github.com/toddmotto/angularjs-styleguide

 [2]: http://google-styleguide.googlecode.com/svn/trunk/angularjs-google-style.html
 [3]: http://blog.angularjs.org/2014/02/an-angularjs-style-guide-and-best.html
 [4]: http://speakerdeck.com/toddmotto
 [5]: http://docs.angularjs.org/guide/module
 [6]: https://docs.angularjs.org/api/ngRoute/service/%24route
 [7]: https://github.com/btford/ngmin
 [8]: https://github.com/olov/ng-annotate