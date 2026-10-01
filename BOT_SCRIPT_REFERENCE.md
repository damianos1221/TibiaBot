# Bot Script and Configuration Reference

This document describes the interfaces currently implemented by the bot:

- waypoint (`.txt`/`.wpt`) line types;
- every built-in `action` command;
- supported user-facing `call` functions;
- every intended persistent action;
- the complete per-character script configuration (`scripts/*.json`);
- related `GLOBALCFG.json` settings.

The reference is based on the current code in `core/Walker.py`,
`core/Targeting.py`, `core/WaypointActions.py`, `core/lib.py`,
`core/Presistents.py`, `core/Hotkeys.py`, `core/Healer.py`, and
`core/BattleList.py`.

## 1. File locations and syntax

Script files are loaded relative to the `scripts` directory.

- The active character configuration is selected by `CONFIGFILE` in
  `scripts/GLOBALCFG.json`.
- The active waypoint file is selected by `WAYPOINTSFILE` in
  `scripts/GLOBALCFG.json`.
- A configuration file uses JSON syntax.
- A waypoint file contains one command per line.
- Blank waypoint lines are ignored.

Waypoint `call` arguments use Python syntax, not JSON syntax:

```text
call wait(2.5)
call npc_say(['trade', 'yes'])
call check_supplies(Potions=True, Runes=False, Ammo=True)
call conditional_jump_position([(100, 200, 7), (101, 200, 7)], 'hunt', 'leave')
```

Use `True`, `False`, `None`, quoted strings, lists (`[]`), and tuples (`()`) in
`call` expressions. The expression is evaluated with Python `eval()`. Waypoint
files must therefore be treated as trusted code.

While a waypoint `action` or `call` is running, persistent actions are paused.

## 2. Waypoint line types

Coordinates always use `(x, y, z)` in Tibia world coordinates.

| Line | Purpose |
|---|---|
| `node (x, y, z)` | Normal walking waypoint. It is considered reached inside the `NodeIgnoreRadio` radius on the same floor. |
| `stand (x, y, z)` | Walk to the exact position. |
| `lure (x, y, z)` | Walk using the lure pack logic, then optionally enable targeting according to `script_options`. |
| `stand&attack (x, y, z)` | Walk to the exact position, wait `delay_before_checkmonsterscount`, recount monsters, and enable targeting only when the configured threshold is met. |
| `rope (x, y, z)` | Use the configured `tools.rope` hotkey on the player's current tile and verify a floor change. The stored coordinates are required by the parser but are not used as the click position. |
| `shovel (x, y, z)` | Use the configured `tools.shovel` hotkey on the specified tile and verify a floor change. |
| `use (x, y, z)` | Right-click the specified tile. |
| `ladder (x, y, z)` | Right-click the player's current tile and use the ladder fallback menu if necessary. The stored coordinates are not used. |
| `stairs (x, y, z)` | Step or click onto an adjacent stair tile. The stair must be at most one tile away. |
| `door (x, y, z)` | Try left-click/right-click sequences on the specified door tile until the character moves. Use the singular keyword `door`. |
| `label name` | Named control-flow marker. Label matching for jumps is case-insensitive. |
| `action name` | Run one predefined no-argument action from section 3. |
| `call expression` | Evaluate and, when applicable, await a function call from section 4. |


## 3. Waypoint `action` commands

Syntax:

```text
action action_name
```

Unknown names print an error and the walker proceeds to the next waypoint.

### General, depot, and control actions

| Action | Behavior |
|---|---|
| `bank` | Says `deposit all`, then `yes` to the current NPC. |
| `wait` | Waits 1 second. |
| `wait_ten_min` | Waits 10 minutes. |
| `communication_task` | Says `communication`, then `yes`. |
| `deposit` | Reaches the nearest depot tile and stows the configured loot backpack. Requires `containers.main_bp` and `containers.loot_bp`. |
| `opendepot` | Reaches the nearest depot and opens the depot container. |
| `openlocker` | Reaches the nearest depot and opens the locker. |
| `buy_potions` | Runs `npc_refill(Potions=True)` using `hunt_config`. |
| `buy_runes` | Runs `npc_refill(Runes=True)` using `hunt_config`. |
| `buy_ammo` | Runs `npc_refill(Ammo=True)` using `hunt_config`. |
| `buy_ticket` | Says `ticket`, then `yes`. |
| `logout` | Waits for the battle sign to disappear, then sends Ctrl+L and closes dialogs. |
| `forceexit` | Terminates every process named `client.exe`; use with care. |
| `setwalkeron` | Enables the walker. |
| `setwalkeroff` | Disables the walker. |
| `setlootingon` | Enables looting. |
| `setlootingoff` | Disables looting. |
| `settargetingon` | Enables targeting. |
| `settargetingoff` | Disables targeting. |

