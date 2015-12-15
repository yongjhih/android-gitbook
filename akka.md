# akka

## akka + kotlin

```kotlin
object start {}

fun main(args: Array<String>) {
    val system = ActorSystem.create();
    val helloActor = system.actorOf(Props(javaClass<HelloActor>()));
    helloActor.tell(start)
}

class HelloActor: UntypedActor() {

    var worldActor: ActorRef = getContext().actorOf(Props(javaClass<WorldActor>()))

    public override fun onReceive(msg: Any?) {
        when (msg) {
            start -> worldActor.tell("Hello", self())
            is String-> {
                println("Received message: $msg")
                getContext().system().shutdown()
            }
            else -> unhandled(msg)
        }
    }

}

class WorldActor: UntypedActor() {

    public override fun onReceive(msg: Any?) {
        when (msg) {
            is String -> getSender().tell(msg.toUpperCase() + " world");
            else -> unhandled(msg);
        }
    }
}
```

## quasar + kotlin

```kotlin
data class Msg(val txt: String, val from: ActorRef<Any?>)

class Ping(val n: Int) : Actor() {
    Suspendable override fun doRun() {
        val pong = ActorRegistry.getActor<Any?>("pong")
        for(i in 1..n) {
            pong.send(Msg("ping", self()))          // Fiber-blocking
            receive {                               // Fiber-blocking
                when (it) {
                    "pong" -> println("Ping received pong")
                    else -> null                    // Discard
                }
            }
        }
        pong.send("finished")                       // Fiber-blocking
        println("Ping exiting")
    }
}

class Pong() : Actor() {
    Suspendable override fun doRun() {
        while (true) {
            // snippet Kotlin Actors example
            receive(1000, TimeUnit.MILLISECONDS) {  // Fiber-blocking
                when (it) {
                    is Msg -> {
                        if (it.txt == "ping")
                            it.from.send("pong")    // Fiber-blocking
                    }
                    "finished" -> {
                        println("Pong received 'finished', exiting")
                        return                      // Non-local return, exit actor
                    }
                    is Timeout -> {
                        println("Pong timeout in 'receive', exiting")
                        return                      // Non-local return, exit actor
                    }
                    else -> defer()
                }
            }
            // end of snippet
        }
    }
}

public class Tests {
    Test public fun testActors() {
        spawn(register("pong", Pong()))
        spawn(Ping(3))
    }
}
```

https://gist.github.com/abreslav/5046126

## See Also

* macroid
* http://blog.paralleluniverse.co/2015/05/21/quasar-vs-akka/
* http://blog.paralleluniverse.co/2015/06/04/quasar-kotlin/
