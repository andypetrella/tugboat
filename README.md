# tugboat

> a small boat that maneuvers [Docker](http://www.docker.com/) vessels

[Docker](http://www.docker.com/) is a manager for the containment of applications. Tugboat is a small library
that speaks the docker protocol for moving applications and containers in and out of your local seaport.

## usage

Tugboat provides interfaces for interacting with docker returning Scala [Futures](http://www.scala-lang.org/api/current/index.html#scala.concurrent.Future)
containing docker responses.
It's up to your application to decide how to deal with the result of a future.

```scala
import scala.concurrent.ExecutionContext.Implicits.global

// create a docker tugboat client
val tb = tugboat.Client()

// print the make and model of docker harbor
// $ docker version
tb.version().onComplete(println)

// search the sea of docker images for something that looks like a ship
// $ docker search ship
tb.images.search.term("ship")().map(_.map(_.name)).onComplete(println)

// list the images docked at your station
// $ docker images
tb.images.list().map(_.map(_.id)).onComplete(println)

// be your own shipping port

// usher a ship out to sea
// $ docker build -t ssScala path/to/dir/Dockerfile/is/in
import tugboat.Build 
tb.images.build(new java.io.File("path/to/dir/Dockerfile/is/in")).tag("ssScala").stream {
  case Build.Progress(prog)   => println(prog)
  case Build.Status(status)   => println(status)
  case Build.Error(err, _, _) => println(err)
}

// usher foreign ships into harbor
// $ docker pull captain/ship
import tugboat.Pull
tb.images.pull("captain/ship").stream {
  case Pull.Status(msg) => println(msg)
  case Pull.Progress(msg, _, details) =>
    println(msg)
    details.foreach { dets =>
      println(dets.bar)
    }
  case Pull.Error(msg, _) =>  println(msg)
}


// verify with the admiral that your captain credentials are still respectable
// $ docker login -u username -p password -e email
import tugboat.AuthConfig
val auth = AuthConfig(user, password, email)
tb.auth(auth)().onComplete(println)

// announce your captainship when issuing orders to the crew
// $ docker pull internalregistry.com/captain/ship
tb.as(auth).images.pull("captain/ship").registry("internalregistry.com").stream {
  case Pull.Status(msg) => println(msg)
  case Pull.Progress(msg, _, details) =>
    println(msg)
    details.foreach { dets =>
      println(dets.bar)
    }
  case Pull.Error(msg, _) =>  println(msg)
}

// fashion a new boat from a dependable stack of material and start the engines
// $ docker run -p 80:80 captain/ship
(for {
  container <- tb.containers.create("captain/ship")()
  run       <- tb.containers.get(container.id).start.bind(
               tugboat.Port.Tcp(80), tugboat.PortBinding.local(80)
            )()
} yield container.id).onComplete(println)

// produce a roster of ships out to sea
// $ docker ps
tb.containers.list().map(_.map(_.id)).onComplete(println)

// anchor to a live boat
val ship = tb.containers.get(id)

// inspect the boat
// $ docker inspect ship
ship().onComplete(println)

// fetch the the captains logs
// $ docker logs ship
ship.logs.follow(true).stdout(true).stderr(true).stream(println)

// stop the boat after 5 seconds
// $ docker stop ship
import scala.concurrent.duration._
ship.stop(5.seconds)().onComplete(println)

// restart the boat in 5 second
// $ docker restart ship
ship.restart(5.seconds)().onComplete(println)

// retire the ship
// $ docker rm ship
ship.destroy.force(true)().onComplete(println)
```

Doug Tangren (softprops) 2014
