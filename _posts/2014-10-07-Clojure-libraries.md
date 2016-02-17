---
title:  "Using Clojure At MixRadio"
date:   2014-10-19 10:33:00
author: Ben Griffiths
author-twitter: wsbfg
categories: clojure
excerpt: An overview of the Clojure libraries that we use and an introduction to Mr Clojure, the MixRadio leiningen
layout: post
---

## Please note
Please note that this post originally appeared on the MixRadio dev site. Following the shutdown I've moved it over here as a permanent archive. Also bear in mind that some new libraries have appeared that may be good alternatives to those suggested here. This post was written in the end of 2014.

## Clojure at MixRadio

For the last two and a bit years we've been using Clojure to power our backend at MixRadio. We've moved from a largely Java platform to one where almost all new development is done in Clojure. As of today we have approximately 50 web services written in Clojure.

In this post I want to introduce the Leiningen template that we use to quickly bootstrap new services and talk about some of the Clojure libraries that we use. In the next couple of weeks we'll be publishing a series of articles by [Neil Prosser](http://twitter.com/neilprosser) talking about how we deploy our Clojure (and other) services.

## A MixRadio Web service

We've recently [open sourced](http://github.com/mixradio/mr-clojure) our internal Leiningen template. Leiningen is a build automation and dependency management tool for Clojure that's been enthusiastically embraced by the community. It has many useful plugins and functions as a Swiss Army Knife for working with Clojure projects. Here we are using `lein new` which allows you to create Clojure projects from templates. Our new template allows you to create a new web service in the style of MixRadio. We've developed and tested this over the last two years and it serves many millions of requests every day to our users.

Creating a new service using our template is really simple using Leiningen:

`lein new mr-clojure <your-project-name>`

