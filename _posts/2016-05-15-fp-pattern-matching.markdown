---
layout: post
title:  Pattern Matching
date:   2016-05-15 11:22:30
author: Isaac Jarquin
categories: software
image: /images/blogs/functional-programming/pattern-matching.jpg
excerpt: Discover how pattern matching works in functions and how can we use it to solve some of the problems caused by using conditionals.
comments: true
resume: "Like many developers, I come from an OOP background. Throughout my time as an OOP developer, I have encountered its strengths and weaknesses. I believe that many of the weaknesses can be overcome through functional programming paradigms, and I want to demonstrate exactly how through an example involving pattern matching ..."
---

Like many developers, I come from an OOP background. Throughout my time as an OOP developer, I have encountered its strengths and weaknesses. I believe that many of the weaknesses can be overcome through functional programming paradigms, and I want to demonstrate exactly how through an example involving pattern matching.

Before we actually start looking at what pattern matching is, we are going to start with a known problem in Object Oriented Programming: conditional statments. I want to specifically target the case statement.

Using a conditional means that you are checking on a type. Type checking is a smell in OOP because you are 'asking' an object if it responds to a particular method. Instead, you should send messages from one object to another, instead of asking what is capable of. In fact some Object Oriented languages such as "Smalltalk" don't even have conditionals at all, and the code can be much cleaner and easier to maintain than that which uses conditionals.

There are several ways you can fix this problem in Object Oriented Programming, my favourite is polymorphism. Let's move from talking in the abstract to a concrete example. Imagine we have the following class:

{% highlight ruby %}

class Salary
  def amount
    case employee_type
    when "engineer"
      BaseSalary.amount
    when "salesman"
      BaseSalary.amount + Commission.amount
    when "manager"
      BaseSalary.amount + Bonus.amount
  end 
end

{% endhighlight %}

As you can see we need to check the type of employee before returning a particular value. In Object oriented programming we shouldn't be asking, we should be telling the class to return the amount, the class should be clever enough to send back the right amount without having to check which type of employee is it. We are going to use polymorphism to fix this:

{% highlight ruby %}

class Salary
  def initialize(employee:)
    @employee = employee
  end

  def amount
    @employee.salary
  end
end

class Engineer
  def salary
    BaseSalary.amount
  end
end

class Salesman
  def salary
    BaseSalary.amount + Commission.amount
  end
end

class Mannager
  def salary
    BaseSalary.amount + Bonus.amount
  end
end

{% endhighlight %}

Clearly there is more code using polymorphism, but the implications and advantages of using this style of programming go beyond the amount of lines of code we need. This code is a lot easier to maintain (we wont need to touch it if we want to extend it, i.e. it adheres to the "Open Close" principle). By isolating the behaviour into small classes, it is now both reusable, and easier to test. In addition, we avoid duplication (DRY). There are many advantages of using this kind of code.

So now let's see what pattern matching is and how can we can use it to solve the same kind of problem. Pattern matching provides a way to "dispatch control" based on structural properties of a value. That sounds a bit abstract, so let's use a very simple example in Elixir to understand what pattern matching is:

{% highlight ruby %}

iex> a = 1  
1
iex> 1 = a  
1
iex> 2 = a
**​ (MatchError) no match of right hand side value: 1

{% endhighlight %}

In the last output you can see Elixir complaining because the value on the left hand side (2) doesn't match the one on the right hand side ("a" was assigned the value "1" in the fist step ).

Pattern matching can do a lot more complex matching, it can match lists: 

{% highlight ruby %}

iex> [a, b, c] = [1, 2, 3]

iex> [1, 2 , 3] = [1 ,a, 3]
**​ (MatchError) no match of right hand side value: 1

{% endhighlight %}

You can also match lists that contains other lists, tuples, head and tail of lists, among much more. Lets take a closer look at what I find most interesting and see how we can do pattern matching on functions. When it comes to functions, Elixir finds a match between the signature of the function definition and the function call. Lets see an example to understand what that means.

{% highlight ruby %}

defmodule MathOper do
  def operation(:sum, a, b) do
    a + b
  end

  def operation(:div, a, b) do
    a / b
  end
end

{% endhighlight %}

Lets say we now call that function as follow

{% highlight ruby %}

MathOper.operation(:sum, 1, 2)

{% endhighlight %}

As you probably guessed, the result of that is the result of calling the first function, and the return value is "3". What happened here was that Elixir tried to match the signature of the function definition with the call of the function and succeed. In this particular case, `operation(:sum, a, b)`. 

Now we can go back to our original problem with the case statement. At this point I'm sure most of you have already figured out how simple the solution using a case statement is going to be with pattern matching. What we will do is create the same function name with different signatures, one for each type of employee:

{% highlight ruby %}

defmodule Salary do
  def amount(:engineer) do
    base_salary
  end

  def amount(:salesman) do
    base_salary + commission
  end

  def amount(:manager) do
    base_salary + bonus
  end
end
{% endhighlight %}

And that could be even simplify to :

{% highlight ruby %}

defmodule Salary do
  def amount(:engineer), do: base_salary
  def amount(:salesman), do: base_salary + commission
  def amount(:manager), do: base_salary + bonus
end

{% endhighlight %}

When you call the function, Elixir will try to match the function call with one of the three previous function definitions. If it finds one, it will return the value, if not, then it will raise an error. As you can see, pattern matching has beautifully fixed our original problem, in a much nicer way, using less code and making it look simple and elegant.

## When is pattern matching useful?

In a surprisingly large number of cases! Every time you have a complex structure of nested if statements, pattern matching can do a much better job with less code.

Another benefit of pattern matching is that if you need to add or modify conditions, you don't end up with a single huge function. You simply add (or modify) appropriate function definitions. The more complex your conditions are, the more pattern matching will help you. Once you're used to it, you start wondering how you ever got through your day without it.

## Final thoughts

Pattern matching is one of the many advantages that functional programming languages offer us. While I really like and enjoy Object Oriented Programming, and I'm a firm believer that both of the paradigms have their own places, they have been designed to solve different kinds of problems, I'm delighted by the way that functional programming solve some kind of problem. We as software engineers have the responsibility to analyse our needs, the needs of the apps, and decide which paradigm or language fits our particular problem.

<b>References</b>

[Programming Elixir 1.2](https://pragprog.com/book/elixir12/programming-elixir-1-2)

[Refactoring, Improving the Design of Existing Code](https://books.google.co.uk/books/about/Refactoring.html?id=1MsETFPD3I0C)

