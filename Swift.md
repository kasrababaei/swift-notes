# Swift

- [Swift](#swift)
  - [Protocol Oriented Programming (POP)](#protocol-oriented-programming-pop)
  - [Functional Programming](#functional-programming)
  - [Comments](#comments)
  - [Compound Types](#compound-types)
  - [Type Annotations](#type-annotations)
  - [Tuple Types](#tuple-types)
  - [Function Type](#function-type)
    - [Autoclosures](#autoclosures)
  - [Codable](#codable)
  - [Type Erasure](#type-erasure)
  - [Delegates](#delegates)

This page contains contents that are mostly about the language itself or the
compiler. It also contains a few concepts like delegates that at the moment
can't find a better place to document them.

## Protocol Oriented Programming (POP)

Swift is known to be a POP programming language. Protocol-Oriented Programming
(POP) in Swift is a paradigm that emphasizes protocols and protocol extensions
as a means of designing flexible, reusable, and modular code. Swift’s design
encourages developers to compose functionality using protocols, rather than
relying exclusively on inheritance hierarchies as in traditional
Object-Oriented Programming (OOP).

> [!NOTE]
> POP was part of a talk that Dave Abrahams gave in WWDC 2015 when Swift protocols
> started having default implementation (the famous story about Crusty).
> However, everyone misunderstood it and thought POP is _a thing_ which made it a
> buzzword in Swift.
>
> ![image Crusty](./.images/.cursty.png)
>
> In fact, that video is removed from Apple's website. The goal of that session
> was to push for using generics and creating creating generic algorithms.

In essence, the paradigm advocates using protocols and value types that conform to
protocols whereas OOP is mainly talking about inheritance.

## Functional Programming

Functional programming (FP) in Swift is a paradigm that emphasizes writing code
using pure functions, immutability, and avoiding side effects. In FP, functions
are treated as first-class citizens, meaning they can be passed as parameters,
returned from other functions, and assigned to variables.

Key concepts in Swift:

- **Pure Functions**: Functions that always return the same output for the
same input and have no side effects, making them predictable and easy to test.
- **Immutability**: Variables and constants, especially with `let`, are
preferred to avoid unintended state changes, which improves code safety and concurrency.
- **Higher-Order Functions**: Swift includes functions like `map`, `filter`,
and `reduce` that allow transforming collections without mutating them, central
to FP.
- **Closures**: Anonymous functions, or closures, make Swift flexible and enable
functional-style chaining of operations.

## Comments

Comments don't add much to the compile time. In fact, `LLVM` even has a vectorized
comment skipper. It lets the CPU work on a chunk of the input with a single operation,
rather than each byte/character. For comment skipping you’re basically trying to
scan a byte stream for the literals `/*` or `//`, and then if you find a `/*`
you switch to looking for a `*/`. This problem vectorized fairly cleanly and
avoids linear processing.

## Compound Types

A compound type is a type without a name, defined in the Swift language itself.
There are two compound types: function types and tuple types. A compound type may
contain named types and other compound types. For example, the tuple type
`(Int, (Int, Int))` contains two elements: The first is the named type Int, and
the second is another compound type (Int, Int).

## Type Annotations

A type annotation explicitly specifies the type of a variable or expression.
Type annotations begin with a colon (:) and end with a type, as the following
examples show:

```Swift
let someTuple: (Double, Double) = (3.14159, 2.71828)
func someFunction(a: Int) { /* ... */ }
```

## Tuple Types

A tuple type is a comma-separated list of types, enclosed in parentheses.
You can use a tuple type as the return type of a function to enable the function
to return a single tuple containing multiple values. You can also name the elements
of a tuple type and use those names to refer to the values of the individual elements.
An element name consists of an identifier followed immediately by a colon (:).

When an element of a tuple type has a name, that name is part of the type.

```Swift
// someTuple is of type (top: Int, bottom: Int)
var someTuple = (top: 10, bottom: 12) 
someTuple = (top: 4, bottom: 42) // OK: names match
someTuple = (9, 99)              // OK: names are inferred
someTuple = (left: 5, right: 5)  // Error: names don't match
```

All tuple types contain two or more types, except for Void which is a type alias
for the empty tuple type, ().

## Function Type

A function type represents the type of a function, method, or closure and consists
of a parameter and return type separated by an arrow `(->)`:

```Swift
(<#parameter type#>) -> <#return type#>
```

### Autoclosures

An [autoclosure](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/closures/#Autoclosures)
is a closure that’s automatically created to wrap an expression that’s being
passed as an argument to a function. It doesn’t take any arguments, and when
it’s called, it returns the value of the expression that’s wrapped inside of it.
This syntactic convenience lets you omit braces around a function’s parameter by
writing a normal expression instead of an explicit closure.

An autoclosure lets you delay evaluation, because the code inside isn’t run until
you call the closure. Delaying evaluation is useful for code that has side effects
or is computationally expensive, because it lets you control when that code is
evaluated. The code below shows how a closure delays evaluation.

```Swift
var customersInLine = ["Chris", "Alex", "Ewa", "Barry", "Daniella"]
print(customersInLine.count)
// Prints "5"

let customerProvider = { customersInLine.remove(at: 0) }
print(customersInLine.count)
// Prints "5"

print("Now serving \(customerProvider())!")
// Prints "Now serving Chris!"
print(customersInLine.count)
// Prints "4"
```

Even though the first element of the customersInLine array is removed by the code
inside the closure, the array element isn’t removed until the closure is actually
called. If the closure is never called, the expression inside the closure is never
evaluated, which means the array element is never removed. Note that the type of
customerProvider isn’t String but () -> String — a function with no parameters
that returns a string.

You get the same behavior of delayed evaluation when you pass a closure as an argument
to a function.

## Codable

A type that conforms to this protocol, is a type that can convert itself into and
out of an external representation. This protocol is just a typealias for:

- `Encodable`: A type that can encode itself to an external representation.
- `Decodable`: A type that can decode itself from an external representation.

When dealing with a JSON like the following, can change the `keyDecodingStrategy`
to map snake_case to camel_case.

```Swift
let json = """
[
    {
        "first_name": "Paul",
        "last_name": "Hudson"
    },
    {
        "first_name": "Andrew",
        "last_name": "Carnegie"
    }
]
"""

struct User: Codable {
    var firstName: String
    var lastName: String
}

let decoder = JSONDecoder()
decoder.keyDecodingStrategy = .convertFromSnakeCase
```

Alternatively, can map different key names by using types that conform to `CodingKey`.

```Swift
struct User: Codable {
    var id: Int
    var name: String
    var isActive: Bool

    enum CodingKeys: String, CodingKey {
        case id = "user_id"
        case name = "full_name"
        case isActive = "is_active"
    }
}

let jsonData = """
{
    "user_id": 123,
    "full_name": "John Doe",
    "is_active": true
}
""".data(using: .utf8)!

do {
    let decoder = JSONDecoder()
    let user = try decoder.decode(User.self, from: jsonData)
} catch {
    print("Error decoding JSON: \(error)")
}
```

There are many ways of working with dates on the internet, but ISO-8601 is the
most common. It encodes the full date in YYYY-MM-DD format, then the letter “T”
to signal the start of time information, then the time in HH:MM:SS format, and
finally a timezone. The timezone “Z”, short for “Zulu time” is commonly used to
mean UTC.

```Swift
let data = """
{"time_of_birth": "1999-04-03T17:30:31Z"}
""".data(using: .utf8)!

struct Birth: Decodable {
  let timeOfBirth: Date
}

let decoder = JSONDecoder()
decoder.keyDecodingStrategy = .convertFromSnakeCase
decoder.dateDecodingStrategy = .iso8601
let timeOfBirth = try decoder.decode(Birth.self, from: data).timeOfBirth
```

Alternatively, it's possible to use a custom date formatter:

```Swift
let data = """
{"time_of_birth": "31-08-2001"}
""".data(using: .utf8)!

struct Birth: Decodable {
  let timeOfBirth: Date
}

let decoder = JSONDecoder()
decoder.keyDecodingStrategy = .convertFromSnakeCase
let formatter = DateFormatter()
formatter.dateFormat = "dd-MM-yyyy"
decoder.dateDecodingStrategy = .formatted(formatter)
let timeOfBirth = try decoder.decode(Birth.self, from: data).timeOfBirth
```

When dealing with more complicated and hierarchical data structure, can implement
the decoding/encoding calls. This requires CodingKeys as well but for that, can
also use a less type-safe approach:

```Swift
let data = """
{
  "name": "Ginger",
  "details": {
    "breed": "Corgi",
    "age": 2
  }
}
""".data(using: .utf8)!

struct AnyCodingKey: CodingKey, ExpressibleByStringLiteral {
  var intValue: Int?
  var stringValue: String
  
  init?(intValue: Int) { nil }
  
  init?(stringValue: String) {
    self.stringValue = stringValue
  }
  
  init(stringLiteral value: StringLiteralType) {
    self.stringValue = value
  }
}

struct Dog: Decodable {
  let name: String
  let breed: String
  let age: Int
  
  init(from decoder: any Decoder) throws {
    let container = try decoder.container(keyedBy: AnyCodingKey.self)
    self.name = try container.decode(String.self, forKey: "name")
    let details = try container.nestedContainer(
      keyedBy: AnyCodingKey.self,
      forKey: "details"
    )
    self.breed = try details.decode(String.self, forKey: "breed")
    self.age = try details.decode(Int.self, forKey: "age")
  }
}

let decoder = JSONDecoder()
let dog = try decoder.decode(Dog.self, from: data)
print(dog)
```

## Type Erasure

Type erasure is an abstraction principle where some information about a type is
removed at run-time. By hiding the specific type of an instance, can work with
instances of different types in a more flexible way while still maintaining
type safety. It's also useful when types are long and complex, for instance publishers.

Another use-case for type erasure is when dealing with generics. It allows using
a single type with some extra functionality that wraps other concrete types. In
the example below, the property `prettyName` has some common logic that gets shared
between different types of `Factory` but the exact type of `Factory` gets hidden
from the caller:

```Swift
protocol Factory {
  associatedtype Product
  var product: Product { get }
}

extension Factory {
  func eraseToAnyFactory() -> AnyFactory<Self.Product> {
    AnyFactory(product)
  }
}

struct Tesla: Factory {
  let product = "Model3"
}

struct Ford: Factory {
  let product = "F150"
}

struct AnyFactory<Product>: Factory {
  private let _product: () -> Product
  var product: Product { _product() }
  var prettyName: String { "Product name is: \(product)" }
  
  init(_ product: Product) {
    self._product = { product }
  }
}

for factory in [Tesla().eraseToAnyFactory(), Ford().eraseToAnyFactory()] {
  print(factory.prettyName)
}
```

## Delegates

A delegate is an object that acts on behalf of, or in coordination with, another
object when that object encounters an event in a program<sup>[*](https://developer.apple.com/library/archive/documentation/General/Conceptual/CocoaEncyclopedia/DelegatesandDataSources/DelegatesandDataSources.html)</sup>.
