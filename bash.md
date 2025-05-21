# Bash

- [Bash](#bash)
  - [Shebang or hash-bang](#shebang-or-hash-bang)
  - [Writing Contents](#writing-contents)
  - [Exit](#exit)
  - [RakeFile vs Makefile: Summary Comparison](#rakefile-vs-makefile-summary-comparison)
    - [When to Use](#when-to-use)
  - [File Conditions](#file-conditions)
  - [Directories](#directories)
  - [Working with Strings and Files](#working-with-strings-and-files)
  - [Generate XCFileList](#generate-xcfilelist)
  - [Download and Unzip](#download-and-unzip)
  - [Sparse Checkout](#sparse-checkout)

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

## File Conditions

To check if a file exists:

```bash
FILE_NAME="Sample.txt"

if [ ! -e "$FILE_NAME" ]; then
    touch "$FILE_NAME"
fi
```

To check if the content of the file is or is not empty:

```bash
if [ -z "$(cat "$FILE_NAME")" ]; then
    echo "Is empty"
else
    echo "Not empty"
fi
```

It's also possible to use `-n` for not empty, example:

```bash
if [ -n "$(cat "$FILE_NAME")" ]; then
    echo "Is not empty"
fi
```

## Directories

To get the directory where the bash script is located at:

```bash
DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" >/dev/null 2>&1 && pwd )"
```

## Working with Strings and Files

To read content a file and assign it to a variable:

```bash
FILE="./List.txt"
CONTENT=$(cat "$FILE")
```

To read content of a file line by line:

```bash
while read -r line; do
  echo "Line: ${line}"
done <<< "$FILE_PATHS"
```

In case we want to create a multiline string variable but we don't want the
trailing `\n`, can do:

```bash
CONTENT=""
while read -r line; do
  CONTENT="$CONTENT"$'\n'"$LINE"
done <<< "$FILE_PATHS"
```

For string comparison:

```bash
if [ "$str1" == "$str2" ]; then
    echo "Equal"
fi

if [ "$str1" != "$str2" ]; then
    echo "Not Equal"
fi
```

Always quote the variables `"$str1"` to avoid issues with spaces or empty strings.

It's also possible to use double brackets `[[  ]]` instead of single
brackets `[  ]`. Single brackets are the original and POSIX-compliant syntax.
It is more portable (works in sh, dash, etc.), but has more limitations.
Double brackets are a Bash-only (and Zsh/Ksh) enhancement. Safer and more powerful:

- No need to quote variables (though it's still good practice).
- Supports regex matching `=~`, pattern matching, and better string comparisons.
- `<` and `>` are not treated as redirection operators.
- Logical operators like `&&` and `||` are supported inside.

## Generate XCFileList

The following script can create an XCFileList of Swift files that created/modified/copied
which then can be fed to lint tools such as SwiftLint. It also maps wildcards
such as `**` or `*` to regex because

```Bash
#!/bin/bash

set -e

echo "Generating Swift XCFileList..."

SCRIPT_DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" >/dev/null 2>&1 && pwd )"
ROOT=$(realpath "${SCRIPT_DIR}/../../" )

pushd "${ROOT}" > /dev/null

XCFILELIST="${ROOT}/Tooling/Linters/Swift.generated.xcfilelist"

if [ ! -e "$XCFILELIST" ]; then
  # Always generate the XCFileList since it is used as an input file list by Xcode
  # when running the SwiftLint and SwiftFormat scripts. This is to ensure that those
  # scripts won't fail due to missing inputs.
  touch "$XCFILELIST"
fi

if [[ -n ${IS_CI+x} ]]; then
  echo "Skipping generating XCFileList when building on CI"
  exit 0
fi

# Excludes files that are not in the Sources directory.
# The `|| true` is used to prevent the script from failing if no files are found.
FILE_PATHS=$(git diff --name-only --diff-filter=ACMD HEAD -- '*.swift' | grep '^Sources/' || true)

if [[ -z "$FILE_PATHS" ]]; then
  echo "No Swift file was added or modified."
  exit 0
fi

SWIFTLINT_CONFIG="${ROOT}/.swiftlint.yml"
if [[ ! -e "$SWIFTLINT_CONFIG" ]]; then
  echo "error: Missing shared rules. Run 'make' in repo's root directory." >&2
  exit 1
fi

HAS_EXCLUDED_FILE_PATHS=false
declare -a EXCLUDED_FILE_PATHS

while read -r line; do
  if [[ "${line}" == "excluded:" ]]; then
    HAS_EXCLUDED_FILE_PATHS=true
  fi

  if [[ $HAS_EXCLUDED_FILE_PATHS = true && "${line}" =~ ^\s*-.* ]]; then
    FILE_PATH="\\\${SRCROOT}/${line#*- }"
    FILE_PATH=$(echo "$FILE_PATH" | perl -pe 's/\./\\./g')
    FILE_PATH=$(echo "$FILE_PATH" | perl -pe 's/\*\*/.*/g')
    FILE_PATH=$(echo "$FILE_PATH" | perl -pe 's/(?<!\.)\*(?!\.)/\\w*/g')

    EXCLUDED_FILE_PATHS+=("$FILE_PATH")
  elif [[ $HAS_EXCLUDED_FILE_PATHS = true && -z "${line}" ]]; then
    break
  fi
done < "${SWIFTLINT_CONFIG}"

EXPANDED_FILE_PATHS=""

while read -r line; do
  FILE_PATH="\${SRCROOT}/${line#Sources/}"

  for EXCLUDED_FILE_PATH in "${EXCLUDED_FILE_PATHS[@]}"; do
    if [[ "${FILE_PATH}" =~ $EXCLUDED_FILE_PATH ]]; then
      echo "Excluded file: ${FILE_PATH}"
      continue 2
    fi
  done

  if [ -z "$XCODE_VERSION_ACTUAL" ]; then
    # Expand the file path. This is needed when the script isn't run
    # by Xcode, which expands the paths automatically.
    FILE_PATH=$(eval echo "$FILE_PATH")
  fi

  if [[ -z "$EXPANDED_FILE_PATHS" ]]; then
    EXPANDED_FILE_PATHS="$FILE_PATH"
  else
    EXPANDED_FILE_PATHS="$EXPANDED_FILE_PATHS"$'\n'"$FILE_PATH"
  fi
done <<< "$FILE_PATHS"

if [[ "$(cat "$XCFILELIST")" = "$EXPANDED_FILE_PATHS" ]]; then
  exit 0
else
  echo "$EXPANDED_FILE_PATHS" > "$XCFILELIST"
fi

# Used when used by another script like the pre-commit hook.
export XCFILELIST

pushd > /dev/null
```

## Download and Unzip

To download a zip file and unzip it, can use the following code block:

```bash
curl --fail "${artifact_url}" -L -o "${TEMP_DIR}/${tool_name}.zip" || {
  echo "error: Failed to download ${tool_name} artifact." >&2
  exit 1
}

# Unzip the artifact quietly, then copy the binary to the bin directory and make it executable
unzip -q "${TEMP_DIR}/${tool_name}.zip" -d "${TEMP_DIR}" || {
  echo "error: Failed to unzip ${tool_name} artifact." >&2
  exit 1
}
```

## Sparse Checkout

To checkout a remote repo, but a sparse checkout, to only copy over a certain
file(s), run:

```bash
git -C "${TEMP_DIR}" sparse-checkout init --cone
git -C "${TEMP_DIR}" checkout
ACTION_DIR="${SOME_DIR}/${REPO}"
mkdir -p "${NEW_DIR}"
cp -f "${TEMP_DIR}/${FILENAME}" "${NEW_DIR}/${FILENAME}" || {
  echo "error: Failed to copy ${FILENAME}." >&2
  exit 1
}
```
