# Frustration Driven Development

!SLIDE bottom-left
# Frustration Driven Development
}}} images/FFFFFUUUUUUU-.png
!NOTE
* FDD is about the fail I have seen and participated in
* It's about you and how you can avoid it
* But I'm going to relate it all to my personal life...
* Because it made writing the slides more fun...
* Because I thought you guys would get a laugh out of it...
* And, because, hell, it felt pretty cathartic for me!


!SLIDE
}}} images/mature.jpg


!SLIDE middle
# Evan Light
## **<u>Freelance Code Janitor</u>**
## [@elight](http://twitter.com/elight)
## [evan.light@tripledogdare.net](mailto:evan.light@tripledogdare.net)
!NOTE
* Freelance developer
* I'm always looking for work


!SLIDE middle
}}} images/stuff_i_do.png


!SLIDE
# Real Talk
## (or "a recipe to put me out of business")
!NOTE
# Not saying other talks aren't real
# I'm going to drop some truth bombs


!SLIDE top-right
# How I want to feel
}}} images/dalai-lama.jpg
!NOTE
* Happy
* Gregarious
* Relaxed
* Forgiving


!SLIDE bottom-left
# How I used to feel
}}} images/vader.jpg
!NOTE
* This is also me
* Angry
* Bitter


!SLIDE bottom-left
# How did I get this way?
}}} images/bushy.jpg
!NOTE


!SLIDE top-left
# Your tax dollars at work
}}} images/usgovt.png
!NOTE
* ~12 years
* Piss away tax dollars


!SLIDE top-right
# Easy come, easy go!
}}} images/bubble.png
!NOTE
* 2x failed startup employee
* '99 and '08


!SLIDE bottom-right
# Workin for the man... me!
}}} images/bizhell.jpg


!SLIDE top-left
# I've been doing this a while...
}}} images/atari400.jpg
!NOTES
# 18 years or so


!SLIDE top-right
# ... and I've seen a lot of shit
}}} images/crap.png


!SLIDE bottom-left
# It pisses me off
}}} images/angry.jpg


!SLIDE
}}} images/fear-anger-hate.jpg


!SLIDE bottom-right
# Let's talk about hate
}}} images/jarjar.jpg::mangaholix::::http://mangaholix.deviantart.com/art/Kriss-HATES-Jar-Jar-Binks-163018356


!SLIDE
# We _love_ to talk about TDD
!NOTE
* Straw poll


!SLIDE
# Most Rubyists don't write tests!
### FFFFUUUUUUU
!NOTE
* I've straw-polled many confs. Maybe half of attendees claim to TDD
* I don't believe conferences are cross-sections: above average


!SLIDE
# [TATFT](http://smartic.us/2008/08/15/tatft-i-feel-a-revolution-coming-on/)

!SLIDE
# Callbacks
### FFFFUUUUUUU

!SLIDE
# Callback example
#### &nbsp;
``` ruby
class User
  after_save do
    if name_changed?
      notify_listeners(:name, self)
    end
  end
  
  def self.register_listener(listener)
     # ...
  end
end
```

!SLIDE
# Callback example (cont'd)
#### &nbsp;
``` ruby
class ClientProxy
    def initialize
     User.register_listener(self)
   end
		  
   def receive_notification(field_changed, source)
     push_change_to_client(field_changed, source)
   end
end
```


!SLIDE
# AR callbacks are Observers
#### &nbsp;
> The \[Observer pattern\] is useful mostly for dynamic relationships between objects.

#### - [C2 wiki](http://c2.com/cgi/wiki?ObserverPattern)

!SLIDE
# Tightly coupled example
#### &nbsp;
``` ruby
class ClientProxy
  def push_change_to_client(message, source)
    # ...
  end
end
		 
class UserController
  def update
    User.transaction do
      user.name = params[:name]
      user.save
      client_proxy.push_change_to_client(:name, user)
    end
  end
end
```
###### More discussion on this: [Use Rails until it hurts](http://evan.tiggerpalace.com/articles/2012/11/21/use-rails-until-it-hurts/)

