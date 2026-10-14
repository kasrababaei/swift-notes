# AI

A collection of my findings about using AI in programming

## Adding Cursor to Xcode

[This article](https://www.polpiella.dev/cursor-xcode-27) explains
it in very simple steps. I tested it in Xcode 26.6 (17F113).

## Listing all models

Just run `agent --list-models` to list all your models.

## Cursor

### Interaction modes

Cursor has a few interaction modes.

| | What it does | Edits code? | Best for |
| --- | --- | --- | --- |
| **Ask** | Read-only Q&A / exploration | No | “How does this work?” without changing anything |
| **Plan** | Research → clarifying questions → a plan you review → **Build** | Only after you approve | Multi-file / ambiguous / architectural work |
| **Debug** | Hypothesize → instrument → you reproduce → fix from logs | Yes (targeted) | Tricky bugs, races, perf where guessing fails |
| **Goal** | Long-lived objective (`/goal …`) the agent keeps working toward | Yes | Open-ended work until done (e.g. “make CI green”) |

## Coding Agent

Xcode stores files that are related to the coding agent in the following directory:

```text
$HOME/Library/Developer/Xcode/CodingAssistant/ClaudeAgentConfig
```

Any feedback that needs to be stored for following sessions gets stored under the
project name. Example, for a project called *Isolation*:

```text
$HOME/Library/Developer/Xcode/CodingAssistant/ClaudeAgentConfig/projects/-Users-[name]-iphone/memory
```

where `[name]` would be your username.

### Exporting Skills

The SwiftUI Specialist Skill can help you follow SwiftUI best practices in your
apps. The What's New In SwiftUI Skill can guide you through adopting new APIs
from the 2027 releases. And to use these skills with other tools, you can export
them with the `xcrun agent skills export` command.

This will create markdown files you can import in your workflows.
