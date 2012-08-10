# Frustration Driven Development

!SLIDE bottom-left
# Frustration Driven Development
}}} images/FFFFFUUUUUUU-.png
!NOTE
* FDD is about the fail I have seen and participated in
* Really, it's about you and how you can avoid it
* But I'm going to relate it all to my personal life...
* Because it made writing the slides more fun...
* Because I thought you guys would get a laugh out of it...
* And, because, hell, it felt pretty fucking cathartic for me!


!SLIDE
}}} images/mature.jpg


!SLIDE middle
# Evan Light
## [@elight](http://twitter.com/elight)
## [evan.light@tripledogdare.net](mailto:evan.light@tripledogdare.net)
!NOTE
* Freelance developer
* I'm always looking for work


!SLIDE middle
}}} images/stuff_i_do.png


!SLIDE
# Real Talk


!SLIDE top-right
# How I want to feel
}}} images/dalai-lama.jpg
!NOTE
* Happy
* Relaxed
* Forgiving
* Gregarious


!SLIDE bottom-left
# How I often feel
}}} images/vader.jpg
!NOTE
* This is also me
* Bitter
* Angry


!SLIDE bottom-left
# How did I get this way?
}}} images/bushy.jpg
!NOTE
* While writing this...
* EVAN Y U SO BITTER?


!SLIDE top-left
# Your tax dollars at work
}}} images/usgovt.png
!NOTE
* ~12 years
* Spend money like water


!SLIDE top-right
# Easy come, easy go!
}}} images/bubble.png
!NOTE
* Some time spent in startups...


!SLIDE bottom-right
# Workin for the man... me!
}}} images/bizhell.jpg


!SLIDE top-left
# I've been doing this a while...
}}} images/atari400.jpg


!SLIDE top-right
# ... and I've seen a lot of crap!
}}} images/crap.jpg


!SLIDE bottom-left
# It pisses me off
}}} images/angry.jpg


!SLIDE
}}} images/fear-anger-hate.jpg


!SLIDE top-right
# Let's talk about hate
}}} images/jarjar.jpg::mangaholix::::http://mangaholix.deviantart.com/art/Kriss-HATES-Jar-Jar-Binks-163018356


!SLIDE
# We love to talk about TDD


!SLIDE
# Mosts Rubyists don't TDD!
!NOTE
* Most apps that I inherit: extremely poor to zero test coverage


!SLIDE
# The Long Poorly-Named Method


!SLIDE
```ruby
class User
  def rectify
    if self.payments.find { |p| p.status == "pending" }
      self.payments.select { |p| p.status == "pending" } do |p|
        p.apply_to!(self)
      end
    end
    if self.bills.find { |b| b.due_date > Time.now }
      self.bills.select { |b| b.due_date > Time.now }.each do |b|
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
* Should be at least 3 different methods
* try extracting blocks of code into well named methods


!SLIDE
# Put it on life support
!NOTES
* Write a thorough suite of tests, hitting the various paths
* Isolate the method from its external dependencies
* Stubbing works well here


!SLIDE
# Now, operate!
!NOTES
* Extract method


!SLIDE
# A little better...
```ruby
class User
  def process_open_invoices
    process_pending_payments
    notify_collections_about_overdue_bills
    send_me_my_remaining_bills
  end

  def process_pending_payments
    #...
  end

  def notify_collections_about_overdue_bills
    if self.bills.find { |b| b.due_date > Time.now }
      self.bills.select { |b| b.due_date > Time.now }.each do |b|
        b.submit_to_collections!
      end
    end
  end

  def send_me_my_remaining_bills
    # ...
  end
end

# etc..
```
!NOTE
* Should the User even know how to process his own open invoices?
* Seems like something perhaps better suited to a separate class entirely


!SLIDE
# Let's focus here
```ruby
  def notify_collections_about_overdue_bills
    if self.bills.find { |b| b.due_date > Time.now }
      self.bills.select { |b| b.past_due? }.each do |b|
        b.submit_to_collections!
      end
    end
  end
```
!NOTE
# "notify collections" sounds like sending a message
# what's up with this #find?


!SLIDE
# Extract class
```ruby
class Collections
  def self.file_claim_against(bill)
    # ...
  end
end
```


!SLIDE
# Maybe not perfect but a lot better
```ruby
  def notify_collections_about_overdue_bills
    past_due_bills.each do |bill|
      Collections.file_claim_against bill
    end
  end

  private

  def past_due_bills
    self.bills.select { |bill| b.past_due? }
  end
```


!SLIDE
# Should User know how to process open invoices?


!SLIDE
# The  typical "User" class
!NOTE
* Classes tend to suffer from this before methods
* Cluttered full of all manner of behavior user-related


!SLIDE bottom-left
}}} images/mil.jpg
!NOTE
* Unlike my mother-in-law


!SLIDE
# Move code into modules?
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
##### See [Mixins, a Refactoring Anti-Pattern](http://blog.steveklabnik.com/posts/2012-05-07-mixins--a-refactoring-anti-pattern)

!NOTES
* Steve Klabnik: still just pushing bits around.
* Now they're just in several files
* Still too many responsibilities


!SLIDE
# Get a lackey!


!SLIDE
# Decorator
```ruby
require 'delegate'