This will download and create a new web service that's ready to go and already contains a couple of sample resources. If you're the sort who wants to cut to the chase, you can check out the [resulting code on github](http://github.com/mixradio/mr-clojure-exploded).

For more detailed documentation and examples [see the github page](http://github.com/mixradio/mr-clojure). Here are some of the highlights:

* A REST service template aimed loosely towards JSON (not a requirement)
* Embedded Jetty 9
* Based on [compojure](https://github.com/weavejester/compojure) routes
* Generates deployable artifacts suitable for use in production
* A good basis of metrics reporting and logging
* A well tested set of libraries

## Clojure Libraries

When we started working with Clojure we were, at times, overwhelmed by the choice available to us. Below is a list of those libraries that crop up again and again, along with a few gems that might not be so obvious.

## [emacs-live](https://github.com/overtone/emacs-live)

<a href="/assets/blog/UsingClojureAtMixRadio/emacs.png"><img src="/assets/blog/UsingClojureAtMixRadio/emacs.png" style="width:450px;" title="Emacs-Live" /></a>

Firstly a special mention for Sam Aaron's Emacs config. Whilst not a Clojure library, almost our whole development team bases their emacs config on Sam's work. Out of the box it provides a series of sane (he would say opinionated) defaults and sets up some important libraries such as [CIDER](http://github.com/clojure-emacs/cider) REPL integration. A few people have moved on to other editors but having such an easy to use base allowed us to get a whole team of programmers up and running quickly. For those who appreciate the finer things in life, vim integration can be added with [evil mode.](http://www.emacswiki.org/Evil)

## [amazonica](https://github.com/mcohen01/amazonica)

Amazonica is a well featured Clojure library for accessing AWS. As the library is reflection based it keeps up with the Java client and is fully featured. Amazonica was particularly useful in developing our deployment tooling.

{% highlight clojure %}
(describe-images :owners ["self"]
                 :image-ids ["ami-f00f9699" "ami-e0d30c89"])
{% endhighlight %}

## [at-at](https://github.com/overtone/at-at)

at-at is a really simple ahead of time function scheduler. It's proven to be reliable in long running situations, so long as you're careful to ensure that no exceptions can escape.

{% highlight clojure %}
(def my-pool (mk-pool))

(at (+ 1000 (now)) #(println "Hello future world!") my-pool)
{% endhighlight %}

Also be aware that console output from a scheduled function won't go into your REPL.

## [bouncer](https://github.com/leonardoborges/bouncer)

There are [absolutely loads](http://www.clojure-toolbox.com/) of Clojure validation libraries. We're using quite a few of them already here at MixRadio. Bouncer though is probably the most common. It strikes a good balance between simplicity and extensibility. It also produces validation code that looks a lot like the map that you're validating, which makes the code nice and simple to read.

{% highlight clojure %}
(validate song
          :length              v/number
          [:artist :firstname] v/required
          [:artist :age]       v/number)
{% endhighlight %}

## [camel-snake-kebab](https://github.com/qerub/camel-snake-kebab)

This is one of those libraries that turns out to be far more useful than you would expect. Camel_SNAKE_kebab is a library for naming conventions.

{% highlight clojure %}
(->CamelCase 'foo-bar)') ; => "FooBar"

(->SNAKE_CASE "foo bar") ; => "FOO_BAR"

(->HTTP-Header-Case "x-forwarded-for") ; => "X-Forwarded-For"
{% endhighlight %}

## [cascalog](https://github.com/nathanmarz/cascalog)

Cascalog is a logic data processing library. We use it for Hadoop data processing. It takes a little getting used to but is extremely powerful and expressive.

{% highlight clojure %}
(?<- (stdout)
     [?word ?count]
     (sentence ?line)
     (tokenise ?line :> ?word)
     (c/count ?count))
{% endhighlight %}

It's probably best to [watch the video](https://www.youtube.com/watch?v=7qq_PmwplEc) or work through [some tutorial examples](http://cascalog.org/articles/marz_intro_1.html).

## [cheshire](https://github.com/dakrone/cheshire)

We've found that Cheshire is the best JSON parser and generator available for Clojure. It offers great performance combined with a full feature set and good safety.

{% highlight clojure %}
(generate-string {:hello "world" :number 5}) ; => "{\"number\":5,\"hello\":\"world\"}"

(parse-string "{\"foo\":\"bar\"}" true) ; => {:foo "bar"}
{% endhighlight %}

## [clj-http](https://github.com/dakrone/clj-http)

clj-http is a nice wrapper around Apache's client. The interface is pleasant to use and it offers plenty of configuration options. Bear in mind that the default request throws an exception on non 200 and 300 series HTTP responses. It also doesn't set a default timeout which is fairly essential in most cases. Not doing so can tie up all your threads and completely lock your server.

{% highlight clojure %}
(-> (http/get "http://example.com"
              {:socket-timeout 3000 :conn-timeout 3000}
    :body))
{% endhighlight %}

## [clj-time](https://github.com/clj-time/clj-time)

clj-time is a date and time library based on the [Joda Time](http://www.joda.org/joda-time/) library.

{% highlight clojure %}
(date-time 1986 10 14) ; => #<DateTime 1986-10-14T00:00:00.000Z>

(after? (date-time 1986 10) (date-time 1986 9)) ; => true

(parse formatter "20100311") ; => #<DateTime 2010-03-11T00:00:00.000Z>

(take 10 (periodic-seq (now) (hours 12)))
{% endhighlight %}

## [compojure](https://github.com/weavejester/compojure)

Compojure is a routing library that provides an elegant syntax around [ring](https://github.com/ring-clojure/ring). We like to ensure that we don't put more than a line of code in the route definitions. This avoids clutter and allows us to tell at a glance what resources a service provides. The best way to achieve this is to simply destructure then defer down to a handling function.

{% highlight clojure %}
(defroutes routes

  (GET "/healthcheck"
       [recurse] (healthcheck))

  (GET "/ping"
       [] "pong")

  (route/not-found (error-response "Resource not found" 404)))
{% endhighlight %}

## [conch](https://github.com/Raynes/conch)

Conch is a library that allows you to shell out and call executables as if they were Clojure functions. It's a very neat way of interacting with other programs and it exposes some low level options should you require them. 

{% highlight clojure %}
(programs echo ls)
(echo "Hi!") ; => "Hi!"

(ls {:seq true}) ; => ("lots" "of" "files")
{% endhighlight %}

## [core.async](https://github.com/clojure/core.async)

We've tentatively started using core.async in a few parts of our data analytics pipeline. Currently it's still in alpha status so expect change and the potential for bugs. We also feel that it's currently targeted at a lower level of abstraction than is necessary for most of our purposes. However it's a really useful library and an amazing example of what's possible in Clojure.

## [environ](https://github.com/weavejester/environ)

Environ is a library for managing configuration and environment settings. It can read from a project.clj file, system environment variables and Java system properties. We define local test properties in the project.clj file which we sometimes override in our acceptance and integration tests. When deployed, our tooling makes those same settings available as environment variables with the correct production values. It's a simple library, but one that solves the question of test and production configuration for us.

{% highlight clojure %}
(def logging-level (env :logging-level))
{% endhighlight %}

## [faraday](https://github.com/ptaoussanis/faraday)

Faraday is a library for working with Amazon's [DynamoDB](http://aws.amazon.com/dynamodb/). Unlike Amazonica (the generalised AWS Clojure client), Faraday is not reflected. This gives a performance increase that can be important if it's frequently called. Also by focusing on one part of the AWS offering the API has been tuned to be really nice to work with.

{% highlight clojure %}
(far/put-item opts :my-table {:id 0 :name "Bob" :age 22})
{% endhighlight %}

## [midje](https://github.com/marick/Midje)

Midje is a testing framework that is built around a [philosophy of top down development](https://github.com/marick/Midje/wiki/The-idea-behind-top-down-development). We've adopted it almost exclusively across our Clojure code. Whilst I think it's fair to say that Midje has some issues and frustrations, for us it is still the best testing library in Clojure.  

{% highlight clojure %}
(fact "Maths is of questionable reliability"
      (+ 3 4) => 7
      (new-plus 3 4) => 8
      (provided
        (new-plus 3 4) => 8))
{% endhighlight %}

## [monger](https://github.com/michaelklishin/monger)

Monger is a MongoDB driver based on the official Java Driver. It's fast, reliable and pleasant to work with. Clojure, MongoDB and JSON make for extremely quick development. We once had the requirement to get a service developed in a couple of days, so we built it quickly backed by MongoDB before changing out the persistence layer, behind the scenes, into something more suited to the problem. As a rule of thumb we've found that [clojurewerkz](http://clojurewerkz.org/) libraries are of a high quality and are generally our first port of call if available.

{% highlight clojure %}
(insert-and-return db "songs" {:artist "Madonna" :title "Like a Prayer" :released 1988})

(find-maps db "songs" {:released 1988}) ; => {:artist "Madonna" ...}
{% endhighlight %}

## [rest-cljer](https://github.com/whostolebenfrog/rest-cljer)

rest-cljer is a wrapper for the [rest-driver](https://github.com/rest-driver/rest-driver) library. It allows mocking of HTTP requests for the testing of REST services. Unlike most mocking libraries, rest-driver sets up a web server to respond to the request providing a very thorough test whilst still allowing separation from dependencies. Both of these libraries were developed in-house an open sourced a number of years ago.

{% highlight clojure %}
(fact "Example of testing two"
      (rest-driven
          [{:method :GET :url "/gety"}
           {:type :JSON :status 200}

           {:method :GET :url "/something"}
           {:type :JSON :status 200}]

          (client/get "http://localhost:8081/gety") => (contains {:status 200})
          (client/get "http://localhost:8081/something") => (contains {:status 200})))
{% endhighlight %}

## [slingshot](https://github.com/scgilardi/slingshot)

Slingshot brings support for maps and destructuring into Clojure's exception handling. Whilst this use of exceptions is not suitable everywhere, it can simplify some interfaces and can be used as a tool to stop concepts from leaking out of namespaces. 

{% highlight clojure %}
(defn throwy-fn []
  (if (pos? (rand-int 2))
    (throw+ {:type :one  :foo "foo"})
    (throw+ {:type :zero :bar "bar"})))

(try+ (throwy-fn)
      (catch [:type :one]  {:keys [foo]} foo)
      (catch [:type :zero] {:keys [bar]} bar)) ; => "foo" or "bar"
{% endhighlight %}

## Wrapping up

That's just a small section of the libraries we use a MixRadio but hopefully this will give you some insight into how we work. We also hope that you are able to make use of our skeleton project, we'd appreciate feedback and pull requests on the github page or in the comments here.

## The Author

Ben Griffiths is an engineer at MixRadio and was one of the early team who pushed Clojure adoption. You can find out more about our adoption story [in this talk.](https://skillsmatter.com/skillscasts/3891-clojure-at-nokia-entertainment) You can read his blog [here](http://whostolebenfrog.github.io/).
