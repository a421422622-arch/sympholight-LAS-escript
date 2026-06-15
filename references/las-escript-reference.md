# LAS e:script Reference Notes

These notes summarize practical facts from e:cue LAS 7.0 SR1 System Manual and Advanced System Manual. They are not a replacement for `Help | e:script Language Reference` inside LAS Programmer.

## Mental Model

LAS has several layers:

- `Programmer`: highest capability; can run e:script, generate effects, control media, receive triggers, update Action Pad, and output DMX through engines.
- `Cuelist`: stores cues, timings, fades, loops, actions, and show playback.
- `Cue Action`: runs Actions when a cue is played; can call macros, start/stop cuelists, media, serial/MIDI, etc.
- `Trigger`: executes Actions when a condition/event is met.
- `Action Pad`: user-facing GUI; can trigger Actions and can be manipulated by e:script.
- `Butler S2/XT2`: standalone-capable DMX engines; exported show data is rendered to device memory/SD. Standalone supports fewer actions than Programmer.

Use e:script to connect these layers, not to replace them.

## e:script Characteristics

- C-like macro language embedded in LAS Programmer.
- Case-sensitive command and variable names.
- Statements end with semicolons.
- Supports integers, strings, arrays, loops, if/switch, functions, arithmetic, and global values.
- Macros are usually short one-time snippets. For continuous behavior, store state globally and call the macro repeatedly from a looping cuelist or trigger.
- Many LAS object IDs in e:script are zero-based. For example, `CuelistStart(0)` starts visible cuelist 1.

## Confirmed Capability Areas

### Cuelists

Useful commands and concepts:

```c
CuelistStart(0);     // visible cuelist 1
CuelistStop(0);
CuelistStopAll();    // respects release times
ResetAll();          // immediate reset style behavior
```

Status and metadata:

```c
CuelistGetName(0);
CueGetCurrent(0);       // current cue, zero-based, or -1
CuelistGetStatus(0, 0); // status field; parameter meaning depends on reference
CueGetCount(0);
```

Cue properties:

```c
CueGetProperty(cuelist_id, cue_id, "Command");
CueSetProperty(cuelist_id, cue_id, "Command", 1);    // 1 = Wait
CueSetProperty(cuelist_id, cue_id, "Para0", 5000);   // wait/runtime in ms
CueSetProperty(cuelist_id, cue_id, "InFadeTime", 800);
```

Known command values from the manual:

- `0`: Manual
- `1`: Wait, with `Para0` as runtime in ms
- `2`: Timecode, with `Para0` as timecode in ms
- `3`: Until end of cuelist, with `Para0` as cuelist ID

### Programmer View and Fixtures

Typical pattern:

```c
int fix_type;
int fix_count;

fix_type = GetFixtureTypeId("RGBFader");
if (fix_type < 0) {
    MessageError("Fixture Type RGBFader not found in show.");
    exit;
}

fix_count = GetFixtureTypeCount(fix_type);

proSelectOut();
proSelectRange(fix_type, 0, fix_count);
SetPosition(65535);   // 100%; value range 0..65535
proLoadValue(0);      // channel index is zero-based
proLoadValue(1);
proLoadValue(2);
```

Use fixture profile names exactly as patched in LAS. Do not assume channel order; verify in the fixture profile and DMX chart.

### Live FX

The manual describes:

```c
SetFx(func_id, size, speed, offset, ratio);
ClearFx();
```

Common function IDs include:

- `1`: Sin
- `2`: Cos
- `3`: Ramp
- `4`: Rectangle
- `5`: Triangle
- `8`: Ping Pong

Use Live FX carefully for demos. Slow, subtle motion looks more professional than aggressive chase/flicker.

### LoadValue

`LoadValue` can write directly to output using a running cuelist handle and desk channel. It can apply fades and delays. It is more advanced and closer to core output behavior than `proLoadValue`.

Use only after confirming the exact function signature in LAS reference and after identifying desk channels with commands such as `GetDeskChannel`.

### Action Pad

Useful commands:

```c
ActionPadSetCurrentPage(0);
ActionPadSetCurrentPageByScriptID("demo");
ActionPadGetCurrentPageIndex();
ActionPadSetItemText(0, "Status: <cuelist 1 status>");
ActionPadSetItemBackgroundColors(0, RGB(0, 160, 80), RGB(0, 80, 40));
```

Action Pad item member codes are per page and visible in edit mode. Switch to the correct page before updating an item.

Auto Text can display live status such as:

```text
<cuelist 1 status>
<cuelist 1 current>
<cuelist 1 name>
```

Visible cuelist numbers in Auto Text are not zero-based.

### External Communication

e:script can interact with:

- HTTP client driver
- UDP
- TCP client driver
- RS-232
- MIDI
- Network Parameters

Use these to demonstrate integration with AV control, touch panels, sensors, or third-party control systems. Add required drivers in Device Manager first and use the configured driver alias/handle.

### Network Parameters

LAS exposes a network parameter set useful for communication with external applications and Action Pad controls. Treat it as a shared variable interface. Confirm index mapping and write/read direction before use.

## Standalone Boundaries

For Butler standalone, distinguish between:

- Rendered DMX show data exported to the engine.
- Triggers/actions transferred through Device Manager/Quick Update.
- Programmer-only runtime logic that will not run on the engine.

Important patterns:

- Define at least one startup/initialization trigger so the standalone engine does something after power-up or leaving online mode.
- Check the export log for unsupported triggers/actions.
- Use `Check only` export before final export when possible.
- For SD card file export, remember that show data alone may not include triggers/actions; use Quick Update for those.

Known device capability themes from LAS System Manual:

- Butler S2 and Butler XT2 support 1024 DMX channels and up to 99 cuelists.
- Butler S2/XT2 can run multiple cuelists simultaneously, but supported standalone actions differ.
- Butler XT2 has stronger standalone interaction support than S2, including Remote Action Pad support in the documented workflow.

## Recommended Answer Shape

When answering LAS scripting questions, use:

1. Assumptions and missing facts.
2. Recommended LAS layer: Cuelist, Action, Trigger, Action Pad, e:script, or device export.
3. e:script draft.
4. Where to attach it: macro list, Cue Action, Trigger, Action Pad button, or export workflow.
5. Test checklist.

