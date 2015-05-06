Frames, Call Stacks, and Exceptions
===================================

What are exceptions
-------------------

Exceptions a way to break the normal path that Ruby takes through a program.
Exceptions don't evaluate to an expression that has a value, but instead bubble
back up the callstack.

This allows them to return to an arbitrary location that knows how to handle
the exception (or kills the program if nothing does). They are primarily used
for error messages that the code does not know how to handle.

Example
-------

Here, Ruby goes down the callstack like normal,
but once the exception is raised, it does not continue on,
and instead it goes back up the callstack into a, and then main,
and then kills the program. So we do not see `b2`, `a2`, or `main2` printed,
as we normally would.

```ruby
def a
  puts "a1"
  b
  puts "a2"
end

def b
  puts "b1"
  raise RuntimeError, "die die die!"
  puts "b2"
end

puts "main1"
a
puts "main2"

# >> main1
# >> a1
# >> b1
```


REscuing an exception
---------------------

When an exception bubbles up through your code, you can deal with it by "rescuing" it.
This means that instead of going up to the next spot in the callstack,
it goes to the spot that you placed it.

Here, we rescue any runtime errors that get raised in the method body.
The error is assigned to the local variable `err`, and the code between
`rescue` and `end` is executed before returning like normal.

This code skips `b2` and `a2`, but resumes normal flow for `main2`

```ruby
def a
  puts "a1"
  b
  puts "a2"
rescue RuntimeError => err
  puts "a3, rescued: #{err.message}"
end

def b
  puts "b1"
  raise RuntimeError, "die die die!"
  puts "b2"
end

puts "main1"
a
puts "main2"

# >> main1
# >> a1
# >> b1
# >> a3, rescued: die die die!
# >> main2
```

Rescuing outside a method
-------------------------

When you're not in a method, you can wrap a `begin`/`end` block around the code,
and place the rescue inside of it.

```ruby
begin
  raise RuntimeError, 'zomg'
rescue RuntimeError => err
  err.message  # => "zomg"
end
```


You can rescue multiple exceptions
----------------------------------

Sometimes you want to do different things in different cases.
Perhaps if we try to open a file that doesn't exist, we want to tell the user
that they gave a bad path and reprompt, but if the file is empty,
we want to run an initialization script to get the default value.

The first rescue is executed, because the file does not exist.

```ruby
begin
  File.read '/this/file/does/not.exist'
rescue Errno::ENOENT => err
  err.message  # => "No such file or directory @ rb_sysopen - /this/file/does/not.exist"
rescue RuntimeError => err
  err.message
end
```

The second rescue is executed, because the file is empty.

```ruby
begin
  raise RuntimeError, 'File is empty!'
rescue Errno::ENOENT => err
  err.message
rescue RuntimeError => err
  err.message  # => "File is empty!"
end
```


Exception inheritance
---------------------

Exceptions are objects, like most things in Ruby.
If we rescue one of the superclasses, lets say `Exception`,
then we're saying "I can rescue any kind of exception".
Since a `RuntimeError` inherits from `Exception`,
we can handle it, too, and our rescue block is invoked.


```ruby
RuntimeError.ancestors # => [RuntimeError, StandardError, Exception, Object, PP::ObjectMixin, Kernel, BasicObject]

begin
  raise RuntimeError, 'zomg'
rescue Exception => err
  err.class   # => RuntimeError
  err.message # => "zomg"
end
```


Custom exceptions
-----------------

Given exceptions are objects, we can inherit from one of these classes to make
our own exceptions.

```ruby
class MissingConfigFile < StandardError
end

class BadConfigValue < StandardError
end

begin
  raise BadConfigValue, "Name is expected to be a string"
rescue MissingConfigFile => err
  err.message
rescue BadConfigValue => err
  err.message  # => "Name is expected to be a string"
end
```


`ensure` that an exception doesn't mess your code up
----------------------------------------------------

Sometimes you need to make sure something happens, even if an exception gets raised.

```ruby
begin
  file = File.open "myfile.csv"
  raise RuntimeError, 'zomg'
  file.close # <-- this will never get run, which can mess things up!
end
```

We could rescue every exception, do the thing, and then raise the exception again.
But this is common enough that ruby gives us an `ensure` keyword.

```ruby
begin
  file = File.open "myfile.csv"
  raise RuntimeError, 'zomg'
ensure
  file.close # <-- this will always be run
end
```

The `ensure` will run our code, but not stop the exception from bubbling up.

```ruby
def a
  puts 'a1'
  raise RuntimeError, 'zomg'
  puts 'a2'
ensure
  puts 'a3'
end

puts 'm1'
a
puts 'm2'
```



You figure out the answer (+ break)
-----------------------------------

Spend the next half hour figuring out the answers to these questions.
Also make sure you get a break in there somewhere to contemplate the
knowledge imparted by this exceptional lesson.

* Can I name the variable something other than `err`?
* Does a rescue block get run if no exception is raised?
* Do I have to accept the exception in a local variable? What happens if I omit `=> err`?
* You can omit the class, for convenience `raise 'zomg'` what type of exception gets raised?
* What happens if you rescue without giving a class?

  ```ruby
  begin
  rescue
    puts 'hello!'
  end
  ```
* Can you both rescue an exception and ensure that something happens?
* You can get a list of errors by running this code:

  ```ruby
  Object.constants.grep(/error|exception/i)
  ```

  Take a look at the inheritance (use `.ancestors`) of a few of them.
  Can you infer anything from this?

