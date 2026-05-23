# Hangar Markdown Export Investigation

## Goal
Provide a simple export (or in-game copyable report) of current **Meks in hangar** including:
- loadout (weapons/equipment, including jump jets)
- armor value
- heat sinks
- quirks

## Existing code surfaces

1. `HangarReport` + `HangarReportDialog` already expose a report entry point, but the current report output is a `JTree` of categorized units, not detailed fit/loadout data.
2. `UnitTableModel` already has a Quirks column through `unit.getQuirksListHTML()` and is used for hangar unit listings.
3. Unit-level data are already available from `Unit` / `Entity` objects in the campaign hangar model.

## Easiest implementation path

### Recommendation: Add a **copy-to-clipboard Markdown export action** in Hangar/Unit UI

This is the lowest-risk path because it avoids introducing file I/O UI flows and can reuse existing selected/filtered unit data.

Suggested approach:

1. Add a context-menu or toolbar action in the hangar/unit list view:
   - `Copy Meks as Markdown`
2. Build markdown text from currently displayed units (respecting filters).
3. Put markdown on system clipboard (and optionally show a success toast/dialog).

### Why this is easiest

- No new persistence layer or save-dialog UX.
- Reuses existing Swing/report patterns.
- Immediately usable for Discord/forum/wiki pasting.
- Keeps behavior deterministic with current hangar filter state.

## Proposed formatter contract

Generate one markdown section per Mek:

```markdown
## <Unit Name>
- Chassis/Model: ...
- Tonnage: ...
- Armor: <current>/<max>
- Heat Sinks: <type> <count>
- Jump Jets: <count> (<type if available>)
- Quirks: quirk1, quirk2, ...

### Loadout
- RA: ...
- LA: ...
- RT: ...
...
```

## Suggested extraction rules

- **Scope**: include only `Entity` instances that are Meks.
- **Loadout**: iterate mounted equipment by location and summarize weapon/equipment names.
- **Jump Jets**: count from misc equipment with jump-jet flags.
- **Armor value**: sum per-location current and max armor.
- **Heat sinks**: include sink type and installed count.
- **Quirks**: reuse the existing unit/entity quirk list helpers where possible.

## Minimal code organization

1. Add a utility class (example):
   - `mekhq.utilities.HangarMarkdownExporter`
2. Add one public method:
   - `String exportMeksToMarkdown(List<Unit> units)`
3. Add one UI integration point to trigger export and clipboard copy.

## Optional follow-up

If desired later, add `Export to .md` file action that reuses the same formatter output string.
