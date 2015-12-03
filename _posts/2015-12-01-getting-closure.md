---
layout: post
title: Getting Closure
---

I've spent the last couple years learning JavaScript, but I'm just now starting to get the hang of closure. It's a tricky but important concept, one that trips up a lot of programmers new to JavaScript and sometimes even eludes experienced developers.

In this post, I'm going to try and explain the concept to the best of my abilities. This will by no means be a comprehensive explanation, and given my limited expertise I encourage readers to check out more authoritative sources before carrying whatever knowledge gained here into practice.

In particular, I highly recommend reading Kyle Simpson's [chapter on closure](https://github.com/getify/You-Dont-Know-JS/blob/master/scope%20&%20closures/ch5.md) from his excellent [You Don't Know JS](https://github.com/getify/You-Dont-Know-JS) series and the relevant passages from Douglas Crockford's classic, [JavaScript: The Good Parts](http://www.amazon.com/JavaScript-Good-Parts-Douglas-Crockford/dp/0596517742).

## Let's talk about scope

Before getting to closure, it's important to understand what scope is and how it works.

**Put simply, in JavaScript scope refers to the rules that the engine abides by when it performs variable lookups.** As you probably know, successfully looking up variables depends a lot on the namespaces they were assigned to and the context from which they are being looked up.

Most developers know that trying to reference variables that were defined within a function from beyond the context of that function will return an error. This is because the variables were scoped to the namespace of the function they were defined in, and trying to call these variables from outside that namespace is forbidden by JavaScript's engine.

Take a look at the following code snippet:

{% highlight javascript %}
var a = "I'm scoped to global!";

function foo() {
  var b = "I'm scoped to foo!";
  console.log(a + " " + b);
}

foo(); // I'm scoped to global! I'm scoped to foo!

console.log(b); // error
{% endhighlight %}

When we invoke `foo`, the engine steps inside the function to execute its code, so to speak. This means that while its executing `foo`, the engine has access to all the variables mapped to that function's namespace. For these reasons, `foo` is able to log successfully both `a` and `b`.

We run into problems when we attempt to log `b` from outside of `foo`. While it may seem strange that that the engine raises an error when we try to call `b` just six lines after defining it, according to the rules of scope, `b` is mapped to `foo`'s namespace. Trying to access `b` from outside the context of `foo`'s namespace is just not possible.

That being understood, why then was `foo` able to call `a`? It seem's like a contradiction to say that on one hand we are unable to look up `b` from outside `foo` because `foo` is where `b` was created, but on the other hand to say that `foo` can totally look up `a` even though `a` wasn't defined there. Both lookups refer to variables that were defined somewhere other than the execution context of the call, so why does JavaScript forbid one lookup but not the other?

The answer has less to do with what kind of lookups are forbidden and allowed by the JavaScript engine, and more with how that actual lookup process is performed. **At this point, I should come clean about something: when I say scope is all about rules, I'm being a little misleading. In truth, it's more appropriate to think of scope in terms of a constrained route the engine takes when it's tasked with looking up variables.**

When you reference a variable in your code, the JavaScript engine takes a strongly specified path to look it up. First, it looks to see if the variable was added to the current scope, that is, it checks to see if that variable exists within the namespace of the current execution context (ie, function). If it doesn't find the variable in the current namespace, it take one step backward and checks to see if the namespace of the parent scope contains the requested variable. If again the engine fails to find the variable there, it checks the namespace of the parent's parent scope, and so on all the way to the `global` scope.

The `global` scope happens to be the final namespace on the engine's lookup path. It provides access to variables that were defined outside of any function or variables that were declared without the preceding keyword `var`. There is no parent scope to `global`, so if the engine reaches it all the way here without finding the variable it needs, it raises a reference error.

All functions and their namespaces, no matter how deeply nested, have access to the variables scoped to `global`. As a result, it can be very tempting to define all of the variables you'll ever need in the namespace of the `global` scope. **In truth, however, assigning variables to `global` is rarely a good idea, and it should be avoided.**

Now that we know a little more about scope, the answer to our question about why `foo` was able to reference `a` should be clear. Here's the code snippet from earlier:

{% highlight javascript %}
var a = "I'm scoped to global!";

function foo() {
  var b = "I'm scoped to foo!";
  console.log(a + " " + b);
}

foo(); // I'm scoped to global! I'm scoped to foo!

console.log(b); // error
{% endhighlight %}

`a` was defined within the `global` scope. When we invoke `foo`, the engine is tasked with finding `a`. When it fails to find it within `foo`'s namespace, the engine takes a step backward and finds it in `foo`'s parent, the `global` scope.

Knowing how variables are looked up by the engine is important, and it allows us to better appreciate what the engine is doing to find variables. Before moving on to closure, however, I think it would serve our purpose to examine one more code snippet:

{% highlight javascript %}
function foo() {
  var a = "I'm scoped to foo!";

  function bar() {
    var b = "I'm scoped to bar!";
    console.log(a + " " + b);
  }

  bar();
}

function baz() {
  var c = "I'm scoped to baz!";
  console.log(a + " " + b + " " + c);
}

foo(); // I'm scoped to foo! I'm scoped to bar!
baz(); // error
{% endhighlight %}

Before you continue reading, try and figure out why invoking `foo` succeeded and invoking `baz` failed. What path does the engine take to look up the variables these two functions reference?

As I mentioned earlier, it's only somewhat correct to say that the reason invoking `baz` raises an error when it attempts to log `a` and `b` is because the rules of scope forbid the engine from breaking the current execution context (which happens to be the function `baz`) and accessing the namespaces of `foo` and `bar`, the functions where those variables are defined. It's better to say that the reason has to do with the route the engine takes when it tries to find those variables, a route that `foo` and `bar` just aren't on.  

Following the variable lookup path helps reveal why `baz` came up short when it tried calling `a` and `b`. Tracing the lookup path taken by the engine when we invoke `foo` also explains why that function was able to successfully look up the same variables.

The final line of `foo` invokes `bar`, a function nested inside `foo` that logs the two variables. This means that invoking `foo` pushes our execution context to `bar`. Finding `b` from inside `bar` is easy for the JavaScript engine since that's the function where `b` was declared. The process the engine takes to find `a`, however, is complicated by the fact that there is no variable named `a` in `bar`'s namespace. Upon finding this out, the engine consults the namespace of `bar`'s parent scope, `foo`, where there is a variable named `a`. Hence, both variables are successfully able to be looked up by the engine and logged to the console.

We started this discussion by thinking about scope in terms of rules of access the engine follows when it performs variable lookups. We ended it, however, by demonstrating a subtle but important reformulation: **scope is the constraining mechanism that binds the engine to a specified path of nested namespaces in order to complete variable lookups.** Starting with the scope of the current execution context, upon failing to find a variable the engine navigates backward consulting each parent scope until reaching the `global` scope, ultimately either finding the variable it needs and returning its value or not and raising a reference error.

### On to closure...

Let's take a look at another code snippet:

{% highlight javascript%}
function foo() {
  function bar() {
    console.log("I'm scoped to bar!");
  }

  var obj =  {
    bar: bar
  };

  return obj;
}

bar(); // error

var baz = foo();
baz.bar(); // I'm scoped to bar!
{% endhighlight %}

Here we've created a function that returns an object. The object contains only one property, a method named `bar` that happens to point to a function of the same name nested inside `foo`. Trying to call `bar()` from the execution context of the `global` scope results in an error.

So far, none of this is surprising. When we try to invoke `bar` from the `global` scope, we are essentially trying to call a function that cannot be accessed by the engine. `bar` is scoped to `foo`, so trying to call `bar` from the context of the `global` scope, which is the parent of `foo`, will fail. **Remember, when the engine can't find something in the namespace of the current execution context, it starts looking in the namespace of each parent scope.** The `global` scope doesn't have any parents, so when the engine fails to find `bar` in the `global` namespace, it's forced to return an error.

Notice, however, that invoking `baz.bar` executes correctly. Given our knowledge of scope, this should raise an important question: why is it that calling a method from the `global` scope that happens to point to `bar` succeeds and just calling `bar` directly fails? Aren't both of these invocations performing lookups on a function that is defined in a namespace that cannot be reached from `global`?

**The reason this works is because `baz.bar`, being a property on the object returned by `foo`, has closure over `foo`.** The upshot of this is that when `baz.bar` gets invoked, the execution context slides momentarily back inside `bar`, meaning that `baz.bar` can call any variables that `bar` had access to as a function nested inside `foo`.

This might give you the impression that `baz.bar` is breaking the "rules" of scope by being able to reference variables that aren't within the current execution context of `baz` (ie, the `global` scope). **In truth, calling `baz.bar` shifts the execution context, so to speak, so no "rules" are being broken here, and lookups get performed just as would be expected.**

If you think this has something to do with the fact that `baz.bar` is a method on the object returned by `foo`, think again. The following code operates according to the same logic of closure:

{% highlight javascript%}
function foo() {

  var a = "I've been scoped to foo!"

  function bar() {
    console.log(a);
  }

  return bar;
}

console.log(a);  // error
bar();           // error

var baz = foo(); // returns a pointer to the function object bar and assigns it to baz
baz();           // I've been scoped to foo!
{% endhighlight %}

`a` and `bar` are both scoped to `foo`'s namespace, so attempting to call them from the `global` scope results in an error. `baz`, on the other hand, is assigned to the result of invoking `foo`, meaning that `baz` points to `bar`. Just like in our earlier example when I said that `baz.bar` had closure over `foo`, here `bar` also has closure over `foo`. This means that invoking `baz` in turn invokes `bar`, and this, speaking a little crudely, **is what slides the execution context back inside `bar`, a subtle maneuver affording the engine a variable lookup path on which it can locate `a`.**

In both these examples, `bar` has closure over `foo`, the function it was created in. And since `foo` returned `bar`—as either a method on an object or directly itself—in a certain fashion `bar` can continue to live on and function even outside `foo`'s scope.

### Privacy

If you understand closure and it doesn't seem surprising to you, congratulations, you must have an excellent understanding of scope. Being able to intuitively determine how context shifts as code gets executed and how variables are looked up according to the scoping mechanism are the prerequisites to understanding and employing closure. **When these things become second nature, so too will the idea of closure.**

With that said, it's worthwhile to dwell on the powerful functionality that closure affords. **Because closure allows us to reference variables scoped to namespaces we otherwise could not access, it's very easy to write code whose execution relies on private resources.**

Consider the following code snippet:

{% highlight javascript %}
function createProfile(name, yearOfBirth) {

  function updateUsername(name, yearOfBirth) {
    if (name.length > 6) {
      name = name.slice(0, 6);
    }
    return name + yearOfBirth;
  }

  function changeName(newName) {
    this.name = newName;
    this.username = updateUsername(this.name, this.yearOfBirth);
  }

  function changeYearOfBirth(newYearOfBirth) {
    this.yearOfBirth = newYearOfBirth;
    this.username = updateUsername(this.name, this.yearOfBirth);
  }

  var profile = {
    name: name,
    yearOfBirth: yearOfBirth,
    username: updateUsername(name, yearOfBirth),
    changeName: changeName,
    changeYearOfBirth: changeYearOfBirth
  };

  return profile;
}

var profile = createProfile("Franklin", 1988);

console.log(profile.name);        // Franklin;
console.log(profile.yearOfBirth); // 1988
console.log(profile.username);    // Frankl1988

profile.username();               // error

profile.changeName("Montgomery");
profile.changeYearOfBirth(1950);

console.log(profile.name);        // Montgomery
console.log(profile.yearOfBirth); // 1950
console.log(profile.username);    // Montgo1950
{% endhighlight %}

In this example, I have created a `createProfile` function that takes as arguments a name and a year of birth and returns a `profile` object. The `profile` object contains five properties, two of which are assigned to whatever name and year of birth that was originally passed to the `createProfile` function.

The property `username` is set to a string that is created as the result of passing `name` and `yearOfBirth` to the `updateUsername` function. **Notice that `username` is not set to the function `updateUsername`; instead it's set to the result of invoking `updateUsername`.** If you tried to invoke `username` as a property of `profile` you would receive an error because `profile.username` is not a function. It's just a string.

Unlike `username`, the properties `changeName` and `changeYearOfBirth` are methods. These properties point to the functions of the same name. Both functions reset the `name` and `yearOfBirth` properties of whatever object they are called from, respectively.

They also change the `username` property by invoking `updateUsername`, a function scoped to `createProfile`. `changeName` and `changeYearOfBirth` can invoke `updateUsername` even when the object they are called from is outside the scope of `createProfile` because these functions have closure over `createProfile`.

**Essentially this makes `updateUsername` a private function.** It cannot be called directly outside of the context of `createProfile`, but it can continue to operate when `changeName` and `changeYearOfBirth` get called.

In this example `updateUsername` isn't that interesting of a function. It seems kind of trivial that we would want to keep private a function that performs a pretty simple concatenation operation. But you can imagine scenarios where keeping functions private is anything but trivial.

What if instead of returning a small `profile` object, we had a function that returned a huge public API, with tons of methods and data? It's a safe bet that some of the methods that get returned in our public API rely on helper functions or variables that we don't want the public to be able to access directly. In order for the methods that were returned by the API to work outside the context of their creation, we need a way to allow them to slide back into their original scope and look up those private methods. This is precisely what closure affords.

*\*\*Note: for an extended discussion on the relation of closure to APIs and modules, see [this](https://github.com/getify/You-Dont-Know-JS/blob/master/scope%20&%20closures/ch5.md#modules) section in Kyle Simpson's [You Don't Know JS: Scope & Closure](https://github.com/getify/You-Dont-Know-JS/tree/master/scope%20%26%20closures)*

### Wrapping up

Hopefully by now you understand a little more about what closure is and why it's important to JavaScript. It's not the easiest concept to figure out, and I gather that fully appreciating it requires a somewhat sophisticated understanding of scope.

With that said, I hope you'll continue to research this topic by referencing some of the resources I mentioned earlier or others that you find online. As always seems to be the case with JavaScript, for every important programming topic to be learned, there are thousands great explanations floating around the web just waiting to be absorbed.
