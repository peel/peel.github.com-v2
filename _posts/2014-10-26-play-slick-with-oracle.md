---
layout: post-no-feature
title: "Play Slick with Oracle"
description: "Working with Oracle database never is a pleasure. Right on from the environment setup till the very first CRUD operations. Yet often times we're forced to do so. Here's a quick guide on how to integrate Oracle into Play/Slick app."
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
In Play application.conf file set your database connection settings to (whereas default is db name):

```
db.default.slickdriver=com.typesafe.slick.driver.oracle.OracleDriver  
db.default.driver=oracle.jdbc.OracleDriver  
db.default.url="jdbc:oracle:thin:@host:1521:sid"  
db.default.user=username  
db.default.password="password"  
```

# Usage
In your model classes import `import com.typesafe.slick.driver.oracle.OracleDriver.simple._` and you're good to go.
