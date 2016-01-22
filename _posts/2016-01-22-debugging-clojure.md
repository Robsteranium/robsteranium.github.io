---
title: Debugging Clojure
layout: post
date: 2016-01-22
tags: clojure clojurebridge debugging
---

*This post was originally prepared for [ClojureBridge Berlin](http://clojurebridge-berlin.github.io/)*.

If you ask any programmer what they spend most of their day doing, and if they give you an honest answer, they will probably reveal that they spend very little time writing amazing algorithmns and much more time trying to figure out what's going wrong.

All code has mistakes. These mistakes are not always obvious. They lurk like bugs in the machine. The term "bug" has a [long history of use in engineering](https://en.wikipedia.org/wiki/Software_bug#Etymology), and predates computers. In 1947, the discovery of a moth stuck in the Mark II computer, however, was amusing enough to Grace Hopper and her colleagues that they taped it into their log book.

<figure><a href="https://commons.wikimedia.org/wiki/File:H96566k.jpg#/media/File:H96566k.jpg"><img alt="H96566k.jpg" src="https://upload.wikimedia.org/wikipedia/commons/8/8a/H96566k.jpg"></a><figcaption>By Courtesy of the Naval Surface Warfare Center, Dahlgren, VA., 1988. - U.S. Naval Historical Center Online Library Photograph <a rel="nofollow" class="external text" href="http://www.history.navy.mil/photos/images/h96000/h96566kc.htm">NH 96566-KN</a>, Public Domain, https://commons.wikimedia.org/w/index.php?curid=165211</figcaption></figure>

Unforunately debugging your programs won't be as simple as looking for uninvited insects crawling through the source code. On the contrary, **debugging is hard**. Often you'll only have some cryptic error messages to work with.

## What do bugs look like

Sadly the bugs themselves don't look like anything. As with other invisible things (like the wind or neutrinos) we can only tell they exist from the effect that they have on other things.

### Misbehaviour or unexpected results

For example you might discover a bug because the program doesn't do what you're expecting:

```clojure
user> (defn greater-than-two? [x] (> 2 x))    ; bug lurking here
#'user/greater-than-two?
user> (greater-than-two? 3)                   ; whoops, 3 should be greater than 2
false
user> (defn greater-than-two? [x] (> x 2))    ; bug squashed
#'user/greater-than-two?
user> (greater-than-two? 3)
true                                          ; hooray
```

### Code that can't be read

You may also find that the program won't even run in the first place. The computer may have trouble reading your code. In the example below there's a stray `]`. Notice that it also means that the compiler (which converts your clojure code into machine language) is then surprised to see the `)` as it can't match it up with the opening `(`:

```clojure
user=> (str "beetle" ])   ; errant ]

RuntimeException Unmatched delimiter: ]  clojure.lang.Util.runtimeException (Util.java:221)
RuntimeException Unmatched delimiter: )  clojure.lang.Util.runtimeException (Util.java:221)
```

### Exceptional circumstances

You will also find that your program stops all of it's own accord. These *Exceptions* happen when the machine can't figure out how to proceed. Here, the machine can't figure out how to add a string to a number:

```clojure
user> (+ 1 "louse")
ClassCastException java.lang.String cannot be cast to java.lang.Number  clojure.lang.Numbers.add (Numbers.java:128)
```

The `ClassCastException` occurs when a value of one type can't be converted (or "cast") into another.

Your code should tell the machine what to do in all circumstances, but you can't always rely on other people using it correctly and then you may find it's too late to do anything about it. That's why most programming languages provide some way for you "throw" an exception (or otherwise halt the program).



## Dealing with bugs

### Just search for it

A good approach is to copy the text from the error messages into a search engine. You'll often find someone else has had the same trouble. Very often this will take you to [StackOverflow](http://stackoverflow.com/) where someone else has already asked about the problem (and hopefully received a correct answer).

Many of the values in the error message will be specific to your circumstances (values, line numbers, file names etc) so you will want to remove anything like that so that your query is general enough to find equivalent matches.

### Track it down

When you get an error message you may also get a long list of lines of what looks like gibberish. This is probably the stack trace. That tells you what the machine was doing when it found the error, and then what it did just before that, and just before that... and so on right up to the point that you told it to start doing something.

Typically this is way more information than you need. You might be tempted to just ignore it completely but if you know where to look then the stack trace can be a great help in tracking down your bug.

You'll want to look for any lines that mention your code - i.e. files that you've edited. You should then also see which line number was involved. If you open up the code and start reading there you should find the problem sooner rather than later.

### Take it apart

One of the beautiful things about clojure is that you have a REPL to play with. If you can't see what wrongs with your code then take it apart to find out whats happening.

Copy the buggy code into the REPL. You may need to provide some context to get the code working (`let` comes in handy for making temporary data structures to test the code with). Explore the code form by form, piece by piece, building-up from simple bits that you understand (and have checked) to the larger more complex whole.


## Some common examples

The examples listed below happen to clojure programmers on a daily basis. They seem a bit strange at first but you soon grow accustomed to them and develop a sense of where to look to find your mistakes.

### Something cannot be cast to clojure.lang.IFn (ClassCastException)

This means that something cannot be executed as a function. That something is usually the class of a literal e.g. `java.lang.Long` (for a number like `1`) or `java.lang.String` (for text like `"moth"`). This exception is quite common among programmers who aren't accustomed to prefix notation. Remember that clojure expects the first thing in a form (inside a set of parentheses) to be a function:

```clojure
user> (1 + 2)   ;      :(
ClassCastException java.lang.Long cannot be cast to clojure.lang.IFn  user/eval7972 (form-init826537272185236344.clj:2)

user> (+ 1 2)   ;      :)
3
```

### Don't know how to create ISeq from something (IllegalArgumentException)

This means that something can't be interpreted as a sequence. That something is usually a reference to a function e.g. the `even?` function which will appear in the stack trace as `clojure.core$even_QMARK_` (note how `?` becomes `_QMARK_`). Again this is often a mistake in the order of arguments:

```clojure
user> (map (range 0 5) even?)   ;    :(  map expects a function, then a collection
IllegalArgumentException Don't know how to create ISeq from: clojure.core$even_QMARK_  clojure.lang.RT.seqFrom (RT.java:528)
user> (map even? (range 0 5))
(true false true false true)    ;    :)

user> (first 1)                 ;    :(  first expects a collection, not a single value
IllegalArgumentException Don't know how to create ISeq from: java.lang.Long  clojure.lang.RT.seqFrom (RT.java:528)
user> (first [1 2])             ;    :)
1
```

###  Wrong number of args (x) passed to some function (ArityException)

The arity of a function is how many argument it expects. If you call the function with the wrong number of arguments you'll get an `ArityException`:

```clojure
user> (str)                     ;    :)  str works without any arguments
""                              ;    it just creates an empty string

user> (max)                     ;    :(  max does expect at least one argument
ArityException Wrong number of args (0) passed to: core/max  clojure.lang.AFn.throwArity (AFn.java:429)
user> (max 1)                   ;    :)
1
user> (max 1 10)                ;    :)  and it's fine with more
10

user> (first 1 2)               ;    :(  first expects one argument - a collection
ArityException Wrong number of args (2) passed to: core/first--4110  clojure.lang.AFn.throwArity (AFn.java:429)
user> (first [1 2])             ;    :)  but that collect can have many values
1

user> (defn choose-randomly [x y]    ; two arguments are expected
        (if (> 0.5 (rand)) x y))
#'user/choose-randomly
user> (choose-randomly :mate)        ;   ;(  only one option given
ArityException Wrong number of args (1) passed to: user/choose-randomly  clojure.lang.AFn.throwArity (AFn.java:429)
user> (choose-randomly :mate :bier)  ;   :)
:bier                                ;   :p
```
