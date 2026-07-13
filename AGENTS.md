# Contributor and Agent Guide

## Mission

Document how the Dark Ages 4.21 Stone client works from startup through shutdown. Cover architecture, control flow, rendering, audio, input, Windows messages, internal event routing, file formats, network framing, serialization, and transformations such as XOR.

The result must help a Dark Ages community developer understand the client while remaining precise enough to support proxy, server, tool, and compatible-client implementations.

## Audience

Write for four overlapping groups:

1. Dark Ages client developers who already know the game and may know older, newer, or custom client versions.
2. Proxy and server developers who understand the protocol and want to follow messages into client-side behavior.
3. Developers investigating a specific client quirk, edge case, or behavior difference between versions.
4. Developers implementing file formats that remain in use across clients and community tools.

Assume familiarity with the game, but not with the internals of this exact executable. Do not spend time teaching basic game concepts unless a detail is required to understand the implementation. Explain old Windows, DirectX, compiler, and reverse engineering concepts when they affect the result.

Make pages useful as both a guided explanation and a lookup reference. A reader should be able to enter through a visible behavior, packet, file format, or known function and reach the other relevant views through links.

## Non-negotiable rules

1. Do not commit game executables, DLLs, assets, archives, IDA databases, memory dumps, packet captures, or other copyrighted game material.
2. Only publish behavior that is confirmed directly from the code or supported with high confidence by code patterns and observed behavior.
3. Do not derive behavior from names or nearby strings alone. Trace instructions, call sites, data flow, state changes, or runtime behavior.
4. Do not use em dashes or emojis.
5. Do not copy large decompiler listings. Explain behavior in original prose and use small C-like pseudocode when it improves clarity.
6. Do not patch the executable as part of documentation work unless the task explicitly requires a controlled experiment.

## Documentation model

Each subsystem page explains the same system at two levels and ends with a function table.

### High-level operation

Explain what the subsystem does in natural language. Describe its lifecycle, owned state, inputs, outputs, and relationships with other systems. Use important function names when they make the flow concrete, but keep addresses and dense implementation detail out of the main explanation.

Divide a large subsystem into useful topics such as initialization, resource loading, main processing, event handling, and shutdown.

When applicable, connect the explanation to observable game behavior, related packet messages, consumed file formats, and established differences from other client versions.

### Code-level flow

Describe the implementation as a call tree or ordered path through the code. Include:

- Function address and current IDA name
- Calling convention and prototype when known
- Meaning of arguments and return values
- Important callers and callees
- Branch conditions, loops, state transitions, and dispatch decisions
- Globals, object fields, buffers, or tables read and written
- Ownership, lifetime, failure paths, and cleanup

Use short C-like pseudocode where a decision or algorithm is clearer as code. A pseudocode identifier is explanatory and does not imply that the original program used that source-level name.

### Function table

End each subsystem page with a compact reference table:

| Address | Current IDA name | Prototype | Purpose | Call relationships and notes |
|---:|---|---|---|---|

The global function map in `docs/appendices/function-map.md` is a cross-subsystem index. The subsystem table is the primary reference for understanding a specific system.

## Documentation threshold

The published book is not a notebook of competing hypotheses. Add a behavior when either:

- Direct static analysis establishes it through instructions, control flow, data flow, callers, and callees.
- Code patterns and controlled runtime observations agree strongly enough to make the interpretation reliable.

If a lead is still speculative, keep it in local notes or mark a tentative IDA name with `_maybe`. Do not add it to the published explanation until the interpretation reaches the threshold above.

Use addresses in this form: `Darkages.exe:0x00401234`, current IDA name `net_receive_packet`. Addresses are virtual addresses from the analyzed image unless a page explicitly labels them as file offsets or RVAs.

## Research workflow

1. Define a narrow question before exploring.
2. Find candidate code through imports, strings, cross-references, callers, and callees.
3. Read both decompiler output and disassembly where types, signedness, control flow, or optimizer artifacts matter.
4. Trace inputs, outputs, state changes, ownership, and error paths.
5. Compare multiple call sites before assigning a semantic name or type.
6. Update the IDA database with established names, types, local names, and concise comments.
7. Add the high-level explanation, code-level flow, and function table entries to the relevant subsystem page.
8. Update the global function or data map when the result is useful across subsystems.

