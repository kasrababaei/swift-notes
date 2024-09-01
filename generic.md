# Generic types

- [Generic types](#generic-types)
  - [Opaque return type vs generics](#opaque-return-type-vs-generics)
  - [Boxed protocol](#boxed-protocol)

## Opaque return type vs generics

- [SE-0244: Opaque Result Types](https://github.com/apple/swift-evolution/blob/main/proposals/0244-opaque-result-types.md)
- [Swift documentation: Opaque and Boxed Types](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/opaquetypes/)

Assuming we want to create a type that conforms to the `Shape` protocol, there's different ways to do it.

```Swift
protocol Shape {
    func draw() -> String
}
```

- Using generics, by writing `struct VerticalShapes<S: Shape>` and `var shapes: [S]`, makes an array whose elements are some specific shape type, and where the identity of that specific type is visible to any code that interacts with the array.
- Using an opaque type, by writing `var shapes: [some Shape]`, makes an array whose elements are some specific shape type, and where that specific type’s identity is hidden.
- Using a *boxed protocol* type, by writing `var shapes: [any Shape]`, makes an array that can store elements of different types, and where those types’ identities are hidden.

## Boxed protocol

A *boxed protocol* type is also sometimes called an existential type, which comes from the phrase “there exists a type T such that T conforms to the protocol”. To make a boxed protocol type, write any before the name of a protocol.

```Swift
protocol AnyShape {
    func draw() -> String
}

protocol Shape: AnyShape, Equatable {
    //
}

struct Square: Shape {
    let size: Int
    func draw() -> String {
        let line = String(repeating: "*", count: size)
        let spaces = String(repeating: " ", count: size - 2)
        
        return """
        \(line)
        *\(spaces)*
        *\(spaces)*
        \(line)
        """
    }
}

struct Triangle: Shape {
    func draw() -> String {
        """
           *
          * *
        *     *
        *******
        """
    }
}

struct Merge2<T: Shape, U: Shape>: Shape {
    let first: T
    let second: U
    
    init(_ first: T, _ second: U) {
        self.first = first
        self.second = second
    }
    
    func draw() -> String {
        "\(first.draw())\n\(second.draw())"
    }
}

struct Diamond: Shape {
    func draw() -> String {
        """
           *
          * *
         *   *
          * *
           *
        """
    }
}

struct Flip<T: Shape>: Shape {
    var shape: T
    func draw() -> String {
        "\(shape.draw().reversed())"
    }
}

// Opaque : when you want the function to decide the return type
// Generic: when you want the caller to decide the return type

enum Shapes {
    // The caller of this method, still thinks it's getting a Shape
    // Why not just return Merge? It'll expose types we don't want to.
    // We lose the ability to change Merge to be another type in the future.
    // Also, we'll lose the abality to run some operations such as == since it needs to know the
    // type of the left and right side of the operator.
    static func merge2<T: Shape, U: Shape>(_ first: T, _ second: U) -> some Shape {
        // An important proviso here is that functions with opaque return types must always return
        // one specific type.
        // The body of this function knows exactly the return type, which is Merge2.
        let merged = Merge2(first, second)
        return merged
    }
    
    // Not acceptable : static func flip(_ shape: any Shape) -> any Shape
    // Also acceptable: static func flip<T: Shape>(_ shape: T) -> any Shape {
    static func flip(_ shape: some Shape) -> any Shape {
        if shape is Square {
            // Using return tome `some Shape` would throw an error because it sees that the body
            // can potentially return two different types.
            
            // Error:
            // Function declares an opaque return type 'some Shape', but the return statements in
            // its body do not have matching underlying types
            return shape
        } else {
            return Flip(shape: shape)
        }
    }
    
    static func random<T: Shape, U: Shape>(_ first: T, _ second: U) -> any Shape {
        [first, second].randomElement()!
    }
}

let square = Square(size: 2)
let triangle = Triangle()
let diamond = Diamond()
let mergedOne = Shapes.merge2(square, triangle)
let mergedTwo = Shapes.merge2(Square(size: 4), triangle)

// This wouldn't be possible with `any Shape`
// Opaque types solve this problem because even though we just see a protocol being used,
// internally the Swift compiler knows exactly what that protocol actually resolves to
mergedOne == mergedTwo

// By using generics, the caller can decide the type.
let flip: AnyShape = Shapes.random(square, triangle)
```
