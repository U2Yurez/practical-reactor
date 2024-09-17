In Project Reactor, you can manage and propagate context within reactive sequences using the `Context` API. This API allows you to carry additional state or context through the reactive stream, which can be particularly useful for things like user authentication, tracing, or logging.

Here's a step-by-step guide on how to add and use context in a reactive sequence with Project Reactor:

### 1. **Create a `Context`**

The `Context` in Project Reactor is a simple, immutable map of key-value pairs. You create it using the `Context.of()` method.

```java
import reactor.util.context.Context;

Context context = Context.of("key", "value");
```

### 2. **Pass Context to a Reactive Sequence**

You can use `Mono.deferContextual()` or `Flux.deferContextual()` to access the `Context` within your reactive streams. Here’s how you can pass and access context:

#### Example: Adding Context to a `Mono`

```java
import reactor.core.publisher.Mono;
import reactor.util.context.Context;

public class ContextExample {

    public static void main(String[] args) {
        Mono<String> monoWithContext = Mono.deferContextual(ctx ->
                Mono.just("Hello, " + ctx.getOrDefault("name", "Guest"))
        ).contextWrite(Context.of("name", "World"));

        monoWithContext.subscribe(System.out::println);  // Output: Hello, World
    }
}
```

In this example:
- `Context.of("name", "World")` creates a context with the key `"name"` and value `"World"`.
- `Mono.deferContextual()` allows us to access this context within the `Mono` chain.

#### Example: Adding Context to a `Flux`

```java
import reactor.core.publisher.Flux;
import reactor.util.context.Context;

public class FluxContextExample {

    public static void main(String[] args) {
        Flux<String> fluxWithContext = Flux.deferContextual(ctx ->
                Flux.just("Item1", "Item2")
                    .map(item -> item + " for " + ctx.getOrDefault("user", "Guest"))
        ).contextWrite(Context.of("user", "Alice"));

        fluxWithContext.subscribe(System.out::println);  // Output: Item1 for Alice
                                           //         Item2 for Alice
    }
}
```

### 3. **Modifying Context**

You can also modify the context dynamically during processing. Use `contextWrite()` to update the context.

```java
Mono<String> modifiedContextMono = Mono.deferContextual(ctx ->
        Mono.just("Current context: " + ctx.getOrDefault("status", "unknown"))
).contextWrite(Context.of("status", "active"));

modifiedContextMono.subscribe(System.out::println);  // Output: Current context: active
```

### 4. **Context Propagation in Asynchronous Code**

If you have asynchronous code (e.g., `Schedulers` or other async operations), you might need to ensure the context is propagated correctly. Reactor handles context propagation for most scenarios automatically, but if you need explicit control, you can use `Context`.

For example, if you're working with custom `Schedulers`, you might want to ensure the context is passed along:

```java
Mono.deferContextual(ctx ->
        Mono.just("Data with context: " + ctx.getOrDefault("info", "No Info"))
).contextWrite(Context.of("info", "Important Info"))
.subscribeOn(Schedulers.boundedElastic())
.subscribe(System.out::println);
```

### Summary

In Project Reactor, the `Context` API allows you to inject and manage additional state within your reactive streams. By using `Context`, `contextWrite()`, and `deferContextual()`, you can pass along contextual information through your reactive sequences effectively. This is useful for scenarios like user-specific data, tracing, or logging.