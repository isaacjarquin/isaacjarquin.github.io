---
layout: post
title:  Interface Segregation Principle
date:   2016-08-19 10:00:00
author: Isaac Jarquin
categories: software
excerpt: What is the Interface Segregation Principle and how can I use it to improve readability and maintainability in my code.
comments: true
resume: "No client should be forced to depend on methods it does not use. The interface-segregation principle (ISP) is one of the famous SOLID principles. The solid principles are a group of good practices that allow you to write better and cleaner code in a way that is easy to maintain, reusable, highly cohesive and strongly decoupled ..."
---

## No client should be forced to depend on methods it does not use.

The interface-segregation principle (ISP) is one of the famous SOLID principles. The solid principles are a group of good practices that allow you to write better and cleaner code in a way that is easy to maintain, reusable, highly cohesive and strongly decoupled. When we talk about SOLID principles, we see a lot of attention paid to single responsibility, open closed, and dependency inversion principles,  but people don't seem to pay much attention to interface segregation and Liskov substitution principles. This is the main reason why I want to talk about these two principles, so in this article I'm going to focus on the interface segregation principle and leave the Liskov substitution principle for the next post.

The interface segregation principle, as its name implies, has to do with software interfaces, so let's define first what an interface is.

### What is an interface ?
As a general definition, an interface is a shared boundary across which two separate components of a computer system exchange information, whether those components are software or hardware, it doesn't really matter. The important key here is the communication between two different components.
Within object-oriented design, interfaces allow you to communicate to the user of your code the behaviour they need to implement. An interface is just a contract between your modules or classes and their clients. Any class that implements this interface has to fulfil this contract.

### Interface Segregation Principle
The interface-segregation principle states that if an interface grows too much it probably needs to be split into smaller interfaces, because no client should ever be forced to depend on methods it does not use. ISP is intended to keep a system decoupled and thus easier to refactor, change, and redeploy. Look at it this way, let’s said that you don't own a computer, yet someone is trying to sell you a mouse and keyboard. You are not likely to buy them -- why would you if you cannot even use them ?

In the next example (in Java) we will describe the issue and how we can solve this problem. Although I am a Ruby developer at heart, Ruby does not have interfaces builts into the language, so I will provide an example in Java first and then we will talk about how we could implement interfaces in Ruby and why I think is a good idea to use interfaces when you are writing code.

This next example describes a living animal. Let's assume that all the living animals need at least to eat, be able to sleep, and can breathe. With this in mind, if we want to create new animals, they all should implement those three behaviours. So let's create an interface to force this on the client :

```
public interface Animal {
  void eat(int food);
  void breathe(int time);
  void sleep(int time);
}
```

Every animal that we create needs to implement this interface, all the animals need to eat, sleep, and breathe, so for now this interface could fairly represent a living animal. Let's say that a new requirement from the business comes saying they want to implement two kinds of animals, dogs and cats. As we all know dogs bark and cats meow, so let's implement those two behaviours too. If we add those two methods to the interface we will have something like :

```
public interface Animal {
  void eat(int food);
  void sleep(int time);
  void breathe(int time);
  void bark(int time);
  void meow(int time);
}
```

There is something here that doesn't really fit. Eating, sleeping, and breathing are clearly qualities of any living animal but not all the animals bark or meow. If a client wanted to implement those two animals using the previous interface, they would need to write something like :

```
public class Dog implements Animal {
  void eat(int food);
  void sleep(int time);
  void breathe(int time);
  void bark(int time);
  void meow(int time);
}
```
```
public class Cat implements Animal {
  void eat(int food);
  void sleep(int time);
  void breathe(int time);
  void bark(int time);
  void meow(int time);
}
```

As you can see, Cat class needs to implement bark and Dog class needs to implement meow, although we all know that a cat will never bark and vice versa. So why do those two classes need to implement methods that they will never use? the answer to that question is very simple: they don't. This implementation is clearly breaking the Interface Segregation Principle, a client should never be forced to depend on methods it does not use. The Animal interface should only implement methods that all animals should have, and leave the specific qualities of animals to be implemented by a different interface or subclass. A possible solution could be something like :

