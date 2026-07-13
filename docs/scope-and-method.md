# Scope and Method

## Scope

The project covers behavior implemented by the analyzed `Darkages.exe`, including:

- Process initialization, configuration, main loop, state changes, and shutdown
- Rendering, surface management, palette behavior, sprites, text, and presentation
- Sound effects, music, resource lifetime, and playback control
- Keyboard, mouse, Win32 messages, and internal event delivery
- File lookup, archive or container access, decoding, parsing, and serialization
- Socket lifetime, connection states, packet framing, message dispatch, and field serialization
- Byte transformations such as XOR when their exact behavior and order are understood
- Compiler and runtime mechanisms when they materially explain application behavior

Compatibility wrappers such as cnc-ddraw are outside the client architecture unless their interaction reveals a client requirement worth documenting.

## Clean-room policy

The repository contains original documentation, diagrams, tables, and limited explanatory pseudocode derived from analysis. It does not contain the executable, supporting DLLs, game assets, private runtime captures, an IDA database, or reconstructed source code.

Local copies may be used as analysis inputs. Proprietary materials must never be committed. The ignored `.local/` directory may hold a private working IDA database.

## Two levels of explanation

Every subsystem chapter has a high-level section and a code-level section.

The high-level section explains responsibilities, lifecycle, data flow, and interactions with other parts of the client. It should be understandable without following assembly or decompiler output.

The code-level section follows the real implementation. It records call trees, function addresses, prototypes, arguments, return values, branches, state changes, important data, and cleanup paths. The chapter ends with a function table for readers who need a compact technical reference.

## What belongs in the book

The book contains behavior confirmed through direct code analysis or supported with high confidence when code patterns and observed behavior agree. Speculative leads remain in local notes or tentative IDA names until they are understood well enough to explain clearly.

The decompiler is a useful view of the program, but its types and reconstructed control flow are not automatically correct. Disassembly and call sites should be checked whenever signedness, ownership, object layout, calling convention, or optimizer behavior affects the explanation.

## Address and naming conventions

A function reference uses the form `Darkages.exe:0x00401234`, current IDA name `net_receive_packet`. Unless a page says otherwise, addresses are virtual addresses from the analyzed image rather than file offsets or RVAs.

IDA names are descriptive research names. They are not presented as original source symbols. The function address remains the stable reference if a name improves later.
