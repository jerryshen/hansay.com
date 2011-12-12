---
layout: post
title: Ruby 代码规范
categories:
- ruby
---

### 前奏

>  Ruby作为一种敏捷开发语言, 真的很难去规范每个人的编程风格, 但是好的风格对整个项目和协作开发都会起到很好的作用  
>  好的代码风格是区分好和非常好的评判标准  
>  -- Jerry Shen

作为一个从事Ruby开发将近4年的developer来说, 我们基本是没有一种比较官方的代码规范的指南, 文档或者最佳实践, 不像Python, 它拥有一个
很棒的编程规范参考[PEP-8][0]. 但我一直相信在开发项目的时候, 特别是多人协作, 代码风格至关重要, 我同时也相信作为Ruby开发者的我们
能够创作出一种比较好的风格体系.

很多人问过我写起Ruby代码来太自由了, 导致多人协作开发的时候, 在没有文档的情况下很难去读别人的代码. 我是一个极力推荐每个项目spec测试覆盖率要超过90%的人,
面对这种问题, 我会很无奈, 但确实有时候并不是所有的人都能像自己想象的那样, 这也是我翻译并修改了这篇规范的原因.

顺带一句, 如果你还想深入了解Rails代码风格, 可以参考[Ruby on Rails 3 代码规范][1]

### 规范内容

