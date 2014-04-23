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

{ % highlight xml %}
<entity name="myField" transformer="script:addDynamicField" query="..."></entity>
{ % endhighlight %}

I created a dead simple function to achieve the goal:

{ % highlight javascript %}
function addDynamicField(row) {
    var dynamicColumn = "field_" + row.get('field_key');
    row.put(dynamicColumn, row.get('field_value'));
    return row;
}       
{ % endhighlight %}

I ran the import and... whoops... Instead of around 1 minute it took more than 2 
hours to complete! In our setup we were supposed to rebuild the collection every
15 minutes so this was not acceptable. You might have already guessed that using
Javascript was the culprit here (I'm not really good at suspense). Fortunately 
there was an easy solution. Instead of using the Script Transformer I used a 
regular transformer which utilises a Java class.

{ % highlight xml %}
<entity name="myField" transformer="my.solr.transformer.FieldTransformer" query="..."></entity>
{ % endhighlight %}

{ % highlight java % }
package my.solr.transformer;

import java.util.Map;

public class FieldTransformer {

    public Object transformRow(Map<String, Object> row) {
        row.put("field_"+row.get("field_key"), row.get("field_value"));
        return row;
    }
}
{ % endhighlight % }
 
The import took now again less than a minute. In my database I had around 5mln 
records which needed to be transformed by the script. This doesn't seem to be 
that much yet the Script Transformer just can't handle the load.
 
## Avoid nulls when using TemplateTransformer

## Make sure columns match types when using the CachedSqlEntityProcessor

- CachedSqlEntityProcessor - in the `where` clause you must make sure that the columns match the type or an java cast error occurs.
- in TemplateTransformer make sure that the field is not null as it will generate warnings.


