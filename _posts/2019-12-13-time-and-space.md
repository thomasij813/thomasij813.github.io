---
layout: post
title: Considering Time Complexity with Two Sum
date: 2019-12-13
---

Before attending Hack Reactor, I knew very little about how to think or talk about a piece of code's performance. Of course, as a self-taught developer, I did have a couple general intuitions that I had picked up. I knew, for example, that more lines of code generally indicated more operations that had to be performed, and that that in turn meant longer execution time. Rely heavily on for loops or `forEach` calls, and that sequence of operations can get out of hand. Makes sense, right?

Generally speaking, that's true, but engineers value precision. And they don't want to talk about something as important code performance with murky language based on intuition. Instead they use **Big O Notation**, a structured approach to evaluating algorithms and classifying them according to their complexity relative to the size of their input. When we are counting the total number of operations an algorithm performs, we are measuring its time complexity, and when we are looking at how much total memory it uses, we are evaluating its space complexity.

For the rest of this post, I'm going to give an example of how to think about a piece of code's time complexity using Big O Notation. In the coming weeks, I'll follow up with an additional post that explores space complexity.

Remember when I said earlier that Big O Notation is about considering an algorithm's complexity relative to the size of the input? That phrase "relative to the size of the input" is important. Consider a function that takes an array as one of its parameters. Does the number of operations that that function performs stay the same, no matter how big the array is? If so, congratulations, it has a time complexity of constant time, notated as _O(1)_.

Working with arrays, however, usually requires iterating over some or all of the contained elements in some kind of loop, so it's likely that the total number of operations will increase as the size of the array grows. If the rate of growth for this increase stays the same (ie, you have one loop or multiple that are back to back), that's referred to as linear time or _O(n)_. If the rate of growth exceeds that (ie, you have several loops nested inside one another), then you're venturing into the realm of quadratic time (_O(n&sup2;)_) or, God help you, exponential time (_O(2^n)_).

In most circumstances, you want to keep your functions to constant time or linear time. Code that has a time complexity of anything greater than that will likely not perform adequately in real-world situations. Your function consisting of several nested for loops and sorting operations might work just fine when you're testing it with an input array of only several elements. But that same function probably won't cut it in a non-contrived scenario when the array length is likely to be much higher.

Unfortunately, constraining algorithms to linear time is rarely easy. When you're new to writing software, looping through arrays can feel like the bread and butter of programming, and limiting yourself to one loop at a time can present a real challenge. A complicating factor is that many built in methods themslves rely on some kind iteration. Do you really want to use `indexOf` or `slice` on an array that you're currently looping through? Both of those methods also entail looping through the array, so going down that path will result in quadratic time complexity. That's why earning to correctly intuit which built in methods and techniques operate in constant time is the first step toward writing minimally complex code.

For example, consider this common coding challenge, a favorite, it seems, among technical recruiters. Given an array of integers and a targeted sum, write a function that returns the indices of the two numbers that when added together equal the specified target.

At first glance, we might come up with the following solution:

```javascript
const twoSum = (nums, target) => {
  for (let i = 0; i < nums.length; i++) {
    let num1 = nums[i];

    for (let j = 0; j < nums.length; j++) {
      let num2 = nums[j];
      if (num1 + num2 === target) {
        return [i, j];
      }
    }
  }
};
```

The natural intuition is to use a nested for loop, comparing each number as it's encountered with all the other numbers in the array. This gives us a working solution, but that nested for loop gives it a time complexity of _O(n&sup2;)_. Surely, we can do better.

The reason that we need that second loop is because we are relying on the input array to find the compliment value. As we do our first loop we can figure out what that compliment value should be (it's simply the target sum minus whatever the currently encounted number is), but so long as we're using an array to look up values we will be forced to loop through it again to see if that compliment is present.

Is there an alternative data structure that we can use instead, one that doesn't rely on iteration to do lookups? Yes--objects! Finding a property in an object is done in constant time, so maybe we can use that feature to our advantage.

```javascript
const twoSum = (nums, target) => {
  const compliments = {};

  for (let i = 0; i < nums.length; i++) {
    let num = nums[i];
    compliments[num] = i;
  }

  for (let i = 0; i < nums.length; i++) {
    let num = nums[i];
    let compliment = target - num;
    if (compliments[num] >= 0) {
      return [i, compliments[num]];
    }
  }
};
```

This time we are doing two non-nested for loops. The first loop is used to establish a `compliments` object, which stores in key-value pairs each number in `nums` and its index. Then we loop through `nums` again, checking to see if its compliment exists in `compliments`. Because finding a property in an object is in constant time, the lookup operation does not entail a nested loop, giving this version a time complexity of _O(n)_.

But wait a second! There's two loops there. Surely the time complexity can't be linear when there's more than one loop like that, right?

While there are multiple loops in this version, the reason that its time complexity is linear is because the second loop is not nested inside the first one. In this case we are iterating over the input array two times back-to-back, whereas in the first example we were iterating over it with each and every iteration. That's the difference between _O(2n)_ and _O(n&sup2;)_.

How worried should we be about that second loop? From the standpoint of optimizing for time complexity, probably not too much. The second loop results in the "2" in _O(2n)_, but most engineers would observe that that that number is a constant value since it won't change as the input array grows. And that means we can remove it from the notation, putting us back at _O(n)_.

For code clarity, however, two loops is unnecessary. We can combine both operations--building `compliments` and making using of it--in one fell swoop.

```javascript
const twoSum = (nums, target) => {
  const compliments = {};

  for (let i = 0; i < nums.length; i++) {
    let num = nums[i];
    let compliment = target - num;

    if (compliments[num] >= 0) {
      return [i, compliments[num]];
    } else {
      compliments[num] = i;
    }
  }
};
```

This time we add numbers to `compliments` as they are encountered. When a compliment isn't found, we add the number to the object in the offchance that it will exist as the compliment for a number that's located further along in the array.

To summarize, our gut reaction was to over-rely upon the `nums` array, using it to iterate through the numbers and look up the location of their compliments. But because looking up the location of values in an array is itself done in linear time, that approach would have resulted in a time complexity of _O(n&sup2;)_. By using an object to store and look up the location of the complimenting values, however, we were able to cut out that second loop entirely, resulting in the desired time complexity of _O(n)_.
