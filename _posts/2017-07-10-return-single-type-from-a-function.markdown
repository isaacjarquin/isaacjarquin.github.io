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

While working on a ticket, a couples of weeks ago I created a PR that included something like the following:

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


While I was quite happy with the first two functions, I wasn’t entirely happy with the third one but I couldn’t think of an improvement at the time so I decided to create a PR and see if anyone could come up with a more elegant solution. One of the things that someone suggested was to remove the else because undefined is also falsey, and this will make the code to look less verbose too. While this is true, the fact that we are now returning two different types of data from the same function (boolean or undefined) made me worry. Before we go into details about why this is a very bad idea let's go back first to the very basics of functional programming.

Functional Programming is about data transformations, and the workflow usually goes as follow:

*  You have some data input

*  You pass this data into a function

*  The function apply a series of transformation on the data through some sort of operations

*  The function returns the result of this transformation without altering the original data


In functional programming, functions are first class citizens, this means that you can treat a function pretty much like any other variable, so you can pass a function to another function as a parameter. With all of this in mind, imagine now that you have a function that return two different types of data. If you pass a function that return two different types of data, you will have to check on types in the  caller which will make you to add some unnecessary logic to protect yourself against that ambiguity.

Going back to the initial example, we wanted to filter the products that are no hardware and then map through the result of that looking for a product with missing content to decided weather to show an error or not to the customer. Initially I wrote something like the following


{% highlight ruby %}

const hasMissingProductContent = products => {
    products.filter(isNotHardware).map(hasMissingContent).includes(true);
};

{% endhighlight %}


This solution was great until I realized that products could actually return either a list of products or undefined/null. The filter operation works only on arrays, if  we try to call filter on undefined or null it will raise an error. In order to avoid that we will have to do a check on null or undefined to make sure that we are only calling filter on the array type, so we will have something like :  


{% highlight ruby %}

const hasMissingProductContent = products => {
  if (products) {
    products.filter(isNotHardware).map(hasMissingContent).includes(true);
  }  
};

{% endhighlight %}

But because I didn’t want to leave someone dealing with the same situation that I had to I decided to add an else to make sure that my function was always returning the same type (boolean)

{% highlight ruby %}

const hasMissingProductContent = products => {
  if (products) {
    return products.filter(isNotHardware).map(hasMissingContent).includes(true);
  }  else {
    return false
  }
};

{% endhighlight %}

In this case because products can have two types, we had to add some logic to check and protect against undefined or null. Analyzing the issue I realized that if I could somehow force products to return always some sort of array, either with elements or an empty array, we wouldn't need the protection against null or undefined and I could removed the conditional, so I come up with something like :

{% highlight ruby %}

const hasMissingProductContent = products => {
    const customerProducts = products || [];
    return customerProducts.filter(isNotHardware).map(hasMissingContent).includes(true);
};

{% endhighlight %}

Diving in deeper, if products has already been forced, by whoever who pass it into the function to be an array, we could even have something like

{% highlight ruby %}

const hasMissingProductContent = products => products.filter(isNotHardware).map(hasMissingContent).includes(true);

{% endhighlight %}

Which is even more clear, this was the solution I was originally trying to come up with till I found the issue with two types of products.

### Final thoughts

Remember in functional programming functions are first class citizens so you can pass them around to other functions, as a consequence we need to make sure that our functions return always the same type to avoid adding logic to deal with different scenarios. This will allow you to write more simple and elegant code.
