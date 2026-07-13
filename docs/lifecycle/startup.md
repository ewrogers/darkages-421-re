# Startup and Shutdown

This chapter follows the process from the PE entry point through runtime initialization, window creation, subsystem setup, the main loop, and shutdown.

## High-level lifecycle

This section will explain startup order, command-line and directory handling, window creation, subsystem dependencies, entry into the main loop, normal exit, connection loss, fatal errors, and cleanup.

The client lifecycle has not been mapped yet.

## Code-level flow

This section will trace the PE entry point, compiler runtime handoff, `WinMain` or its equivalent, initialization calls, failure branches, window procedure connection, main loop, and each shutdown path.

No lifecycle call tree is documented yet.

## Function table

| Address | Current IDA name | Prototype | Purpose | Call relationships and notes |
|---:|---|---|---|---|
| | | | No functions documented yet | |
