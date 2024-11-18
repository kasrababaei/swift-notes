# RunLoop

## Timer

Assigning `Timer` to `nil` isn't sufficient to stop it and it must be
invalidated; otherwise, because setting to `nil` would cause the timer to
deallocate and invoke its deinit function. But since the timer is retained
by the `RunLoop`, that cannot happen. Therefore setting to `nil` does
nothing except clear out your local property.

```Swift
final class Foo {
    var timer: Timer?

    init() {
        timer = Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { _ in
            print("Hello, World!")
        }
    }

    deinit {
        timer.invalidate()
    }
}
```

When using a timer in the Playground, need to assign `needsIndefiniteExecution`
to true:

```Swift
PlaygroundPage.current.needsIndefiniteExecution = true

let timer = Timer.scheduledTimer(withTimeInterval: 2, repeats: true) { _ in
    print("Hello, World!")
}

Thread.sleep(forTimeInterval: 10)
```

But when using it in a Swift Package Manager, have to keep the RunLoop running:

```Swift
final class Foo {
    var timer: Timer?

    init() {
        timer = Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { _ in
            print("Hello, World!")
        }

        RunLoop.current.run(until: Date() + 10)
    }

    deinit {
        timer.invalidate()
    }
}
```

This is because unlike Playground that has behind the scenes code to run the
`RunLoop`, SPM doesn't provide that.
