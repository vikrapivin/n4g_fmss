# Chapter 4 — The register window as the sequencer sees it


Summary: the address forms the sequencer uses, the segments and
mirrors of its window, the register file with the roles of register numbers 8,
10 and above, parameter RAM, system memory and program fetch, and what the
sequencer sees differently from the CPU.


## 4.1 Window offsets and target widths

| Address form | Meaning | Example |
|---|---|---|
| CPU MMIO address | ARM physical register address | `mem:0x38a00c30` is r6/result |
| Sequencer window offset | offset used by direct `wr`/`st`/`ld`, or a register value used by `pld`/`pst` | `0xc30` names the same register |
| System-memory bus address | code and data address, used by `C04` or by `ldi`/`sti` | a buffer in SDRAM from `0x08000000` |
| Program offset | byte offset within the current program, used by branches, `jr`, the PC and the fault vector | the value in `C4C` |

Branch, `jr` and vector targets are byte offsets within the current program,
not absolute addresses; `C04` supplies the bus base. Their low 3 bits are
ignored and they do not wrap between 2^15 and 2^24, so the PC has at least
25 bits (M2-105, M2-122, M2-725). STOP sets PC = 0 (M2-53), so a final PC of 0 does not
describe the path a program took.

The primary FMC storage words have these widths (M2-83): `00c`, `010`, `034`
32 bits; `014` 16 bits (`0xffff`); `018` 4 bits; `01c` 1 bit; `02c` 3 bits;
`030` 13 bits (`0x1fff`); `058` 8 bits; `05c` bits 15:8 (`0xff00`). `028` holds
9 bits (M2-173). `C58` stores the bits of `0xd6` (M2-70). The sequencer and the
CPU see the same masks; the further measured masks are `000` `0x77077fff`,
`804` `0x3f04`, `808` `0x1f81`, `814` `0x2f`, `844` and `940..950` 0, `C10`
`0xdf`, `C54` `0x83f` and `C68` 7, with `C3C` and `C48` 32 bits (M2-154,
M2-191).

`pld` and `pst` use offset bits 11:0 only: offsets with any of bits 12..31
set (bits 12, 13, 16 or 20; each of bits 17..30; `0x1d64`, `0x2d64`, `0x8d64`,
`0xfd64`, `0x10d64`, `0x80000d64`) and the aliases `0x1c18`, `0x1c1c`,
`0x1c28`, `0x10c28`, `0x1d68`, `0x10d70`, `0xfd6c` reach the word at offset &
`0xfff` and change no other word. Through `0xc18..0xc34` `pld` reads r0..r7
(through its pointer register r1 it reads the pointer itself) and `pst` writes
r0..r7. Unaligned `pld`/`pst` offsets (low bits 1..3) read 0 and drop the
write (M2-08, M2-175).

## 4.2 Segments and mirrors, sequencer view