### Levitate actions

These turn the character and say `exani hur up` or `exani hur down`.

```text
action levitate_north_up
action levitate_north_down
action levitate_south_up
action levitate_south_down
action levitate_east_up
action levitate_east_down
action levitate_west_up
action levitate_west_down
```

### Travel actions

Most travel actions say the destination and then `yes`. The underlying
`npc_say` function first sends `hi`; the table lists the words sent after that:

| Action | NPC words sent |
|---|---|
| `travel_edron` | `edron`, `yes` |
| `travel_thais` | `thais`, `yes` |
| `travel_oramond` | `oramond`, `yes` |
| `travel_krailos` | `krailos`, `yes` |
| `travel_port_hope` | `port hope`, `yes` |
| `travel_venore` | `venore`, `yes` |
| `travel_kazordoon` | `kazordoon`, `yes` |
| `travel_carlin` | `carlin`, `yes` |
| `travel_liberty_bay` | `liberty bay`, `yes` |
| `travel_ankrahmun` | `ankrahmun`, `yes` |
| `travel_darashia` | `darashia`, `yes` |
| `travel_svargrond` | `svargrond`, `yes` |
| `travel_gray_island` | `gray island`, `yes` |
| `travel_goroma` | `goroma`, `yes` |
| `travel_farmine` | `farmine`, `yes` |
| `travel_yalahar` | `yalahar`, `yes` |
| `travel_abdendriel` | `ab'dendriel`, `yes` |
| `travel_cormaya` | `cormaya`, `yes` |
| `travel_mistrock` | `mistrock`, `yes` |
| `travel_roshamuul` | `roshamuul`, `yes` |
| `travel_passage` | `passage`, `yes` |
| `travel_east` | `east`, `yes` |
| `travel_west` | `west`, `yes` |
| `travel_center` | `centre`, `yes` |
| `travel_banuta` | `banuta`, `yes` |
| `travel_fenrock` | `fenrock`, `yes` |
| `travel_alchemist` | `pass`, `alchemist` |
| `travel_cemetery` | `pass`, `cemetery` |
| `travel_magician` | `pass`, `magician` |
| `travel_arena` | `pass`, `arena` |
| `travel_foreign` | `pass`, `foreigner` |

## 4. Waypoint `call` functions

The safest supported interface is the public async functions defined in
`core/lib.py`. For a normal `call expression`, a truthy return value advances to
the next waypoint and a falsy return value (including `None`) moves to the
previous waypoint. Functions that jump to a label or modify the current
waypoint also change walker control flow before this final advancement. Use the
documented conditional helpers for routing; do not build new routes around
`nextlabel()` or a direct `jumplabel()` unless that extra movement is intended.

### Timing, keys, and movement

| Call and defaults | Behavior |
|---|---|
| `wait(seconds)` | Wait for the given number of seconds plus small random/ping delay. |
| `press(key)` | Press a physical key and wait briefly. |
| `use_hotkey(hotkeyname)` | Press a configured hotkey by logical name, respecting cooldown reservation. |
| `turn(direction)` | Turn without walking. Valid values: `'n'`, `'s'`, `'e'`, `'w'`. |
| `levitate(direction, spell, delay_move=0.3, delay_spell=1)` | Turn toward `north/south/east/west`, then use `up/down`. |
| `use_sqm((dx, dy))` | Right-click a tile relative to the player. |
| `use_hotkey_at_sqm(hotkeyname, (dx, dy))` | Select a hotkey and left-click a relative tile. |
| `use_hotkey_at_pos(hotkeyname, (x, y, z))` | Select a hotkey and left-click a world position. It must be on the current floor and inside the game window range. |
| `Rclick((screen_x, screen_y))` | Right-click absolute screen coordinates. Advanced and resolution-dependent. |
| `Lclick((screen_x, screen_y))` | Left-click absolute screen coordinates. Advanced and resolution-dependent. |
| `open_doors(dx, dy)` | Try click sequences on a relative door tile until player position changes. |
| `waitfortibianight()` | Wait until the local clock minute is `00..15` or `45..59`. This is a clock approximation, not an in-client day/night detector. |

Examples:

```text
call wait(2)
call turn('n')
call use_hotkey('haste')
call use_hotkey_at_sqm('machete', (0, -1))
call use_hotkey_at_pos('shovel', (32683, 32789, 8))
```

### NPC, trade, and depot calls

