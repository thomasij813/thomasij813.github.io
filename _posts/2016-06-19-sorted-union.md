---
layout: post
title: Algorithm Scripting -- Sorted Union
---

Now that I've completed the first two tracks in Free Code Camp's curriculum, lately I've been returning to some of my prior work to refactor the code I had submitted. So far this has meant reengineering some of my earliest projects to take advantage of React (see the weather application and the pomodoro timer on my [projects page](/projects)).

But I would also like to return to some of the algorithms that make up Free Code Camp's earliest challenges. I remember solving a lot of these challenges by resorting to heavily nested for loops, an operation flow that's as computationally taxing as it is difficult to read.

I want to devote at least few posts revising some of these algorithms and explaining my solutions. Taken together, I hope these posts will be a good demonstration of my particular approach to problem solving and may help new developers who are struggling in this area.

### The challenge: sorted union

For this first post, I want to focus on [Sorted Union](https://www.freecodecamp.com/challenges/sorted-union), an algorithm challenge that was recently posted about in the [Free Code Camp forum](http://forum.freecodecamp.com/). Here's a description of the challenge from the Free Code Camp website:

>Write a function that takes two or more arrays and returns a new array of unique values in the order of the original provided arrays.
>
>In other words, all values present from all arrays should be included in their original order, but with no duplicates in the final array.
>
>The unique numbers should be sorted by their original order, but the final array should not be sorted in numerical order.

As usual with Free Code Camp's algorithm challenges, the challenge description is a little hard to understand. Fortunately, the test assertions provide a good indicator about what is being asked (`uniteUnique` is the name of the function we're being asked to create):

{% highlight javascript %}
// I'm using the expect testing library to frame these assertions.

expect(
  uniteUnique([1, 3, 2], [5, 2, 1, 4], [2, 1])
).toEqual([1, 3, 2, 5, 4])

expect(
  uniteUnique([1, 3, 2], [1, [5]], [2, [4]])
).toEqual([1, 3, 2, [5], [4]])
{% endhighlight %}

Basically, the idea here is that `uniteUnique` will take some arrays as arguments and return a single array filled with unique elements found in each of the passed in arrays. Moreover, the items in the array that gets returned must be in the order of their original placement. There's one last thing worth mentioning thing that's not said in the description but is made explicit from the test assertions: `uniteUnique` is a shallow operation, applying to only the arguments and not to any arrays that may be found within them.

How do we solve this? First `uniteUnique` needs to create an empty array to which it is going to push each unique element it encounters among the arguments (this array will be returned at the end of the function). Determining the uniqueness of elements is going to require creating a nested loop: the outer loop will iterate through the arrays that get passed in as arguments, while the inner loop will iterate over each individual array to examine the elements that need to get pushed to the array that will be returned. Structured in a conventional manner, there is no need to worry about sorting because the elements will get pushed to the empty array in the order in which they are encountered.

On to implementation. We might initially set up the `uniteUnique` function like so, initially declaring an empty array (`out`):

{% highlight javascript %}
function uniteUnique() {
  var out = [];
  // Our soon to be written loops
}
{% endhighlight %}

Did you notice how I didn't write in any parameters? According to the specification `uniteUnique`, can take any number of arguments, a requirement that complicates the traditional approach of handling arguments by way of parameter names. So how do we access the arguments that get passed to `uniteUnique` if we don't know what to call them and we don't know how many there are?

Functions in javascript have access to something called `arguments`, an array-like object that refers to the arguments handed to the function. This is convenient, you might think, because the first of our nested loops involves looping over the arguments! Isn't it nice that the `arguments` is an iterable object?

Not so fast. Because we want to lean towards semantic clarity, we're going to want to make use of something like `forEach` rather than a traditional for loop. Unfortunately for us, `arguments` is array-like, meaning it's not a true array and is therefore not on the prototype chain that would grant access to `forEach`. We need to figure out a way of turning `arguments` from an array-like object into an actual array.

Do you know how to copy an array? Consider the following snippet in which we attempt to clone array `a`:

{% highlight javascript %}
var a = [1,2,3];
var b = a;

b.push(4);

console.log(b); // [1,2,3,4]
console.log(a); // [1,2,3,4]
{% endhighlight %}

What gives? We pushed `4` onto `b` not `a`! The problem is that `b = a` does not actually do what we wanted it to do. What we wanted was for `b` to get assigned to completely new array that was filled with the same values stored in `a`. Instead, we simply pointed `b` to `a`, meaning that they both now refer to the same object. Hence, using `push` to add a value to `b` will mutate the array that `a` also happens to be pointing at.

The best way to rewrite this code so that it actually achieves the intended outcome is to make use of `Array.slice`, a non-mutating function that returns a specified part of an array. The part of the array that gets returned when you call `slice` without passing any arguments is the entire array, creating what is essentially a copy of the array slice is called from:

{% highlight javascript %}
var a = [1,2,3];
var b = a.slice();

b.push(4);

console.log(b); // [1,2,3,4]
console.log(a); // [1,2,3]
{% endhighlight %}

