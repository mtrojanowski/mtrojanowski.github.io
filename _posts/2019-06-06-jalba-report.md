---
layout: post
category : conferences
tags : [jvm, java, reports, tdd, craftsmanship]
---
{% include JB/setup %}

![The Firth of Forth bridges](/assets/jalba/cover.jpg)

My visit to JAlba
-----------------

Recently I returned from JAlba, an open conference held in the capital of Scotland and it was a delightful experience.
If you don't know what an open space conference is (or an unconference as it is also called) - it is a platform
for people from a given field to meet and freely exchange their knowledge and experience. It differs from the
usual conferences that you might be familiar with. First of all, there is usually much less attendees and there are
no speakers. The attendees each day propose and choose topics which they want to talk about on the given day. Important
thing is - to propose a topic you don't have to be an expert, it might be something you actually want to learn about.
Your responsibility, if it's selected, is to act as a moderator for the discussion. This gives an amazing setting for
knowledge exchange as during each session there is a lively discussion instead of a simple presentation. If you want to
learn more about the concept of open conferences you can read about it here:
[Open Space Technology](https://en.wikipedia.org/wiki/Open_Space_Technology)

JAlba is an unconference that focuses on JVM-based languages and software development in general. Engineers and Java
experts from around the world visited Scotland - people working in different companies, freelancers, teachers and
consultants, which helped to get even more interesting insights during the discussions. Discussions on topics
touched in the morning sessions were usually continued during afternoon outings, which allowed us to admire the amazing
Scottish landscapes (and the weather, though some claim it's actually not usual to have such sunny and warm days).

The sessions, the attendees, the surroundings were all that made JAlba an unusual experience. One which I would
recommend to any software engineer. Especially if you're looking for something different from the usual conference that
you go to.

So let's dive into the summaries of some more interesting sessions I attended on JAlba:

Is TDD dead?
------------

A catchy title meant that people were thoroughly interested in the session, and contributed vastly. The conclusion was,
that TDD is obviously not dead, although many people understand the idea differently. Tests are a very important part of the development process, something which some businesses have still not quite understood. But it's also crucial to
test only the right things and keep the right amount of tests. An important thing is not to focus on code coverage but on testing functionalities - to test behaviours, not implementations.

An interesting point was made on writing tests in open-source projects. People often won't write tests when they
contribute as it can be too complicated to do so. We agreed that it should be a good practice to write documentation
that will help people write tests in a way consistent with the style used by the project they contribute to.

Jim Gough, who proposed the session, has written a comprehensive summary of it on [his site](https://jpgough.github.io/blog/2019/05/26/jalba-tdd-dead).

Java & Serverless
-----------------

We've exchanged experiences on using Java in a serverless environment. There a couple of points to remember if you plan
to go serverless with Java:

  1. A function on Amazon Lambda can run for a maximum duration of 3 minutes, and you should take into consideration
     the time that will be needed to start the container. This makes you think more carefully about the design of your
     application, as you have to keep things simple. You should also limit your dependencies to keep the distribution
     packages as small as possible.
  2. If you have reusable code or libraries you can setup shared lambda layers. These layers run between your function
     and the container and can contain code that is needed by many different lambdas.
  3. Although there are some frameworks that help you run serverless there are still some tools missing. People told
     they had to create their own scripts to orchestrate the deployment of sets of lambdas.
  4. Lambdas are not services. You need to have a different place that will handle your workflow - call different
     lambdas in a given flow. E.g. an edge gateway.
  5. It's tricky to work with many lambdas on a local machine, so development might be difficult.

When it's useful to go serverless:
  - when you want to quickly deploy functionalities, e.g. for spikes
  - when you want to do some experiments or POCs
  - when you have functionalities with low request frequencies, especially when you experience unexpected peaks in the
    traffic

Future of Java
--------------

Jose Paumard gave us a quick overview of some interesting changes coming in Java 13 and beyond:
  1. Project Valhalla is trying to adopt Java language and runtime to modern hardware. As part of it an "inline"
     keyword will be introduced. If you mark a class as "inline" this will instruct the JVM that arrays of such objects
     should be kept together in memory - physically next to each other. This will avoid having to do pointer chasing
     when the collection needs to be loaded from memory to the CPU for computations.
  2. Project Amber will, among others, introduce record classes, which you might be familiar with if you use Kotlin.
     These will be classes where you only need to define the constructor parameters - those will be automatically set
     as class properties and you will receive appropriate getters.
  3. There's also project Loom, which is trying to introduce a lightweight concurrency construct to Java.
  4. Since Java 12 you can use GraalVM's JIT compiler instead of C2. (by using the -XX:+UseJVMCICompiler switch) The
     compiler goes well especially with small objects.

If you want to test some of these features but don't feel like building the JDK from source you can download builds
containing early versions of these functionalities from [this repo](https://github.com/forax/java-next/releases)
maintained by Rémi Forax.


Sharing craftsmanship skill in the team
---------------------------------------

When you introduce junior developers to teams it's vital to make sure people share their knowledge across the team.
During the session we gathered the activities which people use in their own environments to achieve this task. Some of
them might seem rather obvious but nevertheless here they are:

  - Organise pair programming.
  - Assign a mentor to the junior.
  - Make sure code reviews are meaningful, that people can learn from them, not only are told what to correct.
  - Try to avoid having "Rock Stars" / "Lone wolves" on the team. People who will make others feel intimidated or
    uncomfortable.
  - Do pair designing - when designing new solutions make sure juniors also take part in it.
  - Send newsletters with tips and interesting findings (either through mail or a communicator). Some people organise a
    "tip of the day", where each senior team member, in turn, was supposed to send some interesting tip to the whole team.
  - Organise mob code reviews - code reviews done by the whole team together. Open-source projects can be used for that if there's a fear that it will intimidate team members who wrote the code under review.
  - Make sure the culture in your organisation encourages exchange of experiences and knowledge.
  - Organise lighting-talk sessions. These might also show some tool tricks.
  - "Techflix" - get together to watch sessions from conferences.
  - Have curated reading lists that you can pass to your juniors. You can also ask them to write some short reviews of
    the books they've read.
  - Organise a "book club" where people can discuss either books or articles they've read recently. As books about
    programming can be quite long the discussions can be organised for separate chapters.

How to do reckon
----------------

Tomasz Borek interviewed JAlbans on how they do reckon, i.e. - what steps do they take whenever they have to decide
whether they want to go with option A or B, regardless if it's a new tool, a new language, a design principle, etc. He
then shared what he's gathered:

  1. Search for alternatives to your choice on services like [alternativeto.net](https://alternativeto.net/),
     [Stackshare](https://stackshare.io/) or by just typing into Google "X vs " and waiting for the hints :)
  2. Google for terms like "X rocks", "X sucks", "X wins", "X loses", "X fast", "X slow", "10 reasons to do X", "10
     reasons not to do X", etc.
  3. Browse techradars to see what people think of your choice.
  4. See what the leaders, makers, domain bloggers, field experts have to say about your choice.
  5. If it's an open-source solution, check how popular is the repo, how many contributors there are, what were the last dates of commits
     and releases, what are the numbers of open issues / PRs, etc.
  6. Check what are the licences, if they are sufficient for your needs.
  7. Check how good are the docs. Are there any tutorials, "5 min guides", etc. ?
  8. Do some time-boxed hands-on. Spend 40-60 minutes on each alternative and then move on.
  9. When trying things out remember to solve the same problems with each alternative and use the same data set, to
    avoid any bias.