!SLIDE
# The Long & Poorly-Named Method
##### (or, in the words of Samuel L. Jackson, <u>"it's Red, Green, **Refactor** mother%&@#er!"</u>)
### FFFFUUUUUUU
!NOTE


!SLIDE
# Most Rubyists don't refactor!
### FFFFUUUUUUU

!SLIDE
# Don't try to read this code
#### &nbsp;
```ruby
class User
  def rectify
    if self.payments.find { |p| p.status == "pending" }
      self.payments.select { |p| p.status == "pending" } do |p|
        p.apply_to!(self)
      end
    end
    if self.bills.find { |b| b.due_date < Time.now }
      self.bills.select { |b| b.due_date < Time.now }.each do |b|
        b.submit_to_collections!
      end
    end
    if self.bills.find { |b| b.payment_date.nil? }
      self.bills.select { |b| b.payment_date.nil? }.each do |b|
        b.send_to self
      end
    end
  end
end
```
!NOTE
* 3 different things going on here
* try extracting blocks of code into descriptively named methods


!SLIDE
# We're going to operate!


!SLIDE
# Put it on life support


!SLIDE left
## Stub out dependencies in test setup
## Test all branches
## Test all boundaries
## Watch tests pass
###### Great book on the topic: [Working with Legacy Code](http://www.amazon.com/Working-Effectively-Legacy-Michael-Feathers/dp/0131177052_)


!SLIDE
# Now, operate!
!NOTES
* Extract method


!SLIDE
# [Extract method](http://www.refactoring.com/catalog/extractMethod.html)
#### &nbsp;
```ruby
class User
  def rectify
    process_pending_payments
    notify_collections_about_overdue_bills
    send_me_my_remaining_bills
  end
end
```
!NOTE
* Seems like something perhaps better suited to a separate class entirely


!SLIDE
# [Extract method](http://www.refactoring.com/catalog/extractMethod.html) (cont'd)
#### &nbsp;
```ruby
class User
  def process_pending_payments
    #...
  end

  def notify_collections_about_overdue_bills
    if self.bills.find { |b| b.due_date < Time.now }
      self.bills.select { |b| b.due_date < Time.now }.each do |b|
        b.submit_to_collections!
      end
    end
  end

  def send_me_my_remaining_bills
    # ...
  end
end
```


!SLIDE
# [Rename method](http://www.refactoring.com/catalog/renameMethod.html)
#### &nbsp;
```ruby
class User
  # previously just 'rectify'
  def rectify_account
    process_pending_payments
    notify_collections_about_overdue_bills
    send_me_my_remaining_bills
  end
end
```


!SLIDE
# Let's focus here
#### &nbsp;
```ruby
  def notify_collections_about_overdue_bills
    if self.bills.find { |b| b.due_date < Time.now }
      self.bills.select { |b| b.due_date < Time.now }.each do |b|
        b.submit_to_collections!
      end
    end
  end
```
!NOTE
# "notify collections" sounds like sending a message
# notify = verb
# collections = object, rcvr
# object should rcv msg


!SLIDE
# [Extract class](http://www.refactoring.com/catalog/extractClass.html)
#### &nbsp;
```ruby
class CollectionClaim
  def self.file_against(bill)
    self.new(bill)
  end

  def initialize
    # biz logic would start here
  end
end
```

!SLIDE
# Then we have...
#### &nbsp;
```ruby
class User
  def notify_collections_about_overdue_bills
    if self.bills.find { |b| b.due_date < Time.now }
      self.bills.select { |b| b.due_date < Time.now }.each do |b|
        CollectionsClaim.file_against bill
      end
    end
  end
end
```


