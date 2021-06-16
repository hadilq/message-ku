# Message KU

This library is so similar to an event bus or a message queue. The differences are

- For what we called event or message in other libraries here we named them _command_.
- Commands are in request-result pairs, which means for each request command we have a result
  command.
- No broadcast command, because this library matches the results with requests, which is not
  achievable for broadcasts.

To handle the request-result pairs we used Kotlin coroutines' biggest power, which is making
callbacks similar to ordinary lines of code with an input and output.

## Usage

Let's have some `RequestCommand` and `ResultCommand` like

```kotlin
data class RequestCommand(val request: String) : Command
data class ResultCommand(val result: String) : Command
```

we want to have a few lines of coroutines code to send the request and receive the result.

```kotlin
suspend fun CommandExecutor.runCommand(request: RequestCommand): CommandResult<ResultCommand> =
  exe(request)

launch {
  when (val result = executor.runCommand(RequestCommand("Are you there?"))) {
    is Available<*> -> assert(result.command == ResultCommand("Yes! Of course!"))
  }
}
```

There are a few notes here.

- It doesn't have any thread executor, so the coroutine context of calling `exe` will be used to
run all `suspend` functions.
- As you can see it's possible that no callback is registered to handle the requested command,
in that case the `is Available<*>` branch of `when` will *not* be called.

To receive the request and respond to it

```kotlin
val registration = commandRegister.register(RequestCommand::class,
  CommandCallbackImpl(commandShooter) {
    ResultCommand("Yes! Of course!")
  })
```

whenever is needed we can dispose the `registration`.