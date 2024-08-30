# Swift

- [Swift](#swift)
  - [Comments](#comments)

This page contains contents that are mostly about the language itself or the compiler.

## Comments

Comments don't add much to the compile time. In fact, `LLVM` even has a vectorized comment skipper. It lets the CPU work on a chunk of the input with a single operation, rather than each byte/character. For comment skipping you’re basically trying to scan a byte stream for the literals `/*` or `//`, and then if you find a `/*` you switch to looking for a `*/`. This problem vectorizes fairly cleanly and avoids linear processing.
