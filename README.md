# scala-useful

Schedule method to run every second.

```scala
import scala.concurrent.duration._
import scala.concurrent.ExecutionContext
import ExecutionContext.Implicits.global

val system = akka.actor.ActorSystem("system")
system.scheduler.schedule(0 seconds, 1 seconds)(methodName)
```

Use StatsD to send Stats:

```scala
package com.statsd

import akka.actor.{
  Props,
  Actor,
  ActorSystem
}

import scala.concurrent.duration._

class Stat(namespace: String = "com.domain", environment: String = "test", componentName: String, metricName: String) {
  def format =
    String.format("%s.%s.%s.%s", namespace, environment, componentName, metricName)
}

object StatsDTest {
  class StatsDActor extends Actor {
    val statsD = new StatsD(context = context, "192.168.99.100", 8125, false)

    val Stat = classOf[Stat]

    def receive = {
      case stat: Stat => {
        println("Received")

        statsD.increment(stat.format)
      }
    }
  }

  def main(args: Array[String]): Unit = {
    import scala.concurrent.ExecutionContext
    import ExecutionContext.Implicits.global

    val system = ActorSystem("System")
    val statsActor = system.actorOf(Props[StatsDActor], name = "statsActor")

    system.scheduler.schedule(0 seconds, 5 seconds)(statsActor ! new Stat(componentName = "component", metricName = "status"))
  }
}

```
