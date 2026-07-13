# File System

The file subsystem covers path construction, lookup order, file access, containers, caching, and the shared readers and writers used by format-specific code.

## High-level operation

This section will explain where the client looks for data, how resources are identified, which content is loaded eagerly or on demand, and how file data reaches rendering, audio, user interface, and game systems.

File system behavior has not been mapped yet. Specific layouts will be linked from the [Format Catalog](../formats/overview.md).

## Code-level flow

This section will trace path construction, open and close operations, archive or container access, allocation, common byte and string readers, bounds handling, caching, decoding, and cleanup. Format-specific flows will link back to their reader and writer functions.

No file system call tree is documented yet.

## Function table

| Address | Current IDA name | Prototype | Purpose | Call relationships and notes |
|---:|---|---|---|---|
| | | | No functions documented yet | |
