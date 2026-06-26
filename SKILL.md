---
name: vibe-variables
description: Recognize and implement "Vibe Variables", bracket style dynamic placeholders like {Username}, {Today's Date}, {Project Name}, {Status} found in Figma designs, UI mockups, screenshots, or any visual design handed off as static content. Use this skill any time a design contains curly brace placeholder text, any time the user says "vibe variable", "dynamic placeholder", "make this live", or "AI Ready Design", and especially for dashboards or multi screen interfaces where the same bracket variable repeats across several screens or components. Works whether the design comes through Figma MCP tools or as a plain screenshot or pasted image, since the same analysis applies either way. Always use this skill before writing any code from a design that contains curly braces, even if only one bracket is visible in the requested piece, since related variables and state hints are usually present nearby.
---

# Vibe Variables

A Vibe Variable is any text wrapped in curly braces inside a design, for example `{Username}`, `{Today's Date}`, `{Project Name}`. It is a stand in for real, live data. Treat curly brace text as an instruction to build something functional, never as literal copy to render on screen.

Some Vibe Variables carry a second bracket pair right after them, like `{Today's Date}{Hover-state}`. That second bracket is not a separate variable. It is a state label describing how that same variable should look in a particular UI state. Splitting these correctly is the most common point of failure, so handle it first.

A third pattern: visible text glued directly to a bracket with no space, like `John Doe{username}`. This is not two values. The text before the bracket is an example or preview value, and the bracket names the variable it belongs to. Read it as "this is the `username` field, currently showing `John Doe` because nothing is wired up to real data yet." Treat the bracket as the field's identity and the attached text as its current mock value, both go into the same row of the Variable Map, not two separate rows.

A fourth pattern, common in tables and dashboards with numbers: a **computed variable**, like `{Profit}` sitting in a column next to `{Price}` and `{Margin}`. Its value isn't its own input, it's the result of a formula run against other variables in the same row. The tell is usually the column name itself reading as a result rather than a raw fact (Total, Profit, Subtotal, Score, Net, Average). Never give a computed variable its own mock value to edit directly, it always derives from the variables it depends on.

A fifth pattern: a bracket where the **value sits inside the bracket itself**, like `{20%}`, with no name inside it at all. Here the column header right above the cell supplies the field's identity, and the bracket's content is the real starting value, not a placeholder to replace. Read `{20%}` under a "Margin" header as "the Margin field, currently 20%."

When a design has both raw and computed variables in the same row (something like Price and Margin feeding a Profit cell), never assume a formula on your own. There are two cases:

- **If the design or the user actually states the formula** (written out in the design, in a column header, or said directly), use exactly that.
- **If only a bare name is given** (a column just says "Profit," "Loss," "Margin," "Total," with no formula spelled out anywhere), don't guess one. A name like "Margin" alone is genuinely ambiguous, it can mean a cut of the sale price or a markup on cost, and those produce different numbers from the same inputs. In this case, search for the standard definition of that term if it's a common business or domain calculation, and also ask the user to confirm before writing any code that depends on it. Getting this wrong means building working code that produces the wrong number, which is worse than asking first.

## Step 1: Scan before you build

Never implement code from a single isolated piece. Bracket variables are almost always part of a set, and nearby elements carry the other states or related variables you need. This applies no matter how the design arrives.

**If given a Figma link or node id:**
1. Call `get_metadata` on the page or parent frame (not just the requested node) to see the full node tree.
2. Call `get_design_context` on the parent frame that contains the requested node, so every sibling with curly braces comes back in one pass.
3. If the user only gave you a node id for one row or one card, walk up to its parent frame and pull that instead.

**If given a screenshot or pasted image instead of a Figma link:**
1. Look at the whole image, not just the obvious focal element. Read every piece of visible text for curly brace patterns.
2. If the screenshot shows multiple visual treatments of what looks like the same element (different background fills, different borders, greyed out versions, highlighted versions), treat those as state variants of one variable, the same way you would for a labeled state suffix in Figma, even if the image has no labels at all. Infer the state from the visual treatment itself: a filled or highlighted version usually reads as active or selected, a muted or lower contrast version usually reads as disabled, a plain version is the normal state.
3. If text is cut off, blurry, or ambiguous in the screenshot, say what you can read and ask rather than guessing at the exact wording.
4. Proceed through the rest of this skill exactly the same way regardless of source. Figma MCP tools are one way to gather the design, not a requirement for the skill to apply.

Skipping this scan step is the single biggest cause of inconsistent output, like building one card's variable with mock data and a second card's same variable with different mock data on the same screen.

## Step 2: Split each instance into its pattern

For every curly brace instance found, work out which of the five patterns it is, then separate it into parts:

- **Plain variable**: just a bracket with a name inside, e.g. `{Username}`. No example value shown, no state. Needs a placeholder value chosen in Step 4.
- **Variable + state**: a bracket followed immediately by a second bracket, e.g. `{Today's Date}{Hover-state}`. The second bracket is never a separate variable, it is a state label describing how the first variable looks in that condition.
- **Example value + variable**: visible text glued directly to a bracket with no space, e.g. `John Doe{username}`. The text is the current mock value, the bracket is the field's name. This means the value already shown is the intended placeholder, don't replace it with a different generic one, use exactly what's there.
- **Computed variable**: a bracket whose value is a formula result, not an input, e.g. `{Profit}` next to `{Price}` and `{Margin}`. Identify what it depends on before writing any code for it.
- **Value-in-bracket**: a bracket containing the actual starting value with no field name inside, e.g. `{20%}`, where the column header or nearby label supplies the name. Use the value shown, it's real, not a placeholder to swap out.

State labels are not a fixed list. Any label that reads as describing a visual condition, however it's worded, gets handled as a state, as a different look for the same underlying value, not a new variable. Two things to watch for specifically:

- **The word "state" is not required.** `{Active}`, `{Hover}`, `{Selected}` with no "-state" suffix at all are still state labels, read the same as `{Active-state}`, `{Hover-state}`, `{Selected-state}`. Don't treat the missing suffix as a sign it must be a separate variable.
- **Different words can mean the same condition.** `{Active}`, `{Active-state}`, `{Selected}`, `{Selected-state}`, and `{Select-state}` are all the same thing, "the one currently chosen," just phrased differently. Match them by what they mean, not by exact spelling.

When the underlying values are date based and one of them happens to be today, default the active or selected state to that entry rather than picking the first item arbitrarily, this matches how a real date list usually behaves.

If you see the same variable name appear multiple times with different state labels (e.g. `{Today's Date}{Hover-state}` and `{Tomorrow's Date}{Normal-state}` stacked together), that is one interactive component shown in two states, not two separate static elements. Build ONE element that switches between those visual treatments (via `:hover`, a `.is-active` class, `:focus`, etc), not two permanent divs frozen in their respective states.

## Step 3: Build the Variable Map

Before writing code, lay out a short table and share it in chat so it can be sanity checked:

| Variable | Type | States found | Real value / source | Notes |
|---|---|---|---|---|
| Today's Date | date | Hover, Normal | `new Date()`, formatted | shared across 3 cards |
| Username | string | Normal | mock value, swappable | repeats on every screen |
| Project Name | string | Normal, Empty | mock value or input field | empty state needs placeholder copy |
| Status | enum | Active, Pending, Done | small mock dataset | drives a colored badge |
| Margin | percent | Normal | 20%, value shown in bracket | value-in-bracket pattern, name comes from column header |
| Profit | number | Normal | computed, depends on Price and Margin | confirm formula before building |

For a table or any structure where a variable repeats down multiple rows (a product list, a user list), the Variable Map only needs one row per variable, not one per repetition. Note that it repeats and what distinguishes one row of data from the next, the per row values themselves don't need to be enumerated in the map.

This table is the deliverable that lets the user catch a wrong mapping before any code gets written, which is the whole point of doing this systematically instead of re-explaining each variable by hand.

Before moving on, count every bracket instance found during the scan in Step 1 and check that each one is accounted for in this table, either as its own row or as a stated repetition of a row already listed. A variable that was visible in the scan but never made it into the map will also never make it into the code, this is an easy, silent way to drop something. If the scan turned up more instances than the table currently has rows for, go back and add the missing ones before continuing.

## Step 4: Implement with one source of truth

- Match whatever stack the project is already using, don't introduce a different language or framework than what's already there. Check for this before assuming: look at earlier files or code already built in this conversation or project, an uploaded codebase, or a framework the user has stated, and build within whatever that already is. If none of that exists, ask the user what stack to use rather than silently picking one.
- Create a single source of truth holding every variable's current value, a data object, a config, a state store, a model, whatever fits the stack in use, and bind the interface to it. Do not hardcode the same variable's value separately in multiple places. The mechanism changes by stack, but the rule is the same regardless: one value lives in one place, and everything displaying it reads from that place rather than getting its own separate copy.
- Look at what the variable actually represents and analyze it on its own terms rather than forcing it into a fixed category list:
  - If it clearly represents a date or time, compute it live with `Date()` and format it, never hardcode a date string.
  - If it represents an identity or label field (a name, a title, a project, anything that's just a string standing in for real content), check whether an example value was already glued to the bracket in the design (the `John Doe{username}` pattern). If so, use that exact value, it was chosen on purpose. Only invent a generic placeholder, something neutral and obviously a placeholder like "Item Name" or "Sample User", when no example value was shown at all, a bare `{Username}` with nothing attached.
  - If it represents a status, tier, role, or anything with a small fixed set of possible values, build a small mock array covering that set and implement the full range of visual states it can take, not just whichever one happened to be visible.
  - State labels found in Step 2 (hover, active, disabled, or any other condition, however it was named or however it was inferred from a screenshot): implement using whatever the stack's real mechanism is for visual state, never as a separate static copy of the element.
  - **Computed variables**: store its formula as a function, never as a hardcoded result, and re-run that function whenever any variable it depends on changes. Label the result plainly using whatever annotation mechanism the stack already supports, so someone can see the actual expression at a glance without digging through logic to find it. Never let a computed cell be directly edited, since editing it would break the thing it represents.
  - **Tables and repeating rows**: model each row as one object holding that row's own variables (raw and computed), and keep an array or list of those row objects as the single source of truth. A change to one row's inputs must only ever affect that row's own computed values, never another row's. Render rows from that array using the stack's own repetition mechanism rather than writing out repeated elements by hand, so adding a row later means adding one object, not duplicating a block of code. Every row gets its real computed value, not just the one styled as active or selected.
- Basic input validation matters once a variable is editable, even in a mock or demo build: a blank input, a negative number where one doesn't make sense, or letters typed into a number field shouldn't silently compute to zero or a wrong number without any sign anything went wrong. At minimum, guard against `NaN` and visibly flag it rather than rendering a confident looking wrong answer.
- For dashboards or multi screen handoffs: if the same variable name shows up on multiple screens or components, it must read from the same single source, not be re-implemented separately per screen.
- Editable inputs (a text box standing in for a backend field) are useful for proving a formula or a binding works, but say plainly that this is a demo convenience, not the production shape. A real version usually renders raw variables as read only values updated by fetched data, with no input box at all, unless the actual product is meant to let a user type that value in. Don't leave this ambiguous in the handoff.

## Step 5: Confirm scope before going wide

If the requested node is part of a larger file with many more bracket variables than what's visible in the current node (common in full dashboard handoffs), say so and ask whether to implement just the requested piece or scan and implement the whole file in one pass. Building the whole thing unprompted can waste effort if the user only wanted one screen; building only one piece without flagging the rest can leave inconsistent variables across the file. A one line heads up is enough, then proceed.

## Output

- Lead with the Variable Map table.
- Then deliver working code in whatever the project's existing stack is. If no stack was found per Step 4, this should already have been asked rather than assumed.
- Before calling it done, check the actual rendered result for leftover literal text and for mismatched rows, especially in a repeating set. If one row shows a real computed value and a sibling row still shows the bracket's label as plain text, that row was never wired up. If every row shows a value but the active or selected styling doesn't land on the row that actually matches it (the highlighted row isn't actually today, the literal label text and the real value swapped positions), the array and the rendering are out of sync. Either way, fix it before delivering, this is the single most common way a build looks finished but isn't.
- Cross check the finished build against the Variable Map one more time: every row in the map should correspond to something real and working in the output. A variable that made it into the map but never got implemented is just as much a miss as one that got skipped during the scan.
- Briefly note anything you inferred (a mock value, a date format, a state you implemented that wasn't explicitly shown) so it's easy to spot and correct.
