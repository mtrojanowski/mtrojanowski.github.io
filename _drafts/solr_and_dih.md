---
layout: post
category : programming
tags : [solr]
---
{% include JB/setup %}


- CachedSqlEntityProcessor - in the `where` clause you must make sure that the columns match the type or an java cast error occurs.
- in TemplateTransformer make sure that the field is not null as it will generate warnings.
- do not use ScriptTransformer - write in java instead. For ~5mln rows the difference was 2h - 40s