class Permissions < SimpleDelegator
  def can_perform?(action_name)
    # ...
  end
end
```
!NOTE
* delegate is in stdlib
* SimpleDelegator is easy to use


!SLIDE
```ruby
def controller_action
  user = User.find(...)
  user = UserWithPermissions.new(user)

  unless user.can_perform?(:controller_action)
    raise PermissionError, "..."
  end
end
```
!NOTE
* In effect, adds more behaviors to the object without further cluttering the User clas


!SLIDE
# Here's another kind


!SLIDE ruby
``` ruby
class Permissions
  def self.for(actor)
    @actor = actor
  end

  def allows?(action_name)
    # ...
  end
end
```
!NOTES
# Depends on an abstract "Actor", not a User
# "Actor" is just a duck type
# User isn't mentioned anywhere in this code


!SLIDE
```ruby
def controller_action
  user = User.find(...)
  permissions = Permissions.for(user)

  unless permissions.can_perform?(:controller_action)
    raise PermissionError, "..."
  end
end
```
!NOTES
# Permission is "plain old Ruby code"
# No dependency on Rails
# Easy to unit test


!SLIDE
# Let's face it:
# Most templates suck


!SLIDE
# How about a templating example?


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
# What if we delegate to real objects?


!SLIDE
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
# We still have to pick a "presenter"


!SLIDE
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
# Delegation should not be your first idea
!NOTES
* Delegation, abstraction really, adds more classes
* Abstraction always has a cost
* Especially useful if a template has multiple conditional states
* Encapsulate each state in a class


```

%div.cleaner_templating
  = conditional_presenter.render
```
!NOTES
* Delegate accordingly


!SLIDE
# How do I choose?
!NOTES
* I often default to using helpers
* I extract class when there are a few related behaviors
* I tend to use helpers when a template has few external dependencies
* I tend to use presenters when a template has more external dependencies


!SLIDE
# Want to try using presenters?
## [modest_presenter](http://github.com/elight/modest_presenter)
### Simple implementation but flexible
## [draper](http://github.com/jcasimir/draper)
### Convention driven usage but complex implementation


!SLIDE
# Monkey patching
``` ruby
module Resque
  class Worker
    # Unforunately have to override Resque::Worker's +run_hook+
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
# Unforunately have to override Resque::Worker's +run_hook+
# method to call hook on [MySpecialSnowflake gem] rather on
# Resque directly. Any suggestions on how to make this more
# flexible are more than welcome.
```


!SLIDE top-left
# Dependency injection
}}} images/needle.jpg


!SLIDE
# Fork, patch, and pull request!
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
# "Law" of Demeter
!NOTE


!SLIDE
# Demeter
## You can play with your friends
## You can play with your privates
## You shouldn't play with your friend's privates


!SLIDE
# Style
```ruby
  def busy_method
    please_dont_write_longs_lines(of: code, that: go, on: and, on: and, on: and)
    followed_by_more(really: hard, to: read, crap: that, just: hurts, my: eyes)
    because_i_will_find_you(and: hurt, you: for: writing, code: that, is: this, hard: to_read)
  end
```
### Get some


!SLIDE top-left
# Insensitive to others feelings
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


!SLIDE bottom-right
# Breaking convention
}}} images/c3p0-backwards.png


!SLIDE
# FAIL
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


!SLIDE
# Vocabulary
```ruby
class UserTeam
  belongs_to :user
  belongs_to :team
end

class CandidateEmployerJob
end
```
### Get one
!NOTE
* Naming
* Intent revealing
* Put some effort into it
* Joining "User" and "Teams" and call the class "UserTeams"? Buy a fucking thesaurus!


!SLIDE
# Better
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


!SLIDE
}}} images/han.jpg


!SLIDE bottom-right
# Your life & career, at first
}}} images/starwars.jpg
!NOTES
* Everything is great!


!SLIDE bottom-left
# Midichlorians?!?!1!!
}}} images/fulloffuck.png
!NOTES
* Ugly code
* Meetings
* Difficult clients


!SLIDE bottom-left
# What you're left with
}}} images/episode1.jpg


!SLIDE bottom-right
# Stretch out with your *feelings*
}}} images/feelings.jpg
!NOTES
* Who do you serve?
* All serve somebody..
* What do you love?
* What makes you want to live?


!SLIDE bottom-left
# We believe in what we do...
}}} images/orthodox.jpg
!NOTES
* Idealogical purity
* Arguing about the "right way"
* It's important but...


!SLIDE bottom-right
# ... but we often lose sight of [*what matters*](https://www.hdsa.org/donations.html)
}}} images/kim.jpg
!NOTES
* 2005
* HD
* Kim's health now
* THIS is what matters


!SLIDE bottom-right
# Thank you
}}} images/keep-flying.jpg
