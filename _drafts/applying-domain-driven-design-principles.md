---
layout: post
title: "Applying Domain Driven Design Principles"
meta: post
photo: kandinsky.jpg
---

![](/images/kandinsky.jpg)

[Domain Driven Design][ddd] (DDD) is a methodology I've heard about more and more over the last year, from technical book clubs to interviews. It focuses on how you design your application, including the language and naming used within it. Namely, everything should match the business domain it is serving, a so-called ["ubiquitous language"][ubiq].<!--more-->

The [book][ddd_book] that first introduced this concept is quite long, and goes into very specific detail on the concepts and how to implement this method, including it's own specific patterns, such as entities, values, repositories, etc. [Explorable Places'][ep] domain logic started off simple years ago, but over time, the complexity grew. Models got fat, controllers got fat, and we had business logic in a few views. It was bad and it needed to be wrangled. However, most of the app remained simple CRUD actions.

So while I don't think we needed to refactor the entire app to implement the very specific DDD methodology everywhere, I think there were definitely some key learnings we could take away and apply. Specifically, we should encapsulate our business logic around booking separately into small testable units of 'action' (what DDD would call [_domain events_][domain_event]), and we should name everything according to what and how the user is doing that action (our own _ubiquitous language_).

## Existing Tooling

There are plenty of tools that can help achieve this. Previously at eBay, we used [Trooba][trooba] in Node to create streamable pipelines with handlers. In Ruby, there is the popular [Interactor][interactor], as well as [Wisper][wisper] (which is an event pub/sub based solution), and [Pathway][pathway].

### Pub/Sub

Wisper was particularly interesting because DDD does focus on domain events, and the app reacting to an event, which could represent a user's action. I like this approach, but a couple things struck me as potential issues.

- Asynchronous events: It would be nice to have these events be asynchronous, but let's say one event completes, and triggers another event asynchronously. That second event could fail, and depending on our business logic, may need to rollback the changes of the first handler. This would be very hard to achieve, which would leave us to use the default syncronous handling.

- Tangled mess of handlers: If we are not careful, these events and handlers could expand quickly to become a tangled mess. This also makes it difficult to conceptualize just what happens when a single event is triggered, which could lead to unintended consequences when adding or editing new events.

### Interactors

An "interactor" is basically a reimplementation of the Command Design Pattern, but specifically intended to encapsulate and manage business logic. Each interactor handles one small piece of logic, such as "send email" or "save record". These are small, reusable, and easily testable, but rarely do we need to do something so small when a user performs some action in our app. That's where the real power of this gem shines through. We can group multiple interactors together with an "organizer". Each interactor has a shared context and can fail the whole chain if need be. At the end, a result is returned where you can store the success or failure of the action. Conceptually, a single interactor or organizer could represent a domain event in this case.

I like this solution because we get the same benefit of something like a pub/sub solution, but we can see with just a glance at the code all that happens with a specific action and the order that they will happen in. Below I'll go over a small example that we had that I think illustrates this well.

## Example: Confirm a booking

At Explorable Places, we have bookings for learning experiences. Once a booking is made, the learning institution must confirm or decline it. Previously, we relied on a mix of methods defined directly on the booking model along with ActiveRecord callbacks. Beyond that, some of the logic was also executed via delayed jobs, but that's essentially how it would "work". This resulted in a massive pile of spaghetti code that was difficult to maintain and reason about.

Using interactors, we could encapsulate all the logic of a user's action of confirming a booking to a single file. It is highly readable, and we can see the steps that are taken at each stage. If there is a failure, we can catch it and roll back any database changes. Of course, we only fail in such cases where we would want to roll back.

{% highlight ruby %}
class ConfirmBooking
  include Interactor::Organizer

  organize UpdateBookingStatus, CaptureCardAuthorization, SendMessages, SyncCalendars

  around do |organizer|
    ActiveRecord::Base.transaction do
      begin
        organizer.call
      rescue Interactor::Failure
        raise ActiveRecord::Rollback
      end
    end
  end

  def call
    context.status_event = :confirm
    context.message_times = [:confirmation_complete]
    super
  end
end
{% endhighlight %}

I want to point out the language used also. Each interactor defined uses the natural language we use when talking about the product. We update the booking status to confirmed. We capture the credit card authorization, send messages to users, and sync calendars with updated events. We can clearly see what happens whenever we confirm any booking. Here is an example of one of those interactors:

{% highlight ruby %}
class SendMessages
  include Interactor

  def call
    booking = context.booking
    message_times = context.message_times

    booking.messages.for_times(message_times).each do |message|
      UserMailer.send_message(booking.id, message.id).deliver_later
    end
  end
end
{% endhighlight %}

This interactor is responsible for one thing and one thing only, sending an email for each configured message associated with that booking.

And in our controller, we now have a very succinct action method:

{% highlight ruby %}
def confirm
  booking = Booking.find(booking_params[:id])

  result = ConfirmBooking.call({ booking: booking })
  @booking = result.booking

  respond_to do |format|
    if result.success?
      format.html { redirect_back fallback_location: root_path, notice: 'Booking was successfully confirmed.' }
      format.json { render status: 200, json: @booking }
    else
      format.html { redirect_back fallback_location: root_path, alert: 'There was an error confirming your booking.' }
      format.json { render status: 500, json: @booking.errors }
    end
  end
end
{% endhighlight %}

In our other organizers for creating a booking, canceling, declining or rescheduling, we reuse nearly all the same interactors. While we haven't followed the DDD methodology to a tee, we've successfully encapsulated our domain logic, used the common language that our domain experts and users would use, and made it the primary focus of our app. By designing everything around that, we made managing our complex business logic easier, clearer, and more reliable.

[ddd]:https://en.wikipedia.org/wiki/Domain-driven_design
[ddd_book]: https://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software-ebook/dp/B00794TAUG/ref=sr_1_3?crid=1CTX2TZEYWY3E&dchild=1&keywords=domain+driven+design&qid=1590456005&s=books&sprefix=domain+dri%2Caps%2C152&sr=1-3
[ep]: https://www.explorableplaces.com
[ubiq]: https://en.wikipedia.org/wiki/Domain-driven_design#concepts
[domain_event]: https://en.wikipedia.org/wiki/Domain-driven_design#Building_blocks
[trooba]: https://github.com/trooba/trooba
[wisper]: https://github.com/krisleech/wisper
[pathway]: https://github.com/pabloh/pathway
[interactor]: https://github.com/collectiveidea/interactor
