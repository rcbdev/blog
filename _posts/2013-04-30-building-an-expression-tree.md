---
title:  "Building an Expression Tree"
date:   2013-04-30
categories: Expression
description: A look into expressions in .NET and how to build an expression tree for use in LINQ queries.
---

Expressions are key to a few areas of the .Net framework, and extensions to the framework. IQueryables, and frameworks that use them (like Entity Framework), make extensive use of them in order to convert your LINQ queries into another language (e.g. converting a .Where() into a SQL WHERE clause). MVC’s HTML helpers also use Expressions in order to understand the properties you want to display labels, inputs, e.t.c. for, and pull out key information about them (like validation, display names).

Expressions are used for the above purposes as they allow for the code to be inspected at run time. They are essentially functions that have not yet been compiled, and so they can be inspected and understood by code at runtime. This means you can pull method calls, properties, types, and many other details out of the function at runtime, and use them to carry out complex processes like converting LINQ queries to SQL queries.

There’s two main ways we can create Expressions in our code. The first way is very simple, and most people will have already done this. You can create an Expression from a lambda by just implicitly or explicitly converting it to a type of Expression<Func<T1,T2,…>>. This is the main method people use for creating expressions, and is commonly used whenever the desired Expression is known during coding. An example of this technique is below.

{% highlight csharp %}
Expression<Func<string, bool>> expression = s => s.Contains("a") && s.Length > 5;
{% endhighlight %}

Lambdas will be suitable for the majority of use cases you come across. However, there are times when you need to do something more complex. If, for example, you needed to take a user input where they can select any property to filter a set of data on, it would not make sense to create a lambda expression with every possible property they could select, and the a switch to choose which one to use. Instead, you can use the Expression API to build an Expression Tree at runtime. In order to explain how to do this, I will recreate the above lambda expression using the Expression API.

{% highlight csharp %}
// Create the parameter "s", that we pass into the lambda (s => ...)
// The first parameter is the type, the second is the name
var parameter = Expression.Parameter(typeof(string), "s");

// Create the constant "a", that we use in the Contains("a") call
// The first parameter is the value, the second is the type
var stringConstant = Expression.Constant("a", typeof(string));

// Create the constant "5", that we use in the Length > 5 check
// The first parameter is the value, the second is the type
var intConstant = Expression.Constant(5, typeof(int));

// Create the call of s.Contains("a")
// The first parameter is the object to call a method on
// The second is the name of the method to call
// The third is a type array (used for generic methods)
// The other parameters are the objects to pass into the method being called
var contains = Expression.Call(parameter, "Contains", Type.EmptyTypes, stringConstant);

// Get the property "Length" on our parameter
// The first parameter is the object to get the property on
// The second is the name of the property
var lengthProperty = Expression.Property(parameter, "Length");

// Create the greater than comparison (s.Length > 5)
// The parameters are left and right hand side of comparison
var greaterThan = Expression.GreaterThan(lengthProperty, intConstant);

// Create the overall body of the expression (s.Contains("a") && s.Length >5)
// The parameters are left and right of the and
var andExpression = Expression.And(contains, greaterThan);

// Finally, create the lambda (s => s.Contains("a") && s.Length > 5)
// The first parameter is the expression body, the second is the parameter
var expression = Expression.Lambda<Func<string, bool>>(andExpression, new []{ parameter });
{% endhighlight %}

It’s quite clear to see that this process of building up the Expression manually via the Expression API is a lot longer and harder than using a lambda, and so it shouldn’t be used for creating Expressions that are known during coding, however it’s extremely useful and powerful if you need to allow any sort of use input into a query you are generating, and want the user defined queries to be efficient and filter down to whatever your underlying data store is.
