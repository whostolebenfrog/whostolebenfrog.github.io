---
layout: post
title:  "Updating an old version of clojars"
date:   2014-07-02 13:12:00
categories: clojure
---

At MixRadio we run a local copy of clojars which we use as a repository for internal clojure libraries that for whatever reason we don't want to host publicly on a clojars.org. This has worked out pretty well as a simpler place to deploy to than the nexus repo we used for java projects. Unfortunately I've not updated it since I first set it up a few years ago, in my defence its been working pretty well... ahem...

Anyway if you've not checked recently [clojars.org](clojars.org) has just had a great redesign and looks lovely. Probably time to try my hand at updating it.

On the whole it wasn't too bad, particularly given how long its been. Fortunately the actual jars are hosted by apache running on the same machine serving out of `/repo` (I did this so that it was possible to list the directories). This meant that I could partially break the site but not interrupt people resolving their dependencies so long as the directory remained in place. I also, somewhat lazily, use `lein run` to host rather than packaging up an executable jar file which simplified the process a little.

My approach was as follows:

* Check that my automated backup was up-to-date
* Stop the clojars service
* Git pull
* Start the service
* Failure and sadness

The stacktrace tells me that there is a missing column in the DB. A quick ack-grep of the source code reveals a migration.clj file. It wasn't, however, clear whether or not I could safely run everything in this file so I fired up sqlite and compared the migrations to my database. In case you haven't used sqlite here are the basic commands for poking around:

* `sqlite3 data/dev_db`
* `.tables` to view tables
* `.schema users` to describe the users table
* `select * from users`
* `.exit` when you are done

I found that we had none of the migrations performed. You can either paste into the console by hand or run the helpful `lein migrate` from the root of clojars-web to apply them all.

* `service clojars start`
* Failure and more sadness

Now clojars is complaining that it can't find `all.edn`, quite right as it doesn't exist. There's nothing in the project but `src/clojars/stats.clj` requires it. Very odd. It looks like they are generating this file with a separate process that parses their request logs (presumably with a cronjob or similar), see `src/clojars/stats/process.clj`. You can use this if your request log format match theirs exactly, and you are recording them of course. Alternatively you can create a file at that path that contains just `{}` or `curl https://clojars.org/stats/all.edn > data/stats/all.edn` to take a copy of the live stats.

* `service clojars start`
* looking good...

At this point all appears to be reasonably well, however a little more digging reveals a strange issue where we can't add ssh-keys to users. The message is that the keys format is incorrect. A few well placed `prns` and it looks like they are being url encoded even though they are form params. I actually spent quite a while digging into this as my first suspect was that something on our network was mangling them. As it turned out though all that was required was:

* `lein clean`

Presumably something was left over from the old version.

That's probably all the problems you will have getting it updated. I actually had another issue where scping jars was now failing to auth. A glace at `/var/log/secure` told me that the data folder in clojars didn't have the correct permissions. Presumably this was reset by the git pull, `chmod 700 data` fixed this issue but did mean that apache was no longer able to access the folder to serve the jars. The easiest way around this was to run apache with the `clojars` user. Assuming a normal box you should just be able to get the `User` and `Group` properties in `/etc/httpd/conf/httpd.conf` to be clojars. As my box is puppetised I had to add these in to our global apache puppet template.

Hopefully that's helpful to you, I couldn't find anything about this subject so I decided to write up my findings.
