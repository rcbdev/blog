---
layout: post
title:  "Filtering with Reflection in .NET - Extension Method"
date:   2013-03-10
categories: Reflection
---

I’ve written a couple of posts already on the generic data set filterer, the initial introductory post ([http://rcbdev.co.uk/filtering-reflection-net-1](http://rcbdev.co.uk/filtering-reflection-net-1)) and details on how to add excluded properties and display names to the property finding method ([http://rcbdev.co.uk/filtering-reflection-net-2](http://rcbdev.co.uk/filtering-reflection-net-2)). This post will show you how easy it is to now convert the filter method into an extension method. Currently, the way we call the filterer is not very natural, and the filter method feels like a third party citizen. By converting the method into an extension method, we can promote this method so that it appears as a first class method. The code snippet below demonstrates the difference.

{% highlight csharp %}
// Normal method
var filtered = DataSetFilterer.Filter(myDataSet, myFilters);
// Extension method
var filteredNew = myDataSet.Filter(myFilters);
{% endhighlight %}

As you can see from the above snippet, the extension method is much shorter and easier to read. It also now looks like it’s meant to be there, rather than something we’ve stuck onto the side of application.

Making an extension method is actually really simple. Firstly, we need to make sure both our class and method are static so that the method can be called without a new instance of the class being instantiated. Then, we just add the “this” keyword before the first parameter in the method. The “this” keyword is the bit of magic that creates our extension method. When we add the “this” keyword before the first parameter in the method, it is telling the compiler that we want to be able to call the method on an object of that type. For example  Doubled(this int number)  would create an extension method on an integer, so that I could call  myInt.Doubled()  and the compiler will call my method, passing  myInt  in as the first parameter.

Below is the declaration of our data set filterer modified to act as an extension method.
	
{% highlight csharp %}
public static IEnumerable<T> Filter<T>(this IEnumerable<T> dataSet, IEnumerable<FilterValue> filters)
{% endhighlight %}
 
This is all that is necessary in order to promote our method into a first class citizen, and allow it to be called directly from any IEnumerable<T>.