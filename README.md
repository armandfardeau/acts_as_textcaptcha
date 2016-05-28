## ActAsTextcaptcha

[![Gem Version](https://img.shields.io/gem/v/acts_as_textcaptcha.svg?style=flat)](http://rubygems.org/gems/acts_as_textcaptcha)
[![Travis Build Status](https://img.shields.io/travis/matthutchinson/acts_as_textcaptcha.svg?style=flat)](https://travis-ci.org/matthutchinson/acts_as_textcaptcha)
[![Coverage Status](https://coveralls.io/repos/mroth/lolcommits/badge.svg?branch=master&service=github)](https://coveralls.io/r/matthutchinson/acts_as_textcaptcha?branch=master)
[![Code Climate Analysis](https://img.shields.io/codeclimate/github/matthutchinson/acts_as_textcaptcha.svg?style=flat)](https://codeclimate.com/github/matthutchinson/acts_as_textcaptcha)
[![Gem Dependencies](https://img.shields.io/gemnasium/matthutchinson/acts_as_textcaptcha.svg?style=flat)](https://gemnasium.com/matthutchinson/acts_as_textcaptcha)

ActsAsTextcaptcha provides spam protection for your Rails models using logic
questions from the excellent [Text CAPTCHA](http://textcaptcha.com/) web service
(by [Rob Tuley](http://openknot.com/me/) of [Openknot](http://openknot.com/)).
It is also possible to configure your own text captcha questions (instead, or as
a fall back in the event of any remote API issues).

This gem is actively maintained, has good test coverage and is compatible with
Rails >= 3.0.0 (including Rails 4). If you have any issues please report them
[here](https://github.com/matthutchinson/acts_as_textcaptcha/issues/new).

Logic questions from the web service are aimed at a child's age of 7, so they
can be solved easily by even the most cognitively impaired users. As they
involve human logic, questions cannot be solved by a robot. There are both
advantages and disadvantages for using logic questions over image based
captchas, find out more at [TextCAPTCHA](http://textcaptcha.com/why).

## Demo

Try a [working demo here](http://textcaptcha.herokuapp.com)!

## Installing

*NOTE:* The steps to configure your app changed with the v4.0.0 release. If you
are having problems please carefully check the steps below or refer to the [upgrade
instructions](https://github.com/matthutchinson/acts_as_textcaptcha/wiki/Upgrading-from-3.0.10).

First add the following to your Gemfile, then `bundle install`;

    gem 'acts_as_textcaptcha'

Next [grab an API key for your website](http://textcaptcha.com/api), then add
the following code to models you would like to protect;

    class Comment < ActiveRecord::Base
      # (this is the simplest way to configure the gem, with an API key only)
      acts_as_textcaptcha :api_key => 'PASTE_YOUR_TEXTCAPTCHA_API_KEY_HERE'
    end

Next in your controller's *new* action you'll want to generate and assign the
logic question for the new record, like so;

    def new
      @comment = Comment.new
      @comment.textcaptcha
    end

Finally, in your form view add the textcaptcha question and answer fields using
the textcaptcha_fields helper.  Feel free to arrange the HTML within this block
as you like;

    <%= textcaptcha_fields(f) do %>
      <div class="field">
        <%= f.label :textcaptcha_answer, @comment.textcaptcha_question %><br/>
        <%= f.text_field :textcaptcha_answer, :value => '' %>
      </div>
    <% end %>

*NOTE:* If you'd rather NOT use this helper and would prefer to write your own
form view code, see the html produced from the textcaptcha_fields method in the
[source
code](https://github.com/matthutchinson/acts_as_textcaptcha/blob/master/lib/acts_as_textcaptcha/textcaptcha_helper.rb).

### Toggling Textcaptcha

You can toggle textcaptcha on/off for your models by overriding the
`perform_textcaptcha?` method. If it returns false, no questions will be fetched
from the web service and textcaptcha validation is disabled.

This is useful for writing your own custom logic for toggling spam protection
on/off e.g. for logged in users. By default the `perform_textcaptcha?` method
[checks if the form object is a new (unsaved)
record](https://github.com/matthutchinson/acts_as_textcaptcha/blob/master/lib/acts_as_textcaptcha/textcaptcha.rb#L54).

So out of the box, spam protection is only enabled for creating new records (not
updating).  Here is a typical example showing how to overwrite the
`perform_textcaptcha?` method, while maintaining the new record check.

    class Comment < ActiveRecord::Base
      acts_as_textcaptcha :api_key => 'YOUR_TEXTCAPTCHA_API_KEY'

      def perform_textcaptcha?
        super && user.admin?
      end
    end

## More configuration options

You can configure captchas with the following options;

* *api_key* (_required_) - get a free key from http://textcaptcha.com/api
* *questions* (_optional_) - array of question and answer hashes (see below) A random question from this array will be asked if the web service fails OR if no API key has been set.  Multiple answers to the same question are comma separated (e.g. 2,two). So do not use commas in your answers!
* *cache_expiry_minutes* (_optional_) - minutes for answers to persist in the cache (default 10 minutes), see [below for details](https://github.com/matthutchinson/acts_as_textcaptcha#what-does-the-code-do)
* *http_read_timeout* (_optional_) - Net::HTTP option, seconds to wait for one block to be read from the remote API
* *http_open_timeout* (_optional_) - Net::HTTP option, seconds to wait for the connection to open to the remote API

For example;

    class Comment < ActiveRecord::Base
      acts_as_textcaptcha :api_key              => 'YOUR_TEXTCAPTCHA_API_KEY',
                          :http_read_timeout    => 60,
                          :http_read_timeout    => 10,
                          :cache_expiry_minutes => 10,
                          :questions            => [{ 'question' => '1+1', 'answers' => '2,two' },
                                                    { 'question' => 'The green hat is what color?', 'answers' => 'green' }]
    end

### YAML config

The gem can be configured for models individually (as shown above) or with a
config/textcaptcha.yml file.  The config file must have an api_key defined and
optional array of questions. Options definied inline in model classes take
preference over the configuration in textcaptcha.yml.  The gem comes with a
handy rake task to copy over a
[textcaptcha.yml](http://github.com/matthutchinson/acts_as_textcaptcha/raw/master/config/textcaptcha.yml)
template to your config directory;

    rake textcaptcha:config

### Configuring _without_ the TextCAPTCHA web service

To use only your own logic questions, simply ommit the api_key from the
configuration and define at least 1 logic question and answer (see above).

## Translations

The gem uses the standard Rails I18n translation approach (with a fall-back to
English). Unfortunately at present, the Text CAPTCHA web service only provides
logic questions in English.

    en:
      activerecord:
        errors:
          models:
            comment:
              attributes:
                textcaptcha_answer:
                  incorrect: "is incorrect, try another question instead"
                  expired: "was not submitted quickly enough, try another question instead"
      activemodel:
        attributes:
          comment:
            textcaptcha_answer: "Textcaptcha answer"

## Without Rails or ActiveRecord

Although this gem has been built with Rails in mind, is entirely possible to use
it without ActiveRecord, or Rails.  As an example, take a look at the
[Contact](https://github.com/matthutchinson/acts_as_textcaptcha/blob/master/test/test_models.rb#L44)
model used in the test suite
[here](https://github.com/matthutchinson/acts_as_textcaptcha/blob/master/test/test_models.rb#L44).

Please note that the built-in
[TextcaptchaCache](https://github.com/matthutchinson/acts_as_textcaptcha/blob/master/lib/acts_as_textcaptcha/textcaptcha_cache.rb)
class directly wraps the
[Rails.cache](http://api.rubyonrails.org/classes/ActiveSupport/Cache/Store.html).
An alternative  TextcaptchaCache implementation will be necessary if Rails.cache
is not available.

## Testing and docs

In development you can run the tests and rdoc tasks like so;

* rake test (all tests)
* rake test:coverage (all tests with code coverage)
* rake rdoc (generate docs)

## What does the code do?

The gem contains two parts, a module for your ActiveRecord models, and a single
view helper method.  The ActiveRecord module makes use of two futher classes,
[TextcaptchaApi](https://github.com/matthutchinson/acts_as_textcaptcha/blob/master/lib/acts_as_textcaptcha/textcaptcha_api.rb)
and
[TextcaptchaCache](https://github.com/matthutchinson/acts_as_textcaptcha/blob/master/lib/acts_as_textcaptcha/textcaptcha_cache.rb).

A call to @model.textcaptcha in your controller will query the Text CAPTCHA web
service.  A GET request is made with Net::HTTP and parsed using the default
Rails ActiveSupport::XMLMini backend.  A textcaptcha_question and a random cache
key is assigned to the record.  An array of possible answers is stored in the
TextcaptchaCache with this random key.  The cached answers have (by default) a
10 minute TTL in your cache. If your forms take more than 10 minutes to be
completed you can adjust this value setting the `cache_expiry_minutes` option.
Internally TextcaptchaCache wraps
[Rails.cache](http://api.rubyonrails.org/classes/ActiveSupport/Cache/Store.html)
and all cache keys are namespaced.

On saving, validate_textcaptcha is called on @model.validate checking that the
@model.textcaptcha_answer matches one of the possible answers (retrieved from
the cache).  By default, this validation is _only_ carried out on new records,
i.e. never on edit, only on create.  All attempted answers are case-insensitive
and have trailing/leading white-space removed.

Regardless of a correct, or incorrect answer the possible answers are cleared
from the cache and a new random key is generated and assigned.  An incorrect
answer will cause a new question to be prompted.  After one correct answer, the
answer and a mutating key are sent on further form requests, and no question is
presented in the form.

If an error or timeout occurs during API fetching, ActsAsTextcaptcha will fall
back to choose a random logic question defined in your options (see above).  If
the web service fails or no API key is specified AND no alternate questions are
configured, the @model will not require textcaptcha checking and will pass as
valid.

For more details on the code please check the
[documentation](http://rdoc.info/projects/matthutchinson/acts_as_textcaptcha).
Tests are written with [MiniTest](https://rubygems.org/gems/minitest).  Pull
requests and bug reports are welcome.

## Requirements

What do you need?

* [Rails](http://github.com/rails/rails) >= 3.0.0  (including Rails 4)
* [Rails.cache](http://api.rubyonrails.org/classes/ActiveSupport/Cache/Store.html) - some basic cache configuration is necessary
* [Ruby](http://ruby-lang.org/) >= 2.0.0 (tested with stable versions of 2.0, 2.1, 2.2, 2.3) - 1.8/1.9 support ended at version 4.1.1
* [Text CAPTCHA API key](http://textcaptcha.com/register) (_optional_, since you can define your own logic questions)
* [MiniTest](https://rubygems.org/gems/minitest) (_optional_ if you want to run the tests)
* [SimpleCov](https://rubygems.org/gems/simplecov) (_optional_ if you want to run the tests with code coverage reporting)

*Note*: The built-in
[TextcaptchaCache](https://github.com/matthutchinson/acts_as_textcaptcha/blob/master/lib/acts_as_textcaptcha/textcaptcha_cache.rb)
class directly wraps the
[Rails.cache](http://api.rubyonrails.org/classes/ActiveSupport/Cache/Store.html)
object.

## Rails 2 support

Support for Rails 2 was dropped with the release of v4.1.0. If you would like to
continue to use this gem with an older version of Rails (>= 2.3.8), please lock
the version to `4.0.0`. Like so;

    # in your Gemfile
    gem 'acts_as_textcaptcha', '=4.0.0'

    # or in environment.rb
    config.gem 'acts_as_textcaptcha', :version => '=4.0.0'

Check out the
[README](https://github.com/matthutchinson/acts_as_textcaptcha/tree/v4.0.0) for
this release for more information.

## Links

* [Demo](http://textcaptcha.herokuapp.com)
* [Travis CI](http://travis-ci.org/#!/matthutchinson/acts_as_textcaptcha)
* [Test Coverage](https://coveralls.io/r/matthutchinson/acts_as_textcaptcha?branch=master)
* [Code Climate](https://codeclimate.com/github/matthutchinson/acts_as_textcaptcha)
* [RDoc](http://rdoc.info/projects/matthutchinson/acts_as_textcaptcha)
* [Wiki](http://wiki.github.com/matthutchinson/acts_as_textcaptcha/)
* [Issues](http://github.com/matthutchinson/acts_as_textcaptcha/issues)
* [Report a bug](http://github.com/matthutchinson/acts_as_textcaptcha/issues/new)
* [Gem](http://rubygems.org/gems/acts_as_textcaptcha)
* [GitHub](http://github.com/matthutchinson/acts_as_textcaptcha)

## Who's who?

* [ActsAsTextcaptcha](http://github.com/matthutchinson/acts_as_textcaptcha) and [little robot drawing](http://www.flickr.com/photos/hiddenloop/4541195635/) by [Matthew Hutchinson](http://matthewhutchinson.net)
* [Text CAPTCHA](http://textcaptcha.com) API and service by [Rob Tuley](http://openknot.com/me/) at [Openknot](http://openknot.com)

## Usage

This gem is used in a number of production websites and apps. It was originally
extracted from code developed for [Bugle](http://bugleblogs.com).  If you're
happily using acts_as_textcaptcha in production, let me know and I'll add you to
this list!

* [matthewhutchinson.net](http://matthewhutchinson.net)
* [pmFAQtory.com](http://pmfaqtory.com)
* [The FAQtory](http://faqtoryapp.com)
* [DPT Watch, San Francisco](http://www.dptwatch.com)
