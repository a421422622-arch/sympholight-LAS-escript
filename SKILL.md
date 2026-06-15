---
name: ecue-las-escript
description: Write, review, and design e:cue Lighting Application Suite LAS e:script macros and control logic. Use when the user asks about e:cue LAS, Programmer, e:script, cuelist scripting, Cue Actions, Triggers, Action Pad scripting, Butler standalone export readiness, DMX fixture test macros, or stage/architectural lighting control demos using LAS.
---

# e:cue LAS e:script

Use this skill to design practical e:script macros for e:cue LAS Programmer and to connect them cleanly with cuelists, Cue Actions, Triggers, Action Pad pages, and Butler standalone workflows.

## Core Approach

Treat e:script as the control/automation layer, not the show engine.

- Put primary lighting looks and timed fades in `Cuelist`s.
- Use e:script for reset logic, scene routing, validation, Action Pad status, fixture tests, external communication, and small generated effects.
- Keep macros short. LAS e:script is intended for one-time snippets; for continuous behavior, use a looping cuelist or repeated trigger plus persistent/global state.
- Confirm exact command names in `Help | e:script Language Reference` when writing production code. Do not invent unsupported commands.
- State uncertainty when the target LAS version, device, fixture profile, or standalone capability is unknown.

## Before Writing Code

Collect or infer these inputs first:

- LAS version and target mode: online Programmer, Butler S2 standalone, Butler XT2 standalone, or mixed.
- Fixture types, fixture profile names, DMX mode, and channel table.
- Cuelist numbers/names and whether IDs should be zero-based in script.
- Cue timing intent: manual, wait, timecode, loops, fade-in/out.
- Action Pad page script IDs and item member codes if updating UI.
- External interfaces: UDP, TCP, HTTP client, MIDI, RS-232, dry contacts, e:bus, DALI.
- Export target and standalone constraints.

If any of these are missing, write a clear assumption block before code.

## Workflow

1. Define the control objective in plain language.
2. Choose the simplest LAS layer:
   - Cuelist only for fixed looks and fades.
   - Cue Action for an action at a precise cue.
   - Trigger for external/time/event activation.
   - Action Pad for operator control and status.
   - e:script only when built-in Actions are too limited or repeated logic is needed.
3. Decide online vs standalone.
   - Online Programmer supports the broadest scripting and external integration.
   - Butler standalone mainly plays exported cuelists and supported triggers/actions.
4. Write small macros with explicit IDs and comments for zero-based numbering.
5. Include a test checklist: start/stop, blackout/full, export check, Action Pad feedback, and failure state.

## Reference Files

- Read `references/las-escript-reference.md` before answering detailed scripting questions or reviewing e:script.
- Read `references/demo-macro-patterns.md` when designing a lighting demo, Action Pad interface, system check, or Butler export workflow.

## Code Style

- Prefer named constants for cuelist IDs, fixture type names, Action Pad item IDs, and magic values.
- Use zero-based comments whenever a visible LAS number maps to a script ID.
- Keep macros idempotent where possible: a reset macro should be safe to run twice.
- Avoid long blocking loops. Use cuelist waits/triggers for timing instead.
- Log meaningful status with `printf` or user messages when useful for commissioning.
- Separate "show look" logic from "control routing" logic.

## Common Boundaries

- Do not promise motorized zoom, focus, shutter, fan, or color-temperature control unless the fixture DMX chart confirms those channels.
- Do not assume a Programmer-only macro will run inside a Butler in standalone mode.
- Do not leave standalone demo cuelists in manual-step mode unless a supported trigger is explicitly configured.
- Do not rely on Action Pad browser/mobile behavior without testing the actual LAS version and client environment.

