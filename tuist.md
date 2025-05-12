# Tuist

- [Tuist](#tuist)
  - [Manifest](#manifest)

Since I've been exploring different Xcode project (and dependency) management
tools, and Tuist is an emerging tool that is widely getting adopted by many
teams, thought it's good to collect my findings in a file.

## [Manifest](https://docs.tuist.dev/en/guides/develop/projects/manifests)

Tuist defaults to Swift files as the primary way to define projects and workspaces
and configure the generation process. These files are referred to as manifest files.
This decision was inspired by Swift Package Management, which also uses Swift files
to define the package. This also means we can leverage the compiler to validate
the correctness of the content and reuse code across different manifest files,
and Xcode to provide a first-class editing experience thanks to the syntax
highlighting, auto-completion, and validation.

Since manifest files are Swift files that need to be compiled, Tuist caches the
compilation results to speed up the parsing process. Therefore, you'll notice
that the first time you run Tuist, it might take a bit longer to generate the
project. Subsequent runs will be faster.
