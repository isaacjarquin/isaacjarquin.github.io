---
layout: post
title:  Supervision Tree
date:   2017-08-03 10:00:00
author: Isaac Jarquin
categories: software
excerpt:  Introduction to supervision trees in Erlang and Elixir.
comments: true
resume: "While these sorts of problems affect equally both functional and OOP programming, we are going to be talking primarily about functional programming, mainly because the style of coding used in this example is functional, but everything in this post can be extrapolated into the OOP world ..."
---

## Have you try turn it off and on again ?

In one of my previous post (Let it crash) we talk about how good elixir and Erlang were at fault tolerance, and how they could restart an app when the app crashes so our customers wouldn’t notice any issues. We also mentioned that all of this was handle by the supervision tree. So now is time to talk a bit about the supervision tree.

In Elixir and Erlang, we have what are known as Supervisors. The responsibility of a supervisor is to check that a process is alive, and restart it if it dies.  A Supervisor can check for processes and other supervisors too. That way we can build a hierarchical process structure called a supervision tree. Supervision trees are a nice way to structure fault-tolerant applications. The main idea of a supervision tree is to have a way to keep our software going in case of errors by just restarting the faulty processes:

![shared](/images/functional-programming/erlang/supervision-tree-general.jpg)


The sole responsibility of a Supervisor is to observe an attached child process, check to see if it goes down, and take some action if that happens. If a worker dies, the supervisor can restart the dead worker and, it processes children if it has any. In a bit more technical details, our supervisors would start a worker process, link to it, and trap exit signals with process_flag(trap_exit, true). The way that the process will get restarted by the supervisor, it will depend on the strategy selected. Supervisors support different supervision strategies :

* one for one: - if a child process terminates, only that process is restarted.

* one for all: - if a child process terminates, all other child processes are terminated and then all child processes (including the terminated one) are restarted.

* rest for one: - if a child process terminates, the “rest” of the child processes, i.e., the child processes after the terminated one in start order, are terminated. Then the terminated child process and the rest of the child processes are restarted.

* simple one for one: - similar to :one_for_one but suits better when dynamically attaching children. This strategy requires the supervisor specification to contain only one child. Many functions in this module behave slightly differently when this strategy is used.

Once a process has died the supervisor will try to restart it again, if it keeps dying it will keep trying until it reaches the max number of restarting. This is a parameter option that will be passed to the supervisor when it is created if it reaches that number it will gives up and terminates. Why would it give up? The main reason is that you don’t want your Supervisor to infinitely restart its children when something is genuinely wrong in your application.

Because supervisors have a very simple job, they are very unlikely to crash themselves.
The beautiful outcome of this is, when something inevitably goes wrong in one of the leaves or the branches of the tree, the problem is isolated and easily recovered without disturbing the rest of the application.

The following code is been extracted from a real backend application and represent the supervision tree of the whole app.


{% highlight ruby %}

defmodule ItemsApi do
  use Application

  def start(type, args) do
    import Supervisor.Spec, warn: false

    children = [
      supervisor(ItemsApi.Endpoint, []),
      supervisor(ItemsApi.Repo, []),
      supervisor(ItemsApi.Mailer, []),
    ]

    opts = [strategy: :one_for_one, name: ItemsApi.Supervisor]
    Supervisor.start_link(children, opts)
  end
end

{% endhighlight %}

This backend system is composed of an API that has a few end points, a db to store some items, that get posted through the API and a mail system that will send emails to certain users when some events happens. If we represent the previous code in a supervision tree map, we will have something like:

![shared](/images/functional-programming/erlang/supervision-tree.jpg)

When the application start, it will call the start function which will create three supervisors children:

{% highlight ruby %}

children = [
  supervisor(ItemsApi.Endpoint, []),
  supervisor(ItemsApi.Repo, []),
  supervisor(ItemsApi.Mailer, []),
]

{% endhighlight %}

The first supervisor will check the worker in charge of keeping  the endpoint up and running, the second one will check the Ecto repository worker and the third one that the mail system is running. All those three processes are completely independent from one another and if one of those process crash, it won’t affect any of the other two, and right after the crash its own supervisor will try to restart the process again. The beauty here is that we will have isolated problems without affecting the rest of the system, creating a fault tolerant system.

{% highlight ruby %}

opts = [strategy: :one_for_one, name: ItemsApi.Supervisor]
Supervisor.start_link(children, opts)

{% endhighlight %}

Then we start the gen server supervisor that will supervise all the children supervisors with the strategy selected. If any of the child supervisors die, then the main supervisor will restart the dead supervisor. But who supervise that main supervisor ? what happen if the main supervisor die ? Well if the main supervisor die, then we will be in trouble at this point, but you need to bare in mind that the only mission of a supervisor is to supervise and they are very old, they were first implemented more than 20 years go in Erlang so they have gone through all kind of testing and are very solid and stable processes that pretty much never crashes unless you do it on purpose, so chances are that you will rarely see one of this supervisor die.

This a fairly basic introduction the the world us supersivors and the supervision three, if you want to know more about I definitely recommend to have a look at
[learn you some erlang - supervisors](http://learnyousomeerlang.com/supervisors)


### Final thoughts

Statically typed languages are a lot more strict and it won’t allow you to return two different types from a function - if you try that, the compiler will scream at you.

Dynamically typed languages give you a lot more freedom to do what you please with your data and functions and this can be very powerful and useful in some situations, but with great power comes great responsibility - as we have shown, returning two different types of data from a function is not a very good idea and it will force others to add conditional logic to protect them against that ambiguity.

There are tools in Javascript that help you avoid this issue, forcing functions to always return a single type - among the most popular ones are typeScript and Flow. These tools add type-checking to Javascript which allows you to force functions to always return a single type of value, and parameters passed into functions to be of a certain type.

<b>References</b>

[Learn you some Erlang - supervisors](http://learnyousomeerlang.com/supervisors)

[Refactoring, Improving the Design of Existing Code](https://books.google.co.uk/books/about/Refactoring.html?id=1MsETFPD3I0C)
