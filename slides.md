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


!SLIDE middle
# Evan Light
## [@elight](http://twitter.com/elight)
## [evan.light@tripledogdare.net](mailto:evan.light@tripledogdare.net)
!NOTE
* Freelance developer
* I'm always looking for work


!SLIDE top-right
# How I want to feel
}}} images/dalai-lama.jpg
!NOTE
* Me on the ouside
* Sometimes on the inside
* How I want to feel
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


!SLIDE bottom-left
# Let's talk about hate
}}} images/jarjar.jpg::mangaholix::::http://mangaholix.deviantart.com/art/Kriss-HATES-Jar-Jar-Binks-163018356


!SLIDE bottom-right
# Writing presentations
}}} images/writing-presentations.jpg


!SLIDE
!NOTES
* Blank page syndrome


!SLIDE
# Outlining a.k.a TDD
!SLIDE bottom-left
}}} images/mil.jpg


!SLIDE
# Can't mind her own business!


!SLIDE
# A typical User class

``` ruby
class User < ActiveRecord::Base
  # Elided for your sanity
  # Yes, this is real code
  # Yes, I helped write it...

  include User::Associations
  include User::Validations
  include User::Search
  include User::DefaultSettings
  include User::DefaultPrivacies
  include User::Invitations

  # even more fucking includes ...
end
```

!SLIDE
# DCI to the rescue!

``` ruby
class SessionsController < ApplicationController
  def create
    # ...
    user = find_or_create_from_oauth_data(oauth_data)
    # ...
  end

  def find_or_create_from_oauth_data(oauth_data)
    # ...
    User.new.tap do |user|
      user.extend NewUserProvisioner
      user.provision_with oauth_data
      user.save!
    end
  end
end
```

##### Lifted and slightly tweaked from [here](https://github.com/rubypair/rubypair/blob/master/app/controllers/sessions_controller.rb)


!SLIDE bottom-left
# Monkey rape
}}} images/monkey-rape.gif


!SLIDE
## Protip
### Be careful what you google
### What is seen cannot be unseen
##### Helpful hint: "[monkey raping](https://www.google.com/search?q=monkey+raping&hl=en&safe=off)"


!SLIDE

# Monkey rape

``` ruby
module Resque
  class Worker
    alias_method :unregister_worker_without_before_hook, :unregister_worker

    def unregister_worker
      run_hook(:before_unregister_worker, self)
      unregister_worker_without_before_hook
    end

    # Unforunately have to override Resque::Worker's +run_hook+ method to call hook on
    # ... (elided to protect the innocent) rather on Resque directly. Any suggestions on
    # how to make this more flexible are more than welcome.

    def run_hook(name, *args)
      # ...
      # elight: 4 hours of my life... gone
    end
  end
end

```


!SLIDE
<img src="images/Bullshit.gif" style="width: 600px; height: 300px;"></img>
``` ruby
# Unforunately have to override Resque::Worker's +run_hook+ method to call hook on
# ... (elided to protect the innocent) rather on Resque directly.
```


!SLIDE top-left

# Dependency injection

}}} images/needle.jpg


!SLIDE

# "Law" of Demeter

!NOTE

* images/duck-rape.jpg


!SLIDE

# Demeter
## You can play with your friends
## You can play with your privates
## You can't play with your friend's privates


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


!SLIDE top-left

# Be a hero in private

}}} images/r2d2-hero.jpg

!NOTE
* Wearing your genius cap
* Do it in isolation
* Document the fuck out of it


!SLIDE bottom-left

# Remember this guy?

}}} images/prima.jpg

!NOTE
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
* Yehuda talked all about this yesterday
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
```

### Learn some

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
```


!SLIDE
# But enough code


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


!SLIDE bottom-left
# The Force will be with you, always
}}} images/obiwan.jpg


!SLIDE

# Now do some fucking work
### Thank you
