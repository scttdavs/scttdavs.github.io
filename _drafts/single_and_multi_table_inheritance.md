---
layout: post
title: "Single and Multi Table Inheritance"
meta: post
---

Recently, among the major refactoring of Explorable Places I've been doing, some of the cleanup has centered around better making use of Single Table and Multi Table Inheritance.<!--more--> 

## Single Table Inheritance

Often in object oriented programming, it is beneficial to inherit common logic from a parent or abstract class. In Rails and other frameworks, where model classes are associated with database tables, this can be a little trickly.

With STI though, we can allow inheritance with models while sharing the same table by adding a new column called `type`, where the framework will save the class type on the record. This way, when fetching a record for any subclass, it will query the same table, but return the correct class instance for that record.

STI gives us a few advantages if our use case meets a couple criteria. If our subclasses all use the same data, but require different logic or behavior around that data, then this would be a good solution. We don't want to waste database space with several unused columns between the two that are not shared, and will likely just grow over time.

If we also expect to query across all subclasses at once, this is more easily achieved with STI as we still have just the one parent class to query. In the case of Rails, you will even get an ActiveRecord::Relation that contains the correct model instances of each subclass, even though we queried through the parent class.

### Example with TimeSlots

In a simplified example of how we utilized STI at EP, we have a concept of `TimeSlots` for scheduling bookings. We have learning experiences that are in person, but we now also support virtual learning experiences.

Virtual experiences do not have such strict time requirements, and usually have a range of available times that can work, as opposed to 'physical' or in-person experiences, that depend on the actual space being open.

Below, you can see how we set up our classes to support this. All time slots share the same types of data, but the logic around how that data is treated is different. Virtual time slots have a list of times that are available. Physical time slots have just one. We can grab the time off of a physical time slot, but a virual time slot will not have a single time.

{% highlight ruby %}
class TimeSlot < ApplicationRecord
  validates :times, length: { minimum: 1, message: "are required. Please add at least one." }

  # lots of other common logic
end

class VirtualTimeSlot < TimeSlot  
  def time
  end
end

class PhysicalTimeSlot < TimeSlot
  validates :times, length: { maximum: 1, message: "may only have one." }
  
  def time
    times.first
  end
end
{% endhighlight %}

## Multiple Table Inheritance

When the requirements for various subclasses differ too much, such as having some data in common, but not all, but there is still significant logic shared between them, MTI may be another good fit.

Similar to STI, we can subclass a model class to share logic, but we want to store the data separately, as if they are traditional models. This gives us the advantage of efficiently storing our data, as well as remaining DRY. Since data is stored in separate tables, a new `type` column is no longer required, but there is another problem (at least in Rails). Rails assumes the name of the table from the first class to inherit from `ApplicationRecord`, which would be the parent class. So to solve this, each model is responsible for specifying its own table name.

### Example with Bookings and Holds

Another example of how I refactored EP, was to utilize MTI. Since we offer scheduling as a service, we need two important models to make this work, a `Booking` that a user makes, and a `Hold` that a learning institution may want to place for a user themselves, until the user can then book.

As both of these are meant to reserve a spot, they share much of the same functionality and logic. However, the booking model is vastly more complex, as it is reponsible for far more than is needed to just reserve a spot. Since the data and logic are that different, it makes a good use case for MTI.

Below, you can also see a simplified version of how we solved this.

{% highlight ruby %}
class Blocking < ApplicationRecord
  validates_with BlockingValidator
  # lots of common logic
end

class Booking < Blocking
  self.table_name = "bookings"
  validates_with BookingValidator
end

class Hold < Blocking
  self.table_name = "holds"
  validates_with HoldValidator
end
{% endhighlight %}

You can't see it here, but 90% of the validation logic alone between these two is shared within `BlockingValidator`. Not only do we not repeat ourselves, we ensure that these two classes stay in lock step with each other as they are and will always meant to behave similarly.

## What not to use STI or MTI for

Often, it can be confusing when to use these methods. One common misconception is to use them when you want a model to be able to 'belong' (as in a has_many or has_one association) to multiple class types. The best approach for cases like this is to use the built-in polymorphism supported by most frameworks, including Rails.

