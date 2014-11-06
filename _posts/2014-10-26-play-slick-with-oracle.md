---
layout: post-no-feature
title: "Play Slick with Oracle"
description: "Working with Oracle database never is a pleasure. Right on from the environment setup till the very first CRUD operations. Yet often times we're forced to do so. 
As I haven't found one, here's a quick guide on how to integrate Oracle into Play/Slick app."
category: articles
tags: [scala, programming, play]
---

# Dependencies
Oracle is supported via a closed-source slick-extensions plugin from Typesafe that wraps JDBC driver.
Pull it into your build by adding `slick-extensions` library and [appropriate version](https://github.com/playframework/play-slick#versioning) of `play-slick` module to your build:

{% highlight scala %}
libraryDependencies ++= "com.typesafe.slick" %% "slick-extensions" % "2.0.0" ::
                        "com.typesafe.play" %% "play-slick" % "0.8.0" ::
                        Nil
{% endhighlight %}

# Configuration
In Play `application.conf` file set your database connection settings to (whereas default is db name):

{% highlight scala %}
db.default.slickdriver=com.typesafe.slick.driver.oracle.OracleDriver  
db.default.driver=oracle.jdbc.OracleDriver  
db.default.url="jdbc:oracle:thin:@host:1521:sid"  
db.default.user=username  
db.default.password="password"  
{% endhighlight %}

# Usage
In your model classes import `import com.typesafe.slick.driver.oracle.OracleDriver.simple._` and you're good to go.

# Known Issues

## AutoInc Id
A known issue with Oracle database is that whenever passing an empty value or nothing with an AutoInc index the db complains.
To solve the issue you must provide the value which effectively means no AutoInc at all.
Thus, I employed a simple solution of creating a spin-off data object without the id (and in most cases it is also my domain object as I usually don't need ids) and then map it into the DB-compatible one.
For the last task you might use a type class (I would not recommend using [implicit conversion](http://stackoverflow.com/questions/8524878/implicit-conversion-vs-type-class)).