| Call and defaults | Behavior |
|---|---|
| `reach_npc()` | Walk to the closest visible NPC, stopping adjacent to it. |
| `npc_say(words)` | Press Enter, say `hi`, wait, then say every word from a list. |
| `sell_items(items)` | Open NPC trade and sell `(item_name, amount)` pairs. `amount` may be an integer or `'all'`. |
| `buy_item(name, amount)` | Buy an item from an already opened NPC trade window. |
| `npc_refill(Potions=False, Runes=False, Ammo=False)` | Buy configured resources up to their `take_*` amounts. |
| `reach_depot()` | Walk to the nearest mapped depot tile on the current floor. |
| `open_depot()` | Open a depot and then its container. The character must already be on a mapped depot tile. |
| `open_locker(back_label=None)` | Open a locker. `back_label` currently has no effect. |
| `stash_lootbp()` | Stow the configured loot backpack contents into the depot. |
| `withdraw_item_from_stash(hotkeyname, itemname, amount, breakifhave_amount=False)` | Reach the depot, open stash, and withdraw enough items to reach `amount`. |

Examples:

```text
call reach_npc()
call npc_say(['trade'])
call sell_items([('strong mana potion', 'all'), ('flask of warrior sweat', 10)])
call npc_refill(Potions=True, Runes=True)
call withdraw_item_from_stash('smp', 'strong mana potion', 500, True)
```

### Supply and session checks

| Call and defaults | Behavior |
|---|---|
| `check_potions(label_jump=None, label_skip=None)` | Check configured potion `*_leave` limits. Session timeout always jumps to `leave`. Low potions jump to `label_jump` when supplied. No shortage jumps to `label_skip` when supplied. |
| `check_supplies(Potions=False, Runes=False, Ammo=False, label_jump=None)` | Check capacity, session timeout, and selected `*_leave` resources. A shortage jumps to `leave`; success jumps to `label_jump` or `hunt`. |
| `check_gohunt(Potions=False, Runes=False, Ammo=False, label_jump=None)` | Check capacity/session state and selected `take_*` amounts before hunting. A shortage jumps to `label_jump` or `mana_shop`; success proceeds. |
| `check_time(label_jump=None, label_skip=None)` | Test `session_time`/`hours_leave` or the session scheduler. On timeout it jumps to `leave` when the scheduler is enabled, otherwise to `label_jump`. |
| `set_time_leave_after(hours)` | Set an earlier runtime leave deadline. Mainly used by the autoresponder; it may also be called from a waypoint. |

Resource flags are case-sensitive Python keyword arguments:

```text
call check_supplies(Potions=True, Runes=True, Ammo=False)
call check_gohunt(Potions=True, Ammo=True, label_jump='mana_shop')
call check_time('leave', 'hunt')
```

Current timeout details:

- `check_potions` jumps to `label_jump` when missing any potions, else jump to nextWPT or `label_skip` if exist.
- `check_supplies` jumps to `leave` on low capacity or checktime is true or missing any SUPPLIES, else jump to `hunt` or `label_jump` if exist.
- `check_gohunt` jumps to `mana_shop` on low capacity or checktime is true or missing any SUPPLIES, else jump to nextWPT or `label_jump` if exist.

### Conditional jumps and labels

| Call and defaults | Behavior |
|---|---|
| `jumplabel(labelname)` | Jump to the named label. Prefer the conditional helpers below for normal routing. |
| `nextlabel()` | Move the current waypoint index forward once. Advanced; the dispatcher can also advance after the call. |
| `conditional_jump_floor(z, label_jump=None, label_skip=None)` | Jump to `label_jump` when the current floor equals `z`, otherwise optionally jump to `label_skip`. |
| `conditional_jump_item_count_below(hotkeyname, amount, label_jump=None, label_skip=None)` | Read the item count displayed on a hotkey and branch on `count < amount`. |
| `conditional_jump_position(coords, label_jump=None, label_skip=None)` | Branch when the exact `(x, y, z)` player position occurs in `coords`. |
| `conditional_jump_position_range(start, end, label_jump=None, label_skip=None)` | Branch when player `(x, y)` lies inside the inclusive rectangle. Floor is not checked. |

Examples:

```text
call conditional_jump_floor(7, 'hunt', 'stairs_back')
call conditional_jump_item_count_below('gmp', 300, 'mana_shop', 'hunt')
call conditional_jump_position([(33200, 31893, 7)], 'inside', 'outside')
call conditional_jump_position_range((33190, 31880), (33210, 31900), 'inside', 'outside')
```

### Items, imbuements, and lure helpers

| Call and defaults | Behavior |
|---|---|
| `drop_items(names=None, backpacknames=None)` | When no monsters are visible, temporarily stop the walker and drop matching items from the named open backpacks. |
| `use_item_frombp_on_pos(itemname=None, backpackname=None, pos=None)` | Find an item in an open backpack and use it on a world position. |
| `check_imbu(label_jump=None, label_skip=None)` | Verify configured `script_options.imbuements`; jump on missing/present state. |
| `use_imbu_scroll(hotkey=None, slot=None)` | Use a configured hotkey on an equipment slot. |

