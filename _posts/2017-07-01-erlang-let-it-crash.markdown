---
layout: post
title:  Let It Crash
date:   2017-07-01 10:00:00
author: Isaac Jarquin
categories: software
excerpt: What is all this let it crash motto than Erlang and Elixir people talk about all the time ?.
comments: true
resume: "The first time a heard about Erlang and Elixir I was very confused about that 'Let it crash' motto that everyone was talking about. This is probably one of the few reason why I like elixir. Surely you shouldn’t let your system crash I thought back in time. Why would I let my system crash and annoy a lot of my customers ? So what was all of this about it ?"
---

![shared](/images/functional-programming/erlang/let-it-crash.jpg){: .image-full-width}

## Let it crash and we will handle it for you.

The first time I heard about Erlang and Elixir I was very confused about that "Let it crash" motto that everyone was talking about. This is probably now one of the few reason why I like elixir. Surely you shouldn’t let your system crash I thought back in time. Why would I let my system crash and annoy a lot of my customers ? So what was all of this about it ? Erlang and elixir was supposed to be great for concurrency and fault tolerance, and here I was being told to let things crash. So let’s dig a bit into it to see what they really mean about that. Imagine you could control your system in a way that can let crash things that are not critical and recover automatically right after the crash in a known stable state, so no one will actually notice that your system has crashed. Im sure you have heard the have you try turning it off and on again motto in IT.

![shared](/images/functional-programming/erlang/itcrowd-turn-off-on.jpg)

So all we need to do is turning it off and on again and problems solve. It sounds idyllic right ? well this is one of the beauties that Elixir and Erlang do for you, and the best part of it is that it come out of the box with absolutely not settings on your side. But before we get excited let's dig a bit about it and see how and when is all this handle.

When we first started talking about let it crash, we obviously didn't mean to let things crash uncontrolled, this will cause a chaos in your system and we don't wanna do that. What Erlang and Elixir want to express is more about crash control, so it doesn’t means you should have uncontrolled failures and let your application burn in case of any errors, what it means is to avoid excessive code for handling errors that can be handled gracefully. Imagine that whenever your system crash for unknown reason, it can be self recover automatically without any external help. This is where things gets very interesting, if we could recover our system to a known stable state after a crash, our customer wouldn’t even notice that the system has crashed, and that would avoid a lot of pain for them. So until now we have talk about letting our system crash only in a controlled way. So the question now is what should I let crash and what should I handle ? to answer those questions we are going to give an introduction to the kind of errors we can find in a system and the impact they cause to our customers.

The type of errors in a computer system can be categorized in many different ways, we are going to pay attention to how often this errors appears and how easy is to replicate then. Into this category we are going to focus in the two most commons errors, Bohrbugs and Heisenbugs:

*  Bohrbugs: a bohrbug is a bug that is solid, observable, and easily repeatable, this sort of errors are the predictable ones. They can be predicted all the time, they are very easy to reproduce and they usually don’t get into production because they have been seen in a previous state of development and handled properly.

*  Heisenbugs: the name of those sort of errors come from the uncertainty principle postulate by Werner Heisenberg, that states that you can't be certain of the position and velocity of electrons measured, exactly, at the same time. All experienced programmers have faced situations where the bug that crashed the software just disappears when the software is restarted. Heisenbugs have unreliable behaviour that manifests itself under certain conditions, they are nasty bugs that happen once in a million, billion, or trillion times and no matter how much time and effort is spent trying to reproduce the problem, we fail.

An study in a few different systems showed that 95% of the bugs that you can find in a computer system are Heisenbugs, is precisely because of the uncertainty and difficulty to reproduce them that they get easily into production. Here is where the let it crash philosophy of Erlang and elixir wins. Those are the sort of bugs, that you should let them crash and then recover the system in a known stable state.


### Final thoughts

Fault tolerance systems like Erlang and elixir will allow the system crash when unpredictable errors appear, log the error in the log system and then will restart the process in an stable state to create the smallest possible impact in your system and customers. Once the error is logged into the system you can decided whether the error is important enough to fix it or assume the failure if it does not have a big impact in your customers. But the most important thing here, is that your system will be up and running in the meantime and your customers will not notice the failure. The way that elixir and Erlang handle those errors is through the supervision tree but I will leave this topic for my next post. Happy handling.
