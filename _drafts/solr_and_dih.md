---
title: Tips on using Data Import Handler with SOLR
description: Some useful tips when using a Data Import Handler to create collection
             of documents for SOLR.
layout: post
category : programming
tags : [solr]
---
{% include JB/setup %}

I've worked with SOLR for almost two years know and I'm still impressed by its' 
possibilities. And the Data Import Handler makes SOLR even more exciting (you 
can think it's creepy but I really enjoy working with DIH). 

For those of you who are not familiar with the topic a quick explanation. 
[SOLR](https://lucene.apache.org/solr) is a powerful open-source search engine 
developed by Apache. It allows you to create collections of documents which can 
be then queried to quickly retrieve accurate search results. The 
[Data Import Handler](http://wiki.apache.org/solr/DataImportHandler) is used to 
efficiently create and update collection of documents used by SOLR.

As the [SOLR wiki](http://wiki.apache.org/solr/) is very well written and there
are some tutorials out there on this topic I won't go into the details of implementing
theses tools. Nevertheless I want to share some findings I made when working 
with DIH. I hope this will help someone solve a problem quicker (I really hope 
this will help me when I get stuck on the same issues in five years time).


## Be careful when using the ScriptTransformer

When fetching rows of data using DIH you can use different Transformers to 
prepare the data the way you need. Especially interesting is the Script Transformer
which enables you to run JavaScript code on the row of data before persisting it
as a document. In my case I needed to create dynamic fields in the document using
data from database as both the field name and the field value. 

{% highlight xml %}
<entity name="myField" transformer="script:addDynamicField" query="..."></entity>
{% endhighlight %}

I created a dead simple function to achieve the goal:

{% highlight javascript %}
function addDynamicField(row) {
    var dynamicColumn = "field_" + row.get('field_key');
    row.put(dynamicColumn, row.get('field_value'));
    return row;
}       
{% endhighlight %}

I ran the import and... whoops... Instead of around 1 minute it took more than 2 
hours to complete! In our setup we were supposed to rebuild the collection every
15 minutes so this was not acceptable. You might have already guessed that using
Javascript was the culprit here (I'm not really good at suspense). Fortunately 
there was an easy solution. Instead of using the Script Transformer I used a 
regular transformer which utilises a Java class.

{% highlight xml %}
<entity name="myField" transformer="my.solr.transformer.FieldTransformer" query="..."></entity>
{% endhighlight %}

{% highlight java %}
package my.solr.transformer;

import java.util.Map;

public class FieldTransformer {
    public Object transformRow(Map<String, Object> row) {
        row.put("field_"+row.get("field_key"), row.get("field_value"));
        return row;
    }
}
{% endhighlight %}
 
The import took now again less than a minute. In my database I had around 5mln 
records which needed to be transformed by the script. This doesn't seem to be 
that much, yet the `ScriptTransformer` just can't handle the load.

 
## Avoid nulls when using TemplateTransformer

`TemplateTransformer` is another one of the family of Transformers you can apply
to your field. This one allows to modify the value of the field based on a given
template. One important thing is to avoid having empty fields when using this 
transformer. Normally DIH would just ignore the field and skip adding it to the
document, but when it's used in the template this is not the case. 

So to avoid runtime errors, if you use something like this:

{% highlight xml %}
<entity name="category" query="..." transformer="TemplateTransformer">
    <field name="category_name"           column="c_name" />
    <field name="main_category_name"      column="mc_name" />
    <field name="category_name_with_main" column="c_name_wm"  
        template="${category.c_name}_${category.mc_name}" />
</entity>
{% endhighlight %}

Make sure that the `c_name` and `mc_name` columns are not `null`. 


## Using the CachedSqlEntityProcessor

The `CachedSqlEntityProcessor` is a great way of keeping the database calls limited.
For example, when fetching all the products from a database and assigning the 
category name to each product DIH, when you use `CachedSqlEntityProcessor` will
make just one call to the database to get all the categories and then use the 
cached data. You have to specify a proper key so that the engine knows how to find
the relevant rows:

{% highlight xml %}
<entity name="product" query="SELECT product_id, category_id FROM Products">
    <field name="id" column="product_id" />
    <entity name="category" query="SELECT id AS c_id FROM Categories" 
        processor="CachedSqlEntityProcessor" where="c_id=product.category_id">
        <field name="category_name" column="name" />
    </entity>
</entity>
{% endhighlight %}

Note that there is no `WHERE` part in the query - you specify the condition used
to match products and categories in the `where` parameter of the `<entity>` tag.

An important thing to remember: the columns used to match the values must have 
the same types (the `category_id` in \`Products\` and `id` in \`Categories\`in this 
example). If the types are different (e.g. one column uses unsigned integers and
the other signed) a mismatch can happen inside Java and you will see a 
`ClassCastException` - "*type* cannot be cast to _type_".


## To sum up

These are of course only some tips for people who would run into troubles I've 
already had. I hope that you didn't expect me to write a tutorial on using DIH 
with SOLR ;) (you must feel really disappointed now if you did) I encourage you
to leave comments so that my future posts are just a tiny bit better.