Okay, so we can use `slice` to make a clone of `arguments`. But if `arguments` isn't already an array, how are we going to call `slice` on it? The reason we couldn't call `forEach` on `arguments` was because `forEach` is on the prototype chain of Array. Isn't `slice` also on the same prototype chain?

All this is entirely correct, and indeed using slice directly from `arguments` would fail. Fortunately, however, we can use `call` to shift the `this` context of `slice` so that it will work properly. This means writing some intimidating code that invokes `slice` directly from `Array.prototype` and then chains it to `call`, but I assure you this is well-traversed territory for many javascript developers (see [this post](https://davidwalsh.name/arguments-array) for additional details).

_*** It should also be noted that using `slice` on `arguments` obstructs optimizations in some engines. You can read about an alternative approach [here](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/arguments) ***_

We now have a way of transforming `arguments` into an actual array, so let's write it into `uniqueUpdate`:

{% highlight javascript %}
function uniteUnique() {
  var out = [];
  var args = Array.prototype.slice.call(arguments);
  // Our soon to be written loops
}
{% endhighlight %}

The next bit is easy. Here's how we would use `forEach` to loop over `args` (our `arguments` clone):

{% highlight javascript %}
function uniteUnique() {
  var out = [];

  var args = Array.prototype.slice.call(arguments);

  args.forEach(function(arg) {
    // Our soon to be written inner loop.
  });
}
{% endhighlight %}

Next, we need to add another `forEach` loop that iterates over the individual element of each argument, which would look something like this:

{% highlight javascript %}
function uniteUnique() {
  var out = [];

  var args = Array.prototype.slice.call(arguments);

  args.forEach(function(arg) {
    arg.forEach(function(element) {
      // Determines whether or not the value should be added
      // to the out array  
    });
  });
}
{% endhighlight %}

Now that we have the loops settled, we now need to create some type of test that will determine whether or not an individual element needs to be added to the array that will ultimately get returned (assigned to the variable `out` in our function). According to the instructions, `uniteUnique` must return only unique elements. This means, for example, that if among our arguments there exists multiple instances of the value `5`, only the first of those instances should be added to `out`.

Since we are traversing the arguments and their elements sequentially, we can test each value for uniqueness simply by checking to see if it is already present in the `out` array. This is doubly useful because it also ensures that unique elements are added as they are encountered, meaning that the elements in `out` will be in the correct order.

We can check if an element is in an array by using the `indexOf` property. `indexOf` will return the index of a given value in the array it is called on. In instances when the value is not found, it will return `-1`. Using `indexOf` we can now write the test case that will add elements to out like so:

{% highlight javascript %}
function uniteUnique() {
  var out = [];

  var args = Array.prototype.slice.call(arguments);

  args.forEach(function(arg) {
    arg.forEach(function(element) {
      if (out.indexOf(element) < 0) {
        out.push(element);
      }
    });
  });
}
{% endhighlight %}

 We have now completed the logic that will add unique values to the `out` array. The only thing left to do is return it:

 {% highlight javascript %}
 function uniteUnique() {
   var out = [];

   var args = Array.prototype.slice.call(arguments);

   args.forEach(function(arg) {
     arg.forEach(function(element) {
       if (out.indexOf(element) < 0) {
         out.push(element);
       }
     });
   });

   return out;
 }
 {% endhighlight %}

### Refactoring with `reduce`

As it currently exists `uniteUnique` is perfectly functional. Notice, however, that our logic involves filling an initially empty array that potentially mutates each time our loops advance. You should know that any time you write a `forEach` loop that adds items to an array you are overlooking an opportunity to use `reduce` instead.

`reduce`, like `forEach`, loops over an array performing a callback with each iteration. Unlike `forEach`, however, the callback given to `reduce` requires that something is returned, a value that will be provided to the next iteration of the loop. Basically `reduce` gives us a way of gradually building something up over the course of looping through an array's elements.

Consider the classic demonstration of using `reduce` to get the sum of an array of numbers. With `forEach`, we would write the following code that increments a counter by each number in the array:

{% highlight javascript %}
var array = [1,2,3,4,5];

var counter = 0;

array.forEach(function(number) {
  counter += number;
});

return counter; // 15
{% endhighlight %}

We can rewrite this code using `reduce` like so:

{% highlight javascript %}
var array = [1,2,3,4,5];

var counter = array.reduce(function(accumulator, currentNumber) {
  return accumulator + currentNumber;
}, 0);

return counter; // 15
{% endhighlight %}

The callback function handed to `reduce` receives two arguments, what I've called `accumulator` and `currentNumber`. `accumulator` refers to the value that was returned by the callback on the previous iteration (notice how the callback returns a value, unlike `forEach`).`currentNumber` refers to the item in the array that is currently being examined. The `0` that is passed in as the second argument to `reduce` refers to the value `accumulator` should assume the first time the callback is run.

Taking this step-by-step, the first time `reduce` runs, `accumulator` is set to `0` and `currentNumber` is set to `1`. The callback returns the sum of these two numbers, which becomes the `accumulator` for the second iteration when `2` will be assigned to `currentNumber`. In that case, the callback will return `3`, which becomes the `accumulator` for the next cycle, and so on. In this way, all the numbers are added together resulting in `15` (see the [MDN article](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce) on `reduce` to learn more).

`reduce` is a handy function. Its syntax is a little tricky to get a handle of, but once you have it all figured out you'll find it to be useful way of applying a compounding function on all the elements of an array, ultimately 'reducing' it to some final result.

In our case, we can substitute initializing `out` to an empty array by assigning it directly to a reducing function that builds an array of unique elements found in the arguments:

{% highlight javascript %}
function uniteUnique() {
  var args = Array.prototype.slice.call(arguments);

  var out = args.reduce(function(accumulator, currentArg) {
    currentArg.forEach(function (element) {
      if (accumulator.indexOf(element) < 0) {
        accumulator.push(element);
      }
      return accumulator;
    });
  }, []);

  return out;
}
{% endhighlight %}

In fact, since the callback we are handing to `reduce` returns an array, we can dispense with assigning the result to `out` and just return it directly:

{% highlight javascript %}
function uniteUnique() {
  var args = Array.prototype.slice.call(arguments);

  return args.reduce(function(accumulator, currentArg) {
    currentArg.forEach(function (element) {
      if (accumulator.indexOf(element) < 0) {
        accumulator.push(element);
      }
      return accumulator;
    });
  }, []);
}
}
{% endhighlight %}

### Wrapping up

Before closing, there are three additional points I want to make regarding `uniteUnique`. When it comes to completing algorithm challenges like this, you'll find that there are many different ways to achieve the desired outcome. Sometimes you'll encounter overly complicated solutions with lots of nested for loops and counters. Other times you'll read one-liners that make use of method chaining and regex tests. As a rule of thumb, I recommend striving for a solution that balances clarity with performance. You certainly want to avoid writing functions replete with taxing operations, but I also think it's important to avoid single-line solutions that are especially cryptic.

On that note, here are three additional ways of rewriting the above solution.

First, you might have wondered we used `call` to access the `slice` method for `arguments` when we could have done the same thing for `reduce` (or `forEach`). In that case, there would be no need to create an `args` variable at the top of our function.

Indeed, using `call` directly with `reduce` (or `forEach`) and `arguments` will work just fine. It would look something like this:

{% highlight javascript %}
function uniteUnique() {
  return Array.prototype.reduce.call(arguments,
    function(accumulator, currentArg) {
      currentArg.forEach(function(element) {
        if (accumulator.indexOf(element) < 0) {
          accumulator.push(element);
        }
      });
      return accumulator;
  }, []);
}
}
{% endhighlight %}

