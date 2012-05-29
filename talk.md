# I can haz good code?
* Code isn’t good when it’s tested, DRY, or SOLID. Code is good when it works and we can work with it.
When our teammates can understand our code quickly, respond to change, and are relaxed and happy, we’re writing good code.
* Slow or brittle tests make code harder to change. 
We have to be careful about writing the correct tests and the correct amount of tests.
* Test-driven development helps us understand code because we state the reason for each line of application code in the test. 
It also makes us happy by providing a clear process for working.
* An automated test suite helps us change code because we have an early warning system when we break an existing component. 
It is fun to work with because we’re more confident and less fearful

# Lofty Goals
* Any change of the logic of the application should be driven by a failed test
* A test should be as close as possible to the assoicated code
* If its not tested, it's broken
* Testing is supposed to help for the long term. The long term could be tomorrow or even sooner
* Tests are code, refactor them too
* Start a bug fix by writing a test

# What can testing do for us?
* The number one thing testing can accomplish is preventing regression bugs
* Being able to catch a regression bug before youve even commited the code will save everyone time
* It can also serve as documentation for our code
* When developing it makes it easier to find a place to start. 
Its easy to think of something the program should do so write a test for that
 
# Classic TDD Process
1. create a test, it should be short and only test one thing
2. make sure the test fails
3. write the simplest code to make the test pass
4. Once the test passes go back and optimize/dry up your code

* If followed this process should produce code made of small methods, that each do one thing.
These methods tend to be loosely coupled and have minimal side effects

# CI
* We use Jenkins which is an open source fork of Hudson
* CI lives at ci.amsapps.com
* fubot jenkins list 
* bulid status badge in markdown

## CI still needs
* security/users
* a real OS
* to run headless integration tests using waitir or capybara-webkit

# Functional
* Tests for controllers
* We have none, but these aren't the most important tests for now

# Integration
* Tests for controllers interacting together.
* We use Capybara and some other goodies to make it headless so it can be ran by ci
* A basic integration test would be logging into webadmit

# Unit
* Tests for models
* This is the main area where we need to expand our test coverage

## Testing tools:
* MiniTest Unit/Spec
* ActiveSupport::TestCase
* Shoulda

## How to run tests
* rake test:units This runs all the unit tests. this currently what ci is running
* rake test TEST=test/unit/my_awesome_test.rb

# Fixtures
test/fixtures/users.yml
```yml
fred:
  first_name: Fred
  last_name: Flinstone
  email: fflinstone@yabadaba.do
barney:
  first_name: Barney
  last_name: Rubble
  email: brubble@yabadaba.do
```
## Why we are going to factories instead of fixtures
* Fixtures are global, spread out, and brittle
* We would like a system to be local, compact and robust
* So every test can have an individual setup tuned to that test
* So the test data is easy to generate
* So tests dont depend on changes made to setup data in other tests
* We should be able to specify more data in the current test without breaking other tests

# Factory Girl

```ruby
# Returns a saved User instance
# Persists user and any attributes/associations in the db
# slowest
user = FactoryGirl.create(:user)

# Returns a User instance that's not saved in the db
# This will create records for an associations it may have
# faster than create
user = FactoryGirl.build(:user)

# Returns an object with all defined attributes stubbed out
# does not persist any records for the object or its attributes
# fastest
stub = FactoryGirl.build_stubbed(:user)

# Returns a hash of attributes that can be used to build a User instance
attrs = FactoryGirl.attributes_for(:user)

# Passing a block to any of the methods above will yield the return object
FactoryGirl.create(:user) do |user|
  user.posts.create(attributes_for(:post))
end
```

* keep factories as light as possible; the base factory should include only what is needed for validations(and maybe a name/title)

* We include the factory girl method in test_helper.rb so you only need to say create instead of FactoryGirl.create
this is great but has at least one thing to watch out for:
+ if you use a FactoryGirl method in your factory you still need to explictly say FactoryGirl.method. If you forget this it will cause weird errors that are pretty hard to trace.
+ You could always include the test_helper in the factory but creating factories within a factory is generally something to try to avoid.
+ Instead try to create all the things within the setup method of the test

* So I know you are thinking 'that shit aint DRY mang', well it can be.
+ We can create setup helpers to avoid repeating ourselves.
+ For now we can just deal with large setup methods until we get better test coverage.

* factories can generate name with 'sequence(:name) {|n| "Category #{n}" }'
  this will create Category 1, Category 2, etc
* DO NOT rely on this convention for testing

```ruby
  FactoryGirl.define do
    factory :category do
      sequence(:name) {|n| "Category #{n}" }
    end
  end

  describe Foo do
    before do
      @cat = create(:category)
    end

    it "must have a name of Category 1" do
      @cat.name.must_equal "Category 1"
    end

  end
```
