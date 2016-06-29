---
layout: post
title:  Demystifying range
date:   2016-06-29 20:14:29 +0530
categories: clojure
---

*tl;dr The whole exercise of stepping into code can be done with the help of Cursive plugin and IntelliJ IDEA but this post is to share the experience in modifying the code and rebuilding things*

Digging up the source code is always an interesting way to learn how things really work and also gives us a better understanding of the underlying methods. In this post I decided to study the range method to see how its implemented under the hood

# Requirements
- Download or clone the clojure source code from [Github](https://github.com/clojure/clojure).
- Install maven as we will be using it as the build tool.
- Run the command as below which skips tests as they increase the build time.
{% highlight text %} mvn package -Dmaven.test.skip=true {% endhighlight %}
- It will download all the necessary stuff and generate a jar file under the folder target as target/clojure-1.9.0-master-SNAPSHOT.jar.
- Start the repl with the command
{% highlight text %} java -jar target/clojure-1.9.0-master-SNAPSHOT.jar {% endhighlight %}
- Open a new shell session with the source code. I recommend [Ack](http://beyondgrep.com) or [Sliver searcher](https://github.com/ggreer/the_silver_searcher) if you have cloned the repo as they offer more speed since we will be doing a lot of grepping (Pun intended)

We are all set to go. You can view the docs for the range function as below

{% highlight clojure %}

user=> (doc range)
-------------------------
clojure.core/range
([] [end] [start end] [start end step])
  Returns a lazy seq of nums from start (inclusive) to end
  (exclusive), by step, where start defaults to 0, step to 1, and end to
  infinity. When step is equal to 0, returns an infinite sequence of
  start. When start is equal to end, returns empty list.

{% endhighlight %}

Its pretty much Python's range function returning the range of numbers with the end excluded. So lets search for the code of range. It can be easily found out with source function

{% highlight clojure %}

user=> (source range)
(defn range
  "Returns a lazy seq of nums from start (inclusive) to end
  (exclusive), by step, where start defaults to 0, step to 1, and end to
  infinity. When step is equal to 0, returns an infinite sequence of
  start. When start is equal to end, returns empty list."
  {:added "1.0"
   :static true}
  ([]
   (iterate inc' 0))
  ([end]
   (if (instance? Long end)
     (clojure.lang.LongRange/create end)
     (clojure.lang.Range/create end)))
  ([start end]
   (if (and (instance? Long start) (instance? Long end))
     (clojure.lang.LongRange/create start end)
     (clojure.lang.Range/create start end)))
  ([start end step]
   (if (and (instance? Long start) (instance? Long end) (instance? Long step))
     (clojure.lang.LongRange/create start end step)
     (clojure.lang.Range/create start end step))))
nil

{% endhighlight %}

We see that range without arguments generates a lazy list of numbers. This deserves another post. Lets look at the other scenarios wherein its common to specify the start, end and step value. It checks if they are all of Long instance to create a Long range or else it creates a normal range with create method. Now lets look for the LongRange class.

{% highlight shell-session %}

➜  ack -inr 'LongRange' *

clj/clojure/core.clj
2996:     (clojure.lang.LongRange/create end)
3000:     (clojure.lang.LongRange/create start end)
3004:     (clojure.lang.LongRange/create start end step)

jvm/clojure/lang/LongRange.java
21:public class LongRange extends ASeq implements Counted, IChunkedSeq, IReduce {
55:private LongRange(long start, long end, long step, BoundsCheck boundsCheck){
62:private LongRange(long start, long end, long step, BoundsCheck boundsCheck, LongChunk chunk, ISeq chunkNext){
71:private LongRange(IPersistentMap meta, long start, long end, long step, BoundsCheck boundsCheck, LongChunk chunk, ISeq chunkNext){
# Some results are hidden

{% endhighlight %}

We go to the file LongRange.java and we find that there is a LongRange class and it has the constructors made private and also has the create method to return us the instances of the LongRange method. It has an interesting constant of CHUNK_SIZE which is the amount of elements to be evaluated. It implements several interfaces like and also has several static classes inside the LongRange class. Now lets change the create method to insert a simple print statement with "Hello" and run the command {% highlight java %} mvn package -Dmaven.test.skip=true {% endhighlight %} Exit the repl and start it agin with the new clojure jar under target folder. Generate a range and you will see that the "Hello" is printed.

{% highlight clojure %}

user=> (range 10)
Hello
(0 1 2 3 4 5 6 7 8 9)

{% endhighlight %}

Things are getting along well except the build times took around 15 seconds for me on my AWS t2.micro instance so each build is a little costly. The create method is overloaded to call the respective overloaded constructors for the number of given arguments. In some cases where it doesn't make sense to have the end less than or equal to start with step negative it returns PersistentList.EMPTY. We see a method called forceChunk where its made sure that the range doesn't overflow. It generates the current chunk of 32 elements and also creates a new LongRange for the next with end of the chunk as start till the upper limit. It creates an intermediate object. This is used in the next method where the dropFirst returns another LongRange with the first element removed and we also store the LongRange generated at forceCHunk returning the whole. My explanation is a litle confusing but the code is straightforward as it can be seen from the methods.

Change the code by adding print statement to the methods and rebuild them to see the whole method flow. I picked up the function nth to see how {% highlight clojure %} (nth (range 10) 2) {% endhighlight %} is done as we generate a LongRange. I added a print statement to overloaded nth functions and rebuilt them to see if they actually return the nth item in the range. Sadly they didn't but they seem to implement the necessary logic to return the nth element with multiplication but it doesn't print. Guess we have to go to the nth function in clojure's core.clj again. 

{% highlight clojure %}

user=> (source nth)
(defn nth
  "Returns the value at the index. get returns nil if index out of
  bounds, nth throws an exception unless not-found is supplied.  nth
  also works for strings, Java arrays, regex Matchers and Lists, and,
  in O(n) time, for sequences."
  {:inline (fn  [c i & nf] `(. clojure.lang.RT (nth ~c ~i ~@nf)))
   :inline-arities #{2 3}
   :added "1.0"}
  ([coll index] (. clojure.lang.RT (nth coll index)))
  ([coll index not-found] (. clojure.lang.RT (nth coll index not-found))))

{% endhighlight %}

We find clojure.lang.RT.java file in the same way we found the LongRange.java file. As we go there we find the source for nth which is called with the java interop syntax. The source is straightforward it checks if the collection is an instance of Indexed. Since LongChunk implemented IChunk which inherits from Indexed interface it should be called I hoped and added a print statement there to find that its not so. It must go to the next statement nthFrom then. I added a print inside the else to find that this is the correct method. Next we move on to the inner call Util.ret1. I looked into the Util.java file to get the ret1 method which simply returns the object. This must be strange why does it just return the object with an extra layer of indirection and I found that there are close to 3000+ places where this method is called. Lets just look at the nthFrom function implementation.

{% highlight java %}

# RT.java

static public Object nth(Object coll, int n){
	if(coll instanceof Indexed)
		return ((Indexed) coll).nth(n);
	return nthFrom(Util.ret1(coll, coll = null), n);
}

# Util.java

static public Object ret1(Object ret, Object nil){
		return ret;
}

static public ISeq ret1(ISeq ret, Object nil){
		return ret;
}

{% endhighlight %}

As we glance through the source we find its a series of if-else checking for the instance and hence as we get down we find that at last it checks for object to be an instance of sequential class. I went through the LongRange class again we find that it inherits from abstract class Aseq and Aseq implements the Sequential interface thus this must be it. I added a bunch of print statements and also tried with a large index to verify that this is the place where the exception is thrown and it does. Finally some sense of relief for the whole work.

{% highlight java %}

else if(coll instanceof Sequential) {
     System.out.println(coll);
     ISeq seq = RT.seq(coll);
     System.out.println("after");
     System.out.println(coll);
     coll = null;
     for(int i = 0; i <= n && seq != null; ++i, seq = seq.next()) {
             if(i == n)
                  return seq.first();
     }
     throw new IndexOutOfBoundsException();

{% endhighlight %}


{% highlight clojure %}

user=> (nth (range 1 10) 2)
(1 2 3 4 5 6 7 8 9)
after
(1 2 3 4 5 6 7 8 9)
3
user=> (nth (range 1 10) 24)
(1 2 3 4 5 6 7 8 9)
after
(1 2 3 4 5 6 7 8 9)
IndexOutOfBoundsException   clojure.lang.RT.nthFrom (RT.java:901)

{% endhighlight %}

But we are not done yet. Its doing a sequential scan which I think is expensive since the with the LongRange object we have the start, end and step. Hence getting the nth number should be start + (step * index) with index being the argument to nth. The time to access the range object becomes linear though the same can be calculated by the formula. Similar optimisation has been done in Python as seen from this (question)[http://stackoverflow.com/questions/30081275/why-is-1000000000000000-in-range1000000000000001-so-fast-in-python-3]. So I will reserve a post or add in the link in case someone has found an answer to this scenario.

{% highlight clojure %}

user=> (def foo (range 10 1000000 2))
nil
user=> (dotimes [_ 5] (time (nth foo 1)))
"Elapsed time: 0.038077 msecs"
"Elapsed time: 0.004212 msecs"
"Elapsed time: 0.003806 msecs"
"Elapsed time: 0.003631 msecs"
"Elapsed time: 0.003999 msecs"
nil
user=> (dotimes [_ 5] (time (nth foo 10)))
"Elapsed time: 0.043773 msecs"
"Elapsed time: 0.004983 msecs"
"Elapsed time: 0.004264 msecs"
"Elapsed time: 0.004659 msecs"
"Elapsed time: 0.004423 msecs"
nil
user=> (dotimes [_ 5] (time (nth foo 100)))
"Elapsed time: 0.044054 msecs"
"Elapsed time: 0.006996 msecs"
"Elapsed time: 0.00576 msecs"
"Elapsed time: 0.006805 msecs"
"Elapsed time: 0.005855 msecs"
nil
user=> (last (dotimes [_ 5] (time (nth foo 1000))))
"Elapsed time: 0.091424 msecs"
"Elapsed time: 0.020456 msecs"
"Elapsed time: 0.019588 msecs"
"Elapsed time: 0.020227 msecs"
"Elapsed time: 0.01979 msecs"
nil
user=> (dotimes [_ 5] (time (nth foo 10000)))
"Elapsed time: 0.581812 msecs"
"Elapsed time: 0.512817 msecs"
"Elapsed time: 0.426505 msecs"
"Elapsed time: 0.377394 msecs"
"Elapsed time: 0.397603 msecs"
nil
user=> (dotimes [_ 5] (time (nth foo 100000)))
"Elapsed time: 4.477103 msecs"
"Elapsed time: 5.257183 msecs"
"Elapsed time: 5.217161 msecs"
"Elapsed time: 3.529831 msecs"
"Elapsed time: 3.458972 msecs"

{% endhighlight %}

