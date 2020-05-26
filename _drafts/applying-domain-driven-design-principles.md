---
layout: post
title: "Applying Domain Driven Design Principles"
meta: post
---

Domain Driven Design is a methodology I've heard about more and more over the last year, from technical book clubs to interviews. It focuses on how you design your application, including the language and naming used within it. Namely, everything should match the business domain it is serving. The book that first introduced this concept is quite long, and goes into very specific detail on how to implement this method, including it's own specific patterns, such as entities, values, repositories, etc. Explorable Places' domain logic started off simple years ago, but over time, the complexity grew. Our models got fat, our controllers got fat, we had business logic in a few views, it was bad and it needed to be wrangled. While I don't think we needed to refactor the entire app to implement the very specific Domain Driven Design methodology, I think there are definitely some key learnings we can take away and apply. Specifically, we should encapsulate our business logic separately into small testable units of 'action', and we should name everything according to what and how the user is doing an action.

## Existing Tooling

There are plenty of tools that can help achieve this. Previously at eBay, we used Trooba in Node to create streamable pipelines with handlers. In Ruby, there is the popular Interactor, as well as Wisper (which is an event pub/sub based solution), and Pathway.

### Pub/Sub

Wisper was particularly interesting because DDD focuses also on events, and the app reacting to an event, which could represent a user's action. I like this approach, but a couple things stuck out as potential issues.

- Asynchronous events: It would be nice to have these events be asynchronous, but let's say one event completes, and triggers another event asynchronously. That second event could fail, and depending on our business logic, may need to rollback the changes of the first handler. This would be very hard to achieve, which would leave us to use the default syncronous handling.

- Tangled mess of handlers: If we are not careful, these events and handlers could expand quickly to become a tangled mess. This also makes it difficult to conceptualize just what happens when a single event is triggered, which could lead to unintended consequences when adding or editing new events.

### Interactors

An "interactor" is basically a reimplementation of the Command Design Pattern, but specifically intended to encapsulate and manage business logic. Each interactor handles one small piece of logic, such as "send email" or "save record". Independently, these are small, reusable, and easily testable. But rarely do we need to do something so small when a user performs some action in our app. That's where the real power of this gem shines through. We can group multiple interactors together with an "organizer". Each interactor has a shared context and can fail the while chain if need be. At the end, a result is returned where you can store the success or failure of the action.

I like this solution because we get the same benefit of something like a pub/sub solution, but we can see at a glance all that happens with a specific action and the order that they will happen in. Below I'll go over a small example that we had that I think illustrates the usefullness of it.

## Example: Confirm a booking

Previously, we relied on a mix of methods defined directly on the model and ActiveRecord callbacks. Beyond that, some of them were delayed jobs, but that's how it would "work". Using interactors, we can define a user's action of confirming a booking as such below. It is highly readable, and we can see the steps that are taken at each stage. If there is a failure, we can catch it and roll back any database changes. Of course, we only fail in such cases where we would want to roll back.

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

I want to point out the language used also. Each interactor defined uses the natural language we use when talking about the product. We update the booking status to confirmed. We capture the credit card authorization, send messages to users, and sync calendars with updated events. We can clearly see what happens whenever we confirm any booking.

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

In our other organizers for creating a booking, canceling, declining or rescheduling, we reuse nearly all the same interactors. While we haven't followed the DDD methodology to a tee, we've encapsulated our domain logic and made it the main focus of our app, and designed around it, making managing our complex business logic easier and clearer.
