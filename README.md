# ActiveRecord Callback Analyzer

Arca is a callback analyzer for ActiveRecord like models ideally suited for digging yourself out of callback hell. Arca helps you answer questions like:

* how spread out callbacks are for each model
* how many callbacks use conditionals (`:if` and `:unless`)
* how many possible permutations exist per callback type (`:commit`, `:create`, `:destroy`, `:find`, `:initialize`, `:rollback`, `:save`, `:touch`, `:update`, `:validation`) taking conditionals into consideration

The Arca library has two main components, the collector and the reporter. Include the collector module in each ActiveRecord model you want to analyze and then use the reporter to analyze and present the data.

## Usage

Add the gem to your Gemfile and run `bundle`.

```
gem 'arca'
```

Add an initializer to require the library and configure it (`config/initializers/arca.rb` for example). There's no magic here and Arca doesn't assume your root project path or the path to your `ActiveRecord` models so you have to specify those paths yourself in the initializer.

```
require "arca"

Arca.root_path = Rails.root
Arca.model_path = Rails.root.join("app", "models")
```

Include `Arca::Collector` in the models you want to analyze. It must be included before any callbacks so I recommend including it right after the class definition.

```ruby
class Ticket < ActiveRecord::Base
  include Arca::Collector
  include Announcements

  before_save :set_title, :set_body
  before_save :upcase_title, :if => :title_is_a_shout?

  def set_title
    self.title ||= "Ticket id #{SecureRandom.hex(2)}"
  end

  def set_body
    self.body ||= "Everything is broken."
  end

  def upcase_title
    self.title = title.upcase
  end

  def title_is_a_shout?
    self.title.split(" ").size == 1
  end
end
```

In this example the `Annoucements` module is included in `Ticket` and defines it's own callback.

```ruby
module Announcements
  def self.included(base)
    base.class_eval do
      after_save :announce_save
    end
  end

  def announce_save
    puts "saved #{self.class.name.downcase}!"
  end
end
```

Next use `Arca[Ticket].report` to analyze the callbacks for the `Ticket` class.

```ruby
> Arca.root_path = `pwd`.chomp
=> "/Users/jonmagic/Projects/arca"
> Arca[Ticket].report
{
                :model_name => "Ticket",
           :model_file_path => "test/fixtures/ticket.rb",
           :callbacks_count => 4,
       :lines_between_count => 6,
  :included_callbacks_count => 1,
        :conditionals_count => 1,
   :calculated_permutations => 2
}
> Arca[Ticket].analyzed_callbacks
{
  :before_save => [
    {
      :callback                       => :before_save,
      :callback_file_path             => "test/fixtures/ticket.rb",
      :callback_line_number           => 5,
      :external_callback              => false,
      :target                         => :set_title,
      :target_file_path               => "test/fixtures/ticket.rb",
      :target_line_number             => 8,
      :external_target                => false,
      :lines_to_target                => 3,
      :conditional                    => nil,
      :conditional_target             => nil,
      :conditional_target_file_path   => nil,
      :conditional_target_line_number => nil,
      :external_conditional_target    => nil,
      :lines_to_conditional_target    => nil
    },
    {
      :callback                       => :before_save,
      :callback_file_path             => "test/fixtures/ticket.rb",
      :callback_line_number           => 5,
      :external_callback              => false,
      :target                         => :set_body,
      :target_file_path               => "test/fixtures/ticket.rb",
      :target_line_number             => 12,
      :external_target                => false,
      :lines_to_target                => 7,
      :conditional                    => nil,
      :conditional_target             => nil,
      :conditional_target_file_path   => nil,
      :conditional_target_line_number => nil,
      :external_conditional_target    => nil,
      :lines_to_conditional_target    => nil
    },
    {
      :callback                       => :before_save,
      :callback_file_path             => "test/fixtures/ticket.rb",
      :callback_line_number           => 6,
      :external_callback              => false,
      :target                         => :upcase_title,
      :target_file_path               => "test/fixtures/ticket.rb",
      :target_line_number             => 16,
      :external_target                => false,
      :lines_to_target                => 10,
      :conditional                    => :if,
      :conditional_target             => :title_is_a_shout?,
      :conditional_target_file_path   => "test/fixtures/ticket.rb",
      :conditional_target_line_number => 20,
      :external_conditional_target    => false,
      :lines_to_conditional_target    => nil
    }
  ],
  :after_save  => [
    {
      :callback                       => :after_save,
      :callback_file_path             => "test/fixtures/announcements.rb",
      :callback_line_number           => 4,
      :external_callback              => true,
      :target                         => :announce_save,
      :target_file_path               => "test/fixtures/announcements.rb",
      :target_line_number             => 8,
      :external_target                => false,
      :lines_to_target                => 4,
      :conditional                    => nil,
      :conditional_target             => nil,
      :conditional_target_file_path   => nil,
      :conditional_target_line_number => nil,
      :external_conditional_target    => nil,
      :lines_to_conditional_target    => nil
    }
  ]
}
```

## License

The MIT License (MIT)

Copyright (c) 2015 Jonathan Hoyt

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
