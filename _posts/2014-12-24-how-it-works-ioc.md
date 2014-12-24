---
title: "How it Works: IoC Containers"
date: 2014-12-24
categories: IoC
---

As developers, we often use powerful frameworks and tools that allow us to do complex operations without a large amount of work. These simplify the job of making large applications, and help us to produce more maintainable and efficient systems. However, when one of these frameworks breaks or goes wrong, it can be a nightmare to debug and fix the issue. This is why it's important to understand how these frameworks we use actually work. One such area is inversion of control (or IoC) containers.

IoC containers are hugely important in modern development and allow for dependency injection throughout your applications. Without the use of an IoC container, dependency injection becomes a lot more complex and less manageable. Despite this, when you have an issue with an IoC container it can be difficult to track down what went wrong. In this post I'll look at how a simple IoC container works internally.

In order to understand how an IoC container works, we first need to understand its purpose and what it does. An IoC container handles the creation of objects in your system, and provides them with any dependencies. For example, if you are creating an MVC website, an IoC container can be used to construct your controllers and pass in any required services.

For this post, I'll be using a simple IoC container I've created as a demo. You can view the full code for this IoC container [here](https://gist.github.com/rcbdev/e8bb66ba8b4638065e51).

{% highlight csharp %}
public static void Register<TInterface, TClass>(bool instantlyCreate = false) where TClass : class, TInterface
{
  lock (SyncLock)
  {
    Registration.Add(typeof (TInterface), typeof (TClass));
    if (instantlyCreate)
    {
      Resolve<TInterface>();
    }
  }
}
{% endhighlight %}

The first thing to look at in the IoC container is registering a type. This allows you to register an interface and its associated implementation (other methods allow registering of a type without an interface). This method is very simple, taking in two generic types (an interface and an implementation) and adding these types to a dictionary. If you wish for the type to be instantly created, it then resolves the type.

Once the registration of types is out of the way, the real work can begin. This is the resolving of a type. When you want to create a new object using an IoC container, you ask it to resolve a type (be it a class or an interface) and it will return an instance of the requested type. This involves the use of reflection in order to construct the type and to determine its dependencies.

{% highlight csharp %}
if (!Rot.ContainsKey(type))
{
  // snip
}
return Rot[type];
{% endhighlight %}

To begin with, we check if we already have an instance of the requested type we can just return. If not, we need to create a new instance. This is where the fun begins.

{% highlight csharp %}
var resolveTo = Registration[type] ?? type;
var constructorInfos = resolveTo.GetTypeInfo().DeclaredConstructors.ToArray();

if (constructorInfos.Length > 1)
{
  throw new Exception(string.Format("Cannot resolve type {0} as it has more than one constructor.", type.Name));
}

var constructor = constructorInfos[0];
var parameterInfos = constructor.GetParameters();

if (parameterInfos.Length == 0)
{
  Rot[type] = constructor.Invoke(EmptyArgs);
}
else
{
  var parameters = new object[parameterInfos.Length];
  foreach (var parameterInfo in parameterInfos)
  {
    parameters[parameterInfo.Position] = Resolve(parameterInfo.ParameterType);
  }
  Rot[type] = constructor.Invoke(parameters);
}
{% endhighlight %}

In order to create a new instance of the requested type, we first check if there is a implementation registered. If an interface has been requested, we need to find the registered implementation of it in order to construct it. If there is no implementation registered for an interface (or multiple), the IoC container will not know how to create the object, which will lead to the construction failing.

Once we know what type we need to create, we use a bit of reflection in order to find the constructors available. If there is more than one constructor, we cannot determine how the object should be constructed and so the creation fails. This is one way in which the use of an IoC container can fail. If you define multiple constructors on an object, the IoC container does not know which constructor to use. Some more complex IoC containers (such as Unity) allow you to define how you wish for an object to be created, and this can allow for multiple constructors wihtout this issue.

Once we have determined how our object should be created, we use reflection again. This time we look for the parameters the constructor takes in. For each of the required parameters, we just go back into the resolve method asking for the parameter's type. If any of the parameters cannot be constructed, for example its type has not been registered, the construction of the object will fail.

IoC containers such as Unity do a lot more complex operations than this (that's why we use them), but this has shown the basic operations an IoC container has to perform. A lot of the more complex functionality simply builds upon this. If you are interested in findind out more about how they work, I'd recommended looking into open source projects such as [TinyIoC](https://github.com/grumpydev/TinyIoC) or [Unity](https://unity.codeplex.com/).
