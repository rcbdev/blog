---
title:  "Custom HTML Helpers in ASP.NET MVC"
date:   2013-01-31
categories: MVC
---

HTML helpers are a really useful feature in ASP.NET MVC. They allow for commonly used sections of mark-up to be generated and reused, helping you with generating items like inputs, forms, e.t.c. By default, there are a reasonable range of helpers built in to MVC, however at times it’s useful to be able to generate your own. If, for example, you find yourself making a large number of navigation links with the same class, and logic to check if you are currently on that page, why not move all this into a helper?

Personally I prefer to keep all my helpers within a folder in the MVC project, however it’s possible to use helpers from class libraries and external dlls if desired. All that is required to use an HTML helper is the namespace the helper is contained in.

In order to create your own helpers, you must first create a static class for your helpers. Helpers are essentially all extension methods on the HtmlHelper class, so you must create a static class with all your helper methods as static methods under this class. Below is an example of a very basic html helper that just outputs the string “Hello, World!” within a span.

{% highlight csharp %}
using System.Web.Mvc;

namespace Custom.Helpers
{
    public static class CustomHelper
    {
        public static MvcHtmlString HelloWorld(this HtmlHelper html)
        {
            var span = new TagBuilder("span");
            span.SetInnerText("Hello, World!");

            return MvcHtmlString.Create(span.ToString());
        }
    }
}
{% endhighlight %}

This method will then be accessible from your views via  `@Html.HelloWorld()`. If we tried to use this straight away, however, we will get a runtime error. This is because the view engine will not be able to locate the custom helper. In order to allow the view engine to find our helper, we need to register the namespace in the web.config. This is done by updating the namspaces tag in the below area, adding the tag  `<add namespace="Custom.Helpers">`.

{% highlight xml %}
<system.web>
    <httpRuntime targetFramework="4.5" />
    <compilation debug="true" targetFramework="4.5" />
    <pages>
        <namespaces>
            <add namespace="System.Web.Helpers" />
            <add namespace="System.Web.Mvc" />
            <add namespace="System.Web.Mvc.Ajax" />
            <add namespace="System.Web.Mvc.Html" />
            <add namespace="System.Web.Routing" />
            <add namespace="System.Web.WebPages" />
        </namespaces>
    </pages>
</system.web>
{% endhighlight %}

Obviously this is only a very basic helper, which doesn’t perform a useful action. However, they can be useful in any situation when you have repeatable sections of mark-up within your website that you want to simplify the production of. They also help to move more of the logic (for example the “which page am I on?” logic for navigation links) out of your view.
