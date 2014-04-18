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
possibilities. For those of you who do not know what [SOLR](https://lucene.apache.org/solr) 
is a quick explanation. SOLR is a powerful open-source search engine developed 
by Apache. It allows you to create collections of documents which can be then 
queried to quickly retrieve accurate search results. (If you really don't know 
much about SOLR you should definitely take a look at their website, as it's not
possible to mention all the goodies it offers.)

A vital part of working with the search engine is creating the collection of 
documents. The API to create and delete documents is pretty neat but proves to 
be rather insufficient when there is a need to frequently insert or remove large 
number of documents.


## Data Import 
 
## Another section?

- CachedSqlEntityProcessor - in the `where` clause you must make sure that the columns match the type or an java cast error occurs.
- in TemplateTransformer make sure that the field is not null as it will generate warnings.
- do not use ScriptTransformer - write in java instead. For ~5mln rows the difference was 2h - 40s

