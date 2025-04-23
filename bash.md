# Bash

- [Bash](#bash)
  - [Shebang or hash-bang](#shebang-or-hash-bang)
  - [Writing Contents](#writing-contents)
  - [Exit](#exit)
  - [RakeFile vs Makefile: Summary Comparison](#rakefile-vs-makefile-summary-comparison)
    - [When to Use](#when-to-use)

A shell script is a computer program designed to be run by a Unix shell, a
command-line interpreter. Typical operations performed by shell scripts
include file manipulation, program execution, and printing text.

## Shebang or hash-bang

In computing, a shebang is the character sequence `#!`, consisting of the
characters number sign (also known as sharp or hash) and exclamation mark (also
known as bang), at the beginning of a script. It is also called
sharp-exclamation, sha-bang, hashbang, pound-bang, or hash-pling.
The shebang, or hash-bang, is a special kind of comment which the system uses
to determine what interpreter to use to execute the file. The shebang must
be the first line of the file, and start with "#!". In Unix-like operating
systems, the characters following the `#!` prefix are interpreted as a path
to an executable program that will interpret the script.

Some typical shebang lines:

- `#! /bin/sh`: – Execute the file using the [Bourne shell](https://en.wikipedia.org/wiki/Bourne_shell),
or a compatible shell, assumed to be in the _/bin_ directory
- `#! /bin/bash`: – Execute the file using the [Bash shell](https://en.wikipedia.org/wiki/Bash_(Unix_shell))
- `#! /usr/bin/pwsh`: – Execute the file using _PowerShell_
- `#! /usr/bin/env python3`: – Execute with a Python interpreter, using the `env`
program search path to find it
- `#! /bin/false`: – Do nothing, but return a non-zero exit status, indicating
failure. Used to prevent stand-alone execution of a script file intended for
execution in a specific context, such as by the `.` command from _sh/bash_,
`source` from _csh/tcsh_, or as a .profile, .cshrc, or .login file.

Here's a good [cheat sheet for bash](https://devhints.io/bash).

## Writing Contents

The >> symbols following our echo command redirects the output of echo to our
file. A single > will replace the entire file contents while a double >> will
append to the file.

```bash
echo "" >> file.txt
echo "" > file.txt
```

To create a new file if it doesn't exist, and clear the contents:

```bash
touch > file.txt
true > file.txt // Option 1
echo "" > file.txt // Option 2
```

## Exit

Shell functions provide their return value in a variable named 1 and their error
message in a variable named 2. We access the output and error message by using
a 1 or 2 after the function call.

Adding _error:_ to the message is a pattern that [Xcode recognizes from scripts/logs](https://developer.apple.com/documentation/xcode/running-custom-scripts-during-a-build#Log-errors-and-warnings-from-your-script).

```bash
#!/bin/bash

echo "error: This is a placeholder which shows up in output navigator." >&2
exit 1
```

## RakeFile vs Makefile: Summary Comparison

| Feature                 | **RakeFile**                                       | **Makefile**                                        |
| ----------------------- | -------------------------------------------------- | --------------------------------------------------- |
| **Language**            | Ruby-based                                         | Make-specific syntax using shell commands           |
| **Flexibility**         | Highly flexible (full Ruby capabilities)           | Limited to build tasks and shell scripting          |
| **Speed**               | Slower (Ruby interpreted)                          | Faster (optimized for compiling/building)           |
| **Cross-Platform**      | Works anywhere Ruby is available                   | Widely supported; may need tweaks on Windows        |
| **Ease of Use**         | Clean syntax for Ruby developers                   | Simple but can be cryptic, especially for beginners |
| **Dependency Handling** | Built-in and flexible via Ruby syntax              | Native and efficient dependency tracking            |
| **Adoption**            | Popular in Ruby community                          | Extremely widespread, especially in C/C++ projects  |
| **Error Handling**      | Straightforward with Ruby exception handling       | Requires shell-specific handling (e.g., `set -e`)   |
| **Use Case**            | Complex workflows, automation, testing, deployment | Compiling source code, running shell-based tasks    |

### When to Use

- **Use RakeFile if:**
  - You’re working in Ruby or want to write custom automation tasks.
  - You need high-level logic, scripting, or integration with Ruby gems.

- **Use Makefile if:**
  - You’re compiling C/C++ or managing a lightweight shell-based build process.
  - You want a fast, minimal setup for task automation.
