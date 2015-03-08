---
layout: post
category : programming
tags : [AWS, monitoring]
---
{% include JB/setup %}

Amazon's [Simple Storage Service](http://aws.amazon.com/s3/) (or S3 as it's more 
commonly known) is one of the most popular cloud storages on the market. It's 
reliable, easy to manage and relatively cheap. One of its' great features is 
the powerful API offered by Amazon and the choice of SDKs<sup>1)</sup> for some most popular 
languages to facilitate the implementation of the storage in your applications. 
Not only are the SDKs available but they are also well documented which is 
equally important.

There is one feature though the API is missing. It is a straightforward method
to get the disk usage of your S3 bucket (i.e. something like the "root folder" 
or "namespace" for your files in S3). You can get this information from the web
interface by generating some special reports or with using some CLI tools, but you
can't do it directly from the API.

### The easy way

The API provides methods to list all the files in your bucket in alphabetical order.
It is worth noting here that although you can give your files in S3 folder-like 
names S3 in fact uses a flat structure and the "path" is part of the file name 
(AWS names this the object's key).

So you can give your object a key like
{% highlight php %}
 path/to/my/clients/logos/1234/logo.png
{% endhighlight %}
to help keep things organised around your bucket but this does not create any
folder structure inside S3. Even though some GUI tools for manipulating buckets 
will display them to you as folder structure.

This fact gives you the advantage that when you list objects in your buckets you
don't have to worry about traversing the folder tree. With this idea in mind it
is relatively simple to create a simple script that will count the disk space 
usage of your bucket. Using the `getObjectList` (or similar) method from the SDK
you can iterate through the list and sum the sizes of all your objects. The API
will return on each call a maximum of 1000 results in the set. If your bucket 
is larger you will get in the result information that the set is truncated and 
a marker with the key to the object which is next on the list after your last 
returned object (so the one from which you should start reading the next part of
list). 

Here's what it might look like in PHP:


{% gist 22fa648439934f0282cb %}
  
That looks nice and easy. There is a problem though. The more objects we have in
our bucket the more time it takes to run this script. In my company we had 
a production bucket with more than 25 million objects. It was taking more than 
2 hours to count the bucket. That's not cool.

### The cool way

Fortunately there is a way to solve this problem using the AWS API. All we have 
to do is to chunk our list of objects into pieces and call the API 
simultaneously to speed up the process. We can easily obtain the desired part of
our objects' list by passing the `prefix` parameter. Let's assume that we have 
the following objects in our bucket:

{% highlight php %}
 path/to/my/clients/logos/1234/logo.png
 path/to/my/clients/logos/1235/logo.png
 path/to/my/clients/logos/1236/logo.png
 path/to/my/clients/favicons/1234/favicon.ico
{% endhighlight %}

We can pass a `path/to/my/clients/logos` prefix to get only the first three objects
from the bucket. 

So the only problem we have now is to decide how to best chunk our object list 
into pieces. The best would be to make the chunks as even as possible to get even
better results. In our case we had all the keys begin with the ids of our users
so we decided to use part of the id as the prefix. This allowed us to easily create
prefixes automatically: 

 * use ids starting with `10` to those starting with `12`
 * use ids starting with `13` to those starting with `15`
 * use ids starting with `16` to those starting with `18`

...and so on.

Once we know how we want to create the prefixes we can implement a way to 
simultaneously call the API for the results. In my company we decided to use 
the RabbitMQ and a Java Spring-based consumer for the messages. We have one script
which creates lists of prefixes and sends them to Rabbit. Each message contains
the prefixes which a counting thread has to process (using the logic presented 
in the previous chapter). Thanks to using the Spring Rabbit library we can easily 
control the numbers of consumers used to do the counting.

Thanks to splitting the list of our object to 10K pieces and using 20 consumers
we managed to lower the time needed to count space of the 25 million objects 
to less than 4 minutes.


### Other usages for the counting script

Although you could probably find some other solutions to address the problem of
counting your disk space usage on S3 there is a nice advantage of the solution
presented here. In my company we found ourselves a couple of times in situations
where we had to make an operation on each object we had in the storage - e.g. 
change the permissions or Content-Types as we had other scripts which screwed 
these things up. 

To make such operations on all the objects we had to hook into our counting script
and add a proper WebAPI call for each object at the moment its' size was being 
retrieved. This saved us a lot of time as we didn't have to create any custom
scripts from scratch.

### Summary

That might not be the best way, but this solution certainly does the trick. It 
is not very complicated and gives you some control over the time and resources 
taken by the process - either by changing the amount of prefixes you generate or
by the number of threads executing the counting. One thing worth noting at the end
is that Amazon's API tends to bump a request every once in a while if it is 
flooded with too many of them. So if you add to many threads to your script you 
might run into some communication problems with Amazon.


---
1) The API and SDKs are provided for all of the Amazon Web Services, not only S3.
Have a look at [the documentation](http://aws.amazon.com/) to learn more