---
layout: post
title:  "Update Without Selecting With Entity Framework"
date:   2013-05-26
categories: Entity Framework
---

When using Entity Framework (and other ORMs), updating an entity normally involves first querying the database to fetch the current entity. Sometimes this is less than ideal (e.g. when performing a bulk update), and it is desirable to be able to perform an update without having to first fetch entities from the database. Imagine a situation when a user selects a large set of items (e.g. orders), and and updates a single property on all the items (e.g. to mark the orders as complete). If we were to first fetch back all the orders from the database, have Entity Framework convert them to objects, update the status to say they are complete, and then update the entity in the database, this can take a large amount of time. What we ideally want to be able to do is the equivalent of what we would do when we write pure SQL, i.e.  `UPDATE [orders] SET [completed] = 1 WHERE [id] = @id`. This doesn’t require us to first find the entire order in the database, we already know its ID, so we just update the single value based on its ID. So how do we do this in Entity Framework?

Producing a simple single value update in Entity Framework is possible, it just requires updating in a slightly different way to how we would usually handle our updates. The way we do this is by using a dummy object, containing only our primary key property, and the properties we want to update. We then attach this dummy object to the Entity Framework context, and mark the updated properties as modified. By doing our update this way, Entity Framework knows the value of our primary key (and so can correctly match it in the database), and will not attempt to update any other value. Below is an example of how to do this in Entity Framework 5 (a very similar technique is possible in Entity Framework 4, but the syntax is different on marking a property as modified).

{% highlight csharp %}
public void MarkAsComplete(int orderId)
{
    using (var ctx = new MyEntities())
    {
        // This line may be needed, depending on your application.
        // It disables the validation, stopping errors from using
        // dummy object below.
        ctx.Configuration.ValidateOnSaveEnabled = false;

        // Create a dummy object, with only the properties we need
        // in our update.
        var order = new Order
            {
                Id = orderId,
                Complete = true
            };

        // Attach the dummy object to the EF context.
        ctx.Orders.Attach(order);

        // Mark the single property as updated. This means EF will
        // ignore all other properties.
        ctx.Entry(order).Property(o => o.Complete).IsModified = true;
    }
}
{% endhighlight %}

By using this technique to do our updates, dependant on the complexity of our entities and the number of navigation properties being fetched, we can increase the speed of updates hugely. This technique of carrying out updates means huge volumes of updates can be carried out almost instantly, and with minimal hits of the database. It isn’t the only technique that should be used for updates in Entity Framework, and working with the complete object is best in single or small volume updates, but in the case of bulk updates where only one or two properties are being changed, the performance gains can be significant due to the reduction in database queries.