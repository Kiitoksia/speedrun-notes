# Persona 5 Royal: Buying Will Seeds from Jose

Why Jose won't sell the Will Seeds you skipped (for example the Casino of Jealousy ones) when your first real
Mementos trip is on 12/24, and what a route has to do to make them show up.

*PC version (Steam). Static analysis of the decompiled field, NPC and scheduler scripts and Jose's stock table
(`INIT/FCLTABLE.BIN`), November 2026. In game, the route behaviour matches: without an optional 2nd Mementos
visit, the seeds aren't for sale on 12/24. The flag-level details still need a check in game; the last section
lists what to look at.*

## In plain terms

Jose only sells a palace's Will Seeds if **two** things are true when you open his shop:

1. **The seeds are unlocked.** This happens the first time you **leave Mementos through the normal exit after
   you've met Jose**. The forced 5/7 visit doesn't count: you meet Jose there, but that visit ends at the first
   barrier and sends you straight back to the real world without using the exit. From 12/24 to 12/31 the exit is
   replaced by the "airsoft shop / clinic" menu, which doesn't unlock them either. So without a 2nd visit before
   12/24, the seeds stay locked until your first normal Mementos exit in the 3rd semester.
2. **His stock is new enough.** Jose's stock depends on the **deepest Mementos barrier you've opened**, not on
   which palaces you've cleared. The Casino of Jealousy seeds only appear once the 6th barrier is open (opens on
   any date from 11/21), or from the 3rd semester on, when Jose gets his full stock anyway.

**The 2nd visit can be tiny:** enter Mementos, walk to the exit on the entrance platform and pick "Return to the
real world". No fights, no floors, no talking to Jose. It costs the time slot. Any day from 5/8 to 12/23 works.
On the first entry in some date windows a one-time "Mementos has expanded" scene plays (6/6–7/9, 7/10–8/21,
8/22–10/11). 5/8–6/5 and 10/12–12/23 have none.

**On 12/24:** buy the seeds at a Jose station **after** the 6th barrier. The station at the very top shows the old
stock if you haven't opened the deeper barriers yet. Walking down to the Depths opens them in order.

The rest of this page is the technical detail.

Flag notation: `5026` means `0x20000000 + 5026`, the same flag the decompiler shows as global bit 11170.

## 1. Jose's stock table

`INIT/FCLTABLE.BIN` (a big-endian PAK) holds `fclMementosShopTable.ftd` (199 rows × 20 bytes) and
`fclMementosShopFlagTable.ftd` (8 flags). A row is:

```
u32 cond      1 = conditional row
u32 flagA     must be ON (0x200013A2 = 5026 on every Will Seed row)
u32 flagB     the seed's "already obtained" flag: the row is hidden once it's ON
u16 item      Will Seeds are key items 0x40E1..0x40FB
u16 price     400 for seeds
u32 stock
```

An all-zero row ends each lineup. That gives 8 lineups, matched in order to the flag table:

| Lineup | Flag | Set by | Will Seeds |
|---|---|---|---|
| 0 | `89`   | Barrier 1, `FHIT_191_011_00` `MEME_LOCK_CLEAR_01` (the forced 5/7 visit) | Lust |
| 1 | `3962` | Barrier 2, `FHIT_191_061_00` `MEME_LOCK_CLEAR_02`, from 6/6 | + Vanity |
| 2 | `3963` | Barrier 3, `FHIT_193_011_00` `MEME_LOCK_CLEAR_03`, from 7/10 | + Gluttony |
| 3 | `3964` | Barrier 4, `FHIT_193_061_00` `MEME_LOCK_CLEAR_04`, from 8/22 | + Wrath |
| 4 | `3965` | Barrier 5, `FHIT_194_011_00` `MEME_LOCK_CLEAR_05`, from 10/12 | + Greed |
| 5 | `3966` | Barrier 6, `FHIT_194_061_00` `MEME_LOCK_CLEAR_06`, from 11/21 | **+ Jealousy** |
| 6 | `3100` | Depths of Mementos cleared | **none** |
| 7 | `4097` | Start of the 3rd semester (`SCHEDULER_01`) | all, + Pride |

("MEME" is Atlus's internal shorthand for Mementos.)

Jose uses the lineup with the **highest** flag that's on. The game's own script copy of this check,
`MEMESHOP_CHECK_NEW_LINEUP` in `FNPC191_001_00` (Jose's NPC script), tests `4097`, `3966` … `3962`, `89` in that
order. The 3rd semester fits this too: flag `89` is still on, but Jose shows the full stock.

A barrier flag is only set when you physically reach that barrier while its date window is open. A route that
skips Mementos has only `89`, so on 12/24 Jose starts on lineup 0 (Lust seeds only). Lineup 6, used after the Depths
are cleared and before the 3rd semester, has no seeds at all.

## 2. The seed unlock flag (5026)

- **Checked:** `flagA` on all 84 Will Seed rows. Jose's script checks the same flag before his "I've got seeds
  now" greeting (`MEMESHOP_BRANCH_GREETING`, `MEMESHOP_BRANCH_LINEUP_TXT`, which test `5026 == 1 && 5027 == 0`).
- **Set:** only by `AT_D_EXIT_SCENARIO_SET` in `FHIT_190_001_00` (the Mementos entrance):
  `if (BIT_CHK(5037)) BIT_ON(5026);`. No other field, NPC, event or scheduler script sets it.
- **When that runs:** only from `GOTO_EXIT`, on the normal "Return to the real world" path (`DUNGEON_EXIT_CHECK`
  option 1, which sets local flag 48), right before the return to the calendar.
- **`5037` = met Jose.** The first talk with Jose (`AT_D_shop_Main`) plays his intro event E499_200 and sets it.

### Why the 5/7 visit doesn't unlock them

- Before barrier 1 is open, `GOTO_EXIT` takes the `88 == 1 && 89 == 0` branch (`GOTO_EXIT_NG_EX`): you can't
  leave through the entrance.
- The visit ends at barrier 1. `MEME_LOCK_CLEAR_01` plays its events, sets `89`, then calls its own
  `dungeon_return()` and `CALL_CALENDAR()`. It never goes through `AT_D_EXIT_SCENARIO_SET`.

So after 5/7, `5037` is on and `5026` is still off. Any later normal exit turns `5026` on.

### Why 12/24 doesn't unlock them

`GOTO_EXIT` starts with `if (BIT_CHK(3485)) { GOTO_SHOP(); return; }`. Flag `3485` is on from **12/24 to 12/31**
(set by the Mementos entrance init `FINI_190_001` for that date range and by the 12/24 scheduler, cleared when
the period ends). `GOTO_SHOP` is the "We don't know when the police might come after us…" exit that only offers
the airsoft shop or the clinic, and it never sets `5026`.

### The minimum 2nd visit

Enter Mementos, use the exit on the entrance platform, pick "Return to the real world". It needs:

- `89` and `5037` on (both from 5/7),
- a date outside 12/24–12/31.

The one-time entry scenes come from `D190_01_MEME_OPEN_EV` in `FINI_190_001`: E218_003 (6/6–7/9), E267_003
(7/10–8/21) and E311_103 (8/22–10/11), each played on the first entry in its window.

## What to check in game

1. On 12/24 before talking to Jose: `5026` off (if no 2nd visit), only `89` on among the lineup flags, `3485` on.
2. A quick enter-and-exit before 12/24 should turn `5026` on.
3. With `5026` on, the top station on 12/24 (only `89`) should show only the Lust seeds you didn't pick up. With
   `3966` on as well, the Casino seeds (`0x40F5`–`0x40F7`) should appear.
