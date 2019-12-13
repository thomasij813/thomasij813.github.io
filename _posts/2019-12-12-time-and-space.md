---
layout: post
title: Algorithm Scripting -- Considering Time Complexity
date: 2019-12-13
---

One question that software engineers grapple with when writing an algorithm is How long does it take to complete? The answer to this question is what's referred to as a piece of code's **time complexity**, and determining it is more than just running the code a few times and finding the averag execution time. Instead, time complexity means figuring out how many operations are done by a piece of software **relative** to the size of a given input.

That phrase "relative to the size of a given input" is important. Consider a function that takes an array as input. Does the number of operations that function perform stay the same, no matter how big the array is? If so, congratulations, it's time complexity is **constant time**, which is notated at **O(1)**.

Working with arrays, however, usually requireing iterating over some or all of the contained elements, so it's likely that the number of operations will increase with the size of the array. If the rate of growth for this increase stays the same (ie, you have one for loop, or multiple that are back to back), that's referred to as **linear time** or **O(n)**. If the rate of growth exceeds that (ie, you have nested for loops), then you're venturing into the realm of **quadratic time (O(n&sup2;)** or, God help you, **exponential time (O(2^n))**.

In most circumstances, you want to keep your functions to constant time or linear time. Code that has a time complexity of anything greater will likely not perform adequately in real-world situations. Your function consisting of several nested for loops and sorting operations might work just fine when the input array is only five elements long. But it probably won't cut it in a non-contrived scenario when the array length is likely to be much higher.

Constraining algorithms to linear time is rarely easy. When you're new to writing software, for loops can feel like the bread and butter of programming, and limiting yourself to one at a time can present a real challenge. One way to get around this limitation is to learn techniques that rely on operations that execute in constant time.

For example, consider this common coding challenge, a favorite, it seems, among technical recruiters. Given an array of integers and a targeted sum, write a function that returns the indices of the two numbers that when added together equal the specific target.

A brute-force approach might look like the following:

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

The natural intuition is to use a nested for loop, comparing each number as its encountered for the first time with all the other numbers in the array. But this requires a nested for loop, giving it a time complexity of **O(n&sup2;)**. Surely, we can do better.

The reason that we need that second for loop is because we are relying on the input array as storage mechanism for finding the compliment value. As we do our first loop we can figure out what that compliment value should be (it's simply the target sum minus whatever the currently encounted number is), but so long as we're using an array to store our numbers we will be forced to loop through it again to see if that compliment value exists.

Is there an alternative data structure that we can use instead, one that doesn't rely on iteration to do lookups? Yes--objects! Finding a property in an object is done in constant time, so maybe we use that to our advantage.

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

This time we are doing two non-nested for loops. The first loop is used to establish a `compliments` object, which stores in key value pairs each number in `nums` and its index. Then we loop through `nums` again, this time checking to see if its compliment exists in `compliments`. Because finding a property in an object is in constant time, the lookup operation does not entail a second nested for loop meaning this function's time complexity is linear.

Why is this version of the code linear? After all, there's two for loops, just like in the earlier version. The reason is because the second loop is not nested inside the first one. In this case we are iterating over the input array twice, whereas in the first example we were iterating over it with each iteration. That's the difference between **O(2n)** and **O(n&sup2;)**. And just because we've done that second loop resulting in the "2" in **O(2n)**, most engineers would observe that that that number is a constant value that won't change as the input array grows and that it can be removed from the notation. Now we're back to **O(n)**.

But for code clarity, two loops is unnecessary. We can combine both operations--building `compliments` and/or using it--in one fell swoop.

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

In this version we add numbers to `compliments` as they are encountered. When a compliment isn't found, we add the number to the object in the offchance that it exists as the compliment for a number that's located further along in the array.

To summarize, our gut reaction was to over-rely upon the **nums** array, using it to iterate through the numbers and lookup the location of their compliments. But because looking up the location of values in an array is itself done in linear time, that approach would have resulted in a time complexity of **O(n&sup2;)**. By using an object to store and lookup the location of the complimenting values, however, we were able to cut out that second loop, resulting in the desired time complexity of O(n).
