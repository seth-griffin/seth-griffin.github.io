---
layout: post
title:  "Introduction to the Arc Programming Language"
date:   2025-12-11 13:01:00 -0600
categories: programming functional-programming lisp scheme arc
---

# Introduction to the Arc Programming Language

Arc is a lisp dialect built on top of Racket (formerly scheme)

## Quick start 

### Installation


{% highlight bash %}
sudo apt install racket
{% endhighlight %}

Create a working directory for racket

{% highlight bash %}
wget www.arclanguage.org/arc3.2.tar .
tar -xf arc3.2.tar
{% endhighlight %}

# Run the command line interpreter

{% highlight bash %}
cd arc3.2
racket -f as.scm 
{% endhighlight %}

Language core library is contained in arc.arc

## Useful commands

{% highlight bash %}
>(system "clear") ; clear the racket terminal returning to a single prompt
{% endhighlight %}

{% highlight bash %}
>(quit) ; exit the arc shell and return to the terminal 
{% endhighlight %}

# Language Overview

A comment is prefixed with a semi-colon

{% highlight bash %}
; This is a comment
{% endhighlight %}

## Hello, World!

The function prn means print

{% highlight bash %}
(prn "Hello, World!")
Hello, World!
"Hello, World!"
{% endhighlight %}

## Arithmetic

LISP uses something like reverse polish notation. All evaluation is done from the inside out and each operator is treated like a function

{% highlight bash %}
(+ 3 6 11)
{% endhighlight %}

{% highlight bash %}
(* 3 6 (+ 3 11))
{% endhighlight %}

increment and decrement operators are ++ and --, respectively

{% highlight bash %}
(list 1 5 2 1)
{% endhighlight %}

## Function definitions

Return the product of an argument multiplied by 2

{% highlight bash %}
(def foo (arg) (* arg 2))
#<procedure: foo>
{% endhighlight %}

Average two numbers and display the arguments passed

{% highlight bash %}
(def average (x y)
    (prn "My arguments were: " (list x y))
    (/ (+ x y) 2))
{% endhighlight %}

## Logic

Logic evaluates to true (t) or nil (false) nil is also null.

if the number is odd output the first quoted item, else output the second

A single quote ' is syntactic sugar for quote, therefore 'a is the same as (quote a). It just means don't evaluate.

{% highlight bash %}
(if (odd 1) 'a 'b)

(if (odd 2) 'a 'b)
{% endhighlight %}

An if statement with more than three arguments is equivilent to a nested if

{% highlight bash %}
(if a b c d e)
{% endhighlight %}

is the same as

{% highlight bash %}
(if a
    b
    (if c
        d
        e))
{% endhighlight %}

Use do if you want to do multiple things based on the outcome of a test

{% highlight bash %}
(do (prn "hello")
    (+ 2 3))
{% endhighlight %}

Evaluate several expressions when some condition is true

{% highlight bash %}
(if a
    (do b
        c))
{% endhighlight %}

Use a when expression for this type of situation

{% highlight bash %}
(when a 
    b
    c)
{% endhighlight %}

Conditional logic uses and / or not or ! is expressed as ```no``` ```is``` tests for equivalence as does == in other languages

{% highlight bash %}
(and nil
    (pr "You'll never see this"))
{% endhighlight %}

{% highlight bash %}
(or (is x y) (is x z))
{% endhighlight %}

{% highlight bash %}
(def mylen (xs)
    (if (no xs)
        0
        (+ 1 (mylen (cdr xs)))))
{% endhighlight %}

{% highlight bash %}
(is 'a 'a) 

> t
{% endhighlight %}

{% highlight bash %}
(is 'a 'b)

> nil
{% endhighlight %}

in is used to test if a value is in a list

