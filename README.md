# Ruby Advanced Class Methods

## Objectives

1. Build class finders.
2. Build class constructors.
3. Build class operators.

## Advanced Class Methods

Class methods can act as class readers for class variables as in the case of `Song.all`. Classes are just objects. Objects encapsulate and provide functionality unique to their scope through methods. Instance methods work on an individual level, acting upon specific instances of a class. In the same way, an entire class can be responsible for providing an interface to its data and behavior to the rest of your code through building class methods.

```ruby
class Person
  attr_accessor :name
  @@all = []

  def self.all
    @@all
  end
end
```

`def self.all` defines a class method that simply exposes the value in the class variable `@@all`. This is a class reader, very similar to an instance reader method that reads out of an instance level property such as `@name`.

What else can class methods help us with? What other common class-level functionality exposed through class methods?

## Class Finders

Imagine a Person class that provides access to all its instances through Person.all.

```ruby
class Person
  attr_accessor :name
  @@all = []

  def self.all
    @@all
  end

  def initialize(name)
    @name = name
    @@all << self
  end
end

grace_hopper = Person.new("Grace Hopper")
sandi_metz = Person.new("Sandi Metz")

Person.all #=> [#<Person @name="Grace Hopper">, #<Person @name="Sandi Metz">]
```

How might you find a specific person by name given this `Person` model?

```ruby
class Person
  attr_accessor :name
  @@all = []

  def self.all
    @@all
  end

  def initialize(name)
    @name = name
    @@all << self
  end
end

Person.new("Grace Hopper")
Person.new("Sandi Metz")

sandi_metz = Person.all.detect{|person| person.name == "Sandi Metz"}
sandi_metz #=> #<Person @name="Sandi Metz">

grace_hopper = Person.all.detect{|person| person.name == "Grace Hopper"}
grace_hopper #=> #<Person @name="Grace Hopper">

avi_flombaum = Person.all.detect{|person| person.name == "Avi Flombaum"}
avi_flombaum #=> nil
```

We're using the [`#detect`](http://ruby-doc.org/core-2.2.3/Enumerable.html#method-i-detect) to return the first object from `Person.all` that satisfies the condition of its name property equaling the name we are looking for.

Every time your application requires you to find a particular person by name you will have to use `#detect` or `#each` or some sort of iteration logic on `Person.all` to find a specific instance of a person that has the name you want.

Instead of implementing that logic throughout your code, you should encapsulate it into a class method `Person.find_by_name`. Make the `Person` class responsible for knowing how to find a person by name.

```ruby
class Person
  attr_accessor :name
  @@all = []

  def self.all
    @@all
  end

  def initialize(name)
    @name = name
    @@all << self
  end

  def self.find_by_name(name)
    @@all.detect{|person| person.name == name}
  end
end

Person.new("Grace Hopper")
Person.new("Sandi Metz")

sandi_metz = Person.find_by_name("Sandi Metz")
sandi_metz #=> #<Person @name="Sandi Metz">

grace_hopper = Person.find_by_name("Grace Hopper")
grace_hopper #=> #<Person @name="Grace Hopper">

avi_flombaum = Person.find_by_name("Avi Flombaum")
avi_flombaum #=> nil
```

We call class methods like `Person.find_by_name` finders. Finder class methods are responsible for finding instances based on properties or conditions.

But we can improve the code above slightly. Code that relies on abstraction is more maintainable and extendable over time. In general, we advance as a species and a civilization when technology provides an abstraction for us to use instead of the literal implementation. When you want light, you don't need to start a fire, you can just flick a light switch. That is an abstraction. The light switch is an abstraction for how people used to create light from fire. We promise. If creating and using abstractions have gotten people this far, we should probably continue embracing that design principle in our code. So where is the literal thing that could be abstracted in the code above?

Our current implementation of `Person.find_by_name` reads the instance data for the class directly out of the class variable `@@all`. But imagine if this variable changes? We'd have to update both Person.all to read out of the new variable and `Person.find_by_name` and any other method that relied on the literal variable name would break otherwise.

```ruby
class Person
  attr_accessor :name
  @@people = [] # changed from @@all

  def self.all
    @@people # changed from @@all
  end

  def initialize(name)
    @name = name
    @@people << self # changed from @@all
  end

  def self.find_by_name(name)
    @@people.detect{|person| person.name == name} # changed from @@all
  end
end
```

Variable names are a very low level abstraction. They are like making light by fire. Methods that read out of a variable provide an abstraction for the literal variable name. Relying and using a reader method is almost always better than using the variable.

We already have a method to read `@@all`, `Person.all`, why not use that method in `Person.find_by_name`? Within a class method, how do we call another class method? What is the scope of the class method? What is self? The class itself. Consider:

```ruby
class Person
  attr_accessor :name
  @@people = []

  def self.all
    @@people
  end

  def initialize(name)
    @name = name
    self.class.all << self
  end

  def self.find_by_name(name)
    self.all.detect{|person| person.name == name}
  end
end
```

`self.all` in the context of `Person.find_by_name` is equivalent to `Person.all` as the scope of the method is the class. `self` within a class method is the class itself.

Within `#initialize`, an instance method, `self` will refer to an instance, an individual person, not the entire class, so we can't simply say `self.all` within an instance method. Instead, we go from the instance, `self`, to the class via `self.class`, returning `Person`, and then evoke the `Person.all` method.

If the variable `@@all` changes names, we only have to update it in one place, the `Person.all` reader. All code that relies on that method still works. 1 conceptual change, 1 line-of-code (LOC) change. That is a proportionate amount of work.

In addition to the maintainability of our code through class method level encapsulation (when we build class methods to consolidate the logic of how the class operates so we only have to update 1 place in our code when something changes), class methods provide a more readable API for the class in the rest of the code. Consider just one more time the difference in seeing the following two lines of code littered throughout your code:

```ruby
Person.all.detect{|p| p.name == "Ada Lovelace"} # Literal implementation, no abstraction or encapsulation

Person.find_by_name("Ada Lovelace") # Abstract implementation with logic entirely encapsulated.
```

Whenever we use `Person.find_by_name` the intention of our code is revealed. Instead of iterating on an array, our code reads clearly. Instead of describing the implementation of finding a person by name, our code simply says what it is doing, not how. This is called an API. You want to build objects that provide a semantic and obvious API. Methods that reveal what the object will do, not how it does that. Always hide the how and show the what.

Finders are just one example of a more semantic API for our classes. Let's look at another way class methods can provide a higher level of semantics for our code.

## Custom Class Constructors

Our marketing team has provided us with a list of people in Comma Separated Format (CSV), a common formatting convention when exporting from spreadsheets. The raw data looks like:

```
Elon Musk, 44, Tesla/SpaceX
Steve Jobs, 56, Apple
Martha Stewart, 74, MSL
```

They tell us that they will often need to upload CSVs of people data.  Let's look at how we'd create a person instance from a csv.

```ruby
class Person
  attr_accessor :name, :age, :company
end

csv_data = "Elon Musk, 44, Tesla
Steve Jobs, 56, Apple
Martha Stewart, 74, MSL
"

rows = csv_data.split("\n")
people = rows.collect do |row|
  data = row.split(", ")
  name = data[0]
  age = data[1]
  company = data[2]
  person = Person.new
  person.name = name
  person.age = age
  person.company = company
  person
end
people #=> [#<Person @name="Elon Musk"...>, #<Person @name="Steve Jobs"...>...]
```

Pretty complex. We don't want to do that through our application. In an ideal world every time we got csv data we'd just want the `Person` class to be responsible for parsing it. Could we build something like `Person.new_from_csv`? Of course! Let's look at how we might implement a custom constructor.

```ruby
class Person
  attr_accessor :name, :age, :company

  def self.new_from_csv(csv_data)
    rows = csv_data.split("\n")
    people = rows.collect do |row|
      data = row.split(", ")
      name = data[0]
      age = data[1]
      company = data[2]

      person = self.new # This is an important line.
      person.name = name
      person.age = age
      person.company = company
      person
    end
    people
  end
end

csv_data = "Elon Musk, 44, Tesla
Steve Jobs, 56, Apple
Martha Stewart, 74, MSL
"

people = Person.new_from_csv(csv_data)
people #=> [#<Person @name="Elon Musk"...>, #<Person @name="Steve Jobs"...>...]

new_csv_data = "Avi Flombaum, 31, Flatiron School
Payal Kadakia, 30, ClassPass
"

people << Person.new_from_csv(csv_data)
people #=> [
#<Person @name="Elon Musk"...>,
#<Person @name="Steve Jobs"...>
#<Person @name="Martha Stewart"...>,
#<Person @name="Avi Flombaum"...>,
#<Person @name="Payal Kadakia"...>
# ]
```

We can see that when needing to parse two sets of CSV data, having a class method `Person.new_from_csv` greatly simplifies our code. Let's look at how the class method works a little more closely.

```ruby
class Person
  attr_accessor :name, :age, :company

  def self.new_from_csv(csv_data)
    # Split the CSV data into an array of individual rows.
    rows = csv_data.split("\n")
    # For each row, let's collect a Person instance based on the data.
    people = rows.collect do |row|
      # Split the row into 3 parts, name, age, company, at the ", " in the data.
      data = row.split(", ")
      name = data[0]
      age = data[1]
      company = data[2]

      # Make a new instance
      person = self.new # self refers to the Person class. This is Person.new
      # Set the properties on the person.
      person.name = name
      person.age = age
      person.company = company
      # Return the person to collect
      person
    end
    # Return the array of newly created people.
    people
  end
end
```