!SLIDE
# A couple more [Extract method](http://www.refactoring.com/catalog/extractMethod.html)s later
#### &nbsp;
```ruby
class User
  def notify_collections_about_overdue_bills
    past_due_bills.each do |bill|
      CollectionsClaim.file_against bill
    end
  end

  private

  def past_due_bills
    self.bills.select { |bill| b.past_due? }
  end
end

class Bill
  def past_due?
    due_date < Time.now
  end
end
```


!SLIDE middle
# Should User know how 
# to rectify his own account
#### &nbsp;
## Hint: **rectify** & **account**
## Think about it (later ;-) )


!SLIDE
# The typical "User" class
### FFFFUUUUUUU
!NOTE
* Classes tend to suffer from this before methods
* Cluttered full of all manner of behavior user-related


!SLIDE bottom-left
}}} images/mil.jpg
!NOTE
* Doesn't mind her own business
* User class can't mind its own business
* It has to know about everything


!SLIDE
# Move code into modules?
#### &nbsp;
```ruby
class User
  # Elided for your sanity
  # Yes, this is (mostly) real code
  # Yes, I helped write it a year or so ago
  # Yes, I hate my past self at times

  include User::Associations
  include User::Validations
  include User::Search
  include User::DefaultSettings
  include User::DefaultPrivacies
  include User::Permissions
  include User::Invitations
  include User::OAuthProvisioning

  # even more includes.... HALP!
end
```
###### See [Mixins, a Refactoring Anti-Pattern](http://blog.steveklabnik.com/posts/2012-05-07-mixins--a-refactoring-anti-pattern)
!NOTES
* Steve Klabnik's blog post
* Now they're just in several files
* Still too many responsibilities

!SLIDE
}}} images/picard-facepalm.gif


!SLIDE
# Get a lackey!


!SLIDE ruby
``` ruby
require 'delegate'
class UserWithNormalPermissions < SimpleDelegator
  ALLOWED_METHODS = %w[do_something]

  def method_missing(args={})
    unless ALLOWED_METHODS.include?(args[0])
      fail PermissionError, ...
    end
    # Call "super" or else delegation doesn't happen!
    super
  end
end
```
!NOTES
# User isn't mentioned anywhere in this code
# Can be used for anything that responds to ALLOWED_METHODS


!SLIDE
```ruby
def some_controller_action
  user = UserWithNormalPermissions.new(
    User.find(...)
  )

  user.do_something
rescue PermissionError => p
  # ...
end
```
!NOTES
# UserWithNormalPermissions is "plain old Ruby code"
# No dependency on Rails
# Easy to unit test
# "Tell, don't ask"


!SLIDE
# Let's face it:
# Most templates suck
### FFFFUUUUUUU


!SLIDE
```

%div.poor_template
  - if some_condition
    = render_this
  - elsif another_condition
    = render_that
  - else
    = render_those
```


!SLIDE
# You could use Helpers...


!SLIDE
```ruby
module SomeHelper
  def render_whatever_through_a_helper
    if some_nasty_condition
      render_this
    elsif another_nasty_condition
      render_that
    else
      render_those
    end
  end
end
```

```

%div.better_templating
  = render_whatever_through_a_helper
```


!SLIDE
# Helpers are poor objects


!SLIDE
```ruby
module EmployerHelper
  def redirect_to_not_allowed_if_not_employer
  end

  def city_province_country_string(model)
  end

  def welcome_text
  end

  def employer_videos_create_job
  end

  def employer_videos_read_results
  end

  def employer_videos_customize_jobs
  end
end
```
!NOTES
* Modules are good but..
* Helpers are behavior buckets without data... without context
* Much like "FooUtils" back in the java days
* Junkyard or Island of Misfit Toys of your app
* Or, if you prefer, structured programming in an OO world


!SLIDE
# What if we delegate 
# to real objects?


!SLIDE
#Replace this...
#### &nbsp;
```ruby
module SomeHelper
  def render_whatever_through_a_helper
    if some_nasty_condition
      render_this
    elsif another_nasty_condition
      render_that
    else
      render_those
    end
  end
end
```

