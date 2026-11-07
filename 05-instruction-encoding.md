# Chapter 5 — Instruction encoding


Summary: The eight-byte instruction, the meaning of each field per opcode, the
opcode summary table, the ALU operand rule, branch and jump targets, register
write ordering, instruction cost and byte order. The notation follows M1. Details
per opcode are in [chapter 6](06-opcode-reference.md); register numbers above 7
are in [chapter 4](04-sequencer-memory-map.md). Evidence IDs (M2-nn) are defined
in [chapter 18](18-ledger.md).

## 5.1 Encoding

```
word0 = opcode << 24 | R << 16 | arg    (arg: u16)
word1 = imm                            (u32)

byte:   +0       +1        +2   +3      +4       +5        +6         +7
        arg[7:0] arg[15:8] R    opcode  imm[7:0] imm[15:8] imm[23:16] imm[31:24]
```

Both words are little-endian (M2-01). Every instruction is eight bytes and is
fetched from `FMSS_PROG + pc`. A program is an array of instructions; branch and
jump targets are byte offsets from the program start (section 5.6). For example,
`movi r6, #0x600d600d` encodes `00 00 06 05 0d 60 0d 60`, and `wr [0xc58], #4`
encodes `58 0c 00 01 04 00 00 00`, as in the M1 notation.

The fields mean:

- `R` names a register, an event number, an interrupt line number or a tested
  register, depending on the opcode.
- `arg` is a window byte offset (register-access instructions) or a register
  selector (register-operand instructions); `wait 12` uses it as a status operand.
- `imm` is a value, a mask or a branch target.

`rR` means register R, and `rA` the register selected by `arg`. `[REG]` is a
window offset; `[rA]` is an address held in rA, whose address space depends on
the instruction (`ldi`/`sti`: system memory; `pld`/`pst`: register window).

## 5.2 Operand map

What each field means for each opcode. **used**: the field carries the named
operand. **use 0**: the ordinary encoding; other values are not measured on
M2. **tested**: the listed nonzero values were run on a device and gave the same
result as the zero form, unless the cell states a different outcome; other
values are open. M1's
exhaustive ignored-field results (M1 chapter 5.2)
are not M2 measurements.

