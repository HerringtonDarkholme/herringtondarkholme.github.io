title: play framework with scalate and activerecord
date: 2014-08-18 22:04:58
tags: Scala
---

> WRYYYYYYYYYYYYYYYY
>
> -- <cite>Dio Brando on Scala crazy dependencies</cite>

Scala works like CSS selectors in that every successor overrides its predecessor.

You will have to work as a detective to figure out the correct recipe to manage a huge casserole of hodgepodge.

To achieve a working _Play_ configuration with `scalate` and `activerecord` needs:

In `build.sbt`:

```scala
scalaVersion :=  "2.10.3"

libraryDenpendencies ++= Seq(
  jdbc,
  "org.scalatra.scalate" %% "scalate-core" % "1.7.0",
  "com.github.aselab" %% "scala-activerecord" % "0.2.3",
  "com.github.aselab" %% "scala-activerecord-play2" % "0.2.3",
  "com.h2database" % "h2" % "1.3.170"
)
```

Several notes:

1. Currently `scala-activerecord` only supports 2.10.3
2. `scalate` must be `1.7.0`+ for better support on scala 2.10 but the current stable version is `1.6.0`

Then in the root path of play project, create a new file located at `app/lib/ScalateIntegration.scala`
```scala

package controllers

import play.api._
import http.{Writeable, ContentTypeOf, ContentTypes}
import mvc.Codec
import play.api.Play.current
import org.fusesource.scalate.layout.DefaultLayoutStrategy
import collection.JavaConversions._

object Scalate {

  import org.fusesource.scalate._
  import org.fusesource.scalate.util._

  var format = Play.configuration.getString("scalate.format") match {
    case Some(configuredFormat) => configuredFormat
    case _ => "scaml"
  }

  lazy val scalateEngine = {
    val engine = new TemplateEngine
    engine.resourceLoader = new FileResourceLoader(Some(Play.getFile("app/views")))
    engine.layoutStrategy = new DefaultLayoutStrategy(engine, "app/views/layouts/default." + format)
    engine.classpath = "tmp/classes"
    engine.workingDirectory = Play.getFile("tmp")
    engine.combinedClassPath = true
    engine.classLoader = Play.classloader
    engine
  }

  def apply(template: String) = Template(template)

  case class Template(name: String) {

    def render(args: java.util.Map[String, Any]) = {
      ScalateContent{
        scalateEngine.layout(name, args.map {
          case (k, v) => k -> v
        } toMap)
      }
    }

  }

  case class ScalateContent(val cont: String)

  implicit def writeableOf_ScalateContent(implicit codec: Codec): Writeable[ScalateContent] = {
    Writeable[ScalateContent]((scalate:ScalateContent) => codec.encode(scalate.cont))
  }

  implicit def contentTypeOf_ScalateContent(implicit codec: Codec): ContentTypeOf[ScalateContent] = {
    ContentTypeOf[ScalateContent](Some(ContentTypes.HTML))
  }
}

```

Again only works on scalate 1.6.

Finally, according to `activerecord`'s doc.
To load the plugin, in `conf/play.plugin`

```
9999:com.github.aselab.activerecord.ActiveRecordPlugin
```

To configure database, in `conf/application.conf`

```
# Database configuration
# ~~~~~
#

# Scala ActiveRecord configurations
db.activerecord.driver=org.h2.Driver
db.activerecord.url="jdbc:h2:mem:play"
db.activerecord.user="sa"
db.activerecord.password=""

# Schema definition class
activerecord.schema=models.Tables
```

And in `app/models/person.scala`

```
package models

import com.github.aselab.activerecord._
import com.github.aselab.activerecord.dsl._

case class Person(@Required name: String) extends ActiveRecord
object Person extends ActiveRecordCompanion[Person] with PlayFormSupport[Person]
```

In `app/models/tabels.scala`

```
package models

import com.github.aselab.activerecord._
import com.github.aselab.activerecord.dsl._

object Tables extends ActiveRecordTables with PlaySupport {
  val models = table[Person]
}
```

And finally you can try this in console

```shell
activator console

import models._
import play.core.StaticApplication

new StaticApplication(new java.io.File("."))

Person("f@ck").save
Person.findBy("name", "f@ck")
```

WTF have I done? I just typed mechanically what I have ferreted on Google and StackOverflow.
WTF is `play.core.StaticApplication` that is just one confusing [page](https://www.playframework.com/documentation/2.2.x/api/scala/index.html#play.core.StaticApplication) on the doc?
Speciously tantializing is the magical code under which lurk complicated dependencies.

Reference: [PlayframeWork Quick Tip](http://www.javacodegeeks.com/2012/04/play-framework-2-quicktip-scala-console.html)
