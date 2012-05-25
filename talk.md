## Factory Girl

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
** if you use a FactoryGirl method (create/build/stub) in your factory you still need to explictly say FactoryGirl.method. If you forget this it will cause weird errors that are pretty hard to trace.
You could always include the test_helper in the factory but creating factories within a factory is generally something to try to avoid.
Instead try to create all the things within the setup method of the test
* So I know you are thinking 'that shit aint DRY mang', well it can be.
** We can create setup helpers to avoid repeating ourselves.
** For now we can just deal with large setup methods until we get better test coverage.

* factories can generate name with 'sequence(:name) {|n| "Category #{n}" }'
** will create Category 1, Category 2, etc
** DO NOT rely on this convention for testing
```ruby
FactoryGirl.define do
  factory :category do
    sequence(:name) {|n| "Category #{n}" }
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