## IDA database conventions

Use lower snake case names and a subsystem prefix:

- `app_` for process lifecycle and top-level coordination
- `audio_` for sound and music
- `file_` for file access and container handling
- `fmt_` for a specific format encoder or decoder
- `input_` for keyboard, mouse, and device input
- `net_` for direction-neutral sockets, framing, messages, and transformations
- `net_c_` for established client-to-server packet builders, serializers, and direction-specific helpers
- `net_s_` for established server-to-client packet readers, handlers, and direction-specific helpers
- `render_` for graphics, surfaces, palettes, and presentation
- `ui_` for windows, controls, screens, and widgets
- `event_` for internal dispatch and queues
- `win_` for Win32 integration and window procedures
- `util_` only for genuinely cross-cutting helpers

Use names that describe observed behavior, such as `net_read_u16_be`, rather than unproven intent. If a tentative IDA rename is useful during active investigation, use an `_maybe` suffix and remove it before documenting the function as established.

Keep an owning-subsystem prefix when direction is secondary. For example, a pane method that dispatches server packets remains `ui_...`, while a network-owned server packet parser uses `net_s_...`.

For each understood function:

- Rename the function to describe its established role.
- Apply the calling convention, return type, and parameter types supported by the code.
- Rename parameters, locals, globals, and structure fields when their roles are known.
- Add a short function comment describing the contract, major side effects, and important callers.
- Add line comments where an operation, constant, state transition, or format field would otherwise be unclear.
- Preserve the original address in the subsystem function table and global function map.

Do not rename a broad family of functions from pattern similarity alone. Check representative call sites and note meaningful exceptions.

## Documentation structure

- `docs/architecture/` describes the whole client and relationships between subsystems.
- `docs/lifecycle/` follows startup, main loop, state transitions, and shutdown.
- `docs/subsystems/` explains major runtime systems at both documentation levels.
- `docs/formats/` records on-disk structures and algorithms.
- `docs/protocol/` records transport, framing, message schemas, and transformations.
- `docs/appendices/` holds cross-subsystem maps, addresses, terminology, and sample identity.

Add new pages to `docs/SUMMARY.md`. Keep one clear subject per page. Use matching topic headings in the high-level and code-level sections when that makes navigation easier.

## Writing style

Write in direct, natural English. Start with what a component does, then explain how it does it. Define old Windows, DirectX, networking, and compiler terms the first time they matter.

Prefer short paragraphs and concrete names. Use tables for packet fields, file layouts, state values, and function references. Use diagrams only when they make control or data flow easier to understand.

Pseudocode must be implementation-neutral C-like code. Use fixed-width types such as `uint8_t` when the width is established. Show byte order explicitly.

## Protocol and format rules

For every packet or file structure, record:

- Direction or file context
- Framing and total length rules
- Field offsets, widths, signedness, and byte order
- String encoding and termination rules
- Optional or version-dependent fields
- Validation, bounds behavior, and failure path
- Transformation order, including compression, checksums, encryption, or XOR
- Reader, writer, handler, and dispatch function addresses
- Client-side effects that matter to server, proxy, or compatible-client implementations

Use `unknown_XX` for a field with an established location but unknown meaning, where `XX` is the field offset. Do not call a transformation encryption unless the code and protocol behavior support that term.

## Commit messages

Always use [Conventional Commits](https://www.conventionalcommits.org/) for commit messages. Use the form `type(scope): description`, or `type: description` when a scope would not add useful context.

Use a short, imperative, lowercase description without a trailing period. Common types for this repository are `docs`, `research`, `ci`, `chore`, `fix`, `refactor`, `build`, and `test`. Use `!` and a `BREAKING CHANGE:` footer when a change is intentionally incompatible.

Examples:

- `docs(network): describe receive buffer framing`
- `research(audio): map directsound initialization`
- `ci: deploy mdbook to github pages`
- `chore: initialize reverse engineering documentation`

## Before finishing a change

- Confirm that no prohibited binary or captured material is staged.
- Check links from `docs/SUMMARY.md`.
- Confirm that every added behavior meets the documentation threshold.
- Check IDA synchronization for any functions or data analyzed during the change.
- Check that each edited subsystem retains its high-level, code-level, and function-table structure.
- Build the mdBook when the tool is available.
- Use a Conventional Commit message for every commit.