| op | mnemonic | R | arg | imm |
|---|---|---|---|---|
| `00` | `end` | tested: 1, 0x80, 0xff (latched in `C40`) | tested: non-zero up to 0xffff (latched in `C40`) | tested: 0xff, 0x80000000, 0xffffffff (latched in `C44`) |
| `01` | `wr` | tested: 1, 3, 7, 0x80, 0xff | used: window offset | used: value |
| `02` | `st` | used: source register | used: window offset | tested: 1, 4, 0x10, 0x100, 0xff, 0x80000000, 0xffffffff |
| `03` | `ldi` | used: destination register | used: register holding the bus address (a selector with any of bits 8..15 set uses r0) | tested: 1, 4, 0x10, 0x100, 0xff, 0x80000000, 0xffffffff |
| `04` | `ld` | used: destination register | used: window offset | used: AND mask |
| `05` | `movi` | used: destination register | tested: 1, 0xa, 0x1234, 0xd70, 0x8000, 0xffff | used: value |
| `06` | `mov` | used: destination register | used: source register (a register-form source with any of bits 8..15 set reads the value the last source selector read) | tested: 1, 4, 0x10, 0x100, 0xff, 0x80000000, 0xffffffff |
| `07` | `wait` | used: event number | event-dependent: `wait 12` status operand | event-dependent: `wait 12` mask |
| `08` | `nop` | tested: 0..0xff; R = 0..7 continue, R = 8..0xff hold in the wait state (C08 = 4) until the timeout fault | tested: 1, 2, 0xd60, 0x8000, 0xffff | tested: 1, 0xd64, 0x80000000, 0xffffffff |
| `09` | `wait2` | used: interrupt line number N | tested: 0x40, 1, 0xd60, 0xffff | tested: 0x40, 0xd64, 0xffffffff |
| `0a..0d`, `13`, `14` | ALU | used: destination, also first source in the register form | used: source register (a register-form source with any of bits 8..15 set reads the value the last source selector read) | used: immediate; 0 selects the register form |
| `0e`, `17` | `bnz`, `bz` | used: tested register | tested: 1, 0xff, 0xffff | used: byte target |
| `0f` | `jr` | used: condition register (jump iff r[R] ≠ 0) | used: register holding the target (a selector with any of bits 8..15 set uses r0) | tested: 1, 0xffffffff |
| `10` | `sig` | tested: 1, 3, 7, 0xff | tested: 1, 0xd60, 0xffff | tested: 1, 0xd64, 0x80000000, 0xffffffff |
| `11` | `sti` | used: source register | used: register holding the bus address (a selector with any of bits 8..15 set uses the value the last source selector read) | tested: 1, 4, 0x10, 0x100, 0xff, 0x80000000, 0xffffffff |
| `12` | `pause` | tested: 0..15 | tested: 1, 0xd60, 0xd64 | tested: 0xd64, 0xff00, 0xffffffff |
| `15` | `tmwait` | tested: 1, 0xff (and 24 earlier forms) | tested: 1, 0xffff | tested: 1 .. 0x10000 |
| `16` | `evjmp` | tested: 1, 2, 3, 4, 7, 0xff | tested: 1, 2, 0xd60, 0xffff | tested: 0xd64, 0xffffffff, a program label (never a jump target) |
| `18`, `19` | `pld`, `pst` | used: destination / source register | used: register holding the window offset (bits 11:0; a selector with any of bits 8..15 set acts as an access through r0) | tested: 1, 4, 0x10, 0x100, 0xff, 0x80000000, 0xffffffff |
| `ff` | `nop2` | tested: 3, 7, 0xff | tested: 1, 0xd60, 0xffff | tested: 0xd64, 0xffffffff |

