# Ruby

## frozen_string_literal

What it does:

- Makes all string literals in the file frozen by default
- Improves performance by allowing Ruby to reuse identical string objects
  instead of creating new ones
- Reduces memory allocations and garbage collection pressure

If you delete it:

- The code will still work exactly the same functionally
- String literals become mutable (can be modified)
- Slightly higher memory usage (Ruby creates new String objects each time)
- In this specific file, it makes virtually no difference since there are only
  a few string literals

Example difference:
Without `frozen_string_literal`

```ruby
str = "hello"
str << " world"  # This works, modifies the string
```

With `frozen_string_literal: true`

```ruby
str = "hello"
str << " world"  # This raises FrozenError
```

It's a Ruby community best practice to include it (especially in gems and
libraries), which is why it appears in git_metadata.rb too. You can safely
remove it from this file without breaking anything, but keeping it maintains
consistency with the rest of the codebase.

## system(...) vs sh(...)

*system(...):*

Built-in Ruby method:

- Returns true/false/nil based on exit status
- Does NOT raise exception on failure
- Silent about what command is running
- Always available (no requires needed)

```ruby
success = system("xcrun simctl create ...")
# Returns false if command fails, but continues execution
```

*sh(...):*

Rake/FileUtils method:

- Raises RuntimeError if command fails (non-zero exit)
- Prints the command being executed (verbose by default)
- Requires Rake context (require 'rake' or inside Rakefile)
- Stops execution on failure (unless rescued)

```ruby
sh "xcrun simctl create ..."
# Prints: xcrun simctl create ...
# Raises exception and halts if command fails
```
