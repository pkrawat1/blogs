---
layout: post
title: "Tussle of the State Machines"
date: 2014-11-15 14:30:44 +0530
comments: true
sharing: true
footer: true
categories: [Ruby, Rails]
---

There's no shortage of Ruby state machine libraries, but when we needed to implement a formal state machine we don't find one which met all of our requirements.

I had a same problem in my project.What i needed was a polymorphic class that could have multiple number of state machines in it.
Depending on the the relation the appropriate state machine should be used.

### What i Wanted

```ruby call.rb
class Call
  include Mongoid::Document
  field :scheduled_at,          type: DateTime
  field :is_existing_customer,  type: Boolean
  field :note
  
  belongs_to :callable, polymorphic: true
  case callable_type
    when 'Car'
      state_machine :state, :initial => :fresh, namespace: 'car' do
        event :schedule do
          transition [:fresh, :schedule] => :scheduled
        end
        .....
        .....
      end
    when 'Personal'
       state_machine :state, :initial => :fresh, namespace: 'Personal' do
        event :schedule do
          transition :fresh => :scheduled
        end
        .....
        .....
      end
     when 'any other'
      .....
  end
end
```
# The Main Problem
Since AASM State Machine does not supports multiple state machine in a single class. So i tried to achive it through state\_machine gem with namespaces
BUT AGAIN we can not have same states under namespaced state machine in a single class.

# How to do it?
My basic requirement was to have a state machine that should be easily composable with other Ruby objects.So what i need to do was to define a state machine as a separate class and selectively apply itto our Rails models.
since Mongodb supports embeded obects. I could use it to store states in it.

## The Solution
We wanted a state machine that could be easily integrated with other Ruby objects. So we decided to define a state machine as a separate class and selectively apply it to our Rails models. We were using MongoDB, so we embedded these objects.

```ruby car_state_machine.rb
class CarStateMachine
  include Mongoid::Document
 
  field :state
  embedded_in :call
 
  # no need for name space and we can use AASM directly
  state_machine :state, :initial => :fresh do
    #states: fresh, scheduled, lead, succeed
    event :schedule do
      transition [:fresh, :schedule] => :scheduled
    end
    #...
    #...
  end
end
```
```ruby personal_state_machine.rb
class PersonalStateMachine
  include Mongoid::Document
  include AASM
 
  field :state
  embedded_in :call
 
  #states: hello, meet, bye
  state_machine :state, :initial => :hello do
    event :wow do
      transition :hello => :meet
    end
    #...
    #...
  end
end
```
###My Call class now
So to call access these embedded objects i defined a method call_state that returns the embedded on the basis of the callable\_type of Call Class.
```ruby call.rb
class Call
  include Mongoid::Document
 
  field :scheduled_at, type: DateTime
  field :is_existing_customer, type: Boolean
  field :note
  field :callable_type
  
  embeds_one :car_state_machine
  embeds_one :personal_state_machine
  
  # Method to access state machine
  def call_state
    case self.callable_type
    when 'Car'
      self.car_state_machine || self.build_car_state_machine
    when 'Personal'
      self.personal_state_machine || self.build_personal_state_machine
    end
  end
end

# Example
call = Call.first.callable_type # => "Car"
call.call_state.state # => 'fresh'
call.call_state.schedule!
call.call_state.state # => 'scheduled'
 
#####
 
call = Call.last.callable_type # => "Personal"
call.call_state.state # => 'hello'
call.call_state.wow!
call.call_state.state # => 'meet'
```