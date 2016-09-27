---
layout: post
title:  Liskov Substitution Principle
date:   2016-09-27 10:00:00
author: Isaac Jarquin
categories: software
excerpt: What is the liskov substitution principle and how can I use it to improve maintainability in my code.
comments: true
resume: "The Liskov Substitution principle is one of the famous SOLID principles. The solid principles are a group of good practices that allow you to write better and cleaner code in a way that is easy to maintain, reusable, highly cohesive and strongly decoupled ..."
---

## Don't change the behavior of the parent or base class in derived classes

The Liskov Substitution Principle (LSP) is a concept in Object Oriented Programming and one of the SOLID principles that was initially introduced by Barbara Liskov in a 1987. This principle states that if a class inherits from a Base class, then the reference to the Base class should be able to be replaced by a Derived class without affecting the functionality of the program. If we inherit from a class, creating a child class, we must make sure that the new derived class only extends functionality without replacing or modifying any of the functionality of the base class. Otherwise the new class could produce undesired effects. In other words you should never modify the behavior of the parent or base class from within derived classes.

In dynamic languages like Ruby, the Liskov principle (LSP) works slightly differently because Ruby less rigidly enforces how types work (so-called “Duck Typing”) as opposed to a language like Java, where type safety is enforced by the compiler. This means that LSP winds up applying more to the messages an object responds to rather than its type.

Imagine a situation like this:

{% highlight ruby %}

class Person
  def talk(words:)
    words
  end
end

class Politician < Person
  def talk
    "Empty promises" 
  end
end

{% endhighlight %}


According to LSP the following should both work

{% highlight ruby %}

class HonestPerson < Person
  def talk_honestly
    talk(words: "Talking honestly")
  end
end

class HonestPerson < Politician
  def talk_honestly
    talk(words: "Talking honestly")
  end
end

{% endhighlight %}


But this will never work because Politician has corrupted the behaviour of the Person class. So an HonestPerson can no longer be a Politician. The Politician class is clearly breaking the Liskov substitution principle because it is modifying the person's behaviour and as a consequence Person is no longer substitutable by a Politician. The talk method was originally defined in the Person class and according to LSP, derived classes should never change the parent class’ behaviour. Because Politician has changed the behaviour of the talk method, when inheriting from Politician, HonestPerson can no longer call the talk method in the same way he did when inheriting from Person class. The second call (inheriting from Politician) in this particular case, will even cause an error because the talk method defined in Politician class does not accept any arguments. If you need to change the parent behaviour, this is a clear sign that this behavior is more likely in the wrong abstraction and you should consider moving it somewhere else.

We should remember that inheritance should be used for specialization. Specialization means creating new subclasses from an existing class so the subclasses can share some behaviour. With this in mind, let's fix the previous example. I know that is very difficult to fix a politician, but I’m going to do my best to achieve the impossible.

{% highlight ruby %}

class Person
  def talk(words:)
    words
  end
end

class Politician < Person
  def empty_promises
    talk(words: "Empty promises")
  end
end

{% endhighlight %}


So now the following will work:

{% highlight ruby %}

class HonestPerson < Person
  def talk_honestly
    talk(words: "Talking honestly")
  end
end

class HonestPerson < Politician
  def talk_honestly
    talk(words: "Talking honestly")
  end
end

{% endhighlight %}

The new implementation of Politician now respects the Person interface. The LSP is no longer violated, and as a consequence the Politician class is no longer breaking LSP so he can now behave like a Person. We can now interchange both classes without breaking any existing functionality.

### Final thoughts

A Politician would do anything to keep lying to us, and even if we try really hard, they will find a way to keep lying. All jokes aside though, the first consequence of not using LSP is that class hierarchies become a mess. If the class hierarchy grows, it will become more and more complicated to know about the behaviour of the child classes. Secondly, unit tests for the superclass would never succeed for the subclass. The code that uses your type will have to have explicit knowledge of the internal workings of derived types to treat them differently. This tightly couples your code and generally makes the implementation harder to use consistently, and also more difficult to change. In the worst case scenario, LSP violations will introduce bugs in your system because you can’t rely anymore in derived classes. LSP is very easy to break if we don’t pay attention to our derived classes, and this can cause a lot of trouble in the feature. As we can see, there are plenty of things that can go wrong when breaking the LSP, so pay attention to your derived classes and try to avoid breaking the LSP. 

