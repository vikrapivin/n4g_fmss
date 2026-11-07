# Chapter 7 — Events, waits and the timeout

<a id="events-waits-and-timeout-interpretation"></a>

Summary: the hardware flags that `wait` (`07`) waits for, the interrupt request
lines that `wait2` (`09`) waits for, the status latch behind `wait` events 5 and
12, the acknowledge rules, the timeout and its length, and the other event-like
opcodes. Finding IDs (M2-nn) refer to [chapter 18](18-ledger.md); M1-Fnnn are the
M1 findings listed there.

**Two separate number lists.** As on M1, `wait` and `wait2` both take a number N
in their R field, but the numbers name different things. "`wait` event N" is an
entry of the event table in [section 7.2](#wait-events); "`wait2` condition N"
is an interrupt request line ([section 7.7](#wait2-lines)).
`wait2` condition 5 has nothing to do with `wait` event 5.

## 7.1 The `wait` instruction

`wait N` (`07`, R = N) blocks the program until event N is set, then continues
with the next instruction. An event that does not come ends in a timeout
([section 7.9](#timeout-length)) when start-word bit 2 is set; with start
word 1 (bit 2 clear) an unmet `wait 3` stays waiting with `CS_STATUS` (`C08`)
= 4 and no fault (M2-56). The timeout raises `CS_IRQ` bit 2 and continues at
the fault vector `C4C` when `C58` bit 2 is set, or at the next instruction when
it is clear ([chapter 8, section 8.15](08-sequencer-control-registers.md#c58);
M2-11). An illegal opcode faults at the same time as an unmet `wait` (M2-81).

The arg and imm fields are not ignored for every event: `wait 12` uses them as
the operands of its status test ([section 7.6](#wait-event-12)).
For the other events the tested forms use arg = imm = 0.

How the original programs use the events (bootloader `bl_NandReadOnly`
RESET/READID/READ and restore-OS WRITE/ERASE, [chapter 11](11-nand-operations.md);
M2-19): `wait` events 0..4 follow the command, address, DMA, data and ECC
operations, each followed by its acknowledge write, and `wait` event 12 with
arg = mask = 0x40 (also 0x20 for enhanced status) qualifies the NAND status.


### 7.1.1 Idle behaviour

With nothing pending, `wait 11` reaches END in about 4 us and the other fifteen
events 0..15 fault after about 126.6 ms under the tested control state (M2-10).
A passing `wait 11` costs 37.4..37.7 ns (1.25 cycles at 33.25 MHz), as `nop`
(`08`) and as a passing `wait 0` whose flag is pending (M2-387).
`CS_IRQ` then reads 5 once the fault handler reaches END: bit 2 (fault) and
bit 0 (END) are separate flags. A bare `wait 12` with zero operands and no
prepared status transaction faults; this is consistent with the operand rule of
section 7.6.

<a id="wait-events"></a>

## 7.2 Wait events

| event | source (`FMC_STATUS` = FMC `048`) | set when | acknowledge | `C50` bit | details |
|---|---|---|---|---|---|
| 0 | `FMC_STATUS` bit 1 | a command byte has been sent (after the bus cycle) | `FMC_STATUS` = 2 | 0 (from the third instruction after the command write) | M2-66, M2-112, M2-152 |
| 1 | `FMC_STATUS` bit 2 | the address cycles have been sent | `FMC_STATUS` = 4 | 1 | M2-66, M2-112 |
| 2 | `FMC_STATUS` bit 20 | a DMA has finished | `FMC_STATUS` = `0x100000` | 2 | M2-66, M2-78, M2-112 |
| 3 | `FMC_STATUS` bit 3 | a NAND data transfer (data read, parity read) has finished | `FMC_STATUS` = 8 | 3 | M2-112 |
| 4 | `ECC_STATUS` (`0x810`) bit 1 after an encode, bit 2 after a decode | the ECC operation has finished | write 2 (encode) or 4 (decode) to `0x810`; `0x81c` = 1 also re-arms after an encode | 4 | [section 7.4](#wait-event-4), M2-97, M2-99 |
| 5 | `FMC_STATUS` bit 23 | the polled chip is ready after a status latch | `FMC_STATUS` = `0x1800000` clears bit 23 | 5 (set at once) | [section 7.5](#wait-event-5), M2-130, M2-193 |
| 6..10 | none found | never passed; event 8 also not after a complete page read with DMA out or a one-sector read (M2-206) | — | — | [section 7.8](#unreleased-events), M2-386 |
| 11 | none found | passes at once in every state tried | — | — | M2-10, M2-69, M2-107, M2-387 |
| 12 | latched NAND status byte (`04c`) | the chip signals ready and status & arg & mask ≠ 0 | — | — | [section 7.6](#wait-event-12), M2-134, M2-137 |
| 13..15 | none found | never passed | — | — | [section 7.8](#unreleased-events), M2-69, M2-107, M2-111, M2-386 |
| 16..255 | none | never pass in eight states; no modulo-16 alias | — | — | M2-159 |

- Events 0..3 and 5 pair with the same `FMC_STATUS` bits as on M1 (M1-F004);
  event 4 follows `0x810` bit 1 or bit 2, and a write of 1 (bit 0) does not
  clear it.
- `wait` event 11 passes in all five tested stimulus groups, in either order
  relative to an acknowledge, and immediately after a block-erase confirm while
  the chip is busy (M2-69, M2-107). It passes at once (END 3..13 us after the
  start) idle with `C58` = 0 or 4, with start word 1 (timeout disabled),
  during an unacknowledged DMA, during a BCH8 encode, with a command flag or a
  status latch pending, after every flag is acknowledged, with `C10` = 0 and
  with `0x044` all ones, and with the operands (0x40, 0x40) and (0xffff,
  0xffffffff); it also passes on every iteration of loops that run through
  NAND array busy periods (after `00h`/address/`30h`, after `FFh` and during a
  block erase) (M2-387). It is therefore not the chips' ready line, as on M1
  (M1-F057). No tested state makes it wait, so the condition it tests is not
  identified ([chapter 17](17-open-questions.md), item 10).
- With command stimulus `wait 0` passes with `FMC_STATUS` = `0x000c3002`; with
  DMA stimulus `wait 2` passes with `0x001c3000`; with status stimulus `wait 5`
  passes with `0x008c3000`. Idle `wait 0` faults. These are qualified
  state/control pairs.
- An operation and its `wait` complete asynchronously: a write that acknowledges
  an event before the operation completes does not clear it, and all five
  immediate-acknowledge controls (command, address, DMA, ECC, status) reach END
  (M2-33).

`C50` bits 0..5 mirror the pending flags of events 0..5 (command, address, DMA,
data/parity transfer, ECC, status latch): each is set while its flag is pending
and clears with its acknowledgement; bits 11..12 read 1 while the ECC engine is
idle and 0 while it works (M2-112, M2-209;
[chapter 8, section 8.14](08-sequencer-control-registers.md#c50)).
Sequencer reads of `FMC_STATUS` or `FMC_NSTAT` before a `wait` do not consume
event 5 (M2-178).

<a id="consumption-and-acknowledgement"></a>

## 7.3 Consumption and acknowledgement

The flags behind events 0..2 are levels. A `wait` does not consume them, and an
acknowledge write does (M2-66). Measured with `C58` = 4, so that a timeout
vectors instead of continuing; before/after snapshots are stored in scratch
memory:

| stimulus / event | repeated and triple `wait` | acknowledge between two `wait`s | acknowledge, trigger again, `wait` |
|---|---|---|---|
| command / `wait 0` | all pass | second `wait` faults after clearing 048 bit 1 | passes |
| address / `wait 1` | all pass | second `wait` faults after clearing 048 bit 2 | passes |
| DMA / `wait 2` | all pass | second `wait` faults after clearing 048 bit 20 | passes |
| ECC encode / `wait 4` | all pass | passes after `0x810` = 1 (bit 0 only); `C50` completion bit remains | passes |
| status / `wait 12`, arg = mask = 0x40 | all pass | passes after 048 = `0x1800000`, with bit 23 and `C50` bit 5 clear | faults at the second command's `wait 0`, before the second `wait 12` |

- A flag left set lets the next `wait` on it pass at once. A second DMA
  programmed after such a stale `wait 2` starts while the first runs and cuts
  it; with every `wait 2` acknowledged the same program is byte exact (M2-78).
- The acknowledge that clears event 4 is the `0x810` bit of the operation
  (2 after an encode, 4 after a decode); writing 1 clears only bit 0 and leaves
  the event set, so an encode status `0x13` becomes `0x12` and `wait 4` still
  passes (M2-67, M2-97).
- A status latch leaves the controller in status mode: the next command's
  `wait 0` stalls until an `FMCTRL1` write (M2-32, M2-181;
  [section 7.5](#wait-event-5)). This is the fault in the last row.
- `wait 12` is not an alias of `FMC_STATUS` bit 23 (M2-68).
- With `C58` = 0 a timeout continues at the next instruction; a program that
  then clears the timeout IRQ can report END although its `wait` failed. Such
  cases are read from their stored IRQ words and elapsed times.

<a id="wait-event-4"></a>

## 7.4 Wait event 4 (ECC)

| after | `wait 4` follows | re-arms the next `wait 4` | does not re-arm |
|---|---|---|---|
| encode | `0x810` bit 1 (W1C) | write 2, `0x1ff`, or `0x81c` = 1 | write 1 (clears only bit 0) |
| decode | `0x810` bit 2 (W1C) | write 4, `0x1ff` | write 1, 2, `0x40` |

(M2-97, M2-99; `0x81c` = 1 sets `0x810` to `0x10`, M2-96.) The original programs acknowledge with 1 after `wait 4`, which
leaves the completion bit set; an `ECC_RESULT` read after `wait 4` and that
acknowledge does not give the current sector's count or uncorrectable bit
(M2-85; retained completion is the inferred cause). Twenty repeated-encode
programs end with completion status `0x13` throughout, with no fresh completion
established (M2-72). The other `0x810` bits are in [chapter 10](10-ecc-engine.md).

<a id="wait-event-5"></a>

## 7.5 Wait event 5 and the status latch

A status latch is the `70h` command followed by `FMCTRL1` = `0xca`. One latch
is enough: the controller keeps polling the chip's status until the chip is
ready (M2-130, M2-136). Each latch sets `FMC_STATUS` bit 23 exactly when the
chip reports ready (`0xe0`); a busy status (`0x80`) leaves it clear (M2-193).
A latch never sets `FMC_STATUS` bit 24 (0 of 2743 latch records, with `0x044` =
0, 1, 2, `0xffff` or any single bit; M2-163).

- **Latch and `wait 5`.** The standard latch (`70h`, `wait 0`, acknowledge,
  latch) passes `wait 5` every time: 1337 of 1337 in repeated series, 2644 of
  2644 first latches (`wait 5` or `wait 12`, `0x044` = 0, 1, 2 or `0xffff`) and
  410 of 410 in eleven forms (M2-161). A latch written straight after `70h`,
  with no `wait 0`, also passes (41 of 41 in a series); it returns `0x70`, later
  in the series `0xe0`. A latch one instruction after a `70h` whose `wait 0` was
  not executed (an acknowledge or one `movi r5,#0` filler instruction between)
  faults with bit 23 clear (M2-178). These are the two faulting orderings of an
  unwaited `70h`; with 20 or 200 filler instructions before the latch `wait 5`
  passes (9 of 11 orderings pass; M2-68). The two faulting orderings fault in
  18 of 18 runs, straight after the per-case reset and after each of eight
  different preceding programs, with `FMC_STATUS` reading `0xc300a` /
  `0xc3000` (bit 23 clear) before the `wait 5`: the fault depends on the
  ordering, not on an earlier program (M2-390). Measured in `nop2` steps, a
  latch written straight after the `70h` (no instruction between) passes with
  `FMC_STATUS` bits 1, 3 and 23 set and status `0x70`, one written after one
  or two `nop2` (30..60 ns), one `nop`, one `movi` or the acknowledge faults:
  bit 23 never rises in 24 samples of 188 ns and bit 3 is set (`0xc300a`)
  before the `wait 5`; with 3 to 20 `nop2`, or the acknowledge followed by 20
  `nop2`, it passes with status `0xe0` (M2-593). Why that window sets bit 3
  instead of bit 23 is unknown ([chapter 17](17-open-questions.md), item 11).
- **During a block erase,** `wait 5` after one latch passes when the chip
  becomes ready, at the erase end (about 2.27 ms), with `04c` = `0xe0`; while
  the chip is busy `04c` reads `0x80` from about 20 us on (M2-130, M2-136,
  M2-198).
- On an idle chip, with the `70h` waited for (`wait 0`) and acknowledged before
  the latch, `wait 5` passes for a single latch, two latches, the acknowledge
  just before the latch and a `nop` just before the latch; a latch without any
  `70h` command also passes (M2-130). Without a chip enable the latch returns the command
  echo `0x70` (M2-87).
- **Next command after a latch.** After a status latch a new command is
  refused: its `wait 0` times out, `FMC_STATUS` bit 1 stays clear and bit 26 is
  not set, until an `FMCTRL1` write. A status clear, an ECC kick and `FFh` do
  not release it (M2-181). A second `70h` + latch with no `FMCTRL1` write after
  the first latch is refused at its `wait 0` (342 of 342; M2-202). After the
  release `FMCTRL1` = `0x30020` and the acknowledgement `0x1800000`, the next
  `70h` passes its `wait 0` (11 of 11 forms; M2-181). An address operation in
  that position (after the latch and only the bit-23 acknowledge) does not
  complete either: its `wait 1` times out, `FMC_STATUS` bit 2 stays clear and
  bit 26 sets (`0x40c3000`), while a refused command leaves bit 26 clear
  (M2-610).
- **Second latch after the release.** After a latch, its `wait 5` and the
  release, a second `0xca` latch without a new `70h` passes `wait 5` (41 of 41)
  with no operation between and after each of: set-up word, `FMCTRL1` =
  `0x100e0`, a 16-byte read, a DMA, address cycles, `FMCTRL1` = `0x20`, status
  clear, CE toggle and ECC kick (M2-237).
- **Second latch without the release.** After a latch, its `wait 5` and only
  the acknowledge of bit 23, a second `0xca` latch without a new `70h` also
  passes `wait 5` and returns `0xe0`, with no operation between and after each
  of `FMCTRL1` = `0x20`, a status clear, a CE toggle, a BCH8 encode with its
  `wait 4`, the set-up word, `FMCTRL1` = `0x100e0` and the release: a latch
  needs no re-arm (M2-494).
- **Operations after an unreleased latch.** After a latch (`70h` waited,
  `0xca`, `wait 12`, acknowledge `0x1800000`) without the `0x30020` release, a
  DMA in, a DMA out, a 512-byte data read, a 16-byte parity read, an 8-byte
  FIFO read, a BCH8 encode and a second latch complete as after the release;
  only a new command and an address operation are refused (M2-641).
- **Two chips.** A second chip's poll after an unreleased latch is refused at
  `wait 0`. With the release (`0x30020` + acknowledge) between polls, polls of
  CE0 and CE1 during an erase pass in both orders (four polls each, status
  `0xe0`) (M2-198).
- **Latch after `90h`.** A latch issued after a `90h` command sets no bit 23
  (`wait 5` times out) and leaves the first READ ID byte `0xad` in
  `FMC_NSTAT`; the next `70h` after the release is accepted (M2-239).
- Among `wait` events 5..10 and 12..15 in seven prepared states, only `wait 5`
  after a status latch passed (M2-111). No `wait` event 0..15 with
  arg = mask = 0 follows the NAND ready/busy line (M2-107): the ready state is
  observed only through the status latch.
- A status transaction survives the sequencer's END and blocks the reset's FMC
  phase until it is released (M2-29, M2-31; [chapter 8, section 8.22](08-sequencer-control-registers.md#blocked-states)).
- The latch returns the chip's status on `FMC_DATA_SEL` = `0x100` as on 1
  ([chapter 9, section 9.6](09-nand-controller.md#96-status-latch-and-fmc_nstat-0x04c); M2-169).

<a id="wait-event-12"></a>

## 7.6 Wait event 12 (masked status)

`wait 12` with operands arg and mask (the arg and imm fields) passes when the
chip signals ready and **status & arg & mask ≠ 0** (M2-134, 18 pairs against
`0xe0` and the no-CE echo `0x70`). With arg = 0 or mask = 0 it never passes.

| `wait 12` case | outcome |
|---|---|
| one latch during an erase, 0x40/0x40 | passes at the erase end (about 2.27 ms) |
| one latch during an erase, arg 0, mask 0x40 | faults |
| no latch during an erase, 0x40/0x40 | never passes |
| one latch, 0x80/0x80 | passes only at the erase end, although `04c` reads `0x80` from about 20 us; at release `04c` = `0x80`, 048 bit 23 clear |
| one latch, 0x20/0x20 | passes at the erase end; `04c` = `0xe0`, 048 = `0x8c3000` |
| one latch, 0x01/0x01 | faults |
| `wait 5`, then 0x80/0x80 | passes at the erase end |
| no chip enable, latch, 0x40/0x40 | passes on the echo `0x70` |

(M2-130, M2-134, M2-136, M2-137.)

Inference (M2-137): `wait 12` is evaluated when the chip signals ready, and the
operand test is then applied to the status byte; the 0x80/0x80 case was
released by that event before the ready status byte (`0xe0`, bit 23) arrived.
`wait 12` passes after 048 bit 23 is cleared (section 7.3). A mask of 0x40
cannot tell the ready byte `0xe0` from the echo `0x70`, which also has bit 6
set (M2-87).

<a id="wait2-lines"></a>

## 7.7 `wait2` and the interrupt request lines

`wait2 N` (`09`, R = N) waits until interrupt request line N is asserted, or
faults at the timeout. N is a VIC0 line number, as on M1 (M1-F174; M2-140,
M2-142). The arg and imm fields are ignored: (0, 0), (0x40, 0x40), (0, 0x40),
(1, 0xd64), (0xd60, 0xffffffff) and (0xffff, 0) give the same outcome (M2-187).

| `wait2` condition N | line | when it passes |
|---|---|---|
| 5 | VIC0 line 5 | in every state tried; permanently asserted, source unidentified; high before and after restored pulses across gates0..108; no during-pulse samples, so their involvement is not excluded (M2-744) |
| 8 | VIC0 line 8, timer | when the CPU-enabled timer interrupt fires, also while the chip is busy (M2-140, M2-167) |
| 16 | VIC0 line 16, DMAC0 | on DMAC0 activity |
| 19 | VIC0 line 19 | in DMA and ECC states; its outcome depends on USB traffic (M2-74, M2-144) |
| 20 | VIC0 line 20, NAND controller | while a NAND controller flag enabled in FMC `0x044` is pending, 4–7 us after the operation (M2-142, M2-204) |
| 32, 33 | VIC1 lines 0, 1 (GPIO interrupt groups) | when that line is asserted by hardware; each condition follows only its own line (M2-723) |
| 34..63 | VIC1 | never in any tested state; soft interrupts never release (M2-211) |
| 64..255 | — | never (M2-74) |

Clock pulses on initially enabled gates 1 and 2 leave VIC0 raw at `0x20`
and VIC1 raw at zero before, during and after the measured 28-us window.
Gate0 was already disabled. A loss of communication during gate3 supplies
no interrupt sample or gate identity, so the source of line 5 remains
unidentified (M2-729). The corrected clock-pulse harness restores clocks
before reading peripherals; its before/after samples cannot establish the
line's state during the disabled interval (M2-730). With that pulse, initially
enabled gate3 also returns with exact restoration and VIC0 raw `0x20` before
and after. The following worker FREE job stays queued while USB remains live;
this supplies no gate identity or during-window line measurement (M2-742).
The identical gate3 manifest also precedes loss of USB in a session without
the EXECFIRMWARE entry marker; no RUN acceptance survives. This does not
establish a clock identity or make entry history the cause (M2-746, M2-747).

- **Line 20 and `0x044`.** `0x044` bits 0, 1, 2, 11 and 12 route the command,
  address, transfer, DMA and status-latch flags onto VIC0 line 20. The line
  follows a routed flag only while it is pending, and of the 32 single
  `0x044` bits only the operation's own bit raises it (M2-200). With the
  command flag pending, `wait2 20` passes for `0x044` bit 0 alone of the 32
  single bits (address bit 1, transfer 2, DMA 11, status 12). It follows the
  line, not the sticky `CS_IRQ` bit 4, and a write of `0x844` does not
  release it (M2-204). With `0x044` = 0 the line is never raised (M2-142). As
  on M1, the controller's sources share line 20 (M1-F175).
- **No NAND busy line.** No `wait2` N in 0..255 tracks a NAND erase, program
  or read busy period; only the always-asserted line 5 and the CPU-device lines
  8, 16 and 19 pass (M2-167).
- **No ECC line.** After BCH8 encodes and decodes no VIC line rises beyond the
  no-candidate baseline, for every tested ECC enable candidate (`0x800`,
  `0x808`, `0x814`, `0x818`/`0x844` single bits, `0x860..0x9fc`, with `0x044`
  = 0 or all ones), and `wait2 20` and `wait2 43` never pass after an encode
  (M2-212, M2-217). The ECC engine's line and enable are not found.
- **VIC1.** Condition N follows VIC line N also above 31: `wait2 32` passes
  when VIC1 line 0 is asserted by hardware and `wait2 33` for line 1, each
  only for its own line (M2-723); line 20 + 32 = 52 never rose in any set-up
  tried (M2-724). Without a hardware line, conditions 32..63 never pass in any
  state (620 cases) or with a VIC1 software interrupt raised. `wait2` 22, 53, 54 and 55 time out also after
  the sequencer polled VIC1 raw status and when free-running in a 32 ms window
  (28 runs in each of four series), although the CPU sampled VIC1 raw bit 22
  set during 10 to 14 of each series (M2-211). VIC1 raw bit 22 is set at the stop after a run that executed an
  END with `C10` non-zero, and clear after an END with `C10` = 0 and after a
  run that stops without an END
  ([chapter 8, section 8.6](08-sequencer-control-registers.md#86-fmss_c10-0xc10);
  M2-182).
- A VIC SOFTINT raise is visible in the raw VIC status but releases `wait2 N`
  on none of 59 lines, held for 2 ms (M2-140). Inference: `wait2` samples the
  peripheral request lines before the VIC's software OR, not the raw status
  word. This is why sampled raw VIC snapshots (initially line 20 and 0 raised)
  did not track `wait2` releases.
- Of the 1,024 sampled conditions in the four tested contexts (idle, command,
  DMA, ECC), 1,010 fault (M2-74). The open points are in
  [chapter 17](17-open-questions.md), item 13.

<a id="unreleased-events"></a>

## 7.8 Events that nothing releases

`wait` events 6..10 and 13..15 never passed:

- idle, and after a block-erase confirm with the chip busy (arg = mask = 0;
  M2-107);
- in seven prepared states: idle, ECC encode, erased decode, status latch and
  others (M2-111);
- event 15 in both orders for all five clear-enable stimuli (M2-69);
- event 8 after a complete CE0 page read with DMA out, after a one-sector CE0
  read with `C58` = 0 or 4 following `wait 5` or a `wait 12` ready poll, and
  idle (M2-206);
- with a CPU write 1 ms into the `wait` to `C0C`, `C10`, `C48`, `C54`, `C60`,
  `C64`, `C68` or `C6C` (72/72 fault; M2-120, M2-125, M2-143);
- after an unacknowledged DMA out, an unacknowledged 512-byte data read and
  spare read of an open page, a FIFO READ ID (`0x040` = `0x52`), a metadata
  write, a DMA with `0x044` all ones, a BCH8 slot-1 and a BCH16 encode,
  `0x81c` = 1, a `70h` on CE1, CE2 or CE3, a pending `sig` (`CS_IRQ` bit 1),
  the completion of an uncorrectable and of an erased-sector BCH8 decode
  (`0x810` = `0x1d` / `0x55`) and a `70h` routed by a CPU-written `0x044` of
  all ones; with a CPU write 1 ms into the `wait` of `C40` = 1, `C44` = 1,
  `C50` = `0xffffffff`, `C14` = 0, `C5C` = 1 or `C3C` = 0; and with all VIC0
  or all VIC1 software lines raised from 1 to 2.5 ms (every case times out;
  M2-386).

Event numbers 16..255 never pass in eight states (2,640 cases), with no
modulo-16 alias onto events 0..15 (M2-159). `wait 12` likewise never passes without a status latch or with a zero
operand (section 7.6). What, if anything, sets events 6..10 and 13..15 is open
([chapter 17](17-open-questions.md), item 9). Idle non-completion does not
establish that an event does not exist.

<a id="timeout-length"></a>

## 7.9 Timeout length

The timeout counts steps of the engine clock (about 33.25 MHz, inferred from
instruction timing; M2-103). Start-word bits 15:8 are the count, bits 7:4 = n
select the step ([chapter 8, section 8.2](08-sequencer-control-registers.md#start-word)):

- **Step.** One step is 2^3 engine cycles for n = 2, 2^5 for n = 3 and
  2^(n+3) for n = 4..11; n ≥ 11 saturates at 2^14 cycles (M2-115, M2-216).
  n = 0 faults at once (0..4 us for every tested count). n = 1 is not
  immediate: counts `0x0f`, `0x40`, `0x80` and `0xff` fault at 2, 4, 8 and
  14..16 us, 2 engine cycles per count within the 2-us resolution (M2-115). For n = 2..6 the fault time is linear in count + 1
  with steps of 0.237, 0.964, 3.844, 7.697 and 15.395 us (five counts per n;
  M2-216), so 2^(n+3) holds from n = 4 on but not for n = 2 and 3.
- **Count.** With n = 11 (start words `count << 8 | 0xb5`, count + 1 = 8..256,
  including multi-bit counts) an unmet `wait 3` and an illegal opcode entered
  at the start of the run both fault at
  **fault_at = 492.747 us × (count + 1) + 447.5 us**, largest residual
  1.1 us (M2-81). 492.747 us equals 2^14 cycles at 33.25 MHz. Start word
  `0x20b5` faults at 16,708 us. The idle ≈ 126.6 ms of section 7.1.1 equals
  the formula at count `0xff`.

Measured fault times:

| n (bits 7:4) | step (cycles) | count `0x07` | count `0xff` | count `0x0f` |
|---|---|---|---|---|
| 0 | at once | 0 us | `0xff05` ≈ 4 us | |
| 1 | 2 | 0..2 us | `0xff15` ≈ 16 us | `0x0f15` ≈ 2 us |
| 2 | 2^3 | 4 us | `0xff25` ≈ 62 us | |
| 3 | 2^5 | 8 us | `0xff35` ≈ 246 us | |
| 4 | 2^7 | 30 us | `0xff45` ≈ 986 us | `0x0f45` ≈ 62 us |
| 5 | 2^8 | 62 us | `0xff55` ≈ 1,978 us | |
| 6 | 2^9 | 124 us | `0xff65` ≈ 3,942 us | |
| 7 | 2^10 | 262 us | `0xff75` ≈ 7,900 us | `0x0f75` ≈ 510 us |
| 8 | 2^11 | 510 us | `0xff85` ≈ 15,784 us | |
| 9 | 2^12 | 1,062 us | | |
| 10 | 2^13 | 2,172 us | | |
| 11 | 2^14 | 4,390 us | | |
| 12..15 | 2^14 | 4,390 us | `0xffc5` ≈ 126,594 us | |

**Timeout instants.** The timeout fault falls on instants one step apart,
whose phase is set before the run starts, not a fixed time after the `wait`
is entered: 447.5 +
492.747 × k us for n = 0xb (period 2^14 cycles) and about 16.5 + 30.8 × k us
for n = 7 (period 2^10 cycles). With count 0 a `wait 3` entered 4..407 us after
the start faults at 446..448 us and one entered at 477 or 544 us at 940/941 us
(start word `0x00b5`); with `0x0075` a wait entered 3..17 us after the start
faults at 16..18 us and one entered at 23 or 29 us at 46/48 us (M2-433). A
count-0 wait therefore faults at the first instant after its entry. Counted
from the entry, a wait of count c ≥ 1 faults at the (c + 2)-th instant and a
`tmwait` of count c ≥ 1 ends at the (c + 1)-th; entered after the first
instant, a wait or `tmwait` of count 1..3 ends exactly one instant later than
entered at the start, for n = 0xb, and the same counting holds for n = 7 and 9
(M2-81, M2-373, M2-584).

The first instant of a count-0 wait entered at the start is 446..448 us for
n = 0xb..0xf, 202 us for n = 0xa, 78 us for n = 9, 16 us for n = 7 and 8, 4 us
for n = 4 and 0..2 us for n = 5 and 6, the same in repeats (three runs after
the per-case reset: 16/16/18 us for n = 7, 78/78/78 us for n = 9, 448/446/446 us
for n = 0xb, 4/0/0 us for n = 4, 0 us for n = 5 and 6). For n = 7..0xb these
are (2^(n+3) − φ mod 2^(n+3)) engine cycles after the start with one phase
φ ≈ 1488 cycles; the same phase predicts 1.4 us for n = 4..6, within the
2-us resolution of the measured times (M2-588, M2-631). Extra CPU parameter
writes before the start bring the first instant closer to the start by about
0.08 us per write (64 writes: 4..6 us; 90 writes: 8 us for n = 9), and without
the per-case reset the instants fall elsewhere (n = 7: 26/10 us; n = 9:
98/96 us; n = 0xb: 96/100 us): the per-case reset sets the prescaler phase,
and the prescaler runs while the parameters are written (M2-588, M2-631).
No single step of the reset sets it alone: for n = 9 the full reset gives
76/78 us and the skipped reset 24/34 us, and of the steps run alone only
`C00` = `0x80000` comes near the reset value (82/82 us), while `FMCTRL1`
bit 31 (30/32 us) and `0x81c` = 1 (28/32 us) stay near the no-reset value
(M2-691, M2-692). The combination of steps that sets the phase and why a wait
ends one instant after a `tmwait` of the same count are unknown
([chapter 17](17-open-questions.md), item 6). Short times include the
measurement's polling quantization.

Only the engine timeout is a hardware event. A host observation window that
ends earlier reports a TIMEOUT without the engine's fault, and a worker, reset
or transport failure means the instruction under test did not execute; see
[chapter 15](15-evidence-interpretation.md).


## 7.10 Other event-like opcodes

- **`tmwait` (`15`).** A `tmwait` entered at the start of a run ends
  **492.747 us × count + 450 us** after the start (within 3 us; count =
  start-word bits 15:8): 943, 3898, 31,986 and 126,103 us for counts 1, 7, 64
  and 255, one 2^14-cycle step less than the `wait` timeout of the same start
  word (M2-373). Counted from its entry it ends at the (count + 1)-th timeout
  instant, so entered after the first instant it ends one instant later; a
  count-0 `tmwait` does not end but takes the timeout fault at the first
  instant, like a count-0 `wait` (M2-584). All 24 forms of the first survey end after
  126,101–126,103 us with `C14` = 0 and `CS_IRQ` = 1, distinct from the
  ≈ 126.6 ms `wait` fault (M2-27). R, arg and imm are ignored, `C58` = 0 gives
  the same END, and `C14` written before it (0 .. `0xffffffff`, eight
  values) does not change the time (M2-373, M2-629); `C14`
  reads `0x7f` before it, counts down by one per step during it and reads 0
  after it (M2-13, M2-172, M2-373). Consecutive `tmwait`s each wait a full
  period; one started with `C14` = 0 does not reload and end at once (M2-150).
  With `C48` = `0xffffffff` or `0x55555555` a `tmwait` does not end after its
  period (3898 us at count 7) but faults at the `wait` timeout (4388..4390 us),
  in every run; `C48` = 0 leaves it unchanged (M2-179). At count 1 each single
  `C48` bit 1..7 makes it fault, bit 0 ends it one instant early (449 us,
  `C14` = 1) and bits 8..31 change nothing (M2-627). At count 7, `C48` =
  v = 1..7 ends it v instants early (3406, 2914, 2421, 1928, 1435, 942 and
  449 us against 3896 us for `C48` = 0) with `C14` = v, and `C48` = 8 makes it
  fault at the `wait` timeout (M2-687): a `tmwait` ends at the instant at which
  its `C14` count reaches `C48` bits 7:0, and a value above the count gives the
  timeout fault (M2-688;
  [chapter 8, section 8.12](08-sequencer-control-registers.md#812-0xc48-and-0xc54)).
  Of ten count-255 runs, five with a `C14` = `0x800` write, one without the
  write ended 23 us early and the others within 126099..126108 us: isolated
  early ends are run-to-run spread, not an effect of a `C14` write
  (M2-689, M2-690).
- **`nop` (`08`).** With R = 0..7 it continues; the zero form costs 1.25 cycles, the cost of a
  passing `wait`, and a loop of it leaves `C14` holding start-word bits 15:8
  as a passing `wait 11` does; with R = 8..`0xff` it holds until the timeout
  fault at the time of an unmet `wait 3` (4388..4390 us with `0x07b5`), also
  with a pending command, address, DMA, ECC or status flag for R = 8..15
  (M2-625). With the timeout disabled `C08` reads 4 during the hold, the value
  of an unmet `wait 3`: the hold is a wait state (M2-685, M2-686). What it
  waits for with R ≥ 8 is unknown ([chapter 17](17-open-questions.md), item 4).
- **`evjmp` (`16`).** `evjmp` waits for an event and then continues at the
  fault vector `C4C` without a fault: it vectors with `CS_IRQ` = 1 for command,
  address, data, DMA, ECC and status stimuli (M2-58). It catches an event that
  arrives during the wait (a command flag 100, 1000 or 5000 us into it) and one
  already pending (DMA), jumps to `C4C` for `C58` = 0, 2 and 4 and leaves the flag
  set (`FMC_STATUS` `0xc3002` / `0x1c3000` at the target). With no event it
  times out after the full period and then continues at the next instruction
  with `C58` = 0 or through the fault vector with `C58` = 4 (`CS_IRQ` bit 2
  set); a departure leaves `CS_IRQ` bit 2 clear in the handler. An
  acknowledged flag is no event, and a `70h` written through `+0x408` 1 ms into
  the wait raises none (M2-186). R, arg and imm are ignored (R = 1, 2, 4,
  `0xff`; arg = 1, 2, `0xffff`; imm = `0xffffffff` or a program label): for a
  pending command, DMA, BCH8 encode, status latch, command + DMA and DMA +
  encode it departs to `C4C` 3..25 us after the start, never to imm, and the
  handler sees the source's flag: `FMC_STATUS` bit 1 and `C50` bit 0
  (command), `FMC_STATUS` bit 20 and `C50` bit 2 (DMA), `ECC_STATUS` `0x13` and
  `C50` bit 4 (encode), `FMC_STATUS` bit 23 and `C50` bit 5 (status latch)
  (M2-391). With start word 1 (no timeout) it stays at `C08` = `0xb`,
  `CS_IRQ` = 0 and survives STOP (M2-57); the `C00` sequences that release it
  are in [chapter 8, section 8.22](08-sequencer-control-registers.md#blocked-states)
  (M2-62, M2-392).
- **`sig` (`10`) and `nop2` (`ff`)** keep their M1 names. In idle, DMA and
  ECC states `sig` sets `CS_IRQ` bit 1 and continues (`CS_IRQ` = 3 at END) and
  `nop2` continues (M2-34, M2-65). `sig` ignores R, arg and imm; writing 2 to
  `CS_IRQ` clears the bit and the next `sig` sets it again; it raises no VIC0
  or VIC1 raw line beyond the run baseline and costs one cycle (M2-370;
  [chapter 6](06-opcode-reference.md)).
- **`pause` (`12`)** hands control to the CPU ([chapter 8, section 8.20](08-sequencer-control-registers.md#pause-12-and-recovery)).

Raw VIC words, enabled IRQs, sequencer IRQ flags and FMC/ECC status bits are
different domains; a coincident bit in a snapshot does not map one onto
another.

References: [chapter 14](14-evidence.md) (outcomes), [chapter 13](13-experiments.md)
(experiments), [chapter 15](15-evidence-interpretation.md) (interpretation),
[chapter 18](18-ledger.md) (findings).

Related chapters: [chapter 6](06-opcode-reference.md), [chapter 8](08-sequencer-control-registers.md),
[chapter 9](09-nand-controller.md), [chapter 10](10-ecc-engine.md),
[chapter 17](17-open-questions.md).
