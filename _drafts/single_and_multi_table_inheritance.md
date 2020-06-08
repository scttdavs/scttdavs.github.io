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

Below is a simplified example of how we utilized STI. At EP, we have a concept of `TimeSlots` for scheduling bookings. We have learning experiences that are in person, but we now also support virtual learning experiences.

Virtual experiences do not have such strict time requirements, and usually have a range of available times that can work, as opposed to 'physical' or in-person experiences, that depend on the actual space being open.

Below, you can see how we set up our classes to support this. All time slots share the same types of data, but the logic around how that data is treated is different. Virtual time slots have a list of times that are available. Physical time sots have just one. We can grab the time off of a physical time slot, but a virual time slot will not have a single time available.

{% highlight ruby %}
class TimeSlot < ApplicationRecord
  # lots of common logic
end

class VirtualTimeSlot < TimeSlot
  validates :times, length: { minimum: 1, message: "are required. Please add at least one." }
  
  def time
  end
end

class PhysicalTimeSlot < TimeSlot
  validates :times, length: { minimum: 1, message: "are required. Please add at least one." }
  validates :times, length: { maximum: 1, message: "may only have one." }
  
  def time
    times.first
  end
end
{% endhighlight %}

## Multiple Table Inheritance
