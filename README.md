# <samp>init</samp>

This data pack provides function tag `#minecraft:init` to be called on the initial `#minecraft:load` after the server process is started.

## How it works

A `StringTag` larger than 65535 bytes can exist in memory with no problem.
However, when it is tried to be saved to a command storage file, a `UTFDataFormatException` occurs and an empty string is saved instead by `StringFallbackDataOutput`.
Once this happens, the `StringTag` becomes out of sync between in-memory (too long) and on-file (empty).
Since the same command storage file is never loaded more than once during the lifetime of the server process, this out of sync will continue until the server is stopped.
When the server process is started again, an empty `StringTag` is loaded from the command storage file.

In light of the above, `#minecraft:init` can be implemented as follows:
```mermaid
---
title: The lifecycle of a server process
---
flowchart TD
  Start --> Load
  Load --> C1{{Does the StringTag exist?}}
  C1 -- Yes --> C2{{Is the StringTag empty?}}
  C1 -- No --> C3
  C2 -- Yes --> C3
  C3[Create a too long StringTag] --> Init
  Init --> Tick
  C2 -- No --> Tick
  Tick --> Tick
  Tick -- Reload --> Load
  Tick --> Stop
  Stop -- Restart --> Start
```
