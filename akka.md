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

https://gist.github.com/abreslav/5046126

## See Also

* macroid
* http://blog.paralleluniverse.co/2015/05/21/quasar-vs-akka/
