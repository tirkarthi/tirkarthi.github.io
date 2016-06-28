---
layout: post
title:  Delicious maps and filters
date:   2016-06-28 20:14:29 +0530
categories: clojure
---

The post assumes some experience with Python but most of the code looks like pseudocode only. The corresponding clojure code is also presented and can be evaluated live by changing the code. First let me explain some basic clojure code and its fairly easy to pick up

{% highlight clojure %}

;; let city be "paris". Its associating a name with a value.
(def city "paris")

;; A vector of numbers with fast access by index. Like arrays but implemeted differently under the hood
[1 2 3 4]

;; A list of numbers similar to a linked list
'(1 2 3 4)

;; A dictionary or hash like {name: "kate", city: "paris"}
{"name" "kate", "city" "paris"}

;; a set of strings and hence it cannot contain duplicates
#{"kate" "kim" "john"}

;; define a new function called say-hello that takes an argument of name and prints a "Hello ," name
(defn say-hello [name]
  (println "Hello ," x))

;; Most clojure code is of the form (function-name arguments)
;; Here you call say-hello with the argument "kate"
(say-hello "kate")

;; Clojure follows prefix notation so its actually applying + function with the arguments 3 4 5
(+ 3 4 5) ;; returns 12

;; Clojure follows prefix notation so its actually applying * function with the arguments 3 4 5
(* 3 4 5) ;; returns 60

;; Define a function that returns square of the given number
(defn square [number]
      (* number number))

(square 5) ;; gives 25

{%  endhighlight %}

For sake of this post lets pick up a problem where and solve it with both Python and Clojure.

# Simple problem

List down all the numbers from 1 to 30 whose square is an odd number

So lets take it step by step

- We generate a list of all numbers less than 10
- Calculate the square of each number
- Check if the square is an odd number and if so add it to the list

{% highlight python %}

odd_squares = []

for number in range(1, 31):
    if (number * number) % 2 != 0:
        odd_squares.push(number * number)

{% endhighlight %}

Here we are directly squaring things up and checking if the number is an odd one. Suddenly if want to change
it so that all the odd cubes are listed then we have to go around and add the code to cube and along with that
we also need to see the part where the odd number is checked which adds up some cognitive load when and as more
logic is added we get a sense of dizziness as we touch the function as we need to look at the whole code. So we
refactor them and name them properly and we will see that the code becomes more clear.

{% highlight python %}

odd_squares = []

def square(number):
    return number * number

def is_odd(number):
    return number % 2 != 0

for number in range(1, 31):
    squared = square(number)
    if is_odd(squared):
        odd_squares.push(square)

{% endhighlight %}

# Maps and filters

Map and filter are basic functional programming constructs that are available under different names in different languages.
Map is a function that takes a function and a list and applies the function every member of the list and returns the results
as a list. Filter does the same thing except that it returns the members that return true for the function. So lets rewrite
our programs with maps and functions. They seem like loops but the important distinction is that they functions as parameters.
Hence these functions that take other functions as parameters and operate on them are higher-order functions. lambda is used
to construct anonymous functions in Python that is functions that don't have a name and are of short term use.

{% highlight python %}

# similar to the clojure code with anonymous functions
square = lambda number: number * number

square(5) # returns 25

odd_squares = filter(lambda x: x % 2 != 0, map(lambda x: x * x, range(1, 31)))

{% endhighlight %}

Whoa! that seems like lot of few code and rewriting them with named functions makes them much more elegant. We come to
see that it pretty much reads like english. map resmebles to mathematical term where we map the list on a function in our
classic set theory classes. These abstractions help us in transforming the code from telling the computer to do things to
saying the properties and letting the computer figure out the looping constructs for us.

{% highlight python %}

def square(number):
    return number * number

def is_odd(number):
    return number % 2 != 0

odd_squares = filter(is_odd, map(square, range(1, 31)))

{% endhighlight %}

Well we wrote some elegant Python but where does Clojure fit in this thing. We will see that these constructs are taken from Lisp
and Clojure offers us much more flexibility in writing better code for the given problem with more abstractions. We will take it from here
by writing code with map and filter and see how we can make it more elegant with macros.

{% highlight clojure %}

(defn square [number]
  (* number number))

;; Clojure has inbuilt odd? function to check if its odd
;; Clojure allows question marks and hypens in names
(def odd-squares (filter odd? (map square (range 1 31))))

{% endhighlight %}

Well that pretty much reads like Python except with more parens that are slightly annoying visually and also as you can see our code scales horizontally with the number of constructs added to it. Thats a bad thing since code mostly involves filtering and mapping contiuously throughout the loops and writing 4-5 such constructs make it more tedious to read and adds up much more mental cognition.
Also the code reads from right to left which also adds some load as we have to hop back and forth between the parens with to keep track
of the whole thing

# Macros to the rescue!

*"Lisp is a programmable programming language." - John Foderaro, CACM, September 1991*

Macros add more syntactic abstraction and takes it down to the next dimension. Since the whole thing is a pipeline won't it be nice to have a pipeline type of operator where we can define our own constructs not functions our own syntactic constructs and operators. Lets see
how that boils down to things.

{% highlight clojure %}

(defn square [number]
  (* number number))

(->> (range 1 31)
     (map square)
     (filter odd?))

{% endhighlight %}

This whole thing seems like black magic what is that double arrow thingy functions cannot have those symbols so are they operators on their own? This leads us to add more and more filters and transformations down the pipeline and the extra advantage is that we can just swap the functions and build a new pipeline without worrying about the whole constructs. Lets say we need to change the function to return even cubes well its more easy to do so. We can swap the map and filter functions and we have a new pipeline on our hands

{% highlight clojure %}

(defn square [number]
  (* number number))

(defn cube [number]
  (* (square number) number))

(->> (range 1 31)
     (map cube)
     (filter even?))

{% endhighlight %}
