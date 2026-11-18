# Persona 5 Royal: Jose's EXP Boost in the Depths and the Qliphoth World

Why the stamp bonus you set with Jose (EXP, money, items) normally switches off when you enter the Depths of
Mementos, why it stays on after you go back up and come down again, and what finally turns it off after
Yaldabaoth.

*PC version (Steam build 15515071, image base `0x140000000`). Static analysis of the executable, the decompiled
field, event and scheduler scripts, and the field hit tables (`FIELD/HIT/*.HTB`), November 2026. Not yet checked in
game; the last section lists what to watch.*

## Short version

- Jose's modifier isn't a timed buff and nothing "clears" it. The stamp points you give to EXP, money and items
  are **permanent save data**. Only a reset with flowers changes them.
- The bonus applies whenever two flags are on: **flag 12310 (`0x50000000 + 22`, "in Mementos")** and flag
  `0x20000000 + 5053` (Jose's shop unlocked, never turned off). The real switch is flag 12310.
- Entering the Depths "clears" the boost because the trigger at the end of the Path of Da'at,
  `FHIT_192_061_00` procedure `GO_TO_161_01`, does `BIT_OFF(0x50000000 + 22)` before `CALL_FIELD(161, 1, 0, 0)`.
- **That trigger only runs once.** Its hit-table entry is active only while flag `0x20000000 + 5056` is **off**,
  and `GO_TO_161_01` turns 5056 **on**. From then on the hit-table entry for `GO_TO_161_01_GENERIC` is used
  (active only while 5056 is **on**), and that procedure sends you to field 161 **without** clearing 12310.
- Every way back into Mementos turns 12310 on again: the Depths entrance exit (`FHIT_161_001_00`
  `DUNGEON_EXIT`), the 12/24 trip to Leblanc and back (`FHIT_010_001_00` `Main_LastDungeonShop`), and entering
  from Shibuya (`FINI_001_006` `GOTO_MEMENTOS`). So after one trip back up, the flag stays on for the rest of the
  Depths and the Qliphoth World. The EXP getter checks only that flag, so battles there get the multiplier.
- **After Yaldabaoth nothing clears it straight away.** No event, scheduler or calendar script turns flag
  12310 off, and nothing in the game code wipes its flag section. The section is saved and loaded with the rest of
  the flags, so the flag carries into the third semester. It goes off at the **first Metaverse exit procedure** that
  runs after that (a `dungeon_return` / `FLD_EXIT_DUNGEON`): leaving Mementos or Maruki's Palace through an exit,
  or a palace boss clear that returns you to the real world. Until then the boost also applies in palace battles,
  Maruki's Palace included.

## 1. Where the modifier lives

Main game data block `0x142851A90` (the same one the party table lives in):

| Address | Offset | Meaning |
|---|---|---|
| `0x142869708` | `+0x17C78` | Stamp records, 10 × 8 bytes (bitmask of stamps + a count byte). `FLD_GET_TOTAL_STAMP_POINT` (`0x140FDF350`) sums them. |
| `0x142869758` | `+0x17CC8` | u16: **EXP** points given to Jose ("cognition change") |
| `0x14286975A` | `+0x17CCA` | u16: **money** points |
| `0x14286975C` | `+0x17CCC` | u16: **item** points |
| `0x142869768` | `+0x17CD8` | used by the reset code (reset/flower cost bookkeeping, not decoded) |

Only the Mementos shop writes these (`0x140F5C680`, `0x140F5CF10`; the flower reset `0x140F148F0` zeroes all three).
The save classes are `DataBlockFldSaveATDngStamp` and `DataBlockFclMementosShop` (RTTI). Nothing else zeroes them.
Jose's tutorial (`SCRIPTTUTORIAL_DUNGEON_030`) says the same: points are "delegated" and only flowers reset them.

The points index a table loaded from `field/ftd/fldATDngBonus.ftd` (pointer at `0x142900940`, 12-byte rows; the
loader itself is in the protected `.sdata` code). Rows from the file (level: EXP%, money%, item value, then three
fields not decoded, probably costs):

```
0: 100 100  100  0  0 0      6: 160 160  700  9 5 3
1: 110 110  200  5  3 1      7: 170 170  800 10 6 4
2: 120 120  300  5  3 1      8: 180 180  900 10 6 4
3: 130 130  400  7  4 2      9: 190 190 1000 11 7 5
4: 140 140  500  7  4 2     10: 200 200 1100 12 7 5
5: 150 150  600  9  5 3
```

## 2. Where it's applied (the gate)

`0x1410CE820` returns the EXP multiplier:

```
if BIT_CHK(0x50000016) && BIT_CHK(0x200013BD):      // flag 12310 "in Mementos", flag 0x20000000+5053
    return table[ word[0x142869758] ].exp * 0.01     // row +0x10 in the loaded table
return 1.0
```

`0x1410CE890` is the same for money (`0x14286975A`, column +0x12). The battle-result routine `0x140927050` calls
both (`0x14092735D`, `0x140927366`) and also checks the same two flags inline (`0x1409275F0`, `0x1409276A0`) for
its bonus display. `0x140D65730` is the game's `BIT_CHK`. Flag section 5 starts at `0x142855800`, so flag 12310 is
bit 22 of the dword at `0x142855800` (mask `0x00400000`). The item column is read by `FLD_GET_SCH_OBJ_ITEM`
(`0x141179DE0`), which checks only 5053. Only Mementos scripts call it.

Nothing in that path checks the field number. Any battle fought while flag 12310 is on gets the bonus, in or
out of Mementos.

Flag 12310 is the game's general "this dungeon is Mementos" flag, not a Jose flag: `dungeon_start_format` uses it to
pick the Mementos progress count (`COUNT(139)` to `COUNT(272)`) instead of the palace one, and about 90 game-code
sites read it (BGM choice, ailment bonus, etc.).

## 3. Who sets and clears flag 12310

On (only these):

| Script | Procedure | When |
|---|---|---|
| `FINI_001_006` | `GOTO_MEMENTOS` | entering Mementos from Shibuya (also turns on 12308 = `+20`) |
| `FINI_191_001` | `D191_01_START_EV` | Mementos floor init on 12/24 (if `0x20000000+2700` and `COUNT(152) < 1000`) |
| `FINI_190_001` | `EXTRA_SET` | Mementos entrance, first arrival |
| `FHIT_161_001_00` | `DUNGEON_EXIT` | the "go back up" choice at the Depths entrance, then `CALL_AT_DUNGEON(8, 1, 1)` |
| `FHIT_010_001_00` | `Main_LastDungeonShop` | 12/24: back from Leblanc to the Mementos entrance (`CALL_FIELD(190, 1, 15, 0)`) |
| others | | `SCHEDULER_05` (story day), `CORP004`/`CORP008` (Mementos confidant fights), `FINI_002_005`, `FINI_003_002` |

Off (only these):

- `FHIT_192_061_00` `GO_TO_161_01`: the first Depths entry (below).
- `FHIT_190_001_00` `GOTO_SHOP`: 12/24 to 12/31 (flag `0x20000000+3485`), "exit" at the Mementos entrance goes to
  Leblanc and turns off 12308 and 12310.
- Every `dungeon_return` procedure (a copy sits in each palace, Mementos and Depths init or hit script; it's called
  by `DUNGEON_EXIT`, `GOTO_EXIT`, boss-clear and quest-end procedures and by `dungeon_return_calendar`):
  `BIT_OFF(+20)`, `BIT_OFF(+22)`, `BIT_OFF(+21)`, then `FLD_EXIT_DUNGEON()`. The native `FLD_EXIT_DUNGEON`
  (`0x141171660`) also clears bits 20 and 22 itself (`and [section5], 0xFFBFFFFF` at `0x14117174F`).
- `CORP008` (a confidant Mementos fight).

No event script, scheduler script or calendar routine writes flag 12310 (E080_* only read it). The game code never
writes it except inside `FLD_EXIT_DUNGEON`. No routine clears flag section 5 on its own: `0x140D63840` clears the
whole 0x640-byte flag area only on a new game, and the save loader `0x140D61160` restores all 0x640 bytes,
section 5 included.

## 4. Why the Depths clears it only once

`FHIT_192_061_00` (end of the Path of Da'at, field 192_061):

```
GO_TO_161_01 (proc 1):
    if COUNT(152) < 10000:  D08_01_e437_011_START(); BIT_OFF(0x50000000+22); BIT_ON(0x20000000+5056); CALL_FIELD(161,1,0,0)
    else:                   prompt "go?";           BIT_OFF(0x50000000+22);                          CALL_FIELD(161,1,0,0)

GO_TO_161_01_GENERIC (proc 2):
    walk the party forward; FLD_REQ_NEXT_SCN_FADE(161,1,0,0)
    if not 1/13..3/31: CALL_FIELD(161,1,0,0) else CALL_AT_DUNGEON(8,2,0)     // no BIT_OFF
```

`FIELD/HIT/F192_061_00.HTB`. Each entry is 0x3C bytes. Six u32 flag slots sit at +0x0C. Slots 0 to 2 mean "active
only while the flag is on" and slots 3 to 5 mean "only while it's off". The procedure index is the u16 at +0x28.

| Entry | Condition | Procedure |
|---|---|---|
| 0, 1 | none | 0 `GO_TO_PREV_SECTION` |
| 2 | slot 3 = `0x200013C0` (5056 must be **off**) | 1 `GO_TO_161_01` |
| 3 | slot 0 = `0x200013C0` (5056 must be **on**) | 2 `GO_TO_161_01_GENERIC` |

I checked the slot meaning against two one-time triggers whose procedures set their own condition flag:
`F161_001_00` entry 3 (`D08_01_e437_020_START`, slot 3 = 3138, sets 3138) and `F192_062_00` entry 2
(`AT_D_e512_040`, slot 3 = 4528, sets 4528).

Only `GO_TO_161_01` (12/24) and `FINI_190_001` (1/13 to 3/31) turn 5056 on, and nothing turns it off. So:

1. You descend on 12/24 with points in EXP. Flag 12310 is on, so the boost applies in Mementos.
2. First Depths entry: `GO_TO_161_01` clears 12310 and sets 5056. **No boost in the Depths.**
3. Back up to Mementos (Depths entrance "exit", or Leblanc and back): 12310 is on again.
4. Down again: 5056 is on, so `GO_TO_161_01_GENERIC` runs and 12310 **stays on**. The boost applies in the Depths.
5. Field 161 init (`FINI_161_001` `f161_001_init`) and the rest of the Depths and Qliphoth World scripts never
   clear 12310. The Holy Grail and Yaldabaoth sequences are boss fights and events (`FLD_START_BOSS` 60 to 63,
   `CALL_EVENT`) with no `dungeon_return`.

The 12/24 entrance "exit" (`GOTO_SHOP`) does clear the flag, but `Main_LastDungeonShop` sets it again on the way
back, and the next descent uses the GENERIC trigger. So leaving doesn't help on 12/24. The normal exit
(`dungeon_return` via `DUNGEON_EXIT_CHECK`) is unreachable while flag 3485 is on, because `GOTO_EXIT` sends you to
`GOTO_SHOP` first.

## 5. What clears it after Yaldabaoth

It isn't the end of the Grail, the ending, the 1/2 transition or a date: none of those scripts touch flag 12310,
and it survives saving. It goes off at the next run of any of these:

- a `dungeon_return` procedure (the Metaverse exit/clear path: Mementos entrance exit `FHIT_190_001_00`
  `GOTO_EXIT` outside 12/24 to 12/31, palace entrance `DUNGEON_EXIT`, boss-clear and quest-end returns), each of
  which ends in `FLD_EXIT_DUNGEON`;
- `GO_TO_161_01` (can't happen again, 5056 is on);
- `GOTO_SHOP` (only 12/24 to 12/31).

In practice that's the **first time you leave the Metaverse in the third semester**, through Mementos or Maruki's
Palace (162). Entering Mementos sets the flag again anyway. Entering Maruki's Palace doesn't clear it
(`FINI_162_001` clears it only in `dungeon_return`). So if the first third-semester Metaverse trip is the palace,
its battles should keep the boost until you leave. One side effect: while the flag is on, the palace's
`dungeon_start_format` takes the Mementos branch (`SET_COUNT(272, COUNT(139))` instead of the palace count).

## What to watch live

Poll these (with a memory reader or flag logger) on 12/24 and after Yaldabaoth:

| What | Address | Check |
|---|---|---|
| Flag 12310 "in Mementos" | dword `0x142855800` | bit 22 (`0x00400000`) |
| Flag 5056 (Depths entry done) | dword `0x142855778` | bit 0 |
| Flag 5053 (shop unlocked) | dword `0x142855774` | bit 29 |
| EXP / money / item points | u16 `0x142869758` / `0x14286975A` / `0x14286975C` | value 0 to 10 |
| EXP multiplier | breakpoint or return value of `0x1410CE820` | 1.0 means no boost |

Expected: 12310 goes off on the first `CALL_FIELD(161)`, on again after "go back up" or the Leblanc trip, stays on
after the second Depths entry, stays on through Yaldabaoth and the date change, and goes off at the first
third-semester Metaverse exit.

Not decoded: which physical trigger runs HTB entry 3 (no hit shape: `type 00000000`). If 12310 goes off on the
second Depths entry in game, that entry isn't what runs. Also not traced: the exact EXP formula inside
`0x140927050`. The getters return the multiplier and the routine keeps it (`xmm12`, `xmm13`), but I didn't follow
the multiplication.
