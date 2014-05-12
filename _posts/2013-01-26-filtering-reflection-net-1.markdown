---
layout: post
title:  "Filtering with Reflection in .NET"
date:   2013-01-26
categories: Reflection
---

Reflection is possibly one of the most powerful tools available in the .net framework. It allows you to dynamically fetch information about types and properties, and their values, at runtime. This can allow you to produce extremely powerful features without having to write a huge amount of code. One example of this is a filtering system, which can be made generic to any type, and provide the entire filtering system without having to manually code for specific property names.

In order to use reflection, first you need to get a Type object referring to the type you are interested in. This can be done one of two ways, either from the class or from an instance of the class. I have shown both these techniques below:

{% highlight csharp %}
// Get the Type object from the class
var type = typeof(MyClass);

// Create a new instance of the class
var myObject = new MyClass();
// Get the Type object from the object
var type = myObject.GetType();
{% endhighlight %}

Once we have a reference to our Type object, the fun can start. The Type object exposes a huge amount of information about the class and what it contains. If you’re interested in finding out about all the methods available to you in the Type object, I recommend checking out the documentation on MSDN ([http://msdn.microsoft.com/en-us/library/system.type.aspx](http://msdn.microsoft.com/en-us/library/system.type.aspx)).

In order to create a filtering system, we are only interested in two of the methods available on the Type object. This are GetProperties and GetProperty. GetProperties provides us with an array of PropertyInfo objects containing details of all the properties available in our type. GetProperty returns a single PropertyInfo object referencing the property whose name matches the string passed in. These methods are both useful to us when creating a filtering system, as GetProperties will allow us to find all the properties available to filter on, whereas GetProperty enables us to get a specific property that the user wishes to filter on.

Now that we know these methods are available to us, we can put them to use. The first thing we want to do is get all the properties available to filter on. This can be done using GetProperties, and looping through each property and extracting its name. I have put this in a foreach loop, rather than using a LINQ Select method as this enables easier expansion in the future.

{% highlight csharp %}
public IEnumerable<string> GetFilterProperties(Type type)
{
    // Get the PropertyInfo collection of all properties in the type
    var properties = type.GetProperties();
    // Loop through the properties and add return their names
    foreach (var property in properties)
    {
        yield return property.Name;
    }
}
{% endhighlight %}

The values returned by this function can then be used to provide the user with a list of properties they can filter a collection by.

The second part to creating a filtering system, is implementing the method to filter a dataset based on some user selected filters. This is where we will use the GetProperty method to get an individual property selected by the user.

Before we get into the main body of the filtering method, I will set up a basic class for a filter value in order to allow us to take in a property name and value pair.

{% highlight csharp %}
public class FilterValue
{
    public string Property { get; set; }
    public string Value { get; set; }
}
{% endhighlight %}

Now we can put together a method for filtering the dataset. This method will take in a collection of objects to be filtered, and a collection of FilterValue objects. It will then return a filtered version of the collection.

{% highlight csharp %}
public IEnumerable<T> Filter<T>(IEnumerable<T> dataSet, IEnumerable<FilterValue> filters)
{
    // Get the type of our objects
    var type = typeof(T);
    // Take a local copy of the data
    IEnumerable<T> filteredDataSet = dataSet.ToArray();
    foreach (var filter in filters)
    {
        // Get the property defined in the filter
        var property = type.GetProperty(filter.Property);
        // Check if the property is null
        if (property == null)
        {
            throw new InvalidOperationException(string.Format("Property {0} does not exist on type {1}.", filter.Property, type.Name));
        }
        // Everything is good, we can now filter
        // We are using a LINQ Where to filter down
        // based on our filter values
        filteredDataSet = filteredDataSet.Where(o =>
        {
            // Get the value for the property on this object
            var value = property.GetValue(o);
            // Check if this value is null
            // if not, check against desired value
            return value != null && value.ToString() == filter.Value;
        });
    }
    // Return our filtered dataset
    return filteredDataSet;
}
{% endhighlight %}

What this method does is take each of the FilterValue objects we pass in and restrict the dataset based on the values provided by getting the property asked for, and then using that to get the value of that property on each object. This may look odd at first. Essentially, the GetValue method on the PropertyInfo object just takes in an instance of the class and returns the value of that property on the instance.

This is currently only a very basic example of filtering a dataset using reflection. It currently only handles an equals comparison, and only supports comparing simple objects that can be converted into a string. There are many ways in which this can be enhanced (for example converting the method into an extension method, or enabling different comparison operators), which I will cover in a future blog post.