# Dark Ages 4.21 Client Reverse Engineering

[Published docs](https://ewrogers.github.io/darkages-421-re/)

Technical documentation of the Dark Ages 4.21 Stone client.

This project maps how the client works from process startup through shutdown. The research covers rendering, audio, input, Windows messages, internal event routing, file formats, and the client-server protocol.

The resulting documentation is intended for preservation researchers, private server developers, compatible client authors, and anyone curious about the internals of this generation of the game.

## Project status

The project is in its initial research phase. The documentation structure and contribution conventions are established, but most client subsystems have not been mapped yet.

The analyzed executable is identified by SHA-256 so findings can be reproduced against the same input. See [Analyzed Binary](docs/appendices/binary-identity.md) for the current sample identity and its unresolved version-resource discrepancy.

## Who this is for

This project is primarily written for the Dark Ages developer community. It should be useful whether a reader already knows the client well or is approaching one part of it for the first time.

- Client developers can use it to understand how the 4.21 client organizes a familiar feature and compare it with older, newer, or custom clients.
- Proxy and server developers can follow a network message past the wire format and into the client-side state changes, handlers, and user interface behavior it causes.
- Developers investigating a specific quirk can start from visible behavior, find the responsible subsystem, and continue into exact functions and branch decisions.
- Format implementers can use the file documentation as a practical reference for layouts, decoding rules, parser behavior, and formats that remain in use.

## Goals

- Explain the overall client architecture and lifecycle.
- Document rendering, audio, input, user interface, file, and event systems.
- Describe supported file formats and their exact serialized layouts.
- Map network framing, packet structures, serialization, and transformations such as XOR.
- Maintain a technical function and data map tied to stable executable addresses.
- Keep useful names, types, variables, and comments synchronized with the local IDA database.
- Present the results as a readable mdBook without distributing proprietary game material.

## Documentation

Start with the [Introduction](docs/introduction.md) or browse the complete [book summary](docs/SUMMARY.md).

Each subsystem is documented at two levels. The first explains how the system works in approachable language. The second follows the implementation through function calls, decisions, addresses, arguments, and state changes. Every subsystem ends with a compact function table.

To build the book locally, install [mdBook](https://rust-lang.github.io/mdBook/), then run:

```text
mdbook serve --open
```

The generated site is written to `book/` and is ignored by Git.

Pull requests automatically build the book as a validation check. Changes merged into `main` are deployed to [GitHub Pages](https://ewrogers.github.io/darkages-421-re/) by the repository workflow.

## Repository layout

```text
.github/workflows/pages.yml  mdBook validation and GitHub Pages deployment
docs/
  architecture/   High-level components and relationships
  lifecycle/      Startup, main loop, state changes, and shutdown
  subsystems/     Rendering, audio, input, events, files, and networking
  formats/        On-disk file and container formats
  protocol/       Transport, messages, fields, and transformations
  appendices/     Binary identity, function map, data map, and glossary
.local/ida/       Optional local IDA workspace, ignored by Git
AGENTS.md          Research, naming, structure, and writing conventions
book.toml          mdBook configuration
```

## Getting started

Clone the repository and enter the project directory:

```text
git clone git@github.com:ewrogers/darkages-421-re.git
cd darkages-421-re
```

Read [Scope and Method](docs/scope-and-method.md) and [AGENTS.md](AGENTS.md) before beginning an investigation. These documents define the documentation threshold, the two-level subsystem structure, and how IDA names should be organized.

If you are reproducing the reverse engineering work, use the executable identified in [Analyzed Binary](docs/appendices/binary-identity.md). Keep the executable, game assets, captures, and supporting libraries outside the repository.

## Contributing research

Focused contributions are easier to verify and review than broad collections of notes. A typical contribution should:

1. Begin with one clear question about client behavior.
2. Trace the relevant instructions, data flow, callers, and callees.
3. Compare the code with observed behavior where that helps establish the interpretation.
4. Update IDA names, types, variable names, and comments as functions become understood.
5. Explain the subsystem at both the high level and the code level.
6. Add important functions to the subsystem table and global function map.

Only add behavior that has been confirmed from the code or is supported with high confidence by code patterns and observed behavior. Decompiled output is a view of the binary, not original source code. Prefer original prose and short C-like pseudocode over copied decompiler listings.

Before opening a pull request, check local Markdown links, build the book, and make sure no proprietary or private analysis material is staged.

## Local IDA workspace

The working IDA database may be stored at `.local/ida/`. The entire `.local/` directory and all common IDA database extensions are ignored by Git.

This location is only a private convenience for local research. Never force-add an IDA database, executable, game asset, packet capture, memory dump, or generated analysis artifact.

## Legal

Dark Ages and all original game software, artwork, music, text, trademarks, and related materials belong to their respective owners. This is an independent, unofficial research project and is not affiliated with or endorsed by the game's developers, publishers, or rights holders.

The project is undertaken in good faith for education, scholarship, preservation, commentary, and interoperability research under fair-use principles. Read the full [Legal Disclaimer](docs/legal-disclaimer.md) before contributing or using this material.