{% highlight bash %}
(let x 'a
    (in x 'a 'b 'c))
> t
{% endhighlight %}

case ; switch structures are available as well

{% highlight bash %}
(def translate (sym)
    (case sym
        apple 'mela
        onion 'cipolla
              'che?))

> (translate 'apple)
> mela
> (translate 'onion)
> cipolla
> (translate 'else)
> che?
{% endhighlight %}

## Flow Control 

; Basic for loop to print a numeric range 1 - 10

{% highlight bash %}
(for i 1 10
    (pr i " "))
{% endhighlight %}

each iterates through the elements of a list or string, nil will commonly follow since it's the return value of the iteration / flow control expressions

{% highlight bash %}
(each x '(a b c d e)
    (pr x " "))

> a b c d e nil
{% endhighlight %}

while

{% highlight bash %}
(let x 10
    (while (> x 5)
        (= x (- x 1))
        (pr x)))

> 98765nill
{% endhighlight %}

; There is also a loop operator (similar to for in C) and repeat for doing something n times

{% highlight bash %}
(repeat 5 (pr "la "))
{% endhighlight %}

## Repetitive Operations without loops / flow-control

### Map

map function takes a function list and returns the result applying the function to successive elements

{% highlight bash %}
(map (fn (x) (+ x 10)) '(1 2 3))
> (11 12 13)
{% endhighlight %}

map will continue until the shorter of the two lists runs out

{% highlight bash %}
(map + '(1 2 3 4) '(100 200 300))
> (101 202 303)
{% endhighlight %}

functions with one argument are very comon, Arc has a special notation for such cases

{% highlight bash %}
(map [+ _ 10] '(1 2 3))
{% endhighlight %}

The _ is a place holder for an anonymous / argument

Composed functions are shorthand 

{% highlight bash %}
(foo:bar x y) is equivalent to (foo (bar x y))
(map odd:car '((1 2) (4 5) (7 9)))
{% endhighlight %}

negate a function with a leading tilde ~ (same as !func(args) in a C style language)

{% highlight bash %}
(map ~odd '(1 2 3 4 5))
{% endhighlight %}

There are other functions like map that apply functions to successive elements of a sequence. The most common is keep which returns the elements satisfying some test

Ex:

{% highlight bash %}
(keep odd '(1 2 3 4 5 6 7))
>(1 3 5 7)
{% endhighlight %}

 * keep - filters by selecting for the first parameter / test
 * rem - opposite of keep
 * all - returns true if the function evaluates true for every list element
 * some - returns true if the function evaluates true for any list element
 * pos - returns the position of the first element for which the function evaluates true
 * trues - returns a list of all the non nil return values

## Data Structures

### Hash tables

Define hash table airports

{% highlight bash %}
(= airports (table))
{% endhighlight %}

Set an element

(= (airports "Boston") 'bos)

Abbreviated form does not require grouping arguments or key quoting

{% highlight bash %}
(let h (obj x 1 y 2)
    (h 'y))

> 2
{% endhighlight %}

{% highlight bash %}
(= codes (obj "Boston 'bos "San Francisco 'sfo "Paris" 'cdg))
>#hash(("Boston . bos) ("Paris" . cdg) ("San Francisco" . sfo))
{% endhighlight %}

func keys shows hash keys

{% highlight bash %}
(keys codes)
> ("San Francisco" "Boston" "Paris")
{% endhighlight %}

func vals shows hash values

{% highlight bash %}
(vals codes)
> (sfo bos cdg)
{% endhighlight %}

# Stack

Operate on lists as stacks using push and pop

{% highlight bash %}
(= x '(c a b))
> (c a b)
{% endhighlight %}

{% highlight bash %}
(pop x)
> c
{% endhighlight %}

{% highlight bash %}
x
> (a b)
{% endhighlight %}

{% highlight bash %}
(push 'f x)
> (f a b)
{% endhighlight %}

{% highlight bash %}
x
> (f a b)
{% endhighlight %}

can be used directly on structures and not just on variables

{% highlight bash %} (push 'f x){% endhighlight %} is just as valid as {% highlight bash %} (push 'l (cdr x)) {% endhighlight %} which would yield

{% highlight bash %}
> (l a b)
{% endhighlight %}

{% highlight bash %}
x
(f l a b)
{% endhighlight %}

### Maptable

maptable is like map for lists except that it returns the original table instead of a new one

{% highlight bash %}
(maptable (fn (k v) (prn v " " k))
    codes)
> sfo San Francisco
bos Boston
cdg Paris
{% endhighlight %}

{% highlight bash%}
#hash(("Boston" . bos) ("Paris" . cdg) ("San Francisco" . sfo))
{% endhighlight %}

### Alists - association lists or alists

Association lists or alists offer benefits over hash tables
    * Sorting
    * Can be built incrementally using recursion
    * Can share the same tail and preserve old values

The same code hash table can be represented as an alist as follows:

{% highlight bash %}
(= codes '(("Boston" bos) ("Paris" cdg) ("San Francisco: sfo)))
> (("Boston" bos) ("Paris" cdg) ("San Francisco" sfo))
{% endhighlight %}

func alref returns the first value corresponding to a key in an alist

{% highlight bash %}
(alref codes "Boston"(
bos
{% endhighlight %}

### String operations 

string - string building

{% highlight bash %}
(string 99 " bottles of " 'bee #\r)
> "99 bottles of beer"
{% endhighlight %}

tostring

{% highlight bash%}
(tostring
    (prn "domesday")
    (prn "book))
> "domesday\nbook\n"
{% endhighlight %}

### Predicates

type is used to find types of things, coerce is used to cast, as far as I can tell

{% highlight bash %}
map (type (list 'foo 23 23.5 '(a) nil car "foo" #\a))
> (sym int num cons sym fn string char)
{% endhighlight %}

coerce is used to automatically convert a data type from one to another similar to an autocast

{% highlight bash %}
(coerce #\A 'int)
> 65
{% endhighlight %}

{% highlight bash %}
(coerce "foo" 'cons)
> (#\f #\o #\o)
{% endhighlight %}

{% highlight bash %}
(coerce "99" 'int)
> 99
{% endhighlight %}

{% highlight bash %}
(coerce "99" 'int 16)
> 153
{% endhighlight %}

## Algorithms

sort returns a copy of a sorted sequence

{% highlight bash %}
(sort < '(2 9 3 7 5 1))
> (1 2 3 5 7 9)
{% endhighlight %}

{% highlight bash %}
(= x '(2 9 3 7 5 1))
> (2 9 3 7 5 1
{% endhighlight %}

zap changes something to the return result of any function
{% highlight bash %}
(zap [sort < _] x)
>(1 2 3 5 7 9)
{% endhighlight %}

insort modifies a sorted list by inserting a new element at the right place

{% highlight bash %}
(insort < 4 x)
> (1 2 3 4 5 7 9)
{% endhighlight %}

sorting according to some property other than value

example of sorting strings from least to most chars

{% highlight bash %}
(sort (fn (x y) (< (len x) (len y)))
    '("orange" "pea" "apricot" "apple"))
> ("pea" "apple" "orange" "apricot")
{% endhighlight %}

In arc sort is stable meaning elements judged equal by the comparator will not have their ordinal changed

{% highlight bash %}
(sort (fn (x y) (< (len x) (len y)))
    '("aa" "bb" "cc"))
> ("aa" "bb" "cc")
{% endhighlight %}

## Advanced Functions

Optional function arguments are prefixed with o as in (o x)

{% highlight %}
(def greet (name (o punc))
> #<procedure: greet>
(greet 'joe)
> "hello joe"
(greet 'joe #\!)
> "hello joe!"
{% endhighlight %}

By default optional parameters default to nil. Expressions following a parameter will be evaluated to produce a default value as in:

{% highlight bash %}
(def greet (name (o punc (case name who #\? #\!)))
    (string "hello " name punc))

*** redefining greet
#<procedure: greet>
(greet 'who)
> "hello who?"
(greet 'joe)
> "hello joe!"
{% endhighlight %}

variable number of arguments are supported using a period and a space before the last parameter

{% highlight bash %}
(def foo (x y . z)
    (list x y z))
(foo (+ 1 2) (+ 3 4) (+ 5 6) (+ 7 8))
> (3 7 (11 15))
{% endhighlight %}

apply is used to supply arguments as a single list

{% highlight bash %}
(apply + '(1 2 3))
> 6
{% endhighlight %}

Remember our two argument average function? We can use apply to give it a variable number of numbers for a more useful / powerful average function

{% highlight bash %}
(def average args
    (/ apply + argS) (len args)))
#<procedure: average>
(average 1 2 3)
> 2
{% endhighlight %}

The above capabilities represent sufficient knowledge to start writing macros

## Macros

Macros, in lisp, generate generate code by expanding at runtime.

Consider the following example:

{% highlight bash %}
(list '+ 1 2)
> (+ 1 2)
{% endhighlight %}

{% highlight bash %}
(mac foo () 
    (list '+ 1 2))
> #(tagged mac #<procedure: foo>)
(+ 10 (foo))
> 13
{% endhighlight %}

Macros:
    * use mac instead of def
    * Expand in place where their symbol is found
    * e.g.: (foo), as in, (+ 10 (foo)) becomes (+ 1 2)

Let's write a more useful macro that takes some arguments

{% highlight bash %}
(mac when (test . body)
    (list 'if test (cons 'do body)))
*** redefining when
#3(tagged mac #<procedure>)
(when 1 (pr "hello ") 2)
> hello 2
{% endhighlight %}

Macro when takes an arbitrary test as an argument and when it evaulates to true executes the code passed as body.

using a back-tick ` stops evaluation until a comma , is encountered

{% highlight bash %}
`(a b c)
(let x 2
    `(a , x c))
> (a 2 c)
{% endhighlight %}

backquoted expressions have holes in them and are partially evaluated

Consider the following

{% highlight bash %}
(let x '(1 2)
    `(a, @x c))
> (a 1 2 c)
{% endhighlight %}

The ,@ in front of anything in a backquoted expression causes the list to be spliced into whatever list it appears in. 

The above example makes it easy to see the effect.

If the need arises use the uniq function to generate symbol names during macro expansion

{% highlight bash %}
(uniq)
> gs1722
(uniq)
> gs1723
{% endhighlight %}

An example where uniq is needed to avoid some very nasty bugs follows:

{% highlight bash %}
(mac repeat (n . body)
    `(for x 1 ,n ,@body))
> #3(tagged mac #<procedure>)
(repeat 3 (pr "blub"))
blub blub blub nil
{% endhighlight %}

{% highlight bash %}
(let x "blub "
    (repeat 3 (pr x)))

123nil
{% endhighlight %}

This happens because the expansion is:

{% highlight bash %}
(let x "blub "
    (for x 1 3 (pr x)))
{% endhighlight %}

There's an ambiguous symbol / colission in the expansion that occurs within the scope. 

To avoid rewrite the macro using uniq

{% highlight bash %}
(mac repeat (n . body)
    `(for ,(uniq) 1 ,n ,@body))
{% endhighlight %}

To use uniq more than once use w/uniq.

Note: w/ is a macro in racket that reads and skips whitespace and linebreaks until a non-whitespace character or second end-of-line is found.

## Web applications

A quick hello world web app

{% highlight bash %}
(defop hello req (pr "hello world"))
> #<procedure: gs1731>
(asv)
> ready to serve port 8080
{% endhighlight %}

Additional examples of defops that can be stored for a web application:

{% highlight bash %}
(defop hello2 req
    (w/link (pr "there")
        (pr "here")))

(defop hello3 req
    (w/link (w/link (pr "end")
        (pr "middle"))
    (pr "start")))

(defop hello 4 req
    (aform [w/link (pr "you said " (arg _ "foo"))
            (pr "click here")
        (input "foo")
        (submit)))
{% endhighlight %}

## Advanced Reading

There is a sample application in blog.arc that can be read for more ideas about how to make more functional web applications

arc.arc defines core functionality and can be studied to learn more about what arc can do out of the box

The following are some of the simpler functions included

{% highlight bash %}
(def cadr (xs)
    (car (cdr xs)))

(def no (x)
    (is x nil))

(def list args
    args)

(def isa (x y)
    (is (type x) y))

(def firstn (n xs)
  (if (and (> n 0) xs)
      (cons (car xs) (firstn (- n 1) (cdr xs)))
      nil))

(def nthcdr (n xs)
    (if (> n 0)
        (nthcdr (- n 1) (cdr xs))
        xs))

(def tuples (xs ( o n 2))
    (if (no xs)
        nil
        (cons (firstn n xs)
        (tuples (nthcdr n xs) n))))

(def trues (f seq)
    (rem nil (map f seq)))

(mac unless (test . body)
    `(if (no, test) (do ,@body)))

(mac n-of (n expr)
    (w/uniq ga
    `(let ,ga nil
        (repeat ,n (push ,expr ,ga))
        (ref ,ga))))
{% endhighlight %}

Condensed from 

<a href="https://arclanguage.github.io/tut-stable.html">https://arclanguage.github.io/tut-stable.html</a>
