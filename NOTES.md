# Planned changes

Status: `[ ]` idea, `[~]` in progress, `[x]` done. Add findings and file pointers under each item as we learn them.

## 1. Move ships near each other in multiplayer

**Goal:** Let players in the same multiplayer session place their ships close together (same system and location) without the current random placement.

**Starting points** (unverified, to confirm during investigation):
- `source/game/StarSystemWorldServer.hpp/.cpp`: `addClientShip`, `shipsAtLocation`, random position for new ships entering a system.
- `source/game/StarUniverseServer.cpp`: system world creation and ship flight handling (`m_shipFlights`).
- `source/game/StarNetPackets.*`: any new request packet would be added here.

**Open questions:**
- Who can trigger it (the player moving, admin only, or by agreement of both players)?
- Where does the UI live (a ship/navigation menu action, a chat command, or a Lua-bound call)?
- Should it work only inside one system, or also across systems?

- [ ] Investigate how ship position/arrival is decided today
- [ ] Choose approach (server command vs. UI action)
- [ ] Implement and test with a local client plus server

## 2. Remember window positions, with a reset-to-defaults button in settings

**Goal:** Panes (inventory, chat, options, etc.) reopen where the player last left them. The options menu gets a button that clears the saved positions.

**Findings (verified by reading code):**
- `PaneManager::displayPane` (`source/windowing/StarPaneManager.cpp:25`) centers any pane that has no anchor and has never been displayed (`hasDisplayed()`), using `pane->centerOffset()`. This is the hook for restoring a saved position.
- Dragging: `Pane::drag` (`StarPane.cpp:229`) calls `setPosition`; the drag ends in `PaneManager::sendInputEvent` (`StarPaneManager.cpp:~223`, `setDragActive(false, {})`). That is the place to save a position.
- Interface-scale changes re-position panes in `PaneManager::update` via `calculateNewInterfacePosition` (anchor-aware), so saved positions must be stored in a scale-independent way.
- `MainInterface` (`frontend/StarMainInterface.cpp`) registers its panes in a `MainInterfacePaneManager` (`windowing/StarRegisteredPaneManager.hpp`) under stable `MainInterfacePanes::*` keys (Inventory, Codex, QuestLog, Chat, ...). Script panes are shown through `displayScriptPane`, with no key beyond their config.
- `ScriptPane` already carries position across a character swap (`takeScriptPanes`/`reviveScriptPanes`), but not across sessions.
- Options menu: `source/frontend/StarOptionsMenu.cpp`, layout patched by `assets/opensb/interface/optionsmenu/optionsmenu.config.patch`. Settings go through `Root::singleton().configuration()->set(key, value)`; buttons are wired with `reader.registerCallback`.

**Proposed design:**
1. Add a `String m_positionKey` to `Pane` (empty means not saved). Set it for registered panes from their `MainInterfacePanes` key, and for script panes from their config path.
2. On drag release, if the pane has a key, store its position as an offset from the window center in `Root::configuration()` under a new `paneSavedPositions` map.
3. In `displayPane`, if a saved entry exists, use it instead of centering, clamped to the visible window.
4. Add a "Reset window positions" button to the options menu (clears `paneSavedPositions` and re-centers the currently displayed saved panes).

**Safety requirement (off-screen panes):** A saved position must never leave a pane unreachable after a resolution, window-size, or interface-scale change.
- Clamp on display, and re-clamp when the window size changes (so the drag area stays on screen).
- "Reset window positions" button in the options menu clears all saved positions.
- A pane manager pane (see below) can recover a single pane without resetting everything.

**Pane inventory** (`MainInterfacePanes` in `frontend/StarMainInterfaceTypes.hpp`). "Save?" is a proposal to review, not a decision. Draggable status is unverified; confirm in-game (`m_lockPosition` / `positionLocked` in the pane config, and anchor):

| Pane | Layer | Save? | Notes |
|---|---|---|---|
| Inventory | Window | yes | Other panes get placed adjacent to it (`bringPaneAdjacent`), so decide whether adjacency overrides a saved position |
| CraftingPlain | Window | yes | Also positioned adjacent to Inventory |
| Codex | Window | yes | |
| QuestLog | Window | yes | |
| Songbook | Window | yes | |
| Ai | Window | yes | |
| Collections | Window | yes | |
| MmUpgrade | Window | yes | |
| Tech, Cockpit | Window | check | Verify layer and draggability |
| CharacterSwap | Window | maybe | Anchored center, position unlocked |
| Options | ModalWindow | no | Modal, shown briefly |
| EscapeDialog, Confirmation, HttpTrustDialog, JoinRequest, Popup | ModalWindow/Window | no | Transient dialogs, always centered |
| ActionBar | Hud | no | Anchored CenterBottom, not draggable today |
| Chat | Hud | maybe | Hud layer, check whether it can be moved or resized |
| TeamBar, StatusPane, QuestTracker | Hud | maybe | Check whether draggable; anchored HUD elements would need anchor-offset handling |
| PlanetText, RadioMessagePopup | Hud | no | Transient or anchored display only |
| WireInterface | World | no | |
| Script panes (containers, crafting stations, mod panes) | Window | later | No stable key beyond config path; per-object panes are better left unsaved at first |

**Pane manager pane (later phase):** A small always-available pane, ideally pinned at the upper left (confirm that area is free of other HUD elements) that lists panes and allows per-pane actions. First version: list of known panes with a "bring back to center" button each. Possible later properties per pane: pin visibility, allow or disallow dragging, auto-hide, transparency.

**Decisions:**
- One global set of saved positions (not per character), so switching characters keeps the layout. Per-profile layouts are a possible later feature.
- Positions are stored as an offset from the window center. The reset button is a last resort (new monitor, config copied to another machine), so it should almost never be needed; clamping on display covers the common cases.

**Open questions:**
- Where the manager pane is opened from (key bind, options menu button, always shown).

**Phases:**
- [x] Find how panes get their initial position and which are draggable
- [~] Phase 1: persistence for `Window`-layer panes, clamping, options menu reset button
- [ ] Phase 2: verify the pane inventory in-game; decide HUD handling
- [ ] Phase 3: pane manager pane (recover individual panes)
- [ ] Phase 4: extra per-pane properties (pin, lock drag, auto-hide, transparency)

## Backlog

(Add new ideas here.)
