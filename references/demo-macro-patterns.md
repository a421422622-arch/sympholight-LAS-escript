# Demo Macro Patterns

Use these patterns for stage-lighting or architectural-lighting demos in e:cue LAS.

## Demo Architecture

Build the demo as layers:

- `QL01_System_Test`: fixture/channel/color test.
- `QL02_Presentation`: warm or neutral key light.
- `QL03_Product_Highlight`: focused subject light plus subdued background.
- `QL04_Warm_Drama`: warm low-CCT atmosphere.
- `QL05_Cool_Tech`: cool saturated atmosphere.
- `QL06_Color_Transition`: color fade across multi-color fixtures.
- `QL07_Slow_Chase`: subtle dynamic effect.
- `QL08_Emergency_Full`: safe full light.
- `QL09_Blackout`: full off.
- `QL10_Demo_Timeline`: sequencer or scripted route through scenes.

Use cuelists for looks. Use e:script for routing, validation, and UI feedback.

## Pattern: Reset and Safe Start

Purpose: stop all current playback and return to a known starting condition before demo.

```c
// Demo_ResetAll
// Safe to run before every customer demo.

CuelistStopAll();
// Use ResetAll() only if immediate hard reset is desired.
// ResetAll();

printf("Demo reset executed.\n");
```

Commissioning note: if Grand Master or Versatile Masters must be set, confirm the exact master commands in LAS Language Reference before adding them.

## Pattern: Scene Router

Purpose: one macro behind multiple Action Pad buttons. Keep visible cuelist numbers in comments.

```c
// Demo_PlayScene
// Assumption: call separate copies or pass a numeric parameter if the LAS action setup supports it.
// IDs are zero-based: 1 visible = 0 script.

int scene;
scene = 2; // example: Product Highlight

CuelistStopAll();

switch (scene) {
    case 1:
        CuelistStart(1); // visible QL02 Presentation
        break;
    case 2:
        CuelistStart(2); // visible QL03 Product Highlight
        break;
    case 3:
        CuelistStart(3); // visible QL04 Warm Drama
        break;
    case 4:
        CuelistStart(4); // visible QL05 Cool Tech
        break;
    case 5:
        CuelistStart(6); // visible QL07 Slow Chase
        break;
    default:
        CuelistStart(1); // fallback: Presentation
        break;
}
```

For production, prefer one of these:

- Create one small macro per scene if parameterized macro calls are awkward in the UI.
- Use Action `Cuelist` directly if no logic is needed.
- Use Mutual Exclude Groups if available and appropriate for the target mode.

## Pattern: Action Pad Status Update

Purpose: show customers that the UI reflects system state.

```c
// Demo_UpdateActionPad
// Switch to the status page before addressing item member codes.

ActionPadSetCurrentPageByScriptID("demo_status");
ActionPadSetItemText(0, "Current: <cuelist 1 status>");
ActionPadSetItemText(1, "Presentation: <cuelist 2 status>");
ActionPadSetItemText(2, "Product: <cuelist 3 status>");
```

If using colors:

```c
ActionPadSetItemBackgroundColors(1, RGB(0, 130, 90), RGB(0, 80, 50));
```

Confirm `RGB()` and color command behavior in the target LAS version.

## Pattern: Fixture Type Existence Check

Purpose: avoid running a test macro against missing profiles.

```c
// Demo_CheckFixtureTypes

if (GetFixtureTypeId("RGBFader") < 0) {
    MessageError("Missing fixture type: RGBFader");
    exit;
}

printf("Fixture type check passed.\n");
```

Replace `RGBFader` with actual patched fixture type names, e.g. the V23/X45/D03 profile names in LAS.

## Pattern: Simple RGB Fixture Test

Purpose: prove channel control on a known RGB-style fixture profile.

```c
// Demo_RGBWhite
// Only use when channel 0/1/2 are confirmed as RGB or equivalent profile channels.

int fix_type;
int fix_count;

fix_type = GetFixtureTypeId("RGBFader");
if (fix_type < 0) {
    MessageError("Fixture Type RGBFader not found.");
    exit;
}

fix_count = GetFixtureTypeCount(fix_type);

proSelectOut();
proSelectRange(fix_type, 0, fix_count);

SetPosition(65535);
proLoadValue(0);
proLoadValue(1);
proLoadValue(2);

printf("RGB white test loaded into Programmer View.\n");
```

For real fixtures, replace channel indices with the confirmed profile channels. Multi-color stage lights may not expose channels as simple RGB in this order.

## Pattern: Standalone Readiness Check

Purpose: find manual cues that can stall a Butler standalone demo.

```c
// Demo_CheckManualCues
// Reports cues with Command == 0 (Manual).

int q;
int c;
int count;
int found;

found = 0;

for (q = 0; q < 99; q++) {
    count = CueGetCount(q);
    if (count > 0) {
        for (c = 0; c < count; c++) {
            if (CueGetProperty(q, c, "Command") == 0) {
                printf("Manual cue found: visible QL %d, visible Cue %d\n", q + 1, c + 1);
                found = 1;
            }
        }
    }
}

if (found == 0) {
    printf("No manual cues found in checked range.\n");
}
```

Only auto-change cue timing when the user explicitly requests it. Reporting is safer than silently changing show behavior.

## Pattern: Convert Manual Cues to Wait

Purpose: create a commissioning utility after the designer approves conversion.

```c
// Demo_ConvertManualToWait
// Changes manual cues to Wait with a 5000 ms runtime.
// Run only on a copy of the show file or after explicit approval.

int q;
int c;
int count;

for (q = 0; q < 99; q++) {
    count = CueGetCount(q);
    if (count > 0) {
        for (c = 0; c < count; c++) {
            if (CueGetProperty(q, c, "Command") == 0) {
                CueSetProperty(q, c, "Command", 1);
                CueSetProperty(q, c, "Para0", 5000);
                printf("Converted visible QL %d Cue %d to Wait 5s\n", q + 1, c + 1);
            }
        }
    }
}
```

Use this as a draft; verify property names and behavior in LAS Language Reference before production.

## Customer Demo Advice

- Make the script support the story: "scene control, user interface, automation, standalone reliability."
- Keep dynamic effects slow and tasteful.
- Use `Emergency Full` and `Blackout` as visible safety controls.
- Show one external trigger if possible: Action Pad button, dry contact, MIDI, UDP, or scheduled trigger.
- Show export readiness: no manual cues, supported triggers/actions, and startup trigger defined.
- Distinguish online LAS capability from Butler standalone capability in customer language.