!SLIDE
#... with this
#### &nbsp;
```ruby
class SomeConditionPresenter
  def render
    #...
  end
end

class AnotherConditionPresenter
  # Same interface as above
end

class DefaultConditionPresenter
  # Get the idea?
end
```


!SLIDE
# We still have to 
# pick a "presenter"
#### &nbsp;
```ruby
module FooHelper
  def conditional_presenter
    klass =
      if some_nasty_condition
        SomeConditionPresenter
      elsif another_nasty_condition
        AnotherConditionPresenter
      else
        DefaultConditionPresenter
      end
    klass.new
  end
end
```


!SLIDE
# Presenters probably should not be your first choice
!NOTES
* Delegation, abstraction really, adds more classes
* Abstraction always has a cost
* Especially useful if a template has multiple conditional states


```

%div.cleaner_templating
  = conditional_presenter.render
```
!NOTES
* Delegate accordingly


!SLIDE
# How do I choose?
!NOTES
* I often default to using Helpers
* I ExtractClass when there are a few related behaviors
* I tend to use Helpers when a template has few external dependencies
* I tend to use Presenters when a template has more external dependencies


!SLIDE
# Want to try using presenters?
## [modest_presenter](http://github.com/elight/modest_presenter)
### Warning: I wrote this ;-)
## [draper](http://github.com/drapergem/draper)
### More powerful
### Similar idea, different philosophy


!SLIDE
# Monkey patching
### FFFFUUUUUUU


!SLIDE
``` ruby
module Resque
  class Worker
    # Unfortunately have to override Resque::Worker's +run_hook+
    # method to call hook on [MySpecialSnowflake gem] rather on
    # Resque directly. Any suggestions on how to make this more
    # flexible are more than welcome.

    def run_hook(name, *args)
      # Me: 4 hours of my life... gone
      # FFFFFFFUUUUUUUUUUUUUUUU

      return unless hook = MySpecialSnowflake.send(name)

      # ...
    end
  end
end
```


!SLIDE
<img src="images/Bullshit.gif" style="width: 600px; height: 300px;"></img>
``` ruby
# Unfortunately have to override Resque::Worker's +run_hook+
# method to call hook on [MySpecialSnowflake gem] rather on
# Resque directly. Any suggestions on how to make this more
# flexible are more than welcome.
```


!SLIDE top-left
# Dependency injection
}}} images/needle.jpg


!SLIDE
# Fork, patch, and pull request!
#### &nbsp;
```ruby
module Resque
  def self.hook_responders
    @hook_responders ||= [self]
    @hook_responders
  end

  def self.register_hook_responder(responder)
    hook_responsers << responder
  end
end
```
!NOTES
# Oh, Resque doesn't let us do that?
# Let's tell Resque that we are an additional dependency
# It's Open Source, right?
# Use your fork until the patch gets integrated


!SLIDE
# There is no monkey patch
#### &nbsp;
```ruby
module Resque
  class Worker
    def run_hook(name, *args)
      hook = Resque.hook_responders.find { |r| r.send(name) }
      return unless hook
      # ...
    end
  end
end
```

!SLIDE
# Breaking Demeter
### FFFFUUUUUUU
!NOTE


!SLIDE
# "Law of Demeter" redux
## You can play with your friends
## You can play with your privates
## You shouldn't play with your friend's privates


!SLIDE
# Style
### FFFFUUUUUUU
```ruby
  def busy_method
    please_dont_write_longs_lines(of: code, that: go, on: and)
    followed_by_more(really: hard, to: read, crap: that, hurts_my: eyes)
    because_i_will_find_you(and: hurt, you: for: writing, this: shit)
  end
```
# Get some


!SLIDE
# Not considering your audience
### FFFFUUUUUUU


!SLIDE top-left
}}} images/nerd-rage.jpeg


