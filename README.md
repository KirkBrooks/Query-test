# Query-test
This is small sample 4D v15 db demostrating different approaches to conducting queries. 

When the database is first opened a samle table of 500,000 records is created. These records include a UUID, a longint, a string and a time stamp with some indexed and non-indexed. This provides a reasonable dataset to test against. 

I was curious what kind of differences exist between running a query 'naked' (directly in a method) vs passing the query param to a method and running the query there. Then I wanted to see if there was a significant difference in passing the the query params as a native 4D data type or in a c-object. 

Here's what I mean by 'naked' query: 
```
// myMethod
QUERY([Table_1];[Table_1]SerialNumber_indx=$aSerial{$i})
```
The query command is simply written into the code. Compare to putting the query in a separate method:
```
// QueryMethod (longint)
QUERY([Table_1];[Table_1]SerialNumber_indx=$1)

// myMethod
QueryMethod($aSerial{$i})
```
Here the parameter is passed to the QueryMethod as a longint param. 
Finally, what if the query params are passed in a c-object (which is essentially a JSON):
```
// QueryMethod2 (c-obj)
$number:=OB GET($1;"value";Is longint)
QUERY([Table_1];[Table_1]SerialNumber_indx=$number)

// myMethod
OB SET($obj;"value";$aSerial{$i})
QueryMethod($obj)
```

Why would I even want to move a simple query like this into a separate method in the first place? Centralizing queries is usually desireable in complex databases. It allows control and optimization, quantifying system time spent doing querys and more. So centralizing looks useful but why would I want to use a c-object instead of native 4D paramters? 

The first reason is feature creep. You write a query method for the needs of today and next month or next year a new need arises. Typically you add another parameter to the method. But somethimes this isn't really hepfull like when the new paramters completely obviate the initial params. Using a c-object to pass parameters is more flexible and so more adaptive. 

Second, if you need to interact with other systems, especially web pages or APIs, it's likely that system will be attempting to communicate with your system using JSON. C-objects are the internal 4D representation fo JSON so that external data may be passed to your query method relatively un modified. 

This database illustrates the differences in these approaches. 

##Results
###Use indexes for critical fields. 
The time difference is huge. 
###Compiled is faster. 
Especially for methods with multiple parameters. 
###When compiled the difference between a single param naked query and a method is not significant. 
	
###The avg difference in a single param query method using a 4D param and usa a c-obj for params is about .0004 ms per query. Small but statistically significant. 
While statistically significant I'm not sure it would be noticeable in the real world. 
##The difference in 2 param queries using 4D params and a c-obj is not significant.
##The difference in 4 param queries using 4D params and a c-obj is significant but small at about .0017 ms. 
Again a tiny difference. 

##The take away
