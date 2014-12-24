---
title:  "Filtering with Reflection in .NET - Exclusions and Display Names"
date:   2013-02-09
categories: Reflection
description: Part 2 of a series of posts about creating a generic filter in .NET. This post looks at how to use attributes to provide additional options.
---

In my first post on creating a generic filtering system in .net, I showed a basic example of how to get properties from a class, so that we can create a list of available properties to filter on. This method (shown below) was kept simplistic, and only returns the internal name of a property.

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

In order to provide a more user friendly experience, we are now going to extend this functionality in order to provide two key pieces of functionality, exclusions and display names. Exclusions will allow us to hide properties from the user to remove them from the filtering (e.g. hiding an internal ID property), and display names will enable us to provide the user with a more friendly name for a property (e.g. First Name versus FirstName).

The first step we will take in enhancing this method is to start returning a collection of SelectListItems (a class in MVC). This will allow us to set both the display text and an internally used value. Any class that enables this type of Text/Value pair could be used if you are not working with MVC. Below is our method converted to use the SelectListItem class.

{% highlight csharp %}
public IEnumerable<SelectListItem> GetFilterProperties(Type type)
{
    // Get the PropertyInfo collection of all properties in the type
    var properties = type.GetProperties();
    // Loop through the properties and add return a SelectListItem
    foreach (var property in properties)
    {
        yield return new SelectListItem
        {
            Text = property.Name,
            Value = property.Name
        };
    }
}
{% endhighlight %}

Now that we have this in place, we can add the display name functionality. For the display names, we will use DisplayName attribute on our properties. This attribute allows you to set a string to be used as the display name of property. Below is an example of this attribute being used.

{% highlight csharp %}
public class MyClass
{
    [DisplayName("My Property")]
    public string MyProperty { get; set; }
}
{% endhighlight %}

Now that we know how to set a display name on our property, we need to be able to check if one is defined, and fetch the value if available. This is achieved through the use of the GetCustomAttributes method on the PropertyInfo object. This method takes two parameters, one describing the type of attribute we would like, and the other saying whether to search the inheritance chain. The type of attribute we are looking for here is a DisplayNameAttribute. The code shown has this functionality added in.

{% highlight csharp %}
public IEnumerable<SelectListItem> GetFilterProperties(Type type)
{
    // Get the PropertyInfo collection of all properties in the type
    var properties = type.GetProperties();
    // Loop through the properties and add return a SelectListItem
    foreach (var property in properties)
    {
        // Attempt to fetch a display name
        var dispNameAttr = property.GetCustomAttributes(typeof(DisplayNameAttribute), false)
                                    .Cast<DisplayNameAttribute>()
                                    .SingleOrDefault();
        // Set the display name to either the attribute or the property name
        var dispName = dispNameAttr == null ? property.Name : dispNameAttr.DisplayName;

        yield return new SelectListItem
        {
            Text = dispName,
            Value = property.Name
        };
    }
}
{% endhighlight %}

What we are doing here is attempting to fetch an attribute of type DisplayNameAttribute, and then checking if we were successful. If the variable is null, this means no attribute was found, and so we should just use the property name. If the variable was not null, we can fetch the DisplayName property and use that instead.

We now have the ability to fetch display names for properties, so the only feature remaining is the ability to exclude properties. For this, we will have to create our own attribute that we will then be able to use in a similar fashion to the display name. In order to create your own attribute, all you need to do is create a class that inherits for the Attribute class. Below I have done this for our ExcludeFromFilter attribute, and have also added this attribute to a property on our class.

{% highlight csharp %}
public class ExcludeFromFilterAttribute : Attribute
{
}
{% endhighlight %}

{% highlight csharp %}
public class MyClass
{
    [ExcludeFromFilter]
    public int ID { get; set; }
    [DisplayName("My Property")]
    public string MyProperty { get; set; }
}
{% endhighlight %}

We can leave the ExcludeFromFilterAttribute class empty, as this isn’t any extra information we require for this attribute because it is just marking a property as excluded from the filter.

Now that we have this attribute in place, we can use  this attribute to allow properties to be excluded from the collection of properties allowed for filtering on. This is done by using the GetCustomAttributes method again, this time with the type ExcludeFromFilterAttribute, and checking if any attributes where found. If we found there was an attribute defined, we skip over the property and go on to the next one. This is shown below.

{% highlight csharp %}
public IEnumerable<SelectListItem> GetFilterProperties(Type type)
{
    // Get the PropertyInfo collection of all properties in the type
    var properties = type.GetProperties();
    // Loop through the properties and add return a SelectListItem
    foreach (var property in properties)
    {
        // Check if an ExcludeFromFilterAttribute is defined
        if (property.GetCustomAttributes(typeof(ExcludeFromFilterAttribute), false).Any())
        {
            continue;
        }
        // Attempt to fetch a display name
        var dispNameAttr = property.GetCustomAttributes(typeof(DisplayNameAttribute), false)
                                    .Cast<DisplayNameAttribute>()
                                    .SingleOrDefault();
        // Set the display name to either the attribute or the property name
        var dispName = dispNameAttr == null ? property.Name : dispNameAttr.DisplayName;

        yield return new SelectListItem
        {
            Text = dispName,
            Value = property.Name
        };
    }
}
{% endhighlight %}

These enhancements to the GetFilterProperties method now allow us to exclude properties (like an internal ID) from being made available to filter from, and provide user friendly display names purely by adding simple attributes to our properties.
