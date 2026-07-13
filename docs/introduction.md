# Introduction

This book explains the internal design of the Dark Ages 4.21 Stone client. It follows the program from process startup into its main loop, then separates the client into rendering, audio, input, user interface, file, event, and network systems.

Each subsystem is explained at two levels. The high-level section describes responsibilities, lifecycle, and data flow in approachable language. The code-level section follows function calls, decisions, addresses, arguments, and state. A function table closes each subsystem for readers who want a compact technical reference.

This is a living reverse engineering reference. Empty sections are intentional at the start of the project. They define where understood behavior will go without filling gaps with speculation.

## Audience

The book is written primarily for people in the Dark Ages developer community. It assumes that many readers already understand the game, its terminology, and the broad shape of its clients and protocol. It does not assume that every reader is comfortable reading disassembly or following compiler-generated C++.

## Ways to use this book

### Understand a client system

Start with a subsystem's high-level section to see its responsibilities, lifecycle, and connections to the rest of the client. Continue into the code-level section when you need the actual call path, arguments, decisions, and state.

### Connect network behavior to client behavior

Protocol and networking pages connect message framing and fields to the client functions that parse, dispatch, and act on them. This path is intended for proxy and server developers who already know the wire side and want to understand what happens after a message reaches the client.

### Investigate a quirk or version difference

Use subsystem headings and function tables to move from visible behavior to the responsible code. Where differences with another client version are established, the relevant page should state what changed and link the comparison to exact functions or data structures.

### Implement a file format

Format pages provide field layouts, byte order, string rules, reader and writer behavior, validation, and parser quirks. They are intended to remain useful for formats that continue to appear in other clients and community tools.

Familiarity with C-like code is helpful for the technical sections. Platform-specific Windows, DirectX, compiler, and networking details are explained when they affect the client behavior being discussed.

## Current state

Repository structure and documentation conventions are established. Subsystem behavior has not yet been mapped.
