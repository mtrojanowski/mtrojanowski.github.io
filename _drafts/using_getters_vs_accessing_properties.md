---
layout: post
category : programming
tags : [php, performance]
---
{% include JB/setup %}

There is this common pattern of keeping the properties of your classes private 
(or protected) and only expose them to the outside world by using getters and setters. 
This a good practice but I have quite frequently seen people to overuse those methods in PHP
by utilising them inside other methods of the owning class. It has an impact 
on the script's performance and has no real advantage over using the property directly. (still
remember that we're talking wbout PHP here, I don't know if this is also the case for other languages)

Let's have a look at an example. Consider this class:

{% highlight php %}

class User 
{
    private $name;

    public function getName()
    {
        return $this->name;
    }

    public function addExclamationToName()
    {
        $this->setName(this->getName().'!');
    }
}

{% endhighlight %}

and this one:

{% highlight php %}

class User 
{
    private $name;

    public function getName()
    {
        return $this->name;
    }

    public function addExclamationToName()
    {
         $this->name .= '!';
    }
}

{% endhighlight %}

Obviously they're almost the same. Although if you try to call the `addExclamationToName`
method a large amount of times (don't ask me why on Earth would you do that - after all it's
your program not mine) you will see a difference in time taken by the script.

... ms compared to ... ms when run a milion times*

So why's that?

In a nutshell in PHP calling a user defined function is considerably slower than 
accessing a variable or using a language construct. This is because much more machine code
 is generated for such call. And of course as it won't hurt much if used just
a few times around your scripts, in complex projects, where you have a large amount of these 
calls, the performance loss can be noticeable.

If you want to understand in details why this happens have a look at [this amazing
article on function calls in PHP by Julien Pauli]()

So my advice is: if it is not necessary avoid using getters and setters inside your 
classes. Just access the properties directly. Oh, and the latter method of accessing 
properties also makes you write less code.;)



* I used ApacheBench to make the benchmark test. If you don't know that tool have
a look at it as it is pretty neat :)