Supported equipment slot names are `amulet`, `weapon`, `ring`, `helmet`,
`armor`, `legs`, `boots`, `backpack`, `shield`, and `extra`.

Control-flow warning: `use_imbu_scroll(...)` has no explicit return value. Used directly
as waypoint calls, those outcomes make the dispatcher move to the previous
waypoint.

### State-control calls

These are also available as waypoint calls, although the equivalent `action`
form is shorter:

```text
call logout()
call forceexit()
call setwalkeron()
call setwalkeroff()
call setlootingon()
call setlootingoff()
call settargetingon()
call settargetingoff()
```

## 5. Persistent actions

Persistent actions are configured as:

```json
{
  "action": "action_name",
  "interval": 1.0,
  "args": {
    "argument": "value"
  }
}
```

- `interval` is in seconds and defaults to `1`.
- `args` may be an object (keyword arguments), an array (positional
  arguments), or omitted for a no-argument function.
- Execution includes approximately 80-120 ms of scheduling jitter after the
  requested interval.
- Persistent actions are paused while a waypoint `action` or `call` runs.
- Most actions require the Tibia window to be active and the player online.
- A misspelled action is skipped with a warning.


### Condition and maintenance actions

| Persistent action | Arguments | Trigger |
|---|---|---|
| `manashield` | `hotkey` | Cast when the player is not mana-shielded. |
| `in_pzzone` | `hotkey` | Cast only while the player is in a protection zone. |
| `anti_risen` | `hotkey` | Cast while the `risen` condition is active. |
| `anti_electrified` | `hotkey` | Cast while electrified. |
| `anti_freezing` | `hotkey` | Cast while freezing. |
| `anti_drunk` | `hotkey` | Cast while drunk. |
| `anti_paralyze` | `hotkey` | Cast while paralyzed. |
| `anti_poison` | `hotkey` | Cast while poisoned. |
| `haste` | `hotkey` | Cast when not hasted and not in a protection zone. |
| `recover_full_mana` | `hotkey` | Use the hotkey while mana is below 99%; sleeps 1 second after use. |
| `eat_food` | `hotkey` | Use food while the `hungry` condition is active. |

Example:

```json
{"action":"anti_paralyze","interval":1,"args":{"hotkey":"exura"}}
```


#### `cast_spell_if_monsters`

```json
{
  "action": "cast_spell_if_monsters",
  "interval": 0.2,
  "args": {
    "min_mp": 30,
    "spell_hotkey": "amp res",
    "monster_count": 2,
    "below_hp_perc": 100,
    "monsters": ["Pirat Scoundrel", "Pirat Bombardier"],
    "min_dist": 1,
    "max_dist": 7,
    "use_with_targeting_off": false
  }
}
```

Arguments and defaults:

| Argument | Default | Meaning |
|---|---:|---|
| `min_mp` | required | Current mana percentage must be strictly greater than this value. |
| `spell_hotkey` | required | Logical name from `hotkeys`. |
| `monster_count` | `3` | Required number of matching monsters. |
| `monsters_count` | `3` | Legacy alias used when `monster_count` remains `3`. Prefer `monster_count`. |
| `below_hp_perc` | `100` | Count only monsters at or below this HP percentage. |
| `monsters` | `'all'` | `'all'` or a list of exact monster names. |
| `min_dist` | `2` | Minimum inclusive Chebyshev distance. |
| `max_dist` | `5` | Maximum inclusive Chebyshev distance. |
| `use_with_targeting_off` | `false` | Permit casting while targeting is disabled. |

The name/HP filter and the distance check are separate in the current
implementation: at least `monster_count` matching monsters must exist, and at
least `monster_count` total visible monsters must be inside the distance range.
The monsters satisfying those two checks do not have to be the same objects.


### Equipment and inventory actions

#### `equip_item`

```json
{
  "action": "equip_item",
  "interval": 0.5,
  "args": {
    "hotkeyname": "might ring",
    "slot": "ring",
    "unequip": true,
    "hppc_to_equip": 40,
    "hppc_to_unequip": 80,
    "selected_monsters": "None",
    "dist": 7,
    "amount": 1
  }
}
```

Defaults: `hotkeyname='amulet'`, `slot='amulet'`, `unequip=true`,
`hppc_to_equip=0`, `hppc_to_unequip=0`, `selected_monsters='None'`,
`dist=7`, `amount=1`.

- `selected_monsters: "None"` uses HP-only mode.
- `selected_monsters: "all"` uses all visible monsters.
- A list uses exact monster names.
- With monsters selected, equip when at least `amount` are within `dist` and
  HP permits it; unequip when the count drops below `amount` and HP permits it.
