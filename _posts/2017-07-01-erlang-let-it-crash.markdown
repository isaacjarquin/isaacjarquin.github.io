---
layout: post
title:  Let It Crash
date:   2017-07-01 10:00:00
author: Isaac Jarquin
categories: software
excerpt: What is all this let it crash motto than Erlang and Elixir people talk about all the time ?.
comments: true
resume: "If you've ever looked at Erlang before, you have probably heard about that 'Let it crash' motto that everyone in Erlang and elixir talks about. This is one of the few reason why I love elixir, although I was very confused when I first heard about it. Surely you shouldn’t let your system crash I thought back in time."
---

![shared](/images/functional-programming/erlang/let-it-crash.jpg)

## Let it crash and we will handle it for you.

If you've ever looked at Erlang before, you've probably heard about that "Let it crash" motto that everyone in Erlang and elixir talks about. This is one of the few reason why I love elixir, although I was very confused when I first heard about it. Surely you shouldn’t let your system crash I thought back in time. Why would I let my system crash and annoy a lot of my customers ? So what was all of this about it ? Erlang and elixir was supposed to be great for concurrency and fault tolerance, and here I was being told to let things crash. So let’s dig a bit into it to see what they really mean about that. What if you could control your system in a way that can let crash things that are not critical and recover automatically right after the crash in a know stable state ?.

![shared](/images/functional-programming/erlang/itcrowd.jpeg)

What Erlang and elixir want to express is more about crash control, so it doesn’t means you should have uncontrolled failures and crashes all over your system and let your application burn in case of any errors, what it means is to avoid excessive code for handling errors that can’t be handled gracefully. We could go even farther, what if whenever your system crash for unknown reason, it can be self recover automatically without any external help ? this is where things gets very interesting, if we could recover our system to a known stable state after a crash, our customer wouldn’t even notice that the system has crashed, and that would avoid a lot of pain for them. Ok so we should only let our system crash in a controlled way. So the question now is what should I let crash and what should I handle ? to answer those questions we are going to give an introduction to the kind of errors we can find in a system and the impact they cause to our customers.

The type of errors in a computer system can be categorized in many different ways, we are going to pay attention to how often this errors appears and how easy is to replicate then. Into this category we are going to focus in the two most commons errors, Bohrbugs and Heisenbugs:

*  Bohrbugs: a bohrbug is a bug that is solid, observable, and easily repeatable, this sort of errors are the predictable ones. They can be predicted all the time, they are very easy to reproduce and they usually don’t get into production because they have been seen in a previous state of development and handled it properly.

*  Heisenbugs: the name of those sort of errors come from “Werner Heisenberg” who postulate Heisenberg's uncertainty principle, that states that the position and velocity of electrons in an atom cannot both be measured, exactly, at the same time. All experienced programmers have faced situations where the bug that crashed the software just disappears when the software is restarted. Heisenbugs have unreliable behaviour that manifests itself under certain conditions, they are nasty bugs that happen once in a thousand, million, billion, or trillion times and no matter how much time and effort is spent trying to reproduce the problem, we fail.

An study in a few different systems showed that 95% of the bugs that you can find in a computer system are Heisenbugs, is precisely because of the uncertainty and difficulty to reproduce them that they get easily into production. Here is where the let it crash philosophy of erlang and elixir wins. Those are the sort of bugs, that you should let them crash and then recover the system in a known stable state. Fault tolerance systems like erlang and elixir will allow the system crash when those errors appear, log the error in the log system and then will restart the process in an stable state to create the smallest possible impact in your system and customers. And all of this come out of the box without any extra work. Once the error is logged into the system you can decided whether the error is important enough to fix it or assume the failure if it does not have a big impact in your customers. But the most important thing here, is that your system will be up and running in the meantime and your customers will not notice the failure.
The way that elixir and erlang handle those errors is through the supervision tree but I will leave this topic for my next post. Happy handling.


### Final thoughts

In a bad design the high level classes are used directly and depends heavily on the low level classes. It’s extremely common when writing software to implement it such that each module or method specifically refers to its collaborators, which does the same. This type of programming typically lacks sufficient layers of abstraction, and results in a very tightly coupled system, since every module is directly referencing lower level modules.
When the dependency inversion principle is applied, it means the high level classes are not working directly with low level classes, they are using interfaces as an abstract layer.
Effectively using the dependency inversion principle reduces coupling between different pieces of code and add a lot more flexibility to our system.

Of course nothing comes for free, using this principle implies an increased effort, it will result in more classes and interfaces to maintain, in a few words it will add more complexity to our system. This principle should not be applied blindly for every class or every module. Think carefully if your class is likely to change in the future, if the answer is yes then go ahead and apply the dependency inversion principle. On the other hand if the answer is No, If we have a class functionality that is more likely to remain unchanged in the future, there is no need to apply the principle.