Like in any class method, `self` refers to the class itself so we can call `self.new` to piggyback, wrap, or extend the functionality of `Person.new`. We parse the raw data, create an instance, and assign the data to the corresponding instance properties.

Why do this? If we need to be able to create people from csvs, why not just build that directly into initialize? Well, the honest answer is because we don't always want to create people from csv data. Anything we build into initialize will happen always. Another key to writing maintainable code is designing functionality that is closed to modification but open to extension.

Initialize should be closed to modification. It should only handle the most required and common cases of initializing an object. Anything we add to initialize should be permanent and never modified. If we need more functionality when making an instance, instead of modifying initialize, we can extend it by wrapping it within a custom constructor.

If we ever need to make people from xml or json we can continue extend the object with custom constructors instead of constantly modifying initialize with complex logic.

Let's look at a somewhat simpler example of a custom constructor that wraps `.new`. When building objects that can be saved into a class variable `@@all`, we might not always want to save the newly instantiated instance.

```ruby
class Person
  @@all = []

  def initialize
    @@all << self
  end
end
```

With that code, no matter what, people instances will always be saved. We could instead implement a simple `.create` class method to provide the functionality of instantiating and creating the instance, leaving `.new` to function as normal.

```ruby
class Person
  @@all = []

  def self.create
    @@all << self.new
  end
end
```

## Class Operators

Beyond finders and custom constructors that return existing instances or create new instances, class methods can also manipulate class-level data.

A basic case of this might be printing all the people in our application.

```ruby
class Person
  attr_accessor :name
  @@all = []
  def self.all
    @@all
  end

  def self.create(name)
    person = self.new
    person.name = name
    @@all << person
  end
end

Person.create("Ada Lovelace")
Person.create("Grace Hopper")

# Printing each person
Person.all.each do |person|
  puts "#{person.name}"
end
```

Even that logic is worth encapsulating within a class method `.print_all`.

```ruby
class Person
  attr_accessor :name
  @@all = []
  def self.all
    @@all
  end

  def self.create(name)
    person = self.new
    person.name = name
    @@all << person
  end

  def self.print_all
    self.all.each{|person| puts "#{person.name}"}
  end
end

Person.create("Ada Lovelace")
Person.create("Grace Hopper")

Person.print_all
```

Way nicer.

## Class Operators

Additionally, class methods might provide a global operation on data. Imagine that one of the csvs we were provided with has peoples' names in lowercase letters. We want proper capitalization. We can build a class method `Person.normalize_names`

```ruby
class Person
  attr_accessor :name
  @@all = []
  def self.all
    @@all
  end

  def initialize(name)
    @name = name
    @@all << self
  end

  def self.normalize_names
    self.all.each do |person|
      person.name = person.name.split(" ").collect{|w| w.capitalize}.join(" ")
    end
  end
end
```

The logic for actually normalizing a person's name is pretty complex. `person.name.split(" ").collect{|w| w.capitalize}.join(" ")`

What we're doing is splitting a name, like `"ada lovelace"`, into an array at the space, `" "`, returning `["ada", "lovelace"]`. With that array we collect each word into a new array after it has been capitalized, returning `["Ada", "Lovelace"]`. We then join the elements in that array with a `" "` returning the final capitalized name, `"Ada Lovelace"`.

Given how complex normalizing a person's name is, we should actually encapsulate that into the `Person` instance.

```ruby
class Person
  attr_accessor :name
  @@all = []
  def self.all
    @@all
  end

  def initialize(name)
    @name = name
    @@all << self
  end

  def normalize_name
    self.name.split(" ").collect{|w| w.capitalize}.join(" ")
  end

  def self.normalize_names
    self.all.each do |person|
      person.name = person.normalize_name
    end
  end
end
```

With `#normalize_name`, we've taught a `Person` instance how to properly convert it's name into a capitalized version. The class method that acts on the global data of all people is simplified and delegates the actual normalization to the original instances. This is a common pattern for global class operators.

Another example of this type of global data manipulation might be deleting all the people. We would build a `Person.destroy_all` class method that will clear out the `@@all` array.

```ruby
class Person
  attr_accessor :name
  @@all = []
  def self.all
    @@all
  end

  def initialize(name)
    @name = name
    @@all << self
  end

  def self.destroy_all
    self.all.clear
  end
end
```

Here our `Person.destroy_all` method uses the [`Array#clear`](http://docs.ruby-lang.org/en/2.0.0/Array.html#method-i-clear) method to empty the `@@all` array through the class reader `Person.all`.

<a href='https://learn.co/lessons/ruby-advanced-class-methods-readme' data-visibility='hidden'>View this lesson on Learn.co</a>