- The slot image must be recognized as an item whose name contains `empty`.

#### Other inventory actions

| Persistent action | Arguments and defaults | Behavior |
|---|---|---|
| `equip_if_empty_slot` | `hotkeyname='ring'`, `slot='ring'` | Press the hotkey when the recognized slot image contains `empty`. |
| `drop_items` | `names`, `backpacknames` | Drop matching items from named open backpacks when no monsters are visible. |
| `refill_quiver` | `hotkey=None`, `count_start_equip=100`, `count_stop_equip=600` | Refill the shield/ammo slot when its count falls below the start threshold. The stop threshold must be at most 600. |
| `anti_trap` | `hotkey=None`, `movetries=15` | Use a tool on the last movement direction after enough failed moves, unless an attacked monster is adjacent. |

### Legacy login action

```json
{
  "action": "login",
  "interval": 15,
  "args": {
    "email": "account@example.com",
    "password": "secret",
    "characterindex": 1
  }
}
```

This older login action runs only while the Tibia window is active and the
player is offline. It is automatically skipped when the session scheduler is
enabled. Prefer `SESSIONS.json` for character switching. Storing credentials in
plain JSON is inherently sensitive.

## 6. Character configuration (`scripts/*.json`)

### Top-level structure

```json
{
  "script_options": {},
  "hotkeys": [],
  "healing": [],
  "target_monsters": [],
  "target_spells": [],
  "tools": {},
  "containers": {},
  "hunt_config": {},
  "persistent_actions": [],
  "monsters_to_ignore": [],
  "special_areas": []
}
```

`hotkeys`, `target_monsters`, `target_spells`, and `tools` should always be
present because parts of startup access them directly. Empty arrays/objects are
valid.

The legacy top-level key `label_actions` occurs in old files but is not read by
the current code.

### `script_options`

| Key | Type/default | Meaning |
|---|---|---|
| `session_time` | number/string/list, `0` | Maximum bot session duration in hours from process start. A one-element list is accepted for compatibility. |
| `hours_leave` | number/string/list, `0` | Legacy fallback used only when `session_time` is absent. |
| `imbuements` | array, `[]` | Items and imbuements checked by `check_imbu`. See below. |
| `block_walk_on_raid_pos` | boolean, `true` | Prevent walker paths from crossing the detected raid/player danger position. |
| `target_monsters_priority` | string, `"distance"` | Target selection mode: `distance`, `listorder`, `lowhp`, or `highhp`. |
| `wptlure_monsters_to_enable_targeting` | integer, `1` | Monster count required to start targeting at a lure waypoint, or when trapped near it. |
| `wptS&A_monsters_to_enable_targeting` | integer, `1` | Monster count required at `stand&attack`. Key spelling and capitalization are exact. |
| `wptlure_monsters_force_targeting` | integer, `8` | Monster count that forces targeting before reaching the lure waypoint. |
| `wptlure_monsters_in_box_togo_next_wpt` | integer, `0` | When positive, cancel targeting and continue if at most this many lure monsters remain, all are within one tile, and all have more than 60% HP. |
| `delay_before_checkmonsterscount` | milliseconds, recommended `500` | Delay before the confirming monster recount at `lure` and `stand&attack`. Startup fallback is currently 400 ms; runtime config reload fallback is 500 ms, so set it explicitly. |
| `use_tool_on_bodies` | `[boolean, hotkey]`, `[false, "none"]` | Walk to reachable corpses and use the named tool hotkey on them. |


Imbuement entry:

```json
"imbuements": [
  {
    "item": "falcon plate",
    "imbuements": ["Powerful Vampirism", "Powerful Void"]
  }
]
```

Names must match item and imbuement image-pattern names.

### `hotkeys`

```json
{
  "name": "exori gran",
  "hotkey": "F12",
  "group": "attack",
  "cooldown": 6,
  "group_cooldown": 2,
  "secondary_groups": ["support"],
  "secondary_group_cooldown": 2,
  "secondary_group_cooldowns": {
    "support": 2
  }
}
```

| Key | Type/default | Meaning |
|---|---|---|
| `name` | string, `"any"` | Case-insensitive logical name referenced by other sections. Names should be unique. |
| `hotkey` | string, `"F1"` | Physical key. Normal values are `F1..F12`, `CF1..CF12` (Ctrl), and `SF1..SF12` (Shift). |
| `group` | string, `"heal"` | Primary cooldown group. Built-in defaults: `heal/healing=1`, `item=1`, `attack=2`, `support=2` seconds. Custom group names are allowed. |
| `cooldown` | seconds, `1` | Individual cooldown. The manager adds 100 ms internally. |
| `group_cooldown` / `group_cd` | seconds | Override the primary group cooldown. `group_cooldown` takes precedence. |
| `secondary_groups` / `secondary_group` | string or array | Additional cooldown groups affected by this hotkey. |
| `secondary_group_cooldown` / `secondary_group_cd` | seconds, `0` | Default cooldown for secondary groups. |
| `secondary_group_cooldowns` | object, `{}` | Per-secondary-group cooldown overrides. |

