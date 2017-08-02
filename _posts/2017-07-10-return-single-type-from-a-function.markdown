---
layout: post
title:  Always return a single type from a function
date:   2017-07-10 10:00:00
author: Isaac Jarquin
categories: software
excerpt:  Why returning differents types from a function is not a good Idea.
comments: true
resume: "In functional programming, functions are first class citizens, this means that you can treat a function pretty much like any other variable, so you can pass a function to another function as a parameter."
---

## Why returning differents types from a function is not a good Idea.

While these sorts of problems affect equally both functional and OOP programming, we are going to be talking primarily about functional programming, mainly because the style of coding used in this example is functional, but everything in this post can be extrapolated into the OOP world.

A couple of months ago I was working on a ticket with the purpose of showing an error message to a customer, in our front end application when one of his TV products had come from the backend system without any content. The ticket was a bit bigger than the following example but, in this post, I want to focus on this particular bit of code. So my original pull request included something like the following:

{% highlight ruby %}

const isNotHardware = product => product.type !== 'HARDWARE';
const hasMissingContent = product => product.productContent === undefined;

const hasMissingProductContent = products => {
  if (products) {
    return products.filter(isNotHardware).map(hasMissingContent).includes(true);
  }  else {
    return false
  }
};

{% endhighlight %}


While I was quite happy with the first two functions, I wasn’t entirely happy with the third one - returning false from the else statement didn’t look nice at all, but I couldn’t think of anything better at the time so I decided to create a PR and see if anyone could come up with a more elegant solution. One of the things that someone suggested was to remove the else statement because undefined is also falsey - this would make the code to look less verbose too. While this is true, the fact that we are now returning two different types of data from the same function (boolean or undefined) made me worry but, before we go into details about why I think this is a bad idea, let’s first go back to the very basics of functional programming.

Functional Programming is about data transformations, and the workflow usually goes as follows:

*  You have some input data

*  You pass this data into a function

*  The function applies a series of transformations on the data through some operations of some sort

*  The function returns the result of these transformations without altering the original data


In functional programming, functions are first class citizens; this means that you can treat a function pretty much like any other variable, so you can pass a function to another function as a parameter.

With all of this in mind, imagine now that you have a function that returns two different types of data. If you pass a function that returns two different types of data into another function as a parameter, you will have to validate what was passed in, so you will have to add some additional logic to protect yourself against that ambiguity. Lets explain this with an example.

Going back to the initial code, we want to filter the products that are not hardware and then map through the result of that looking for a product with missing content to decide whether to show an error or not to the customer. Initially I wrote something like the following:

{% highlight ruby %}

const hasMissingProductContent = products => {
    products.filter(isNotHardware).map(hasMissingContent).includes(true);
};

{% endhighlight %}


This solution was great until I realised that products could actually return either a list of products or undefined/null. The filter operation works only on arrays, if we try to call filter on undefined or null it will raise an error. In order to avoid that we will have to validate that products is not null or undefined to make sure that we are only calling filter on an Array type, so we will have something like :


{% highlight ruby %}

const hasMissingProductContent = products => {
  if (products) {
    products.filter(isNotHardware).map(hasMissingContent).includes(true);
  }  
};

{% endhighlight %}

But because I didn’t want to leave someone dealing with the same situation I had to I decided to add an else to make sure that my function was always returning the same type (boolean)

{% highlight ruby %}

const hasMissingProductContent = products => {
  if (products) {
    return products.filter(isNotHardware).map(hasMissingContent).includes(true);
  }  else {
    return false
  }
};

{% endhighlight %}

In this case, because products can have two types, we had to add some logic to check and protect against undefined or null. Analysing the issue, I realised that if I could somehow force products to always return some sort of array (either with elements or an empty array), we wouldn't need to protect against null or undefined and I could remove the conditional. This would look something like:

{% highlight ruby %}

const hasMissingProductContent = products => {
    const customerProducts = products || [];
    return customerProducts.filter(isNotHardware).map(hasMissingContent).includes(true);
};

{% endhighlight %}

I know that the conditional is still being used, but things are starting to look a bit better now. We could go a step further - if products has already been forced to be an array by whoever who passes it into the function, we could remove the conditional logic altogether and have something like:

{% highlight ruby %}

const hasMissingProductContent = products => products.filter(isNotHardware).map(hasMissingContent).includes(true);

{% endhighlight %}

TThis was the solution I was originally going to go with until I realised there could potentially be two different types of products. This solution is cleaner, simpler and easier to maintain in the future. Always returning a single type from a function will allow you and others to write more elegant code.

### Final thoughts

Statically typed languages are a lot more strict and it won’t allow you to return two different types from a function - if you try that, the compiler will scream at you.

Dynamically typed languages give you a lot more freedom to do what you please with your data and functions and this can be very powerful and useful in some situations, but with great power comes great responsibility - as we have shown, returning two different types of data from a function is not a very good idea and it will force others to add conditional logic to protect them against that ambiguity.

There are tools in Javascript that help you avoid this issue, forcing functions to always return a single type - among the most popular ones are typeScript and Flow. These tools add type-checking to Javascript which allows you to force functions to always return a single type of value, and parameters passed into functions to be of a certain type.

However, if you are not using any of this tools always remember that, in functional programming, functions are first class citizens, so you can pass them around to other functions. As a consequence, we need to make sure that our functions always return the same type to avoid adding logic to deal with different scenarios.

Whether Javascript is a functional language or not is itself already a big discussion, but it has functional capabilities, so if you are using it that way you need to pay attention to what your functions return and how they behave. Following a couple of simple rules will allow you to write simpler and more elegant code.
