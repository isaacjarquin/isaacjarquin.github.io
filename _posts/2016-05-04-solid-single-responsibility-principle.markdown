---
layout: post
title:  Single Responsibility Principle
date:   2016-05-04 12:20:00
author: Isaac Jarquin
categories: Sofware (Object Oriented Design)
image: /images/blogs/object-oriented-design/solid/solid_single_responsability_principle.jpg
excerpt: "How to write clean, maintainable code"
resume: "The Single responsibility principle is one of the famous SOLID principles. The solid principles are a group of good practices that allow you to write better and cleaner code in a way that is easy to maintain, reusable, highly cohesive and strongly decoupled ..."
---

The Single responsibility principle is one of the famous SOLID principles. The solid principles are a group of good practices that allow you to write better and cleaner code in a way that is easy to maintain, reusable, highly cohesive and strongly decoupled.

The single responsibility principle states that a class should have one, and only one reason to change. The first question we should probably ask ourselves is what qualifies as a reason to change?
Some developers think that bug fixes or refactoring are reasons to change, in my opinion your code should not be responsible for bug fixes or refactoring. You don't design your code to introduce bugs, in fact bugs are normally a consequence of bad design. when you refactor your code, you do it in a way that the behaviour of the code remains the same, the code will keep behaving exactly the same as it did before. When we said a class or a module should have only one reason to change what we really mean is that a class should only have one purpose, it should behave in a single way, and that unique behaviour is the only one that can introduce a change in the class. If we think about it, what we are doing with this is increasing the cohesion between things that change for the same reasons, and decreasing the coupling between those things that change for different reasons. 

In the example below, written in Ruby, we show a class that could eventually be used to show information about a book(presentation purpose), to save a book(persistence) and also to find a book in the library(search engine). If we analyze the class carefully we can easily see that this piece of code behaves in three different ways, this class is responsible for three completely different behaviours so as a consequence it could change for different reasons.


{% highlight ruby %}

class Book
  def title
    "a great book"
  end

  def author
    "John Doe"
  end

  def save(book:)
    Library.save(book)
  end

  def find(book_id:)
    Library.find(book_id)
  end
end

{% endhighlight %}

The current approach breaks the single responsibility principle, the class Book is not only responsible for presenting information about a particular book, but also responsible for saving a book into the library and finding a book from a library. A possible solution to this problem could be to separate those three different concerns into three different classes, so each class will be responsible for only one thing :

{% highlight ruby %}

class Book
  def title
    'a great book' 
  end

  def author
    'John Doe'
  end
end

class BookStore
  def save(book:)
    Library.save(book)
  end
end

class BookFinder
  def find(book_id:)
    Library.find(book_id)
  end
end

{% endhighlight %}


With the new implementation if a change in the future is required, let's say to change the save behaviour, this change will only affect the BookStore class. Separating those concerns has isolated each class, increasing cohesion between the things that change for the same reason and decreasing decoupling between those that change for a different one. As a consequence of this we have made our code more robust to changes and easier to maintain in the future.


## Final thoughts
The Single Responsibility Principle leads to a low coupled design with fewer and lighter dependencies, but as any coin, it has two sides. Excessive SRP consideration can easily lead to premature optimizations and instead of a better design, you could end up with a scattered one, where the clear responsibilities of classes or modules might be hard to find or understand. So don't try to design your classes from the beginning using the single responsibility principle, go ahead and design code that make sense for you first, and is as easy as possible to understand. Once you have done that, it will be a lot easier to identify flaws in your code, it will then be time to think about single responsibility, analyze your code, and see if you can find a violation of the SRP. Then try to break the class into as many classes as behaviours you may have identified.
