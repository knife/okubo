Okubo [![Build Status](https://travis-ci.org/rgravina/okubo.png)](https://travis-ci.org/rgravina/okubo) [![Code Climate](https://codeclimate.com/github/rgravina/okubo.png)](https://codeclimate.com/github/rgravina/okubo)
=====

Okubo is a simple spaced-repetition system which you can associate with Active Record models to be learned
(such as words in a foreign language) and users in your system. Users study these
and as they mark these attempts right or wrong, Okubo will determine when they should be reviewed
next.

Okubo determines the next study time for an item using the [Leitner System](http://en.wikipedia.org/wiki/Leitner_system).
In the future it may support other popular algorithms, such as the Supermemo 2 algorithm.

Installation
------------

Add to Gemfile (will put on rubygems once at 0.1):

```
gem 'okubo', :git => "git://github.com/rgravina/okubo.git"
```

Run:

```
bundle install
rails generate okubo
rake db:migrate
```

Quick Start
-----------

Assume you have an application allowing your users to study words in a foreign language. Using the <code>has_deck</code> method
you can set up a deck of flashcards that the user will study:

```ruby
class Word < ActiveRecord::Base
end

class User < ActiveRecord::Base
  has_deck :words
end

user = User.create!(:name => "Student")
word = Word.create!(:kanji => "日本語", :kana => "にほんご", :translation => "Japanese language")
```

You can add words and record attempts to guess the word as right or wrong. Various methods exist to allow you to access subsets of this collection:

```ruby
# Initally adding a word
user.words << word
user.words.untested #=> [word]

# Guessing a word correctly
user.right_answer_for!(word)
user.words.known #=> [word]

# Guessing a word incorrectly
user.wrong_answer_for!(word)
user.words.failed #=> [word]

# Listing all words
user.words #=> [word]
```

As time passes words need to be reviewed to keep them fresh in memory:

```ruby
# Three days later...
user.words.known #=> []
user.words.expired #=> [word]
```

Guessing a word correcly several times in a row results in the word taking longer to expire, and demonstrates mastery of that word.

```ruby
user.right_answer_for!(word)
# One week later...
user.words.expired #=> [word]
user.right_answer_for!(word)
# Two weeks later...
user.words.expired #=> [word]
user.right_answer_for!(word)
# One month later...
user.words.expired #=> [word]
```

Reviewing
---------

In addition to an <code>expired</code> method, Okubo provides a suggested reviewing sequence for all unknown words in the deck.
Words are randomly chosen from all untested words, failed, and finally expired in order of precedence. 

```ruby
user.words.review #=> [word]
user.right_answer_for!(word)
# ... continuing until all untested, failed, and expired words have been guessed correctly.
user.words.review #=> []
```

Study Sessions
--------------

Okubo allows you to review sets of words in "study sessions". New words added to the deck or words which later expire do not get added to study sessions.

```ruby
user.words.sessions #-> []
session = user.words.start_session! # by default, all words ready for review will be in the session
session.review do |word|
 # ...
end
```

To enable users to study in short bursts, supply a size on the review set.

```ruby
session = user.words.start_session!(:size => 5)
session.review do |word|
 # ...
end
```
Study sessions are persisted to the database and can be continued at any time.

```ruby
session = user.words.start_session!(10)
session.size # 10
user.right_answer_for!(session.words.first)
user.right_answer_for!(session.words.second)
# some time later ...
session = user.words.sessions.first
session.size # 10
```
Study sessions also track progress and will report when finished.

```ruby
session = user.words.start_session!(10)
session.size # 10
user.right_answer_for!(session.words.first)
user.right_answer_for!(session.words.second)
session.progress # 20 (%)
user.wrong_answer_for!(session.words.third) # incorrect answers also progress the user through the session
session.progress # 30 (%)
# after all answered
session.progress # 100 (%)
session.complete? # true
session.right # e.g. 4
session.wrong # e.g. 6
session.score # e.g. 60 (%)
```

Examples
--------

[Chuhi](https://github.com/rgravina/chuhi) is a Rails application which uses Okubo to schedule words. Have a look at the project source for real world usage. You can also [use the app online](http://chuhi.herokuapp.com/)!

Thanks!
-------

* [activerecord-reputation-system](https://github.com/twitter/activerecord-reputation-system/) - for providing the design and implementation from which this gem is based.
* [randym](https://github.com/randym/) - for enouraging me work on this gem in the first place.
* [Tokyo Rails](http://www.tokyorails.com/) - for providing collaboration evenings where this gem was worked on.
* [Reviewing the Kanji](http://kanji.koohii.com/) - for providing a great visual representation and implementation of the Leitner system.

Copyright and License
---------------------
Okubo © 2012 by Robert Gravina. Okubo is licensed under the MIT license. Please see the LICENSE document for more information.
