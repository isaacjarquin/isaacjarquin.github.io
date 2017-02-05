---
layout: post
title:  Dependency Inversion Principle
date:   2017-02-01 10:00:00
author: Isaac Jarquin
categories: software
excerpt: What is the dependency inversion principle and how can I use it to improve maintainability in my code.
comments: true
resume: "The Dependency Inversion principle is one of the famous SOLID principles. The solid principles are a group of good practices that allow you to write better and cleaner code in a way that is easy to maintain, reusable, highly cohesive and strongly decoupled ..."
---

## High level policy should not depend on low level details.

Software entities need to cooperate with each others in order to solve complex problems. With that in mind, it is not reasonable to say that a class could have zero dependencies or coupling, but we want to keep this coupling to the minimum. You want to decouple your system so that you can change individual pieces without having to change anything more. The Dependency Inversion Principle is the key to this goal.

The Dependency Inversion principle was first defined by Robert C. Martin in his book Agile Software Development, Principles, Patterns, and Practices, and it is the last of the five SOLID agile principles. The principle states:

*  High-level modules should not depend on low-level modules. Both should depend on abstractions.

*  Abstractions should not depend on details. Details should depend on abstractions.

In the context of static languages like Java it’s easier to explain it, because there are language constructs like Interface and abstract classes to provide the programmer ways to supply “abstract dependencies”.

But what about Ruby? It’s a dynamic language and we don’t need to specify the types of the dependencies (duck typing is the single best feature a language can have, IMHO), so we get to use the DIP for free, right? Well, provided you apply some simple techniques, the answer is yes. Let’s talk about that.


Let’s start with an example of a class that does not follow the dependency inversion principle. Imagine a situation where we have a service that uses directly a Sql DBx

{% highlight ruby %}

class UserService
  def find(user_id)
    SELECT * FROM Users where user.id = user_id
  end
end

{% endhighlight %}


Imagine now that you have a few other services in your system that uses those kind of queries too. For now everything is ok, there is nothing wrong as everything seems to work pretty well. All of a sudden a new Senior joins the team and he's been told that the current database, is operating too slow so he needs to come up with a better alternative. After a long study and analysis he decides to go with a NoSql DB. All of a sudden our current implementation is no longer valid and he needs to change a few queries in different places because the system was depending in low level details of a Sql DB. Changing a few lines of code is definitely not the end of the world but if your system is medium to large in size you will probably have a couple of hundreds of those queries. Changing a couple of hundreds queries is a major problem, which would take you at least a couple of days of changes and testing, in the best case scenario, the larger your system is, the larger your problem will be.

Even though we haven’t change any behaviour or functionality in our system we still need to do a lot of changes. This is a major issue and the problem arise because we violate the dependency inversion principle and rely in low level details of a Sql DB.
So how do we fix this issue? In order to avoid such problems we can introduce an abstraction layer between high level classes and low level classes and invert the dependencies. Since the high level modules contain the complex logic they should not depend on the low level modules and also the new abstraction layer should not be created based on low level modules. Low level modules are to be created based on the abstraction layer. The abstraction layers should describe some interface that all the low level modules needs to implement. So let’s fix this problem step by step. The first thing we need to do is create an abstraction and move the dependency into the abstraction.


{% highlight ruby %}

class UserService
  def find(user_id)
    DBAbstraction.new.find(user_id)
  end
end

class DBAbstraction
  def find(user_id)
    SELECT * FROM Users where user.id = user_id
  end
end

{% endhighlight %}


With the new implementation, if we make all our services to rely on the DBAbstraction layer, we will only need to make changes in one place. This is a clear improvement of our system, we no longer have to do plenty of changes across our objects, all the changes will be done in a single place. So we are done right ? No really, we have definitely improved our system but the DBAbstraction still depends on low level details. We can go further and invert the dependencies.

{% highlight ruby %}

class UserService
  def find(user_id)
    DBAbstraction.new.find(user_id)
  end
end

class  DBAbstraction
  def initialize(db: SqlDB.new)
    @db = db
  end

  def find(user_id)
    @db.find(user_id)
  end
end

class SqlDB
  def find(user_id)
    SELECT * FROM Users where user.id = user_id
  end
end

{% endhighlight %}

Now if we want to change the db all we need to make sure is that the new db implements the abstraction interface(find), so we could easily use a NoSql db now (Mongo for example) :

{% highlight ruby %}

class NoSqlDB
  def find(user_id)
    db.users.find(user_id)
  end
end

{% endhighlight %}

The new implementation is completely db agnostic, is not coupled to any db, instead it defines an abstraction which all the dbs needs to implement. As long as we respect this rule we won’t need to worry about the implementation details of any db’s anymore, not even a single change will have to be made to our system in order to change the data layer in the future. Our system now is a lot more flexible and easy to change. We could eventually change the DB for a different one, for example (Elastic search) and we wouldn't have to change a single line of code in our system.

Finally, this last solution is also much easier to test. Injecting the dependencies into the object allow us to easily mock those objects and make sure they produce the correct output.

### Final thoughts

In a bad design the high level classes are used directly and depends heavily on the low level classes. It’s extremely common when writing software to implement it such that each module or method specifically refers to its collaborators, which does the same. This type of programming typically lacks sufficient layers of abstraction, and results in a very tightly coupled system, since every module is directly referencing lower level modules.
When the dependency inversion principle is applied, it means the high level classes are not working directly with low level classes, they are using interfaces as an abstract layer.
Effectively using the dependency inversion principle reduces coupling between different pieces of code and add a lot more flexibility to our system.

Of course nothing comes for free, using this principle implies an increased effort, it will result in more classes and interfaces to maintain, in a few words it will add more complexity to our system. This principle should not be applied blindly for every class or every module. Think carefully if your class is likely to change in the future, if the answer is yes then go ahead and apply the dependency inversion principle. On the other hand if the answer is No, If we have a class functionality that is more likely to remain unchanged in the future, there is no need to apply the principle.