!SLIDE bottom-left
}}} images/prima.jpg
!NOTE
* Don't be this guy

!SLIDE top-left
# Be a hero in private
}}} images/r2d2-hero.jpg
!NOTE
* Wearing your genius cap
* Do it in isolation
* Document the fuck out of it
* Be sensitive to your team's skill level
* Learn what your team knows and what they don't
* Write code that most of your team can understand


!SLIDE
# Breaking convention
### FFFFUUUUUUU


!SLIDE bottom-right
}}} images/c3p0-backwards.png


!SLIDE
# Conventions? 
# What conventions?
#### &nbsp;
```ruby
class EXCITINGController < ApplicationController
  def timeline; end
  def tag; end
  def search; end
  def detail; end
  # and on, and on, and on...
end
```

!SLIDE
# Predictable is good!
#### &nbsp;
```ruby
class VeryBoringController < ApplicationController
  def index; end
  def new; end
  def create; end
  def update; end
  def show; end
  def destroy; end
end
```
!NOTE
* Predictable code is good!
* Conventions are GOOD!
* Constraints are GOOD!
* Does not have to be repetitive
* Think "the typical Rails controller"
* More surprises = Greater WTFs/minute


# Vocabulary
### FFFFUUUUUUU


!SLIDE
# Vocabulary
#### &nbsp;
```ruby
class UserTeam
  belongs_to :user
  belongs_to :team
end

class CandidateEmployerJob
end
```
# Get one
!NOTE
* Naming
* Intent revealing
* Put some effort into it
* Joining "User" and "Teams" and call the class "UserTeams"? Buy a fucking thesaurus!


!SLIDE
# Better
#### &nbsp;
```ruby
class TeamMembership
  belongs_to :user
  belongs_to :team
end

class Applicant
end
```


!SLIDE
# But enough code


!SLIDE bottom-right
# Your career at first
}}} images/starwars.jpg
!NOTES
* Everything is great!


!SLIDE bottom-left
# Midichlorians?!?!1!!
}}} images/fulloffuck.png
!NOTES
* Java and J2EE
* Meetings and managers and clients
* Deadlines
* Stress


!SLIDE bottom-left
# What you're left with
}}} images/episode1.jpg


!SLIDE bottom-right
# Stretch out with your *feelings*
}}} images/feelings.jpg
!NOTES
* Listen to your frustration
* Frustration =~ pain/suffering
* Suffering is a teacher
* Expectation out of alignment w/ Reality: which one is right?


!SLIDE bottom-left
# We believe in what we do...
}}} images/orthodox.jpg
!NOTES
* Presentations, like this, talking about ways to work better
* Idealogical purity
* It's important but...


!SLIDE bottom-right
# ... but we often lose sight of [*what matters*](https://www.hdsa.org/donations.html)
}}} images/kim.jpg
!NOTES
* Kim
* 5 regrets of the dying
* Being true to yourself
* Lost 2 years of my life
* Damien Katz
* Jump off that cliff


!SLIDE bottom-right
# Thank you
}}} images/keep-flying.jpg


!SLIDE
# Citations

### <u>[Working with Legacy Code](http://www.amazon.com/Working-Effectively-Legacy-Michael-Feathers/dp/0131177052_)</u>
### <u>[Refactoring](http://www.amazon.com/Refactoring-Improving-Design-Existing-Code/dp/0201485672)</u>
### [Mixins, a Refactoring Anti-Pattern](http://blog.steveklabnik.com/posts/2012-05-07-mixins--a-refactoring-anti-pattern)
### [modest_presenter](http://github.com/elight/modest_presenter)
### [draper](http://github.com/jcasimir/draper)
### [Regrets of the Dying](http://www.inspirationandchai.com/Regrets-of-the-Dying.html)
### [Damien Katz @ RubyFringe](http://www.infoq.com/presentations/katz-couchdb-and-me)
### [Huntington's Disease Society of America](https://www.hdsa.org/donations.html)
