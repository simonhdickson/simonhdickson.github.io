---
layout: single
title:  "Running Kafka on Windows"
date:   2014-06-13 20:16 +0000
categories: kafka babun windows
author_profile: true
---
I was introduced to Kafka the other day and was having loads of fun running it from the mac terminal.

Then I wanted to try out [the .net client library](https://github.com/Jroland/kafka-net), so went to run the windows bat files, and sadly it didn't work. Sadness :(. However there is a way to get it working under Windows using some Cygwin magic.

You need two things for this to work:

### Prerequisites

* Java needs to be installed.
* [Babun](https://github.com/babun/babun) installed (if you already have Cygwin set up you can use that, but I'll carry on assuming you're using babun because it makes it so easy!)
* Download the latest version of [Kafka](http://kafka.apache.org/downloads.html) (and untar it).

Once you've got all that, let's get started!

### Instructions

1. Open ``bin\kafka-run-class.sh`` and add double quotes around the $JAVA variables, and change $CLASSPATH to `` `cygpath -wp $CLASSPATH` `` (with the backticks).
2. This part is optional, but if you want to get logging to work you'll need to open ``config\server.properties`` and change the line ``log.dirs=/tmp/kafka-logs`` to ``log.dirs=\\tmp\\kafka-logs``. And you want to do the same for all the rest of the config files you end up using.
3. Open up babun and go the folder where you download kafka to, e.g. ``cd /cygdrive/c/Users/YOURNAME/Downloads/kafka_2.9.2-0.8.1.1`` or wherever you downloaded it to.
4. Then you should be all done, and can follow the instructions [here](http://kafka.apache.org/documentation.html#quickstart).

### Possible Issues  

A few gotcha you might get, these problems shouldn't occur but if they do here are the fixes:

* In my case the server JRE bin folder didn't have the server folder in it, but if you install the JDK you can copy it over to the JRE folder to fix that.
* Probably related to the above fix, if you get an error something like ``Unrecognized VM option 'CheckCompressedOops'`` then if you open the ``bin/kafka-run-class.sh`` file and remove the line ``-XX:+UseCompressedOops`` it should work.

Now that's out the way, Kafka Type Provider anyone?