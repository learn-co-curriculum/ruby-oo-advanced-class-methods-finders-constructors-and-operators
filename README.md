# Ruby Advanced Class Methods

## Outline

weve seen class methods mostly act as class readers for class variables as in the case of Class.all

Class methods can provide more functionality than exposing data through readers. Consider methods as responsibilities of objects, class methods should provide functionality that relates to the entire class in general. What might some of those examples be?

Finders
  Person.find_by_name
 given all, let's build self.find_by_name
  What self in a class method
  instead of reading out of @@all let's use self.all and call the class method. Discuss method scope and how it defines self.
 Naming functionality / building an api
  methods like find_by_name provide a better API. encapsulation. better design.

Custom constructors
  Person.new_from_csv()
  What's self
    using self to call self.new the original constructor
  Why do this?
    Wrapping new (open to extension leave initialize alone, anything that goes in initialize has to happen all the time..use custom constructors to extend basic functionality)
    consider Person.new (doesn't call #save vs Person.create)
  Cool use of Tap

Global operations
  weve seen class methods that find instances, and we've seen class methods that create instances, now lets look at class methods that use and manipulate instances
  Person.print_all
  Person.reset_all
  Person.capitalize_names!