This is a little too messy for my tastes. I begin to worry that my code is becoming unreadable once its shape starts resembling a sideways pyramid. This solution manages to bypass crafting an actual array from the array-like `arguments` object, but I don't think trading in that small operation for this hard-to-read cascade of indented code is worth it.

`reduce` is definitely the way to go when it comes to managing our outer loop, but `forEach` is not our only option when it comes to the inner one. The [solution I submitted to Free Code Camp](https://www.freecodecamp.com/challenges/sorted-union#?solution=function%20uniteUnique(arr)%20%7B%0A%20%20var%20args%20%3D%20Array.prototype.slice.call(arguments)%3B%0A%20%20return%20args.reduce(function(accum%2C%20curr)%20%7B%0A%20%20%20%20return%20accum.concat(curr.filter(function(item)%20%7B%0A%20%20%20%20%20%20return%20accum.indexOf(item)%20%3C%200%3B%20%20%0A%20%20%20%20%7D))%3B%0A%20%20%7D%2C%20%5B%5D)%3B%0A%7D%0A%0AuniteUnique(%5B1%2C%203%2C%202%5D%2C%20%5B5%2C%202%2C%201%2C%204%5D%2C%20%5B2%2C%201%5D)%3B%0A) involves using the `filter` method on the `currentArg` array to get an array of unique elements, the contents of which are then added the `accumulator` array by way of `concat`. It looks like this (we don't actually need the empty array that makes up the second argument to`reduce`, by the way; it's perfectly fine that the initial array in `args` be our first `accumulator`, which is what will happen if we leave it absent like below):

{% highlight javascript %}
function uniteUnique() {
  var args = Array.prototype.slice.call(arguments);

  return args.reduce(function(accumulator, currentArg) {
    return accumulator.concat(currentArg.filter(function(item) {
      return accumulator.indexOf(item) < 0;  
    }));
  });
}
{% endhighlight %}

Those nested return statements (and the `concat` and `filter` combo) are a little strange looking, but fortunately ES6 syntax helps make things look a little better:

{% highlight javascript %}
function uniteUnique() {
  const args = Array.prototype.slice.call(arguments);

  return args.reduce((accumulator, currentArg) =>
    [...accumulator,
    ...currentArg.filter(item => accumulator.indexOf(item) < 0)]);
}
{% endhighlight %}