The bot detects duplicate physical hotkeys and logs them. List order matters
when multiple systems are ready at the same time.

### `healing`

```json
{
  "name": "emergency health",
  "type": "hp",
  "below_percent": 55,
  "hotkey": "uhp",
  "min_mana_percent": 0,
  "cooldown": 1
}
```

| Key | Type/default | Meaning |
|---|---|---|
| `name` | string | Descriptive only; the healer does not use it. |
| `type` | `"hp"` or `"mp"` | Required. Other values are ignored. |
| `below_percent` | integer, `50` | HP entries trigger at `HP <= value`; MP entries at `MP < value`. |
| `hotkey` | string, `"none"` | Logical hotkey name, or a raw key as fallback. |
| `min_mana_percent` | integer | HP-only minimum mana. If omitted: 0 for item-group heals, 1 for spells. |
| `min_health_percent` | integer, `1` | MP-only minimum health. |
| `cooldown` | seconds | Optional per-healing-entry cooldown override. |

Entries are evaluated in array order. Only the first matching entry is used per
loop. Mana recovery is suppressed while an eligible HP heal is ready.

### `target_monsters`

```json
{
  "name": "Any",
  "action": "follow",
  "range": 3,
  "only_trapped": false,
  "need_attack_me": "false"
}
```

| Key | Type/default | Meaning |
|---|---|---|
| `name` | string, `"any"` | Exact monster name. Use exactly `"Any"` as the fallback wildcard. |
| `action` | string, `"follow"` | `follow`, `distance`, or `stand`. |
| `range` | integer, `3` | Desired distance for `distance`; ignored for other actions. |
| `only_trapped` | boolean/string, `false` | Entry is eligible only while `Walker.trapped` is true. |
| `need_attack_me` | string, `"false"` | When exactly `"true"` and another player is visible, accept only a monster marked as attacking the player. Use a string because the current comparison is string-based. |

Action behavior:

- `follow`: move adjacent to the selected attacked monster.
- `distance`: choose a reachable stance around the requested `range`.
- `stand`: attack without follow/distance repositioning.

Priority modes:

- `distance`: use battle-list order, selecting the last matching config entry
  for a monster; despite its name, it does not sort candidates by computed
  distance.
- `listorder`: iterate `target_monsters` first, so configuration order is the
  priority; the first matching config entry is used.
- `lowhp`: choose the eligible battle-list target with the lowest HP.
- `highhp`: choose the eligible battle-list target with the highest HP.

Monster names and `monsters_to_ignore` matching are case-sensitive.

### `target_spells`

```json
{
  "type": "Self small explosion",
  "hotkey": "exori gran",
  "min_mana_percent": 10,
  "min_monster_hppc": 0,
  "min_monster_count": 2,
  "monsters": "all",
  "full_harmony": false,
  "pvp_protect": true
}
```

| Key | Type/default | Meaning |
|---|---|---|
| `type` | string, `"Direct"` | Exact spell geometry/type name from the list below. Unknown types are ignored. |
| `hotkey` | string, `"none"` | Logical hotkey name. |
| `min_mana_percent` | integer, `10` | Minimum current mana percentage. Equality is accepted. |
| `min_monster_hppc` | integer, `0` | Current target must have at least this HP percentage. |
| `min_monster_count` | integer, `1` | Required matching monsters/hits; minimum effective value is 1. |
| `monsters` | `"all"` or array, `"all"` | Exact names eligible for this spell. |
| `full_harmony` | boolean/string, `false` | Require the harmony state before casting. |
| `pvp_protect` | boolean/string, `false` | Do not cast when players are visible. |

Supported exact `type` values:

| Type | Geometry/requirement |
|---|---|
| `Direct` | Requires a selected target and a shootable line. |
| `Missile range 4` | Requires a selected target and a shootable target within range 4. |
| `Missile range 1` | Requires a selected target and a shootable target within range 1. |
| `Self big explosion` | Large self-centered area. |
| `Self medium explosion` | Medium self-centered area. |
| `Self small explosion` | Eight adjacent tiles. |
| `Exori min wave` | Turns toward a direction that hits the required count. |
| `Gran Mas Pug` | Directional Gran Mas Pug pattern. |
| `Mas Nia` | Directional Mas Nia pattern. |
| `Mas Pug` | Directional Mas Pug pattern. |
| `Mas San` | Paladin Mas San area. |
| `RuneBestHit` | Selects and clicks the best rune impact tile. |
| `Gran Mas Nia` | Present in the type map, but marked non-working and has no dedicated geometry check. Do not use it. |