```
public interface Animal {
  void eat(int food);
  void sleep(int time);
  void breathe(int time);
}
```
```
public interface Dog extends Animal{
  void bark(int time);
}
```
```
public interface Cat extends Animal {
  void meow(int time);
}
```

Now we could implement our two animals :

```
public class Dog implements Dog {
  void eat(int food);
  void sleep(int time);
  void breathe(int time);
  void bark(int time);
}
```

```
public class Cat implements Cat {
  void eat(int food);
  void sleep(int time);
  void breathe(int time);
  void meow(int time);
}
```

Splitting the interface into smaller interfaces has given us the ability to design a system where classes only rely on things they truly need. This sort of code leads us to a world where things are a lot easier to share across the system and also easier to change and deploy. Let’s say that you change the bark method, this change will only affect the clients that rely on this method, only a small group of animals will be affected by your changes. Before implementing animal-specific interfaces a change in "bark" would affect  all living animals, even cats that don’t need this method. Imagine now that we have a thousand different animals. Your classes will become cluttered with a lot of methods that they don't really need. On top of this, having small interfaces also makes things a lot easier when testing and sharing. As you can see there are several advantages of using this kind of thinking.

So the question now is, why does ruby not implement interfaces? Ruby is a dynamically typed language, which means that code is checked at run time. Interfaces are a concept from statically typed languages as it depends on a compilation step, if you write a client that implements an interface and your client has not implement all the methods from the interface, the compiler will complain about the missing methods. Unfortunately, or fortunately depending on how you look at it (I see it as a mixed blessing), there is no compiler in Ruby, so we don’t really have a good way of having the system yell at us if we don’t implement the methods you were supposed to. But interfaces are only a mechanism or a pattern to fix a particular kind of problem in OOP, it just happens to be built into some static languages. From my point of view, what's important here is the problem that we are trying to fix. We are trying to communicate to other developers what methods or behaviours they need to implement when relying on our classes, whether you use interfaces or not to fix this problem is really up to you, but there is still plenty we can do to help those who are implementing our classes to let them know what they need to implement and to find out what they haven’t implemented yet before their program executes. The fact that ruby doesn't implement interfaces naturally doesn't mean that you can’t implement your own interface system, or at least something that works in a similar way to handle those communication issues. Let's see how could we do that and apply to the previous example in Ruby:

{% highlight ruby %}

class AnimalInterface
  def eat(food)
    raise NotImplementedError, "Implement this method in a child class"
  end

  def sleep(time)
    raise NotImplementedError, "Implement this method in a child class"
  end

  def breathe(time)
    raise NotImplementedError, "Implement this method in a child class"
  end
end

class DogInterface < AnimalInterface
  def bark
    raise NotImplementedError, "Implement this method in a child class"
  end
end

class CatInterface < AnimalInterface
  def meow
    raise NotImplementedError, "Implement this method in a child class"
  end
end

{% endhighlight %}

Now if you want to create classes that implement those interfaces all you need to do is inherit from then:

{% highlight ruby %}
class Dog < DogInterface
  def eat() "now i’m full" end
  def sleep() "sleeping" end
  def breathe() "breathing" end
  def bark() "barking" end
end

class Cat < CatInterface
  def eat() "now i’m full" end
  def sleep() "sleeping" end
  def breathe() "breathing" end
  def meow() "meowing" end
end

{% endhighlight %}

The main difference between the two of them (Java and Ruby) is that in Java you will detect any missing method at compile time while in ruby it will be detected at run time. Although catching bugs at runtime is not ideal, you can still write tests to ensure that the contract between the interface and the client is satisfied before you actually run the code in production.

## Final thoughts
Interfaces express intent, they tell you what you need in order to use a module or class. Writing classes that communicate to their clients what they need to implement is a powerful tool. Code that communicates its intent is code that is easy to extend and use. Also breaking interfaces into smaller ones makes your code easy to change and extend so if you think that your interfaces are getting too big, take a few minutes and try to see if they can be split into smaller pieces to adjust to ISP. Combine this two powerful principles and your code will gain in flexibility and usability.
