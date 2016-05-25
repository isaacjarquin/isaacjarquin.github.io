---
layout: post
title:  Open Closed Principle
date:   2016-07-01 10:00:00
author: Isaac Jarquin
categories: software
excerpt: What is Open Closed principle and how can I use it to improve readability and maintainability in my code.
comments: true
resume: "Software entities (classes, modules …) should be open for extension but closed for modification. In other words, theoretically you should never need to change existing code or classes ..."
---

#### Software entities (classes, modules …) should be open for extension but closed for modification.

In other words, theoretically you should never need to change existing code or classes; all new functionality should be added by adding new subclasses or methods, or by reusing existing code through delegation. This prevents you from introducing new bugs in existing code. If you never change it, you can't break it. All of this sounds a bit like ideas of an  ideal world, so let's have a look at what these two concepts mean separately to have a better understanding of the consequences of this principle:

*  A module will be said to be open if it is available for extension. In other words, if you can add new functionalities. For example, it should be possible to add fields to the data structures it contains, or new elements to the set of functions it performs.

*  A module will be said to be closed if is available for use by other modules without anyone having to modify any existing code in the module.

Think about that very carefully. If we manage to achieve both of the previous concepts together, the behaviour of all the modules in our system could be extended, without modifying them. That means that you could add new features to that system without modifying any old code. As mentioned before this idea might sound a bit ideal at first but it can actually be achieved by finding the right abstraction, inverting your dependencies to make your parent classes rely on abstractions and not on low level details. By doing that you will find yourself in a pleasant world where new requirements are more than welcome.

Let's see if we can take all of those thoughts into practice. Imagine we have a class that represents a vehicle factory, such a class is responsible for building three different kinds of vehicle: car, bus and lorry.


{% highlight ruby %}

class VehicleBuilder
  def initialize(vehicle_type:)
    @vehicle_type = vehicle_type
  end

  def build
    case @vehicle_type 
    when "car"
      CarBuilder.build
    when "bus"
      BusBuilder.build
    when "lorry"
      LorryBuilder.build
    end
  end
end

{% endhighlight %}

Imagine this: a requirement from the business has just come in saying that they would like to start building a new kind of vehicle, a bike. If you want to extend the current behaviour of the class to support the new vehicle, you wont have any other option than to change the existing class. In this particular case, it could be as simple as adding another case to the `build` method to support the new vehicle, but this sort of thinking can lead to many problems. First, this tends to grow and expand across the whole system. People will add a new branch to the case statement or just copy and paste somewhere else for a similar purpose. Second, in most cases we have to face on a daily basis, we need to deal with much more complex systems. A tiny change can become the biggest of pains, so we have two very good reason to avoid this type of code. In this particular example we could find an abstraction instead of modifying the class  with another branch in the case statement to adapt the new vehicle. That way when another requirement for building a new type of vehicle comes in the future, all we will have to do is add that new type of vehicle without touching the existing code again. 

A possible solution could be to create a class for each type of vehicle and make the vehicles responsible for building themselves. If we make all the vehicles respond to the same interface, the abstract class (VehicleBuilder) doesn't need to know how the vehicles are built, the details about how the vehicles are built is contained in each vehicle. The parent class only relies on the fact that every vehicle responds to the "build" method. In other words, the parent class now relies on an abstraction instead of a concretion. The details about how those vehicle are built will be completely transparent to the parent class:

{% highlight ruby %}

class VehicleBuilder
  def initialize(vehicle:)
    @vehicle = vehicle 
  end
 
  def build
    @vehicle.build
  end
end

class Car
  def build
    CarBuilder.build
  end
end

class Bus
  def build
    BusBuilder.build
  end
end

class Lorry
  def build
    LorryBuilder.build
  end
end

class Bike
  def build
    BikeBuilder.build
  end
end

{% endhighlight %}

With the new solution, if we need to extend the functionality of our code to add a new vehicle, all we need to do is to create a new class with the new type of vehicle that responds to the same `build` method. Finding the right abstraction has given us the ability to extend the behaviour in the future without modifying the existing code anymore. As mentioned before, inverting the dependencies has made our system more flexible. So remember, your high level classes or modules should never depend on details, instead they must depend on abstractions.​

### Final thoughts

Upon first reading, the open/closed principle may seem to be nonsensical. Our languages and our designs do not usually allow new features to be written, compiled, and deployed separately from the rest of the system. We usually find ourselves in a place where the current system is closed for modification, and yet can be extended with new features. Indeed, most commonly we add new features by making a vast number of changes throughout the body of the existing code. We've known this was bad long before Martin Fowler wrote his famous book (Object Oriented Software Construction); but we still do it. So why are people still doing it? We now know there are plenty of tools like Eclipse, Visual Studio, Vim, Text Mate, and the list keeps going on, so why do so many of us keep doing it. They all use plugin architecture which is based mainly in the open/closed principle. They used the tools of object oriented design to separate high level policy from low level detail. They carefully managed their dependencies, inverting those that crossed architecturally significant boundaries in the wrong direction to make sure that all dependencies inside the plugin, point at the system; and that nothing in the system points out towards the plugins. All of those systems are proof that open/closed systems are possible, useful, and immensely powerful.

The open/closed principle is one of the most important object oriented design principles and a very powerful one that could help us to to improve our design by creating flexible interfaces that allow extension without modification. The right use of this principle brings us a world of possibilities that can improve your code in many different ways, make it easy to extend, easy to read, less coupled, and, last but not least, a lot more flexible and easy to change in the future.
