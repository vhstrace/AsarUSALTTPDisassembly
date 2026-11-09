# The Legend of Zelda: A Link to the Past (SNES) Game Genie codes

These codes were made from the USA release (version 1.0, the only USA
release). SNES Game Genie codes have no compare byte; the original byte in
each raw form is documentation only. All effects come from the disassembly.
They have not been tested in play unless an entry says so.

## The Lamp casts Bombos

**Codes:** `DDC5-6DFA` `FBC5-6DBA` `11C5-6FFA` `CEC5-6F9A` `D6C5-6FBA`
(use all five)
**Raw form:** `07A270:00` (orig `$02`); `07A272:19` (orig `$2F`);
`07A274:66` (orig `$1C`); `07A275:AF` (orig `$8F`); `07A276:08`
(orig `$09`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 19:17:47

Tags: `demo-ready` `video-ready` `play-start`

**Mechanism:** When Link uses the Lamp, `LinkItem_Lamp` (`Bank07.asm`)
checks for the Lamp and its magic cost, lights torches, and then adds the
Lamp's flame ancilla:

```
LDY.b #$00
LDA.b #$1A
JSL.l AddMagicPowder

JSL.l Dungeon_LightTorch

LDY.b #$02
LDA.b #$2F
JSL.l AddLampFlame
```

The codes change the last call to the one the Bombos Medallion makes in
`LinkState_UsingBombos`: `LDY.b #$00 : LDA.b #$19 : JSL.l AddBombosSpell`.
`A` is the ancilla type (`$19` is the Bombos spell). `Y` is passed to
`Ancilla_CheckForAvailableSlot` as the most copies of that ancilla that can
already be out, minus one. `Y = 0` allows only one Bombos spell at a time;
the Lamp's `Y = 2` would allow three, and they would share the spell's
`$7F58xx` work RAM. `AddBombosSpell` sets its own data bank and ends with
`RTL`, so it can be called from bank `$07`. It places the fire around
Link's position (`$20`, `$22`), loads the medallion graphics (`$0AAA`) and
plays the spell's sound itself.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has these bytes at `$07:A24D` (`LinkItem_Lamp`, PC
`$03A24D`):

```
A5 6C D0 37 20 73 B0 90 32 AF 4A F3 7E F0 1B A2
06 20 AB B0 90 14 A0 00 A9 1A 22 F0 92 09 22 EC
F3 01 A0 02 A9 2F 22 1C 8F 09 ...
```

