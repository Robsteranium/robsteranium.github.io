---
layout: post
title: "Create a linked-data Concept Scheme from a spreadsheet with Grafter"
date: 2016-01-14
tags: linked-data skos rdf grafter clojure csv xls xlsx
---

An `skos:ConceptScheme` is useful for representing lists - e.g. of options or codes. The process of creating ConceptSchemes and adding Concepts too them can be quite repetitive. We can speed up the process using [Grafter](http://grafter.org/) - a clojure library for producing RDF.

You can either clone this repository and adapt it to your needs or follow the instructions in this readme to write your own project from scratch.


## Prerequisites

First you'll need to install the [Java Development Kit](http://www.oracle.com/technetwork/java/javase/downloads/index.html).

Then you'll need to install the [leiningen](http://leiningen.org/) build tool. This will bootstrap our clojure environment and take care of installing Grafter.


## Create a grafter project

We can then use leiningen to build a new clojure project from the grafter template. We're going to create an example using food types.

    $ lein new grafter food-types
    $ cd food-types

This will create a few examples files. You can learn more about these with grafter's [Getting Started](http://grafter.org/getting-started/index.html) guide.


### Add some spreadsheet data

We can do away with the example data and create our own:

    $ rm data/example-data.csv
    $ nano data/food-types.csv

Paste following csv data into the file and save it.

    food,type
    Apple,Fruit
    Banana,Fruit
    Cabbage,Vegetable

Grafter can handle a wide variety of input formats including xls/x, but we're going to stick with some simple csv for now.


### Jack into the REPL

Clojure provides an interactive prompt or read-evaluate-print-loop. To start the REPL do:

    $ lein repl

Once leiningen has downloaded the dependencies and started a java virtual machine etc, you will have a prompt. You can test that everything is working with the following function call: 

{% highlight clojure %}
food-types.pipeline=> (grafter.tabular/read-dataset "data/food-types.csv")
{% endhighlight %}
{% highlight text %}
|       a |         b |
|---------+-----------|
|    food |      type |
|   Apple |     Fruit |
|  Banana |     Fruit |
| Cabbage | Vegetable |
{% endhighlight %}

If everything has gone to plan, then you should see an ascii art version of the table we just created.

We could write the whole program interactively like this but that would be quite tedious if you or someone else ever wanted to use it again. Instead, of course, we will build-up our program in source files. We still want to develop it interactive to test and explore along the way. Eventually you'll want to use a clojure IDE (e.g. emacs+cider, LightTable, or IntelliJ+cursive) but we won't cover that here. For now we'll assume that you have evaluated the function definitions in your repl (copy and paste will do).


### Create a pipeline

Let's remove the example pipeline and create our own

    $ rm src/food_types/pipeline.clj
    $ nano src/food_types/pipeline.clj

First we should add the namespace declaration for this file and `require` common library functions so that our code isn't cluttered with fully-specified namespaces:

{% highlight clojure %}
(ns food-types.pipeline
  (:require
    [grafter.tabular :refer [read-dataset make-dataset move-first-row-to-header defpipe]]))
{% endhighlight %}

Then we'll create a pipeline that reads the dataset, interpreting the first line as the column headings. We're using clojure's nifty threading macro `->` to clarify the code; this just puts the return value of function as the first parameter of the function that follows e.g. so that `(c (b (a 0)))` can be written as `(-> 0 a b c)`. The two forms have the equivalent effect but the latter reads much more like a pipeline. The grafter library is designed to be amenable to composing pipelines in this way, by having the first parameter and the return value of each of it's transformation functions to be the Dataset type.

{% highlight clojure %}
(defpipe read-food-data [filename]
  (-> (read-dataset filename)
      (make-dataset move-first-row-to-header)))
{% endhighlight %}

We can then test this at the repl:

{% highlight clojure %}
food-types.pipeline=> (read-food-data "data/food-types.csv")
{% endhighlight %}
{% highlight text %}
|    food |      type |
|---------+-----------|
|   Apple |     Fruit |
|  Banana |     Fruit |
| Cabbage | Vegetable |
{% endhighlight %}
 
### Create some RDF

Now that we've got some data loaded, let's think about where we're headed with this. What RDF statements do we want to create?

We'll start with the `skos:ConceptScheme` itself. In turtle, we want to create this, and a label:

{% highlight turtle %}
@base <http://example.com/> .
@prefix skos: <http://www.w3.org/2004/02/skos/core#> .
@prefix rdfs: rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

:food-types a skos:ConceptScheme;
  rdfs:label "Food Types";
  .
{% endhighlight %}

Using grafter's DSL we can write this as:

{% highlight clojure %}
(ns food-types.pipeline
  (:require
    ;;...
    [grafter.rdf :refer [s]]
    [grafter.rdf.templater :refer [graph]]
    [grafter.vocabularies.rdf :refer :all]
    [grafter.vocabularies.skos :refer :all]))

(defn build-concept-scheme []
  (graph "http://example.com/food-graph"
         ["http://example.com/food-types"
          [rdf:a skos:ConceptScheme]
          [rdfs:label (s "Food Types")]]))
{% endhighlight %}

We can confirm that this creates the RDF we want by serialising the results to a file or (as here) to a `java.io.StringWriter`:

{% highlight clojure %}
(require '[grafter.rdf.formats :as format])

(let [s-wtr (java.io.StringWriter.)]
  (with-open [wtr (clojure.java.io/writer s-wtr)]
    (let [serialiser (grafter.rdf.io/rdf-serializer wtr :format format/rdf-turtle)
          quads (build-concept-scheme)]
      (grafter.rdf.protocols/add serialiser quads)
      (println (.toString s-wtr)))))
{% endhighlight %}

{% highlight turtle %}
<http://example.com/food-types> a <http://www.w3.org/2004/02/skos/core#ConceptScheme> ;
        <http://www.w3.org/2000/01/rdf-schema#label> "Food Types" .
{% endhighlight %}


For the `skos:Concept`s, we want to have the URIs and labels being set by the data - in other words we want a function that takes these inputs and returns RDF. We can use `graph-fn` to build a template that will extract the relevant cells from rows of an input source. In our case we feed it with the "food" and "type" columns:

{% highlight clojure %}

(defn food-uri [name]
  (str "http://example.com/def/food/" (slugify name)))

(defn type-uri [type]
  (str "http://example.com/def/type/" (slugify type)))

(defn build-concept [concept-label]
  (graph-fn [{:strs [food type]}]
    (graph "http://example.com/food-types"
      [(food-uri food)
       [rdf:a skos:Concept]
       [skos:inScheme food-type-scheme]])))

{% endhighlight %}

### Hierarchical scheme

### Feed spreadsheet rows into template
;;(defgraft

### Run it!
... from repl
... from command prompt
