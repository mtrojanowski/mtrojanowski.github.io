---
layout: post
category : programming
tags : [php, performance]
---
{% include JB/setup %}

There is this common pattern of keeping the properties of your classes private 
(or protected) and only expose them to the outside world by using getters and setters. 
This might be a good practice but I have quite frequently seen people to overuse those methods in PHP
by utilising them inside other methods of the owning class. It has an impact 
on the script's performance and has no real advantage over using the property directly. (still
remember that we're talking about PHP here, this does not have to be the case for other languages)

Consider this class:

{% highlight php %}

class User 
{
    private $name;

    public function getName()
    {
        return $this->name;
    }
	
	public function setName($name)
	{
		$this->name = $name;
	}

    public function addExclamationToName()
    {
        $this->setName($this->getName().'!');
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

Obviously they're almost the same as far as business logic is concerned. Although if you try to call the `addExclamationToName`
method a large amount of times (don't ask me why on Earth would you do that - after all it's
your code not mine) you will see a difference in time taken by the script. The code can be 
more than ten times faster if there are hundreds thousands of such calls.

So why's that?
--------------

In a nutshell, in PHP calling a user defined function is considerably slower than 
accessing a variable or using a language construct. This is because much more machine code
 is generated for such call. And of course as it won't hurt much if used just
a few times around your scripts, in complex projects, where you have a large amount of these 
calls, the performance loss can be noticeable.

If you want to understand in details why this happens have a look at [this great
article on function calls in PHP by Julien Pauli](http://jpauli.github.io/2015/01/22/on-php-function-calls.html)

So my advice is: if it is not necessary avoid using getters and setters inside your 
classes. Just access the properties directly. The latter method also makes your code more
readable (not to mention that you write less code ;) ).