Spells are checked in array order and only one spell is sent per CastSpells
iteration. CastSpells runs only when targeting is enabled or a selected target
is detected. `Direct` and both missile types always require a selected target.

### `tools`

```json
"tools": {
  "rope": {"hotkey": "rope"},
  "shovel": {"hotkey": "shovel"}
}
```

The object key is the tool name used by the waypoint implementation. The
`hotkey` value must be a logical name present in `hotkeys`; rope and shovel
reserve its configured cooldown before sending the physical key.

### `containers`

```json
"containers": {
  "main_bp": "Backpack",
  "loot_bp": "Beach Backpack"
}
```

Only `main_bp` and `loot_bp` are read. They are required by `deposit` and
`stash_lootbp()` and must match the visible container/pattern names.

### `hunt_config`

```json
"hunt_config": {
  "cap_leave": 20,
  "mana_name": "great mana potion",
  "mana_hotkey": "gmp",
  "take_mana": 2400,
  "mana_leave": 300,
  "health_name": "supreme health potion",
  "health_hotkey": "uhp",
  "take_health": 80,
  "health_leave": 15,
  "rune_name": "stone shower rune",
  "rune_hotkey": "stone shower rune",
  "take_rune": 500,
  "rune_leave": 50,
  "ammo_name": "diamond arrow",
  "ammo_hotkey": "ammo",
  "take_ammo": 800,
  "ammo_leave": 150
}
```

- `cap_leave`: leave/not-hunt capacity threshold.
- `*_name`: enables that resource and supplies its NPC item name.
- `*_hotkey`: logical hotkey used to read the displayed item count.
- `take_*`: desired count after buying and minimum count for going hunting.
- `*_leave`: minimum count while hunting.

Supported resource prefixes:

- Potions: `mana`, `health`, `mana2`, `health2`, `mana3`, `health3`.
- Ammo: `ammo`, `ammo2`, `ammo3`.
- Runes: `rune`, `rune2`, `rune3`.

For each prefix `X`, the complete key set is:

```text
X_name
X_hotkey
take_X
X_leave
```

### `persistent_actions`

This is the array documented in section 5. Actions are submitted concurrently
through a worker pool. A long-running invocation can overlap a later invocation
of the same persistent action. Hotkey cooldown reservation prevents duplicate
key use where that action uses the manager, and the waypoint action lock pauses
new persistent submissions while a waypoint `action` or `call` runs.

### `monsters_to_ignore`

```json
"monsters_to_ignore": ["Rat", "Snake"]
```

Exact, case-sensitive names excluded from target selection, lure counts.

### `special_areas`

```json
{
  "name": "do_not_cross",
  "type": "both",
  "startPosition": [32717, 32787, 8],
  "width": 3,
  "height": 2
}
```

| Key | Type/default | Meaning |
|---|---|---|
| `name` | string | Descriptive only. |
| `type` | string, `"both"` | `walking`, `targeting`, or `both`. Invalid values create no active mask. |
| `startPosition` | `[x, y, z]` | Top-left world tile and floor. |
| `width` | positive integer | Number of blocked tiles along X. |
| `height` | positive integer | Number of blocked tiles along Y. |

`walking` affects normal walker paths. `targeting` affects follow, distance,
loot, and target reachability paths. `both` affects both systems.

## 7. Complete configuration template

The following is valid JSON and contains every active section. Remove unused
entries rather than leaving fake hotkeys or items in a production profile.