`A0 02` is at `$07:A26F`, `A9 2F` at `$07:A271` and `22 1C 8F 09`
(`JSL $098F1C`, `AddLampFlame`, PC `$048F1C`) at `$07:A273`.
`AddBombosSpell` is at PC `$042F66` (`$08:AF66`) and starts `8B 4B AB`
(`PHB : PHK : PLB`). The game's two Bombos calls, at `$07:865C`
(`Player_BombosSpell`) and `$07:A641` (`LinkState_UsingBombos`), are both
`A0 00 A9 19 22 66 AF 08`. The codes decode back to their raw forms with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Tested by the user in Mesen2.
- Link needs the Lamp (it is in the chest in Link's house at the start).
  No sword or Bombos Medallion is needed, and the Medallion checks (boss
  rooms, doorways, free ancilla slots) are skipped.
- The Lamp still costs its normal magic and still lights torches. It no
  longer makes its small flame.
- Link isn't held in the medallion pose, so he can keep walking while the
  fire burns. Enemies aren't frozen as they are with the Medallion
  (`LinkState_UsingBombos` sets `$0FC1` on each frame), so they keep
  moving.
- Using the Lamp again while the fire burns costs magic but adds nothing.
- While the spell runs, `$0112` is set, which blocks the Start and map
  menus. If Link leaves the room or screen before the spell ends, the
  spell may be removed before it clears its state, so the menus or sprite
  graphics might stay wrong until another medallion is used. Not checked.

## Skip the intro and title screen, go straight to file select

**Codes:** `D4A4-0DED` `DDA4-07ED` (use both)
**Raw form:** `0CC123:02` (orig `$08`); `0CC12F:00` (orig `$03`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 19:31:03

**Mechanism:** `Module_Intro` (`Bank0C.asm`) runs the intro one
submodule (`$11`) at a time. From submodule `$08` (the title screen) on,
it checks for ABXY or Start and jumps to the file select setup:

```
Module_Intro:
    LDA.b $11 : CMP.b #$08 : BCC .dontCheckInput
        LDA.b $F6 : AND.b #$C0 : ORA.b $F4 : AND.b #$D0 : BEQ .noPressedButtons
            JMP.w FadeMusicAndResetSRAMMirror
        .noPressedButtons
    .dontCheckInput
```

The first code changes `CMP.b #$08` to `CMP.b #$02`, so the check starts
right after submodule `$01`. Submodule `$01` shows the Nintendo logo while
it clears WRAM and loads the text pointers, palettes and item graphics,
then loads the title graphics. The second code sets the `BEQ` offset to
`$00`, so the jump is taken with no button pressed.
`FadeMusicAndResetSRAMMirror` then silences the music, clears the save
buffer and sets module `$01` (file select). This is the same path as
pressing Start on the title screen, only earlier. `FileSelect_InitializeGFX`
loads its own graphics and palettes, and
`FileSelect_TriggerNameStripesAndAdvance` sets the screen brightness
(`$13`), so the skipped title setup isn't needed.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A5 11 C9 08 90 0D A5 F6 29 C0 05 F4 29 D0 F0 03 4C
F0 C2` at `$0C:C120` (`Module_Intro`, PC `$064120`). `C9 08` is at
`$0C:C122`; `F0 03` at `$0C:C12E` skips the 3-byte `JMP $C2F0`
(`FadeMusicAndResetSRAMMirror`, PC `$0642F0`). The codes decode back to
their raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- The Nintendo logo still shows for about half a second, since submodules
  `$00` and `$01` do setup that file select needs. Starting the check at
  submodule `$01` would skip the WRAM clear.
- The title screen, the Triforce animation and the attract mode (the
  story) are never shown.
- Anything that returns to module `$00`, such as resetting the game, also
  skips straight to file select.

## Skip "Nintendo presents", the intro and the title screen

**Codes:** `DFA1-0D8D` `DDA1-077D` `4AA5-0D7D` `D4A4-0DED` `DDA4-07ED`
(use all five)
**Raw form:** `0CC162:01` (orig `$0F`); `0CC16C:00` (orig `$0A`);
`0CC170:2C` (orig `$20`); `0CC123:02` (orig `$08`); `0CC12F:00`
(orig `$03`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 19:33:27

**Mechanism:** The last two codes are "Skip the intro and title screen, go
straight to file select" (`D4A4-0DED` `DDA4-07ED`). With only those, the
Nintendo logo still shows for about 26 frames, because intro submodules
`$00` and `$01` must run first. They clear WRAM over 8 frames and load the
text pointers, palettes and item graphics. The first three codes keep that
work but hide the logo and cut the wait:

```
Intro_Init:
    JSL.l Intro_SetupScreen
    LDA.b #$0F : STA.b $13        ; full brightness
    STZ.b $B0
    INC.b $15
    INC.b $11
    LDA.b #$0A : STA.w $012F      ; logo twinkle sound

    .justLogo
    JSR.w Intro_DisplayNintendoLogo
    LDA.b $B0 : INC.b $B0 : CMP.b #$0B : BCS Intro_LoadTitleGraphics
    ...
Intro_LoadTitleGraphics:
    DEC.b $13 : BNE .waitTillForceBlank
```

- `LDA.b #$0F` becomes `LDA.b #$01`. `Intro_LoadTitleGraphics` fades the
  logo out by decrementing `$13` once per frame and waits until it reaches
  0. Starting at 1 cuts that 15-frame fade to 1 frame. (0 would wrap to
  `$FF` and wait 255 frames.)
- `LDA.b #$0A` becomes `LDA.b #$00`, so no sound is queued.
- `JSR.w Intro_DisplayNintendoLogo` (`$20`) becomes `BIT $ED82` (`$2C`),
  so the logo's sprites are never put in OAM. `Intro_SetupScreen` turns on
  only the sprite layer (`$1C = $10`), so the screen stays blank. The `BIT`
  only reads a byte and sets flags, and the next instruction reloads `A`.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `22 00 80 02 A9 0F 85 13 64 B0 E6 15 E6 11 A9 0A 8D 2F
01 20 82 ED A5 B0 ...` at `$0C:C15D` (`Intro_Init`, PC `$06415D`). `A9 0F`
is at `$0C:C161`, `A9 0A` at `$0C:C16B` and `20 82 ED` (`JSR $ED82`,
`Intro_DisplayNintendoLogo`, PC `$066D82`) at `$0C:C170`. The last two
codes' bytes are in the "Skip the intro and title screen" entry. The codes
decode back to their raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- The screen is blank for about 12 frames (a fifth of a second) before
  file select, while the setup runs.
- The blank screen shows the backdrop color, which should be black at
  power-on. Not checked.
- This uses all five codes a real Game Genie holds. It includes the codes
  of "Skip the intro and title screen", so don't enter both.
- The same caveats as "Skip the intro and title screen" apply.

## Fastest possible text

**Codes:** `DDA5-6F7C` `DE25-AF76` (use both)
**Raw form:** `0ECA74:00` (orig `$02`); `0ED374:0F` (orig `$04`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 19:36:03

Tags: `demo-ready` `video-ready` `curated` `play-start`

**Mechanism:** This is the ALttP version of the Zelda 1 code "Fastest
possible NPC text". It has two parts.

The first code is for draw speed. `VWF_Render` (`Bank0E.asm`) picks how to
draw the next letter from the draw speed `$1CD5`:

```
VWF_Render:
    LDA.w $1CD5 : CMP.b #$02 : BCC .validSpeed
        LDA.b #$02
    .validSpeed
    JSL.l UseImplicitRegIndexedLocalJumpTable
    dw VWF_RenderRecursive ; 0x00
    dw VWF_RenderSingle    ; 0x01
    dw VWF_InvalidSpeed_1  ; 0x02
```

Speed 0 (`VWF_RenderRecursive`) draws letters until the end of the line or
the next command, all in one frame. Speed 2 and up waits a frame by
decrementing `$1CD5` (`VWF_InvalidSpeed_1`), so speed n draws one letter
every n frames. Speed 0 is the default (`Text_InitializationData` sets
`$1CD6` to 0). Only the messages that start with `[Speed 02]` or
`[Speed 03]` are slow. These include Zelda's "Help me…" telepathy at the
start of the game, her message in the dungeon, and the attract mode story.
The code changes `LDA.b #$02` to `LDA.b #$00`, so speeds 2 and up are drawn
as speed 0.

The second code is for scroll speed. When the box is full, `VWF_Scroll`
moves the text up by `$1CEA + 1` pixels per frame, until it has moved one
line (16 pixels). `Text_Initialize` copies `$20` bytes from
`Text_InitializationData` to `$1CD0-$1CEF`. That table is only `$11` bytes
long, so `$1CE9` and `$1CEA` come from the next table,
`RenderText_MoreInitialSettings` (`$1C` and `$04`). The code changes the
`$04` to `$0F`, so each line scrolls up in one frame instead of in 4.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `AD D5 1C C9 02 90 02 A9 02 22 81 87 00 ...` at
`$0E:CA6C` (`VWF_Render`, PC `$074A6C`); `A9 02` is at `$0E:CA73`. It has
`00 00 00 00 00 00 00 00 01 00 00 00 39 00 00 00 00` at `$0E:D35A`
(`Text_InitializationData`, PC `$07535A`), then `00 00 00 00 00 00 00 00
1C 04 ...` (`RenderText_MoreInitialSettings`); `$04` is at `$0E:D374`, which
is `$1CD0 + $1A = $1CEA`. Nothing else uses `RenderText_MoreInitialSettings`
by name. In `Bank1C.asm`, `$7A, $00` appears 11 times, `$7A, $02` 10 times
and `$7A, $03` 10 times. No message uses speed 1, which this code leaves as
it is. The codes decode back to their raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Tested by the user in Mesen2.
- Use only the second code if you want the scroll faster but the slow
  messages still slow.
- Scrolling a whole line copies the text graphics 16 times in one frame.
  This may cause a lag frame each time a line scrolls.
- `VWF_IgnoreCommand` saves the value after `[Window XX]`, `[Name]`,
  `[Number XX]`, `[Position XX]` and `[ScrollSpd XX]` into `$1CEA`. So a
  message with one of these (for example `[Window 02]`) still scrolls at
  its own speed.
- The pause before the game takes input after the box fills (`$1CE9`) is
  not changed.

## Link walks twice as fast

**Codes:** `7D34-6F2A` `4D34-64FA` (use both)
**Optional, also with the sword out or carrying:** `4637-6D2A` `FC37-6FFA`
**Raw form:** `07E227:30` (orig `$18`); `07E228:20` (orig `$10`);
optional `07E233:28` (orig `$14`); `07E234:1A` (orig `$0D`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 19:36:48

Tags: `demo-ready` `video-ready` `curated` `play-start`

**Mechanism:** This is the ALttP version of the Zelda 1 code "Link walks
much faster". `Link_HandleVelocity` (`Bank07.asm`) adds the diagonal flag
to the movement type in `$5E` and loads the speed from `SubVelocityValues`:

```
.load_subvel
    ; Link's speed table.
    LDA.w SubVelocityValues, X
```

The value goes to Link's velocity (`$27`, `$28`), negated for up or left.
`LinkHop_FindArbitraryLandingSpot` multiplies it by 16 and adds it to the
subpixel position, so `$10` is one pixel per frame. Walking (`0x00`, `$18`)
is 1.5 pixels per frame, and walking diagonally (`0x01`, `$10`) is 1 pixel
per axis. The codes double both to `$30` and `$20`. The optional codes
double `0x0C` and `0x0D`, the speeds for walking with the sword held out,
carrying something, or wading in shallow water (`$14` to `$28`, `$0D` to
`$1A`).

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `18 10 0A 18 10 08 08 04 0C 10 09 19 14 0D 10 08 40 2A
...` at `$07:E227` (`SubVelocityValues`, PC `$03E227`). This matches the
table in `Bank07.asm`. `SubVelocityValues` is read only by
`Link_HandleVelocity`. The codes decode back to their raw forms with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Tested by the user in Mesen2, without the optional codes.
- The new speeds stay below the Pegasus Boots dash (`$40`, 4 pixels per
  frame), which the game already moves Link at, so wall checks should
  still work.
- Stairs, slipping, pushing and the dash keep their normal speeds.
- Some cutscenes that walk Link for a set number of frames may move him
  further than usual. Not checked.

## Dash without the Pegasus Boots

**Code:** `3DBC-AFF3`
**Raw form:** `079BA4:E0` (orig `$04`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 19:38:12

Tags: `demo-ready` `curated` `play-start`

**Mechanism:** This is the ALttP idea behind the SMB3 code "P-meter fills
as soon as you move with B held": top speed without the item that
normally gives it. When Link presses A, `Link_HandleAPress`
(`Bank07.asm`) picks an action in X (lift, read, dash, grab, chest and so
on). It then checks that action's mask against the ability flags in
`$7EF379`:

```
.attemptAction
; Check to see if we have the capability for this action.
LDA.w Link_AbilityChecks, X : AND.l $7EF379 : BEQ .cantDoAction
```

`Link_AbilityChecks` entry `0x02` (dash) is `$04`, the Run bit. The game
sets that bit only when Link gets the Pegasus Boots (`ORA.l $7EF379` in
`ancilla_init.asm`). The code changes it to `$E0`, the mask used by the
lift, talk and chest actions. A new file can lift bushes at once, so it
has one of those bits, and the dash passes the check.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `E0 40 04 E0 E0 E0 E0 E0` at `$07:9BA2`
(`Link_AbilityChecks`, PC `$039BA2`); the `$04` is at `$07:9BA4`. The table
is read only at that one place. `Link_PerformDash` doesn't check the boots
item (`$7EF355`) itself. The code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Tested by the user in Mesen2.
- Hold A to charge, as with the boots. The dash works from the moment Link
  can lift bushes.
- The boots still show as missing in the item menu, and the "Run" ability
  isn't listed.
- Dashing into things like trees, rocks and the bonk spots works as it
  does with the boots, so items can be knocked loose early.

## Link walks through walls

**Code:** `1D29-DDFE`
**Raw form:** `07DC50:60` (orig `$A5`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 19:40:35

Tags: `demo-ready`

**Mechanism:** This is the ALttP version of the Metroid and Zelda 1 "walk
through walls" codes. When Link moves, `TileDetect_Execute` (`Bank07.asm`)
reads the tile type under each test point and jumps through a table, one
for dungeons (`TileDetect_Execute_modules`) and one for the overworld
(`TileDetection_Execute_overworld_modules`). Solid tiles (types `0x01` to
`0x03` in both tables, plus a few others such as `0x26`) go to:

```
TileBehavior_StandardCollision:
    ; $0E is the collision bitfield.
    LDA.b $0A : TSB.b $0E
```

The movement code then pushes Link back out of any tile whose bit is set
in `$0E`. The code changes the first byte (`LDA`) to `RTS`, so solid tiles
act like `TileBehavior_Nothing` and never set a bit.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A5 0A 04 0E 60` at `$07:DC50` (PC `$03DC50`). The
`60` after it is `TileBehavior_Nothing`. Nothing branches or jumps to
`TileBehavior_StandardCollision` except the two jump tables. The game also
has a hidden debug flag, `$037F`, that `TileDetect_Execute` checks; it
turns every tile into a plain floor tile, so it also removes stairs,
water, pits and ledges. This code only removes solid tiles. The code
decodes back to its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Only Link's movement uses these tables. Enemies, thrown objects and
  other sprites still collide with walls as usual.
- Water, pits, ledges, stairs, doors and chests have their own tile types,
  so they still work. Deep water still makes Link swim or fall back.
- Walking past the edges of a dungeon room or into parts of the map that
  are never seen can put Link in odd places, or start a room or screen
  change from a spot the game doesn't expect. He may get stuck or fall
  into the void.
- Rocks, bushes and big blocks are objects or special tiles, so some of
  them may still block Link. Not checked.

## Link walks on deep water

**Code:** `1D28-DFBE`
**Raw form:** `07DCB6:60` (orig `$A5`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 19:41:33

Tags: `demo-ready` `video-ready` `curated` `play-start`

**Mechanism:** This is the ALttP version of the Zelda 1 "walk on water"
code. When Link moves, `TileDetect_Execute` (`Bank07.asm`) jumps through a
tile type table. Deep water (type `0x08`) goes to:

```
TileBehavior_DeepWater:
    LDA.b $0A : TSB.w $0341
    RTS
```

The movement code reads `$0341` to make Link jump in and swim, or, without
the Flippers, to stop him at the edge. The code changes the first byte
(`LDA`) to `RTS`, so `$0341` is never set and deep water acts like floor.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A5 0A 0C 41 03 60` at `$07:DCB6` (PC `$03DCB6`). Entry
`0x08` is `$DCB6` in both the dungeon table at `$07:D7D8` and the overworld
table at `$07:DA2A` (read from the ROM; the dungeon table's comment in
`Bank07.asm` names `TileBehavior_ManualStairs` for this entry, but the ROM
has `$DCB6`). So one code covers both. The code decodes back to its raw
form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Tested by the user in Mesen2.
- Link can't swim while the code is on, even with the Flippers, because he
  never goes into the water. Places that need swimming (like the Zora's
  Waterfall area) are walked instead.
- Tile type `0x0B` (`TileBehavior_UnusedDeepWater`, overworld only) is not
  changed. It is probably unused.
- Shallow water, water stairs and waterfalls have their own tile types and
  work as before.
- If Link is already swimming when the code is turned on, he may stay in
  the swimming state until he reaches land.

## Overworld enemies spawn several copies, and come back after you kill them

**Code:** `C5A8-AF49`
**Raw form:** `09C7B4:A7` (orig `$87`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 19:43:02

Tags: `demo-ready` `video-ready` `curated` `play-start` `challenge`

**Mechanism:** This is the ALttP version of the SMW code "Each enemy spawns
several copies, and comes back after you kill it". The overworld keeps one
bit for each 16x16 map cell at `$7FEF80`. `LoadOverworldSprites` clears it
when Link enters an area. When the camera scrolls, the
`Sprite_ActivateWhenProximal_*` routines check the cells just past the
screen edge, and `Overworld_LoadProximaSpriteIfAlive` loads a cell's enemy
only if its bit is clear:

```
LDA.b [$02] : AND.w Overworld_AliveStatusBits, Y : BNE .alpha
    JSR.w Overworld_LoadSprite
```

`Overworld_LoadSprite` finds a free sprite slot and then sets the bit:

```
.emptySlot
LDA.b [$02] : ORA.w Overworld_AliveStatusBits, Y : STA.b [$02]
```

When an enemy scrolls off screen and is removed, the game clears its bit
(with `SpriteDeathMasks`), so it can load again later. A killed enemy has
its `$0BC0` entry set to `$FF` first, so its bit stays set and it doesn't
come back until Link leaves the area. The code changes `STA.b [$02]` (`$87`)
to `LDA.b [$02]` (`$A7`), so the bit is never set. The `LDA` only reads the
byte again, and the next instruction (`PHX`) doesn't use `A`.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A7 02 19 31 C7 87 02 DA 8A 0A AA` at `$09:C7AF`
(`Overworld_LoadSprite` `.emptySlot`, PC `$04C7AF`); `87 02` is at
`$09:C7B4`. The same bytes are also at `$09:C816`, in the overlord loader
(`Overworld_LoadOverlord`). This code doesn't change that one, so overlords
load as before. The code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Tested by the user in Mesen2.
- Overworld only. Dungeon rooms load all their enemies when Link enters
  the room, without this bit.
- While the camera scrolls, the edge cells are checked on every frame that
  it moves, and a 16-pixel cell stays on the edge for several frames. So
  each enemy should spawn a stack of copies in the same spot as it comes
  into view, until the slots fill.
- Overworld enemies can use only slots `$0D` down to `0x00` (some types
  fewer), so there are at most 14 at once. Projectiles and other spawned
  sprites use the slots above that, and may fail to spawn when the screen
  is full.
- Killed enemies come back the next time their cell reaches the screen
  edge.
- Areas that don't scroll load their enemies once when Link enters, so
  the effect shows only in larger areas.
- Soldiers and other enemies that call for help or keep their own state
  may act oddly with copies. Not checked.

## Link swims on dry land (except grass)

**Codes:** `8124-64B3` + `2A24-6423` + `DFE2-D7FF` (use all three)
**Raw form:** `07DA2A:B6` (orig `$5B`) + `07DA2B:DC` (orig `$DE`) +
`04F4DC:01` (orig `$00`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 19:44:57

Tags: `demo-ready` `play-start`

**Mechanism:** This is the ALttP version of the SMB3 and SMW "swim through
the air" codes. When Link moves, `TileDetect_Execute` (`Bank07.asm`) jumps
through a tile type table. On the overworld it is
`TileDetection_Execute_overworld_modules`, and plain ground (type `0x00`)
goes to `TileBehavior_NothingOW` (`$DE5B`). The first two codes point
that entry at `TileBehavior_DeepWater` (`$DCB6`) instead:

```
TileBehavior_DeepWater:
    LDA.b $0A : TSB.w $0341
    RTS
```

So the game treats plain ground as deep water. Link jumps in and swims
(`$5D` = `0x04`) on it. Without the Flippers he is stopped at the edge of
deep water, so the third code gives a new file the Flippers. It changes
the `$x356` byte of the `.normal` part of `DefaultSaveFileItems`
(`Bank04.asm`), which `Bank0C.asm` copies into a new save file.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `5B DE` at `$07:DA2A` (overworld table entry `0x00`)
and `B6 DC` at `$07:DA3A` (entry `0x08`, deep water). The `.normal`
loadout starts at `$04:F4C6` (PC `$0274C6`), and its Flippers byte at
`$04:F4DC` is `$00`. The codes decode back to their raw forms with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Tested by the user in Mesen2.
- The Flippers code only works when a new file is made while it is on. On
  an old file without the Flippers, Link can't go onto the ground at all.
- Link can't use the sword, items or the A button while swimming. Turn the
  codes off to fight, lift or talk. He leaves the water on tiles that have
  other types (grass, stairs, bridges and so on).
- Grass (type `0x40`, `TileBehavior_ThickGrass`) is not changed. Link
  walks out of the water onto grass as normal.
- Only the overworld is changed. The game starts in Link's house, so the
  effect shows only once Link is outside.
- Don't use this with "Link walks on deep water". That code turns the deep
  water handler off, so these codes would do nothing.

## All ground (except grass) is slippery like ice

**Codes:** `C024-64B3` + `2A24-6423` + `C022-A4F2` (use all three)
**Raw form:** `07DA2A:A4` (orig `$5B`) + `07DA2B:DC` (orig `$DE`) +
`07D7D8:A4` (orig `$54`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 19:46:09

Tags: `demo-ready`

**Mechanism:** This is the ALttP version of the SMB2 and SMB3 "all ground
is slippery" codes. `TileDetect_Execute` (`Bank07.asm`) jumps through a
tile type table, one for the overworld
(`TileDetection_Execute_overworld_modules`) and one for dungeons and
houses (`TileDetect_Execute_modules`). Plain floor (type `0x00`) goes to
`TileBehavior_NothingOW` (`$DE5B`) outdoors and `TileBehavior_Nothing`
(`$DC54`) indoors. The codes point both entries at the Ice Palace floor
handler (type `0x0F`):

```
TileBehavior_PalaceIce:
    LDA.b $0A : ASL #4 : TSB.w $0348
    RTS
```

`TileDetect_MainHandler` then sees `$0348` set (`AND.b #$11 : BNE`) and
gives Link the ice movement (`$034A` = 2), so he slides on plain floor.
The first two codes are the overworld entry (both bytes change). The third
is the indoor entry (only the low byte changes).

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `5B DE` at `$07:DA2A` and `54 DC` at `$07:D7D8`
(entry `0x00` of each table). Entry `0x0F` is `A4 DC` in both tables
(`$07:DA48` and `$07:D7F6`), and `$07:DCA4` holds
`A5 0A 0A 0A 0A 0A 0C 48 03 60`. The codes decode back to their raw forms
with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- The effect starts in Link's house (third code). Use only the first two
  for the overworld only, or only the third for indoors only.
- Grass (type `0x40`, `TileBehavior_ThickGrass`) is not changed. Link
  walks on grass as normal.
- Only plain floor (type `0x00`) is changed. Stairs, shallow water and
  other tile types still feel normal.
- These codes change the same bytes as "Link swims on dry land (except
  grass)", so use only one of the two.

## Indoor floors are conveyor belts that push Link right

**Codes:** `7C22-A4F2` + `2322-A492` (use both)
**Raw form:** `07D7D8:3A` (orig `$54`) + `07D7D9:DE` (orig `$DC`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 19:47:05

Tags: `demo-ready`

**Mechanism:** This is the ALttP version of the SMB2 and SMB3 "all ground
is a conveyor belt" codes. `TileDetect_Execute` (`Bank07.asm`) jumps
through `TileDetect_Execute_modules` for dungeons and houses. Plain floor
(type `0x00`) goes to `TileBehavior_Nothing` (`$DC54`). The codes point
it at the rightward conveyor handler (type `0x6B`, `$DE3A`):

```
TileBehavior_Conveyor_Rightwards:
    LDA.b $0A : XBA : ASL #4
    ; Bleeds into the next function.
TileBehavior_Conveyor:
    TSB.w $03F1
```

`StartMovementCollisionChecks_Vertical` and `_Horizontal` read `$03F1`
and `$03F2` and set the conveyor direction in `$03F3` (`0x04` = right),
so plain floor carries Link like the conveyor floors in the dungeons.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `54 DC` at `$07:D7D8` (dungeon table entry
`0x00`), and entries `0x68` to `0x6B` at `$07:D8A8` are
`29 DE 2D DE 35 DE 3A DE`. The bytes at `$07:DE29` match the four
conveyor handlers in `Bank07.asm`. The codes decode back to their raw
forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Indoors only (Link's house, caves, dungeons). The overworld table has
  conveyor entries too, but the outdoor movement code
  (`StartMovementCollisionChecks_Vertical_HandleOutdoors` and
  `_Horizontal_HandleOutdoors`) never reads `$03F1`, so a code for the
  overworld was not possible this way.
- For another direction, keep `2322-A492` and use one of these in place
  of `7C22-A4F2`: up `4B22-A4F2`, down `4222-A4F2` (this pushes Link
  out the door of his house), left `7922-A4F2`.
- Only plain floor (type `0x00`) is changed. Other floor types are not
  affected.
- These codes change the same address as the indoor code of "All ground
  (except grass) is slippery like ice", so use only one of the two.

## The game runs at half speed

**Codes:** `DD67-D46D` + `DD60-DDDD` (use both)
**Raw form:** `00803A:00` (orig `$16`) + `008040:00` (orig `$03`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 19:48:19

Tags: `demo-ready`

**Mechanism:** This is the ALttP version of the Zelda 1 "half speed" code.
It uses unused debug code in the main loop (`Bank00.asm`):

```
BRA .do_frame
; UNUSED: Inaccessible code, used for debug if assembled in.
LDA.b $F6 : AND.b #$20 : BEQ .L_ButtonDown
    INC.w $0FD7
.L_ButtonDown
LDA.b $F6 : AND.b #$10 : BNE .R_ButtonDown
    LDA.w $0FD7 : AND.b #$01 : BNE .skip_frame
```

The first code sets the `BRA` offset to `$00`, so the debug code runs. The
second code sets the `BEQ .L_ButtonDown` offset to `$00`, so `INC.w
$0FD7` runs every frame, not only when L is pressed. Bit 0 of `$0FD7`
then flips every frame, and every other frame skips `Module_MainRouting`.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `80 16` at `$00:8039` and `F0 03` at `$00:803F`.
`General/WRAM.asm` lists `$0FD7` as free, and only this code uses it. The
codes decode back to their raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Everything runs at half speed, from the boot screens on. Music and sound
  effects keep their normal speed, since the sound chip runs on its own.
- A frame always runs when R is pressed, so tapping R fast gives a little
  more speed.
- These codes share `DD67-D46D` with "Time moves only while R is held",
  "L pauses the game, R steps one frame" and "Time moves only while a
  button is held". Use only one of the four.

## No music

**Code:** `4A6E-DDAD`
**Raw form:** `0080F3:2C` (orig `$8D`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 19:49:22

Tags: `demo-ready`

**Mechanism:** This is the ALttP version of the Zelda 1, SMB3 and Metroid
"no music" codes. Game code asks for a song by writing it to `$012C`. Each
NMI (`Bank00.asm`) sends a new request to the sound chip:

```
; Check if we are already playing the same song:
CMP.w $0133 : BEQ .sameSong
    ; The song has changed, tell the APU.
    STA.w SNES.APUIOPort0 : STA.w $0133
```

The code changes the first `STA` (`$8D`) to `BIT` (`$2C`). `BIT` only reads
`$2140`, so the sound chip never gets the song number. `$0133` and `$0130`
still get it, so the game's own checks of the current song work as before.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `CD 33 01 F0 10 8D 40 21 8D 33 01` at `$00:80EE`; the
`8D` is at `$00:80F3`. The song bank uploads write `#$FF` straight to
`SNES.APUIOPort0` in `Bank02.asm` and `Bank09.asm`, not through `$012C`,
so they still work. Sound effects use ports 1 to 3 and are not changed.
The code decodes back to its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- All songs are cut, including short ones like the fanfares for pendants
  and crystals. Fade and volume commands (`$F1` to `$F3`) are cut too, but
  with no song playing they do nothing.
- Sound effects, the rain and other ambient sounds still play.
- If the code is turned on while a song is playing, that song keeps
  playing, since stop and fade commands are cut too. It ends when the
  game loads a new song bank (for example when Link goes between the
  overworld and a dungeon).

## No low-health beep

**Code:** `6D25-D475`
**Raw form:** `0DDC78:80` (orig `$D0`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 19:50:29

**Mechanism:** This is the ALttP version of the Zelda 1 and Metroid "no
low-health beep" codes. The HUD update in `Bank0D.asm` plays sound `$2B`
every `$20` frames when Link's hearts (`$7EF36D`) are below the
`HeartBeepThresholds` entry for his heart containers:

```
; A frozen Link is an impervious Link, so don't beep.
LDA.w $02E4 : BNE .doneWithWarningBeep
    ; If heart refill is in process, we don't beep.
    LDA.l $7EF372 : BNE .doneWithWarningBeep
        LDA.l $7EF36C : LSR #3 : TAX
        LDA.l $7EF36D : CMP.w HeartBeepThresholds, X : BCS .doneWithWarningBeep
            ...
                    LDA.b #$20 : STA.w $04CA
                    LDA.b #$2B : STA.w $012E
```

The code changes the first `BNE` (`$D0`) to `BRA` (`$80`), so the beep
code is always skipped.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `AD E4 02 D0 2E AF 72 F3 7E D0 28 AF 6C F3 7E 4A 4A 4A`
at `$0D:DC75` (PC `$06DC75`); the `D0` is at `$0D:DC78`. The branch target,
`.doneWithWarningBeep`, is where the heart refill code starts, so the
rest of the HUD update still runs. `$04CA` (the beep timer) is used only
here. The code decodes back to its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Only the beep is removed. Hearts, refills and the other sounds work as
  before.

## Every bush, pot and rock gives a fairy

**Codes:** `DD61-6726` + `4365-6D96` + `6F65-6DB6` (use all three)
**Raw form:** `06826F:00` (orig `$F2`) + `068271:2E` (orig `$9C`) +
`068272:81` (orig `$0B`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 19:52:19

Tags: `demo-ready` `video-ready` `curated` `play-start`

**Optional, full magic instead:** `DD61-6726` + `BD65-6DB6`
**Optional, 1 bomb instead:** `DD61-6726` + `8065-6DB6`
**Optional, a heart instead:** `DD61-6726` + `2365-6DB6`
**Optional, 5 arrows instead:** `DD61-6726` + `A865-6DB6`
**Optional, a cucco instead:** `DD61-6726` + `6665-6DB6`
**Optional, a blue rupee instead:** `DD61-6726` + `DB65-6D96` + `6D65-6DB6`

**Mechanism:** When Link lifts, cuts or smashes a bush, pot or rock,
`Sprite_SpawnThrowableTerrain` and `Sprite_SpawnImmediatelySmashedTerrain`
(`Bank06.asm`) call `Sprite_SpawnSecret`:

```
LDA.b $1B : BNE .indoors
    JSL.l GetRandomInt : AND.b #$08 : BNE Sprite_SpawnSecret_fastexit
.indoors
LDY.w $0B9C : BEQ Sprite_SpawnSecret_fastexit
    ...
    LDA.w Pool_Sprite_SpawnSecret_ID-1, Y : BEQ Sprite_SpawnSecret_fastexit
        JSL.l Sprite_SpawnDynamically
```

`$0B9C` is the secret type that the overworld and dungeon secret data set
for that spot (0 means nothing). Outdoors, half of all secrets are also
dropped at random. The first code sets the `BNE` offset to `$00`, so the
random drop never happens. The other two codes change the `LDY.w $0B9C`
operand to `$812E`. The data bank here is `$06`, and `$06:812E` holds
`$14`, the secret type for a fairy (`Pool_Sprite_SpawnSecret_ID` entry
`0x14`, sprite `$E3`). So `Y` is always `0x14`, and every bush, pot and
rock spawns a fairy. The optional versions change only the high byte of
the operand, to a `$XX9C` address in bank `$06` that holds another secret
type: `$909C` = `$0D` full magic, `$B49C` = `$05` bomb, `$DE9C` = `$06`
heart, `$CB9C` = `$09` 5 arrows, `$889C` = `$0E` cucco. The blue
rupee version points the operand at `$8009`, which holds `$07`.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A5 1B D0 08 22 71 BA 0D 29 08 D0 F2 AC 9C 0B F0 ED`
at `$06:8264` (PC `$030264`). The bytes read at `$06:812E`, `$06:909C`,
`$06:B49C`, `$06:DE9C`, `$06:CB9C` and `$06:889C` are `$14`, `$0D`, `$05`,
`$06`, `$09` and `$0E`, and `$06:8009` holds `$07`. Both terrain routines start with `PHB : PHK : PLB`,
so the data bank is `$06`. A fairy or blue rupee type byte at a `$XX9C`
address does not exist in bank `$06`, so the fairy needs both operand
bytes. The codes decode back to their raw forms with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Tested by the user in Mesen2, with the cucco codes instead of the fairy
  ones.
- The magic powder secret (type `0x04`, a random pick) and the planned
  secrets of each spot are replaced too. Spots that hide a hole or a
  staircase (`$0B9C` = `$FF`) still show it and give nothing.
- The game has room for only so many sprites. When too many are out, the
  item doesn't appear.
- Thrown bushes and pots that carry their own secret
  (`ThrowableScenery_TransmuteToDebris`) also give the fairy.
- Dungeon pots that hold a key or a switch may give the fairy instead.
  Rooms that need that key might then be locked. Turn the code off in
  dungeons if that happens. Not checked.

## Lift every rock without the Power Glove

**Code:** `DD27-D4F2`
**Raw form:** `07D438:00` (orig `$02`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 19:53:45

Tags: `demo-ready` `curated` `play-start`

**Mechanism:** When Link faces something he can lift, `Link_HandleAPress`'s
helper in `Bank07.asm` compares the object's glove level
(`Liftable_GloveLevels`: `$00` sign, bush; `$01` gray rocks; `$02` black
rocks) with the gloves in `$7EF354` (0 none, 1 Power Glove, 2 Titan's
Mitt):

```
; Subtract glove strength.
LDA.w Liftable_GloveLevels, Y : SEC : SBC.l $7EF354 : BEQ .strongEnough
    BPL .checkIfOpeningChest
.strongEnough
; Current action is picking something up.
LDX.b #$01
```

The code sets the `BPL` offset to `$00`, so the branch goes to the next
instruction, `.strongEnough`, either way. Link can lift every gray and
black rock, small or big, from the start.

An earlier version of this entry (`DD27-D422`) wrote to `$07:D43B`, the
`LDA` opcode after `LDX.b #$01`. That made a `BRK`, which crashed the game
with a buzzer and a black screen.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `B9 75 D3 38 EF 54 F3 7E F0 02 10 02 A2 01` at
`$07:D42D` (PC `$03D42D`); the `BPL` offset is at `$07:D438`.
`Liftable_GloveLevels` at `$07:D375` is `00 01 00 00 02 01 02`. This is the
only place in Link's code that compares against `$7EF354`. The other
reads in `Bank0D.asm` and `Bank1B.asm` are for the glove's color in the
menu and on Link, and one is in `Sprites/sprite_rupee_crab.asm` (below). The code decodes back
to its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Tested by the user in Mesen2.
- The item menu still shows no gloves, and Link's hands keep their normal
  color.
- Lifting rocks early can open paths and secrets earlier than the game
  expects, for example under the rocks near Link's house. Some of these
  may lead to places Link can't leave without other items.
- The rock that hides a rupee crab (hoarder, sprite `$3E`) is a sprite
  with its own check (`LDA.l $7EF354 : CMP.b #$01 : BCC`), so it still
  needs the Power Glove.

## Start a new game with BAGE debug loadout (almost every item)

**Code:** `DD2F-DF57`
**Raw form:** `0CDC15:00` (orig `$3C`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 19:55:31

Tags: `demo-ready`

**Mechanism:** This is the ALttP version of the Zelda 1 and Metroid "start
with every item" codes. When a new file is made, `Bank0C.asm` copies 60
bytes of `DefaultSaveFileItems` (`Bank04.asm`) to `$x340`-`$x37B` of the
save file:

```
; Branch if it's not the first save game slot.
LDY.w #$003C : CPX.w #$0000 : BNE .loadInitialEquipment
    ...
.loadInitialEquipment
    ; Setup initial equipment and flags values.
    LDA.w DefaultSaveFileItems, Y : STA.l $700340, X
```

`Y` = `$3C` picks the `.normal` loadout (no items, 3 hearts).
`DefaultSaveFileItems` also has a debug loadout at offset 0 (`.bage`),
which the game uses only for a file named "B..." in the first slot with
the second controller turned on in the code. The code changes `#$003C` to
`#$0000`, so every new file gets the debug loadout.

The debug loadout has: the bow, the blue boomerang, the hookshot, 50
bombs, the magic powder, the fire and ice rods, Bombos, Ether, Quake, the
Lamp, the hammer, the working flute, the bug net, the Book of Mudora, the
Cane of Somaria, the Cane of Byrna, the Magic Cape, the Magic Mirror, the
Titan's Mitt, the Pegasus Boots, the Flippers, the Moon Pearl, the
Fighter's Sword and Shield, four bottles (fairy, red, green and blue
potion), 255 rupees, the Hyrule Castle big key, 15 hearts, full magic,
the largest bomb and arrow bags, 50 arrows, and every ability.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A0 3C 00 E0 00 00 D0 2D` at `$0C:DC14` (PC
`$065C14`). The debug loadout at `$04:F48A` starts `01 01 01 32 02 01 01 01`,
and the normal one at `$04:F4C6` is all `$00` except `18 18` (hearts) and
`F8` (abilities). The rest of the debug start (the game state after
Zelda's rescue, `$x3C5` = `$02`) is set in the skipped `BNE` branch, not by
the copy loop, so the story starts as usual. The code decodes back to its
raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Only files made while the code is on get the items. Turn it off after
  the file is made, or every later new file gets them too (which is
  harmless).
- Missing compared to a finished game: the Master Sword and stronger
  swords, the Mirror Shield, better mail, the Silver Arrows, the red
  boomerang, the shovel, the mushroom, the Half Magic upgrade, heart
  pieces, pendants and crystals. The bow is stored as "bow without
  arrows" (`$01`) even though the file has 50 arrows; it may show as empty
  until Link picks up arrows. Not checked.
- The story still starts in Link's bed, and his uncle still gives him the
  sword. Having every item from the start lets Link skip much of the game
  and reach places in the wrong order.

## Start a new game with the Lamp

**Code:** `DFE2-DDFF`
**Raw form:** `04F4D0:01` (orig `$00`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 19:57:31

**Mechanism:** This is one of the ALttP versions of the Zelda 1
"start with" codes. When a new file is made, `Bank0C.asm` copies the
`.normal` loadout of `DefaultSaveFileItems` (`Bank04.asm`) into the save
file (`LDA.w DefaultSaveFileItems, Y : STA.l $700340, X`, with `Y` from
`$3C`). The code changes its `$x34A` byte (`db $00 ; $x34A - no lamp`) to
`$01`, the Lamp.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has the `.normal` loadout at `$04:F4C6` (PC `$0274C6`);
its 11th byte, at `$04:F4D0`, is `$00`. `Link_ReceiveItem`'s
`.item_target_addr` entry for the Lamp is `$F34A` with value `$01`. The
code decodes back to its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Only files made while the code is on get the Lamp.
- The Lamp is in the chest in Link's house too, and the chest still gives
  it. Opening the chest doesn't take anything away.
- Lighting torches with the Lamp costs magic, and the `.normal` loadout
  has 0 magic power (`$x36E`). Link may need to find magic before the
  Lamp can light torches. Not checked.

## Turn on the controller 2 debug features

**Code:** `3C6E-A4DD`
**Raw form:** `0083F8:EA` (orig `$60`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 19:58:41

Tags: `demo-ready` `curated` `play-start`

**Mechanism:** The joypad routine in `Bank00.asm` reads controller 1 and then
bleeds into `Player2JoypadReturn`, which starts with an `RTS`:

```
Player2JoypadReturn:
    RTS
    LDA.w SNES.JoyPad2DataLow  : STA.b $00
    LDA.w SNES.JoyPad2DataHigh : STA.b $01
    ...
```

So controller 2 is never read, and `$F1`, `$F3`, `$F5` and `$F7` stay 0.
The code changes the `RTS` to `NOP`, so these are set every frame. The
game then turns on these debug features:

- `LinkState_Default` and `LinkState_Bunny` (`Bank07.asm`): B on
  controller 2 (`$F5` bit 7) flips `$037F`. `TileDetect_Execute` then
  treats every tile as floor, so Link walks through walls. The Magic
  Mirror also skips its indoor and Dark World checks (`Bank07.asm`).
- New file setup (`Bank0C.asm`): it checks its own code with
  `LDA.l Player2JoypadReturn : AND.w #$00FF : CMP.w #$0060`. With the
  `NOP`, a new file in slot 1 whose name starts with "B" gets the
  `DefaultSaveFileItems_bage` items, the story set to after Zelda is
  rescued, and Link starting in his house.
- The item menu (`Bank0D.asm`): R on controller 2 (`$F7` bit 4) deletes
  the selected item.
- `EasterEgg_BageCode` (`Bank04.asm`): R held and A on controller 1
  upgrades the sword, armor and shield. Nothing calls this routine in the
  normal game, so this part needs "Debug: Name save file 1 "BAGE",
  in-game hold R and press A, B, or Y" (`3C6A-AD96`) too.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `60 AD 1A 42 85 00 AD 1B` at `$00:83F8`
(`Player2JoypadReturn`, PC `$0003F8`). `$F1`, `$F3`, `$F5` and `$F7` are
read only at the places above. The code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Tested by the user in Mesen2.
- A second controller must be plugged in for the button features. With
  none, its buttons read as 0 and nothing happens.
- The "B" name items work only when the file is made while the code is
  on. "Start a new game with BAGE debug loadout (almost every item)"
  (`DD2F-DF57`) gives the same items to any name without this code.
- B on controller 2 is a toggle. Press it again to turn walls back on.
  Link can walk out of the map or fall into the void with it on.
- A second player pressing R on controller 2 in the item menu deletes
  items for good.

## Debug: Name save file 1 "BAGE", in-game hold R and press A, B, or Y

**Code:** `3C6A-AD96`
**Raw form:** `0683C1:EA` (orig `$6B`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 19:58:41

Tags: `demo-ready` `curated` `play-start`

**Mechanism:** `EasterEgg_BageCode` (`Bank04.asm`) is a debug routine that
nothing calls in the normal game. The only call to it is
`EasterEgg_BageCodeTrampoline` (`Bank06.asm`), and nothing calls that
either. But the trampoline comes right after the end of `Sprite_Main`:

```
    LDA.w $0AAA : BEQ .iota
        STA.w $0FC6
    .iota
    RTL                         ; $06:83C1

EasterEgg_BageCodeTrampoline:
    JSL.l EasterEgg_BageCode    ; $06:83C2
    RTL
```

The code changes `Sprite_Main`'s `RTL` to `NOP`, so it runs into the
trampoline and calls `EasterEgg_BageCode` on every frame that sprites
run. `A` and `X` are 8-bit there, as the routine expects. If the name in
save slot 1 (`$7003D9`) is "BAGE" and R is held on controller 1 (`$F2`
bit 4), the routine:

- sets magic use to half (`$7EF37B`), for good;
- on A (`$F6` bit 7), steps up the sword, shield and armor
  (`BAGE_HandleSword`);
- on B (`$F4` bit 7), flips the walk-through-walls flag `$037F`;
- on Y (`$F4` bit 6), fills hearts, magic, bombs and arrows, adds 255
  rupees and sets the keys to 9.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `AD AA 0A F0 03 8D C6 0F 6B 22 9D EA 04 6B` at
`$06:83B9` (PC `$0303B9`). `6B` (`RTL`) is at `$06:83C1`, and `22 9D EA 04`
(`JSL $04EA9D`, `EasterEgg_BageCode`, PC `$026A9D`) follows it. That
`JSL` appears nowhere else in the ROM. The Y check is `24 F4 50 2B`
(`BIT $F4 : BVC`), so the refill happens on the frame Y is pressed. (The
source in `Bank04.asm` writes `BVS` here, but the ROM has `BVC`.) The code
decodes back to its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Tested by the user in Mesen2.
- The file in slot 1 must be named BAGE, and the name is read from save
  RAM, so it must be the file being played. Other slots and names do
  nothing.
- R isn't used for anything else in the US game, so holding it is safe.
- The sword steps through 1 to 4 and back to 1, so it can go down too.
- With "Turn on the controller 2 debug features" (`3C6E-A4DD`), R held
  and A on controller 2 also steps up the sword, armor and shield, for
  any name.
- `Sprite_Main` runs only during play, so the cheats don't work in menus
  or during screen changes.

## L pauses the game, R steps one frame

**Code:** `DD67-D46D`
**Raw form:** `00803A:00` (orig `$16`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 19:58:41

Tags: `demo-ready` `curated` `play-start`

**Mechanism:** The main loop in `Vector_Reset` (`Bank00.asm`) has a debug
frame-step feature. A `BRA .do_frame` jumps over it, so it never runs:

```
.mainGameLoop
    LDA.b $12 : BEQ .NMI_wait_loop
    CLI
    BRA .do_frame

    LDA.b $F6 : AND.b #$20 : BEQ .L_ButtonDown
        INC.w $0FD7
    .L_ButtonDown
    LDA.b $F6 : AND.b #$10 : BNE .R_ButtonDown
        LDA.w $0FD7 : AND.b #$01 : BNE .skip_frame
    .R_ButtonDown
.do_frame
    INC.b $1A
    JSR.w ClearOamBuffer
    JSL.l Module_MainRouting
.skip_frame
```

The code sets the `BRA` offset to `$00`, so the debug code runs. `$F6`
holds the A, X, L and R buttons pressed this frame on controller 1. Each
press of L adds 1 to `$0FD7`. While `$0FD7` is odd, the game skips
`Module_MainRouting`, so nothing moves. On a frame when R is pressed, the
game runs one frame anyway. NMI still runs, so the screen stays up and the
buttons are still read.

An earlier version of this entry (`DD67-D4AD`) wrote to `$00:803B`, the
`LDA` opcode after the `BRA`. That made a `BRK`, which crashed the game.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A5 12 F0 FC 58 80 16 A5 F6 29 20 F0 03 EE D7 0F A5
F6 29 10 D0 07 AD D7 0F 29 01 D0 09 E6 1A` at `$00:8034` (PC `$000034`).
`80 16` is at `$00:8039`; `$803B + $16 = $8051` is `.do_frame`
(`E6 1A`). `$0FD7` is used nowhere else (`General/WRAM.asm` lists it as
free), and `Startup_InitializeMemory` clears it at power-on, so the game
starts unpaused. The code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Tested by the user in Mesen2.
- The L and R buttons aren't used for anything else in the US game.
- The pause works everywhere, including the title screen, menus and
  screen changes.
- Music and sound keep playing while paused, since the sound chip runs
  on its own.
- Holding R doesn't run frames; each new press of R runs one frame.
- This is the first code of "The game runs at half speed", "Time moves
  only while R is held" and "Time moves only while a button is held".
  Those cheats already include it, so use only one of the four.

## Start a new game with the Pegasus Boots

**Code:** `DFE2-D42F` + `EAEE-D72F`
**Raw form:** `04F4DB:01` (orig `$00`), `04F4FF:FC` (orig `$F8`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 19:57:31

**Mechanism:** When a new file is made, `Bank0C.asm` copies the `.normal`
loadout of `DefaultSaveFileItems` (`Bank04.asm`) into the save file. The
first code changes its `$x355` byte (`db $00 ; $x355 - no boots`) to `$01`.
Dashing also needs the Run bit (`$04`) in the ability flags at `$x379`,
which the game normally sets when Link gets the boots
(`ancilla_init.asm`: `LDA.b #$04 ... ORA.l $7EF379 : STA.l $7EF379`). The
second code changes the loadout's `$x379` byte from `$F8` to `$FC`, which
adds that bit.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has the `.normal` loadout at `$04:F4C6` (PC `$0274C6`).
Its `$x355` byte, at `$04:F4DB`, is `$00`, and its `$x379` byte, at
`$04:F4FF`, is `$F8`. `Link_AbilityChecks` in `Bank07.asm` lists `$04` as
Run, and the dash check reads it with `AND.l $7EF379`. Both codes decode
back to their raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Only files made while the codes are on get the boots.
- Without `EAEE-D72F` the boots show in the menu, but Link can't dash.
- Sahasrahla still gives the boots later. Getting them again doesn't take
  anything away.

## Start a new game with the Power Glove

**Code:** `DFE2-D4BF`
**Raw form:** `04F4DA:01` (orig `$00`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 19:57:31

**Optional, the Titan's Mitt instead:** `D4E2-D4BF`

**Mechanism:** When a new file is made, `Bank0C.asm` copies the `.normal`
loadout of `DefaultSaveFileItems` (`Bank04.asm`) into the save file. The
code changes its `$x354` byte (`db $00 ; $x354 - lift 1`) to `$01`, the
Power Glove. The optional code writes `$02`, the Titan's Mitt.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has the `.normal` loadout at `$04:F4C6` (PC `$0274C6`);
its `$x354` byte, at `$04:F4DA`, is `$00`. The rock check in `Bank07.asm`
compares a rock's level with the glove level
(`LDA.w Liftable_GloveLevels, Y : SEC : SBC.l $7EF354`). The code decodes
back to its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Only files made while the code is on get the glove.
- The glove is still in its Desert Palace chest. Getting it there doesn't
  take anything away.
- To lift every rock on any file, see "Lift every rock without the Power
  Glove".

## Link's uncle gives the Master Sword

**Code:** `D466-0D49`
**Raw form:** `098580:02` (orig `$01`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 19:57:31

Tags: `demo-ready`

**Optional, the Tempered Sword and Mirror Shield instead:** `D766-0D49`

**Mechanism:** This is the ALttP version of the Zelda 1 "start with a
better sword" codes. A new file starts with no sword, and Link can't swing
one until his uncle sets `$7EF3C5` to 1, so the code changes the uncle's
gift instead of the new-file loadout. `Uncle_GiveSwordAndShield` gives item
0x00 (`LDY.b #$00 : JSL.l Link_ReceiveItem`). For item 0x00,
`Link_ReceiveItem` (`ancilla_init.asm`) writes `.item_values[0]` to both
the shield (`$F35A`) and the sword (`$F359`). The code changes that value
from 1 to 2, so Link gets the Master Sword and the Fire Shield. The
optional code writes 3: the Tempered Sword and the Mirror Shield.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `.item_values` at `$09:8580` (PC `$048580`), starting
`01 02 03 04`. `.item_target_addr` starts `$F359, $F359, $F359, $F359,
$F35A`: items 0x00-0x03 are swords 1-4 and item 0x04 is the first shield.
The extra write for item 0x00 uses `LDX.b #$08`, the shield's address.
Both codes decode back to their raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Only item 0x00 uses this value, and only the uncle gives item 0x00 (per
  `Uncle_GiveSwordAndShield`; other callers not checked). It takes effect
  when the uncle gives his gift, so an existing file that has already met
  him is not changed.
- The item Link holds up may still look like the Fighter's Sword. Not
  checked.
- Pulling the Master Sword later writes 2 to the sword. With the optional
  code, that turns the Tempered Sword back into the Master Sword.

## Dark rooms are always lit

**Code:** `DD8F-6DA9`
**Raw form:** `01B613:00` (orig `$01`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 20:18:33

**Mechanism:** Bit 0 of a dungeon room's first header byte is its "lights
out" flag. The room loader in `Bank01.asm` copies it to `$7EC005`:

```
; Save whether to turn the lights out or not.
LDA.l $7EC005 : STA.l $7EC006

LDA.b [$0D], Y : AND.b #$01 : STA.l $7EC005
```

The code changes `AND.b #$01` to `AND.b #$00`, so `$7EC005` is always 0.
The palette dimming (`Dungeon_FadedFilter` and the other
`LDA.l $7EC005 : ORA.l $7EC006` checks in `Bank02.asm`), the Lamp's light
cone (`RestoreTorchBackground` in `Bank0D.asm`) and the torch light levels
(`$045A` in `Bank01.asm`) all skip their work when it is 0. The game does
the same thing itself in Link's house: once Link has his gear, it writes
`LDA.w #$0000 : STA.l $7EC005` there.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `B7 0D 29 01 8F 05 C0 7E` at `$01:B610` (PC
`$00B610`), the only place these bytes occur. The `$01` at `$01:B613` is
the `AND` operand. The code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Dungeons and houses only. The overworld has no dark rooms.
- Lighting torches no longer brightens a room, since every room is
  already lit. Rooms that open a door or chest when all torches are lit
  use a separate check, which this code doesn't change. Not checked.
- The Lamp (or the Fire Rod) is still needed to light torches.

## A heart refills all of Link's hearts

**Code:** `CD28-0DF6`
**Raw form:** `06D1B0:A0` (orig `$08`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 20:19:46

**Mechanism:** This is the ALttP version of the Metroid code that makes
energy pickups fill all of Samus's energy. When Link touches a heart,
`HeartRefill_AbsorptionByPlayer` (`Sprites/sprite_absorbable.asm`) adds
`$08` (one heart) to the refill counter at `$7EF372`:

```
; Amount of hearts a standard heart refills.
LDA.b #$08

; $0351B1 ALTERNATE ENTRY POINT
.apply_refill_amount

CLC : ADC.l $7EF372 : STA.l $7EF372
```

The code changes `$08` to `$A0`, 20 hearts. That is the value the big
fairy and the potion shop use to heal Link fully. The refill loop in
`Bank0D.asm` adds one heart at a time and clears `$7EF372` as soon as
health is full, so the extra amount never goes past Link's heart
containers.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A9 08 18 6F 72 F3 7E` at `$06:D1AF` (PC
`$0351AF`); the `$08` is at `$06:D1B0`. The fairy's own amount
(`LDA.b #$38`) is just before it and isn't changed. The code decodes
back to its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Hearts still fill one at a time with the usual sound, so a full refill
  takes a moment.
- Only hearts that drop on the ground use this code. A heart Link gets as
  an item (item 0x42) and apples use their own `ADC.b #$08`, so they
  still give one heart.
- `$7EF372` is one byte. Picking up a second heart while 12 or more
  hearts of refill are still pending (`$60` + `$A0` = `$100`) wraps the
  counter and gives less.

## Items are collected anywhere on the screen

**Code:** `DD2F-0726`
**Raw form:** `06D11F:00` (orig `$04`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 20:20:59

Tags: `demo-ready` `video-ready` `curated` `play-start`

**Mechanism:** This is the ALttP version of the Zelda 1 code that lets
Link collect items from anywhere. Rupees, hearts, bombs, arrows, magic
jars, keys, big keys and the shield pickup all call
`Sprite_CheckAbsorptionByPlayer` (`Sprites/sprite_absorbable.asm`):

```
LDA.w $0F10, X : BNE .sprite_is_paused
    JSR.w Sprite_CheckDamageToPlayer_stagger : BCC .no_player_collision
        JSL.l Sprite_HandleAbsorptionByPlayerLong
```

The code changes the `BCC` offset from `$04` to `$00`, so the branch
lands on the next instruction whether Link touches the item or not. Each
item is picked up on the first frame its `$0F10` delay is 0.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `BD 10 0F D0 09 20 4A F1 90 04 22 25 D1 06 60` at
`$06:D116` (PC `$035116`). The `90 04` is at `$06:D11E`, so the
offset is at `$06:D11F`. Hearts (`Sprites/sprite_heart_refill.asm`)
use the same routine. The code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Tested by the user in Mesen2.
- Items are collected only while the game runs their sprites, which is
  roughly while they are on or near the screen.
- Keys and big keys lying in a room are collected as soon as Link
  enters. The big key still plays the hold-up-the-item scene.
- Fairies use their own code (`Sprites/sprite_fairy.asm`), so they still
  have to be touched or caught.
- Items dropped by enemies or found under bushes are collected as soon
  as their `$0F10` delay ends. How long that delay is was not checked.

## Dropped items never vanish

**Code:** `6D2E-DDF6`
**Raw form:** `06D0F0:80` (orig `$F0`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 20:21:42

**Mechanism:** This is the ALttP version of the SMW code that keeps
power-ups from disappearing. Items that enemies, bushes and pots drop get
a timer of `$FF` in `$0B58` (`LDA.b #$FF : STA.w $0B58, Y` in
`Bank06.asm` and other places). Each frame,
`Sprite_HandleBlinkingPhaseOut` (`Sprites/sprite_absorbable.asm`)
counts it down, makes the item blink near the end, and removes the item
(`STZ.w $0DD0, X`) at 0. It starts:

```
LDA.w $0B58, X : BEQ .phase_out_not_scheduled
```

The code changes that `BEQ` to `BRA`, so the routine always skips to
its `RTS` and the timer never runs.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `BD 58 0B F0 23` at `$06:D0ED` (PC `$0350ED`);
the `F0` is at `$06:D0F0`. `$80` is `BRA` with the same offset. The
routine is called only from `Sprite_DrawTransientAbsorbable`, which
rupees, refills, hearts and fairies use. The code decodes back to its raw
form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Dropped fairies stay forever too.
- Items probably still go away when Link leaves the screen or room, as
  other sprites do. Not checked.
- Items that stay use sprite slots, so a screen full of drops may leave
  fewer slots for enemies or new drops. Not checked.

## Soldiers chase Link at double speed

**Codes:** `ACA4-0D99` + `B6A4-0DB9` (use both)
**Raw form:** `05C521:CA` (orig `$66`) + `05C522:98` (orig `$C5`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 20:25:20

Tags: `demo-ready`

**Optional, also patrol at double speed:** `FDA7-AD90` + `EDA7-ADB0` + `FDA7-AF20` + `EDA7-A4F0`

**Mechanism:** Once a soldier spots Link, `Guard_InPursuit_delay`
(`Bank05.asm`) aims it at him with `Sprite_ApplySpeedTowardsPlayerLong`.
The speed comes from `AppliedSpeed16`, four bytes of `$10` (1 pixel a
frame) picked by sprite slot:

```
TXA : AND.b #$03 : TAY
...
LDA.w AppliedSpeed16, X
JSL.l Sprite_ApplySpeedTowardsPlayerLong
```

The source says `, X`, but the ROM has `B9 66 C5`, `LDA $C566, Y`.
The codes point that load at `$05:98CA`, four bytes of `$20` in the
Zora data (`$02983F-$0298F4 DATA` in
`Sprites/sprite_zora_and_fireball.asm`), so soldiers chase at 2 pixels a
frame. The optional codes change the patrol speeds in `Pool_Soldier`
(`.x_speeds` `08 F8 00 00` and `.y_speeds` `00 00 08 F8`) from
`$08`/`$F8` to `$10`/`$F0`.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `B9 66 C5` at `$05:C520` (PC `$02C520`),
`10 10 10 10` at `AppliedSpeed16` (`$05:C566`), `20 20 20 20` at
`$05:98CA`, and `08 F8 00 00 00 00 08 F8` at `$05:C331`. All six codes
decode back to their raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- This covers the sprites that use `Sprite_Soldier`: the green, blue and
  red spear soldiers (0x41-0x43). Archers, javelin soldiers, the crazed
  soldiers and the ball and chain trooper have their own code.
- With the optional codes this is six codes, one more than a real Game
  Genie holds.
- `Pool_Soldier` is also `Pool_Probe`, so the optional codes may also
  speed up the invisible probes soldiers send out to look for Link. Not
  checked.

## Snakes move four times as fast

**Codes:** `4DC7-6DF1` + `3DC7-6D91` + `4DC7-64B1` + `3DC7-6421` (use all four)
**Raw form:** `05AA30:20` (orig `$08`) + `05AA31:E0` (orig `$F8`) +
`05AA3A:20` (orig `$08`) + `05AA3B:E0` (orig `$F8`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 20:26:02

**Optional, also lunge three times as fast:** `7DC7-6FF1` + `2DC7-6F91` + `7DC7-67B1` + `2DC7-6721`

**Mechanism:** The snakes are ropes, sprite 0x6E
(`Sprites/sprite_rope.asm`). `Rope_Moving` reads their speed from
`Pool_Rope_Moving`:

```
.x_speeds
db $08, $F8, $00, $00
db $10, $F0, $00, $00

.y_speeds
db $00, $00, $08, $F8
db $00, $00, $10, $F0
```

The first row of each table is the wandering speed, half a pixel a frame.
When a rope picks a new direction while Link is lined up with it, it sets
`$0ED0` to 4 and uses the second row to lunge at him. The codes change
the wandering speeds to `$20`/`$E0`, 2 pixels a frame. The optional
codes change the lunge speeds to `$30`/`$D0`.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `08 F8 00 00 10 F0 00 00 00 00 08 F8 00 00 10 F0` at
`$05:AA30` (PC `$02AA30`). All eight codes decode back to their raw
forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Without the optional codes, ropes wander faster than they lunge.
- With the optional codes this is eight codes, more than a real Game
  Genie holds.
- Skullropes use the same sprite, so they are faster too.

## Soldiers move faster

**Codes:** `DCED-6491` + `3CEF-6FF1` (use both)
**Raw form:** `05FA09:0A` (orig `$18`) + `05FA14:EA` (orig `$4A`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 20:27:41

Tags: `demo-ready` `challenge`

**Optional, Octoroks, Moblins and the other bank 06 sprites too:** `DC30-0F2C` + `3C39-0DBC`
**Optional, every sprite in banks 1D and 1E too:** `DC31-0981` + `3C35-0051` + `DCE7-A558` + `3CE0-A178`

**Mechanism:** Sprites move with `Sprite_MoveVert` (`Bank06.asm`);
`Sprite_MoveHoriz` calls it with `X` moved up by `$10`, so it also
does X. A speed is a signed byte in sixteenths of a pixel. The low
nibble goes into the subpixel and the high nibble into the position:

```
LDA.w $0D40, X : BEQ .return
    ASL #4 : CLC : ADC.w $0D60, X : STA.w $0D60, X

    LDA.w $0D40, X : PHP
    LSR #4
    LDY.b #$00 : PLP : BPL .positive
        ORA.b #$F0
```

The codes turn the `CLC` into a fifth `ASL` and the first `LSR` into a
`NOP`, so the speed is read in eighths of a pixel instead. Every sprite
that uses the patched copy moves twice as far each frame, in every
direction. The `ORA.b #$F0` still sign-extends a speed shifted right
three times, and the `PLP` restores the carry from the subpixel add, so
nothing else changes. The `CLC` is gone, so the fifth `ASL` can add up
to 1/256 of a pixel.

Each sprite bank has its own copy of this routine. The main codes patch
the bank 05 copy (`Sprite2_MoveVert`), which `Sprite_Soldier` and the
other bank 05 sprites use through `Sprite2_Move`. Bank 05 runs:

- Soldiers of every kind: blue, green and red spear soldiers,
  archers, the soldiers in bushes, bomb, flail and cannon troopers, and
  the recruits
- Ropes (snakes), Keese, rats, Armos, walking Zoras, Beamos, sparks,
  spike rollers, Lanmolas and the Armos Knights
- Some NPCs and items, such as the running man, the Lost Woods animals,
  heart pieces and the Pegasus Boots dash items

The first optional code pair patches the bank 06 copy
(`Sprite_MoveVert`). Bank 06 runs Octoroks (and their rocks,
Octoballoons and baby Octoroks), Moblins, enemy arrows, Buzzblobs,
Hinoxes, small Helmasaurs, Baris, Deadrocks, Poes, Leevers, mini
Moldorms, Snapdragons, Ropas, Sluggulas, Bubbles, rupee crabs, cuccos,
dropped items and heart refills, the smithy brothers, the flute boy,
the hobo, the storyteller NPCs and movable statues. Bank 06 also runs
the states any sprite can be in: stunned or frozen
(`SpriteStunned_Main`), drowning (`SpriteDrown_Main`) and falling into
a hole (`SpriteCustomFall_Main`). So with this pair, every stunned
enemy that is still sliding moves twice as far too.

The second optional line patches the copies in banks 1D and 1E
(`Sprite4_MoveVert`, `Sprite3_MoveVert`). These run most dungeon
enemies (Stalfos, Gibdos, Zols, Wizzrobes, Lynels, Tektites and others)
and most bosses.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has the same 46 bytes, starting `BD 40 0D F0 28 0A 0A 0A
0A 18`, at `$05:FA00`, `$06:E93E`, `$1D:E95D` and `$1E:FF34`. In each
copy the `CLC` is at +$09 and the first `LSR` at +$14. A step-by-step
model of the routine gives exactly twice the distance for speeds
`$08`, `$0C`, `$10`, `$18`, `$40`, `$7F` and their negatives. All
eight codes decode back to their raw forms with `snes_gg_encode.py`.
The sprite lists come from the `incsrc` lines in each `BankXX.asm`
and the `Sprite_Move`/`Sprite2_Move` calls in each sprite file. The soldier IDs come
from the sprite table in `Bank06.asm`, where 0x41-0x50 go to
`SpriteActive2_Trampoline` (bank 05).

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Everything in bank 05 that moves this way speeds up, not only
  soldiers (see the list above).
- With both optional lines this is eight codes, more than a real Game
  Genie holds. The main codes plus the first optional pair is four.
- Sprites that move twice as far each frame may pass through thin walls.
  Not checked.
- Sprites that set their position with their own code (some bosses) are
  not changed.
- "Soldiers chase Link at double speed" changes other bytes
  (`05C521`, `05C522`). The two can be used together. Not checked how
  fast that makes them.

## Sprites can't move

**Codes:** `6D30-0D9C` + `6DED-6D21` + `6D31-0071` + `6DE7-A9E8` (use all four)
**Raw form:** `06E941:80` + `05FA03:80` + `1DE960:80` + `1EFF37:80` (each orig `$F0`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 20:28:09

**Mechanism:** This is the ALttP version of the Zelda 1 Clock and the
Metroid codes that freeze enemies. Sprites move with a `MoveVert`
routine, one copy in each sprite bank (`Sprite_MoveVert` in `Bank06.asm`,
`Sprite2_MoveVert` in `Bank05.asm`, `Sprite4_MoveVert` in `Bank1D.asm`,
`Sprite3_MoveVert` in `Bank1E.asm`). The `MoveHoriz` routines call it
with `X` moved up by `$10`, so it does both X and Y. It starts:

```
LDA.w $0D40, X : BEQ .return
```

Each code changes one `BEQ` (`$F0`) to `BRA` (`$80`), so the routine
always goes to its `RTS` and no sprite in that bank moves by its speed.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has the same 46 bytes, starting `BD 40 0D F0 28`, at
`$06:E93E`, `$05:FA00`, `$1D:E95D` and `$1E:FF34`; the `F0` is at +3
in each. All four codes decode back to their raw forms with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Each code freezes one bank's sprites. `6D30-0D9C` (bank 06) is
  octoroks, moblins and other early enemies, and `6DED-6D21` (bank 05)
  is soldiers and ropes.
- Every sprite that moves this way stops, not just enemies: enemy shots,
  thrown pots and bushes, dropped items and NPCs. Height is moved by
  other code, so a thrown pot falls straight down.
- Enemies still turn, attack and hurt Link on touch. Knockback from
  Link's sword may not work either. Not checked.
- Sprites that set their position with their own code (some bosses) still
  move.

## Enemies face and walk away from Link

**Codes:** `7D33-6DFC` + `7D3D-AFBC` (use both)
**Raw form:** `06EAE0:30` (orig `$10`) + `06EB06:30` (orig `$10`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 20:29:18

Tags: `demo-ready` `video-ready` `curated` `play-start`

**Mechanism:** This is the ALttP version of the SMB2, SMB3 and SMW codes
that turn enemies away from the player. `Sprite_IsToRightOfPlayer` and
`Sprite_IsBelowPlayer` (`Bank06.asm`) return `Y` = 1 when the sprite is
to the right of or below Link, and `Y` = 0 otherwise. Each ends:

```
... : BPL .same_or_to_left
    INY
```

The codes change both `BPL`s to `BMI`, so each answer is reversed. The
routines that use them then work backwards: `Sprite_DirectionToFacePlayer`
gives the direction away from Link, and `Sprite_ProjectSpeedTowardsPlayer`
(with `Sprite_ApplySpeedTowardsPlayer`) gives a speed away from him,
since it uses the two `Y` values to set the sign of each axis. The other
sprite banks call these through `JSL`s to the `Long` versions, so the
codes change every sprite.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `... FD 30 0D 10 01 C8 60` at `$06:EAD1` (PC
`$036AD1`) and `... 28 69 00 10 01 C8 60` at `$06:EAE8` (PC `$036AE8`);
the `10`s are at `$06:EAE0` and `$06:EB06`. These are the only copies of
the two routines in the ROM. `Sprite2_`, `Sprite3_` and
`Sprite4_DirectionToFacePlayer` are `JSL.l Sprite_DirectionToFacePlayerLong`.
Both codes decode back to their raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Tested by the user in Mesen2.
- Enemy shots aimed at Link fly away from him.
- `Sprite_ApplyRecoilToPlayer` also uses `Sprite_ProjectSpeedTowardsPlayer`,
  so an enemy that hits Link knocks him toward itself instead of away.
- NPCs that turn to face Link turn away from him.
- Enemies that wander at random or follow set paths still do that. Only
  the ones that turn or move toward Link are reversed.

## Soldiers always chase Link

**Code:** `AFAA-64F0`
**Raw form:** `05C2C8:C1` (orig `$03`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 20:30:35

**Mechanism:** This is the ALttP version of the SMW and Zelda 1 codes that
make enemies chase the player. Soldiers (`Sprite_Soldier`, `Bank05.asm`)
pick their action from a table of modes:

```
dw Guard_Idle         ; 0x00 - $C2D4
dw Guard_OnPatrol     ; 0x01 - $C403
dw Guard_Surveying    ; 0x02 - $C490
dw Guard_NoticeKouhai ; 0x03 - $C4C1
dw Guard_InPursuit    ; 0x04 - $C4E8
```

A soldier normally goes from idle to patrol, and only goes to mode 0x03
when its probe or Link's touch finds Link. The code changes the patrol
entry to `$C4C1`, so a soldier that finishes idling runs
`Guard_NoticeKouhai` instead: it turns to face Link, waits out the idle
timer, then goes to mode 0x04 with `$0DF0` = `$FF` and chases him.
When the chase timer runs out the soldier stops and looks around (mode
0x02), goes back to idle, and starts over.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `D4 C2 03 C4 90 C4 C1 C4 E8 C4` at `$05:C2C6` (PC
`$02C2C6`). The table is used with `LDA.w .states, Y : DEC : PHA` and an
`RTS`, so it holds the routine addresses themselves. The `$03` at
`$05:C2C8` is the low byte of `Guard_OnPatrol`. The code decodes back to
its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Soldiers only: the green, blue and red spear soldiers (0x41-0x43).
  Other enemies use their own code.
- Soldiers no longer patrol, so ones that guard a path or a door leave it.
- Soldiers chase from anywhere in the room or screen, not just once they
  see Link.
- It works well with "Soldiers chase Link at double speed".

## Enemies pass through Link

**Code:** `6DE3-AD96`
**Raw form:** `06F3E1:80` (orig `$D0`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 20:31:44

Tags: `demo-ready` `video-ready` `curated` `play-start`

**Mechanism:** This is the ALttP version of the SMB3 code that keeps the
player's mercy invincibility on all the time. When an enemy or an enemy
shot touches Link, `Sprite_CheckDamageToPlayer` and
`Sprite_StaggeredCheckDamageToPlayerPlusRecoil` (`Bank06.asm`) go to
`Sprite_AttemptDamageToPlayerPlusRecoil`. It skips everything while Link
is still flashing from a hit (`$031F`) or `$037B` is set:

```
LDA.w $031F : ORA.w $037B : BNE .dont_damage_player
    LDA.b #$13 : STA.b $46
    LDA.b #$18
    JSR.w Sprite_ApplyRecoilToPlayer
    ...
    LDA.w Bump_Damage_Table, Y : STA.w $0373
```

The code changes that `BNE` to `BRA`, so it always skips. Link takes no
damage and no knockback from it.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `AD 1F 03 0D 7B 03 D0 3B` at `$06:F3DB` (PC
`$0373DB`); the `D0` is at `$06:F3E1`. This is the only place
`Bump_Damage_Table` is read. The code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Tested by the user in Mesen2.
- Other damage has its own code, which this doesn't change: spikes and
  spike floors (`Bank07.asm`), Link's own bombs (`ancilla_bomb.asm`),
  Agahnim's lightning (`garnish_lightning_trail.asm`), some attacks by
  Blind and Trinexx, and the barrier at Hyrule Castle Tower. Falling into
  pits also still costs health.
- Sprites still see that they touched Link, so octorok stones still
  break and enemies that react to touching him still do.

## Link stays invincible four times as long after a hit

**Code:** `EE68-D4BA`
**Raw form:** `0780BA:FF` (orig `$3A`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 20:32:29

**Mechanism:** This is the ALttP version of the Zelda 1 code that makes
Link's flashing after a hit last longer. When an enemy hurts Link,
`Link_ControlHandler` (`Bank07.asm`) starts the hurt timer at `$031F`:

```
LDA.w $031F : BNE .blinkingFromDamage
    LDA.b #$3A : STA.w $031F

.blinkingFromDamage

; Selects the "Link has been hurt" sound.
LDA.b #$26
```

While `$031F` isn't 0, Link flashes and
`Sprite_AttemptDamageToPlayerPlusRecoil` (`Bank06.asm`) skips all enemy
damage. `Bank0D.asm` takes 1 off it each frame. The code changes `$3A`
(58 frames, about 1 second) to `$FF` (255 frames, about 4.25 seconds).

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `AD 1F 03 D0 05 A9 3A 8D 1F 03 A9 26` at `$07:80B4`
(PC `$0380B4`); the `$3A` is at `$07:80BA`. No code tests the sign of
`$031F`, so `$FF` is safe. The code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Three other places set their own `$3A`, so the hits they handle still
  give the usual time: two in `Bank07.asm` (`$07:B8A1`, `$07:D2C1`) and
  Link's own bombs (`$08:98FC`, `ancilla_bomb.asm`).
- The timer only counts down during normal play (`$11` = 0), as before.

## Every enemy dies in one hit

**Code:** `DD35-AD28`
**Raw form:** `06EF73:00` (orig `$73`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 20:34:41

Tags: `demo-ready`

**Mechanism:** This is the ALttP version of the Metroid and Zelda 1 codes
that make enemies die in one hit. When a hit does damage, the game queues
it in `$0CE2` and starts the sprite's hit timer at `$0EF0`. On tick
`$18`, `Sprite_HandleSpecialHits` (`Bank06.asm`) subtracts it from the
sprite's health:

```
LDA.w $0E50, X : STA.b $00

; Subtract off an amount from the enemies HP.
SEC : SBC.w $0CE2, X : STA.w $0E50, X

STZ.w $0CE2, X

BEQ .BRANCH_ALPHA
    BCS Sprite_ScheduleForDeath_exit

.BRANCH_ALPHA
```

If health is left, the `BCS` leaves. Otherwise the code after
`.BRANCH_ALPHA` kills the sprite. The code changes the `BCS` offset from
`$73` to `$00`, so the branch lands on `.BRANCH_ALPHA` either way, and
any hit that does damage kills.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `BD 50 0E 85 00 38 FD E2 0C 9D 50 0E 9E E2 0C F0 02
B0 73` at `$06:EF61` (PC `$036F61`), the only place these bytes occur;
the `$73` is at `$06:EF73`. Hits that do no damage go to
`Sprite_Clear_queued_damage` and never start the timer, and freeze and
stun hits (`$FB` and up) branch off before the subtract. The code
decodes back to its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Enemies that can't be hurt by a weapon still can't be.
- Bosses that use this path die in one hit too. Bosses with phases or
  their own death scenes may skip them or get stuck. Not checked.

## The Fighter's Sword hits as hard as the Golden Sword

**Code:** `D037-0D28`
**Raw form:** `06ED33:04` (orig `$01`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 20:35:42

**Optional, spin attacks and stabs too:** `D937-0F28` + `D037-0428`

**Mechanism:** This is the ALttP version of the Zelda 1 code that makes
the wooden sword stronger. `Sprite_CalculateSwordDamage` (`Bank06.asm`)
picks a damage class from the sword level (`LDA.l $7EF359 : DEC`), plus
`$04` for a spin attack or `$08` for a stab:

```
Sprite_CalculateSwordDamage_damage_class:
db 1, 2, 3, 4 ; Normal strike damage indices
db 2, 3, 4, 5 ; Spin attack damage indices
db 1, 1, 2, 3 ; Stabbing damage indices
```

The class picks a row of `DamageSubclassValue` (`Bank0D.asm`). For most
enemies, class 1 (Fighter's Sword) does `$02` and class 4 (Golden Sword)
does `$10`, eight times as much. The code changes the Fighter's Sword's
normal strike class from 1 to 4. The optional codes change its spin
attack from class 2 to 5 and its stab from class 1 to 4.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `01 02 03 04 02 03 04 05 01 01 02 03` at `$06:ED33`
(PC `$036D33`). The first byte is the Fighter's Sword normal strike, the
5th its spin attack (`$06:ED37`) and the 9th its stab (`$06:ED3B`). All
three codes decode back to their raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Only the Fighter's Sword (level 1) changes. Better swords use their own
  bytes.
- Sword beams take their class from `AncillaDamageClasses` (`Bank06.asm`)
  by ancilla type, so they are not changed.
- Enemies that a sword can't hurt still can't be hurt.

## The Fighter's Sword shoots beams

**Code:** `DFB6-D42E`
**Raw form:** `079C8B:01` (orig `$02`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 20:37:16

Tags: `demo-ready` `video-ready` `curated` `play-start`

**Mechanism:** This is the ALttP version of the Zelda 1 code that lets the
wooden sword shoot. When Link finishes a swing, `Bank07.asm` checks
whether to fire a sword beam:

```
; If(actual health >= (goal health - 4))
LDA.l $7EF36C : SEC : SBC.b #$04 : CMP.l $7EF36D : BCS .cantShootBeam
    ; Check if we have a sword that can shoot the beam.
    LDA.l $7EF359 : INC : AND.b #$FE : BEQ .cantShootBeam
        LDA.l $7EF359 : CMP.b #$02 : BCC .cantShootBeam
```

The code changes `CMP.b #$02` (Master Sword or better) to `CMP.b #$01`,
so the Fighter's Sword passes too.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `AF 6C F3 7E 38 E9 04 CF 6D F3 7E B0 23 AF 59 F3 7E 1A
29 FE F0 1A AF 59 F3 7E C9 02 90 12` at `$07:9C70` (PC `$039C70`), the
only place these bytes occur; the `$02` is at `$07:9C8B`.
`AddSwordBeam` (`Bank0F.asm`) and `Ancilla_SwordBeam` (`Bank08.asm`)
don't read the sword level. The code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Tested by the user in Mesen2.
- Link still needs full health (at most half a heart missing) to shoot,
  as with the Master Sword. See "Sword beams fire at any health".
- The beam does the Master Sword beam's damage, since beams take their
  class from `AncillaDamageClasses`, not from the sword level.

## Sword beams fire at any health

**Code:** `DDB5-D7FE`
**Raw form:** `079C7C:00` (orig `$23`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 20:37:16

Tags: `demo-ready` `video-ready` `curated` `play-start`

**Mechanism:** This is the ALttP version of the Zelda 1 code that lets the
sword shoot without full hearts. When Link finishes a swing, `Bank07.asm`
checks his health before firing a beam:

```
; If(actual health >= (goal health - 4))
LDA.l $7EF36C : SEC : SBC.b #$04 : CMP.l $7EF36D : BCS .cantShootBeam
```

The code changes that `BCS` offset from `$23` to `$00`, so the branch
lands on the next instruction either way and the sword checks run at
any health.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `AF 6C F3 7E 38 E9 04 CF 6D F3 7E B0 23` at
`$07:9C70` (PC `$039C70`); the `$23` is at `$07:9C7C`. The code decodes
back to its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Tested by the user in Mesen2.
- Link still needs the Master Sword or better. Add "The Fighter's Sword
  shoots beams" (`DFB6-D42E`) for the first sword.

## Lamp flames hit as hard as Bombos

**Code:** `D23C-D798`
**Raw form:** `06ECAD:0D` (orig `$0B`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 20:38:19

**Mechanism:** This is the ALttP version of the Zelda 1 code that makes
Candle flames stronger. When an ancilla touches a sprite,
`Ancilla_CheckSpriteDamage` (`Bank06.asm`) gets its damage class from
`AncillaDamageClasses` by ancilla type
(`LDA.l AncillaDamageClasses, X`). The Lamp flame (ancilla 0x2F) has
class `$0B`, the Fire Rod's. The code changes it to `$0D`, Bombos.
`DamageSubclassValue` (`Bank0D.asm`) gives:

```
db $00, $08, $40, $FD, $04, $10, $00, $00 ; 0x0B - Fire rod
...
db $00, $10, $40, $FD, $00, $00, $00, $00 ; 0x0D - Bombos
```

So most enemies take `$10` instead of `$08`, and enemies that burn
(`$FD`) still burn.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `AncillaDamageClasses` at `$06:EC7E` (PC
`$036C7E`); entry 0x2F, at `$06:ECAD`, is `$0B`. The code decodes back
to its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Twice the damage for most enemies. Enemies whose subclass gave `$04`
  or `$10` with the Fire Rod class take 0 now.
- Classes `$0D` and up count as medallion damage in `Bank06.asm`
  (`CPY.b #$0D : BCS .medallion_damage_class`), so flames no longer
  knock enemies back.
- The Fire Rod's own shots use a different entry and are not changed.

## Sword beams fly through enemies

**Code:** `DDEA-A452`
**Raw form:** `0FF7C9:00` (orig `$06`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 20:39:59

Tags: `demo-ready` `video-ready` `curated` `play-start`

**Mechanism:** This is the ALttP version of the Zelda 1 code that keeps
sword beams flying after a hit. Each frame, `SwordBeam` (`Bank0F.asm`)
checks for sprites, then for walls:

```
JSL.l Ancilla_CheckSpriteCollisionLong : BCS .hit_sprite
JSL.l Ancilla_CheckTileCollisionLong : BCC .anohit_sprite_or_tile
```

At `.hit_sprite` the beam turns into a beam hit spark (ancilla 0x04) and
stops. The code sets the `BCS` offset to 0, so after a sprite hit the game
goes on to the wall check. The enemy still takes damage, since that
happens inside the sprite check. The beam now stops only at a wall.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `22 A2 8D 08 B0 06 22 3B 92 08 90 3B` at
`$0F:F7C4` (PC `$07F7C4`), the two checks above. The `BCS` offset
`$06` is at `$0F:F7C9`. The code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Tested by the user in Mesen2.
- A beam can pass through several enemies in a row. It may also hit the
  same enemy more than once while flying through it.
- Enemies that block or absorb the beam may not behave the same. Not
  checked.

## Spin attack charges at once

**Codes:** `DFB5-049E` + `DFB9-6D9E`
**Raw form:** `079D79:01` (orig `$30`), `079E51:01` (orig `$30`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 20:40:55

Tags: `demo-ready` `video-ready` `curated` `play-start`

**Mechanism:** This is the ALttP version of the SMB2 code that charges the
super jump at once. While Link holds B after a sword swing, the code in
`Link_ResetSwordAndItemUsage` (`Bank07.asm`) adds 1 to `$79` each
frame. When it reaches `$30`, it plays the charge sound and adds the
charge sparkle:

```
INC.b $79 : LDA.b $79 : CMP.b #$30 : BNE .return
```

When B is let go, `.maybeDoSpinAttack` spins only if the count is high
enough:

```
LDA.b $79 : CMP.b #$30 : BCC Link_ResetSwordAndItemUsage
```

Both codes change `#$30` to `#$01`. The first makes the charge sound and
sparkle come on the first frame of charging. The second lets a spin start
after that one frame.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A5 79 C9 30 90` at `$07:9D76` (PC `$039D76`) and
`E6 79 A5 79 C9 30 D0` at `$07:9E4C` (PC `$039E4C`). Each pattern
appears once in the ROM. The codes decode back to their raw forms with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Tested by the user in Mesen2.
- Use both codes. With only the second, the spin works at once but the
  sound and sparkle still wait for the full charge. With only the first,
  the sound comes early but the spin still needs the full charge.
- Holding B until the swing ends, even for a moment, now gives a spin
  attack when B is let go. A quick tap is still a normal swing.

## Shield blocks shots from any direction

**Code:** `DDEC-0FB6`
**Raw form:** `06F1A6:00` (orig `$3B`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 20:41:41

Tags: `demo-ready` `video-ready` `curated` `play-start`

**Mechanism:** This is the ALttP version of the Zelda 1 code that makes
the shield block from any side. When a shot that the shield can stop
(`$0BE0` bit `$20`) touches Link and he has a shield, the code in
`Bank06.asm` checks that the shot is moving toward the side Link faces:

```
LDY.w $0DE0, X
CMP.w Sprite_ToLink_Directions_opposing2, Y : BNE .BRANCH_ZETA
```

`.BRANCH_ZETA` hurts Link. The code sets the `BNE` offset to 0, so the
game always goes on to the block: the "tink" sound and the spark.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `BC E0 0D D9 41 F1 D0 3B A9 06 22` at `$06:F19F`
(PC `$03719F`). The `BNE` offset `$3B` is at `$06:F1A6`. The code
decodes back to its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Tested by the user in Mesen2.
- Link still needs a shield. Shots without bit `$20` in `$0BE0` still
  hurt him.
- Contact damage from enemies is not changed.

## Enemies always drop a prize

**Code:** `DDE8-042C`
**Raw form:** `06F9BB:00` (orig `$66`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 20:42:34

Tags: `demo-ready` `curated` `play-start`

**Mechanism:** This is the ALttP version of the Zelda 1 code that makes
monsters always drop an item. When an enemy dies, `Bank06.asm` reads its
prize pack from the low 4 bits of `$0BE0`, then rolls for a drop:

```
JSL.l GetRandomInt : PLY : AND.w PrizePack_Chance, Y : BNE .BRANCH_MU
```

`PrizePack_Chance` is `$01` for most packs, so a random 1 bit skips the
drop half the time. The code sets the `BNE` offset to 0, so the game always
goes on to pick a prize from `PrizePack_Prizes`.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `22 71 BA 0D 7A 39 5C FA D0 66` at `$06:F9B2` (PC
`$0379B2`). `PrizePack_Chance` at `$06:FA5C` is
`01 01 01 00 01 01 01 00`. The code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Tested by the user in Mesen2.
- Enemies with no prize pack (pack 0) still drop nothing.
- Bad luck (`$0CF9` = 2) still blocks drops, since that path skips the
  roll.

## Thrown objects hit enemies anywhere

**Code:** `DD33-0F96`
**Raw form:** `06E1E5:00` (orig `$CB`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 20:43:26

Tags: `demo-ready` `video-ready` `curated` `play-start`

**Mechanism:** This is the ALttP version of the SMB2 code that lets thrown
objects hit enemies anywhere. While a thrown pot, bush, rock or other
throwable sprite is moving, `ThrownSprite_CheckDamageToPeers`
(`Bank06.asm`) checks the other sprites on its layer. For each one,
`ThrownSprite_CheckDamageToSinglePeer` does:

```
JSR.w Utility_CheckIfHitBoxesOverlap : BCC ThrownSprite_CheckDamageToPeers_exit
```

The code sets the `BCC` offset to 0, so the game skips the hitbox check
and goes straight to the damage (`Ancilla_CheckSpriteDamage_preset_class`)
and knockback. The other checks stay: the object must be moving, and the
target must be alive, on the same layer, and not already dying.

An earlier version of this entry (`DD33-0FF6`) wrote to `$06:E1E4`, the
`BCC` opcode. That made a `BRK`, which would crash the game.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `20 36 F8 90 CB B9 20 0E C9 3F` at `$06:E1E1` (PC
`$0361E1`), so the `BCC` offset is at `$06:E1E5`. `$F836` is
`Utility_CheckIfHitBoxesOverlap` (PC
`$037836`). The code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Tested by the user in Mesen2.
- It reaches every active sprite on the layer, onscreen or not. It may also
  hit sprites that aren't enemies. Not checked.
- After each hit the object bounces back and slows down
  (`Sprite_ApplyRicochet`), and it waits 16 frames (`$0F10`) before it
  can hit again. So it hits a few enemies per throw, not all of them.
- Enemies (other than the tutorial soldier) that are hit at a distance
  get knocked back in the thrown object's direction.

## Every sprite acts like a soldier

**Codes:** `DDB5-6F26` + `D9B5-67F6` + `B7B5-6796` + `DD82-07F9`
**Raw form:** `069277:00` (orig `$FF`), `06927C:05` (orig `$83`),
`06927D:93` (orig `$92`), `05B5DC:00` (orig `$FF`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 20:56:38

Tags: `demo-ready`

**Mechanism:** This is the ALttP version of the SMW code that makes
sprites act like a Chuck and the SMB2 code that makes walkers move like
Trouters. Each frame, every active sprite (state `$09`) runs through
`SpriteActive_Main` (`Sprites/sprite_prep.asm`), which jumps through
`SpriteActive_Table` by sprite ID:

```
LDA.w $0E20, X
REP #$30
AND.w #$00FF : ASL : TAY
LDA.w SpriteActive_Table, Y : DEC : PHA
```

The first code makes the `AND` mask `#$0000`, so the index is always 0.
The second and third move the table address from `$9283` to `$9305`, the
entry for sprite 0x41, `SpriteActive2_Trampoline`. That sends every sprite
to `SpriteActive2_Main` (`Bank05.asm`), which does the same jump again
for sprites 0x41 to 0x70:

```
LDA.w $0E20, X : SEC : SBC.b #$41
REP #$30
AND.w #$00FF : ASL : TAY
LDA.w .sprite_routines, Y : DEC : PHA
```

The fourth code makes this mask `#$0000` too, so every sprite runs entry
0, `Sprite_Soldier`.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `BD 20 0E C2 30 29 FF 00 0A A8 B9 83 92` at
`$06:9271` (PC `$031271`) and `BD 20 0E 38 E9 41 C2 30 29 FF 00 0A A8
B9 E8 B5` at `$05:B5D3` (PC `$02B5D3`). `$06:9305` holds `EA BF`
(`$BFEA`, `SpriteActive2_Trampoline`). The codes decode back to their
raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- It changes every active sprite, not just enemies: items, rupees, NPCs,
  switches, bosses and others. Story scenes, shops and doors that need a
  sprite to do its own job may break or lock up the game.
- Only the behavior changes. Each sprite keeps its ID, so code that reads
  `$0E20` (soldier drawing and others) may show broken graphics. Soldier
  graphics are not loaded in most places.
- Sprites keep their own health, damage and other settings. Not checked
  how `Sprite_Soldier` uses them.
- Use all four codes. Without the fourth, most sprites run the wrong
  routines in bank 05 and the game will likely crash.

## Sword hits enemies across most of the screen

**Codes:** `3CED-6F2B` + `FDE1-D7BB` + `3DEC-D72B` + `FDEE-DDFB` +
`3DE7-0D9B`
**Raw form:** `06F607:EA` (orig `$AA`), `06F46E:10` (orig `$02`),
`06F4AF:E0` (orig `$04`), `06F4F0:10` (orig `$02`), `06F531:E0`
(orig `$04`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 20:59:21
**Optional, size `$C0` to hit bigger enemies instead:** `3CED-6F2B` + `FDE1-D7BB` + `ADEC-D72B` + `FDEE-DDFB` + `ADE7-0D9B`

Tags: `demo-ready` `video-ready` `curated` `play-start`

**Mechanism:** This is the ALttP version of the Zelda 1 code that lets the
sword hit every enemy on the screen. `Player_SetupActionHitBox`
(`Bank06.asm`) builds the sword's hitbox from `Pool_ActionHitbox`, one
entry per direction and swing frame:

```
LDA.b $2F : ASL #3 : CLC : ADC.b $3C : TAX
INX
```

The first code turns `TAX` into `NOP`, so `X` stays 0 and `INX` makes it
1. Every swing frame in every direction then uses entry 1 (Up, first frame).
The other four codes change entry 1: offset X and Y `2` → `16`, width and
height `4` → `$E0` (224).

The box can't simply be made bigger than this.
`Utility_CheckIfHitBoxesOverlap` ignores anything more than 128 pixels
from the box's corner on either axis, and it adds the two box sizes as 8-bit
values. If the sword's size plus the enemy's size is `$100` or more, the sum
wraps and the enemy is nearly never hit. `$E0` still works for enemy boxes
up to 31 pixels wide and tall (most normal enemies; see the
`Sprite_SetupHitBox` width and height tables). The +16 offset moves the
corner 16 pixels right of and below Link's position.

With these codes, an enemy is hit when its box's left edge is between 112
pixels left of Link (`$22`) and 143 pixels right of him, and the same for
up and down. Most of the time that is most of the screen.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A5 2F 0A 0A 0A 18 65 3C AA E8` at `$06:F5FF` (PC
`$0375FF`), with `TAX` at `$06:F607`. `Pool_ActionHitbox` entry 1 is
`$02` at `$06:F46E` (`.offset_x`), `$04` at `$06:F4AF` (`.size_x`),
`$02` at `$06:F4F0` (`.offset_y`) and `$04` at `$06:F531` (`.size_y`).
A Python model of `Utility_CheckIfHitBoxesOverlap` gave the ranges above.
The codes decode back to their raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Tested by the user in Mesen2, without the optional codes.
- Five codes, the most a real Game Genie holds.
- There is a gap where the enemy's box overlaps Link. Boxes whose left (or
  top) edge is 15 pixels before to 3 pixels after Link's are missed (fewer
  for wider boxes). Enemies touching Link can hit him first anyway.
- Enemies with boxes 32 pixels or more on an axis (big enemies and most
  bosses) are mostly missed. The optional `$C0` codes hit boxes up to 63
  pixels, with a larger gap near Link.
- The sword still hits only on the frames where `sword_validity` allows
  it. The spin attack and the dash use their own hitboxes and are not
  changed.
- Soldiers that block sword hits (`Guard_ParrySwordAttacks`) use the
  same box, so they may block from far away.

## Pull a bush or pot out of any wall

**Codes:** `DF23-AD2A` + `D489-0FBA` + `6D89-04BA`
**Raw form:** `07D3E3:01` (orig `$03`), `07B156:02` (orig `$08`),
`07B15A:80` (orig `$F0`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 21:03:14

Tags: `demo-ready` `video-ready` `curated` `play-start`

**Mechanism:** This is the closest ALttP gets to lifting almost anything.
When A is pressed, `Link_HandleLiftables` (`Bank07.asm`) picks the
action. Next to a wall (a tree, a cliff, a house and so on) it picks action
3, grab the wall:

```
LDA.b $0E : ORA.w $036D : AND.b #$01 : BEQ .notNearWall
    LDX.b #$03
```

The first code makes it action 1, lift (`Link_Lift`). `Link_Lift` then
calls `Overworld_LiftableTiles` (`Bank1B.asm`), which only knows the
map16 IDs of bushes, rocks and signs. For any other tile it changes
nothing and just returns the tile's attribute. `Link_Lift` looks that
attribute up in `Pool_LinkToss_liftable_tiles` to pick the object to
spawn, and gives up if it isn't there:

```
LDX.b #$08
.find_matching_tile_attrribute
    CMP.w Pool_LinkToss_liftable_tiles, X : BEQ .tile_attribute_match
DEX : BPL .find_matching_tile_attrribute
```

The second code starts `X` at 2, the green bush entry (`$50`). The third
turns `BEQ` into `BRA`, so the first check always matches. Link then
spawns a throwable bush (sprite 0xEC, type 2) and holds it over his head.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A2 02 A5 0E 0D 6D 03 29 01 F0 02 A2 03` at
`$07:D3D7` (PC `$03D3D7`) and `A2 08 DD AD B1 F0 06 CA 10 F8` at
`$07:B155` (PC `$03B155`). `Pool_LinkToss_liftable_tiles` at
`$07:B1AD` is `54 52 50 FF 51 53 55 56 57`. A second copy of the loop at
`$07:C1CB` (rock piles) is not changed. The codes decode back to their
raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Tested by the user in Mesen2.
- The wall stays in place, so Link can pull bush after bush out of the
  same spot. The game has no tile to put there instead.
- Link can no longer grab walls with A.
- Small rocks and signs are lifted as bushes too: they look, break and
  hit like bushes. Their tiles are still removed as usual. Large rocks
  use a separate path and are not changed.
- In dungeons, Link pulls out a pot instead of a bush (seen in play).
- Indoors, `Dungeon_RevealCoveredTiles` (`Bank01.asm`) uses the low 4
  bits of the tile's attribute as the index of a pot or other liftable
  object in the room. Lifting a wall may remove a pot somewhere else in
  the room. Not checked.
- If `$0368` still holds a large rock's value from an earlier lift,
  the first lift may use the heavy lift animation. Not checked.

## New files start after Zelda's rescue

**Codes:** `DD2F-D4E7` + `DD24-DFE7` + `DD27-DD77`
**Raw form:** `0CDC1B:00` (orig `$2D`), `0CDC27:00` (orig `$21`),
`0CDC30:00` (orig `$18`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 21:05:13
**Optional, the normal start items plus the Fighter's Sword instead:** `DD2F-D4E7` + `DD24-DFE7` + `DD27-DD77` + `7A20-DFE7` + `DFE2-D72F`

**Mechanism:** This skips the rainy night and the Hyrule Castle rescue.
The game starts with the overworld open, as it is after Zelda reaches
the Sanctuary. When a new file is made, `Bank0C.asm` has a debug start
that only runs in the first slot, with the second controller turned on
in the code, and with a name starting with "B":

```
LDY.w #$003C : CPX.w #$0000 : BNE .loadInitialEquipment
    LDA.l Player2JoypadReturn : AND.w #$00FF : CMP.w #$0060 : BEQ .loadInitialEquipment
        LDA.l $7003D9 : CMP.w #$0001 : BNE .loadInitialEquipment
            LDA.w #$00F0 : STA.l $700212, X
            ; Set the game mode to after saving Zelda.
            LDA.w #$1502 : STA.l $7003C5, X
            LDA.w #$0100 : STA.l $7003C7, X
            LDY.w #$0000
```

The three codes set the three branch offsets to 0, so every new file goes
through the debug start. It sets the progress value `$x3C5` to 2 (first
Hyrule Castle visit done) and the story flags `$x3C6` to `$15` (uncle's
gear taken, Zelda brought to the Sanctuary, uncle gone from the house).
It sets the map icons `$x3C7` to 0 and the start point `$x3C8` to 1.
`Y` = 0 also picks the debug loadout (`.bage` in
`DefaultSaveFileItems`), which has almost every item and the Fighter's
Sword and Shield. See "Start a new game with BAGE debug loadout (almost
every item)".

With the optional set, the fourth code puts `LDY.w #$003C` back, which
picks the normal loadout. The fifth code gives that loadout the Fighter's
Sword (`$x359` = 1). Link needs it because the uncle won't give it now.
An earlier version of the fourth code (`7A20-DF57`) wrote to `$0C:DC45`,
the bank byte of `STA.l $7003C7`, so the optional set still gave the debug
loadout.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has, from `$0C:DC14` (PC `$065C14`):
`A0 3C 00 E0 00 00 D0 2D AF F8 83 00 29 FF 00 C9 60 00 F0 21 AF D9 03 70
C9 01 00 D0 18 A9 F0 00 9F 12 02 70 A9 02 15 9F C5 03 70 A9 00 01 9F C7 03
70 A0 00 00`. The branch offsets are at `$0C:DC1B`, `$0C:DC27` and
`$0C:DC30`, and `LDY.w #$0000` is at `$0C:DC46` (the fourth optional
code changes its low byte, at `$0C:DC47`, from `$00` to `$3C`). The normal loadout at
`$04:F4C6` (PC `$0274C6`) has `$00` for the sword at `$04:F4DF`
(offset `$19`, `$x359`). The meanings of `$x3C5`-`$x3C8` are from
`Logs/Zelda_3_SRM.log`. The codes decode back to their raw forms with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Only files made while the codes are on are changed. Make the file with
  the codes on, then the file keeps its start.
- The start point `$x3C8` = 1 is the Sanctuary in
  `Logs/Zelda_3_SRM.log`; the source comment says Link's house. Not
  checked. After progress 2 the game may offer a choice of start point.
- Items from the skipped part (the Hyrule Castle chests, such as the
  blue boomerang) are not given. Whether those chests can still be
  reached is not checked.
- The optional set gives no shield, since a sixth code would be over the
  Game Genie's limit of five.
- The debug start does not set the dying priest flag (`$x3C6` bit 1) or
  the pendant map icons (`$x3C7` = 3). Sanctuary scenes and the map may
  not be what they are after a normal rescue. Not checked.
- `$x212` = `$F0` is also written; the source comment guesses it keeps
  Link from getting the magic powder again. Not checked.

## Time moves only while R is held

**Codes:** `DD67-D46D` + `E460-DF0D` + `ED60-D7AD` (use all three)
**Raw form:** `00803A:00` (orig `$16`) + `008045:F2` (orig `$F6`) +
`00804F:F0` (orig `$D0`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 21:44:45

Tags: `demo-ready` `play-start`

**Mechanism:** This is the ALttP version of the SMB3, SMW and Metroid
"time moves only while a button is held" codes. The main loop in
`Bank00.asm` has unused debug code that freezes the game:

```
BRA .do_frame
; UNUSED: Inaccessible code, used for debug if assembled in.
LDA.b $F6 : AND.b #$20 : BEQ .L_ButtonDown
    INC.w $0FD7
.L_ButtonDown
LDA.b $F6 : AND.b #$10 : BNE .R_ButtonDown
    LDA.w $0FD7 : AND.b #$01 : BNE .skip_frame
.R_ButtonDown
.do_frame
INC.b $1A
JSR.w ClearOamBuffer
JSL.l Module_MainRouting
.skip_frame
```

`$F6` holds the buttons pressed this frame (L is `$20`, R is `$10`).
Pressing L flips bit 0 of `$0FD7` (free RAM). While it is set, a frame
runs only on a frame when R was just pressed. The first code sets the
`BRA` offset to `$00`, so the debug code runs. The second code makes the
R check read `$F2` (buttons held, from `NMI_ReadJoypads`) instead of
`$F6`, so the game runs while R is held. The third code changes `BNE
.skip_frame` to `BEQ`, so the game starts frozen (`$0FD7` starts at 0).

An earlier version of this entry (`E460-DF6D` + `ED69-DDDD`) was off by
one byte. It wrote to `$008046` (the `AND` opcode) and `$008050` (the
branch offset) and did not work. "The game runs at half speed" uses the
same debug code and works, which shows the first code is right.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `58 80 16 A5 F6 29 20 F0 03 EE D7 0F A5 F6 29 10 D0
07 AD D7 0F 29 01 D0 09 E6 1A` at `$00:8038`, which matches the code
above. So the `BRA` offset is at `$00:803A`, the second `LDA.b $F6`
operand is at `$00:8045`, and the `BNE .skip_frame` opcode is at
`$00:804F`. `General/WRAM.asm` lists `$0FD7` as free, and only this code
uses it. The codes decode back to their raw forms with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Tested by the user in Mesen2.
- The freeze covers everything, from the boot screens on. With the codes
  on at power-on, the screen stays black until R is held, so hold R to get
  through the boot screens, the title and file select.
- L is a toggle. Press L once to play at normal speed without R, and press
  L again to go back to "only while R is held".
- Music and sound effects keep playing while the game is frozen, since the
  sound chip runs on its own.
- Without the third code the game starts at normal speed, and L turns the
  freeze on.
- The triforce intro and the ending use `$F2` for something else, so the
  freeze may act oddly there. Not checked.
- These codes share `DD67-D46D` with "The game runs at half speed" and
  "L pauses the game, R steps one frame". Use only one of the three.
- "Time moves only while a button is held" changes the same bytes and
  more. Use only one of the two.

## Cuccos (chickens) summon attack cuccos on sight

**Code:** `DDC0-642B`
**Raw form:** `06A64B:00` (orig `$03`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-18 21:51:34

**Mechanism:** Each cucco (`Sprite_Chicken`, `Sprites/sprite_chicken.asm`)
counts the times Link hits it in `$0DA0,X`. Once the count reaches `$23`,
the cucco keeps calling the attack cucco spawner:

```
LDA.w $0DA0, X : CMP.b #$23 : BCC .anospawn_attack_chicken
    JSR.w Chicken_SpawnAvengerChicken
.anospawn_attack_chicken
```

The code sets the `BCC` offset to `$00`, so every cucco calls
`Chicken_SpawnAvengerChicken` from the start, without being hit.
The spawner adds one attack cucco (`$0DB0` = 1) every 16 frames per
cucco. Each one enters from the screen edge and flies at Link.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `BD A0 0D C9 23 90 03 20 D3 A7` at `$06:A645` (PC
`$032645`). This byte pattern occurs only once in the ROM. `$A7D3` is
`Chicken_SpawnAvengerChicken` (PC `$0327D3`). The code decodes back to
its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Attack cuccos hurt Link, so a long stay near cuccos can kill him. Turn
  the code off to stop new ones from coming.
- The spawner only runs on the overworld (`$1B` = 0), so cuccos indoors
  don't start a frenzy.
- A cucco that Link is holding or has thrown doesn't spawn attack cuccos.
  `Sprite_CheckIfActive` returns before the check, so it starts again
  once the cucco lands.
- Attack cuccos don't spawn more of their own. They take a different path
  before the check.
- When the sprite slots are full, no more attack cuccos spawn until one
  leaves.
- Hit cuccos still bawk and flee as before.

## Time moves only while a button is held

**Codes:** `DD67-D46D` + `ED60-DF0D` + `D960-DF6D` + `E460-DFAD` +
`ED60-D7AD` (use all five)
**Raw form:** `00803A:00` (orig `$16`) + `008045:F0` (orig `$F6`) +
`008046:05` (orig `$29`) + `008047:F2` (orig `$10`) + `00804F:F0` (orig
`$D0`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-19 00:39:56

Tags: `demo-ready` `play-start`

**Mechanism:** This is "Time moves only while R is held" with any button
in place of R. It uses the same unused debug code in the main loop
(`Bank00.asm`):

```
BRA .do_frame
; UNUSED: Inaccessible code, used for debug if assembled in.
LDA.b $F6 : AND.b #$20 : BEQ .L_ButtonDown
    INC.w $0FD7
.L_ButtonDown
LDA.b $F6 : AND.b #$10 : BNE .R_ButtonDown
    LDA.w $0FD7 : AND.b #$01 : BNE .skip_frame
.R_ButtonDown
.do_frame
INC.b $1A
JSR.w ClearOamBuffer
JSL.l Module_MainRouting
.skip_frame
```

`NMI_ReadJoypads` keeps the buttons held on controller 1 in two bytes:
`$F0` has B, Y, Select, Start and the d-pad, and `$F2` has A, X, L and
R. The first code sets the `BRA` offset to `$00`, so the debug code runs.
The next three codes change `LDA.b $F6 : AND.b #$10` to `LDA.b $F0 :
ORA.b $F2`, so the `BNE` runs the frame when any button is held. The last
code changes `BNE .skip_frame` to `BEQ`, so with no button held the game
skips the frame (`$0FD7` starts at 0).

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A5 12 F0 FC 58 80 16 A5 F6 29 20 F0 03 EE D7 0F A5
F6 29 10 D0 07 AD D7 0F 29 01 D0 09 E6 1A` at `$00:8034` (PC `$000034`),
the only place these bytes occur. A script search found the pattern, and
the bytes at the five addresses were read back from the ROM: `$16` at
`$00:803A`, `F6 29 10` at `$00:8045`-`$00:8047` and `$D0` at `$00:804F`.
`$05` is `ORA` direct page. The codes decode back to their raw forms with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Only controller 1 counts.
- The freeze covers everything, from the boot screens on. With the codes
  on at power-on, the screen stays black until a button is held.
- L is still a toggle, as in the R version. Pressing L once makes the game
  run at normal speed with no button held, and pressing L again brings
  back the freeze.
- Music and sound effects keep playing while the game is frozen, since the
  sound chip runs on its own.
- Five codes, the most a real Game Genie holds.
- These codes share `DD67-D46D` with "The game runs at half speed" and
  "L pauses the game, R steps one frame", and change the same bytes as
  "Time moves only while R is held". Use only one of the four.

## Sword beams fly twice as fast

**Code:** `3CE9-6FE3`
**Optional, 4x speed instead:** `3CE9-6FE3` + `3CE9-6473`
**Raw form:** `0FFA57:EA` (orig `$4A`); optional also `0FFA58:EA` (orig
`$4A`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-22 21:52:50

Tags: `demo-ready`

**Mechanism:** Each frame, `SwordBeam` (`Bank0F.asm`) moves the beam
with `SwordBeam_MoveVert` and `SwordBeam_MoveHoriz`. `AddSwordBeam`
sets the speeds from `Pool_AddSwordBeam` (`.y_speeds` `$C0, $40, $00,
$00` and `.x_speeds` `$00, $00, $C0, $40`, by Link's direction). The
upper 4 bits of a speed are pixels per frame and the lower 4 bits are
1/16ths of a pixel, so the beam moves 4 pixels per frame. The mover
gets the whole pixels with four shifts:

```
LDA.w $0C22, X : PHP : LSR #4 : PLP : BPL .moving_down
    ; Sign extend from 4-bits to 8-bits.
    ORA.b #$F0
```

The code changes the first `LSR` to `NOP`, so the speed is shifted only
three times. `$40` gives 8 pixels per frame, and `$C0` gives `$18`,
which the `ORA` makes `$F8` (-8). The optional code changes the second
`LSR` too, for 16 pixels per frame. The `PLP` restores the flags, so
the sign test and the carry from the 1/16-pixel add are not changed.
`SwordBeam_MoveHoriz` runs the same code on the X speed, so both
directions change.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `BD 22 0C 0A 0A 0A 0A 18 7D 36 0C 9D 36 0C A0 00
BD 22 0C 08 4A 4A 4A 4A 28 10 03 09 F0` at `$0F:FA43` (PC `$07FA43`),
`SwordBeam_MoveVert`; the `LSR`s are at `$0F:FA57`-`$0F:FA5A`. A search
of bank `$0F` finds only three calls: `JSR $FA43` at `$0F:F78F` and
`$0F:FA3C` (in `SwordBeam_MoveHoriz`), and `JSR $FA37` at `$0F:F792`.
All are sword beam code. The codes decode back to their raw forms with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Only the sword beam (ancilla 0x0C) uses this mover. The beam hit spark
  and other ancillae are not changed.
- At 16 pixels per frame the beam checks for walls and enemies only
  every 16 pixels, so it may skip past small enemies or thin walls.
- Don't use with "Sword beams fly at quarter speed" or "Sword beams stay
  where they appear".

## Sword beams fly at quarter speed

**Codes:** `EDE9-67E2` + `FDE1-6D72` + `EDE1-6F52` + `FDE1-6F82` (use all
four)
**Raw form:** `0FF65F:F0` (orig `$C0`), `0FF660:10` (orig `$40`),
`0FF665:F0` (orig `$C0`), `0FF666:10` (orig `$40`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-22 21:52:50

Tags: `demo-ready` `video-ready` `curated`

**Mechanism:** When Link fires a beam, `AddSwordBeam` (`Bank0F.asm`)
sets its speeds by his direction (`LDA.b $2F : LSR`):

```
LDA.w .y_speeds, Y        : STA.w $0C22, X
LDA.w .x_speeds, Y        : STA.w $0C2C, X
```

`.y_speeds` is `$C0, $40, $00, $00` (up, down, left, right) and
`.x_speeds` is `$00, $00, $C0, $40`. The upper 4 bits are pixels per
frame and the lower 4 bits are 1/16ths of a pixel, so `$40` is 4 pixels
per frame and `$C0` is -4. The codes change the four moving speeds to
`$10` and `$F0`, 1 pixel per frame.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `C0 40 00 00 00 00 C0 40` at `$0F:F65F` (PC
`$07F65F`), `.y_speeds` and `.x_speeds`. The codes decode back to their
raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Link can have only one beam out at a time (`AddSwordBeam` is called
  with `Y` = 0, the most copies `Ancilla_CheckForAvailableSlot`
  allows). A slow beam stays out longer, so Link can't shoot again until
  it hits something or leaves the screen.
- The beam's sound still plays every 16 frames while it flies, so it
  plays longer.
- Don't use with "Sword beams fly twice as fast" or "Sword beams stay
  where they appear".

## Sword beams stay where they appear

**Code:** `1DE0-6DE3`
**Raw form:** `0FFA43:60` (orig `$BD`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-22 21:52:50

Tags: `demo-ready`

**Mechanism:** Each frame, `SwordBeam` (`Bank0F.asm`) moves the beam and
then checks for enemies and walls:

```
JSR.w SwordBeam_MoveVert
JSR.w SwordBeam_MoveHoriz
...
JSL.l Ancilla_CheckSpriteCollisionLong : BCS .hit_sprite
JSL.l Ancilla_CheckTileCollisionLong : BCC .anohit_sprite_or_tile
```

`SwordBeam_MoveHoriz` calls `SwordBeam_MoveVert` with X moved to the X
speed and position. The code changes the first byte of
`SwordBeam_MoveVert` (`LDA.w $0C22, X`, `$BD`) to `RTS` (`$60`), so
neither call moves the beam. The beam stays where `AddSwordBeam` puts
it, by Link, and keeps spinning. It still checks for enemies each
frame, so an enemy that walks into it takes the hit.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `BD 22 0C 0A 0A 0A 0A 18` at `$0F:FA43` (PC
`$07FA43`), the start of `SwordBeam_MoveVert`. A search of bank `$0F`
finds only three calls: `JSR $FA43` at `$0F:F78F` and `$0F:FA3C` (in
`SwordBeam_MoveHoriz`), and `JSR $FA37` at `$0F:F792`. All are sword
beam code. The code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Link can have only one beam out at a time (`AddSwordBeam` is called
  with `Y` = 0). The beam is removed only when it hits an enemy or goes
  off the screen, so Link can't shoot again until then.
- The beam's sound plays every 16 frames for as long as it stays.
- Don't use with "Sword beams fly twice as fast" or "Sword beams fly at
  quarter speed".

## Hold B to swing the sword again and again with no delay

**Code:** `DDB1-0DFE`
**Raw form:** `079D60:00` (orig `$0D`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 18:42:48

Tags: `demo-ready` `video-ready` `curated` `interesting`

**Mechanism:** A sword swing is a list of animation steps in `$3C`.
`Link_CheckForSwordSwing` (`Bank07.asm`) moves to the next step when the
`$3D` timer runs out. From step 4 on, it can start a new swing, but only
if B was let go since the swing started (bit 0 of `$3A`) and is now held:

```
CPX.b #$04 : BCC .BRANCH_DELTA
    LDA.b $3A : AND.b #$01 : BEQ .BRANCH_DELTA
        BIT.b $F0 : BPL .BRANCH_DELTA
            LDA.b $3A : AND.b #$FE : STA.b $3A

            BRL HandleSwordSFXAndBeam
```

So in the normal game you must tap B again to cut a swing short. The
code sets the `BEQ` offset to `$00`, so the "B was let go" check does
nothing. Now holding B starts a new swing each time the swing reaches
step 4. Steps 0-2 have no hit box (`Pool_ActionHitbox_sword_validity`),
and step 3 hits, so each swing still hits once. One swing takes about 5
frames, so Link swings about 12 times per second for as long as B is
held.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `E0 04 90 13 A5 3A 29 01 F0 0D 24 F0 10 09` at
`$07:9D57` (PC `$039D57`). The `$0D` is at `$07:9D60` (PC `$039D60`).
The bytes `A5 3A 29 01 F0` appear only once in the ROM. The code decodes
back to its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- No spin attack. While B is held, the swing keeps starting over, so
  `$3C` never reaches step 9, where the spin attack charge starts.
- Each new swing plays the sword sound and can fire a sword beam (full
  health and Master Sword or better). Only one beam can be out at a
  time.
- Letting go of B ends the swing as normal. Tapping B still works as
  before.

## Tap B to swing the sword again with almost no delay

**Code:** `DDBF-0F2E`
**Raw form:** `079D17:00` (orig `$56`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 19:25:42

Tags: `interesting`

**Mechanism:** This is a version of "Hold B to swing the sword again and
again with no delay" that keeps the spin attack. You still have to let
go of B and press it again for each swing.

A swing is a list of steps in `$3C`. `Link_CheckForSwordSwing`
(`Bank07.asm`) moves to the next step only when the `$3D` timer runs
out, and loads the next wait from `SwordSwingTimers`:

```
DEC.b $3D : BPL .BRANCH_DELTA
    INC.b $3C
    LDA.b $3C : CMP.b #$09 : BCS .maybeDoSpinAttack
        TAX
        LDA.w SwordSwingTimers, X : STA.b $3D
```

`SwordSwingTimers` holds `$01` for step 0, `$03` for step 5 and
`$01` for step 8, so a swing takes about 14 frames. A new swing can
start only on a frame where the step changes, from step 4 on, and only if
B was let go since the swing started and is held again. A quick tap that
falls between those frames is lost, and Link has to finish the swing.

The code sets the `BPL` offset to `$00`, so the step moves on every
frame. A swing now takes 9 frames, and from step 4 on every frame except
step 5 checks for a new swing. Steps 3 to 6 still have a hit box
(`Pool_ActionHitbox_sword_validity`), and step 5 still cuts bushes, so
each swing works as before, only faster.

The spin attack is not changed. When B is still held at step 9,
`.maybeDoSpinAttack` starts the spin charge as usual. The charge and the
spin use their own timers in `Link_ResetSwordAndItemUsage`.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `C6 3D 10 56 E6 3C A5 3C C9 09 B0 52` at
`$07:9D14` (PC `$039D14`). The `$56` is at `$07:9D17` (PC
`$039D17`). The pattern `C6 3D 10 56 E6 3C` appears only once in the
ROM. The `BPL` goes to `$07:9D6E` (`JSR.w CalculateSwordHitbox`). The
code decodes back to its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- The swing animation plays faster, one frame per step.
- Each new swing plays the sword sound and can fire a sword beam (full
  health and Master Sword or better). Only one beam can be out at a
  time.
- With "Hold B to swing the sword again and again with no delay" too,
  holding B swings even faster, but there is no spin attack.

## Link can turn while holding B with the sword

**Code:** `DD3A-64F2`
**Raw form:** `07E6C8:00` (orig `$3E`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 19:38:36

Tags: `interesting` `play-start`

**Mechanism:** Bit 0 of `$50` locks the way Link faces. A sword swing
sets it (`LDA.b #$01 : TSB.b $50` in `Link_CheckForSwordSwing`), and so
does the spin charge while B is held (`Link_ResetSwordAndItemUsage`).
Normally Link walks while charging but keeps facing the same way.

The facing is set from the walking direction in
`Link_HandleMovingAnimation_FullLongEntry` (`Bank07.asm`), which skips
that work if any bit of `$50` is set:

```
; Check if Link can change direction:
LDA.b $50 : BNE Link_HandleMovingAnimation_StartWithDash_check_dashing
```

The code sets the `BNE` offset to `$00`, so the branch goes to the next
instruction either way, and `$2F` (Link's facing) always follows the
D-pad. Holding B, Link turns to face the way he walks, and the charged
sword turns with him. When B is let go, the spin attack starts from the
way he faces then.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A5 50 D0 3E A5 6A` at `$07:E6C5` (PC
`$03E6C5`). The `$3E` is at `$07:E6C8` (PC `$03E6C8`). The `BNE`
goes to `$07:E707`, `.check_dashing` in
`Link_HandleMovingAnimation_StartWithDash` (`AD 72 03` there,
`LDA.w $0372`). The pattern `A5 50 D0 ?? A5 6A F0` also appears at
`$07:E7FF` in `Link_HandleMovingAnimationSwimming`, which this code
does not change. The code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Other moves that set `$50` lose their lock too: Link turns while he
  walks in them.
- Swimming is not changed.
- Two other checks of bit 0 of `$50`, in
  `ChangeAxisOfPerpendicularDoorMovement_Vertical` and `_Horizontal`,
  are not changed. Walking diagonally into a doorway may still keep the
  old facing.

## The Magic Cape is always on, even without it

**Code:** `996C-0F2A`
**Raw form:** `0781A7:55` (orig `$29`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 19:36:39

Tags: `demo-ready` `interesting` `play-start`

**Mechanism:** `$55` is the cape flag (`WRAM.asm`: "when set, makes you
invisible and invincible. You can also go through objects, such as
bungies."). `LinkItem_Cape` (`Bank07.asm`) sets it to 1 when Link puts
the cape on. Each frame in the ground state, `LinkState_Default` resets
Link's height and Z speed:

```
LDA.b #$FF : STA.b $24
             STA.b $25
             STA.b $29
```

The code changes `STA.b $29` to `STA.b $55`, so the cape flag is set to
`$FF` every frame Link is on the ground. Every reader of `$55` only
tests for zero. With it set:

- Link is drawn invisible (`Bank0D.asm`, `.looks_invisible`).
- `Link_ControlHandler` throws away all damage and knockback
  (`STZ.w $0373`, `STZ.b $4D`, `STZ.b $46`).
- Spike floors and spike blocks don't hurt him (`.noSpikeFloorDamage`,
  `.ignoreSpikeBlocks`), and bumpers don't push him
  (`sprite_bumper.asm`, `.using_magic_cape`).

Magic is used only while the cape is the selected item
(`LinkItem_Cape` and `Link_HandleCape_passive` check `$0303`/`$0304`
for 0x13), so without the cape selected, this costs no magic.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A9 FF 85 24 85 25 85 29 9C C6 02` at `$07:81A0`
(PC `$0381A0`), in `LinkState_Default` after `LDA.b #$02 : STA.b $5D :
BRL LinkState_Recoil`. The `$29` is at `$07:81A7`. The same 8 bytes are
also in `LinkState_Bunny` and `LinkState_HoldingBigRock`; those are not
changed. The code decodes back to its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- `$29` is no longer reset to `$FF` on the ground. Every recoil and hop
  the disassembly shows sets `$29` itself, so this should not matter,
  but a jump could start with an old Z speed.
- In other states (swimming, dashing, as a bunny) the flag stays set
  from the last ground frame, unless something clears it. Several
  things do (`Link_ForceUnequipCape`, for example when Link dashes into
  an electric barrier); it comes back as soon as Link is on the ground
  again.
- Link can't be seen, so he is hard to follow. Some enemies may still
  react to him; only the cape checks above are changed.

## Link is a bunny in the Light World (no Moon Pearl)

**Code:** `DD64-0DDB`
**Raw form:** `028520:00` (orig `$0F`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 19:39:51

Tags: `demo-ready` `interesting` `play-start`

**Mechanism:** When Link goes out to the overworld from a house, cave or
dungeon, `PreOverworld_LoadProperties` (`Bank02.asm`) decides whether to
turn him into a bunny:

```
LDA.l $7EF357 : BNE .notBunny
    LDA.l $7EF3CA : BEQ .notBunny
        LDA.b #$01 : STA.w $02E0
                     STA.b $56

        LDA.b #$17 : STA.b $5D

        JSL.l LoadGearPalettes_bunny

.notBunny
```

`$7EF357` is the Moon Pearl and `$7EF3CA` is the world (`$00` Light
World, `$40` Dark World). The code sets the second `BEQ` offset to
`$00`, so the world is no longer checked. Without the Moon Pearl, Link
turns into a bunny every time he goes out to the overworld. The game's
own bunny code does all the work: `$02E0` picks the bunny graphics,
`$56` marks Link as a bunny, `$5D = 0x17` is the permabunny state, and
`LoadGearPalettes_bunny` loads the bunny colors.

On a new game this happens as soon as Link walks out of his house in the
rain.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `AF 57 F3 7E D0 15 AF CA F3 7E F0 0F A9 01 8D E0
02 85 56 A9 17 85 5D 22` at `$02:8515` (PC `$010515`). The `$0F` is at
`$02:8520` (PC `$010520`). The pattern `AF CA F3 7E F0 ?? A9 01 8D E0 02`
appears only once in the ROM. The `BEQ` goes to `$02:8530`
(`A9 09 85 94`, `LDA.b #$09 : STA.b $94`), which is `.notBunny`. The
code decodes back to its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Once Link has the Moon Pearl, the code does nothing.
- A bunny can't use the sword or items and can't lift bushes, so the
  story can't go on far. The bush over the secret way into the castle
  can't be lifted.
- Turning the code off does not change Link back. Without the Moon
  Pearl, going into a house keeps him a bunny (`AdjustBunnyLinkStatus`
  changes nothing without it). Not checked whether saving and loading
  the game changes him back.
- The Light World plays its usual music, since the bunny music is chosen
  only for Dark World areas.

## Select turns Link into a bunny for a while (overworld)

**Codes:** `E9C6-0DAB` `D7C6-0FDB` `6DC6-0FAB` `D3C6-04DB`
(use all four)
**Raw form:** `02A583:F5` (orig `$F0`); `02A584:03` (orig `$1C`);
`02A587:80` (orig `$A5`); `02A588:0E` (orig `$10`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 19:44:53

Tags: `interesting` `play-start`

**Mechanism:** `$03F5` (16-bit) is the temporary bunny timer. A yellow
hunter's beam sets it to `$0100` (`sprite_rabbit_beam.asm`). While it
is not 0, `Link_HandleBunnyTransformation` (`Bank07.asm`), called from
`LinkState_Default`, does the whole change: the smoke cloud, the sound,
20 frames with Link hidden, then state `0x1C`
(`LinkState_TemporaryBunny`) with the bunny graphics (`$02E0`, `$56`)
and colors. `LinkState_TemporaryBunny` counts the timer down and turns
Link back, with another cloud, when it reaches 0.

On the overworld, `Overworld_PlayerControl` (`Bank02.asm`) opens the
save menu while Select is held:

```
LDA.b $F0 : AND.b #$20 : BEQ .selectButtonNotDown
    LDA.w $1CE8 : STA.w $1CF4

    REP #$20

    LDA.w #$0186 : STA.w $1CF0

    SEP #$20

    LDA.b $10 : PHA

    JSL.l Main_ShowTextMessage
    ...
.selectButtonNotDown
```

`$1CF0` is the message to show (`$0186`, the save menu). The first two
codes change `STA.w $1CF0` to `STA.w $03F5`, so the message number
becomes the bunny timer: `$0186` frames, about 6.5 seconds. The last
two change `LDA.b $10` to `BRA .selectButtonNotDown` (offset `$0E`),
so the save menu never opens. The `LDA.w $1CE8 : STA.w $1CF4` before
it still runs; it only saves the menu cursor.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A5 F0 29 20 F0 20 AD E8 1C 8D F4 1C C2 20 A9 86
01 8D F0 1C E2 20 A5 10 48 22 AA FD 0F` at `$02:A571` (PC `$012571`).
The pattern appears only once in the ROM. The `F0 1C` of
`STA.w $1CF0` is at `$02:A583`, and `A5 10` is at `$02:A587`. The new
`BRA` goes to `$02:A597`, `.selectButtonNotDown`. The codes decode back
to their raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Overworld only. Select no longer opens the save menu there.
- The timer is set again every frame Select is held, so Link stays a
  bunny until about 6.5 seconds after Select is let go.
- Works with or without the Moon Pearl.
- As with the hunter's beam, a hit sets the timer to 0 and turns Link
  back. The change happens only in the ground state.
- Uses the same bytes as the other overworld Select cheats (such as the
  Select teleports). Use only one of them.

## Select turns Link into a bunny for 18 minutes (overworld)

**Codes:** `E9C6-0DAB` `D7C6-0FDB` `6DC6-0FAB` `D3C6-04DB` `EEC6-0D0B`
(use all five)
**Raw form:** `02A583:F5` (orig `$F0`); `02A584:03` (orig `$1C`);
`02A587:80` (orig `$A5`); `02A588:0E` (orig `$10`); `02A581:FF`
(orig `$01`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 19:46:21

Tags: `interesting` `play-start`

**Mechanism:** This is "Select turns Link into a bunny for a while
(overworld)" with one more code. The first four codes turn the save
menu's `LDA.w #$0186 : STA.w $1CF0` into a store to the temporary bunny
timer `$03F5`, and skip the menu. The fifth changes the `$01` of
`#$0186` to `$FF`, so the timer starts at `$FF86` (65414 frames, about
18 minutes).

This is as close to a permanent bunny as Select can get in a few
codes. The permabunny state `0x17` (`LinkState_Bunny`) is the
temporary bunny state `0x1C` without the timer:
`LinkState_TemporaryBunny` (`Bank07.asm`) only counts the timer down
and then falls through into `LinkState_Bunny`. Setting state `0x17`
from Select would also need the bunny graphics flags (`$02E0`, `$56`)
and a `JSL.l LoadGearPalettes_bunny`, which the temporary bunny
change (`Link_HandleBunnyTransformation`) already does.

**Evidence:** As in "Select turns Link into a bunny for a while
(overworld)". `alttp_original.smc` has `A9 86 01` (`LDA.w #$0186`) at
`$02:A57F` (PC `$01257F`), so the `$01` is at `$02:A581`. The codes
decode back to their raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Overworld only. Select no longer opens the save menu there.
- A hit sets the timer to 0 and turns Link back, as with the hunter's
  beam. Press Select again to turn back into a bunny.
- Uses the same bytes as the other Select cheats. Use only one of them.

## Select makes Link hold up an item (overworld)

**Codes:** `2CC6-0DAB` `D4C6-0FDB` `6DC6-0FAB` `D3C6-04DB`
(use all four)
**Raw form:** `02A583:DA` (orig `$F0`); `02A584:02` (orig `$1C`);
`02A587:80` (orig `$A5`); `02A588:0E` (orig `$10`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 19:46:21

Tags: `interesting` `play-start`

**Mechanism:** `$02DA` is the item pose flag (`WRAM.asm`): 0 is no
pose, 2 is holding an item up with two hands (the Triforce), and any
other value is holding it up with one hand. `Link_ReceiveItem` sets it
when Link gets an item, and the player draw code in `Bank0D.asm` checks
it first:

```
LDA.w $02DA : BEQ .not_in_hold_item_pose
    STZ.b $02

    LDY.b #$1E

    CMP.b #$02 : BEQ .two_hand_hold_item_pose
        LDY.b #$1D
```

The Link state for getting an item, `0x15` (`LinkState_ShowingOffItem`),
is only an `RTS`: Link is held still until the item ancilla ends it, so
setting the state alone would freeze him for good. These codes set only
the pose.

On the overworld, `Overworld_PlayerControl` (`Bank02.asm`) opens the
save menu while Select is held, starting with

```
REP #$20

LDA.w #$0186 : STA.w $1CF0

SEP #$20

LDA.b $10 : PHA
```

The first two codes change `STA.w $1CF0` to `STA.w $02DA`. A is 16-bit
there, so `$02DA` gets `$86` (one hand) and `$02DB` gets `$01`. The
last two change `LDA.b $10` to `BRA .selectButtonNotDown` (offset
`$0E`), so the save menu never opens.

For the two-hand pose, add `D4C6-0DDB` (`02A580:02`, orig `$86`). That
makes the value `$0102`.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A5 F0 29 20 F0 20 AD E8 1C 8D F4 1C C2 20 A9 86
01 8D F0 1C E2 20 A5 10 48 22 AA FD 0F` at `$02:A571` (PC `$012571`).
The pattern appears only once in the ROM. The `F0 1C` of
`STA.w $1CF0` is at `$02:A583`, and `A5 10` is at `$02:A587`. The new
`BRA` goes to `$02:A597`, `.selectButtonNotDown`. The codes decode back
to their raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Overworld only. Select no longer opens the save menu there.
- Link stays in the pose until the game clears `$02DA`, for example in
  `Link_ResetProperties_C` (when a temporary bunny turns back) or when
  he gets an item. He can still walk, but is drawn in the pose.
- While `$02DA` is set, `Sprite_CheckIfPlayerPreoccupied` says Link is
  busy, so sprites that check it may not react to him.
- `$02DB` (warp tiles off) is set to 1 too. The game clears it when Link
  is not on a warp tile.
- Uses the same bytes as the other Select cheats. Use only one of them.

## Link's sounds come from the wrong sound list

**Code:** `4E64-D7FA`
**Raw form:** `07802C:2F` (orig `$2E`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 19:55:09

Tags: `interesting` `play-start`

**Mechanism:** The game has two lists of sound effects, each played by
writing an ID to its own port: `$012E` (SFX1) and `$012F` (SFX2). The
same ID is a different sound in each list (`WRAM.asm` lists both). Link's
code in `Bank07.asm` plays most of his sounds through `Player_DoSFX2`,
which adds the stereo pan and writes `$012E`:

```
Player_DoSFX2:
{
    JSR.w Player_SetSFXPan : STA.w $012E

    RTS
}
```

The code changes the `$2E` to `$2F`, so all of these play the SFX2
sound with the same ID. About 60 calls in `Bank07.asm` use it. Some of
them:

| Link's sound | ID | Now plays |
|---|---|---|
| Landing | `0x21` | boss taking damage |
| Walking in thick grass | `0x1A` | minor puzzle solved |
| Walking in shallow water | `0x1C` | minor damage to an enemy |
| Deep water splash | `0x24` | map / text box blip |
| Turning into a bunny | `0x14` | clink |
| Turning back | `0x15` | door opening |
| Playing the flute | `0x13` | throwing an item |

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `20 36 80 8D 2D 01 60 20 36 80 8D 2E 01 60 20 36
80 8D 2F 01 60` at `$07:8021` (PC `$038021`): `Player_DoSFX1`,
`Player_DoSFX2` and `Player_DoSFX3`, each `JSR.w Player_SetSFXPan` and a
store. The `$2E` is at `$07:802C`. The code decodes back to its raw form
with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested. The new sounds are read from the ID lists in `WRAM.asm`.
- IDs above `0x34`, such as the error beep `0x3C`, are not in the SFX2
  list there, so what they play is not known.
- A sound in the wrong list can use another channel and cut off an
  enemy or item sound.
- Only Link's code is changed. Sounds that sprites, ancillae and the
  menus play are not.

## Princess Zelda is always following Link

**Codes:** `DF2E-6495` `6D24-D76F` `DD67-ADD6` `DF67-AFD6`
**Raw form:** `05DEF9:01` (orig `$05`); `00D42E:80` (orig `$F0`);
`028330:00` (orig `$0D`); `028334:01` (orig `$00`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 19:59:25

Tags: `demo-ready` `interesting` `play-start`

**Mechanism:** `$7EF3CC` is the follower (tagalong). `$01` is
Princess Zelda. The four codes each do one job:

- `DF2E-6495`: When the uncle leaves the house, `Uncle_AttachZeldaTelepathTagalong`
   (`Sprites/sprite_uncle_and_priest.asm`) does `LDA.b #$05 : STA.l $7EF3CC`.
   Follower `0x05` is Zelda's telepathy, which has no body and only
   calls to Link now and then. The code changes the `$05` to `$01`, so
   the real Zelda follows Link out into the rain.
- `6D24-D76F`: `Tagalong_LoadGfx` (`Bank00.asm`) loads Zelda's
   graphics only when the follower is `$01`:

   ```
   LDY.b #$64

   ;  If your tagalong is princess zelda...
   LDA.l $7EF3CC : CMP.b #$01 : BEQ .doDecomp
       LDY.b #$66
   ```

   The code changes the `BEQ` to `BRA`, so Zelda's graphics set `$64`
   is always loaded. `Module_LoadFile` calls this when the game is
   loaded, before the last two codes set the follower.
- `DD67-ADD6` `DF67-AFD6`: `Module_PreDungeon` (`Bank02.asm`)
   runs each time Link goes into a house, cave or dungeon, and when a
   game is loaded indoors. It throws away a saved Super Bomb:

   ```
   LDA.l $7EF3CC : CMP.b #$0D : BNE .notSuperBombTagalong
       ; If we saved the game having a super bomb, sad to say it's going to be
       ; gone.
       LDA.b #$00 : STA.l $7EF3CC
   ```

   The codes change the `#$0D` to `#$00` and the `#$00` to `#$01`.
   Now, when there is no follower, the game gives Link Zelda. The code
   after it then calls `Tagalong_Init`, which puts her next to Link.

So Zelda is with Link in his bed at the start, walks with him through
the rain, and comes back each time Link goes indoors after she has left.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has:

- `A9 05 8F CC F3 7E A9 F3 8D CD 02` at `$05:DEF8` (PC `$02DEF8`),
  the start of `Uncle_AttachZeldaTelepathTagalong`. The `$05` is at
  `$05:DEF9`. The pattern appears only once in the ROM.
- `AF CC F3 7E C9 01 F0 12` at `$00:D428` (PC `$005428`), after
  `PHB : PHK : PLB : LDY.b #$64` at `Tagalong_LoadGfx` (PC `$005423`).
  The `F0` is at `$00:D42E`.
- `AF CC F3 7E C9 0D D0 0D A9 00 8F CC` at `$02:832B` (PC `$01032B`)
  in `Module_PreDungeon`. The `$0D` is at `$02:8330` and the `$00`
  at `$02:8334`. The pattern appears only once in the ROM.

The codes decode back to their raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Without `DF2E-6495`, Zelda follows Link only until the uncle leaves the
  house. With it, Zelda's telepathy messages are lost, since they come
  from follower `0x05`.
- The game takes Zelda away when it would take the telepathy away: the
  wounded uncle in the secret passage clears the follower. She comes
  back the next time Link goes indoors from the overworld. Zelda's
  own scripts (her cell, the Sanctuary) also clear it.
- The last two codes add Zelda only when there is no follower, so they don't
  replace the old man, the Super Bomb or any other follower. They also
  stop a saved Super Bomb from being thrown away when a game is loaded.
- Starting a game on the Pyramid of Power goes to the overworld, not
  `Module_PreDungeon`, so Zelda shows up only at the next door.
- `6D24-D76F` also loads Zelda's graphics in place of set `$66` or `$59`
  when a game is loaded with another follower, so that follower may
  look wrong until its own graphics are loaded again.
- `Tagalong_TriggerData` (`Sprites/tagalong.asm`) gives Zelda lines at
  set spots in the castle, the Sanctuary and the sewers (messages
  `0x21`, `0x22`, `0x23`, `0x29`, `0x2A`). They now play when Link walks
  past those spots with her, even before he rescues her. Not checked
  whether the story events tied to her then run early.

## Every message is the Chris Houlihan message

**Codes:** `CBA3-DF7B` `99A3-DF5B` `D4A3-DF8B` `6DA3-DFEB` `D4A3-D47B`
(use all five)
**Raw form:** `0EC4E4:A9` (orig `$AD`); `0EC4E5:55` (orig `$F0`);
`0EC4E6:02` (orig `$1C`); `0EC4E7:80` (orig `$0A`); `0EC4E8:02`
(orig `$6D`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 20:08:21

Tags: `interesting` `play-start`

**Mechanism:** Message `0x00C7` (`Message_00C7` in `Bank1C.asm`) is the
one in the hidden Chris Houlihan room: "My name is Chris Houlihan. This
is my top secret room. Keep it between us, OK?"

`$1CF0` holds the number of the message to show. `Text_LoadCharacterBuffer`
(`Bank0E.asm`) looks up where the message is stored. Each entry in the
pointer table at `$7F71C0` is 3 bytes:

```
REP #$30

; X = $1CF0 * 3
LDA.w $1CF0 : ASL : ADC.w $1CF0 : TAX

; Load the address for the text's data from WRAM.
LDA.l $7F71C0, X : STA.b $04
```

The codes rewrite the first line, `AD F0 1C 0A 6D F0 1C AA`, as
`A9 55 02 80 02 F0 1C AA`: `LDA.w #$0255 : BRA` to the `TAX`. `$0255` is
`0x00C7 * 3`, so every message the game shows (NPCs, signs, Zelda's
telepathy, item messages) is read from the Houlihan message's entry.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `C2 30 AD F0 1C 0A 6D F0 1C AA BF C0 71 7F 85 04` at
`$0E:C4E2` (PC `$0744E2`), the start of `Text_LoadCharacterBuffer`.
The pattern `AD F0 1C 0A 6D F0 1C AA` appears only once in the ROM, at
`$0E:C4E4`. The `BRA` at `$0E:C4E7` with offset `$02` goes to the `AA`
at `$0E:C4EB`. The codes decode back to their raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Use all five codes. With only some of them, the game runs a broken
  instruction.
- Messages that ask a question lose their choices. What answer the game
  then takes was not checked.
- The game keeps going on the message number, not the text, so story
  events still happen. Only what is shown changes.
- Not checked whether the opening story and the end credits use this
  routine.

## Link picks up pots and bushes with almost no delay

**Codes:** `DF8B-049A` `DF8B-04BA` `DF8B-042A`
**Raw form:** `07B199:01` (orig `$06`); `07B19A:01` (orig `$07`);
`07B19B:01` (orig `$07`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 20:15:11

Tags: `demo-ready` `interesting` `play-start`

**Mechanism:** When Link lifts a pot, bush, sign or small rock, he
kneels, then raises it in three steps. Link can't move while he does
this. The length of each step is in `Pool_LinkToss_timer_a`
(`Bank07.asm`):

```
.timer_a
db $06, $07, $07
```

`Link_Lift` starts the lift with `LDA.w Pool_LinkToss_timer_a : STA.w
$030B`. Each frame, `Link_APress_LiftCarryThrow` does `DEC.w $030B`.
When it reaches 0, the next step (`$030A`) starts with the next timer,
and after step 3 the lift is done:

```
LDX.w $030A : INX
LDA.w Pool_LinkToss_timer_a, X : STA.w $030B

STX.w $030A : CPX.b #$03 : BNE .return
```

So a lift takes 6 + 7 + 7 = 20 frames. The codes set all three steps
to 1, so it takes 3 frames.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `06 07 07` at `$07:B199` (PC `$03B199`), then the
`timer_b` row `05 0A 00 17`. The table is read in two places:
`AD 99 B1 8D 0B 03` (`LDA.w $B199 : STA.w $030B`) at `$07:B175` in
`Link_Lift`, and `BD 99 B1 8D 0B 03` (`LDA.w $B199, X : STA.w $030B`) at
`$07:B26A` in `Link_APress_LiftCarryThrow`. The codes decode back to
their raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Heavy rocks (`$0368` = 5 or 6) use a different table,
  `.heavy_lift_animation_timers`, so they still take as long.
- The lift poses still play, one frame each.
- Throwing is not changed. It uses `.timer_b`.

## Link lifts the sprite in the slot before the one he touches

**Code:** `3CE5-6D96`
**Raw form:** `06F271:EA` (orig `$1A`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 20:17:45

Tags: `interesting` `play-start`

**Mechanism:** `$0314` holds the slot index + 1 of the liftable sprite
Link is touching (0 if none). Each frame, a liftable sprite (chicken,
fish, bush crab, a frozen enemy, an enemy bomb, a thrown object after
it lands) runs `Sprite_CheckIfLiftedPermissive` (`Bank06.asm`):

```
; Check if the current sprite is the same one Link is touching.
LDA.w $02F4 : DEC : CMP.w $0FA0 : BEQ .player_picks_up_sprite
    ...
    JSR.w Utility_CheckIfHitBoxesOverlap : BCC .return
    TXA : INC : STA.w $0314
                STA.w $0FB2
```

When Link presses A, `Link_HandleAPress` (`Bank07.asm`) copies
`$0314` to `$02F4`, and `Link_Lift` starts the lift pose. The sprite
whose slot (`$0FA0`) equals `$02F4` - 1 then puts itself in Link's
hands.

The code changes the `INC` to `NOP`, so `$0314` gets the slot index
without the + 1. Link is now "touching" the sprite one slot lower, and
on A that sprite flies into his hands from wherever it is.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `AD 72 03 D0 4D AD F4 02 3A CD A0 0F F0 14` at
`$06:F257` (PC `$037257`), which is `LDA.w $0372 : BNE : LDA.w $02F4 :
DEC : CMP.w $0FA0 : BEQ`. The `DEC` (`3A`) is at `$06:F25F`. After the
three `JSR`s comes `90 39 8A 1A 8D 14 03` (`BCC : TXA : INC : STA.w
$0314`), with the `INC` (`1A`) at `$06:F271`. `$0FB2` gets the same
value, but nothing reads it (`General/WRAM.asm`). The code decodes back
to its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Only sprites that call this routine can be lifted. If the slot below
  holds something else, or is empty, Link does the lift pose and holds
  nothing.
- A sprite in slot 0 now sets `$0314` to 0, so it can't be lifted.
- `$02F4` stays set until the next A press. A liftable sprite that later
  appears in that slot may jump into Link's hands without him touching
  it.
- Pots, bushes and rocks on the ground are tiles, not sprites, and are
  not changed.
- Don't use with "Link lifts the sprite in the slot after the one he
  touches". Together they cancel out.

## The Lamp does a spin attack

**Codes:** `0AC5-6F2A` `5CC5-64FA` `C5C5-649A`
**Raw form:** `07A277:4C` (orig `$9C`); `07A278:7A` (orig `$01`);
`07A279:A7` (orig `$03`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 20:19:51

Tags: `interesting` `play-start`

**Mechanism:** `LinkItem_Lamp` (`Bank07.asm`) runs when Link presses Y
with the Lamp. After it makes the flame, it clears Link's item state:

```
    LDY.b #$02
    LDA.b #$2F
    JSL.l AddLampFlame

.cant_use_lamp

STZ.w $0301
STZ.b $3A
STZ.b $3C
STZ.b $50
```

The codes turn `STZ.w $0301` into `JMP.w Link_ActivateSpinAttack`
(`$07:A77A`). That is the routine the sword code jumps to when Link
lets go of a charged B. It adds the spin sparkle, puts Link in mode
`0x03` (`LinkState_SpinAttack`), and sets `$3C` to `$90`, `$3A` to
`$80` and bit 0 of `$50` itself. So the stores it skips are not needed.
Each Y press with the Lamp lights the flame, then Link spins.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `22 1C 8F 09` (`JSL AddLampFlame`) then `9C 01 03 64
3A 64 3C 64 50` at `$07:A277` (PC `$03A277`), the only match in the ROM.
`Link_ActivateSpinAttack` starts with `A0 00 BB A9 2A 22` (`LDY.b #$00 :
TYX : LDA.b #$2A : JSL`) at `$07:A77A`. Both are in bank `$07`, so a
`JMP` reaches it. The codes decode back to their raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- `.cant_use_lamp` is also reached when Link has too little magic, so
  he still spins with no flame.
- Before Link has the sword, the spin pose shows no sword. Whether it
  hurts enemies then was not checked.
- Once Link has the sword, `LinkState_Default` runs `Player_Sword` right
  after the Lamp. With B not held, it goes to
  `Link_ResetSwordAndItemUsage`, which clears `$3C`. Link still spins,
  but the spin hit box needs `$3C` to be negative, so the spin may not
  hurt enemies.
- Works with "The Lamp throws the boomerang, shoots an arrow, pounds the
  hammer and sprinkles powder" (`alttp_game_genie_long_codes.md`). Those
  codes change `$07:A26B`-`$07:A275`, not these bytes.

## Bushes, small rocks and signs stay in place after Link lifts them (overworld)

**Code:** `8EAF-D0CA`
**Raw form:** `1BC012:BF` (orig `$9F`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 20:21:08

Tags: `demo-ready` `video-ready` `interesting` `play-start`

**Mechanism:** When Link lifts something on the overworld, `Link_Lift`
(`Bank07.asm`) calls `Overworld_LiftableTiles` (`Bank1B.asm`). For a
green or brown bush, a small rock or a sign, it picks the tile that is
left behind (`$0DC7`, `$0DC8`, `$0DCA` or `$0DC6`), then writes it to
the map:

```
.liftingSmallObject

STY.b $0E

PHA

JSR.w Overworld_RevealSecret : BCS .noSecret
    LDA.b $0E

.noSecret

STA.l $7E2000, X

JSL.l Overworld_Memorize_Map16_Change
JSL.l Overworld_DrawMap16
```

The code changes `STA.l $7E2000, X` to `LDA.l $7E2000, X`. The map is
not changed, and A now holds the tile that is already there, so
`Overworld_DrawMap16` draws the same bush again. `Link_Lift` still
spawns the bush sprite in Link's hands, since it uses the tile
attribute that the routine returns from the tile on the stack. So Link
can lift the same bush again and again.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A0 C6 0D 84 0E 48 20 A4 C8 B0 02 A5 0E 9F 00 20 7E
22` at `$1B:C005` (PC `$0DC005`): `LDY.w #$0DC6 : STY.b $0E : PHA : JSR
Overworld_RevealSecret : BCS : LDA.b $0E : STA.l $7E2000, X : JSL`. The
`STA.l` opcode (`9F`) is at `$1B:C012`. `BF` is `LDA.l` long,X, with
the same operand. The code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Overworld only. Indoors, pots use `Dungeon_RevealCoveredTiles`.
- Heavy rocks use `Overworld_SmashRockPile_isRockPile` and still go
  away.
- Dashing into a bush also reaches this store, through
  `Overworld_SmashRockPile_checkForBush`. Not checked whether the sword
  does.
- `Overworld_RevealSecret` still runs each time, so a hidden item under
  a bush may come out every time Link lifts it. A hidden hole or stairs
  is not drawn.
- `Overworld_Memorize_Map16_Change` still adds an entry to the list at
  `$7EF800`/`$7EFA00` for each lift, and nothing here limits its
  length. Lifting the same bush many times in one area may overrun it.

## Link walks in his praying pose

**Codes:** `84BE-6D75` (up) `84B0-AD75` (down) `84BB-AD75` (left)
`84B3-AD75` (right)
**Raw form:** `0D9EF0:B2` (orig `$00`); `0D9F40:B2` (orig `$09`);
`0D9F90:B2` (orig `$12`); `0D9FE0:B2` (orig `$1B`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 20:22:13

Tags: `demo-ready` `interesting` `play-start`

**Mechanism:** Link's draw code (`Bank0D.asm`) picks a pose in Y and a
step in `$02`. For walking, the pose is `0x00` and the step is the walk
step `$2E`. It then looks up where that pose starts for the way Link is
facing:

```
; Y is the "pose", then add the offset for the particular direction
; we're facing.
TYA : AND.w #$00FF : ASL : CLC : ADC.b $04 : TAY

; $02 is probably a subpose index...
LDA.w PlayerOam_AnimationStepDataOffsets, Y
CLC : ADC.b $02 : STA.b $02
```

`PlayerOam_AnimationStepDataOffsets` has one row per direction. Its
`0x00 - Walking` entries are `$0000`, `$0009`, `$0012` and `$001B`.
Its `0x0B - Prayer` entries are all `$00B2`, the pose Link uses at the
Desert Palace. The codes set each walking entry's low byte to `$B2`, so
the walk cycle steps through the prayer frames. Each code works alone
for one direction.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has the table at `$0D:9EF0` (PC `$069EF0`), `00 00 24 00
48 00 60 00`. The direction rows are `$50` bytes apart
(`PlayerOam_AnimationDirectionalStepIndexOffset`: `$0000`, `$0050`,
`$00A0`, `$00F0`), so the walking entries are at `$0D:9EF0` (`00 00`),
`$0D:9F40` (`09 00`), `$0D:9F90` (`12 00`) and `$0D:9FE0` (`1B 00`). The
prayer entry for up is `B2 00` at `$0D:9F06`. The codes decode back to
their raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- The walking rows are 9 steps long. The prayer row starts at `$B2` and
  the left shovel row at `$B6`, so the later walk steps likely show
  shovel or other frames.
- Pose `0x00` is also used when Link stands still, so he stands in a
  prayer frame too.
- Bunny Link uses pose `0x21` and is not changed.

## Bunny Link keeps all of Link's abilities (Dark World without the Moon Pearl)

**Codes:** `DB61-D72A` `6F65-DDFA` `6DBF-A493`
**Raw form:** `07806F:09` (orig `$A1`); `078070:81` (orig `$83`);
`079B19:80` (orig `$F0`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 20:23:44

Tags: `demo-ready` `interesting`

**Mechanism:** When Link enters the Dark World without the Moon Pearl,
the game sets `$02E0` and `$56` to 1 (bunny graphics) and Link mode
`$5D` to `0x17`. Each frame, `Bank07.asm` runs the mode's handler with
`JMP.w ($8041, X)`. Mode `0x17` is `LinkState_Bunny`, which moves Link
but never calls the sword or A button code. The first two codes change
that table entry from `LinkState_Bunny` (`$83A1`) to the entry of mode
`0x00`, `LinkState_Default` (`$8109`):

```
dw LinkState_Bunny                   ; 0x17 - $83A1 Permabunny mode
```

So the bunny now runs Link's normal code: sword, A button (lift, read,
talk, open chests, dash) and Y items. `Link_HandleYItem` still blocks Y
items for a bunny, except the Bottle (`$0303` = `$0B`) and the Mirror
(`$14`):

```
LDA.w $02E0 : BEQ .BRANCH_BETA
    LDA.w $0303 : CMP.b #$0B : BEQ .BRANCH_BETA
                  CMP.b #$14 : BEQ .BRANCH_BETA
        RTS
```

The third code changes that `BEQ` to `BRA`, so all items work.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has the mode table at `$07:8041` (PC `$038041`),
starting `09 81` (`LinkState_Default`, `$8109`). Entry `0x17` is at
`$07:8041` + `$2E` = `$07:806F`, `A1 83`. Both handlers start with `20 14
F5` (`JSR CacheCameraPropertiesIfOutdoors`). `Link_HandleYItem` has `AD
E0 02 F0 0C AD 03 03 C9 0B F0 05 C9 14 F0 01` at `$07:9B16`, with the
`BEQ` (`F0`) at `$07:9B19`. Its `+$0C` lands on the same place as
`BRA` would. The codes decode back to their raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Only the Dark World bunny (mode `0x17`). The temporary bunny (mode
  `0x1C`, `LinkState_TemporaryBunny`) runs its own code and then falls
  into `LinkState_Bunny` directly, not through the table.
- The bunny graphics stay loaded. Sword and item poses will likely look
  broken.
- After recoil or a fall, the game jumps to `LinkState_Bunny` directly
  for one frame, then the table is used again.
- Other `$02E0` checks remain: for example, the bunny still can't swim
  with the Flippers.

## Enemies don't knock Link back

**Codes:** `DDE6-672B` `DFEB-6D2B`
**Raw form:** `06F68F:00` (orig `$27`); `06F693:01` (orig `$28`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 20:24:53

Tags: `demo-ready` `interesting`

**Mechanism:** When a sprite hurts or bumps Link, the game calls
`Sprite_ApplyRecoilToPlayer` (`Bank06.asm`) with a speed in A. It
aims that speed away from the sprite and stores it as Link's recoil
speed, `$27` (Y) and `$28` (X), plus a hop speed in `$29`:

```
PHA

JSR.w Sprite_ProjectSpeedTowardsPlayer

LDA.b $00 : STA.b $27
LDA.b $01 : STA.b $28

PLA : LSR : STA.b $29
            STA.b $C7
```

The codes change `STA.b $27` to `STA.b $00` and `STA.b $28` to `STA.b
$01`, so each value is written back where it came from. `$27` and `$28`
keep their old values, and the end of each recoil in
`Link_HandleRecoilAndTimer` (`Bank07.asm`) sets them to 0 with `STZ.b
$27 : STZ.b $28`. So Link still takes damage, flashes and does his small
hop, but he isn't pushed.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `48 20 91 E9 A5 00 85 27 A5 01 85 28 68 4A 85 29` at
`$06:F688` (PC `$037688`). The `A5 00 85 27 A5 01 85 28 68 4A 85 29` run
is found only once in the ROM. The `$27` operand is at `$06:F68F`, the
`$28` operand at `$06:F693`. The code has 7 callers in `Bank06.asm`
(normal enemy damage, the shield and boss bumps, Moldorm, the soldier
sword parry) and one in `Sprites/sprite_octoballoon.asm`. The codes
decode back to their raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Link is still stunned for the normal time (`$46`). "Link is never
  stunned" is a separate item.
- The Beamos laser sets `$27` and `$28` itself right after, so it still
  pushes Link.
- Bonking into walls while dashing uses its own speeds
  (`RepelBonkSpeedY`, `Bank07.asm`) and is not changed.
- If something else left `$27` or `$28` nonzero, a hit would push Link
  by that old speed.

## Enemy hits don't stun Link

**Codes:** `DDE3-AFF6` `DDE3-A796`
**Raw form:** `06F3E4:00` (orig `$13`); `06F3ED:00` (orig `$01`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 20:26:18

Tags: `interesting`

**Mechanism:** When an enemy or its shot touches Link,
`Sprite_AttemptDamageToPlayerPlusRecoil` (`Bank06.asm`) sets the
damage, and also sets `$46` (Link can't move for this many frames) and
`$4D` (Link is in the air from a hit):

```
LDA.w $031F : ORA.w $037B : BNE .dont_damage_player
    LDA.b #$13 : STA.b $46

    LDA.b #$18
    JSR.w Sprite_ApplyRecoilToPlayer

    LDA.b #$01 : STA.b $4D

    ; Determine damage for Link based on his armor value.
```

`LinkState_Default` (`Bank07.asm`) puts Link in mode `0x02`
(`LinkState_Recoil`) when `$4D` is set, and skips his input while
`$46` is set. The codes store 0 in both instead. Link still loses
hearts: `Link_ControlHandler` takes the damage from `$0373` in every
mode and starts the blink timer `$031F`, and the `$031F` check above
keeps him from being hit again while he blinks. But he keeps walking,
swinging and using items.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `AD 1F 03 0D 7B 03 D0 3B A9 13 85 46 A9 18 20 88 F6
A9 01 85 4D` at `$06:F3DB` (PC `$0373DB`), the only match in the ROM.
The `#$13` is at `$06:F3E4` and the `#$01` at `$06:F3ED`. The codes
decode back to their raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Only hits through this routine. The shield and boss bumps, Moldorm and
  the soldier sword parry in `Sprite_CheckDamageFromPlayer` set `$46`
  themselves and still stun Link. Hazards like spikes and falls were not
  checked.
- `Sprite_ApplyRecoilToPlayer` still sets the recoil speeds `$27`/`$28`.
  They are only cleared when a recoil ends, so the next real recoil may
  start with them. Use with "Enemies don't knock Link back" to stop
  that.

## Link can steer while dashing with the Pegasus Boots

**Codes:** `6DB4-0D2A` `4DB4-0FFA`
**Raw form:** `079123:80` (orig `$C5`); `079124:20` (orig `$00`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 20:27:45

Tags: `demo-ready` `video-ready` `curated` `interesting`

**Mechanism:** Each frame of a dash, `LinkState_Dashing` (`Bank07.asm`)
gets the way Link faces (`$2F`) as a joypad direction in `$00`. If a
different direction is held, the dash ends:

```
LDA.b $2F : LSR : TAX
LDA.w LinkDirections26, X : STA.b $00

LDA.b $F0 : AND.b #$0F : BEQ .BRANCH_PSI
            CMP.b $00  : BEQ .BRANCH_PSI
    ; Come out of the dashing submode.
    LDA.b #$12 : STA.b $5D
    ...
.BRANCH_PSI

LDA.b $49 : AND.b #$0F : BNE .BRANCH_OMEGA
    LDA.b $2F : LSR : TAX
    LDA.w LinkDirections26, X

.BRANCH_OMEGA

STA.b $67 : STA.b $26
```

The codes change `CMP.b $00` to `BRA .BRANCH_OMEGA`. When a direction
is held, A still holds it, so it goes into `$67`/`$26` and Link dashes
that way. `Link_HandleMovingAnimation_MainEntry` then sets `$2F` from
`$26` (when `$50` is 0), so Link faces the new way and keeps dashing
that way after the button is let go.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A5 F0 29 0F F0 15 C5 00 F0 11 A9 12 85 5D` at
`$07:911D` (PC `$03911D`), the only match in the ROM. Both `BEQ`s land
on `$07:9138` (`A5 49 29 0F D0 07`, `.BRANCH_PSI`), whose `BNE` lands on
`$07:9145` (`85 67 85 26`, `.BRANCH_OMEGA`). The `CMP` is at
`$07:9123`, so `80 20` there branches to `$07:9125` + `$20` =
`$07:9145`. The codes decode back to their raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Holding a different direction was the normal way to stop a dash. Now
  only hitting something stops it.
- Holding a diagonal makes Link dash diagonally.
- If `$50` is set (for example, by a sword charge held from before the
  dash), Link moves the new way but keeps his old facing, and the dash
  goes back to the old way when the button is let go.

## Pits don't pull Link toward them

**Code:** `54BD-D722`
**Raw form:** `07940F:72` (orig `$67`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 20:29:31

Tags: `interesting`

**Mechanism:** When part of Link is over a hole, he is in mode `0x01`,
`LinkState_Pits` (`Bank07.asm`). `$59` holds which of his corners are
over the hole, and `$67` holds the joypad direction he is moving. The
routine picks a row `X` from `$59` and, unless Link is already moving
toward the hole, adds the direction toward it to `$67`:

```
.BRANCH_CHI

LDA.b $67 : STA.b $00

LDX.w $02C9
LDA.w PitFall_Flags_ForAnimation, X : TSB.b $67
```

`Link_HandleVelocity` then moves Link by `$67`, so he slides into the
hole. The code changes `TSB.b $67` to `TSB.b $72`. `$72` is general
scratch RAM, and every reader of it writes it first (see "Scratch RAM
`$72`-`$75`" in the project's `NOTES.md`). So the pull goes nowhere,
and Link only moves the way the player pushes.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A5 67 85 00 AE C9 02 BD BF 92 04 67 A5 00 F0` at
`$07:9404` (PC `$039404`), the only match in the ROM. `BD BF 92` is
`LDA.w $92BF, X`, and `PitFall_Flags_ForAnimation` is at PC `$0392BF`.
The `TSB` operand (`67`) is at `$07:940F`. The code decodes back to its
raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Walking toward the hole still moves Link at the slow `$5E` = `0x06`
  speed, and once he is over it, he still falls.
- Only the hole tiles that put Link in mode `0x01` are changed.

## Link's tile collision always uses one layer (dungeons)

**Code:** `6D2E-07B3` (always the BG2 layer)
**Alternative:** use `DD2E-0723` instead for always the BG1 layer.
Don't use both.
**Raw form:** `07D9FE:80` (orig `$F0`); alternative `07D9FF:00` (orig
`$04`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 20:31:35

Tags: `interesting`

**Mechanism:** Indoors, each tile has an attribute byte (wall, floor,
pit, stairs, water and so on). Rooms with two layers keep one map of
attributes per layer: BG2's at `$7F2000` and BG1's at `$7F3000`.
`$EE` says which layer Link is on. `TileDetect_Execute` (`Bank07.asm`)
picks the map from it:

```
; Which part of a two level room is Link on.
LDX.b $EE : BEQ .lowerFloor
    ; He's on the upper floor then.
    ; Add this offset in because BG0's tile attributes start at $7F3000.
    CLC : ADC.w #$1000

.lowerFloor

REP #$10

TAX

; Are we figuring out what sort of tile this is.
LDA.l $7F2000, X : PHA
```

The main code changes `BEQ` to `BRA`, so Link's tiles always come from
`$7F2000`. The alternative changes the `BEQ` offset to 0, so the `ADC`
always runs and the tiles always come from `$7F3000`. Link is still
drawn on his real layer (`$EE` is not changed). He just touches the
other layer's walls, floors, pits and stairs.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A6 EE F0 04 18 69 00 10 C2 10 AA BF` at `$07:D9FC`
(PC `$03D9FC`): `LDX.b $EE : BEQ +4 : CLC : ADC.w #$1000 : REP #$10 :
TAX : LDA.l`. The `BEQ` is at `$07:D9FE` and its offset at `$07:D9FF`.
The same bytes are also at `$02:C203` and `$07:E038`
(`PushBlock_GetGeneralizedTileInteractionOfTarget`); those are not
changed. The codes decode back to their raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Indoors only. `TileDetect_Execute` uses
  `TileDetection_Execute_overworld` outdoors.
- `General/WRAM.asm` and the comment above disagree on which value of
  `$EE` is the upper level, so this entry names the layers by BG.
- In rooms with one layer, the BG1 map may be empty or left over from
  an earlier room. Not checked. With the alternative code, Link may
  walk through walls there or get stuck.
- Blocks, sprites and other objects use their own tile checks.

## Link holds up rupees like a treasure

**Codes:** `5629-0FF6` `5629-0FB6` `5629-04F6` `7025-07F6`
**Optional:** add `5629-0DB6` so hearts are held up too (they then give a
green rupee instead of a heart).
**Raw form:** `06D154:78` (orig `$BE`), `06D156:78` (orig `$BE`),
`06D158:78` (orig `$BE`), `06D17C:34` (orig `$32`); optional
`06D152:78` (orig `$AF`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 21:17:09

Tags: `interesting`

**Mechanism:** When Link touches a pickup sprite,
`Sprite_HandleAbsorptionByPlayer` (`Sprites/sprite_absorbable.asm`)
jumps through a table by sprite type. Rupees normally go to a handler
that just adds to `$7EF360`. Only the Big Key's handler calls
`Link_ReceiveItem`, which makes Link hold the item up:

```
dw HeartRefill_AbsorptionByPlayer      ; 0x00 - $D1AF
dw GreenRupee_AbsorptionByPlayer       ; 0x01 - $D1BE
dw BlueRupee_AbsorptionByPlayer        ; 0x02 - $D1BE
dw RedRupee_AbsorptionByPlayer         ; 0x03 - $D1BE
...

BigKey_AbsorptionByPlayer:
    STZ.w $02E9
    LDY.b #$32
    PHX
    JSL.l Link_ReceiveItem
    PLX
    BRA Key_AbsorptionByPlayer_set_event_flag
```

The first three codes point the three rupee entries at
`BigKey_AbsorptionByPlayer` ($D178). The fourth changes its item from
`$32` (Big Key) to `$34` (green rupee). Every rupee Link picks up is
then held up over his head like a chest item, and gives 1 rupee (the
item's own amount) instead of 1, 5 or 20. The optional code does the
same for hearts.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has the table at `$06:D152` (PC `$035152`):
`AF D1 BE D1 BE D1 BE D1 ...`, and `9C E9 02 A0 32 DA 22 AD 99 07 FA`
at `$06:D178`, so the `#$32` is at `$06:D17C`. `Link_ReceiveItem` is
at `$07:99AD`; with `$02E9` = 0 it puts Link in mode `0x15` (holding up
the item). Item `$34` is the green rupee chest item. The codes decode back
to their raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Big Keys that lie on the floor as sprites now give a green rupee
  instead of the Big Key. Big Keys from chests are not changed.
- Rupees now run the Big Key handler's tail
  (`Key_AbsorptionByPlayer_set_event_flag`). In dungeons it sets one of
  the room's key flags in `$0403` from the rupee's `$0CBA` (0 for most
  rupees, so the first key flag). A small key that has not been taken in
  that room may vanish. It also marks the rupee's sprite as dead with
  `Dungeon_ManuallySetSpriteDeathFlag`.
- Each rupee stops Link for the hold-up, so rupee-heavy places (enemy
  drops, the rupee rooms) are slow.

## Sprites are always drawn in front of the background

**Codes:** `7D68-D49B` `7D68-D4BB`
**Raw form:** `0684B9:30` (orig `$20`), `0684BA:30` (orig `$10`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 21:17:53

Tags: `interesting`

**Mechanism:** Each sprite's OAM priority bits (bits 4 and 5 of
`$0B89`) say which background layers it is drawn over. Every frame,
`Sprite_TimersAndOAM` (`Bank06.asm`) sets them from the sprite's floor
(`$0F20`), or uses the top priority when Link uses "super priority"
(`$EE` = 3):

```
LDY.b $EE : CPY.b #$03 : BEQ .player_using_super_priority
    LDY.w $0F20, X

.player_using_super_priority

LDA.w $0B89, X : AND.b #$CF : ORA .priority, Y : STA.w $0B89, X

RTS

.priority
db $20, $10, $30, $30
```

The codes set the floor 0 and floor 1 entries to `$30`, the top
priority. Enemies, NPCs, pickups and other sprites are then drawn in
front of every background layer: they show over tree tops, roofs,
bridges, walls and other things they normally walk behind.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A4 EE C0 03 F0 03 BC 20 0F BD 89 0B 29 CF 19 ..
.. 9D 89 0B 60 20 10 30 30` at `$06:84A4` (PC `$0304A4`). The table is at
`$06:84B9` (`20 10 30 30`). The codes decode back to their raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Sprites only. Link, ancillae (arrows, bombs, the boomerang) and
  garnish use their own priority. Some garnish copy `$0B89` from their
  sprite and change too.
- Sprites that set their own priority after this routine (for example
  Arrghus, which ORs in `$30`) are not changed.
- Sprites can still be hidden behind other sprites or Link, by OAM
  order.

## The Pegasus Boots dash starts at once

**Code:** `DD8B-64BA`
**Raw form:** `07B29A:00` (orig `$1D`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 21:21:16

Tags: `demo-ready` `video-ready` `curated` `interesting`

**Mechanism:** When Link starts a dash, `Link_PerformDash`
(`Bank07.asm`) sets the charge timer `$0374` to `0x1D`. While it counts
down, `LinkState_Dashing` keeps Link running in place and kicking up dust.
He moves only once it goes below 0:

```
LDA.b #$1D : STA.w $0374 ; Dash timer
...
DEC.w $0374 : BPL .BRANCH_LAMBDA
    STZ.w $0374
    ...
    BRL .BRANCH_SIGMA
```

The code sets the timer to 0. The first `DEC` takes it below 0, so Link
dashes off on the first frame with no wind-up.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A9 1D 8D 74 03 A9 40 8D F1 02` at `$07:B299`
(PC `$03B299`): `LDA.b #$1D : STA.w $0374 : LDA.b #$40 : STA.w $02F1`.
That is the only place the game loads `0x1D` into `$0374`. The code
decodes back to its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Needs the Pegasus Boots, or the "Dash without the Pegasus Boots" code.
- The wind-up sound and dust are skipped. A quick tap of A, which
  normally cancels the dash during the wind-up, now dashes.
- With no wind-up, the follower check that runs when the dash starts
  (`FollowerReactionToDashing`) still runs, on the first frame.

## Every enemy electrocutes Link

**Codes:** `44E3-0D26` `3AE3-0FF6`
**Raw form:** `06F1E3:22` (orig `$DB`), `06F1E4:EC` (orig `$F3`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 21:22:27

Tags: `demo-ready` `interesting`

**Mechanism:** When a sprite touches Link, `Sprite_CheckDamageToPlayer`
(`Bank06.asm`) calls `Sprite_AttemptDamageToPlayerPlusRecoil`. Bari,
Biri, Buzzblobs and Agahnim instead go through `Sprite_AttemptZapDamage`,
whose last part sets `$0360`. When `$0360` is set, `Link_ControlHandler`
(`Bank07.asm`) puts Link in the electrocution mode `0x07` instead of
the normal recoil:

```
Sprite_AttemptZapDamage:
    ...
    .attempt_electrocution

    ; But if the sprite's not active, it can't electrocute.
    LDA.w $0DD0, X : CMP.b #$09 : BNE .cant_electrocute
        LDA.w $031F : BNE .player_blinking_invulnerable
            LDA.b #$40 : STA.w $0E00, X
                         STA.w $0360

            JSR.w Sprite_AttemptDamageToPlayerPlusRecoil
```

The codes change `JSR.w Sprite_AttemptDamageToPlayerPlusRecoil` (`$F3DB`)
in `Sprite_CheckDamageToPlayer` to `JSR $EC22`
(`Sprite_AttemptZapDamage_attempt_electrocution`). Every enemy, and every
enemy shot, that touches Link now zaps him: his skeleton flashes, he
stands still for a moment and takes the enemy's normal damage.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `20 DB F3 BD 20 0E C9 0C D0 03` at `$06:F1E2` (PC
`$0371E2`): `JSR $F3DB : LDA.w $0E20, X : CMP.b #$0C`. `$06:EC22` holds
`BD D0 0D C9 09 D0 11 AD 1F 03 D0 0B A9 40 9D 00 0E 8D 60 03 20 DB`,
the start of `.attempt_electrocution`, which ends with the same `JSR
$F3DB`. The codes decode back to their raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Only hits checked by `Sprite_CheckDamageToPlayer` change. Sprites that
  call `Sprite_AttemptDamageToPlayerPlusRecoilLong` directly (the
  Guruguru bar, Helmasaur King) still knock Link back normally.
- It also sets the touching sprite's `$0E00` timer to `0x40`, as the
  Bari's zap does. Enemies that use `$0E00` for their own timing may
  act oddly after a hit.
- Not checked whether Link is still knocked back after the zap. If he
  is not, he can be zapped again once his blinking ends.

## Every enemy moves the sprite in the next slot sideways instead of itself

**Codes:** `FF37-0F9C` `FFEE-0F21` `FFE4-A1E8` `FF39-0981`
**Raw form:** `06E935:11` (orig `$10`), `05F9F7:11` (orig `$10`),
`1EFF2B:11` (orig `$10`), `1DE956:11` (orig `$10`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 22:32:35

Tags: `interesting`

**Mechanism:** This is the ALttP version of the SMW code "Every enemy
steals the horizontal speed of the sprite in the next slot". Each sprite
bank has its own copy of the move routines: `Sprite_MoveHoriz`
(`Bank06.asm`), `Sprite2_MoveHoriz` (`Bank05.asm`),
`Sprite3_MoveHoriz` (`Bank1E.asm`) and `Sprite4_MoveHoriz`
(`Bank1D.asm`). The sprite tables are 16 bytes each (`$0D00` Y low,
`$0D10` X low, `$0D20` Y high, `$0D30` X high, `$0D40` Y speed,
`$0D50` X speed), so the horizontal routine adds `$10` to the slot in
X and reuses the vertical routine on the X tables:

```
Sprite_MoveHoriz:
    ; Do X position adjustment.
    TXA : CLC : ADC.b #$10 : TAX    <- $10 patched to $11
    JSR.w Sprite_MoveVert
    LDX.w $0FA0
    RTS
```

The codes change the `$10` to `$11` in all four copies. Now X points
one slot further along the tables, so each sprite's horizontal move is
applied to the **next sprite slot up**: it reads that sprite's X speed
and changes that sprite's X position. The sprite's own X position is
only changed when the sprite in the slot below it moves.
`Sprite_Main` runs the slots from `0x0F` down to `0x00`. So an enemy
whose slot below is empty or still can't move sideways (it can still move
up and down), and an enemy above a moving sprite moves sideways normally.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`):

- `$06:E932` (PC `$036932`): `8A 18 69 10 AA 20 3E E9 AE A0 0F 60`
- `$05:F9F4` (PC `$02F9F4`): `8A 18 69 10 AA 20 00 FA AE A0 0F 60`
- `$1E:FF28` (PC `$0F7F28`): `8A 18 69 10 AA 20 34 FF AE A0 0F 60`
- `$1D:E952` (PC `$0EE952`): `DA 8A 18 69 10 AA 20 5D E9 FA 60`
  (`PHX` first, so the operand is one byte later)

Each `JSR` goes to that bank's `MoveVert` (`$E93E`, `$FA00`,
`$FF34`, `$E95D`). The byte at each code's address was read back as
`$10`. The codes decode back to their raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Slot `0x0F` has no next slot: X becomes `$20`, so it reads slot 0's
  `$0D60` as a speed and changes slot 0's Y high byte (`$0D20`) and Y
  speed (`$0D40`). Slot 0 may jump up or down a whole screen. This
  can despawn sprites or lock up the game.
- A sprite moving an empty slot changes that slot's leftover position.
  This should be harmless, since a new sprite sets its own position.
- Sprite code that sets X positions directly, without the move routines,
  is not changed.
- A Y-only version was not possible with few codes: the same
  `MoveVert` routine moves both axes, and only the horizontal routine
  has an offset to change.

## Good bees keep coming out and attack enemies

**Codes:** `AE61-ADB6` `2A61-AD26` `F361-AFF6` `DD2D-0558`
**Raw form:** `068362:CF` (orig `$6E`), `068363:DC` (orig `$B0`),
`068364:1E` (orig `$09`), `1EDD0D:00` (orig `$05`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 23:35:23

Tags: `demo-ready` `interesting` `play-start`

**Mechanism:** `PlayerItem_ReleaseBee` (`Sprites/sprite_bees.asm`) is
what the bottle calls to let a bee out. It spawns sprite `0xB2` next to
Link and makes it a good bee only if the bottle holds one:

```
        LDA.l $7EF35B, X : CMP.b #$08 : BNE .not_good_bee   <- offset $05 patched to $00
            LDA.b #$01 : STA.w $0EB0, Y
        .not_good_bee
```

The fourth code sets that branch offset to `$00`, so every bee it
releases is a good bee (`$0EB0` = 1). In `GoodBee_Activated`, only a
good bee runs the attack part of `GoodBee_ScanForTargetableSprites`.
It flies to an enemy and calls `GoodBee_AttackOtherSprite`.

The first three codes make the game call it every frame. `Sprite_Main`
(`Bank06.asm`) runs once each frame of play, before the sprites:

```
    JSR.w Oam_ResetRegionBases
    JSL.l Garnish_ExecuteUpperSlotsLong   <- $09:B06E patched to $1E:DCCF
    JSL.l Tagalong_MainLong
```

The codes change this `JSL` to `JSL PlayerItem_ReleaseBee`
(`$1E:DCCF`). That routine sets its own data bank and keeps X, and
`Sprite_Main` does not use what it returns. So one good bee comes out
next to Link each frame, until every free sprite slot holds one. A new
bee comes out each time a slot is free again (for example, when an
enemy is killed and its prize is picked up).

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `20 D3 83 22 6E B0 09 22 91 9F 09` at
`$06:835E` (PC `$03035E`): `JSR Oam_ResetRegionBases : JSL $09B06E :
JSL $099F91`. `$09:B06E` (PC `$04B06E`) starts with `22 B6 E9 1D`,
the `JSL Filter_MajorWhitenMain` of `Garnish_ExecuteUpperSlotsLong`.
`$1E:DCCF` (PC `$0F5CCF`) starts with `8B 4B AB A9 B2 22 5D F6 1D`
(`PHB : PHK : PLB : LDA.b #$B2 : JSL Sprite_SpawnDynamically`), the
start of `PlayerItem_ReleaseBee`. At `$1E:DD0A` it has `C9 08 D0 05 A9
01 99 B0 0E`. The byte at each code's address was read back as the orig
byte. The codes decode back to their raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- It is not one bee: the bees fill every free sprite slot. New enemies
  that load as Link walks on the overworld may not find a slot. Pots
  and bushes may not lift when there is no free slot.
- The upper garnish slots no longer run, and `Filter_MajorWhitenMain`
  (the white screen flash) no longer runs from here. Some sparkles,
  splashes and flashes are missing.
- A good bee attacks only enemies marked as bee targets (`$0CD2`
  bit 6) on its own layer.
- Link can catch the bees with the bug net. Whether the sword also
  catches them was not checked.
- A real good bee in a bottle still works; a bottle bee is also a
  good bee now.

## Every enemy moves the sprite in the previous slot sideways instead of itself

**Codes:** `DE37-0F9C` `DEEE-0F21` `DEE4-A1E8` `DE39-0981`
(use all four)
**Raw form:** `06E935:0F` (orig `$10`); `05F9F7:0F` (orig `$10`);
`1EFF2B:0F` (orig `$10`); `1DE956:0F` (orig `$10`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 23:54:05

Tags: `interesting`

**Mechanism:** This is the previous-slot version of "Every enemy moves the
sprite in the next slot sideways instead of itself". Each sprite bank has
its own copy of the move routines: `Sprite_MoveHoriz` (`Bank06.asm`),
`Sprite2_MoveHoriz` (`Bank05.asm`), `Sprite3_MoveHoriz` (`Bank1E.asm`) and
`Sprite4_MoveHoriz` (`Bank1D.asm`). The sprite tables are 16 bytes each, so
the horizontal routine adds `$10` to the slot in X and reuses the vertical
routine on the X tables:

```
Sprite_MoveHoriz:
    ; Do X position adjustment.
    TXA : CLC : ADC.b #$10 : TAX    <- $10 patched to $0F
    JSR.w Sprite_MoveVert
    LDX.w $0FA0
    RTS
```

The codes change the `$10` to `$0F` in all four copies. Now X points one
slot back along the tables, so each sprite's horizontal move is applied to
the **next sprite slot down**: it reads that sprite's X speed and changes
that sprite's X position. A sprite moves sideways only when the sprite in
the slot above it moves. It can still move up and down.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`):

- `$06:E932` (PC `$036932`): `8A 18 69 10 AA 20 3E E9 AE A0 0F 60`
- `$05:F9F4` (PC `$02F9F4`): `8A 18 69 10 AA 20 00 FA AE A0 0F 60`
- `$1E:FF28` (PC `$0F7F28`): `8A 18 69 10 AA 20 34 FF AE A0 0F 60`
- `$1D:E952` (PC `$0EE952`): `DA 8A 18 69 10 AA 20 5D E9 FA 60`
  (`PHX` first, so the operand is one byte later)

The byte at each code's address was read back as `$10`. The codes decode
back to their raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Slot 0 has no previous slot: X becomes `$0F`, so its horizontal move uses
  slot `0x0F`'s Y speed, Y fraction and Y position. Slot `0x0F` then moves
  up and down twice as fast. Nothing outside the sprite tables is changed.
- A sprite moving an empty slot changes that slot's leftover position. This
  should be harmless, since a new sprite sets its own position.
- Sprite code that sets X positions directly, without the move routines, is
  not changed.
- Do not use with "Every enemy moves the sprite in the next slot sideways
  instead of itself": they change the same bytes.

## Every enemy dies like a boss

**Codes:** `6DE7-DD26`
**Raw form:** `06F033:80` (orig `$D0`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-26 00:02:05

Tags: `demo-ready` `video-ready` `curated` `interesting`

**Mechanism:** When a sprite's HP runs out, `Sprite_AttemptKillingOfKin`
(`Bank06.asm`) picks how it dies. After the special cases (Helmasaur King,
Trinexx, Agahnim, Bari, Octoballoon and others), it checks the boss death
bit of `$0B6B` (`SprSetting1`, bit 1: "explosive death animation and ...
a different noise while dying"):

```
    LDA.w $0B6B, X : AND.b #$02 : BNE .BRANCH_PHI   <- BNE patched to BRA
        ...
        JMP.w NormalMob_TimeToDie
    ...
    .BRANCH_PHI
    LDA.w $0E20, X : CMP.b #$A2 : BEQ .BRANCH_OMEGA   ; Kholdstare?
        JSL.l Sprite_SchedulePeersForDeath
    .BRANCH_OMEGA
    LDA.b #$04 : STA.w $0DD0, X
    ...
    BRA Sprite_BossScreamAndDisableMenu
```

The code makes the branch always taken, so every enemy takes the boss
path. `Sprite_SchedulePeersForDeath` kills every other sprite on screen
(except Agahnim and sprites with `$0CAA` bit 1). The enemy goes to state
`0x04` (`SpriteExplode_Main`) with a `$FF` timer, and blows up in a
chain of explosions. `Sprite_BossScreamAndDisableMenu` plays the boss
scream (sound `0x22`) and does `INC $0FFC`, which turns off the menu.
When the timer reaches `$20`, `Explode_VerifyPrizing` removes the enemy.
If `Sprite_VerifyAllOnScreenDefeated` then finds nothing left on screen,
it spawns a heart container (`0xEA`), just like after a boss.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `BD 6B 0B 29 02 D0 22` at `$06:F02E` (PC
`$03702E`), inside `Sprite_AttemptKillingOfKin` (`$06:EFE8`-`$06:F086`).
The `D0` at `$06:F033` was read back as the orig byte. The code decodes
back to its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Killing one enemy kills everything else on screen too, including items
  and NPCs that are sprites, unless `$0CAA` bit 1 protects them.
- The menu stays off after a kill. The game only clears `$0FFC` when it
  loads a new room or area (`Sprite_DisableAll`), and the
  `$0D90` = 0 explode path used here never clears it. Leave the room to
  get the menu back.
- The heart container may keep coming back. Indoors, it does not spawn
  if `$0403` bit 6 or 5 is set (picked by `$0D30` bit 0), but taking it
  sets bit 7. Outdoors, it does not spawn if the area's heart piece flag
  (`$7EF280` bit 6) is set, which taking it does not set. So it may give
  a new heart container after every fight.
- Taking a heart container in a normal room sets that room's boss kill
  bit (`$0403` bit 7). What that does in a room with no boss was not
  checked.
- The exploding enemy runs its own AI for part of the explosion.
- The enemy drops no prize. The enemies it kills drop theirs.

## Enemies walk into pits

**Codes:** `DDE3-65E9` + `DD39-07FB` (use both)
**Raw form:** `1DF6EF:00` (orig `$01`), `06E55C:00` (orig `$3D`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-26 00:03:52

Tags: `interesting`

**Mechanism:** Two checks in `Bank06.asm` keep walking enemies out of
pits (tile type `0x20`).

The first is the wall check. `Sprite_CheckTileProperty` looks up each
tile type in `GeneralizedSpriteTileInteraction` (`Bank1D.asm`); a `$00`
entry means the sprite can walk on that tile:

```
    TYX
    LDA.l GeneralizedSpriteTileInteraction, X   <- entry 0x20 patched to $00
    ...
    CMP.b #$00 : BEQ .succeed_and_exit
```

Entry `0x20` is `$01`, so a pit stops the sprite like a wall. (A sprite
with `$0B6B` bit 0 set gets through only while it is knocked back.) The
first code sets the entry to `$00`, so sprites walk onto pits.

The second is the fall check in `Sprite_CheckTileCollisionSingleLayer`.
It reads the tile under a grounded sprite and makes it fall only when
`$0B6B` bit 0 is set:

```
    CMP.b #$20 : BNE .notAPit
        LDA.w $0B6B, X : AND.b #$01 : BEQ .BRANCH_RHO  <- offset to $00
            LDA.b $1B : BNE .notIndoors
                JMP.w Sprite_SetToFalling
            .notIndoors
            LDA.b #$05 : STA.w $0DD0, X
```

The second code sets the branch offset to `$00`, so every sprite on a pit
falls in (state `0x01` outdoors, `0x05` indoors). Both codes together
make enemies walk straight into pits and fall.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`):

- `GeneralizedSpriteTileInteraction` starts at `$1D:F6CF` (PC
  `$0EF6CF`). Entries `0x1C`-`0x23` are `00 01 01 01 01 01 01 00`, so
  entry `0x20` at `$1D:F6EF` is `$01`.
- `$06:E552` (PC `$036552`): `C9 20 D0 2C BD 6B 0B 29 01 F0 3D A5 1B
  D0` (`CMP #$20 : BNE : LDA $0B6B,X : AND #$01 : BEQ : LDA $1B :
  BNE`). The `BEQ` offset is at `$06:E55C`.

The byte at each code's address was read back as the orig byte. The codes
decode back to their raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- The first code alone lets sprites walk over pits. Most of them do not
  fall, since their `$0B6B` bit 0 is clear; they float over the hole.
- Flying sprites and sprites in the air (`$0F70` not 0) skip the fall
  check. Sprites that use `Sprite_SimplifiedTileAttr` (`$0CAA` bit 3)
  do not use the first table.
- It changes all sprites, not only enemies: items and NPCs that walk can
  fall too.
- An enemy that falls is gone, and gives no prize.

## Link can always walk out of sand lion pits

**Codes:** `6D39-6F2A`
**Raw form:** `07E257:80` (orig `$F0`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-26 00:04:51

Tags: `interesting`

**Mechanism:** `$0B7B` (`PlayerPreventMovement`) stops Link from moving
while it is not 0. Only the Debirando (sand lion) pit sets it, when it
catches Link and the R button was not just pressed
(`Sprites/sprite_debirando_pit.asm`). `Link_HandleVelocity`
(`Bank07.asm`) checks it each frame:

```
    LDA.w $0B7B : BEQ .playerCanMove   <- BEQ patched to BRA
        .preventPlayerMovement
        ; Otherwise, Link can't move and has to stay in place.
```

The code changes the `BEQ` to `BRA`, so Link always takes the
`.playerCanMove` path, whatever `$0B7B` holds. In a sand lion pit he can
walk away against the pull, the same way tapping R lets him escape in the
normal game.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `8B 4B AB A5 11 C9 02 D0 06 A5 10 C9 0E F0 05 AD
7B 0B F0 1B` at `$07:E245` (PC `$03E245`), the start of
`Link_HandleVelocity`. The `F0` at `$07:E257` was read back as the orig
byte. The code decodes back to its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- The sand lion still pulls Link toward its center
  (`Sprite_ApplySpeedTowardsPlayerLong`) and can still hurt him.
- The check just before it, which stops Link during the text box (`$10`
  = `0x0E`, `$11` = `0x02`), jumps straight to `.preventPlayerMovement`
  and is not changed.

## Tutorial soldiers go through the first 256 messages

**Codes:** `DD2C-0499`
**Raw form:** `05D5A9:00` (orig `$07`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-26 00:05:51

Tags: `demo-ready` `interesting` `play-start`

**Mechanism:** The tutorial soldiers in Hyrule Castle
(`Sprite_TutorialEntities`, `Sprites/sprite_tutorial_entities.asm`) show
message `$0F + $0B69` when Link touches them, then move `$0B69`
(`TutorialGuardMessage`) to the next one. It goes from 0 to 6 and then
back to 0, so they cycle through messages `0x0F`-`0x15`:

```
        LDA.w $0B69 : PHA
        CLC : ADC.b #$0F
        LDY.b #$00
        JSL.l Sprite_ShowMessageIfPlayerTouching
        
        PLA : BCC .message_not_shown
            INC : CMP.b #$07 : BNE .no_reset   <- $07 patched to $00
                LDA.b #$00
```

With `CMP.b #$00`, the reset only happens when the `INC` wraps from
`$FF` to `$00`, which is 0 anyway. So `$0B69` counts through all 256
values, and each talk shows the next message ID from `0x0F` up to
`0xFF` and then `0x00`-`0x0E`. Y (the high byte of the message ID) stays
`$00`, so only the first 256 messages are reached.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `18 69 0F A0 00 22 59 FA 05 68 90 07 1A C9 07 D0 02
A9 00 8D 69 0B` at `$05:D59B` (PC `$02D59B`) in
`Sprite_TutorialEntities`. The `07` at `$05:D5A9` was read back as the
orig byte. The code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Some messages are made for special scenes, such as yes/no choices or
  the ones that run with a cutscene. What each one does when a soldier
  shows it was not checked. Some may act oddly.
- Blind also uses `$0B69` (`BlindHeadDir`), so the next message may
  change after the Blind fight. This was also the case without the code.
- In area `0x1B` (`$040A`), the soldiers at Y `$50` and `$90` show fixed
  messages `0xB2` and `0xB3` and are not changed.

## Arrows are never deflected

**Codes:** `6D6A-0447`
**Optional, shields don't block arrows or other items either:** `6D6E-0D37`
**Raw form:** `088DC8:80` (orig `$F0`); optional `088DF3:80` (orig `$F0`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-26 00:06:48

Tags: `interesting`

**Mechanism:** `Ancilla_CheckIndividualSpriteCollision` (`Bank08.asm`)
runs when an ancilla (arrow, boomerang, hookshot and so on) touches a
sprite. First it checks the sprite's "deflect arrows" bit (`$0B6B` bit
3). If it is set and the ancilla is an arrow (`0x09`), the arrow bounces
off with `Sprite_CreateDeflectedArrow` (or, with the Silver Arrows, only
against sprite `0x1B`, the enemy arrow). Next it checks `$0CAA` bit 4,
which makes the sprite immune to ancillas that hit it from the front
(a shield):

```
    LDA.w $0B6B, Y : AND.b #$08 : BEQ .doesnt_deflect_arrows   <- BEQ to BRA
        ...
    .undeflected_silver_arrow
    LDA.w $0CAA, Y : AND.b #$10 : BEQ .doesnt_absorb_ancilla  <- optional
        ...
        CMP.w $0C72, X : BEQ .collision_immunity
    .doesnt_absorb_ancilla
```

The main code changes the first `BEQ` to `BRA`, so no sprite deflects
arrows, and the arrow goes on to the normal hit code. The optional code
does the same for the shield check, so ancillas hit shielded enemies from
the front too. That changes every ancilla that uses this routine, not
only arrows.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `B9 6B 0B 29 08 F0 24` at `$08:8DC3` (PC
`$040DC3`) and `B9 AA 0C 29 10 F0 16` at `$08:8DEE` (PC `$040DEE`, the
`.undeflected_silver_arrow` entry), both inside
`Ancilla_CheckIndividualSpriteCollision` (`$08:8DAE`-`$08:8E7C`). The `F0`
at each code's address was read back as the orig byte. The codes decode
back to their raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- An arrow that hits still does that sprite's arrow damage. Sprites
  whose damage for arrows is 0 are hit but not hurt.
- Sprites with `$0BA0` set still can't be hit.

## The sword beam is an ice rod shot

**Codes:** `4DBB-D7FE` `11BB-D79E` `BEBB-D7BE` `3CBB-D72E` (use all four)
**Raw form:** `079C9C:20` (orig `$22`); `079C9D:66` (orig `$7B`);
`079C9E:9F` (orig `$F6`); `079C9F:EA` (orig `$0F`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-26 00:13:49

Tags: `demo-ready` `interesting`

**Mechanism:** When Link swings a Master Sword or better at full health,
`HandleSwordSFXAndBeam` (`Bank07.asm`) adds the spinning sword beam:

```
    LDY.b #$00
    JSL.l AddSwordBeam                 ; 22 7B F6 0F  <- 20 66 9F EA
```

The codes change the `JSL` to `JSR.w LinkItem_IceRod : NOP`. `LinkItem_IceRod`
is in the same bank, and is what the rod item code calls:

```
    LDA.b #$0B
    LDY.b #$01
    JSL.l AddIceRodShot
    RTS
```

So the beam swing adds a rod shot in Link's facing direction instead.
`LDY.b #$00` before it does nothing now. No magic is used.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC offset +
`$200`) has `A0 00 22 7B F6 0F` at `$07:9C9A` (PC `$039C9A`), inside
`HandleSwordSFXAndBeam` (`$039C66`-`$039CBE`). `AddSwordBeam` is PC `$07F67B`
(`$0F:F67B`). `LinkItem_IceRod` has `A9 0B A0 01 22 63 98 09 60` at `$07:9F66`
(PC `$039F66`), with `AddIceRodShot` at `$09:9863`. The byte at each code's
address was read back as the orig byte. The codes decode back to their raw form
with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- The beam still needs full health and a Master Sword or better, and no Cane of
  Byrna in use.
- When the shot can't be added (2 of that shot already out, or no free slot),
  `AddIceRodShot` gives back a rod's magic cost (`$10`, `$08` or `$04`, by the
  magic upgrade). No magic was taken, so swinging then raises magic.
- The beam's own code is not used, so the spinning beam, its sound and its hit
  effects are gone. The rod shot sound plays instead.

## The sword beam is a fire rod shot

**Codes:** `4DBB-D7FE` `1EBB-D79E` `BEBB-D7BE` `3CBB-D72E` (use all four)
**Raw form:** `079C9C:20` (orig `$22`); `079C9D:6F` (orig `$7B`);
`079C9E:9F` (orig `$F6`); `079C9F:EA` (orig `$0F`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-26 00:13:51

Tags: `demo-ready` `interesting`

**Mechanism:** When Link swings a Master Sword or better at full health,
`HandleSwordSFXAndBeam` (`Bank07.asm`) adds the spinning sword beam:

```
    LDY.b #$00
    JSL.l AddSwordBeam                 ; 22 7B F6 0F  <- 20 6F 9F EA
```

The codes change the `JSL` to `JSR.w LinkItem_FireRod : NOP`. `LinkItem_FireRod`
is in the same bank, and is what the rod item code calls:

```
    LDA.b #$02
    LDY.b #$01
    JSL.l AddFireRodShot
    RTS
```

So the beam swing adds a rod shot in Link's facing direction instead.
`LDY.b #$00` before it does nothing now. No magic is used.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC offset +
`$200`) has `A0 00 22 7B F6 0F` at `$07:9C9A` (PC `$039C9A`), inside
`HandleSwordSFXAndBeam` (`$039C66`-`$039CBE`). `AddSwordBeam` is PC `$07F67B`
(`$0F:F67B`). `LinkItem_FireRod` has `A9 02 A0 01 22 B3 80 08 60` at `$07:9F6F`
(PC `$039F6F`), with `AddFireRodShot` at `$08:80B3`. The byte at each code's
address was read back as the orig byte. The codes decode back to their raw form
with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- The beam still needs full health and a Master Sword or better, and no Cane of
  Byrna in use.
- When the shot can't be added (2 of that shot already out, or no free slot),
  `AddFireRodShot` gives back a rod's magic cost (`$10`, `$08` or `$04`, by the
  magic upgrade). No magic was taken, so swinging then raises magic.
- The beam's own code is not used, so the spinning beam, its sound and its hit
  effects are gone. The rod shot sound plays instead.
## Every sound effect comes from the wrong sound list

**Codes:** `0764-0D6D` `0464-04DD` (use both)
**Raw form:** `008122:43` (orig `$42`); `008128:42` (orig `$43`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-26 12:04:53

Tags: `demo-ready` `video-ready` `curated` `interesting` `play-start`

**Mechanism:** This is the broader form of "Link's sounds come from the
wrong sound list". Every part of the game (Link, sprites, ancillae,
menus) plays a sound by writing its ID to `$012E` (SFX1) or `$012F`
(SFX2). Once per frame, the NMI handler in `Bank00.asm` sends both to the
sound chip, each on its own port:

```
.writeSFX

; Addresses will hold SPC memory locations.
LDA.w $012E : STA.w SNES.APUIOPort2
LDA.w $012F : STA.w SNES.APUIOPort3
```

The codes swap the two ports (`$2142` and `$2143`). Now every SFX1 sound
plays the SFX2 sound with the same ID, and every SFX2 sound plays the SFX1
sound with that ID. Some of them (IDs from the lists in `WRAM.asm`):

| Sound | List | ID | Now plays |
|---|---|---|---|
| Fighter Sword swing | SFX1 | `0x01` | sword beam |
| Master Sword swing | SFX1 | `0x03` | loud thud |
| Enemy taking a big hit | SFX2 | `0x08` | arrow hitting a wall |
| Enemy dying | SFX2 | `0x09` | boomerang flying |
| Minor damage to an enemy | SFX2 | `0x1C` | walking in shallow water |
| Collecting a rupee | SFX2 | `0x0A` | hookshot cranking |
| Collecting a heart | SFX2 | `0x0B` | placing a bomb |
| Bomb exploding | SFX1 | `0x0C` | text scrolling flute noise |
| Link taking damage | SFX1 | `0x26` | Agahnim's lightning |
| Low life beep | SFX1 | `0x2B` | Link getting electrocuted |
| Text box blip | SFX2 | `0x24` | deep water splash |

Music (`$012C`) and ambient sounds (`$012D`, such as rain) use other ports
and are not changed.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `AD 2E 01 8D 42 21 AD 2F 01 8D 43 21` once in the
ROM, at `$00:811E` (PC `$00011E`). A script read back `$42` at `$00:8122`
and `$43` at `$00:8128`. No other code in the source writes these two
ports outside the sound program upload. Both codes decode back to their
raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Tested by the user.
- Use both codes. With only one, both stores go to the same port and the
  second one (`$2143`, or `$2142` for the second code alone) always wins,
  even when it writes `$00`. Half the sounds would never play.
- Don't use with "Link's sounds come from the wrong sound list". The two
  swaps cancel out, so Link's own sounds play as normal.
- A sound in the wrong list can use another channel and cut off other
  sounds.

## Link's house door leads to the Chris Houlihan room

**Code:** `6485-A033`
**Raw form:** `1BBB73:82` (orig `$01`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-26 12:15:52

Tags: `demo-ready` `interesting` `play-start`

**Mechanism:** When Link walks into an overworld door, `Overworld_Entrance`
(`Bank1B.asm`) finds the door in its tables and stores that door's entrance
ID:

```
LDA.l Pool_Overworld_Entrance_ID, X : STA.w $010E
```

The first row of `Pool_Overworld_Entrance_ID` is Link's house:

```
db $01 ; 0x00 - ENTRANCE 01 - Link's house
```

Entrance `0x82` is the Chris Houlihan room. The game only uses it when
Link falls into a hole that `Overworld_Hole` can't find in its list. The
code changes the `$01` to `$82`, so the front door of Link's house takes
him to the Houlihan room.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `01 02 03 04 05 06` at `$1B:BB73` (PC `$0DBB73`), the
start of `Pool_Overworld_Entrance_ID`. A script read back `$01` at
`$1B:BB73`. `Pool_Overworld_Hole_entrance` ends with `$82` (its Houlihan
fallback, index `0x13`), and `EntranceData_rooms` gives room `$0003` for
entrance `0x82` and `$0104` (Link's house) for entrance `0x01`. The code
decodes back to its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Only the door changes. The game still starts in Link's bed, and save and
  continue still puts Link in his house; those use starting points, not
  entrance `0x01`.
- Link enters with the Houlihan entrance's position and camera, which were
  made for falling in from a hole. Where he stands when he arrives was not
  checked.
- Room `$0003` is not in `UnderworldExitData_room_id`. If the room has a
  door out to the overworld, the exit search loop in `Bank02.asm` has no
  end check (the source notes this as a `BUG:`), so walking out may hang
  the game. Use save and quit, or the Magic Mirror, to leave. Neither was
  checked.
- Link's house can't be entered through its door. Link is only inside it
  at the start of the game and after save and continue there.

## No delay for opening treasure chests

**Codes:** `D46D-DF31` `DFA4-0F1F` (use both)
**Raw form:** `098807:02` (orig `$38`); `08C525:01` (orig `$28`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-26 12:28:15

Tags: `demo-ready` `interesting` `play-start`

**Mechanism:** When Link opens a chest, `AddReceivedItem`
(`Ancilla/ancilla_init.asm`) adds the item object and sets its timer
`$03B1`. Items from text start at `$60`; items from chests, sprites and
boss prizes start at `$38`. Heart pieces start at `$02`:

```
LDY.b #$60

LDA.w $02E9 : STA.w $0C54, X
BEQ .fromText
    LDY.b #$38
```

For chests, `Ancilla_ReceiveItem` (`Ancilla/ancilla_receive_item.asm`)
lowers the timer by one each frame. The item rises while the timer is
`$18` or more, rupees are given at `$28`, the item's message shows at
`$00`, and the object ends after that:

```
CMP.b #$00 : BEQ .timesUp
    ; Give a rupee gift when the timer reaches value 0x28.
    CMP.b #$28 : BNE .dontGiveRupees
```

The first code changes the `$38` to `$02`, the same start that heart
pieces use, so the message shows 2 frames after the chest opens instead
of 56 frames (almost 1 second). The second code moves the rupee mark from
`$28` to `$01`, so rupees in chests are still given. Without it, the timer
never reaches `$28` and the rupees are lost.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `AD E9 02 9D 54 0C F0 02 A0 38` once in the ROM; a
script read back `$38` at `$09:8807` (PC `$048807`). It has `C9 00 F0 22
C9 28 D0 14 BD 54 0C C9 02` once; a script read back `$28` at `$08:C525`
(PC `$044525`). Both codes decode back to their raw forms with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- The item no longer rises out of the chest. It stays where it first
  appears, just above the chest.
- Link still holds the item up while its message is on screen. Items
  with no message end right away.
- Items from sprites and boss prizes (other than pendants and crystals,
  which use `$68`) also lose the delay.
- The "you got an item" sound that chest items play at the rupee mark now
  plays at the `$01` mark, right before the message.

## Link lifts the sprite in the slot after the one he touches

**Code:** `3CE9-6726`
**Raw form:** `06F25F:EA` (orig `$3A`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-26 12:28:58

Tags: `interesting` `play-start`

**Mechanism:** This is the other direction of "Link lifts the sprite in
the slot before the one he touches". `$0314` holds the slot index + 1 of
the liftable sprite Link is touching (0 if none). Each frame, a liftable
sprite (chicken, fish, bush crab, a frozen enemy, an enemy bomb, a thrown
object after it lands) runs `Sprite_CheckIfLiftedPermissive`
(`Bank06.asm`):

```
; Check if the current sprite is the same one Link is touching.
LDA.w $02F4 : DEC : CMP.w $0FA0 : BEQ .player_picks_up_sprite
    ...
    JSR.w Utility_CheckIfHitBoxesOverlap : BCC .return
    TXA : INC : STA.w $0314
                STA.w $0FB2
```

When Link presses A, `Link_HandleAPress` (`Bank07.asm`) copies `$0314`
to `$02F4`, and `Link_Lift` starts the lift pose. The sprite whose slot
(`$0FA0`) equals `$02F4` - 1 then puts itself in Link's hands.

The code changes the `DEC` to `NOP`, so each sprite compares its slot
with `$02F4` itself, which is the touched slot + 1. The sprite one slot
higher than the one Link touches flies into his hands from wherever it
is.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `AD 72 03 D0 4D AD F4 02 3A CD A0 0F F0 14` once in
the ROM, at `$06:F257` (PC `$037257`), which is `LDA.w $0372 : BNE :
LDA.w $02F4 : DEC : CMP.w $0FA0 : BEQ`. A script read back the `DEC`
(`$3A`) at `$06:F25F`. The code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Only sprites that call this routine can be lifted. If the slot above
  holds something else, or is empty, Link does the lift pose and holds
  nothing. A sprite in the last slot has no slot above it.
- `$02F4` is 0 whenever no lift is in progress, so a liftable sprite in
  slot 0 jumps into Link's hands as soon as it can.
- `$02F4` stays set until the next A press. A liftable sprite that later
  appears in that slot may jump into Link's hands without him touching
  it.
- Pots, bushes and rocks on the ground are tiles, not sprites, and are
  not changed.
- Don't use with "Link lifts the sprite in the slot before the one he
  touches". Together they cancel out.

## Pots stay in place after Link lifts them

**Code:** `4A20-6761`
**Raw form:** `01DA4E:2C` (orig `$20`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-26 12:31:00

Tags: `demo-ready` `interesting` `play-start`

**Mechanism:** This is the indoor form of "Bushes, small rocks and signs
stay in place after Link lifts them (overworld)". When Link lifts
something indoors, `Link_Lift` (`Bank07.asm`) calls
`Dungeon_RevealCoveredTiles` (`Bank01.asm`). For a pot (a tile with
replaceable attributes `$10`-`$1F`), it loads any secret under the pot,
erases the pot from the room, and returns the pot's tile attribute so
`Link_Lift` can spawn the pot sprite in Link's hands:

```
LDA.w $0540, Y
JSR.w Dungeon_LoadSecret

LDY.w $042C
JSR.w Dungeon_EraseInteractive2x2

PLA : AND.w #$000F : ASL : TAX
LDA.l Pool_Dungeon_RevealCoveredTiles_tile_01d9e0, X : STA.b $06
```

`Dungeon_EraseInteractive2x2` writes the floor tiles to the tilemap
(`$7E2000`) and the collision map (`$7F2000`), and queues the VRAM update.
The code changes the `JSR` (`$20`) to `BIT` absolute (`$2C`), with the
same operand, so the call is skipped. The `BIT` only reads a ROM byte and
sets flags, and the `PLA` right after sets them again. The pot stays in
the room, but Link still gets a pot sprite in his hands, so he can lift
the same pot again and again.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `AC 2C 04 20 28 D8 68 29 0F 00` once in the ROM, at
`$01:DA4B` (PC `$00DA4B`): `LDY.w $042C : JSR.w Dungeon_EraseInteractive2x2
: PLA : AND.w #$000F`. `Dungeon_EraseInteractive2x2` is at `$01:D828`
(PC `$00D828`). A script read back the `JSR` opcode `$20` at `$01:DA4E`.
The code decodes back to its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Indoors only. Use "Bushes, small rocks and signs stay in place after
  Link lifts them (overworld)" for the overworld.
- Any other indoor object that uses this path (attributes `$10`-`$1F`)
  also stays. Which objects those are was not checked.
- `Dungeon_LoadSecret` still runs each time, so an item or switch hidden
  under a pot may come out every time Link lifts it. Not checked.
- Smashing a pot with the hammer uses `Dungeon_ToolAndTileInteraction`
  and still removes it.

## The Lamp lets out a good bee

**Codes:** `AEC5-6FFA` `2AC5-6F9A` `F3C5-6FBA` `DD2D-0558` (use all four)
**Raw form:** `07A274:CF` (orig `$1C`); `07A275:DC` (orig `$8F`);
`07A276:1E` (orig `$09`); `1EDD0D:00` (orig `$05`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-26 13:04:38

Tags: `demo-ready` `video-ready` `curated` `interesting`

**Mechanism:** This is a one-bee form of "Good bees keep coming out and
attack enemies", which fills every sprite slot. `LinkItem_Lamp`
(`Bank07.asm`) makes the flame last:

```
LDY.b #$02
LDA.b #$2F
JSL.l AddLampFlame
```

The first three codes change the `JSL` to `JSL PlayerItem_ReleaseBee`
(`$1E:DCCF`), what the bottle calls to let a bee out. It spawns sprite
`0xB2` next to Link in a free slot, sets its own data bank and keeps X.
The code after the call (`.cant_use_lamp`) does not use what it returns.
The fourth code is the one from "Good bees keep coming out and attack
enemies": it sets the `BNE .not_good_bee` offset in
`PlayerItem_ReleaseBee` to `$00`, so the bee is always a good bee
(`$0EB0` = 1), which attacks enemies. So each use of the Lamp lets out
one good bee instead of the flame.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A0 02 A9 2F 22 1C 8F 09` once in the ROM, at
`$07:A26F` (PC `$03A26F`): `LDY.b #$02 : LDA.b #$2F : JSL $098F1C`
(`AddLampFlame`, PC `$048F1C`). `$1E:DCCF` (PC `$0F5CCF`) starts with
`8B 4B AB A9 B2 22 5D F6 1D` (`PHB : PHK : PLB : LDA.b #$B2 : JSL
Sprite_SpawnDynamically`). At `$1E:DD0A` it has `C9 08 D0 05 A9 01`. A
script read back the orig byte at each code's address. The codes decode
back to their raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- The Lamp still costs magic, and still lights torches
  (`Dungeon_LightTorch`) and makes its `AddMagicPowder` effect. Only the
  flame is replaced, so the Lamp no longer burns bushes or enemies.
- With no free sprite slot, no bee comes out.
- Each use adds one more bee. How long a released bee stays was not
  checked.
- The fourth code also makes every bottle bee a good bee. Leave it off
  to get a normal bee, which attacks nothing.
- Uses the same bytes as "The Lamp casts Bombos" and "The Lamp throws
  the boomerang, shoots an arrow, pounds the hammer and sprinkles
  powder". Use only one of them.

## The sword beam is a good bee

**Codes:** `AEBB-D79E` `2ABB-D7BE` `F3BB-D72E` `DD2D-0558` (use all four)
**Raw form:** `079C9D:CF` (orig `$7B`); `079C9E:DC` (orig `$F6`);
`079C9F:1E` (orig `$0F`); `1EDD0D:00` (orig `$05`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-26 13:04:38

Tags: `interesting`

**Mechanism:** When Link swings a Master Sword or better at full health,
`HandleSwordSFXAndBeam` (`Bank07.asm`) adds the spinning sword beam:

```
    LDY.b #$00
    JSL.l AddSwordBeam
```

The first three codes change the `JSL` to `JSL PlayerItem_ReleaseBee`
(`$1E:DCCF`), as in "The Lamp lets out a good bee". The code after the
call reloads A and X, so it does not use what the bee routine returns.
The fourth code makes the bee a good bee, which attacks enemies. So each
beam swing lets out one good bee instead of a beam.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A0 00 22 7B F6 0F` once in the ROM, at `$07:9C9A`
(PC `$039C9A`), inside `HandleSwordSFXAndBeam`. `AddSwordBeam` is at
`$0F:F67B`. The bee routine and the good-bee branch are checked as in
"The Lamp lets out a good bee". A script read back the orig byte at each
code's address. The codes decode back to their raw forms with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- The bee still needs full health and a Master Sword or better, and no
  Cane of Byrna in use. Add "Sword beams fire at any health" and "The
  Fighter's Sword shoots beams" to get a bee on every swing.
- With no free sprite slot, no bee comes out. Many swings can fill the
  free slots, as in "Good bees keep coming out and attack enemies".
- The fourth code also makes every bottle bee a good bee.
- Uses the same bytes as "The sword beam is a fire rod shot" and "The
  sword beam is an ice rod shot". Use only one of them. The other sword
  beam cheats do nothing with this one.

## Enemies drop cuccos instead of prizes

**Codes:** `CBEA-07BC` `D8EA-072C` `3CE2-0DFC` (use all three)
**Raw form:** `06F9CE:A9` (orig `$B9`); `06F9CF:0B` (orig `$72`);
`06F9D0:EA` (orig `$FA`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-26 15:55:44

Tags: `demo-ready` `video-ready` `interesting`

**Mechanism:** When an enemy dies, `Sprite_DoTheDeath` (`Bank06.asm`)
picks a prize from its prize pack and turns the dying sprite into that
prize:

```
PLY
LDA.w PrizePack_Prizes_pack_1, Y

.PrepareEnemyDrop

STA.w $0E20, X : CMP.b #$E5 : BNE .BRANCH_NU
```

The codes change `LDA.w PrizePack_Prizes_pack_1, Y` (`B9 72 FA`) to
`LDA.b #$0B : NOP` (`A9 0B EA`). Every prize is now sprite `0x0B`, the
cucco (`Sprite_Chicken`). `.PrepareEnemyDrop` sets the sprite to the
active state (`$0DD0` = `$09`) and calls `Sprite_LoadProperties`, which
calls `Sprite_ResetProperties` first. That clears `$0DB0` (attack cucco
flag) and `$0EB0` (cucco-to-lady flag), so the drop is a plain cucco.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `B9 72 FA 9D 20 0E C9 E5` once in the ROM, at
`$06:F9CE` (PC `$0379CE`). The routine runs with 8-bit A (`LDA.b
#$09 : STA.w $0DD0, X` follows), so `LDA.b #$0B` is 2 bytes and the
`NOP` fills the third. `Bank06.asm` lists `Sprite_Chicken` as sprite
`0x0B`. A script read back the orig byte at each code's address. The
codes decode back to their raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Enemies still drop something only as often as before (often 50%). Add
  "Enemies always drop a prize" (`06F9BB`, different bytes) to get a
  cucco from every enemy that has a prize pack.
- Keys and other drops that the room data gives an enemy (`$0CBA`) skip
  this load, so they still drop as normal.
- Other code that calls `ForcePrizeDrop_long` (`Bank1D.asm`) also gets a
  cucco.
- The drop code reads `ItemDropBounceProps-$D9, Y` for a small hop. That
  table only covers `$D9`-`$E6`, so for the cucco it reads a code byte
  (`$19` at PC `$037996`): the cucco starts with a small upward speed and
  is moved 9 pixels right.
- The drop code also sets `$0BA0` (ignore projectiles), so arrows and the
  boomerang may pass through the dropped cucco. The sword still hits it.
- The cucco's graphics may not be loaded in every area. There it may
  look wrong.

## Every sound effect is the one after it in its sound list

**Codes:** `4F6A-D93C` `4D6A-D53C` `F264-091C` `FA64-051C` (use all four)
**Raw form:** `1A88C7:21` (orig `$1F`); `1A88CF:20` (orig `$1E`);
`1A8925:1D` (orig `$1B`); `1A892D:1C` (orig `$1A`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Optional, ambient sounds too:** `AF61-D11C` `AD65-D01C` (`1A8869:C1`,
orig `$BF`; `1A8871:C0`, orig `$BE`)
**Recorded:** 2026-10-29 00:52:34

**Mechanism:** This is the ALTTP version of the FF2 and SMW codes with
the same name. It is not the same as "Every sound effect comes from the
wrong sound list", which swaps the two lists. When an SFX1 (`$012E`) or
SFX2 (`$012F`) sound starts, the sound program (`Bank19.asm`) reads its
pointer from the list for that port, 2 bytes per ID. For SFX1:

```
; Get the SFX2 ID and then get the data pointer for that sound.
mov.w A, $03A0+X : asl A : mov Y, A
mov.w A, SFX2_Pointers-1+Y : mov.w $0391+X, A
                             mov.b $2D, A
mov.w A, SFX2_Pointers-2+Y : mov.w $0390+X, A
                             mov.b $2C, A
```

The sound program calls the SFX1 list `SFX2_Pointers` and the SFX2 list
`SFX3_Pointers`. The codes add 2 to the low byte of all four addresses,
so ID `n` plays the sound of ID `n+1` from the same list. The first two
codes are SFX1 and the last two are SFX2. The optional codes do the same
for the ambient list (`$012D`, `Ambient_Pointers`). Some of them (IDs
from the lists in `General/WRAM.asm`):

| Sound | List | ID | Now plays |
|---|---|---|---|
| Fighter Sword swing | SFX1 | `0x01` | the second Fighter Sword swing |
| Placing a bomb | SFX1 | `0x0B` | bomb exploding |
| Bomb exploding | SFX1 | `0x0C` | magic powder |
| Link taking damage | SFX1 | `0x26` | Link passing out |
| Low life beep | SFX1 | `0x2B` | pulling the Master Sword out |
| Enemy dying | SFX2 | `0x09` | collecting a rupee |
| Collecting a rupee | SFX2 | `0x0A` | collecting a heart |
| Opening a chest | SFX2 | `0x0E` | getting an item |
| Door opening | SFX2 | `0x15` | door slamming shut |
| Text box blip | SFX2 | `0x24` | triggering a switch |

Music doesn't change.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `F5 A0 03 1C FD F6 lo hi D5 91 03 C4 2D F6 lo hi
D5 90 03 C4 2C` three times, at `$1A:8863` (ambient, `$17BF`/`$17BE`),
`$1A:88C1` (SFX1, `$181F`/`$181E`) and `$1A:891F` (SFX2,
`$191B`/`$191A`). Adding 2 to these low bytes doesn't carry. The sound
program is in the ROM and is uploaded at power-on. A script read back the
orig byte at each code's address. The codes decode back to their raw
forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- The last ID of each list (`0x3F`) reads past the end of the list and
  may play noise or stop the sound.
- Some sounds have a part 2 on another channel. The part 2 may now
  belong to a different sound, or not play.
- The first two codes alone change only SFX1, and the second two alone
  change only SFX2.

## Every song is the one after it in its song bank

**Code:** `8AE1-6135`
**Raw form:** `19FE6B:BC` (orig `$60`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-29 00:52:34

**Mechanism:** This is the ALTTP version of the FF2 and SMW codes with
the same name. `MusicChangeSong` (`Bank19.asm`) starts a song. It reads
the song's pointer from the list at ARAM `$D000` (`SONG_POINTERS`), 2
bytes per song:

```
MusicChangeSong:
{
    clrc

    ; Set the music fade out timer and volume cache to 0.
    mov.b X, #$00 : mov.w $03CA, X
                    mov.w $03E1, X

    ; Set the current song.
    mov.b $04, A

    ; Get the song pointer and set it to the current song segment.
    asl A : mov X, A
    mov.w A, SONG_POINTERS-1+X : mov Y, A
    mov.w A, SONG_POINTERS-2+X : movw.b $40, YA
```

The code changes the `clrc` to `inc A`, so the song ID is 1 higher
before the list is read. The `asl A` sets the carry again, so the
`clrc` isn't needed. The song commands (`0xF0`-`0xF3`, `0xFF`) are
checked before this, so they still work. Each song bank (overworld,
underworld, credits) uploads its own list to `$D000`, with `$0000` for
the songs that are not in that bank. Some of them (names from
`Bank1A.asm` and `Bank1B.asm`):

| Song | Bank | ID | Now plays |
|---|---|---|---|
| Triforce intro | overworld | `0x01` | Light World |
| Light World | overworld | `0x02` | rain |
| Kakariko Village | overworld | `0x07` | mirror warp |
| Dark World | overworld | `0x09` | pulling the Master Sword |
| Hyrule Castle | underworld | `0x10` | pendant dungeon |
| Pendant dungeon | underworld | `0x11` | cave |
| Fanfare | underworld | `0x13` | sanctuary |
| Boss | underworld | `0x15` | crystal dungeon |
| Shop | underworld | `0x17` | cave |
| Triforce room | credits | `0x20` | ending theme |

**Evidence:** `alttp_original.smc` has `60 CD 00 C9 CA 03 C9 E1 03 C4
04 1C 5D F5 FF CF` once in the ROM, at `$19:FE6B`. `$BC` is `inc A` in
SPC700 code. `$04` (the current song) now holds the ID plus 1, but the
sound program only checks it for 0, and it can't be 0 here. The code
decodes back to its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- When the next song is not in the bank that is loaded, the list holds
  `$0000`. Then the song may play noise, stop the sound or hang the
  music. This happens for the intro fanfare (`0x0F`), the fairy theme
  (`0x0B`), the King of Thieves (`0x1F`) and the credits (`0x22`).
  `0x23` in the underworld bank reads past the end of its list.
- The intro bank's songs shift too.

## Every song plays with the next instrument

**Code:** `D160-D936`
**Raw form:** `1A8047:06` (orig `$00`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Optional, instead, two instruments later:** `DA60-D936` (`1A8047:0C`)
**Recorded:** 2026-10-29 00:52:34

Tags: `demo-ready` `play-start`

**Mechanism:** This is the ALTTP version of the SMW code with the same
name. Voice command `$E0` (`TrackCommandE0_ChangeInstrument`,
`Bank19.asm`) sets a voice's instrument. It reads the instrument from
the table at ARAM `$3D00` (`INSTRUMENT_DATA`), 6 bytes each (sample,
ADSR 1 and 2, gain, pitch multiplier):

```
; Get the index for the instrument data.
mov.b Y, #$06
mul YA : movw.b $14, YA

; Get the address for the instrument data.
clrc : adc.b $14, #INSTRUMENT_DATA>>0
       adc.b $15, #INSTRUMENT_DATA>>8
```

The code changes the `#$00` to `#$06`, so the table starts 6 bytes
later. Every instrument `n` in a song plays instrument `n+1`, with that
instrument's sample, envelope and pitch multiplier. The optional code
uses `#$0C` (2 instruments later). The carry from the low byte goes into
the high byte, so the address stays right. Some of them (names from the
table in `Bank19.asm`):

| Instrument | ID | Now plays |
|---|---|---|
| Strings A | `0x09` | strings B |
| Trombone | `0x0B` | cymbal |
| Ocarina | `0x0D` | chimes |
| Harp | `0x0F` | splash |
| Trumpet | `0x11` | horn |
| Flute | `0x16` | "oof" |
| Oof | `0x17` | piano |

**Evidence:** `alttp_original.smc` has `98 00 14 98 3D 15` once in the
ROM, at `$1A:8046`. The table has 25 instruments (`0x00`-`0x18`), from
`$3D00` up to `NoteStaccato` at `$3D96`. Sound effects read their
instruments from `INSTRUMENT_DATA_SFX` (`$3E00`) in other code. The code
decodes back to its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Each instrument has its own pitch multiplier, so the new instrument
  can play in a different key or octave from the old one.
- The last instrument (`0x18`, piano) reads the `NoteStaccato` and
  `NoteAttack` tables. It may play a wrong sample or no sound.
- Percussion notes use the same table, so they change too.
- Sound effects don't change.

## Music and sound effects play an octave higher

**Code:** `D9E6-0945`
**Raw form:** `19FD84:05` (orig `$06`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Optional, instead, an octave lower:** `D5E6-0945` (`19FD84:07`)
**Optional, instead, two octaves lower:** `D6E6-0945` (`19FD84:08`)
**Recorded:** 2026-10-29 00:52:34

**Mechanism:** This is the ALTTP version of the SMW code with the same
name. `WritePitch` (`Bank19.asm`) turns a note number into a pitch. It
splits the note into an octave (`X`) and one of 12 values from the pitch
table, then halves the value once for each octave below 6:

```
    bra .startOctaveLoop
    ; Divide the calculated pitch by 2 for every octave we need to go down
    ; starting at octave 6.
    .octaveLoop
        lsr.b $15
        ror A
        inc X

        .startOctaveLoop
    cmp.b X, #$06 : bne .octaveLoop
```

The code changes the `#$06` to `#$05`, so every note is halved one time
less and plays an octave higher. The optional codes use `#$07` (one more
halving, an octave lower) or `#$08` (two octaves lower). Songs
(`Tracker_handlePitch`) and sound effects (`PitchSlideSFX`) both get
their pitch from `WritePitch`, so both change.

**Evidence:** `alttp_original.smc` has `C8 06 D0 F8` once in the ROM, at
`$19:FD83`. The code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- The loop counts up from the octave until it reaches the new number.
  With `#$05`, a note in octave 6 or above (only possible with the
  transpose commands `$E9` and `$EA`) never matches until `X` wraps
  around. Its pitch is halved 255 times and the note is silent.
- Going up, high notes with a large instrument pitch multiplier can pass
  the DSP's largest pitch and play a wrong note.

## Music and sound effects play with 5 of the 12 notes out of place

**Codes:** `D86B-00CB` `DA6B-014B` `D66B-054B` `DB6C-004B` `D86C-09CB` (use
all 5)
**Raw form:** `1A8592:0B` (orig `$08`); `1A8598:0C` (orig `$0A`);
`1A859C:08` (orig `$0B`); `1A85A0:09` (orig `$0D`); `1A85A6:0B` (orig
`$0F`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-29 01:05:07

**Mechanism:** This is the ALTTP version of the FF2 and SMW codes with
the same name. `WritePitch` (`Bank19.asm`) gets the pitch of each note
from `NotePitchValues`, 12 values from C to B plus the next C, and
halves it for the octave (see "Music and sound effects play an octave
higher"):

```
NotePitchValues:
{
    dw $085F ; 0x00 - Note: C
    dw $08DE ; 0x01 - Note: Cs
    ...
    dw $0FCD ; 0x0B - Note: B
    dw $10BE ; 0x0C - Note: C (next octave)
}
```

The codes change the high byte of 5 of the values, so those notes play
at the wrong pitch in every octave and the other 7 stay put. They move
by the same amounts as in the FF2 code:

| Note | Was | Now | Change |
|---|---|---|---|
| C# | `$08DE` | `$0BDE` | 5 semitones up |
| E | `$0A8C` | `$0C8C` | 3 semitones up |
| F# | `$0BD6` | `$08D6` | 5 semitones down |
| G# | `$0D4A` | `$094A` | 6 semitones down |
| B | `$0FCD` | `$0BCD` | 5 semitones down |

No new value is above `$10BE`, so this adds no notes past the DSP's
largest pitch.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `5F 08 DE 08 65 09 F4 09 8C 0A 2C 0B D6 0B 8B 0C
4A 0D 14 0E EA 0E CD 0F BE 10` once in the ROM, at `$1A:858F`. A script
read back the orig byte at each code's address. The codes decode back to
their raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Sound effects use the same table, so some of them change pitch too.
- Between two notes (pitch slides, vibrato and fine tuning),
  `WritePitch` adds a part of the gap to the next value, using only the
  gap's low byte. Next to a changed value the gap is wrong, so those
  in-between pitches can be off by up to a semitone.

## Music and sound effects play one semitone higher

**Codes:** `66E3-D935` `6665-6946` `DFE3-D145` `DF65-6916` (use all four)
**Raw form:** `19FCE7:88` (orig `$84`); `1A8274:88` (orig `$84`);
`19FCE8:01` (orig `$50`); `1A8275:01` (orig `$50`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Optional, instead, one semitone lower:** `66E3-D935` `6665-6946` `EEE3-D145` `EE65-6916`
(`19FCE8:FF`, `1A8275:FF`)
**Recorded:** 2026-10-29 01:05:07

**Mechanism:** This is the ALTTP version of the FF2 and SMW codes with
the same name. `HandleNote` (`Bank19.asm`) adds the global transpose
(`$50`, set by track command `$E9`) and the channel transpose to every
note:

```
; Apply the global transposition and the channel transposition
; to the note:
mov A, Y : and.b A, #$7F : clrc : adc.b A, $50
clrc : adc.w A, $02F0+X : mov.w $0361+X, A
```

`TrackCommandF9_SlideOnce` does the same for the note it slides to:

```
call GetTrackByte : clrc : adc.b A, $50 : adc.w A, $02F0+X
```

The first two codes change both `adc.b A, $50` (`84 50`) to `adc A,
#imm` (`88`), and the last two set the number to add to `$01`. Every
note is now one semitone higher. The optional codes add `$FF`, which is
one semitone lower. Sound effects also send their notes through
`HandleNote`, so they change too.

**Evidence:** `alttp_original.smc` has `28 7F 60 84 50 60 95 F0 02`
once in the ROM, at `$19:FCE4`, and `60 84 50 95 F0 02 28 7F` once, at
`$1A:8273`. A script read back the orig byte at each code's address.
The codes decode back to their raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Use all four codes. Without the second and fourth, song pitch slides
  end at the old pitch.
- The global transpose is replaced, not added to. A song that sets one
  with `$E9` plays at the cheat's shift instead. The channel transpose
  (`$EA`) still works.
- The sound effect slide-to command (`$F1`, in `SFXControl`) doesn't
  add the transpose, so those slides end at the old pitch.
- Going down, the lowest note (C of octave 0) wraps around to a very
  high note number.
- A one-semitone shift is small. Many players won't notice it unless
  they know the songs well.

## Music and sound effects play a whole tone higher

**Codes:** `66E3-D935` `6665-6946` `D4E3-D145` `D465-6916` (use all four)
**Raw form:** `19FCE7:88` (orig `$84`); `1A8274:88` (orig `$84`);
`19FCE8:02` (orig `$50`); `1A8275:02` (orig `$50`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Optional, instead, a whole tone lower:** `66E3-D935` `6665-6946` `E3E3-D145` `E365-6916`
(`19FCE8:FE`, `1A8275:FE`)
**Recorded:** 2026-10-29 01:05:07

**Mechanism:** The same as "Music and sound effects play one semitone
higher", but it adds `$02` (2 semitones) to every note instead of `$01`.
The optional codes add `$FE`, 2 semitones lower.

**Evidence:** The same bytes as the one-semitone entry. The codes decode
back to their raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- The caveats of the one-semitone entry apply. Going down, the two
  lowest notes wrap around.
- These write the same 4 bytes as the other semitone cheats. Use only
  one.

## Music and sound effects play 2 whole tones higher

**Codes:** `66E3-D935` `6665-6946` `D0E3-D145` `D065-6916` (use all four)
**Raw form:** `19FCE7:88` (orig `$84`); `1A8274:88` (orig `$84`);
`19FCE8:04` (orig `$50`); `1A8275:04` (orig `$50`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Optional, instead, 2 whole tones lower:** `66E3-D935` `6665-6946` `EAE3-D145` `EA65-6916`
(`19FCE8:FC`, `1A8275:FC`)
**Optional, instead, half an octave higher:** `66E3-D935` `6665-6946` `D1E3-D145` `D165-6916`
(`19FCE8:06`, `1A8275:06`)
**Recorded:** 2026-10-29 01:05:07

Tags: `demo-ready` `play-start`

**Mechanism:** The same as "Music and sound effects play one semitone
higher", but it adds `$04` (4 semitones) to every note instead of `$01`.
The optional codes add `$FC` (4 semitones lower) or `$06` (6
semitones, half an octave higher, like the FF2 code "Music and sound
effects play half an octave higher").

**Evidence:** The same bytes as the one-semitone entry. The codes decode
back to their raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- The caveats of the one-semitone entry apply. Going down, the 4 lowest
  notes wrap around.
- Going up, the highest notes can pass the DSP's largest pitch and play
  a wrong note.
- These write the same 4 bytes as the other semitone cheats. Use only
  one.

## Octoroks spit arrows

**Code:** `F822-DFFB`
**Raw form:** `06D4D4:1B` (orig `$0C`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-11-07 19:09:42

Tags: `demo-ready` `curated` `video-ready`

**Mechanism:** This is the SNES version of the NES Zelda code "Octoroks
launch arrows". `Octorock_SpitOutRock` (`Sprites/sprite_octorock.asm`)
spawns sprite `0x0C`, the rock:

```
    LDA.b #$0C
    JSL.l Sprite_SpawnDynamically : BMI .spawn_failed
```

The code changes it to sprite `0x1B`, the enemy arrow that soldiers shoot
and that Moblins throw as a spear. `Sprite_SpawnDynamically` copies the
Octorok's direction (`$0DE0`) to the new sprite, and the Octorok code then
sets its position and speed (44 a frame in the facing direction), the same
fields `Moblin_SpawnThrownSpear` sets for its spear. `$0D90` is 0, which
`EnemyArrow_Draw` draws as an arrow (the Moblin sets 3 for a spear).

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC offset
+ `$200`) has `A9 07 22 7C BB 0D A9 0C 22 5D F6 1D` at PC `$0354CD`
(`$06:D4CD`), so the operand is at `$06:D4D4`. The code decodes back to its
raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- When an arrow (with `$0D90` = 0) hits a wall, `Sprite_EnemyArrow` sticks
  it in the wall for a moment, as for soldiers' arrows. Rocks broke apart
  instead.
- The arrow's tiles may not be loaded in every area where Octoroks live.
  If not, the arrow is drawn with the wrong tiles. It still works the same.
- The arrow is faster than the Moblin's spear (44 against 32) but the same
  speed as the rock was.
