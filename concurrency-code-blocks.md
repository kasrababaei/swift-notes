# Swift Concurrency Code Blocks

- [Swift Concurrency Code Blocks](#swift-concurrency-code-blocks)
  - [withTaskCancellationHandler](#withtaskcancellationhandler)

This file contains a few code blocks that use Swift Concurrency. Swift Concurrency is changing rapidly; so, it is possible to find a few blocks that aren't valid anymore.

## withTaskCancellationHandler

[Beyond the basics of structured concurrency](https://developer.apple.com/wwdc23/10170) that came out in WWDC2024, talks about the [withTaskCancellationHandler](https://developer.apple.com/documentation/swift/withtaskcancellationhandler(operation:oncancel:isolation:)#) method for cancelling an ongoing `Task`. In the following code-block, because the cancellation handler runs immediately, the state machine is shared mutable state between the cancellation handler and main body, which can run concurrently.

```swift
await withTaskCancellationHandler {
  let result = await kitchen.generateOrder()
  guard state.isRunning else {
    return nil
  }
  return result
} onCancel: {
  state.cancel()
}
```

Evan Wilde states:

> While actors are great for protecting encapsulated state, we want to modify and read individual properties on our state machine, so actors aren't quite the right tool for this.
>
> Furthermore, we can't guarantee the order that operations run on an actor, so we can't ensure that our cancellation will run first. We'll need something else. I've decided to use atomics from the Swift Atomics package, but we could use a dispatch queue or locks.

That's why in the example, they're using another class that uses `ManagedAtomic` under the hood:

```swift
private final class OrderState: Sendable {
  let protectedIsRunning = ManagedAtomic<Bool>(true)
  var isRunning: Bool {
    get { protectedIsRunning.load(ordering: .acquiring) }
    set { protectedIsRunning.store(newValue, ordering: .relaxed) }
  }
  func cancel() { isRunning = false }
}
```

The issue is, `ManagedAtomic` is part of [swift-atomic](https://github.com/apple/swift-atomics) package and it's not always easy to justify adding that as a dependency of your project to use for this instance. Also, by looking at the repo's description, we find more reasons to avoid it:

> Atomic values are fundamental to managing concurrency. However, they are far too low level to be used lightly. These things are full of traps. They are extremely difficult to use correctly. It's always better to rely on higher-level constructs, whenever possible.

I actually looked how Apple is using this method in their packages, and found out that [swift-corelibs-foundation](https://github.com/swiftlang/swift-corelibs-foundation/tree/e55e1d88001997be830fbc01086564431d405dad) uses a type called `CancelState` for cancelling [URLSessionTask](https://github.com/swiftlang/swift-corelibs-foundation/blob/e55e1d88001997be830fbc01086564431d405dad/Sources/FoundationNetworking/URLSession/URLSession.swift#L735-L754):

```swift
public func data(for request: URLRequest, delegate: URLSessionTaskDelegate? = nil) async throws -> (Data, URLResponse) {
  let cancelState = CancelState()
  return try await withTaskCancellationHandler {
    try await withCheckedThrowingContinuation { continuation in
      let completionHandler: URLSession._TaskRegistry.DataTaskCompletion = { data, response, error in
        if let error = error {
          continuation.resume(throwing: error)
        } else {
          continuation.resume(returning: (data!, response!))
        }
      }
      let task = dataTask(with: _Request(request), behaviour: .dataCompletionHandlerWithTaskDelegate(completionHandler, delegate))
      task._callCompletionHandlerInline = true
      task.resume()
      cancelState.activate(task: task)
    }
  } onCancel: {
    cancelState.cancel()
  }
}
```

[CancelState](https://github.com/swiftlang/swift-corelibs-foundation/blob/e55e1d88001997be830fbc01086564431d405dad/Sources/FoundationNetworking/URLSession/URLSession.swift#L686) is actually using a `Mutex` under the hood:

```swift
final class CancelState: Sendable {
  struct State {
    var isCancelled: Bool
    var task: URLSessionTask?
  }
  
  let lock = Mutex<State>(State(isCancelled: false, task: nil))
  init() {
  }
  
  func cancel() {
    let task = lock.withLock { state in
      state.isCancelled = true
      let result = state.task
      state.task = nil
      return result
    }
    task?.cancel()
  }
  
  func activate(task: URLSessionTask) {
    let taskUsed = lock.withLock { state in
      if state.task != nil {
        fatalError("Cannot activate twice")
      }
      if state.isCancelled {
        return false
      } else {
        state.isCancelled = false
        state.task = task
        return true
      }
    }
    
    if !taskUsed {
      task.cancel()
    }
  }
}
```

Sadly, [Mutext](https://developer.apple.com/documentation/synchronization/mutex#) is available in iOS 18. It seems like `Mutex` [is using os_unfair_lock under the hood](https://github.com/swiftlang/swift/blob/main/stdlib/public/Synchronization/Mutex/DarwinImpl.swift). The official documentation discourages developers from using `os_unfair_lock` and recommends using [OSAllocatedUnfairLock](https://developer.apple.com/documentation/os/osallocatedunfairlock#) instead. The officual documentation mentions:

> If you’ve existing Swift code that uses os_unfair_lock, change it to use OSAllocatedUnfairLock to ensure correct locking behavior.