* [Ruby编程风格指南](#guide)
    * [Source Code Layout](#layout)
    * [Syntax](#syntax)
    * [Naming](#naming)
    * [Comments](#comments)
    * [Annotations](#annotations)
    * [Classes](#classes)
    * [Exceptions](#exceptions)
    * [Collections](#collections)
    * [Strings](#strings)
    * [Percent Literals](#literals)
    * [Miscellaneous](#misc)
    * [Design](#design)
* [Contributing](#contributing)
* [Spread the word](#spreadtheword)

### Ruby 代码规范

这篇代码规范指南的初衷就是为了Ruby程序员之间的代码互通, 可以很好的协作开发项目, 本指南被拆分成多个相关联的规范, 我并没有把这些规范做的很完善,
这些大多是我作为一个专业Ruby工程师平时的积累, 朋友的建议还有一些书籍, 比如 [Programming Ruby 1.9][2] 和
[The Ruby Programming Language][3], 这篇指南将会一直会处于完善阶段.

你可以使用 [Transmuter][4] 为这篇指南生成一个pdf或者html备份

## Source Code Layout

* 任何时候文件编码都要使用 `UTF-8`
* 代码的换行都使用 **两个空格**, 或者将一个 **tab** 设置成 **两个空格** 来用

{% highlight ruby %}
    # good
    def some_method
      do_something
    end

    # bad - four spaces
    def some_method
        do_something
    end
{% endhighlight %}

* 碰到 `+` 号, `:` 号, `;` 号还有 `{ }` 前后, 都要加上空格, 

* Use spaces around operators, after commas, colons and semicolons, around `{`
  and before `}`. 空格可能对你来说无关紧要, 但是确实让代码编码变得更加容易读.

{% highlight ruby %}
    sum = 1 + 2
    a, b = 1, 2
    1 > 2 ? true : false; puts 'Hi'
    [1, 2, 3].each { |e| puts e }
{% endhighlight %}

唯一的一个例外就是当我们在用指数运算符的时候:

{% highlight ruby %}
    # bad
    e = M * c ** 2

    # good
    e = M * c**2
{% endhighlight %}

* 在 `(`, `[` 后, `]`, `)` 前, 都不要加空格.

{% highlight ruby %}
    some(arg).other
    [1, 2, 3].length
{% endhighlight %}

* 如果判断的条件过多, 请用 `case when` 语句, 可能有些人不同意这一点, 但这确实是 "The Ruby
Programming Language" 和 "Programming Ruby" 所推荐的.

{% highlight ruby %}
    case
    when song.name == 'Misty'
      puts 'Not again!'
    when song.duration > 120
      puts 'Too long!'
    when Time.now.hour > 21
      puts "It's too late"
    else
      song.play
    end

    kind = case year
           when 1850..1889 then 'Blues'
           when 1890..1909 then 'Ragtime'
           when 1910..1929 then 'New Orleans Jazz'
           when 1930..1939 then 'Swing'
           when 1940..1950 then 'Bebop'
           else 'Jazz'
           end
{% endhighlight %}

* 每个方法间需要空一行, 这样看上去更加清爽.

{% highlight ruby %}
    def some_method
      data = initialize(options)

      data.manipulate!

      data.result
    end

    def some_method
      result
    end
{% endhighlight %}

* Use RDoc and its conventions for API documentation.  Don't put an
  empty line between the comment block and the `def`.
* Keep lines fewer than 80 characters.
* Avoid trailing whitespace.

<a name="syntax" /></a>
### Syntax

* 如果一个方法中需要传入参数, 那么 `def` 后使用括号 `()`, 如果没有参数, 则不需要加括号.

{% highlight ruby %}
     def some_method
       # body omitted
     end

     def some_method_with_arguments(arg1, arg2)
       # body omitted
     end
{% endhighlight %}

* 尽量避免使用 `for` 循环, 用迭代器替代. 从另外一方面说, `for` 其实是用 `each` 来实现的, 没必要再多加一层.

{% highlight ruby %}
    arr = [1, 2, 3]

    # bad
    for elem in arr do
      puts elem
    end

    # good
    arr.each { |elem| puts elem }
{% endhighlight %}

* 在 `if/unless` 后尽量避免使用 `then`.

{% highlight ruby %}
    # bad
    if some_condition then
      # body omitted
    end

    # good
    if some_condition
      # body omitted
    end
{% endhighlight %}

* 可以使用三元运算 `?:` 来替代 `if/else` 语句, 这样使得代码更加简洁.

{% highlight ruby %}
    # bad
    result = if some_condition
                something
              else
                something_else
              end

    # good
    result = some_condition ? something : something_else
{% endhighlight %}
  
* 不要嵌套三元运算, 会让逻辑变的难读, 其中可以用 `if/else` 来替代

{% highlight ruby %}
    # bad
    some_condition ? (nested_condition ? nested_something : nested_something_else) : something_else

    # good
    if some_condition
      nested_condition ? nested_something : nested_something_else
    else
      something_else
    end
{% endhighlight %}
  
* 避免使用 `if x: ...`, 请用三元运算代替, 因为这种用法在 Ruby 1.9 中移除了

{% highlight ruby %}
    # bad
    result = if some_condition: something else something_else end

    # good
    result = some_condition ? something : something_else
{% endhighlight %}

* 使用三元运算代替 `if x; ...` 的写法.

* 如果判断的条件只有一个, 请用 `when x then ...` 来表式, `when x: ...` 这种写法在 Ruby 1.9 中移除了

* 避免使用 `when x; ...`, 请看上一条规范.

* 布尔类型表达式使用 `&&/||`, 逻辑流程中使用 `and/or`.

{% highlight ruby %}
    # boolean expression
    if some_condition && some_other_condition
      do_something
    end

    # control flow
    document.saved? or document.save!
{% endhighlight %}

* 如果 `if/unless` 条件中只有一行代码, 可以直接使用 `xxx if condition`.

{% highlight ruby %}
    # bad
    if some_condition
      do_something
    end

    # good
    do_something if some_condition

    # another good option
    some_condition and do_something
{% endhighlight %}

* 代码中尽量使用 `unless` 来代替 `if !xxx`.

{% highlight ruby %}
    # bad
    do_something if !some_condition
    
    # good
    do_something unless some_condition

    # another good option
    some_condition or do_something
{% endhighlight %}

* 尽量不要用 `unless ... else ...` 这种写法, 请换成 `if ... else ...`

{% highlight ruby %}
    # bad
    unless success?
      puts 'failure'
    else
      puts 'success'
    end

    # good
    if success?
      puts 'success'
    else
      puts 'failure'
    end
{% endhighlight %}

* 在 `if/unless/while` 的条件中不要使用 `()`

{% highlight ruby %}
    # bad
    if (x > 10)
      # body omitted
    end

    # good
    if x > 10
      # body omitted
    end
{% endhighlight %}

* DSL(e.g. Rake, Rails, RSpec) 中的内部方法参数不要使用括号 `()`, 比如 `attr_reader`, `puts` 和 attribute access 等方法.
  当调用某些方法的时候需要用括号 `()`.

{% highlight ruby %}
    class Person
      attr_reader name, age

      # omitted
    end

    temperance = Person.new('Temperance', 30)
    temperance.name

    puts temperance.age

    x = Math.sin(y)
    array.delete(e)
{% endhighlight %}

* 单行的blocks用 `{...}` 替代 `do...end`用法, 多行的blocks用 `do...end` 替代 `{...}`

{% highlight ruby %}
    names = ["Bozhidar", "Steve", "Sarah"]

    # good
    names.each { |name| puts name }

    # bad
    names.each do |name|
      puts name
    end

    # good
    names.select { |name| name.start_with?("S") }.map { |name| name.upcase }

    # bad
    names.select do |name|
      name.start_with?("S")
    end.map { |name| name.upcase }
{% endhighlight %}

* 避免使用 `return`

{% highlight ruby %}
    # bad
    def some_method(some_arr)
      return some_arr.size
    end

    # good
    def some_method(some_arr)
      some_arr.size
    end
{% endhighlight %}

* 当方法中指定参数的默认值时, `=` 号前后用空格空开

{% highlight ruby %}
    # bad
    def some_method(arg1=:default, arg2=nil, arg3=[])
      # do something...
    end

    # good
    def some_method(arg1 = :default, arg2 = nil, arg3 = [])
      # do something...
    end
{% endhighlight %}

> 一些Ruby书籍上也建议过用第一种形式, 但是第二种我觉得更加易读.

* 尽量避免使用 `\` 来换行续写代码.

{% highlight ruby %}
    # bad
    result = 1 - \
             2

    # good (但是还很丑)
    result = 1 \
             - 2
{% endhighlight %}

* 常用 `||=` 来初始化变量.

{% highlight ruby %}
    # set name to Bozhidar, only if it's nil or false
    name ||= 'Bozhidar'
{% endhighlight %}

* 初始化布尔类型变量的时候不要用 `||=`

{% highlight ruby %}
    # bad - would set enabled to true even if it was false
    enabled ||= true

    # good
    enabled = true if enabled.nil?
{% endhighlight %}

* 碰到括号, 方法和括号之间不要加空格.

{% highlight ruby %}
    # bad
    f (3 + 2) + 1

    # good
    f(3 + 2) + 1
{% endhighlight %}

* If the first argument to a method begins with an open parenthesis,
  always use parentheses in the method invocation. For example, write
`f((3 + 2) + 1)`.

* Always run the Ruby interpreter with the `-w` option so it will warn
  you if you forget either of the rules above!

<a name="naming"/>
## Naming

> The only real difficulties in programming are cache invalidation and
> naming things. <br/>
> -- Phil Karlton

* Use `snake_case` for methods and variables.
* Use `CamelCase` for classes and modules.  (Keep acronyms like HTTP,
  RFC, XML uppercase.)
* Use `SCREAMING_SNAKE_CASE` for other constants.
* The names of predicate methods (methods that return a boolean value)
  should end in a question mark.
  (i.e. `Array#empty?`).
* The names of potentially "dangerous" methods (i.e. methods that modify `self` or the
  arguments, `exit!`, etc.) should end with an exclamation mark.
* When using `inject` with short blocks, name the arguments `|a, e|`
  (accumulator, element).
* When defining binary operators, name the argument `other`.

    ```Ruby
    def +(other)
      # body omitted
    end
    ```

* Prefer `map` over *collect*, `find` over *detect*, `select` over
  *find_all*, `size` over *length*. This is not a hard requirement; if the
  use of the alias enhances readability, it's ok to use it.

<a name="comments"/>
## Comments

> Good code is its own best documentation. As you're about to add a
> comment, ask yourself, "How can I improve the code so that this
> comment isn't needed?" Improve the code and then document it to make
> it even clearer. <br/>
> -- Steve McConnell

* Write self-documenting code and ignore the rest of this section. Seriously!
* Comments longer than a word are capitalized and use punctuation. Use [one
  space](http://en.wikipedia.org/wiki/Sentence_spacing) after periods.
* Avoid superfluous comments.

    ```Ruby
    # bad
    counter += 1 # increments counter by one
    ```

* Keep existing comments up-to-date. No comment is better than an outdated
  comment.
* Avoid writing comments to explain bad code. Refactor the code to
  make it self-explanatory. (Do or do not - there is no try.)

<a name="annotations"/>
## Annotations

* Annotations should usually be written on the line immediately above
  the relevant code.
* The annotation keyword is followed by a colon and a space, then a note
  describing the problem.
* If multiple lines are required to describe the problem, subsequent
  lines should be indented two spaces after the `#`.

    ```Ruby
    def bar
      # FIXME: This has crashed occasionally since v3.2.1. It may
      #   be related to the BarBazUtil upgrade.
      baz(:quux)
    end
    ```

* In cases where the problem is so obvious that any documentation would
  be redundant, annotations may be left at the end of the offending line
  with no note. This usage should be the exception and not the rule.

    ```Ruby
    def bar
      sleep 100 # OPTIMIZE
    end
    ```

* Use `TODO` to note missing features or functionality that should be
  added at a later date.
* Use `FIXME` to note broken code that needs to be fixed.
* Use `OPTIMIZE` to note slow or inefficient code that may cause
  performance problems.
* Use `HACK` to note code smells where questionable coding practices
  were used and should be refactored away.
* Use `REVIEW` to note anything that should be looked at to confirm it
  is working as intended. For example: `REVIEW: Are we sure this is how the
  client does X currently?`
* Use other custom annotation keywords if it feels appropriate, but be
  sure to document them in your project's `README` or similar.

<a name="classes"/>
## Classes

* Always supply a proper `to_s` method.

    ```Ruby
    class Person
      attr_reader :first_name, :last_name

      def initialize(first_name, last_name)
        @first_name = first_name
        @last_name = last_name
      end

      def to_s
        "#@first_name #@last_name"
      end
    end
    ```

* Use the `attr` family of functions to define trivial accessors or
  mutators.
* Consider adding factory methods to provide additional sensible ways
  to create instances of a particular class.
* Prefer duck-typing over inheritance.
* Avoid the usage of class (`@@`) variables due to their "nasty" behavior
  in inheritance.
* Assign proper visibility levels to methods (`private`, `protected`)
in accordance with their intended usage. Don't go off leaving
everything `public` (which is the default). After all we're coding
in *Ruby* now, not in *Python*.
* Indent the `public`, `protected`, and `private` methods as much the
  method definitions they apply to. Leave one blank line above them.

    ```Ruby
    class SomeClass
      def public_method
        # ...
      end

      private
      def private_method
        # ...
      end
    end

* Use `def self.method` to define singleton methods. This makes the methods
  more resistant to refactoring changes.

    ```Ruby
    class TestClass
      # bad
      def TestClass.some_method
        # body omitted
      end

      # good
      def self.some_other_method
        # body omitted
      end

      # Also possible and convenient when you
      # have to define many singleton methods.
      class << self
        def first_method
          # body omitted
        end

        def second_method_etc
          # body omitted
        end
      end
    end
    ```

<a name="exceptions"/>
## Exceptions

* Don't suppress exceptions.
* Don't use exceptions for flow of control.
* Avoid rescuing the `Exception` class.

<a name="collections"/>
## Collections

* It's ok to use arrays as sets for a small number of elements.
* Prefer `%w` to the literal array syntax when you need an array of
strings.
* Avoid the creation of huge gaps in arrays.
* Use `Set` instead of `Array` when dealing with lots of elements.
* Use symbols instead of strings as hash keys.
* Avoid the use of mutable object as hash keys.
* Use the new 1.9 literal hash syntax in preference to the hashrocket syntax.
* Rely on the fact that hashes in 1.9 are ordered.
* Never modify a collection while traversing it.

<a name="strings"/>
## Strings

* Prefer string interpolation instead of string concatenation:

    ```Ruby
    # bad
    email_with_name = user.name + ' <' + user.email + '>'

    # good
    email_with_name = "#{user.name} <#{user.email}>"
    ```

* Prefer single-quoted strings when you don't need string interpolation or
  special symbols such as `\t`, `\n`, `'`, etc.

    ```Ruby
    # bad
    name = "Bozhidar"

    # good
    name = 'Bozhidar'
    ```

* Don't use `{}` around instance variables being interpolated into a
  string.

    ```Ruby
    class Person
      attr_reader :first_name, :last_name

      def initialize(first_name, last_name)
        @first_name = first_name
        @last_name = last_name
      end

      # bad
      def to_s
        "#{@first_name} #{@last_name}"
      end

      # good
      def to_s
        "#@first_name #@last_name"
      end
    end
    ```

* Avoid using `String#+` when you need to construct large data chunks.
  Instead, use `String#<<`. Concatenation mutates the string instance in-place
  and is always faster than `String#+`, which creates a bunch of new string objects.

    ```Ruby
    # good and also fast
    html = ''
    html << '<h1>Page title</h1>'

    paragraphs.each do |paragraph|
      html << "<p>#{paragraph}</p>"
    end
    ```

<a name="literals"/>
## Percent Literals

* Use `%w` freely.

    ```Ruby
    STATES = %w(draft open closed)
    ```

* Use `%()` for single-line strings which require both interpolation
  and embedded double-quotes. For multi-line strings, prefer heredocs.

    ```Ruby
    # bad (no interpolation needed)
    %(<div class="text">Some text</div>)
    # should be '<div class="text">Some text</div>'

    # bad (no double-quotes)
    %(This is #{quality} style)
    # should be "This is #{quality} style"

    # bad (multiple lines)
    %(<div>\n<span class="big">#{exclamation}</span>\n</div>)
    # should be a heredoc.

    # good (requires interpolation, has quotes, single line)
    %(<tr><td class="name">#{name}</td>)
    ```

* Use `%r` only for regular expressions matching *more than* one '/' character.

    ```Ruby
    # bad
    %r(\s+)

    # still bad
    %r(^/(.*)$)
    # should be /^\/(.*)$/

    # good
    %r(^/blog/2011/(.*)$)
    ```

* Avoid `%q`, `%Q`, `%x`, `%s`, and `%W`.

* Prefer `()` as delimiters for all `%` literals.

<a name="misc"/>
## Misc

* Write `ruby -w` safe code.
* Avoid hashes as optional parameters. Does the method do too much?
* Avoid methods longer than 10 LOC (lines of code). Ideally, most methods will be shorter than
  5 LOC. Empty lines do not contribute to the relevant LOC.
* Avoid parameter lists longer than three or four parameters.
* If you really have to, add "global" methods to Kernel and make them private.
* Use class instance variables instead of global variables.

    ```Ruby
    #bad
    $foo_bar = 1

    #good
    class Foo
      class << self
        attr_accessor :bar
      end
    end

    Foo.bar = 1
    ```

* Avoid `alias` when `alias_method` will do.
* Use `OptionParser` for parsing complex command line options and
  `ruby -s` for trivial command line options.
* Write for Ruby 1.9. Don't use legacy Ruby 1.8 constructs.
    * Use the new JavaScript literal hash syntax.
    * Use the new lambda syntax.
    * Methods like `inject` now accept method names as arguments.

      ```Ruby
      [1, 2, 3].inject(:+)
      ```

* Avoid needless metaprogramming.

<a name="design"/>
## Design

* Code in a functional way, avoiding mutation when that makes sense.
* Do not mutate arguments unless that is the purpose of the method.
* Do not mess around in core classes when writing libraries. (Do not monkey
  patch them.)
* [Do not program
  defensively.](http://www.erlang.se/doc/programming_rules.shtml#HDR11)
* Keep the code simple and subjective. Each method should have a single,
  well-defined responsibility.
* Avoid more than three levels of block nesting.
* Don't overdesign. Overly complex solutions tend to be brittle and hard to
  maintain.
* Don't underdesign. A solution to a problem should be as simple as
  possible, but no simpler than that. Poor initial design can lead to a lot
  of problems in the future.
* Be consistent. In an ideal world, be consistent with these guidelines.
* Use common sense.

<a name="contributing"/>
# Contributing

Nothing written in this guide is set in stone. It's my desire to work
together with everyone interested in Ruby coding style, so that we could
ultimately create a resource that will be beneficial to the entire Ruby
community.

Feel free to open tickets or send pull requests with improvements. Thanks in
advance for your help!

<a name="spreadtheword"/>
# Spread the Word

A community-driven style guide is of little use to a community that
doesn't know about its existence. Tweet about the guide, share it with
your friends and colleagues. Every comment, suggestion or opinion we
get makes the guide just a little bit better. And we want to have the
best possible guide, don't we?




  [0]: http://www.python.org/dev/peps/pep-0008/
  [1]: https://github.com/bbatsov/rails-style-guide
  [2]: http://pragprog.com/book/ruby3/programming-ruby-1-9
  [3]: http://www.amazon.com/Ruby-Programming-Language-David-Flanagan/dp/0596516177
  [4]: https://github.com/TechnoGate/transmuter