The "tested" values of `08`, `10`, `16` and `ff` include the three forms of the
M1 comparison set (R/arg/imm = 0/0/0, 3/1/0xd64 and 7/0xd60/0xffffffff), all of
which match the M1 expectation (M2-34, M2-65). The further `08` values are those
of M2-377 and M2-625, the `10` values those of M2-370, the `15` values those of M2-27 and M2-373, the
`16` values those of M2-391 and the all-ones `ff` word that of M2-367. The `end`
values are those of M2-164, the `wr`, `st`, `ldi`, `mov`, `movi`, `bnz`, `bz`,
`sti`, `pld` and `pst` values those of M2-165 and the `wait2` pairs those of
M2-187. The `12` forms are those of
[chapter 6](06-opcode-reference.md#12-pause).

Two field rules depart from "ignored": `nop` (`08`) with any R = 8..0xff does
not continue but holds at its own offset in the wait state (C08 = 4) until the
timeout fault, at the time of an unmet `wait 3`, while R = 0..7 continue (M2-377,
M2-625, M2-685, M2-686); and a register-form
source field is decoded with its bits 8..15: with any of them set (0x104, 0x108,
0x1ff alike) the source is neither the low-byte register nor the r0 of sources
9 and 11..255 (M2-40) but the value that the last source selector before it
read: r0 after `movi` (source field 0), the source register after `mov` and
`add`, the instruction offset after a `mov` with selector 8, kept over a
`nop2` and over an `ld`, `st` or `wr`, none of which sets it, and 0 when only
`ld`, `st` or `wr` ran before it since the reset (M2-364, M2-614, M2-618,
M2-674..M2-677). An address selector of `ldi` and a target selector of
`jr` with any of bits 8..15 set use r0, as the undecoded selectors do (M2-615,
M2-673), and `pld`/`pst` with such a selector behave as an access through r0
(the accessed word itself is not observed; M2-615). An `sti` address selector
with any of bits 8..15 set uses the value the last source selector read, r0
after `movi` and r5 after `mov` with selector 5 (M2-680, M2-681). A `mov` to
r9 or r11..r15 discards its write also with source selector 8 or 10 (M2-678,
M2-679). Field widths of R and arg beyond the tested values are not measured.

## 5.3 Opcodes

The M1 mnemonics are the reference names. `nop`, `wait2`, `jr`, `sig`, `pause`,
`tmwait`, `evjmp` and `nop2` are names given in these documents after the
measured behaviour; no original M2 program uses them. A name preserves the M1
comparison; the **status** column states how much of the behaviour is
established on M2: **confirmed** (original programs and/or device), **tested
forms** (device-confirmed in the measured forms and states only), **partial**
(part of the behaviour measured, part open).

| op | mnemonic | semantics | status |
|---|---|---|---|
| `00` | [`end`](06-opcode-reference.md#00-end) | stop, raise CS_IRQ bit 0; r6 is the program-defined result | confirmed |
| `01` | [`wr [REG], #imm`](06-opcode-reference.md#01-wr) | window[arg] = imm, with the register's write side effects | confirmed |
| `02` | [`st [REG], rR`](06-opcode-reference.md#02-st) | window[arg] = rR | confirmed |
| `03` | [`ldi rR, [rA]`](06-opcode-reference.md#03-ldi) | rR = mem32[rA], system-memory bus address | confirmed |
| `04` | [`ld rR, [REG], mask`](06-opcode-reference.md#04-ld) | rR = window[arg] AND imm | confirmed |
| `05` | [`movi rR, #imm`](06-opcode-reference.md#05-movi) | rR = imm | confirmed |
| `06` | [`mov rR, rA`](06-opcode-reference.md#06-mov) | rR = rA | confirmed |
| `07` | [`wait N`](06-opcode-reference.md#07-wait) | block until event N is set, else time out and fault | confirmed for the measured events ([chapter 7](07-events-and-waits.md)) |
| `08` | [`nop`](06-opcode-reference.md#08-nop) | no operation for R = 0..7 (zero form 1.25 cycles, the cost of a passing wait); R = 8..0xff hold as an unmet wait (C08 = 4) to the timeout fault (M2-625, M2-686) | tested forms |
| `09` | [`wait2 N`](06-opcode-reference.md#09-wait2) | block until interrupt request line N is raised (sampled before the VIC software OR, inferred), else time out and fault | partial: lines 5, 8, 16, 19, 20 pass, 32..255 never; line 5 source and ECC line open |
| `0a` | [`and`](06-opcode-reference.md#0a-and) | ALU, section 5.5 | confirmed |
| `0b` | [`or`](06-opcode-reference.md#0b-or) | ALU | confirmed |
| `0c` | [`add`](06-opcode-reference.md#0c-add) | ALU, wraps modulo 2^32 | confirmed |
| `0d` | [`sub`](06-opcode-reference.md#0d-sub) | ALU, wraps modulo 2^32 | confirmed |
| `0e` | [`bnz rR, target`](06-opcode-reference.md#0e-bnz) | if rR ≠ 0: pc = imm | confirmed |
| `0f` | [`jr rA`](06-opcode-reference.md#0f-jr) | if r[R] ≠ 0: pc = rA; no link value saved | confirmed |
| `10` | [`sig`](06-opcode-reference.md#10-sig) | set CS_IRQ bit 1 (W1C) and continue; no interrupt line; 1 cycle | tested forms |
| `11` | [`sti [rA], rR`](06-opcode-reference.md#11-sti) | mem32[rA] = rR, system-memory bus address | confirmed |
| `12` | [`pause`](06-opcode-reference.md#12-pause) | stop on the instruction with C08 = 0x0a and CS_IRQ bit 3; the CPU resumes it | tested forms |
| `13` | [`shl`](06-opcode-reference.md#13-shl) | ALU, logical left; count ≥ 32 gives 0 | confirmed |
| `14` | [`shr`](06-opcode-reference.md#14-shr) | ALU, logical right; count ≥ 32 gives 0 | confirmed |
| `15` | [`tmwait`](06-opcode-reference.md#15-tmwait) | timed wait: ends without a fault at the (count + 1)-th timeout instant after its entry (492.747 us × count + 450 us when entered at the start); count 0 faults; it ends at the instant at which the count reaches `C48` bits 7:0 and faults if they exceed the count (M2-373, M2-584, M2-627, M2-688) | tested forms |
| `16` | [`evjmp`](06-opcode-reference.md#16-evjmp) | wait for an FMC/ECC event, then jump to the fault vector without a fault; idle times out and faults | tested forms |
| `17` | [`bz rR, target`](06-opcode-reference.md#17-bz) | if rR = 0: pc = imm | confirmed |
| `18` | [`pld rR, [rA]`](06-opcode-reference.md#18-pld) | rR = window[rA] | confirmed |
| `19` | [`pst [rA], rR`](06-opcode-reference.md#19-pst) | window[rA] = rR | confirmed |
| `ff` | [`nop2`](06-opcode-reference.md#ff-nop2) | no operation; 1 cycle; an all-ones (erased) word executes as `nop2` | tested forms |
| `1a..fe` | [undefined](06-opcode-reference.md#1afe-and-other-values-without-an-established-meaning) | same outcome as an unmet `wait`: faults at the timeout, no other change; also with a pending flag; the operand fields of `1a` have no effect (M2-282, M2-583, M2-684) | tested forms |

## 5.4 Unassigned opcodes

Every opcode byte other than the 19 common instructions was run in three operand forms under
C58 = 0 and C58 = 4, and seven control opcodes were run with command, address,
DMA, ECC and status stimuli (M2-64, M2-65). The signatures of the control
opcodes are consistent with the named set: `nop` 08, `jr` 0f, `sig` 10, `pause`
12, `tmwait` 15, `evjmp` 16 and `nop2` ff. Every value `1a..fe` gives exactly
the outcome of `1a` and behaves as an unmet `wait`: with the timeout enabled it
faults at the same time as an unmet `wait 3` (M2-81), with the timeout disabled
it stays in C08 = 4 without a fault (M2-56), and it changes no register, scratch
or window word (M2-282). The same holds for every value `1a..fe` with a
pending command, address, DMA, ECC or status flag (M2-583). No value in
`1a..fe` acts as an instruction in the tested forms; there is no distinct
illegal-instruction trap. An erased
(all-ones) word is `ff` with all-ones fields and executes as `nop2`, also in
runs of 8 and 64 words and with a command, DMA, ECC or status operation pending
(M2-367). With 64 random R/arg/imm words `1a` gives the zero-form outcome in
every case: its operand fields have no effect (M2-684).

## 5.5 ALU operand rule

```
imm != 0: rR = r(arg) OP imm
imm == 0: rR = rR     OP r(arg)
```

- There is no distinct immediate-zero form; `mov` performs the copy (M2-06).
- ADD and SUB wrap modulo 2^32; SHL and SHR are logical. Counts below 32 shift
  normally and every count ≥ 32 gives 0, in the immediate and the register form
  (counts up to `0xffffffff`, values 1, `0x80000000`, `0xffffffff` and
  `0x80000001`): there is no modulo-32, 64 or 256 and no 8-bit-immediate
  aliasing of the count (M2-06, M2-07, M2-146).
- Source selector 10 reads the destination register in every MOV and ALU form,
  immediate and register, for all six ALUs and all eight GPR destinations
  (M2-52, M2-141); with the broadcast destination r10 each GPR is its own source
  (`add r10, sel10, #5` adds 5 to every GPR), and source 8 gives every GPR the
  instruction offset (M2-365).
- A register-form source field with any of bits 8..15 set reads the value that the last source selector read, in `mov`, `add`, `or` and `sub` (r0's value after `movi`), and 0 when no source has been read since the reset, also in `and`, `shl` and `shr` (M2-364, M2-614, M2-618). M1 uses the destination only in the MOV and immediate forms
  and r0 in the register form; this is an M2 difference in selector decoding,
  not in the immediate-zero encoding.
- There is no compare instruction; comparisons are synthesized by subtraction
  and a conditional branch. The original programs have no unconditional jump:
  they branch on a register known to be non-zero. `jr` (`0f`) jumps through a
  register, conditional on r[R] (section 5.6).

## 5.6 Branch and jump targets

- `bnz`/`bz` targets (`imm`) and `jr` targets (r[arg]) are byte offsets from the
  program start (M2-105, M2-122).
- The low three bits are ignored: targets +1, +2 and +4 land on the aligned
  instruction, +8 on the next one.
- Targets are not reduced modulo 2^15..2^24: `bnz` and `jr` land exactly on
  targets up to 2^24 + 0x100, so the PC has at least 25 bits (M2-105, M2-725). M1's rule that a `jr` target above 0xfff does not jump
  (M1-F097) does not apply to M2 in these measurements (M2-122).
- A `jr` whose target lies outside the program fetches from the system memory
  at that offset; its outcome depends on that memory's contents (M2-144).
- `jr` jumps iff r[R] ≠ 0 (M2-110). Writing a local target into r8 with `movi`,
  `mov` or `add` also jumps (M2-43).

## 5.7 Register write ordering

There is no register-write hazard. A register written by the immediately
preceding instruction is seen at distance 0 by `add`, `mov`, the `ldi` address,
the `sti` data and the `bnz` condition (32 of 32 cases, M2-108). A `jr` with
R = 0 and r0 = 0 does not jump, whatever the distance from the instruction that
writes its target register (M2-106, M2-109); this is the conditional rule of
section 5.6 (M2-110), not a hazard.

## 5.8 Instruction cost

Per-instruction times are loop differences over 1,000 and 21,000 iterations,
with the empty loop subtracted; costs add linearly per copy (1..8 copies in a
20,000-iteration loop, residual ≤ 1.1 us, and 16 copies in a 30,000-iteration loop, M2-378). The cycle column assumes the inferred engine
clock of 33.25 MHz (M2-103), which also matches the timeout step of 2^14 cycles.
The program placement (SDRAM or internal SRAM) does not change the times
(±0.5 ns, M2-102).

| instruction | ns (device) | cycles at 33.25 MHz |
|---|---:|---:|
| `movi`, `mov`, `add`, `nop2` (`ff`), an all-ones word, `sig`, untaken `bnz`, `bnz` taken to the next instruction | 30.1 | 1.00 |
| `nop` (`08`), a passing `wait 11`, a passing `wait 0` | 37.65 | 1.25 |
| `ld` from GPR / PRAM / FMC / ECC, from a `+0x400` word or an unused offset; `wr`, `st` and `pld` of PRAM or FMSS words | 60.6..61.2 | 2.0 |
| `sti` to SDRAM | 97.8 | 3.25 |
| `ldi` from SDRAM | 199.5..211 | 6.6..7.0 |
| `sub` + taken `bnz` | 67.6 | 2.25 |

Sources: M2-102; `sig` M2-370, the all-ones word M2-367, the branch forms and
linearity M2-378, `wait 11` and `wait 0` M2-387, the `+0x400` and unused-offset `ld` M2-596. The quarter-cycle fractions (`nop` 08,
`wait 11`, branch, `sti`) are measured, not explained
([chapter 17](17-open-questions.md), item 6). A simple counter loop gives a slope of about 33.8 ns per instruction
(M2-09).

## 5.9 Byte order

- Instructions, system-memory words (`ldi`/`sti`) and parameter words are
  little-endian (M2-01, M2-05). The tested unaligned `ldi`/`sti` addresses round
  down to a word boundary (M2-05); branch targets ignore their low three bits
  (section 5.6).
- The BCH8 and BCH16 code streams are taken most significant bit first: data,
  then metadata, then parity (M2-77, M2-86).
- Encode parity is readable at `0x840..0x85c` with the word order reversed and
  big-endian words (M2-116). See [chapter 10](10-ecc-engine.md).
