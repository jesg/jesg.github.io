---
layout: post
title:  "A Little Lisp"
date:   2014-04-08 12:00:00
categories: lisp
---

Create a subset of Scheme in 100 lines of ruby.

Originally this post was going to be written in java but it ended up being to verbous.  The java implementation can be found on [github](https://github.com/jesg/jschemer).

Scheme Primitives
-----------------

<table border="1">
<tr><th>Expression</th><th>Sematics and Example</th></tr>
<tr><td>var</td><td>Variable name that resolves to a value<br/>Example: x</td></tr>
<tr><td>number</td><td>Constant literal<br/>Example: 2 or 3.14</td></tr>
<tr><td>(quote exp)</td><td>Return the expression without evaluating it<br/>Example: (quote (z r q)) -> (z r q)</td></tr>
<tr><td>(if test conseq alt)</td><td>evaluate test; return evaluated conseq if true; return evaluated alt if false<br/>Example: (if (eq 1 1) (+ 1 2) (+ 1 3)) -> 3</td></tr>
<tr><td>(set! var exp)</td><td>Evaluate exp and assign to var<br/>Example: (set! y (* 2 x))</td></tr>
<tr><td>(define var exp)</td><td>Evaluate the expression exp and assign it to var<br/>Example: (define pi 3.14) or (define inc (lambda (x) (+ x 1)))</td></tr>
<tr><td>(lambda (var...) exp...)</td><td>A function that takes parameter(s) var... and evaluates exp... returning the result of the last expression<br/>Example: (lambd (x) (+ x 1))</td></tr>
<tr><td>(proc exp...)</td><td>proc is anything other than if, set!, define, lambda, quote.  All the exp... are evaluated then the function is called.<br/>Example: (inc 1) -> 2</td></tr>
</table>
<br/>


## Read-Evaluate-Print Loop (REPL)

-----------------------------------------------
1. An interpeter arses the program into a syntax tree based on rules.  The parser is implemented with a function called `parse`.  Normally there will be a lexer that splits the input into tokens and a parser that creates a syntax tree from the tokens.
2. The interpeter evaluates the syntax tree according to the semantic rules described above.  The function `eval` executes the code.

------------------------------------------------

{% highlight text %}

>>> parse(lexer("(+ 2 2)"))
=> ['+', 2, 2]

>>> eval(parse(lexer("(+ 2 2)")))
=> 4
{% endhighlight %}


Evaluate
--------


Ruby already has an eval function so we create our eval function in a module we create called `Lisp`.  The `eval` function evaluates the expression in an environment.  The environment maps variable names to values.

{% highlight ruby %}
def Lisp.eval(x, env)
  if x.instance_of? String
    return env.find(x)[x]
  elsif not x.instance_of? Array
    return x
  elsif x.empty?
    return x
  elsif 'quote' == x.first
    (_, exp) = x
    return exp
  elsif 'if' == x.first
    (_, test, conseq, alt) = x
    return Lisp::eval((Lisp::eval(test, env) ? conseq : alt), env)
  elsif 'set!' == x.first
    (_, var, exp) = x
    env.find(var)[var] = Lisp::eval(exp, env)
  elsif 'define' == x.first
    (_, var, exp) = x
    env[var] = Lisp::eval(exp, env)
  elsif 'lambda' == x.first  # (lambda (arg1 arg2...) exp1 exp2...)
    (_, vars) = x
    return proc do |args|
      result = []
      local_env = Lisp::Env.new(vars, args, env)
      x.slice(2..-1).each do |exp|
        result = Lisp::eval(exp, local_env)
      end
      result
      end
  else
    exps = x.collect { |exp| Lisp::eval(exp, env) }
    proc = exps.shift
    return proc.call(*exps)
  end
end

class Lisp::Env < Hash
  def initialize(params=[], args=[], outer=nil)
    merge params.zip(args).to_h
    @outer = outer
  end

  def find(var)
    key?(var) ? self : @outer.find(var)
  end
end
{% endhighlight %}

Now we need to define some basic functions.  Note that `car`, `cdr`, `cons`, and `eq` are all primitive functions in scheme.  Next we define basic arithmetic with ruby's `send` method.

{% highlight ruby %}
global_env = Lisp::Env.new
global_env['car'] = proc {|x| x.last}
global_env['cdr'] = proc {|x| x.slice(0..-2)}
global_env['cons'] = proc {|head, tail| tail.push(head) }
global_env['eq'] = proc {|x, y| x == y }
['+', '-', '*', '/', '%'].each do |msg|
  global_env[msg] = proc {|x, *y| x.send msg, *y}
end
{% endhighlight %}

Note that `car`, `cdr`, and `cons` all work on the end of an array.

Parsing
-------

Parsing involves two stages the lexer that breaks the input into tokens and the parser that builds a syntax tree from the tokens.

We will build the lexer in ruby with the `split` method to break up a string by white space.

{% highlight ruby %}
def Lisp.lexer(str)
  tokens = Array.new
  str
    .gsub(/\(/, ' ( ')
    .gsub(/\)/, ' ) ')
    .split(/\s/)
    .each do |token|
      next if token.empty?
      tokens << case
        when token =~ /^\d+$/
          token.to_i
        when token =~ /^\d+\.\d+$/
          token.to_f
        else
          token
        end
    end
  tokens
end
{% endhighlight %}

The parser will take the tokens from the lexer and produce a bunch of arrays.

{% highlight ruby %}
def Lisp.parse(tokens)
  first = tokens.shift
  if first =~ /\(/
    ast = Array.new
    until tokens.first =~ /\)/
      ast << parse(tokens)
    end
    tokens.shift
    return ast
  else
    return first
  end
end
{% endhighlight %}

REPL
---

The `repl` function will be our interactive Lisp interpreter.

{% highlight ruby %}
def Lisp.repl(env, cursor='> ')
  loop do
    print cursor
    result = Lisp::eval(Lisp::parse(Lisp::lexer(gets())), env)
    puts(result.to_s)
  end
end
{% endhighlight %}

To run the interpreter execute `ruby lisp.rb`.

The full source is available on [gist](https://gist.github.com/jesg/10146611).