The segments are those of [chapter 3, section 3.2](03-cpu-memory-map.md#32-memory-map-the-register-window):
NAND controller `0x000..`, ECC engine `0x800..`, control block `0xc00..`,
parameter RAM `0xd00..0xd7c`. Measured differences from the M1 layout:

- **Offset bit 9:** not decoded below `0x400` and in `0x800..0xbff`
  (`0x200..0x3fc` and `0xa00..0xbfc` are copies), decoded in `0xc00..0xfff`:
  none of the 38 holding words of `0xc00..0xdff` (`C10`, `C3C`, `C48`, `C4C`,
  `C54`, `C58`, `0xd00..0xd7c`) appears at +`0x200` (M2-155).
- **`0x100..0x1fc`:** a block of unknown function, visible again at `0x300`
  only (each of `0x1bc..0x1d8` copied to +`0x200`, none to +`0x400` or
  +`0x600`; M2-155). Its sequencer-writable words are `0x1b8` (1 bit),
  `0x1bc..0x1c8`, `0x1d4`, `0x1d8` (32 bits) and `0x1cc`, `0x1d0` (24 bits);
  the other words keep nothing and read `0x12` from the sequencer (M2-160).
  `0x3a8..0x3b4` read `0x12` from the sequencer and 0 from the CPU (M2-210).
  Details in [chapter 9, section 9.12](09-nand-controller.md#912-the-0x100-block).
- **`0x400..0x7fc`:** reads 0 (M2-155). Its storage words read 0 from the
  sequencer after sequencer or CPU writes and do not alias the primary words
  (M2-82); no `FMC_CTRL0` bit makes them hold values (M2-139); a READ ID and
  DMAs programmed through `+0x400` do nothing (M2-183, M2-185). A sequencer
  `ld` of `0x40c` costs the same 2 cycles as one of `0x00c` or of the unused
  `0x60c`, so access time does not distinguish a decoded slot from undecoded
  space (M2-596). There is no second register set.
- **`0x900..0x9fc`:** reads 0 after encodes and decodes; `0x940..0x950` drop
  writes (M2-171, M2-191).
- **`0xd80..0xdfc`:** reads 0; CPU writes and program `wr`/`pst` writes do not
  persist, with no alias into `0xd00..0xd7c` (M2-04, M2-147).
- M1's second-channel decode does not apply; M1 results are comparison
  hypotheses only.

## 4.3 Register file r0..r7, PC and extended register roles

The sequencer has eight 32-bit GPRs at `0xc18..0xc34`; the CPU may preload
them, and r6 at `0xc30` is the result register, with no separate result
latch in the tested cases (M2-02). Register numbers above 7 follow the M1
roles ([M1 chapter 4.3](../../nano3g/FMSS_M1/04-sequencer-memory-map.md#43-register-file-r0r7-the-pc-and-register-numbers-above-7)),
except that source 10 in an ALU or MOV register form is the destination itself
(M2-43..M2-48, M2-52, M2-65, M2-141). The selector matrices (258 destination,
256 branch, 255 indirect-address, 287 store-source, 256 JR, 256 argument-source
and 64 source-10 cases) give the M1 roles except selector 10 (M2-43, M2-44,
M2-46, M2-47, M2-48, M2-52).

| Role | 8 | 10 | 9 and 11..255 |
|---|---|---|---|
| MOVI/MOV/ADD destination | PC: a local write jumps | all eight GPRs receive the value; with source 10 each GPR is its own source (M2-365) | GPRs unchanged |
| LD/PLD/LDI and immediate ALU destination | `ld`/`pld`: discarded, no jump (M2-616) | the loaded or computed value goes to r0..r5 and r7; the PC is not written (M2-616) | not measured |
| MOV destination with source 8 or 10 | — | — | 9 and 11..15: no GPR written, latch unchanged (M2-678, M2-679) |
| MOV / immediate ALU argument source | current instruction offset | destination's own value | r0 |
| Register ALU argument source | current instruction offset | destination's own value, for all six ALUs | r0 |
| ST register-field source | register-bus latch | register-bus latch | register-bus latch |
| BZ/BNZ tested register | never branches | never branches | never branches |
| JR target source | self-loop | r0 | r0 |
| LDI address source | at PC = `0x60` reads `e1a0f00e` | r0 | r0 |
| PLD address source | at PC = `0x60` reads 0 | r0 | r0 |

**Source 10.** With destination r1, r0 = `11001111` and r1 = `22002222`,
register-form AND and OR give `22002222`, ADD gives `44004444` and SUB gives 0
(M2-44). Across all eight GPR destinations and eight operand pairs, all six
ALUs use the destination itself; with r0 = 1 and r1 = 3, source-10 SHL gives
24 where source-0 SHL gives 6 (M2-52). Where two candidate sources yield the
same value, a matching result does not distinguish them.

**Register-bus latch.** For every extended `st` source selector after a
seeded r7, and for 13 predecessor contrasts at selectors 9, 14 and 200, the
stored value is the previous register-bus value: MOVI, MOV, ADD, LD, PLD and
LDI provide their write value, ST its read value, WR and NOP r0; WAIT11
preserves the previous value (M2-47). A MOV with the broadcast destination r10
does not update the latch: after `mov r10, r3` an extended `st` stores the
value r0 held before the broadcast (M2-366). A `movi` or `mov` to a discarded
extended destination (r12) does not update it either (M2-622), nor does a `mov` to r9 or
r11..r15 with source selector 8 or 10 (M2-679). No constant-zero extended source
is inferred.

**Source field width.** A register-form source selector with any of bits 8..15
set uses neither its low byte nor the r0 fallback of selectors 9 and 11..255
(M2-364, M2-614): for `mov`, `add`, `or` and `sub` it reads the value
that the last source selector before it read (r0 after `movi`, the source
register after `mov` or `add`, the instruction offset after `mov` with
selector 8), kept over a `nop2` and over an `ld`, `st` or `wr` (none of them
sets it: neither the loaded word, the stored register nor the immediate, M2-674..M2-677),
and 0 when only `ld`, `st` or `wr` came before it since the reset, as also in
`and`, `shl` and `shr` (M2-614, M2-618). This value is not the extended-`st` latch
(M2-622). An address selector with bits 8..15 set uses r0, the fallback of the
undecoded address selectors, for `ldi`, and `pld`/`pst` behave as an access
through r0 (M2-615); a `jr` target selector with bits 8..15 set also takes r0
(M2-673). For `sti` a high-bit address selector uses the value the last
source selector read: r0 after `movi`, r5 after `mov r2` with selector 5
(M2-680, M2-681).
With destination r10, source 8 writes the instruction offset to every GPR
(M2-365).

**PC.** A MOVI to r8 jumps locally; MOV r2, r8 returns the instruction offset
(M2-38, M2-43). A `wr` or `st` to the window word `0xc38` also jumps to the
written offset, and an `ld` of `0xc38` reads the instruction's own offset
(M2-162). `jr` with target source 8 loops on itself (M2-46). The low bus
word read by LDI at PC = `0x60` is not proof of a full ROM mirror (M2-48).

**Low bus.** `ldi` from `0x20000000` + off and from off return the same word
for each of off = 0, 4, `0x60`, `0x100`, `0x1000`, `0x8000`, `0xfffc` (M2-432).
Over `0x20000000..0x2000ffff` the word depends only on the 16-KiB quarter
of the address, never the word at the addressed offset: `0xe1a0f00e`
(`mov pc, lr`, a word at 18 ROM offsets in `0..0x3fff`) for `0x0000..0x3fff`,
`0x1affffef` (the ROM word at `0x4d9c`, a `bne` loop) for `0x4000..0x7fff`,
`0xe59f42b0` (the ROM word at `0x9b5c`) for `0x8000..0xbfff` and 0 for
`0xc000..0xffff`, at every sampled offset (M2-682, M2-683). Inferred: each
quarter returns a word that the ROM's own data path last held, plausibly
the last word the CPU fetched from that quarter before leaving the ROM; the
ROM contents themselves are not readable by the sequencer at these
addresses (M2-683).

**Reset.** STOP and abort preserve the GPRs (M2-53, M2-422); the clearing step
`C00` = `0x80000`, alone or with STOP or abort, zeroes them (M2-54, M2-422).
At power-on they read 0 (M2-430).

An `ldi` from an unmapped bus address (0x50000000) hangs the whole system;
a DMA from that address completes (M2-717, M2-718).

**Open:** extended `pst` register roles other than those above and fetch from
an invalid target ([chapter 17](17-open-questions.md#171-instruction-set-and-sequencer-behaviour), items 2 and 7).

## 4.4 Parameter RAM

- **Size and place:** 32 words (128 bytes) at `0xd00..0xd7c`, written and read
  by the CPU and read by the sequencer (M2-03). `pld` and `pst` reach the same
  parameter/GPR window (M2-08, M2-175).
- **Upper window:** `0xd80..0xdfc` reads 0; CPU writes and program `wr`/`pst`
  writes do not persist, read back from both sides, and do not alias into
  `0xd00..0xd7c` (M2-04, M2-147).
- **Reset:** STOP and abort preserve all 32 words; separate buffer, FMC and
  ECC reset steps preserve them too; the clearing step `C00` = `0x80000`
  zeroes them, as does the complete reset (M2-53, M2-54, M2-123, M2-422). They
  read 0 at power-on (M2-430). Parameters written before a reset are therefore
  lost (M2-55).
- **Use:** the meaning of each word is defined by the program
  ([chapter 12](12-cpu-interface.md)).

## 4.5 System memory and the program

- **Program fetch:** 8-byte instructions from the bus address in `C04`
  (M2-01).
- **`ldi`/`sti`:** a 32-bit bus address held in a GPR; words are little-endian
  and the address aligns down to a multiple of 4 (M2-05). A store followed by
  an unaligned reload returns the stored word.
- **Cost:** `sti` to SDRAM 97.8 ns, `ldi` from SDRAM 211 ns; SRAM costs the
  same as SDRAM (M2-102); per copy in a loop `ldi` costs 199.5 ns (M2-378).
- **DMA:** the RAM address aligns down to a multiple of 4 bytes in both
  directions (M2-664); a rewrite of `FMC_DMA_ADDR` during a DMA takes effect at
  a 32-byte boundary (M2-671).
- **Not established:** unaligned instruction fetch. Coherency between the CPU cache and the sequencer or DMA is not
  provided by any measured snooping.

## 4.6 What the sequencer sees differently from the CPU

- **`C08`:** the sequencer reads 5 in every tested control, while the CPU reads
  0, 1, 2, 3, 4, 9, `a` or `b` (M2-70, M2-384).
- **Register-bus latch:** extended `st` source selectors store the previous
  register-bus value instead of a register (M2-47).
- **Extended address sources:** LDI at PC = `0x60` reads `e1a0f00e` and PLD
  reads 0 through extended address selectors 8 (M2-48).
- **`0x400..0x7fc`:** reads 0 (M2-82, M2-155).
- **`0x100` block:** words without storage read `0x12` from the sequencer;
  `0x3a8..0x3b4` read 0 from the CPU (M2-160, M2-210).
- **Indirect window:** `pld`/`pst` reach the window through a register, with
  offset bits 11:0 only; unaligned offsets read 0 and drop writes (M2-08,
  M2-175).
- **Low bus:** `ldi` from `0x0000xxxx` and `0x2000xxxx` returns the same words,
  one fixed boot-ROM word per 16-KiB quarter instead of the addressed word
  (M2-432, M2-682, M2-683).
