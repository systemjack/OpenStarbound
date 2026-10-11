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

**Starting points** (unverified):
- `source/windowing/StarPaneManager.*` and `StarPane.*`: where panes are positioned on display and dragged.
- `source/frontend/`: individual pane classes and the options menu.
- `assets/opensb/interface/optionsmenu/` and `assets/opensb/interface/windowconfig/`: UI layout JSON for the settings panel.
- User settings are persisted via the client configuration; saved positions should go there.

**Open questions:**
- Key each pane by a stable ID (config name or class); not every pane has one today.
- Handle resolution or window-size changes (clamp to visible area, store relative to screen size).
- Panes whose layout depends on state (e.g. container panes opened per object) may be better left unsaved.

- [ ] Find how panes get their initial position and which are draggable
- [ ] Add persistence of positions to the configuration
- [ ] Add reset button and wire it to clear saved positions
- [ ] Clamp restored positions to the visible area

## Backlog

(Add new ideas here.)
