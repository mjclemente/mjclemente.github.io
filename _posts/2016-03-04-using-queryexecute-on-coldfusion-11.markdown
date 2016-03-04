---
published: true
title: Using QueryExecute (ColdFusion 11)
layout: post
tags: [coldfusion,cfml, QueryExecute]
---
We're still using ColdFusion 10 in production, but we're using ColdFusion 11 for some new development (I doubt we'll use ColdFusion 2016 until it's been out in the wild for a few months.) One of the "new" features in 11 is `QueryExecute`, which provides a more streamlined way to run queries from cfscript. I couldn't find too much out there on how to use it, so I did a little digging as we started to implement it in our code. <!--more-->

In CF10, we ran queries in cfscript using a version of the [syntax that Adam Cameron laid out](http://blog.adamcameron.me/2014/01/using-querycfc-doesnt-have-to-be-drama.html):

	var qry_getSomeThingById = new Query();
    qry_getSomeThingById.addParam(name="fieldID", value=idVariable, cfsqltype="CF_SQL_INTEGER");
    var sql = "SELECT TOP 1 fieldID, fieldName
    FROM theTable
    WHERE fieldID = :fieldID";

    qry_getSomeThingById = qry_getSomeThingById(sql=sql).getResult();

This worked well enough. Similarly, if we wanted to get back a generated ID from an insert, we would just modify the last line to be something like `qry_saveSomething = qry_saveSomething.execute(sql=sql).getPrefix();` and then we would be able to reference `qry_saveSomething.generatedKey`. Scoping the variables was fairly straightforward, as was accessing the results.

So how is this done with  `QueryExecute`? It took a little mucking around, but we got it to work. Here's the basic structure we use for running the queries:

	var params = {
      fieldID = { value = idVariable, cfsqltype = "CF_SQL_INTEGER" }
    };
	var sql = "SELECT TOP 1 fieldID, fieldName
		FROM theTable,
		WHERE fieldID = :fieldID";
	
	var qry_getSomeThingById = QueryExecute( sql, params );

I do like this approach and think it's an improvement. Where I got tripped up was when we needed to access the generated ID. It turns out that you need to pass in a third argument (`{result = "resultName"}`) to the function in order get that information back:

	var resultName = '';
	QueryExecute( sql, params, {result = "resultName"} );

With this syntax, `resultName.generatedKey` is available to use.

Two additional notes on the structure for inserts:

1) You don't need to name the query, because it's not returning any results. In fact, any name you assign to it won't be defined.

2) If you're in a cfc and you don't *var* scope `resultName`, it ends up in the variables scope. Unfortunately, in this situation, I couldn't find an elegant way to *var* scope (you can't run `{var result = "resultName"}`). So, you either need to declare and scope the variable before running `QueryExecute`, or explicitly use the *local* scope (`{result = "local.resultName"}`). 





	



