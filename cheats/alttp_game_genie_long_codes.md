# The Legend of Zelda: A Link to the Past (SNES) long Game Genie codes

Cheats here need more codes than a real Game Genie can hold (it takes 5 at
most). Use them in an emulator that takes any number of codes, such as
Mesen. Cheats with 5 codes or fewer are in `alttp_game_genie.md`.

These codes were made from the USA release (version 1.0, the only USA
release). SNES Game Genie codes have no compare byte; the original byte in
each raw form is documentation only. All effects come from the disassembly.
They have not been tested in play unless an entry says so.

## X jumps (the map button)

**Codes:** `696E-D7AC` `026D-0DDC` `696D-0D0C` `4B6D-0D6C` `696D-0DAC`
`016D-0FDC` `1D6D-0F0C` `0AC1-0DDB` `EEC1-0D0B` `66C1-0D6B`
(use all ten)
**Raw form:** `0288FF:85` (orig `$AD`); `028900:4D` (orig `$0C`);
`028901:85` (orig `$04`); `028902:29` (orig `$C9`); `028903:85`
(orig `$FF`); `028904:46` (orig `$F0`); `028905:60` (orig `$15`);
`02A560:4C` (orig `$9C`); `02A561:FF` (orig `$00`); `02A562:88`
(orig `$02`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 19:31:25

**Mechanism:** Link has no jump, but the recoil state (`$5D` = 0x02) is
one. `LinkState_Recoil` (`Bank07.asm`) calls
`Link_HandleChangeInZVelocity`, which takes 2 off the Z speed `$29`
each frame (gravity), and `LinkHop_FindArbitraryLandingSpot` adds `$29`
to Link's height `$24` (in 1/16 pixels). `LinkState_Default` goes to
the recoil state when the aux state `$4D` is not 0. `$46` is the
recoil timer: `Link_HandleRecoilAndTimer` lands Link only when `$46`
has counted down to 0 and he is on the ground. So a jump needs three
writes: `$4D`, `$29` and `$46`.

The map button (X) handlers have room for this. In `Dungeon_Normal`
(`Bank02.asm`):

```
LDA.b $F6 : AND.b #$40 : BEQ .dontActivateMap
    LDA.w $040C : CMP.b #$FF : BEQ .dontActivateMap
```

The first seven codes change the code after the `BEQ` to

```
STA.b $4D : STA.b $29 : STA.b $46 : RTS   ; 85 4D 85 29 85 46 60
```

A still holds `$40` from the `AND`. Link gets a Z speed of `$40` (4
pixels per frame up) and a 64-frame timer. The `RTS` leaves
`Dungeon_Normal` early, so `Player_Main` is skipped for that one frame.
Next frame `LinkState_Default` sees `$4D` and starts the recoil state.
Link rises for 32 frames, about 66 pixels (4 tiles), then falls back
down. Landing is handled as after a hit: a landing sound, and a small
bounce or two from `$02C7` (the Z speed of the last recoil).

`Overworld_PlayerControl` has only 5 bytes before `.changeSubmodule`,
which the Start and Select handlers also use:

```
LDA.b $F6 : AND.b #$40 : BEQ .xButtonNotDown
    STZ.w $0200
    LDA.b #$07
    .changeSubmodule
```

The last three codes change `STZ.w $0200` to `JMP.w $88FF`
(`4C FF 88`), the new code in `Dungeon_Normal`. Both routines are in
bank `$02`, both are called the same way and both end with `RTS`, so the
`RTS` there returns from `Overworld_PlayerControl` too.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has
`A5 F6 29 40 F0 1C AD 0C 04 C9 FF F0 15 A5 A0 F0 11 9C 00 02 A9 03` at
`$02:88F9` (PC `$0108F9`, `Dungeon_Normal`) and
`A5 F6 29 40 F0 11 9C 00 02 A9 07` at `$02:A55A` (PC `$01255A`,
`Overworld_PlayerControl`). Each pattern appears only once in the ROM.
The codes decode back to their raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested. The height and air time are worked out from the
  disassembly.
- This uses X, not Y. Y uses the selected item and is read by each item's
  own code, so there is no one place to change it. X opens the map, so
  the map can't be opened while these codes are on.
- Link can't steer while in the air. He jumps straight up and lands in
  the same place.
- Pressing X again in the air starts the jump again, so tapping X keeps
  Link up.
- The recoil state also clears what Link was doing, the same as a hit:
  he drops out of a sword swing, item use or swimming.
- The jump works only where these handlers run, in normal play on the
  overworld and in dungeons and houses.

## Select moves Link 3 tiles right (overworld)

**Codes:** `E0C5-0D6B` `4FC5-076B` `7DC6-0DDB` `DDC6-0D0B` `19C6-0D6B` `44C6-0DAB`
`69C6-0FDB` `44C6-0F0B` `34C6-0F6B` `4DC6-0FAB` `6DC6-04DB` `D2C6-040B`
(use all twelve)
**Raw form:** `02A572:F4` (orig `$F0`); `02A57E:21` (orig `$20`); `02A580:30`
(orig `$86`); `02A581:00` (orig `$01`); `02A582:65` (orig `$8D`);
`02A583:22` (orig `$F0`); `02A584:85` (orig `$1C`); `02A585:22`
(orig `$E2`); `02A586:E2` (orig `$20`); `02A587:20` (orig `$A5`);
`02A588:80` (orig `$10`); `02A589:0D` (orig `$48`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 19:33:44

Tags: `interesting` `play-start`

**Mechanism:** On the overworld, `Overworld_PlayerControl`
(`Bank02.asm`) opens the save menu while Select is held:

```
LDA.b $F0 : AND.b #$20 : BEQ .selectButtonNotDown
    LDA.w $1CE8 : STA.w $1CF4
    REP #$20
    LDA.w #$0186 : STA.w $1CF0
    SEP #$20
    LDA.b $10 : PHA
    ...
.selectButtonNotDown
```

The first code changes `$F0` (buttons held) to `$F4` (buttons pressed
this frame), so one press moves Link once. The other codes change the
code after `STA.w $1CF4` to

```
REP #$21              ; 16-bit A, and clear the carry.
LDA.w #$0030 : ADC.b $22 : STA.b $22
SEP #$20
BRA .selectButtonNotDown
```

`$22` is Link's X coordinate (16 bits, in pixels), so each press moves
Link 48 pixels (3 16x16 tiles) right. `REP #$21` clears the carry
along with the `M` flag, so no `CLC` is needed. `LDA.w $1CE8 : STA.w $1CF4` still runs;
it only copies a text box value.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has
`A5 F0 29 20 F0 20 AD E8 1C 8D F4 1C C2 20 A9 86 01 8D F0 1C E2 20 A5 10`
at `$02:A571` (PC `$012571`). The pattern appears only once in the
ROM. The `BEQ .selectButtonNotDown` goes to `$02:A597`, so the new
`BRA` at `$02:A588` uses offset `$0D`. The codes decode back to their
raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Overworld only. Select no longer opens the save menu there; it still
  works in dungeons and houses.
- Nothing checks for walls, so Link can land inside a wall, a tree or
  water. Moving past the edge of an area may start a screen transition.
- Uses the same bytes as the other Select teleport and the other
  overworld Select cheats. Use only one of them.
- For a different distance, change `30` in the `02A580` code (the low
  byte of the amount).

## Select moves Link 3 tiles up (overworld)

**Codes:** `E0C5-0D6B` `4FC5-076B` `2DC6-0DDB` `EEC6-0D0B` `19C6-0D6B` `4DC6-0DAB`
`69C6-0FDB` `4DC6-0F0B` `34C6-0F6B` `4DC6-0FAB` `6DC6-04DB` `D2C6-040B`
(use all twelve)
**Raw form:** `02A572:F4` (orig `$F0`); `02A57E:21` (orig `$20`); `02A580:D0`
(orig `$86`); `02A581:FF` (orig `$01`); `02A582:65` (orig `$8D`);
`02A583:20` (orig `$F0`); `02A584:85` (orig `$1C`); `02A585:20`
(orig `$E2`); `02A586:E2` (orig `$20`); `02A587:20` (orig `$A5`);
`02A588:80` (orig `$10`); `02A589:0D` (orig `$48`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 19:33:44

Tags: `interesting` `play-start`

**Mechanism:** On the overworld, `Overworld_PlayerControl`
(`Bank02.asm`) opens the save menu while Select is held:

```
LDA.b $F0 : AND.b #$20 : BEQ .selectButtonNotDown
    LDA.w $1CE8 : STA.w $1CF4
    REP #$20
    LDA.w #$0186 : STA.w $1CF0
    SEP #$20
    LDA.b $10 : PHA
    ...
.selectButtonNotDown
```

The first code changes `$F0` (buttons held) to `$F4` (buttons pressed
this frame), so one press moves Link once. The other codes change the
code after `STA.w $1CF4` to

```
REP #$21              ; 16-bit A, and clear the carry.
LDA.w #$FFD0 : ADC.b $20 : STA.b $20   ; -48
SEP #$20
BRA .selectButtonNotDown
```

`$20` is Link's Y coordinate (16 bits, in pixels), so each press moves
Link 48 pixels (3 16x16 tiles) up. `REP #$21` clears the carry
along with the `M` flag, so no `CLC` is needed. `LDA.w $1CE8 : STA.w $1CF4` still runs;
it only copies a text box value.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has
`A5 F0 29 20 F0 20 AD E8 1C 8D F4 1C C2 20 A9 86 01 8D F0 1C E2 20 A5 10`
at `$02:A571` (PC `$012571`). The pattern appears only once in the
ROM. The `BEQ .selectButtonNotDown` goes to `$02:A597`, so the new
`BRA` at `$02:A588` uses offset `$0D`. The codes decode back to their
raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Overworld only. Select no longer opens the save menu there; it still
  works in dungeons and houses.
- Nothing checks for walls, so Link can land inside a wall, a tree or
  water. Moving past the edge of an area may start a screen transition.
- Uses the same bytes as the other Select teleport and the other
  overworld Select cheats. Use only one of them.
- For a different distance, change `D0` in the `02A580` code (the low
  byte of the amount).

## Select lifts Link 6 tiles into the air (overworld)

**Codes:** `E0C5-0D6B` `69C5-0FAB` `01C5-04DB` `69C5-040B` `02C5-046B`
`CBC5-04AB` `1DC5-07DB` `69C5-070B` `40C5-076B` `6DC5-07AB` `F1C6-0DDB`
(use all eleven)
**Raw form:** `02A572:F4` (orig `$F0`); `02A577:85` (orig `$AD`);
`02A578:46` (orig `$E8`); `02A579:85` (orig `$1C`); `02A57A:4D`
(orig `$8D`); `02A57B:A9` (orig `$F4`); `02A57C:60` (orig `$1C`);
`02A57D:85` (orig `$C2`); `02A57E:24` (orig `$20`); `02A57F:80`
(orig `$A9`); `02A580:16` (orig `$86`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 19:35:13

Tags: `interesting` `play-start`

**Mechanism:** `$24` is Link's height off the ground (Z), in pixels. In
the ground state `LinkState_Default` (`Bank07.asm`) sets it to `$FFFF`
every frame, so a plain write lasts one frame. The recoil state keeps it
and lets gravity pull Link down, as in "X jumps (the map button)":
`LinkState_Default` goes to the recoil state when `$4D` is not 0 (and
clears `$25`, the high byte of `$24`), and `Link_HandleRecoilAndTimer`
lands Link when the timer `$46` has run out and he is on the ground.

On the overworld, `Overworld_PlayerControl` (`Bank02.asm`) opens the
save menu while Select is held:

```
LDA.b $F0 : AND.b #$20 : BEQ .selectButtonNotDown
    LDA.w $1CE8 : STA.w $1CF4
    ...
.selectButtonNotDown
```

The first code changes `$F0` (buttons held) to `$F4` (buttons pressed
this frame). The others change the code after the `BEQ` to

```
STA.b $46 : STA.b $4D     ; A = $20 from the AND.
LDA.b #$60 : STA.b $24
BRA .selectButtonNotDown
```

`Player_Main` runs right after, and Link starts the recoil state 96
pixels (6 16x16 tiles) up. `Link_HandleChangeInZVelocity` takes 2 off
the Z speed `$29` each frame, so he falls back down in about 40 frames
and lands with the normal landing sound. The 32-frame timer runs out
first; `Link_HandleRecoilAndTimer` then keeps it at 1 until he lands.

The height of a fall in normal play is not one fixed number. South
ledge drops (`LinkHop_FindTileToLandOnSouth`) set `$24` to the distance
down to the landing tile, so it depends on the cliff. The checks on `$24`
in `LinkState_Recoil` and `Link_HandleRecoilAndTimer` are 8-bit, and
treat `$80` and up as below the ground, so `$7F` (127 pixels) is the
most these codes can use. `$60` is used here as a tall drop that stays
on screen; change the `02A57C` code for a different height.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A5 F0 29 20 F0 20 AD E8 1C 8D F4 1C C2 20 A9 86`
at `$02:A571` (PC `$012571`). The pattern appears only once in the
ROM. The `BEQ` goes to `$02:A597`, so the new `BRA` at `$02:A57F` uses
offset `$16`. The codes decode back to their raw forms with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested. The fall time is worked out from the disassembly.
- Overworld only. Select no longer opens the save menu there.
- Link can't move while in the air. He lands where he was.
- The recoil state clears what Link was doing, as a hit does.
- Uses the same bytes as the other overworld Select cheats (such as the
  Select teleports). Use only one of them.

## Select warps Link to the other world (overworld)

**Codes:** `E0C5-0D6B` `6DC5-0FAB` `DCC5-04DB` `CBC6-0DAB` `F0C6-0FDB`
`69C6-0F0B` `92C6-0F6B` `6CC6-04DB` `4BC6-040B` `0DC6-046B` `69C6-04AB`
`58C6-07DB` `CBC6-070B` `47C6-076B` `FFCB-0DDB` `1DCB-0DAB`
(use all sixteen)
**Raw form:** `02A572:F4` (orig `$F0`); `02A577:80` (orig `$AD`);
`02A578:0A` (orig `$E8`); `02A583:A9` (orig `$F0`); `02A584:14`
(orig `$1C`); `02A585:85` (orig `$E2`); `02A586:5D` (orig `$20`);
`02A588:8A` (orig `$10`); `02A589:29` (orig `$48`); `02A58A:40`
(orig `$22`); `02A58B:85` (orig `$AA`); `02A58C:7B` (orig `$FD`);
`02A58D:A9` (orig `$0F`); `02A58E:23` (orig `$68`); `02A590:11`
(orig `$10`); `02A593:60` (orig `$A9`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 19:42:17

Tags: `interesting` `play-start`

**Mechanism:** A warp tile or the Magic Mirror starts a world change in
`Mirror_TryToWarp` (`Bank07.asm`). Outdoors it saves the world Link is
in (`$8A AND #$40`) in `$7B`, sets the overworld submodule `$11` to
`0x23` (`Overworld_MirrorWarp`), clears `$B0`, and puts Link in state
`0x14` (`LinkState_CrossingWorlds`). `Overworld_InitMirrorWarp` then
flips `$7EF3CA` between the Light World (`$00`) and the Dark World
(`$40`) and runs the wavy warp. `LinkState_CrossingWorlds` checks where
Link lands and turns him into a bunny in the Dark World if he has no
Moon Pearl.

`Mirror_TryToWarp` is in bank `$07` and is called with `JSR`, so the
Select code in bank `$02` can't call it. These codes do the same
stores there instead. On the overworld, `Overworld_PlayerControl`
(`Bank02.asm`) opens the save menu while Select is held:

```
LDA.b $F0 : AND.b #$20 : BEQ .selectButtonNotDown
    LDA.w $1CE8 : STA.w $1CF4
    ...
.selectButtonNotDown
```

The first code changes `$F0` (buttons held) to `$F4` (buttons pressed
this frame). The others change the code after the `BEQ` to

```
BRA $02A583
...
$02A583:
LDA.b #$14 : STA.b $5D
LDA.b $8A : AND.b #$40 : STA.b $7B
LDA.b #$23 : STA.b $11
STZ.b $B0                 ; Already in the ROM.
RTS
```

The `RTS` ends `Overworld_PlayerControl` for this frame, as the Start
and X handlers do after they change the submodule. The bytes are laid
out to keep as many of the original bytes as they can (`A5`, `85`,
`64 B0`).

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A5 F0 29 20 F0 20 AD E8 1C 8D F4 1C C2 20 A9 86
01 8D F0 1C E2 20 A5 10 48 22 AA FD 0F 68 85 10 64 B0 A9 0B` at
`$02:A571` (PC `$012571`). The pattern appears only once in the ROM.
The new `BRA` at `$02:A577` goes to `$02:A583` (offset `$0A`). The
stores match `Mirror_TryToWarp` at `$07:A95C` (PC `$03A95C`). The codes
decode back to their raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Overworld only. Select no longer opens the save menu there.
- Works both ways: Select in the Dark World warps back to the Light
  World.
- Works before the story reaches the Dark World. Without the Moon Pearl
  Link is a bunny there.
- If Link lands in a wall or deep water without the Flippers, the game's
  own check sends him back (submodule `0x2C`), as with the mirror.
- `Mirror_TryToWarp` also clears `$27`, `$28` and `$03F8` and sets
  `$02DB`; these codes don't. They are 0 in normal walking.
- In the special areas (area `$80` and up, such as the Master Sword
  grove) `Overworld_InitMirrorWarp` stops the warp at once, but Link is
  still put in state `0x14` for a frame. `Mirror_TryToWarp` doesn't
  allow this.
- Uses the same bytes as the other overworld Select cheats (such as the
  Select teleports). Use only one of them.

## Select makes Link do a spin attack (overworld)

**Codes:** `D7C6-0DDB` `DDC6-0D0B` `92C6-0DAB` `DDC6-0FDB` `6DC6-0FAB`
`D3C6-04DB`
(use all six)
**Raw form:** `02A580:03` (orig `$86`); `02A581:00` (orig `$01`);
`02A583:5D` (orig `$F0`); `02A584:00` (orig `$1C`); `02A587:80`
(orig `$A5`); `02A588:0E` (orig `$10`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 19:47:51

Tags: `interesting` `play-start`

**Mechanism:** `$5D` is Link's state, the index into the state table at
the top of `Bank07.asm`. State `0x03` is `LinkState_SpinAttack`. It
steps `$031D` through 12 steps of `Pool_SpinAttack_anim_step`, one row
per facing, starting at row offset `$031E`. It plays the spin sound on
step 2 and goes back to state `0x00` after step 12, clearing `$031D`
for the next spin.

On the overworld, `Overworld_PlayerControl` (`Bank02.asm`) opens the
save menu while Select is held, starting with

```
REP #$20

LDA.w #$0186 : STA.w $1CF0

SEP #$20

LDA.b $10 : PHA
```

The codes change this to

```
REP #$20

LDA.w #$0003 : STA.w $005D

SEP #$20

BRA .selectButtonNotDown
```

A is 16-bit, so `$5E` (Link's speed setting) is set to 0 too, which is
the normal walking speed. The `BRA` (offset `$0E`) skips the save menu.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `C2 20 A9 86 01 8D F0 1C E2 20 A5 10` at
`$02:A57D` (PC `$01257D`). The pattern `A5 F0 29 20 F0 20 AD E8 1C`
before it appears only once in the ROM. The new `BRA` at `$02:A587`
goes to `$02:A597`, `.selectButtonNotDown`. The codes decode back to
their raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Overworld only. Select no longer opens the save menu there.
- The spin does not set up `$031E` as `Link_ActivateSpinAttack` does,
  so it uses the row of the last real spin, or the "up" row if Link has
  not spun yet. There is no start sparkle.
- Holding Select starts a new spin as soon as one ends.
- Not checked whether this spin hits enemies, or what is drawn before
  Link has a sword.
- Uses the same bytes as the other Select cheats. Use only one of them.

## Select puts Link to bed (overworld)

**Codes:** `DFC6-0DDB` `DDC6-0D0B` `5AC6-0DAB` `D7C6-0FDB` `CBC6-0FAB`
`F1C6-04DB` `69C6-040B` `92C6-046B` `6DC6-04AB` `DCC6-07DB`
(use all ten)
**Raw form:** `02A580:01` (orig `$86`); `02A581:00` (orig `$01`);
`02A583:7C` (orig `$F0`); `02A584:03` (orig `$1C`); `02A587:A9`
(orig `$A5`); `02A588:16` (orig `$10`); `02A589:85` (orig `$48`);
`02A58A:5D` (orig `$22`); `02A58B:80` (orig `$AA`); `02A58C:0A`
(orig `$FD`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 19:47:51

Tags: `interesting` `play-start`

**Mechanism:** State `0x16` is `LinkState_Sleeping` (`Bank07.asm`), the
start of a new game. It runs one of three steps by `$037C`:
`Link_SnoringInBed` (0), `Link_SittingUpInBed` (1) and
`Link_JumpingOutOfBed` (2). The player draw code in `Bank0D.asm` draws
the in-bed pose (`$1F`) with `$037D` as the frame, unless `$037D` is 2.

Step 0 only snores and waits for the intro to move it on, so these
codes use step 1. `Link_SittingUpInBed` waits for a new press of the
D-pad or any button but Start (`$F4` and `$F6`), then sets `$037D` to 1
(sitting up) and moves to step 2. `Link_JumpingOutOfBed` waits 4 frames and makes Link hop
out, down and to the right, in the recoil state, as at the start of the
game.

On the overworld, `Overworld_PlayerControl` (`Bank02.asm`) opens the
save menu while Select is held. The codes change it to

```
REP #$20

LDA.w #$0001 : STA.w $037C    ; $037C = 1, $037D = 0.

SEP #$20

LDA.b #$16 : STA.b $5D
BRA .selectButtonNotDown
```

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `C2 20 A9 86 01 8D F0 1C E2 20 A5 10 48 22 AA FD`
at `$02:A57D` (PC `$01257D`). The pattern `A5 F0 29 20 F0 20 AD E8 1C`
before it appears only once in the ROM. The new `BRA` at `$02:A58B`
goes to `$02:A597` (offset `$0A`), `.selectButtonNotDown`. The codes
decode back to their raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Overworld only. Select no longer opens the save menu there.
- There is no bed or bedspread (`AddLinksBedSpread` isn't called) and
  no Zzz, which only step 0 makes.
- Link lies down for as long as Select is held. The button that wakes
  him has to be pressed after Select is let go.
- Uses the same bytes as the other Select cheats. Use only one of them.

## The Super Bomb is always following Link

**Codes:** `DD67-ADD6` `D267-AFD6` `9624-DFAF` `6D24-D76F` `DD29-DD0F`
`4F29-D40F`
(use all six)
**Raw form:** `028330:00` (orig `$0D`); `028334:0D` (orig `$00`);
`00D427:58` (orig `$64`); `00D42E:80` (orig `$F0`); `00D451:00`
(orig `$FF`); `00D459:21` (orig `$07`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 20:01:49

Tags: `interesting` `play-start`

**Mechanism:** `$7EF3CC` is the follower (tagalong). `$0D` is the
Super Bomb. Two codes give Link the bomb and four load its graphics.

`DD67-ADD6` `D267-AFD6`: `Module_PreDungeon` (`Bank02.asm`) runs each
time Link goes into a house, cave or dungeon, and when a game is loaded
indoors. It is where the game throws the Super Bomb away:

```
LDA.l $7EF3CC : CMP.b #$0D : BNE .notSuperBombTagalong
    ; If we saved the game having a super bomb, sad to say it's going to be
    ; gone.
    LDA.b #$00 : STA.l $7EF3CC
```

The codes change the `#$0D` to `#$00` and the `#$00` to `#$0D`. Now,
when there is no follower, the game gives Link the Super Bomb, and a
bomb Link already has is no longer thrown away. The code after it calls
`Tagalong_Init`, which puts the bomb next to Link.

The other four codes change `Tagalong_LoadGfx` (`Bank00.asm`).
`Module_LoadFile` calls it when a game is loaded, before the follower is
set, so without them the bomb would be drawn with the wrong graphics.

```
LDY.b #$64

;  If your tagalong is princess zelda...
LDA.l $7EF3CC : CMP.b #$01 : BEQ .doDecomp
...
LDA.l $7EF3CC : AND.w #$00FF : ASL : TAX
LDA.b $00 : CLC : ADC.l .GFX_offset, X
```

- `9624-DFAF` changes `LDY.b #$64` to `#$58`, the graphics set the
  game picks for followers `0x0C` and `0x0D`.
- `6D24-D76F` changes the `BEQ` to `BRA`, so set `$58` is always used.
- `DD29-DD0F` changes `AND.w #$00FF` to `AND.w #$0000`, so X is always 0.
- `4F29-D40F` moves the `ADC.l` from `LoadFollowerGraphics_GFX_offset`
  (`$00:D407`) to its `0x0D - Super bomb` row (`$00:D421`, `$0900`).

So the bomb follows Link from his bed at the start. After it goes off,
it comes back the next time Link goes indoors.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has:

- `AF CC F3 7E C9 0D D0 0D A9 00 8F CC` at `$02:832B` (PC `$01032B`)
  in `Module_PreDungeon`. The `$0D` is at `$02:8330` and the `$00`
  at `$02:8334`. The pattern appears only once in the ROM.
- `8B 4B AB A0 64 AF CC F3 7E C9 01 F0 12` at `$00:D423` (PC
  `$005423`), the start of `Tagalong_LoadGfx`. The `$64` is at
  `$00:D427` and the `F0` at `$00:D42E`.
- `AF CC F3 7E 29 FF 00 0A AA A5 00 18 7F 07 D4 00` at `$00:D44C`. The
  `FF` is at `$00:D451` and the `07` at `$00:D459`.
- `00 09` (`$0900`) at `$00:D421`, the Super Bomb row of
  `LoadFollowerGraphics_GFX_offset`.

The codes decode back to their raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- The bomb's timer starts and the bomb goes off only outdoors
  (`Tagalong_NoTimedMessage` and `Tagalong_NotFollowing` check `$1B`).
  Indoors, pressing A puts the bomb down, and Link picks it up again by
  walking to it.
- The A button puts the bomb down outdoors as in the normal game, so
  talking, reading signs and lifting also light the fuse.
- When the uncle leaves the house, he sets the follower to Zelda's
  telepathy (`0x05`) and the bomb is gone. The wounded uncle in the
  secret passage clears that, so the bomb comes back at the next door.
- The first two codes add the bomb only when there is no follower, so
  they don't replace Zelda, the old man or any other follower. But the
  last four codes draw every follower with the bomb's graphics.
- The game's own Super Bomb events, such as blowing open the cracked
  wall of the Pyramid of Power, are not checked with a bomb this early.

## The Lamp throws the boomerang, shoots an arrow, pounds the hammer and sprinkles powder

**Codes:** `4DC1-642A` `3DC1-67FA` `CDC1-679A` `4DC1-67BA` `0BC1-672A`
`CDC5-6DFA` `4DC5-6D9A` `A6C5-6DBA` `BEC5-6D2A` `6DC5-6FFA` `DFC5-6F9A`
(use all eleven)
**Raw form:** `07A26B:20` (orig `$22`); `07A26C:E0` (orig `$EC`);
`07A26D:A0` (orig `$F3`); `07A26E:20` (orig `$01`); `07A26F:49` (orig
`$A0`); `07A270:A0` (orig `$02`); `07A271:20` (orig `$A9`);
`07A272:C8` (orig `$2F`); `07A273:9F` (orig `$22`); `07A274:80` (orig
`$1C`); `07A275:01` (orig `$8F`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 20:13:54

Tags: `interesting` `play-start`

**Mechanism:** With the Lamp on Y, one press of Y now does the work of
four items at once: the boomerang, the bow, the hammer and the Magic
Powder.

The Lamp's code, `LinkItem_Lamp` (`Bank07.asm`), already sprinkles
powder. On a new press of Y, if Link has the Lamp and enough magic, it
does:

```
LDY.b #$00
LDA.b #$1A
JSL.l AddMagicPowder

JSL.l Dungeon_LightTorch

LDY.b #$02
LDA.b #$2F
JSL.l AddLampFlame
```

The codes replace the last two calls (12 bytes, `22 EC F3 01 A0 02 A9 2F
22 1C 8F 09`) with:

```
JSR $A0E0 ; 20 E0 A0 - the boomerang throw in LinkItem_Boomerang
JSR $A049 ; 20 49 A0 - the arrow shot in LinkItem_Bow
JSR $9FC8 ; 20 C8 9F - the pound in LinkItem_Hammer
BRA +1     ; 80 01    - skip the old last byte
```

Each `JSR` goes into the middle of that item's own code, just where the
item does its work, and that code ends in an `RTS`:

- `$07:A0E0` is `LDY.b #$00 : LDA.b #$05 : JSL.l AddBoomerang`, after the
  check for a boomerang already in the air. The code after it ends the
  throw (`.reset`) and returns.
- `$07:A049` copies Link's position to `$72`-`$75`, then does `LDX.b
  $2F : LDY.b #$02 : LDA.b #$09 : JSL.l AddArrow`. The code after it takes
  away one arrow, or cancels the arrow with the error sound if Link has
  none, then ends the shot.
- `$07:9FC8` is `PHX : LDY.b #$03 : JSR.w TileDetect_MainHandler`, then
  the hit stars, the pound sound and the water splash. Then it returns.

The Lamp's code after them clears `$0301`, `$3A`, `$3C` and `$50` as
before.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has, at `$07:A24D` (PC `$03A24D`, `LinkItem_Lamp`):
`A5 6C D0 37 20 73 B0 90 32 AF 4A F3 7E F0 1B A2 06 20 AB B0 90 14 A0 00
A9 1A 22 F0 92 09 22 EC F3 01 A0 02 A9 2F 22 1C 8F 09 9C 01 03 64 3A`. The
12 bytes start at `$07:A26B` and appear only once in the ROM. The
targets:

- `$07:A0E0`: `A0 00 A9 05 22 0F 82 09 A5 3C C9 09 B0 31`.
- `$07:A049`: `A5 20 85 72 A5 21 85 73 A5 22 85 74 A5 23 85 75 A6 2F A0 02
  A9 09 22 A4 90 09`. It ends with `60` at `$07:A0BA`.
- `$07:9FC8`: `DA A0 03 20 77 D0 A0 00 A9 16 22 24 80 09 FA`. It ends with
  `60` at `$07:A002`.

The new `BRA` at `$07:A274` goes to the `9C` (`STZ.w $0301`) at
`$07:A277`. The ROM differs from the source here: the hammer calls
`AddHitStars` with `JSL` (`22 24 80 09`), not `JSR`. The bytes above are
from the ROM. The codes decode back to their raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Use all eleven codes. With only some of them, the game runs broken
  instructions.
- The Lamp no longer lights torches or shows its flame. It still costs
  magic and needs the Lamp.
- Arrows need arrows. With none, the arrow is cancelled and the error
  sound plays, as with the bow.
- `AddBoomerang` reads the boomerang type from `$7EF341` minus 1. With no
  boomerang that is `$FF`, and the throw distance, speed and look come
  from past the end of their tables.
- The pound uses the hammer's tile check (`TileDetect_MainHandler` with
  `Y = 3`), so it should flatten pegs in front of Link. Enemies are hit
  by the hammer's swing in the normal game, which is not run here, so
  the pound may not hurt them.
- The boomerang check for one already in the air is skipped. Not
  checked how many can be out at once.

## Fireballs shot by most enemy types

**Codes:** `F6EC-DDF3` `6CEC-DD93` `19EC-DDB3` `FCEC-DD23` `4BEC-DFF3`
`7EEC-DF93` `2DEC-DFB3` `D0EC-DF23` `44EC-D4F3` `D1EC-D493` `2CEC-D4B3`
`D2EC-D423` `9AEC-D7F3` `4FEC-D793` `EFEC-D7B3` `D1EC-D723` `CDCB-A4F1`
`E6CB-A491` `D5CB-A4B1` `CDAF-6980` `E6AF-69E0` `D5AF-6170` (use all
twenty-two)
The first sixteen write a new routine into unused ROM. Of the last six,
the first three cover the bank $05 enemies and the last three the bank
$1D enemies. Either group of three also works alone, with the first
sixteen.
**Raw form:** `07F8A0:18` `07F8A1:8A` `07F8A2:65` `07F8A3:1A`
`07F8A4:29` `07F8A5:3F` `07F8A6:D0` `07F8A7:04` `07F8A8:22` `07F8A9:06`
`07F8AA:DA` `07F8AB:0D` `07F8AC:5C` `07F8AD:21` `07F8AE:F1` `07F8AF:06`
(all orig `$FF`); `05AB98:A0` (orig `$21`); `05AB99:F8` (orig `$F1`);
`05AB9A:07` (orig `$06`); `1DC216:A0` (orig `$21`); `1DC217:F8` (orig
`$F1`); `1DC218:07` (orig `$06`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-11-02 21:09:03

Tags: `demo-ready` `play-start` `video-ready`

**Mechanism:** This is a rate-limited version of "Fireballs
continuously shot by most enemy types", which spawns a fireball every
frame. It uses the same two wrappers (`Sprite2_CheckDamage` in
`Sprites/sprite_cannon_trooper.asm`, `Sprite4_CheckDamage` in
`Bank1D.asm`):

```
Sprite2_CheckDamage:
    JSL.l Sprite_CheckDamageFromPlayerLong
    JSL.l Sprite_CheckDamageToPlayerLong

    RTS
```

The last six codes point the second `JSL` at a new routine at
`$07:F8A0`, in the free space at the end of bank $07:

```
    CLC
    TXA                         ; sprite slot
    ADC.b $1A                   ; frame counter
    AND.b #$3F : BNE +
        JSL.l Sprite_SpawnFireball
    +
    JML.l Sprite_CheckDamageToPlayerLong
```

Each enemy shoots one fireball every 64 frames (about once a second).
Adding the sprite slot to the frame counter spreads the shots out, so
enemies don't all fire on the same frame. The routine then jumps on to
`Sprite_CheckDamageToPlayerLong`, so enemies still hurt Link by touching
him, and the wrapper returns what it did before. The same enemies shoot
as in the other entry.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `$FF` in all 16 bytes at `$07:F8A0` (PC
`$03F8A0`). `Bank07.asm` marks `$03F89D-$03FFFF` as "NULL (Can be used
for expansion)". It has `22 21 F1 06` at both `$05:AB97` and
`$1D:C215`, the second `JSL` in each wrapper. `$1A` is the frame counter
(`INC.b $1A` in the main loop in `Bank00.asm`). The sprite code runs with
8-bit A and index registers, and X holds the sprite slot.
`Sprite_SpawnFireball` keeps X (`PHX` ... `PLX`) and sets its own data
bank. The codes decode back to their raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- The byte at `07F8A5` sets the rate. To change it, use one of these in
  place of `7EEC-DF93` (`$3F`, every 64 frames): `FEEC-DF93` (`$1F`,
  every 32 frames) or `5EEC-DF93` (`$7F`, every 128 frames).
- With many enemies on screen, the fireballs can still fill the free
  sprite slots for a while.
- Uses the same wrapper bytes as "Fireballs continuously shot by most
  enemy types".
  Use only one of the two.
- `Sprite2_CheckDamage` also runs for the tutorial soldiers and some
  NPC-like sprites, so they shoot too.

## Fireballs continuously shot by most enemy types

**Codes:** `D1CB-A4F1` `2CCB-A491` `D2CB-A4B1` `D1AF-6980` `2CAF-69E0`
`D2AF-6170` (use all six)
The first three cover the bank $05 enemies and the last three the bank
$1D enemies. Each group of three also works alone.
**Raw form:** `05AB98:06` (orig `$21`); `05AB99:DA` (orig `$F1`);
`05AB9A:0D` (orig `$06`); `1DC216:06` (orig `$21`); `1DC217:DA` (orig
`$F1`); `1DC218:0D` (orig `$06`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 21:20:38

Tags: `interesting` `demo-ready` `play-start` `video-ready`

**Mechanism:** Enemies in banks $05 and $1D call a local wrapper once a
frame to check damage both ways (`Sprite2_CheckDamage` in
`Sprites/sprite_cannon_trooper.asm`, `Sprite4_CheckDamage` in
`Bank1D.asm`):

```
Sprite2_CheckDamage:
    JSL.l Sprite_CheckDamageFromPlayerLong
    JSL.l Sprite_CheckDamageToPlayerLong

    RTS
```

The codes change the second `JSL` in each wrapper to
`JSL.l Sprite_SpawnFireball` (`$0D:DA06`), the routine Zoras and
Medusas use. It spawns a sprite `0x55` fireball at the enemy (sprite
slot in X) and aims it at Link. These enemies now shoot a fireball every
frame while a sprite slot is free:

- Bank $05: rats, crabs, Keese, Ropes, bomb soldiers, Armos, the Armos
  Knights, Lanmola, Zoras, Toppos, spike rollers and others.
- Bank $1D: Lynels, Tektites, Vultures, Ravens, Hokbok, Chain Chomps,
  Stals, Trinexx, Vitreous, Ganon and others.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `22 AA F2 06 22 21 F1 06 60` at both `$05:AB93`
(PC `$02AB93`) and `$1D:C211` (PC `$0EC211`): `JSL $06F2AA : JSL $06F121 :
RTS`. `$06:F121` starts `8B 4B AB 20 45 F1`
(`Sprite_CheckDamageToPlayerLong`), and `$0D:DA06` starts `8B 4B AB A9`
(`Sprite_SpawnFireball`). The codes decode back to their raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- These enemies no longer hurt Link by touching him. Only their
  fireballs, swords and other attacks do.
- The fireballs fill every free sprite slot, so new enemies, drops and
  other sprites may fail to spawn. The fireball sound plays almost
  every frame.
- Enemies in bank $06 (soldiers, Octoroks and most early enemies) and
  bank $1E (`Sprite3_CheckDamage`) are not changed. Changing
  `Sprite_CheckDamageToPlayerLong` itself would also hit the fireball,
  which uses it to hurt Link, so this entry does not.
- `Sprite2_CheckDamage` also runs for the tutorial soldiers and some
  NPC-like sprites, so they shoot too.

## Link climbs stairs faster (dungeons)

**Codes:** `F034-649A` `F437-6D9A` `F6A8-A4D0` `41B9-DFAB` `4FB1-DF6B`
`4D67-0768` `F660-0708` `4667-A4A8` `4160-AF08` `4668-AFD8` `4168-A768`
(use all eleven)
**Raw form:** `07E229:14` (orig `$0A`); `07E231:12` (orig `$09`);
`01C3B8:18` (orig `$20`); `029457:26` (orig `$3C`); `029466:21` (orig
`$32`); `028D3E:20` (orig `$30`); `028D4D:18` (orig `$20`);
`028F3B:28` (orig `$3C`); `028F45:26` (orig `$38`); `028FB4:28` (orig
`$3C`); `028FBE:26` (orig `$38`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 21:30:45

Tags: `interesting`

**Mechanism:** On stairs, `$57` is 1, so `Link_HandleVelocity`
(`Bank07.asm`) takes Link's speed from the stairs entries of
`SubVelocityValues` and adds `$57`:

```
db $0A ; 0x02 - walking on stairs
...
db $09 ; 0x0A - intraroom stairs
```

During a stair transition, Link walks for as many frames as the step
counter `$0464` says. When the counter reaches a set value (`0x10`, or
`0x14` for stairs inside a room), `$57` becomes 2 and Link slows down
for the last frames:

```
LDA.w $0464 : BEQ .counterElapsed
    DEC.w $0464

    CMP.b #$10 : BNE .noSlow
        ; When the counter is down to 0x10, slow the player sprite down.
        LDA.b #$02 : STA.b $57
```

The first two codes about double the stair speed (`$0B` to `$15`, and
`$0A` to `$13`). The other nine halve the part of each counter before
the slowdown, so Link covers about the same distance in fewer frames:

| Stairs | Routine | Counter |
|---|---|---|
| Straight stairs between rooms, walking in | `Dungeon_DetectStaircase` | `$20` → `$18` |
| Straight stairs, walking out (up / down) | `StraightStairs_11` | `$3C` → `$26`, `$32` → `$21` |
| Wide stairs between rooms (up / down) | `DungeonTransition_AdjustForFatStairScroll` | `$30` → `$20`, `$20` → `$18` |
| Stairs inside a room, north | `Dungeon_NorthIntraRoomStairs_InitStairs` | `$3C` → `$28`, `$38` → `$26` |
| Stairs inside a room, south | `Dungeon_SouthIntraRoomStairs_InitStairs` | `$3C` → `$28`, `$38` → `$26` |

Link also walks faster on ordinary stair tiles in a room, which use the
same speed entries.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`):

- `$07:E227` (PC `$03E227`): `18 10 0A 18 10 08 08 04 0C 10 09 19`
  (`SubVelocityValues`).
- `$01:C3B0`: `38 F0 04 C9 39 D0 13 A2 20 8E 64 04` (`CMP #$39 : BNE :
  LDX #$20 : STX $0464`).
- `$02:9452`: `A9 01 85 57 A2 3C AD 62 04 29 04 F0 08 A0 19 C6 A4 C6 A4
  A2 32 8E 64`.
- `$02:8D3B`: `A0 17 A2 30 AD 62 04 29 04 F0 08 A0 19 C6 A4 C6 A4 A2 20
  8E 64 04`.
- `$02:8F38` and `$02:8FB1`: `A0 19 A2 3C`; `$02:8F42` and `$02:8FBB`:
  `A0 17 A2 38`.

The codes decode back to their raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Only the walking part gets shorter, by about 8 to 22 frames per
  staircase. The fades and the room loading take as long as before.
- Spiral staircases are not changed. Link's walk there is set by his
  position (`HandleLinkOnSpiralStairs`), not by `$0464`, so a shorter
  counter could start the next step before Link arrives.
- Link's speed is in 1/16 pixels, so the distances are close but not
  exact. If Link stops a pixel short or long, he may stand on a stair
  tile or a little off where he normally stops.

## Every enemy moves with the speed of the sprite in the next slot

**Codes:** `0F37-072C` `0F30-072C` `0FED-6D91` `0FEF-6D91` `0FE7-A958`
`0FE0-A958` `0F39-0581` `0F31-0581`
(use all eight)
**Raw form:** `06E93F:41` (orig `$40`); `06E94F:41` (orig `$40`);
`05FA01:41` (orig `$40`); `05FA11:41` (orig `$40`); `1EFF35:41`
(orig `$40`); `1EFF45:41` (orig `$40`); `1DE95E:41` (orig `$40`);
`1DE96E:41` (orig `$40`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 23:28:40

Tags: `interesting`

**Mechanism:** This is the ALttP version of the SMW code "Every enemy
steals the horizontal speed of the sprite in the next slot". Each sprite
bank has its own copy of the vertical move routine: `Sprite_MoveVert`
(`Bank06.asm`), `Sprite2_MoveVert` (`Bank05.asm`), `Sprite3_MoveVert`
(`Bank1E.asm`) and `Sprite4_MoveVert` (`Bank1D.asm`). It reads the
speed from `$0D40,X` twice, once for the fraction and once for the
whole pixels:

```
Sprite_MoveVert:
    LDA.w $0D40, X : BEQ .return          <- $40 patched to $41
        ASL #4 : CLC : ADC.w $0D60, X : STA.w $0D60, X
        LDA.w $0D40, X : PHP              <- $40 patched to $41
        LSR #4
        ...
              ADC.w $0D00, X : STA.w $0D00, X
        TYA : ADC.w $0D20, X : STA.w $0D20, X
```

The matching `MoveHoriz` routine adds `$10` to X and runs the same
code, so `$0D40,X` becomes `$0D50,X` (the X speed). The codes change
both reads to `$0D41,X` in all four copies. Each sprite now moves by the
speed of the **next sprite slot up**, on both axes, but still changes
only its own position. The sprite's own speed is not changed, so its AI
works as normal; only the movement uses the other sprite's speed. An
enemy above an empty slot moves with that slot's leftover speed, and
often does not move at all.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has the same 46 bytes at all four routines, `$06:E93E`
(PC `$03693E`), `$05:FA00` (PC `$02FA00`), `$1E:FF34` (PC
`$0F7F34`) and `$1D:E95D` (PC `$0EE95D`):
`BD 40 0D F0 28 0A 0A 0A 0A 18 7D 60 0D 9D 60 0D BD 40 0D 08 4A ...`.
The two `BD 40 0D` (`LDA $0D40,X`) operands are at routine + 1 and
routine + `$11`. The byte at each code's address was read back as
`$40`. The codes decode back to their raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- There is no code for only the horizontal or only the vertical speed.
  One routine moves both axes, so both are always changed. This entry
  covers both TODO items.
- Slot `0x0F` has no next slot. It reads slot 0's X speed as its Y
  speed, and slot 0's `$0D60` fraction as its X speed, so it can drift
  oddly.
- Sprites that move with their own code, not these routines, are not
  changed.
- 8 codes is more than a real Game Genie holds. For fewer codes, use
  only the first four (banks `$06` and `$05`); sprites in the other
  two banks then move normally.

## Every enemy moves the sprite in the next slot onto itself

**Codes:** `DF31-0DBC` `4F31-049C` `DFE4-6FF1` `4FE4-6421` `DFE9-A178`
`4FE9-A5E8` `DF36-0051` `4F36-0171`
(use all eight)
**Raw form:** `06E962:01` (orig `$00`); `06E969:21` (orig `$20`);
`05FA24:01` (orig `$00`); `05FA2B:21` (orig `$20`); `1EFF58:01`
(orig `$00`); `1EFF5F:21` (orig `$20`); `1DE981:01` (orig `$00`);
`1DE988:21` (orig `$20`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 23:29:57

Tags: `interesting`

**Mechanism:** This is the Y-position companion of "Every enemy moves the
sprite in the next slot sideways instead of itself" (in
`alttp_game_genie.md`). That code changes the `ADC.b #$10` in each
`MoveHoriz` routine. Nothing like it exists for the vertical move, so
these codes take another path. The end of each bank's vertical move
routine (`Sprite_MoveVert` in `Bank06.asm`, `Sprite2_MoveVert` in
`Bank05.asm`, `Sprite3_MoveVert` in `Bank1E.asm`, `Sprite4_MoveVert`
in `Bank1D.asm`) adds the speed to the sprite's Y position:

```
              ADC.w $0D00, X : STA.w $0D00, X   <- STA $0D00 patched to $0D01
        TYA : ADC.w $0D20, X : STA.w $0D20, X   <- STA $0D20 patched to $0D21
```

The codes change only the two stores, in all four copies. The game still
adds the sprite's own speed to its own position, but writes the result
into the **next sprite slot up** (`$0D00`/`$0D20` are the Y low/high
bytes, 16 slots each). The sprite's own position is never changed by its
own move. `MoveHoriz` runs the same code with X + `$10`, so the stores
there become `$0D11`/`$0D31`: the next sprite's X position. So each
moving sprite puts the sprite in the next slot just ahead of itself, on
both axes. Enemies should pile up on top of each other, following the
lowest moving slot, which itself can't move.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `9D 00 0D` (`STA $0D00,X`) at `$06:E961`,
`$05:FA23`, `$1E:FF57` and `$1D:E980`, and `9D 20 0D` (`STA
$0D20,X`) 7 bytes later in each. These are inside the four 46-byte
`MoveVert` routines at `$06:E93E` (PC `$03693E`), `$05:FA00` (PC
`$02FA00`), `$1E:FF34` (PC `$0F7F34`) and `$1D:E95D` (PC
`$0EE95D`), which are byte for byte the same. The byte at each code's
address was read back as the orig byte. The codes decode back to their
raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- There is no Y-only code. One routine moves both axes, so X is
  changed too.
- Slot `0x0F` has no next slot. Its Y move writes slot 0's X position
  (`$0D10`/`$0D30`), and its X move writes slot 0's Y high byte and Y
  speed (`$0D20`/`$0D40`). Slot 0 may jump a whole screen and
  despawn. The game may lock up.
- Sprites can be pushed into walls, since the move that places them
  skips their own tile checks.
- Sprites that set their positions with their own code are not changed.
- 8 codes is more than a real Game Genie holds. The first four (banks
  `$06` and `$05`) work alone; sprites in the other banks then move
  normally.

## Enemies drop a small key instead of a green rupee

**Codes:** `30E5-6FBC` `30E5-649C` `30E5-642C` `30E5-672C` `30EB-642C`
`30EC-6DFC` `30E6-079C`
(use all seven)
**Raw form:** `06FA76:E4` (orig `$D9`); `06FA79:E4` (orig `$D9`);
`06FA7B:E4` (orig `$D9`); `06FA7F:E4` (orig `$D9`); `06FA9B:E4`
(orig `$D9`); `06FAA0:E4` (orig `$D9`); `06F98D:E4` (orig `$D9`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 23:31:28

Tags: `interesting`

**Mechanism:** When an enemy dies, `Sprite_DoTheDeath` (`Bank06.asm`)
picks its drop and turns the dead sprite into that item. Most drops come
from `PrizePack_Prizes`, 8 entries for each prize pack. Sprite `$D9`
is the green rupee and `$E4` is the small key. The green rupee is in
three packs:

```
.pack_1  db $D8, $D8, $D8, $D8, $D9, $D8, $D8, $D9
.pack_2  db $DA, $D9, $DA, $DB, $DA, $D9, $DA, $DA
.pack_6  db $DF, $D9, $D8, $E1, $DF, $DC, $D9, $D8
```

The first six codes change those six `$D9` bytes to `$E4`. The
seventh code is for sprites that the room data sets to drop one green
rupee (`$0CBA` = 3):

```
                ; Green rupee.
                LDA.b #$D9              <- $D9 patched to $E4
                BRA PrepareEnemyDrop
```

`.PrepareEnemyDrop` already checks for `$E4` and calls
`SpritePrep_Key_set_item_drop` for it, as it does for a real key drop. So
every green rupee an enemy would drop is a small key instead.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `PrizePack_Prizes` at `$06:FA72` (PC
`$037A72`): `D8 D8 D8 D8 D9 D8 D8 D9 DA D9 DA DB DA D9 DA DA ...`,
with exactly six `$D9` bytes, at the six addresses above. At
`$06:F980` it has `A9 E4 C0 01 F0 4B A9 E5 C0 03 D0 45 A9 D9`, the
small key, big key and green rupee loads of `Sprite_DoTheDeath`. The
byte at each code's address was read back as `$D9`. The codes decode
back to their raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Each key adds to the current dungeon's key count. Outdoors, where
  there is no real key count, the effect of picking one up was not
  checked.
- Keys use `ItemDropBounceProps` for small keys, so they bounce as
  keys do, not as rupees.
- Green rupees from bushes, pots, chests and NPCs are not changed. Only
  enemy drops are.
- For 5 codes or fewer, use the codes for one or two packs; the other
  packs then still drop green rupees.

## Every enemy jumps onto the sprite in the next slot

**Codes:** `DF39-072C` `4F31-0FBC` `DFE4-6D91` `4FE4-64F1` `DFE9-A958`
`4FE9-A578` `DF35-0581` `4F36-0951`
(use all eight)
**Raw form:** `06E95F:01` (orig `$00`); `06E966:21` (orig `$20`);
`05FA21:01` (orig `$00`); `05FA28:21` (orig `$20`); `1EFF55:01` (orig `$00`);
`1EFF5C:21` (orig `$20`); `1DE97E:01` (orig `$00`); `1DE985:21` (orig `$20`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 23:49:53

Tags: `interesting`

**Mechanism:** Each sprite bank has its own copy of the vertical move routine:
`Sprite_MoveVert` (`Bank06.asm`), `Sprite2_MoveVert` (`Bank05.asm`),
`Sprite3_MoveVert` (`Bank1E.asm`) and `Sprite4_MoveVert` (`Bank1D.asm`).
The sprite tables are 16 bytes each, one byte per slot (`$0D00` Y low,
`$0D10` X low, `$0D20` Y high, `$0D30` X high, `$0D40` Y speed, `$0D50` X
speed, `$0D60`/`$0D70` the fractions). The matching `MoveHoriz` adds
`$10` to X and runs the same code, so each change below works on both
axes:

```
Sprite_MoveVert:
    LDA.w $0D40, X : BEQ .return                       ; speed
        ASL #4 : CLC : ADC.w $0D60, X : STA.w $0D60, X ; fraction
        LDA.w $0D40, X : PHP                           ; speed
        LSR #4
        ...
              ADC.w $0D00, X : STA.w $0D00, X          ; position, low
        TYA : ADC.w $0D20, X : STA.w $0D20, X          ; position, high
```

The codes change the two position reads (`ADC.w $0D00, X` and
`ADC.w $0D20, X`) to `$0D01` and `$0D21` in all four copies. The game now
adds the sprite's speed to the position of the **next sprite slot up**,
and stores the result as the sprite's own position. So every moving
sprite jumps onto the sprite in the next slot (plus one frame of its own
speed), on each axis it is moving on. The sprite in the next slot is not
moved. This is the reverse of "Every enemy moves the sprite in the next
slot onto itself".

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has the same 46 bytes at all four `MoveVert` routines,
`$06:E93E` (PC `$03693E`), `$05:FA00` (PC `$02FA00`), `$1E:FF34` (PC
`$0F7F34`) and `$1D:E95D` (PC `$0EE95D`):
`BD 40 0D F0 28 0A 0A 0A 0A 18 7D 60 0D 9D 60 0D BD 40 0D 08 4A 4A 4A 4A
A0 00 28 10 03 09 F0 88 7D 00 0D 9D 00 0D 98 7D 20 0D 9D 20 0D 60`. A
script checked each instruction's opcode and table address at its offset
before patching, and read the byte at each code's address back as the
orig byte. The codes decode back to their raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- `MoveVert` returns early when the speed is 0, so a sprite that stands still
  on an axis does not jump on that axis.
- Slot `0x0F` has no next slot. It reads slot 0's X position as its Y
  position, and slot 0's Y high byte and Y speed as its X position. It can
  jump far away and despawn.
- A sprite that jumps onto an empty slot takes that slot's leftover position.
- Do not use with "Every enemy moves the sprite in the next slot onto itself"
  or the other next-slot position codes: they change the same routine.
- Sprites that move with their own code, not these routines, are not changed.
- 8 codes is more than a real Game Genie holds. The codes come in banks
  (`$06`, `$05`, `$1E`, `$1D`, a quarter each). Using only the first half
  (banks `$06` and `$05`) leaves sprites in the other banks moving normally.

## Every enemy pushes the sprite in the next slot instead of moving itself

**Codes:** `DF39-072C` `DF31-0DBC` `4F31-0FBC` `4F31-049C` `DFE4-6D91`
`DFE4-6FF1` `4FE4-64F1` `4FE4-6421` `DFE9-A958` `DFE9-A178` `4FE9-A578`
`4FE9-A5E8` `DF35-0581` `DF36-0051` `4F36-0951` `4F36-0171`
(use all sixteen)
**Raw form:** `06E95F:01` (orig `$00`); `06E962:01` (orig `$00`);
`06E966:21` (orig `$20`); `06E969:21` (orig `$20`); `05FA21:01` (orig `$00`);
`05FA24:01` (orig `$00`); `05FA28:21` (orig `$20`); `05FA2B:21` (orig `$20`);
`1EFF55:01` (orig `$00`); `1EFF58:01` (orig `$00`); `1EFF5C:21` (orig `$20`);
`1EFF5F:21` (orig `$20`); `1DE97E:01` (orig `$00`); `1DE981:01` (orig `$00`);
`1DE985:21` (orig `$20`); `1DE988:21` (orig `$20`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 23:49:56

Tags: `demo-ready` `interesting`

**Mechanism:** Each sprite bank has its own copy of the vertical move routine:
`Sprite_MoveVert` (`Bank06.asm`), `Sprite2_MoveVert` (`Bank05.asm`),
`Sprite3_MoveVert` (`Bank1E.asm`) and `Sprite4_MoveVert` (`Bank1D.asm`).
The sprite tables are 16 bytes each, one byte per slot (`$0D00` Y low,
`$0D10` X low, `$0D20` Y high, `$0D30` X high, `$0D40` Y speed, `$0D50` X
speed, `$0D60`/`$0D70` the fractions). The matching `MoveHoriz` adds
`$10` to X and runs the same code, so each change below works on both
axes:

```
Sprite_MoveVert:
    LDA.w $0D40, X : BEQ .return                       ; speed
        ASL #4 : CLC : ADC.w $0D60, X : STA.w $0D60, X ; fraction
        LDA.w $0D40, X : PHP                           ; speed
        LSR #4
        ...
              ADC.w $0D00, X : STA.w $0D00, X          ; position, low
        TYA : ADC.w $0D20, X : STA.w $0D20, X          ; position, high
```

The codes change all four position operands (`ADC`/`STA $0D00, X` and
`ADC`/`STA $0D20, X`) to `$0D01` and `$0D21` in all four copies. The game
still uses the sprite's own speed and fraction, but adds them to the
position of the **next sprite slot up** and stores the result there. So
each sprite moves the next sprite by its own speed, on both axes, and
does not move itself. A sprite is only moved by the sprite in the slot
below it. This is the both-axes version of "Every enemy moves the sprite
in the next slot sideways instead of itself", except that the push uses
the pushing sprite's speed.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has the same 46 bytes at all four `MoveVert` routines,
`$06:E93E` (PC `$03693E`), `$05:FA00` (PC `$02FA00`), `$1E:FF34` (PC
`$0F7F34`) and `$1D:E95D` (PC `$0EE95D`):
`BD 40 0D F0 28 0A 0A 0A 0A 18 7D 60 0D 9D 60 0D BD 40 0D 08 4A 4A 4A 4A
A0 00 28 10 03 09 F0 88 7D 00 0D 9D 00 0D 98 7D 20 0D 9D 20 0D 60`. A
script checked each instruction's opcode and table address at its offset
before patching, and read the byte at each code's address back as the
orig byte. The codes decode back to their raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Slot `0x0F` has no next slot. Its Y move changes slot 0's X position, and
  its X move changes slot 0's Y high byte and Y speed. Slot 0 may jump a
  whole screen and despawn. The game may lock up.
- The lowest moving slot is never moved, so it stands still while it walks.
- These codes include all 8 codes of "Every enemy moves the sprite in the next
  slot onto itself" and all 8 of "Every enemy jumps onto the sprite in the
  next slot". Do not use them together with those.
- Sprites that move with their own code, not these routines, are not changed.
- 16 codes is far more than a real Game Genie holds. The codes come in banks
  (`$06`, `$05`, `$1E`, `$1D`, a quarter each). Using only the first half
  (banks `$06` and `$05`) leaves sprites in the other banks moving normally.

## Every enemy moves with the speed of the sprite in the previous slot

**Codes:** `7E37-072C` `7E30-072C` `7EED-6D91` `7EEF-6D91` `7EE7-A958`
`7EE0-A958` `7E39-0581` `7E31-0581`
(use all eight)
**Raw form:** `06E93F:3F` (orig `$40`); `06E94F:3F` (orig `$40`);
`05FA01:3F` (orig `$40`); `05FA11:3F` (orig `$40`); `1EFF35:3F` (orig `$40`);
`1EFF45:3F` (orig `$40`); `1DE95E:3F` (orig `$40`); `1DE96E:3F` (orig `$40`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 23:54:05

Tags: `demo-ready` `video-ready` `interesting`

**Mechanism:** Each sprite bank has its own copy of the vertical move
routine: `Sprite_MoveVert` (`Bank06.asm`), `Sprite2_MoveVert`
(`Bank05.asm`), `Sprite3_MoveVert` (`Bank1E.asm`) and `Sprite4_MoveVert`
(`Bank1D.asm`). The sprite tables are 16 bytes each, one byte per slot
(`$0D00` Y low, `$0D10` X low, `$0D20` Y high, `$0D30` X high, `$0D40` Y
speed, `$0D50` X speed, `$0D60`/`$0D70` the fractions). The matching
`MoveHoriz` adds `$10` to X and runs the same code, so each change below
works on both axes:

```
Sprite_MoveVert:
    LDA.w $0D40, X : BEQ .return                       ; speed
        ASL #4 : CLC : ADC.w $0D60, X : STA.w $0D60, X ; fraction
        LDA.w $0D40, X : PHP                           ; speed
        LSR #4
        ...
              ADC.w $0D00, X : STA.w $0D00, X          ; position, low
        TYA : ADC.w $0D20, X : STA.w $0D20, X          ; position, high
```

The codes change both speed reads (`LDA.w $0D40, X`) to `$0D3F` in all four
copies. Each sprite now moves by the speed of the **next sprite slot down**,
on both axes, but still changes only its own position. The sprite's own
speed is not changed, so its AI works as normal; only the movement uses the
other sprite's speed. An enemy above an empty slot moves with that slot's
leftover speed, and often does not move at all.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has the same 46 bytes at all four `MoveVert` routines,
`$06:E93E` (PC `$03693E`), `$05:FA00` (PC `$02FA00`), `$1E:FF34` (PC
`$0F7F34`) and `$1D:E95D` (PC `$0EE95D`):
`BD 40 0D F0 28 0A 0A 0A 0A 18 7D 60 0D 9D 60 0D BD 40 0D 08 4A 4A 4A 4A
A0 00 28 10 03 09 F0 88 7D 00 0D 9D 00 0D 98 7D 20 0D 9D 20 0D 60`. A
script checked each instruction's opcode and table address at its offset
before patching, and read the byte at each code's address back as the
orig byte. Each table address is lowered by 1. For `$0D00` that is
`$0CFF`, so both operand bytes change (`00` → `FF`, `0D` → `0C`); the
other tables only need the low byte. The codes decode back to their raw
form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Slot 0 has no previous slot. Its Y move reads slot `0x0F`'s X high byte
  (`$0D3F`) as a speed, and its X move reads slot `0x0F`'s Y speed. It can
  drift slowly downward or to the right.
- Do not use with "Every enemy moves with the speed of the sprite in the next
  slot": they change the same bytes.
- Sprites that move with their own code, not these routines, are not changed.
- 8 codes is more than a real Game Genie holds. The codes come in banks
  (`$06`, `$05`, `$1E`, `$1D`, a quarter each). Using only the first half
  (banks `$06` and `$05`) leaves sprites in the other banks moving normally.

## Every enemy moves the sprite in the previous slot onto itself

**Codes:** `EE31-0DBC` `DA31-0D2C` `FE31-049C` `EEE4-6FF1` `DAE4-6F91`
`FEE4-6421` `EEE9-A178` `DAE9-A158` `FEE9-A5E8` `EE36-0051` `DA36-0081`
`FE36-0171`
(use all twelve)
**Raw form:** `06E962:FF` (orig `$00`); `06E963:0C` (orig `$0D`);
`06E969:1F` (orig `$20`); `05FA24:FF` (orig `$00`); `05FA25:0C` (orig `$0D`);
`05FA2B:1F` (orig `$20`); `1EFF58:FF` (orig `$00`); `1EFF59:0C` (orig `$0D`);
`1EFF5F:1F` (orig `$20`); `1DE981:FF` (orig `$00`); `1DE982:0C` (orig `$0D`);
`1DE988:1F` (orig `$20`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 23:54:05

Tags: `interesting`

**Mechanism:** Each sprite bank has its own copy of the vertical move
routine: `Sprite_MoveVert` (`Bank06.asm`), `Sprite2_MoveVert`
(`Bank05.asm`), `Sprite3_MoveVert` (`Bank1E.asm`) and `Sprite4_MoveVert`
(`Bank1D.asm`). The sprite tables are 16 bytes each, one byte per slot
(`$0D00` Y low, `$0D10` X low, `$0D20` Y high, `$0D30` X high, `$0D40` Y
speed, `$0D50` X speed, `$0D60`/`$0D70` the fractions). The matching
`MoveHoriz` adds `$10` to X and runs the same code, so each change below
works on both axes:

```
Sprite_MoveVert:
    LDA.w $0D40, X : BEQ .return                       ; speed
        ASL #4 : CLC : ADC.w $0D60, X : STA.w $0D60, X ; fraction
        LDA.w $0D40, X : PHP                           ; speed
        LSR #4
        ...
              ADC.w $0D00, X : STA.w $0D00, X          ; position, low
        TYA : ADC.w $0D20, X : STA.w $0D20, X          ; position, high
```

The codes change the two position stores (`STA.w $0D00, X` and `STA.w $0D20,
X`) to `$0CFF` and `$0D1F` in all four copies. The game still adds the
sprite's own speed to its own position, but writes the result into the
**next sprite slot down**. The sprite's own position is never changed by its
own move. So each moving sprite puts the sprite in the previous slot just
ahead of itself, on each axis it is moving on. Enemies should pile up,
following the highest moving slot, which itself can't move.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has the same 46 bytes at all four `MoveVert` routines,
`$06:E93E` (PC `$03693E`), `$05:FA00` (PC `$02FA00`), `$1E:FF34` (PC
`$0F7F34`) and `$1D:E95D` (PC `$0EE95D`):
`BD 40 0D F0 28 0A 0A 0A 0A 18 7D 60 0D 9D 60 0D BD 40 0D 08 4A 4A 4A 4A
A0 00 28 10 03 09 F0 88 7D 00 0D 9D 00 0D 98 7D 20 0D 9D 20 0D 60`. A
script checked each instruction's opcode and table address at its offset
before patching, and read the byte at each code's address back as the
orig byte. Each table address is lowered by 1. For `$0D00` that is
`$0CFF`, so both operand bytes change (`00` → `FF`, `0D` → `0C`); the
other tables only need the low byte. The codes decode back to their raw
form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Slot 0 has no previous slot. Slot 0's Y position low byte goes to `$0CFF`,
  the high byte of `SprFrozenPal`, which the game expects to be 0. Sprites
  may get the wrong palette. Its Y high byte goes to slot `0x0F`'s X low
  byte, so slot `0x0F` can jump sideways. Its X move writes slot `0x0F`'s Y
  position.
- Do not use with the other next-slot or previous-slot position codes: they
  change the same routine.
- Sprites that move with their own code, not these routines, are not changed.
- 12 codes is more than a real Game Genie holds. The codes come in banks
  (`$06`, `$05`, `$1E`, `$1D`, a quarter each). Using only the first half
  (banks `$06` and `$05`) leaves sprites in the other banks moving normally.

## Every enemy jumps onto the sprite in the previous slot

**Codes:** `EE39-072C` `DA31-0DFC` `FE31-0FBC` `EEE4-6D91` `DAE4-6DB1`
`FEE4-64F1` `EEE9-A958` `DAE9-A988` `FEE9-A578` `EE35-0581` `DA35-05E1`
`FE36-0951`
(use all twelve)
**Raw form:** `06E95F:FF` (orig `$00`); `06E960:0C` (orig `$0D`);
`06E966:1F` (orig `$20`); `05FA21:FF` (orig `$00`); `05FA22:0C` (orig `$0D`);
`05FA28:1F` (orig `$20`); `1EFF55:FF` (orig `$00`); `1EFF56:0C` (orig `$0D`);
`1EFF5C:1F` (orig `$20`); `1DE97E:FF` (orig `$00`); `1DE97F:0C` (orig `$0D`);
`1DE985:1F` (orig `$20`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 23:54:05

Tags: `interesting`

**Mechanism:** Each sprite bank has its own copy of the vertical move
routine: `Sprite_MoveVert` (`Bank06.asm`), `Sprite2_MoveVert`
(`Bank05.asm`), `Sprite3_MoveVert` (`Bank1E.asm`) and `Sprite4_MoveVert`
(`Bank1D.asm`). The sprite tables are 16 bytes each, one byte per slot
(`$0D00` Y low, `$0D10` X low, `$0D20` Y high, `$0D30` X high, `$0D40` Y
speed, `$0D50` X speed, `$0D60`/`$0D70` the fractions). The matching
`MoveHoriz` adds `$10` to X and runs the same code, so each change below
works on both axes:

```
Sprite_MoveVert:
    LDA.w $0D40, X : BEQ .return                       ; speed
        ASL #4 : CLC : ADC.w $0D60, X : STA.w $0D60, X ; fraction
        LDA.w $0D40, X : PHP                           ; speed
        LSR #4
        ...
              ADC.w $0D00, X : STA.w $0D00, X          ; position, low
        TYA : ADC.w $0D20, X : STA.w $0D20, X          ; position, high
```

The codes change the two position reads (`ADC.w $0D00, X` and `ADC.w $0D20,
X`) to `$0CFF` and `$0D1F` in all four copies. The game now adds the
sprite's speed to the position of the **next sprite slot down**, and stores
the result as the sprite's own position. So every moving sprite jumps onto
the sprite in the previous slot (plus one frame of its own speed), on each
axis it is moving on. The sprite in the previous slot is not moved.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has the same 46 bytes at all four `MoveVert` routines,
`$06:E93E` (PC `$03693E`), `$05:FA00` (PC `$02FA00`), `$1E:FF34` (PC
`$0F7F34`) and `$1D:E95D` (PC `$0EE95D`):
`BD 40 0D F0 28 0A 0A 0A 0A 18 7D 60 0D 9D 60 0D BD 40 0D 08 4A 4A 4A 4A
A0 00 28 10 03 09 F0 88 7D 00 0D 9D 00 0D 98 7D 20 0D 9D 20 0D 60`. A
script checked each instruction's opcode and table address at its offset
before patching, and read the byte at each code's address back as the
orig byte. Each table address is lowered by 1. For `$0D00` that is
`$0CFF`, so both operand bytes change (`00` → `FF`, `0D` → `0C`); the
other tables only need the low byte. The codes decode back to their raw
form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- `MoveVert` returns early when the speed is 0, so a sprite that stands still
  on an axis does not jump on that axis.
- Slot 0 has no previous slot. Its Y move reads `$0CFF` (the high byte of
  `SprFrozenPal`, normally 0) as its Y low byte and slot `0x0F`'s X low byte
  as its Y high byte. It can jump far away and despawn. Its X move jumps it
  onto slot `0x0F`'s Y position.
- A sprite that jumps onto an empty slot takes that slot's leftover position.
- Do not use with the other next-slot or previous-slot position codes: they
  change the same routine.
- Sprites that move with their own code, not these routines, are not changed.
- 12 codes is more than a real Game Genie holds. The codes come in banks
  (`$06`, `$05`, `$1E`, `$1D`, a quarter each). Using only the first half
  (banks `$06` and `$05`) leaves sprites in the other banks moving normally.

## Every enemy pushes the sprite in the previous slot instead of moving itself

**Codes:** `EE39-072C` `DA31-0DFC` `EE31-0DBC` `DA31-0D2C` `FE31-0FBC`
`FE31-049C` `EEE4-6D91` `DAE4-6DB1` `EEE4-6FF1` `DAE4-6F91` `FEE4-64F1`
`FEE4-6421` `EEE9-A958` `DAE9-A988` `EEE9-A178` `DAE9-A158` `FEE9-A578`
`FEE9-A5E8` `EE35-0581` `DA35-05E1` `EE36-0051` `DA36-0081` `FE36-0951`
`FE36-0171`
(use all twenty-four)
**Raw form:** `06E95F:FF` (orig `$00`); `06E960:0C` (orig `$0D`);
`06E962:FF` (orig `$00`); `06E963:0C` (orig `$0D`); `06E966:1F` (orig `$20`);
`06E969:1F` (orig `$20`); `05FA21:FF` (orig `$00`); `05FA22:0C` (orig `$0D`);
`05FA24:FF` (orig `$00`); `05FA25:0C` (orig `$0D`); `05FA28:1F` (orig `$20`);
`05FA2B:1F` (orig `$20`); `1EFF55:FF` (orig `$00`); `1EFF56:0C` (orig `$0D`);
`1EFF58:FF` (orig `$00`); `1EFF59:0C` (orig `$0D`); `1EFF5C:1F` (orig `$20`);
`1EFF5F:1F` (orig `$20`); `1DE97E:FF` (orig `$00`); `1DE97F:0C` (orig `$0D`);
`1DE981:FF` (orig `$00`); `1DE982:0C` (orig `$0D`); `1DE985:1F` (orig `$20`);
`1DE988:1F` (orig `$20`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 23:54:05

Tags: `interesting`

**Mechanism:** Each sprite bank has its own copy of the vertical move
routine: `Sprite_MoveVert` (`Bank06.asm`), `Sprite2_MoveVert`
(`Bank05.asm`), `Sprite3_MoveVert` (`Bank1E.asm`) and `Sprite4_MoveVert`
(`Bank1D.asm`). The sprite tables are 16 bytes each, one byte per slot
(`$0D00` Y low, `$0D10` X low, `$0D20` Y high, `$0D30` X high, `$0D40` Y
speed, `$0D50` X speed, `$0D60`/`$0D70` the fractions). The matching
`MoveHoriz` adds `$10` to X and runs the same code, so each change below
works on both axes:

```
Sprite_MoveVert:
    LDA.w $0D40, X : BEQ .return                       ; speed
        ASL #4 : CLC : ADC.w $0D60, X : STA.w $0D60, X ; fraction
        LDA.w $0D40, X : PHP                           ; speed
        LSR #4
        ...
              ADC.w $0D00, X : STA.w $0D00, X          ; position, low
        TYA : ADC.w $0D20, X : STA.w $0D20, X          ; position, high
```

The codes change all four position operands (`ADC`/`STA $0D00, X` and
`ADC`/`STA $0D20, X`) to `$0CFF` and `$0D1F` in all four copies. The game
still uses the sprite's own speed and fraction, but adds them to the
position of the **next sprite slot down** and stores the result there. So
each sprite moves the previous sprite by its own speed, on both axes, and
does not move itself. A sprite is only moved by the sprite in the slot above
it.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has the same 46 bytes at all four `MoveVert` routines,
`$06:E93E` (PC `$03693E`), `$05:FA00` (PC `$02FA00`), `$1E:FF34` (PC
`$0F7F34`) and `$1D:E95D` (PC `$0EE95D`):
`BD 40 0D F0 28 0A 0A 0A 0A 18 7D 60 0D 9D 60 0D BD 40 0D 08 4A 4A 4A 4A
A0 00 28 10 03 09 F0 88 7D 00 0D 9D 00 0D 98 7D 20 0D 9D 20 0D 60`. A
script checked each instruction's opcode and table address at its offset
before patching, and read the byte at each code's address back as the
orig byte. Each table address is lowered by 1. For `$0D00` that is
`$0CFF`, so both operand bytes change (`00` → `FF`, `0D` → `0C`); the
other tables only need the low byte. The codes decode back to their raw
form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Slot 0 has no previous slot. Its Y move changes `$0CFF` and slot `0x0F`'s X
  low byte. `$0CFF` is the high byte of `SprFrozenPal`, which the game
  expects to be 0, so sprites may get the wrong palette. Its X move pushes
  slot `0x0F` up or down.
- The highest moving slot is never moved, so it stands still while it walks.
- These codes include all codes of "Every enemy moves the sprite in the
  previous slot onto itself" and "Every enemy jumps onto the sprite in the
  previous slot". Do not use them together with those, or with the next-slot
  position codes.
- Sprites that move with their own code, not these routines, are not changed.
- 24 codes is more than a real Game Genie holds. The codes come in banks
  (`$06`, `$05`, `$1E`, `$1D`, a quarter each). Using only the first half
  (banks `$06` and `$05`) leaves sprites in the other banks moving normally.

## A good bee comes out each time Link enters a room or area

**Codes:** `44A9-6440` `AEA9-6410` `2AA9-64C0` `F3A9-6430` `CBA9-6740`
`DD2D-0558`
(use all six)
**Raw form:** `09C258:22` (orig `$9C`); `09C259:CF` (orig `$88`);
`09C25A:DC` (orig `$0B`); `09C25B:1E` (orig `$9C`); `09C25C:A9` (orig `$99`);
`1EDD0D:00` (orig `$05`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 23:59:29

Tags: `interesting`

**Mechanism:** `Sprite_DisableAll` (`Bank09.asm`) clears every sprite
slot and resets some sprite variables. The game calls it each time it
loads the sprites of a new room or area: `Dungeon_ResetSprites` (every
dungeon room change and staircase), `Sprite_ResetAll` and
`Sprite_OverworldReloadAll`. After the slot loop it zeroes a list of
variables:

```
    STZ.w $0B6A : STZ.w $0B9B : STZ.w $0B88 : STZ.w $0B99
                               <- patched to JSL $1EDCCF : LDA.b #$0B
```

The first five codes change `STZ.w $0B88 : STZ.w $0B99` (`9C 88 0B 9C
99 0B`) into `JSL PlayerItem_ReleaseBee : LDA.b #$0B` (`22 CF DC 1E A9
0B`). `PlayerItem_ReleaseBee` (`$1E:DCCF`) is what the bottle calls to
let a bee out. It spawns sprite `0xB2` next to Link in the first free
slot from `0x0F` down, sets its own data bank, and keeps X. Since it runs
right after the slots are cleared, the bee gets slot `0x0F`. Dungeon
rooms load their sprites into slots from `0x00` up, and the overworld
fills only free slots, so the room's own sprites leave the bee alone. The
`LDA.b #$0B` only uses up the last byte of the old `STZ`; A is loaded
again before it is used.

The sixth code is the one from "Good bees keep coming out and attack
enemies": it sets the `BNE .not_good_bee` offset in
`PlayerItem_ReleaseBee` to `$00`, so the bee is always a good bee
(`$0EB0` = 1), which attacks enemies.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `9C EC 02 9C 6A 0B 9C 9B 0B 9C 88 0B 9C 99 0B 9C
B4 0F` at `$09:C24F` (PC `$04C24F`), inside `Sprite_DisableAll`
(`$09:C22F`-`$09:C28F`). The slot loop before it ends with `DEX : BPL`,
so X is `$FF` at the hook. `Sprite_SpawnDynamically` reads the
parent's tables with X but writes only to the new slot (Y), and
`PlayerItem_ReleaseBee` sets the bee's floor (`$0F20`) itself. `$1E:DCCF`
(PC `$0F5CCF`) starts with `8B 4B AB A9 B2 22 5D F6 1D` (`PHB : PHK :
PLB : LDA.b #$B2 : JSL Sprite_SpawnDynamically`), and the bottle code
in `Bank07.asm` calls it with `22 CF DC 1E`. At `$1E:DD0A` it has `C9
08 D0 05 A9 01 99 B0 0E`. The byte at each code's address was read back
as the orig byte. The codes decode back to their raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Walking from one overworld screen to the next does not make a bee.
  That path (`Overworld_LoadNewSprites`) loads the new sprites without
  `Sprite_DisableAll`. Going in or out of a door, cave or dungeon,
  every dungeon room change, the mirror, the whirlpools, and getting up
  after a game over do make one.
- `Module_PreDungeon` calls `Sprite_DisableAll` twice. The first bee is
  cleared by the second call, so there is still only one bee.
- `$0B88` (archery hit count) and `$0B99` (archery arrows left) are no
  longer zeroed on a new area. The archery game sets both when it starts.
  If Link leaves the archery game with arrows left, the next few arrows
  he shoots outside are refunded.
- A dungeon room with 16 sprites loads over the bee's slot.
- Link can catch the bee with the bug net, so this is also a free good
  bee for an empty bottle.
- The sixth code also makes every bottle bee a good bee. Leave it off to
  get a normal bee.

## Link gives off sleeping Zs all the time

**Codes:** `DD2B-DD75` `4F2B-D775` `B82C-DD75` `442C-DD55` `A62C-DD85`
`6D2C-DDE5` `DB2C-DF75`
(use all seven)
**Raw form:** `0DDC90:00` (orig `$17`); `0DDC9C:21` (orig `$20`);
`0DDCA0:9B` (orig `$A9`); `0DDCA1:22` (orig `$2B`); `0DDCA2:C8` (orig `$8D`);
`0DDCA3:80` (orig `$2E`); `0DDCA4:09` (orig `$01`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-26 00:11:01

Tags: `interesting`

**Mechanism:** In bed, `Link_SnoringInBed` (`Bank07.asm`) makes a Z every
32 frames with `LDY.b #$01 : LDA.b #$21 : JSL.l AddLinksSleepZs`.
`AddLinksSleepZs` (`$09:80C8`, `Ancilla/ancilla_init.asm`) adds ancilla
A (`0x21`, `Ancilla_SleepIcon`) next to Link's head. Y + 1 is how many of
that ancilla may be out at once. These codes make the low-health beep call
it instead.

`HUD_RefillLogic` (`Bank0D.asm`) runs every frame of play. When health
is under the `HeartBeepThresholds` entry for Link's heart count, it plays
the beep every 32 frames:

```
    LDA.l $7EF36C : LSR #3 : TAX
    LDA.l $7EF36D : CMP.w HeartBeepThresholds, X : BCS .doneWithWarningBeep
        LDA.w $04CA : BNE .decrementBeepTimer
            LDA.w $012E : BNE .doneWithWarningBeep
                LDA.b #$20 : STA.w $04CA
                LDA.b #$2B : STA.w $012E
        .decrementBeepTimer
        DEC.w $04CA
```

The first code sets the `BCS` offset to `$00`, so the beep code runs at
any health. The other six change the last 10 bytes, `A9 20 8D CA 04 A9 2B
8D 2E 01`, to `A9 21 8D CA 04 9B 22 C8 80 09`:

```
                LDA.b #$21 : STA.w $04CA   ; timer, and the ancilla ID
                TXY                        ; Y = Link's heart count
                JSL.l AddLinksSleepZs
```

So every 33 frames a Z comes out of Link, wherever he is. `$21` is both
the beep timer and the ID of the Z ancilla. X holds Link's heart count
(3 to 20), so `TXY` gives a limit of at least 4 Zs at once. A Z ends when
it is `$18` pixels above Link, so only one or two are out at a time.
The beep sound is gone. X is set again before the code after it uses it.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `B0 17 AD CA 04 D0 0F AD 2E 01 D0 0D A9 20 8D CA 04
A9 2B 8D 2E 01 CE CA 04` at `$0D:DC8F` (PC `$06DC8F`), just after `CMP
$DB60,X` (`HeartBeepThresholds` is at PC `$06DB60`). `Link_SnoringInBed`
has `A0 01 A9 21 22 C8 80 09` at `$07:9A68` (PC `$039A68`), so the
`JSL` bytes are the same as the game's own call. `$09:80C8` starts with
`8B 4B AB 20` (`PHB : PHK : PLB : JSR AddAncilla`). The byte at each
code's address was read back as the orig byte. The codes decode back to
their raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Without the first code, the Zs come only when health is low, in place
  of the beep.
- No Zs while Link is frozen (`$02E4`), while hearts refill, or on a
  frame when another sound is waiting in `$012E`.
- Each Z uses one of the 5 ancilla slots for a short time. When all
  slots are busy, an item may not work for a moment, or the Z is skipped.
- `Ancilla_SleepIcon` writes `$0109` each frame to animate its tiles.
  Other effects that use `$0109` may show the wrong tiles while a Z is
  out.
- 7 codes is more than a real Game Genie holds.

## The boomerang picks up any sprite

**Codes:** `446C-AD96` `B26C-ADB6` `E66C-AD26` `D56C-AFF6` `3C6C-AF96`
`3C6C-AFB6` `4464-6DC7` `2964-6D37` `E664-6F47` `D564-6F17` `3C64-6FC7`
`BD64-6F37` `63EB-D793` `CDEB-D7B3` `DEEB-D723` `82EC-DDF3` `2DEC-DD93`
`D2EC-DDB3` `ABEC-DD23` `DBEC-DFF3` `2DEC-DF93` `45EC-DFB3` `8AEC-DF23`
`40EC-D4F3` `D4EC-D493` `EDEC-D4B3` `49EC-D423` `8BEC-D7F3` `0BEC-D793`
`DAEC-D7B3` `EDEC-D723` `FCE8-DDF3` `8BE8-DD93` `D7E8-DDB3` `DAE8-DD23`
`B2E8-DFF3` `FDE8-DF93` `D2E8-DFB3` `8BE8-DF23` `F5E8-D4F3` `DAE8-D493`
`B2E8-D4B3` `7DE8-D423` `D2E8-D7F3` `8BE8-D793` `EBE8-D7B3` `D8E8-D723`
`B2EA-DDF3` `DDEA-DD93` `D2EA-DDB3` `8BEA-DD23` `D2EA-DFF3` `DAEA-DF93`
`B2EA-DFB3` `4DEA-DF23` `D2EA-D4F3` `6DEA-D493` `D1EA-D4B3` `B3EA-D423`
`2DEA-D7F3` `D2EA-D793` `B3EA-D7B3` `40EA-D723` `D4E2-DDF3` `9AE2-DD93`
`2CE2-DDB3` `60E2-DD23` `D1E2-DFF3` `8BE2-DF93` `CCE2-DFB3` `DAE2-DF23`
`4BE2-D4F3` `D4E2-D493` `2DE2-D4B3` `F0E2-D423` `6CE2-D7F3` `FCE2-D793`
`BBE2-D7B3` `40E2-D723` `D4E3-DDF3` `8BE3-DD93` `0DE3-DDB3` `D3E3-DD23`
`DBE3-DFF3` `6DE3-DF93` `BBE3-DFB3` `0DE3-DF23` `D3E3-D4F3` `16E3-D493`
`16E3-D4B3` `16E3-D423` `9AE3-D7F3` `11E3-D793` `63E3-D7B3` `D6E3-D723`
`76EE-DDF3` `18EE-DD93` (use all 97)
**Raw form:** `0683A1:22` (orig `$8E`); `0683A2:9D` (orig `$A0`);
`0683A3:F8` (orig `$0F`); `0683A4:07` (orig `$20`); `0683A5:EA` (orig `$E2`);
`0683A6:EA` (orig `$84`); `088E22:22` (orig `$B9`); `088E23:D5` (orig `$AA`);
`088E24:F8` (orig `$0C`); `088E25:07` (orig `$29`); `088E26:EA` (orig `$02`);
`088E27:90` (orig `$F0`); and 85 bytes at `07F89D`-`07F8F1` (orig all
`$FF`), one code per byte, in order:
`8E A0 0F BD D0 0D C9 09 D0 27 BC 24 02 F0 25 B9 49 0C F0 1A B9 03 0C 9D 10
0D B9 17 0C 9D 30 0D B9 F9 0B 9D 00 0D B9 0D 0C 9D 20 0D 80 06 9E D0 0D 9E
24 02 5C DA 84 06 B9 AA 0C 29 02 D0 14 8A 1A 99 24 02 B9 40 0E 09 80 99 40
0E 68 68 68 5C 66 8E 08 38 6B`
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-26 11:04:11

**Mechanism:** In the normal game only items, hearts and fairies follow
the boomerang. `Ancilla_CheckIndividualSpriteCollision` (`Bank08.asm`)
starts a drag only for sprites with `$0CAA` bit 1 set:

```
LDA.w $0CAA, Y : AND.b #$02 : BEQ .not_draggable_sprite
    .is_arrghus_spawn
    TXA : INC : STA.w $0DA0, Y
    BRA .indicate_dragging_ancilla
```

and only those sprites call `Sprite_HandleDraggingByAncilla`, which reads
`$0DA0` to copy the ancilla's position each frame. Enemies use `$0DA0` for
their own state, so these codes use the free RAM at `$0224` (16 bytes,
one per sprite slot; `Free_224` in `WRAM.asm`) as a second drag flag and
add new code in the unused `$FF` block at the end of bank `$07`
(`NULL_07F89D`, "Can be used for expansion").

Hook 1 changes the 7 bytes above (`$08:8E22`) to `JSL $07F8D5 : NOP :
BCC .not_draggable_sprite`. The new routine at `$07:F8D5` is:

```
LDA.w $0CAA, Y : AND.b #$02 : BNE .item  ; Items: normal drag via $0DA0.
TXA : INC : STA.w $0224, Y               ; Flag = ancilla slot + 1.
LDA.w $0E40, Y : ORA.b #$80 : STA.w $0E40, Y  ; Make the sprite harmless.
PLA : PLA : PLA                          ; Drop the JSL return address.
JML $088E66                              ; .indicate_dragging_ancilla
.item
SEC : RTL                                ; Fall into the old drag code.
```

So the sprite is not damaged or stunned. It takes the same exit as a
dragged item, and the boomerang turns back as it does for an item.

Hook 2 changes `STX.w $0FA0 : JSR.w Sprite_ExecuteSingle` in the
`Sprite_Main` slot loop (`$06:83A1`) to `JSL $07F89D : NOP : NOP`. The new
routine at `$07:F89D` runs once per sprite slot each frame, after
`Ancilla_Main`:

```
STX.w $0FA0
LDA.w $0DD0, X : CMP.b #$09 : BNE .drop  ; Not active: clear the flag.
LDY.w $0224, X : BEQ .run                ; Not carried.
LDA.w $0C49, Y : BEQ .absorb             ; $0C4A: the ancilla has ended.
LDA.w $0C03, Y : STA.w $0D10, X          ; Copy the ancilla X and Y
LDA.w $0C17, Y : STA.w $0D30, X          ; ($0C04, $0C18, $0BFA, $0C0E)
LDA.w $0BF9, Y : STA.w $0D00, X          ; to the sprite, as
LDA.w $0C0D, Y : STA.w $0D20, X          ; Sprite_HandleDraggingByAncilla does.
BRA .run
.absorb
STZ.w $0DD0, X                           ; Remove the sprite.
.drop
STZ.w $0224, X
.run
JML Sprite_ExecuteSingleLong             ; Its RTL goes back to the loop.
```

The `- 1` in the addresses makes up for the `+ 1` in the flag. The sprite
then runs its own code at the boomerang's spot and is drawn there. When
the boomerang gets back to Link and ends, the sprite is removed, like an
item that Link picks up from the boomerang.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `B9 AA 0C 29 02 F0 07 8A 1A 99 A0 0D 80 36` at
`$08:8E22` (PC `$040E22`), and the `BRA` lands on `BD 4A 0C 99 B0 0B`
(`LDA.w $0C4A, X : STA.w $0BB0, Y`) at `$08:8E66`. `Sprite_Main` has
`A2 0F 8E A0 0F 20 E2 84 CA 10 F7` at `$06:839F`; the `BPL` goes back to
the `STX` at `$06:83A1`. `Sprite_ExecuteSingleLong` (PC `$0304DA`) is
`8B 4B AB 20 E2 84 AB 6B` at `$06:84DA`. All 85 bytes at `$07:F89D`
(PC `$03F89D`) are `$FF`. No code in the disassembly reads or writes
`$0224`-`$0233`. `$0000`-`$1FFF` is cleared by `Startup_InitializeMemory`
at boot. The branch offsets and each code's address and value were checked
by script; the byte at each address was read from the ROM as the orig
byte, and every code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- The hookshot also takes this path (it shares `.boomerang_ancilla`), so
  it pulls any sprite back too and no longer stuns enemies. A boomerang-
  only check did not fit in 100 codes. Arrghus's puffs still use their own
  hookshot code.
- Removed sprites don't set the dungeon death flag, so they come back
  when the room loads again. Removing a boss or a story NPC (for example
  Link's uncle) can block progress until you leave and come back. A boss
  removed this way likely drops no heart container or prize.
- Enemies made of several sprites (Lanmola, Trinexx and others) may look
  broken while one part is carried.
- A carried sprite still runs its own code each frame. It may move a few
  pixels from the boomerang, fall into a pit it is carried over, or be
  hit by Link's sword. If it leaves the active state (falls, is frozen),
  the drag stops and it keeps the harmless bit (`$0E40` bit 7).
  Some sprites write `$0E40` themselves and can hurt Link again.
- Its height (`$0F70`) is not set to 0 (to save codes), so a sprite in
  the air is drawn above the boomerang.
- No pickup sound plays when a sprite is removed (also to save codes).
- If the boomerang's ancilla slot is reused by a new ancilla in the same
  frame it ends, the sprite follows the new one.
- Nothing else in the game uses `$0224` or the bank `$07` block, but other
  ROM hacks may.
- 97 codes is far more than a real Game Genie holds. All 97 are needed;
  the bank `$07` routine crashes the game if a hook is on without it.

## Sprites move up and down with their sideways speed

**Codes:** `9D37-072C` `9D30-072C` `4A37-0F2C` `9DED-6D91` `9DEF-6D91`
`4AEE-0491` `9DE7-A958` `9DE0-A958` `4AE4-A558` `9D39-0581` `9D31-0581`
`4A39-0171` (use all 12)
**Raw form:** `06E93F:50` (orig `$40`); `06E94F:50` (orig `$40`);
`06E937:2C` (orig `$20`); `05FA01:50` (orig `$40`); `05FA11:50` (orig
`$40`); `05F9F9:2C` (orig `$20`); `1EFF35:50` (orig `$40`); `1EFF45:50`
(orig `$40`); `1EFF2D:2C` (orig `$20`); `1DE95E:50` (orig `$40`);
`1DE96E:50` (orig `$40`); `1DE958:2C` (orig `$20`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-26 13:00:21

Tags: `interesting`

**Mechanism:** Each sprite bank has its own copy of the move routines:
`Sprite_MoveVert` / `Sprite_MoveHoriz` (`Bank06.asm`), `Sprite2_`
(`Bank05.asm`), `Sprite3_` (`Bank1E.asm`) and `Sprite4_` (`Bank1D.asm`).
The sprite tables are 16 bytes each (`$0D00` Y low, `$0D10` X low,
`$0D20` Y high, `$0D30` X high, `$0D40` Y speed, `$0D50` X speed,
`$0D60` Y subpixel, `$0D70` X subpixel). `MoveVert` adds the Y speed to
the Y position, and `MoveHoriz` reuses it on the X tables by adding `$10`
to X:

```
Sprite_MoveHoriz:
    TXA : CLC : ADC.b #$10 : TAX
    JSR.w Sprite_MoveVert

Sprite_MoveVert:
    LDA.w $0D40, X : BEQ .return
        ASL #4 : CLC : ADC.w $0D60, X : STA.w $0D60, X
        LDA.w $0D40, X : PHP
        ...
              ADC.w $0D00, X : STA.w $0D00, X
        TYA : ADC.w $0D20, X : STA.w $0D20, X
```

Since one routine moves both axes, the speed and position tables are
always `$40` apart, so a clean swap of the axes is not possible with byte
changes. In each bank, this cheat changes both `LDA.w $0D40, X` reads in
`MoveVert` to `$0D50`, so the up/down move uses the X speed. It also
changes the `JSR` (`$20`) in `MoveHoriz` to `BIT` absolute (`$2C`), so
the sideways move is skipped. (Kept on, it would read the Y subpixel
`$0D60` as the X speed and make sprites jitter sideways.) A sprite that
would walk right now walks down, and one that would walk left walks up.
Sprites that would move only up or down stand still.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has the `MoveVert` bytes `BD 40 0D F0 .. 0A 0A 0A 0A 18
7D 60 0D 9D 60 0D BD 40 0D 08 4A 4A 4A 4A A0 00 28 10 03 09 F0 88 7D 00
0D 9D 00 0D 98 7D 20 0D 9D 20 0D 60` at exactly four places: `$06:E93E`,
`$05:FA00`, `$1E:FF34` and `$1D:E95D`. The `MoveHoriz` copies at
`$06:E932`, `$05:F9F4`, `$1E:FF28` and `$1D:E952` each `JSR` to their
bank's `MoveVert`. A script found each address from these patterns and
read back every orig byte in the raw form. The `BIT` reads a ROM byte in
the sprite bank and only sets flags. All codes decode back to their raw
forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Use all 12. A bank left out moves its sprites normally.
- Tile collision still checks the direction of the real speeds, so
  sprites may walk through walls, stick to them, or leave the screen and
  despawn.
- Sprite code that sets positions directly, without these routines, is
  not changed. Link, ancillae and garnish have their own movement and are
  not changed. Thrown pots and bushes are sprites and are changed.
- Don't use with "Sprites move sideways with their up-and-down speed":
  together, sprites move sideways with their own X speed and never move
  up or down.
- The slot cheats ("Every enemy moves the sprite in the next slot
  sideways instead of itself" and the previous-slot version) change the
  `$10` in `MoveHoriz`. With this cheat, that move is skipped, so they do
  nothing.

## Sprites move sideways with their up-and-down speed

**Codes:** `FD39-072C` `FD31-0DBC` `7D31-0FBC` `7D31-049C` `4A37-0F2C`
`FDE4-6D91` `FDE4-6FF1` `7DE4-64F1` `7DE4-6421` `4AEE-0491` `FDE9-A958`
`FDE9-A178` `7DE9-A578` `7DE9-A5E8` `4AE4-A558` `FD35-0581` `FD36-0051`
`7D36-0951` `7D36-0171` `4A39-0171` (use all 20)
**Raw form:** `06E95F:10` (orig `$00`); `06E962:10` (orig `$00`);
`06E966:30` (orig `$20`); `06E969:30` (orig `$20`); `06E937:2C` (orig
`$20`); `05FA21:10` (orig `$00`); `05FA24:10` (orig `$00`); `05FA28:30`
(orig `$20`); `05FA2B:30` (orig `$20`); `05F9F9:2C` (orig `$20`);
`1EFF55:10` (orig `$00`); `1EFF58:10` (orig `$00`); `1EFF5C:30` (orig
`$20`); `1EFF5F:30` (orig `$20`); `1EFF2D:2C` (orig `$20`); `1DE97E:10`
(orig `$00`); `1DE981:10` (orig `$00`); `1DE985:30` (orig `$20`);
`1DE988:30` (orig `$20`); `1DE958:2C` (orig `$20`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-26 13:00:21

Tags: `interesting`

**Mechanism:** This is the other direction of "Sprites move up and down
with their sideways speed"; see that entry for the move routines and the
sprite tables. In each bank's `MoveVert`, this cheat changes the position
it adds to: `ADC.w $0D00, X : STA.w $0D00, X` becomes `$0D10` (X low) and
`ADC.w $0D20, X : STA.w $0D20, X` becomes `$0D30` (X high). The Y speed
now moves the sprite sideways. The Y subpixel `$0D60` is kept as the
fraction, which is harmless since nothing else moves Y. As in the other
entry, the `JSR` in `MoveHoriz` becomes `BIT`, since the sideways pass
would now write the Y high byte and the Y speed. A sprite that would walk
down now walks right, and one that would walk up walks left. Sprites
that would move only sideways stand still.

**Evidence:** Same patterns and addresses as "Sprites move up and down
with their sideways speed". In each `MoveVert`, the four operand low
bytes are at `+$21`, `+$24` (`$00`) and `+$28`, `+$2B` (`$20`). A script
found each address and read back every orig byte in the raw form. All
codes decode back to their raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Use all 20. A bank left out moves its sprites normally.
- Tile collision still checks the direction of the real speeds, so
  sprites may walk through walls, stick to them, or leave the screen and
  despawn.
- Sprite code that sets positions directly, without these routines, is
  not changed. Link, ancillae and garnish are not changed.
- Don't use with "Sprites move up and down with their sideways speed"
  (see there).
- The slot cheats ("Every enemy moves the sprite in the next slot
  sideways instead of itself" and the previous-slot version) do nothing
  with this cheat.

## Select lets out a good bee

**Codes:** `E0C5-0D6B` `44C5-0FAB` `AEC5-04DB` `2AC5-040B` `F3C5-046B`
`6DC5-04AB` `FCC5-07DB` (overworld) `E06F-07DC` `4464-0D0C` `AE64-0D6C`
`2A64-0DAC` `F364-0FDC` `6D64-0F0C` (dungeons and houses) `DD2D-0558`
(good bee) (use all 14)
**Raw form:** `02A572:F4` (orig `$F0`); `02A577:22` (orig `$AD`);
`02A578:CF` (orig `$E8`); `02A579:DC` (orig `$1C`); `02A57A:1E` (orig
`$8D`); `02A57B:80` (orig `$F4`); `02A57C:1A` (orig `$1C`); `02891C:F4`
(orig `$F0`); `028921:22` (orig `$AF`); `028922:CF` (orig `$C5`);
`028923:DC` (orig `$F3`); `028924:1E` (orig `$7E`); `028925:80` (orig
`$F0`); `1EDD0D:00` (orig `$05`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-26 13:04:38

Tags: `interesting`

**Mechanism:** This is a one-bee-per-press form of "Good bees keep
coming out and attack enemies". Select opens the save menu in two
places in `Bank02.asm`. On the overworld, `Overworld_PlayerControl`:

```
LDA.b $F0 : AND.b #$20 : BEQ .selectButtonNotDown
    LDA.w $1CE8 : STA.w $1CF4
```

In dungeons and houses, `Dungeon_Normal`:

```
LDA.b $F0 : AND.b #$20 : BEQ .ignoreInput
    LDA.l $7EF3C5 : BEQ .ignoreInput
```

`$F0` is the buttons held, so it is true every frame Select is down.
The first code in each group changes it to `$F4`, the buttons newly
pressed this frame, so it is true once per press. On the overworld, the
other six codes change `LDA.w $1CE8 : STA.w $1CF4` (`AD E8 1C 8D F4
1C`) to `JSL PlayerItem_ReleaseBee : BRA .selectButtonNotDown` (`22 CF
DC 1E 80 1A`). In dungeons, the other five change `LDA.l $7EF3C5 : BEQ
.ignoreInput` (`AF C5 F3 7E F0 20`) to `JSL PlayerItem_ReleaseBee : BRA
.ignoreInput` (`22 CF DC 1E 80 20`). `PlayerItem_ReleaseBee`
(`$1E:DCCF`) is what the bottle calls to let a bee out; see "The Lamp
lets out a good bee". The last code makes the bee a good bee, which
attacks enemies.

**Evidence:** `alttp_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A5 F0 29 20 F0 20 AD E8 1C 8D F4 1C` once in the
ROM, at `$02:A571` (PC `$012571`); the `BEQ` goes to `$02:A597`, so the
new `BRA` at `$02:A57B` uses offset `$1A`. It has `A5 F0 29 20 F0 26 AF
C5 F3 7E F0 20` once, at `$02:891B` (PC `$01091B`); the `BEQ` at
`$02:8925` goes to `$02:8947` (`JSL.l Player_Main`), and the `BRA` keeps
its offset `$20`. `WRAM.asm` gives `$F4` as the rising-edge copy of
`$F0` (`BYsS udlr`). Both sites run with 8-bit A and index registers, as
the bottle's call does. A script read back the orig byte at each code's
address. The codes decode back to their raw forms with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; others not checked.

**Caveats:**

- Not tested.
- Select no longer opens the save menu. Save and quit is not possible
  while these codes are on.
- The overworld and dungeon groups work on their own; use one group and
  the last code for only one of them.
- In dungeons, the check that Link has left his bed is gone, so Select
  may let out a bee in Link's house at the start. Not checked.
- With no free sprite slot, no bee comes out. Each press adds one more
  bee, so many presses can fill the free slots.
- The last code also makes every bottle bee a good bee.
- Uses the same bytes as the other Select cheats. Use only one of them.