```json
{
  "script_options": {
    "session_time": 6,
    "imbuements": [],
    "block_walk_on_raid_pos": true,
    "target_monsters_priority": "distance",
    "wptlure_delay_multiplier": 1.2,
    "wptlure_monsters_to_enable_targeting": 3,
    "wptS&A_monsters_to_enable_targeting": 3,
    "wptlure_monsters_force_targeting": 8,
    "wptlure_monsters_in_box_togo_next_wpt": 1,
    "delay_before_checkmonsterscount": 500,
    "use_tool_on_bodies": [false, "none"]
  },
  "hotkeys": [
    {
      "name": "exura",
      "hotkey": "F1",
      "group": "heal",
      "cooldown": 1
    },
    {
      "name": "gmp",
      "hotkey": "F2",
      "group": "item",
      "cooldown": 1
    },
    {
      "name": "exori",
      "hotkey": "F3",
      "group": "attack",
      "cooldown": 4,
      "group_cooldown": 2
    },
    {
      "name": "food",
      "hotkey": "F4",
      "group": "item",
      "cooldown": 1
    },
    {
      "name": "rope",
      "hotkey": "F5",
      "group": "item",
      "cooldown": 1
    },
    {
      "name": "shovel",
      "hotkey": "F6",
      "group": "item",
      "cooldown": 1
    }
  ],
  "healing": [
    {
      "name": "hp spell",
      "type": "hp",
      "below_percent": 85,
      "min_mana_percent": 10,
      "hotkey": "exura"
    },
    {
      "name": "mana potion",
      "type": "mp",
      "below_percent": 70,
      "min_health_percent": 45,
      "hotkey": "gmp"
    }
  ],
  "target_monsters": [
    {
      "name": "Any",
      "action": "follow",
      "only_trapped": false,
      "need_attack_me": "false"
    }
  ],
  "target_spells": [
    {
      "type": "Self small explosion",
      "hotkey": "exori",
      "min_mana_percent": 10,
      "min_monster_count": 2,
      "monsters": "all",
      "full_harmony": false,
      "pvp_protect": false
    }
  ],
  "tools": {
    "rope": {"hotkey": "rope"},
    "shovel": {"hotkey": "shovel"}
  },
  "containers": {
    "main_bp": "Backpack",
    "loot_bp": "Beach Backpack"
  },
  "hunt_config": {
    "cap_leave": 20,
    "mana_name": "great mana potion",
    "mana_hotkey": "gmp",
    "take_mana": 2400,
    "mana_leave": 300
  },
  "persistent_actions": [
    {
      "action": "anti_paralyze",
      "interval": 1,
      "args": {"hotkey": "exura"}
    },
    {
      "action": "eat_food",
      "interval": 10,
      "args": {"hotkey": "food"}
    }
  ],
  "monsters_to_ignore": [],
  "special_areas": []
}
```

## 8. `GLOBALCFG.json`

These values are global rather than character-profile settings:

| Key | Type/default | Meaning |
|---|---|---|
| `CONFIGFILE` | string, required | Active JSON file under `scripts`. Include the `.json` extension. |
| `WAYPOINTSFILE` | string, required | Active waypoint file under `scripts`. Include the extension. |
| `NodeIgnoreRadio` | integer, `3` | X/Y radius in which a `node` waypoint counts as reached. |
| `WalkingMethod` | string, `"arrows"` | `arrows` uses keyboard movement; `mouse` uses minimap clicks. |
| `MapClickDelay` | milliseconds, `1500` | Wait after a minimap click in mouse walking mode. |
| `Human_Input` | boolean, `false` | Enable the alternate human-input behavior used by `MoveMouse`. |
| `DistanceLoot` | boolean, `false` | Walk to tracked corpse positions before looting. |
| `LootHotkey` | string, `"F1"` | Physical loot hotkey. |
| `DC_STREAM` | boolean, `false` | Start the bot's OBS-backed Discord stream window. |
| `DC_USER_ID` | integer, `0` | Discord user ID used by message notifications. |

Example:

```json
{
  "DC_USER_ID": 0,
  "DC_STREAM": false,
  "NodeIgnoreRadio": 3,
  "Human_Input": true,
  "MapClickDelay": 1500,
  "WalkingMethod": "arrows",
  "DistanceLoot": false,
  "LootHotkey": "F5",
  "CONFIGFILE": "character.json",
  "WAYPOINTSFILE": "spawn.txt"
}
```

Editing only the file names in `GLOBALCFG.json` updates the GUI's configured
labels but does not apply the files until the corresponding reload command or a
session switch succeeds. The GUI shows pending names when configured and loaded
files differ.

## 9. Known compatibility notes

- `call traven ...` occurs in an old waypoint file but no `traven` function
  exists. Use an `action travel_*` command or `npc_say(...)` instead.
- `doors (...)` is emitted by the WPT Creator but is not dispatched; use
  `door (...)`.
- `skip_portal` and `skip_portal_2` are registered actions but currently call
  an undefined function named `jump_label`. They raise an error instead of
  jumping; use a conditional jump helper until the implementation is fixed.
- `check_imbu(..., label_skip='name')` currently calls the logger with two
  arguments and raises an error on the no-missing-imbuement branch. The
  `label_jump` missing-imbuement branch works.
- Top-level `label_actions` is currently unused.
- `target_monsters[].use_item` is parsed but currently has no effect.
- `Gran Mas Nia` is registered but has no implemented directional test.
- Prefer JSON booleans for most boolean fields. Keep
  `target_monsters[].need_attack_me` as the string `"true"`/`"false"` because
  that field is compared as a string.
- Names of monsters, containers, items, and image patterns should match the
  client/pattern data exactly.
