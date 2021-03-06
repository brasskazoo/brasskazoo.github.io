---
layout: post
title: Converting an existing javascript file to a require.js AMD module
date: '2013-02-06T13:47:00.001+11:00'
categories: web-development javascript

author: brasskazoo
tags:
- AMD
- require.js
- Javascript
- Code Quality
modified_time: '2013-02-06T13:47:21.630+11:00'
blogger_id: tag:blogger.com,1999:blog-4222683507658340156.post-51747706073316474
blogger_orig_url: http://blog.brasskazoo.com/2013/02/converting-existing-javascript-file-to.html
---

[Require.js](http://requirejs.org/) is a Javascript module loader, helping to 
reduce complexity of applications by managing dependencies and allowing easier 
separation and organisation of code into modules.

## Step 0: Use jslint
First of all, you should be using [jslint](http://www.jslint.com/) to ensure your
javascript code is clean and well formatted (I know its available as a plugin 
for IntelliJ, not sure about eclipse). 

## Step 1: Organise

To make this process as straightforward as possible, get
your existing code into an organised fashion (i.e. a collection of vars and 
functions). Using jslint has probably got it into a good state already! 

{% highlight javascript %}
var foo = "..."; 

function func1() { 
 ... 
} 

function func2() { 
 ... 
} 
{% endhighlight %}

## Step 2: The define call

It is fairly trivial to then wrap a '`define`' call
around the code (see require.js 
[documentation](http://requirejs.org/docs/whyamd.html#amd)), and turn it into 
an Asynchronous Module Definition (AMD): 

{% highlight javascript %}
define(function () { 
    "use strict"; 

    return { 
     // Your code goes here.. 
    }; 
}); 
{% endhighlight %}

But if you have dependencies existing in the file (e.g. jquery), you will need 
to resolve them as module dependencies and refactor their usages before the 
module can function.

## Step 3: Dependencies
Add the dependencies by name as an array in the `define()` call, and corresponding parameters in the module
function.

Existing usages of those dependencies will then need to be
updated to call functions via the parameter objects (Note: to use dependencies 
in this way, they must also be AMD-compatible modules. Otherwise, you may need 
to configure a [shim](http://requirejs.org/docs/api.html#config-shim) for the 
dependency). 

{% highlight javascript %}
define(["jquery", "another_dependancy"], function ($, lib) { 
    "use strict"; 

    return { 
     var foo = "..."; 

 function func1() { 
            var content = $('#body-content'); 
         ... 
 } 

 function func2() { 
     lib.someFunction(); 
     ... 
 } 
    }; 
}); 
{% endhighlight %}

Now your require.js module is ready to be used! But there's more you should do
before that..

## Step 4: Encapsulation

One of the advantages of this module pattern is that it can be structred so that variables and functions can
be internalised and effectively defined in a 'private' scope. The module's 
return object can define exactly what should be publicly accessible. 
In the previous code snippet, the variable `foo` and both functions were all 
returned by the module. This means that they would all be publicly accessible 
by any client code. Instead, we want to ensure that `foo` and the functions 
are initially defined as private, and then expose the functions explicitly in 
the return object.

{% highlight javascript %}
define(["jquery", "another_dependancy"], function ($, lib) { 
    "use strict"; 

 // private variable 
    var foo; 

 function func1() { 
  var content = $('#body-content'); 
  ... 
 } 

 function func2() { 
  if (foo === null) { 
   foo = lib.someFunction(); 
  } 
  return foo; 
 } 

 return { 
  doTheThing: func1, 
  getTheFoo: func2 
    }; 
}); 
{% endhighlight %}

Done!