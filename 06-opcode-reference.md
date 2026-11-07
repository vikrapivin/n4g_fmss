# Chapter 6 — Opcode reference


Summary: One entry per named opcode, in opcode order, followed by one entry for
the opcode values without an established meaning. The entries use the M1
mnemonics and operand notation and are not numbered. The summary table, the ALU
operand rule, target encoding and instruction cost are in
[chapter 5](05-instruction-encoding.md). Evidence IDs (M2-nn, M1-Fnnn) are
defined in [chapter 18](18-ledger.md).

## 6.1 Conventions and measurement notes

- Fields are those of [chapter 5.1](05-instruction-encoding.md#51-encoding): `R`
  (8 bits), `arg` (16 bits), `imm` (32 bits). `rR` is register R; `rA` the
  register selected by `arg`. `[REG]` is a register-window byte offset; `[rA]`
  uses the address held in rA. `ldi`/`sti` access system memory, `pld`/`pst` the
  register window.
- `nop`, `wait2`, `jr`, `sig`, `pause`, `tmwait`, `evjmp` and `nop2` are
  documentation mnemonics retained from M1, not names recovered from Apple code;
  no original M2 program uses these opcodes. A name does not import M1's ignored
  operands, event routing, timing or control-register side effects.
- **Original** means use in the original M2 programs
  ([chapter 14](14-evidence.md#original-code-observations)); **device** means a
  qualified M2 capture; **inferred**, **open** and **emulated** keep the meanings
  of [chapter 1](01-overview.md). A field is not called ignored because the
  original programs leave it zero, and an opcode is not called illegal because a
  short observation times out.
- Times are device measurements; cycle counts assume the inferred 33.25 MHz
  engine clock (M2-102, M2-103). Program placement in SDRAM or SRAM does not
  change them.
- Register numbers 8..255 decode differently per operand role. The role table is
  in [chapter 4](04-sequencer-memory-map.md); each entry below gives the roles
  that apply to it (M2-43, M2-46, M2-47, M2-48, M2-141).
- **Status** is **confirmed** (original programs and/or device, in all measured
  forms), **tested forms** (device-confirmed only in the listed forms and
  states), **partial** or **open**.

## 6.2 Opcodes

### `00` `end`

- Stop the program.
- Syntax: `end`.
- Encoding: R = arg = imm = 0, as in the original programs. Non-zero R, arg and
  imm are ignored (R 1, 0x80, 0xff; arg up to 0xffff; imm 0xff, 0x80000000,
  0xffffffff): the instruction stops like the zero form, without a fault
  (CS_IRQ = 1), and the following instruction does not run (r6 keeps its
  value). C40 latches R << 16 | arg and C44 the imm (M2-164).
- Operation: the program stops and CS_IRQ bit 0 is set; r6 (`0xc30`) holds the
  program-defined result (M2-02). An END reached after a fault, at the fault
  vector or at the next instruction, also shows the fault bit: CS_IRQ = 5
  (M2-10, M2-11, M2-282). No C58 value diverts an END to the
  fault vector (M2-148, M2-149). An END with C10 non-zero sets VIC1 raw bit 22,
  also an END in a fault handler; with C10 = 0 it stays clear (M2-182).
- Original programs: used by every program.
- Status: confirmed.
- Evidence: M2-01, M2-02, M2-10, M2-11, M2-148, M2-149, M2-164, M2-182, M2-282, [chapter 14](14-evidence.md#physical-capture).

### `01` `wr`

- Write an immediate value to a register-window word.
- Syntax: `wr [REG], #imm`.
- Encoding: arg = window byte offset, imm = value; R = 0.
- Operation: window[arg] = imm. The addressed register supplies any side effects
  (a command byte to the NAND controller, a start code, an acknowledgement).
- Timing: `wr` to PRAM 60.6..61.2 ns, 2 cycles (M2-102, M2-378).
- R = 1, 3, 7, 0x80 and 0xff are ignored (M2-165). A `wr` to `0xc38` (the PC) jumps to the
  written offset without executing the fall-through instruction (M2-162).
- Original programs: used for every fixed register value.
- Status: confirmed.
- Evidence: original programs; device control, PRAM and C58 cases (M2-03, M2-70,
  M2-162, M2-165).

### `02` `st`

- Store a sequencer register to a register-window word.
- Syntax: `st [REG], rR`.
- Encoding: R = source register, arg = window byte offset; imm = 0.
- Operation: window[arg] = rR.
- Register numbers: with R = 8..255, `st` stores the register-bus latch, the last
  value that passed through the register file. `movi`, `mov`, `add`, `ld`, `pld`
  and `ldi` leave the value they wrote; `st` leaves the value it read; `wr` and
  `nop` leave r0; `wait 11` keeps the previous value. Measured for every extended
  selector after a seeded r7, with 13 predecessor contrasts at selectors 9, 14
  and 200; no constant-zero extended source is inferred (M2-41, M2-47). A
  broadcast `mov r10, r3` does not update the latch: the following extended
  `st` stores the value r0 held before the broadcast, not the broadcast value
  (M2-366), and a write to a discarded extended destination (`movi r12`,
  `mov r12`) leaves the latch unchanged (M2-622). This latch differs from the
  value a high-bit source selector reads (M2-618).
- Non-zero imm (1, 4, 0x10, 0x100, 0xff, 0x80000000, 0xffffffff) is ignored
  (M2-165). A `st` to `0xc38` jumps to the
  stored offset (M2-162).
- Original programs: used (control words built in a register, list pointers).
- Timing: 61.2 ns, 2 cycles (M2-378).
- Status: confirmed.
- Evidence: M2-41, M2-47, M2-162, M2-165, M2-366, M2-378, M2-622; device control case storing a CPU-preloaded r3.

### `03` `ldi`

- Load a word from system memory.
- Syntax: `ldi rR, [rA]`.
- Encoding: R = destination, arg = register holding a bus address; imm = 0
  (the tested non-zero imm values are ignored, M2-165).
- Operation: rR = mem32[rA], a little-endian 32-bit load. The tested unaligned
  addresses round down to the word boundary (M2-05). An address register written
  by the preceding instruction is used at distance 0 (M2-108).
- Register numbers: address selector 8 reads the low bus word: at instruction
  offset 0x60 it returns `e1a0f00e`; selectors 9..255 use r0 (M2-48). An `ldi`
  from `0x20000000` + off and one from off return the same word for off = 0, 4,
  0x60, 0x100, 0x1000, 0x8000, 0xfffc (M2-432). Over `0x20000000..0x2000ffff`
  the loaded word depends only on the 16-KiB quarter of the address and is
  never the addressed ROM word: `0xe1a0f00e` (`mov pc, lr`, at 18 ROM offsets
  in the first quarter) for `0x0000..0x3fff`, the ROM word at `0x4d9c`
  (`0x1affffef`) for `0x4000..0x7fff`, the ROM word at `0x9b5c` (`0xe59f42b0`)
  for `0x8000..0xbfff`, 0 for `0xc000..0xffff` (M2-682, M2-683). Inferred: each
  quarter returns the word its ROM data path last held, plausibly the last word
  the CPU fetched from that quarter (M2-683). An address selector with any of bits 8..15 set uses r0,
  not its low byte (M2-615). Destination 10 writes the loaded word to all
  GPRs and not to the PC (M2-616); destination 8 and the other extended
  destinations are open. A load from the unmapped bus address 0x50000000
  hangs the whole system, also when the CPU reads no FMSS register during the
  run; only a power cycle recovers it (M2-718, M2-611).
- Timing: from SDRAM 199.5..211 ns, 6.6..7 cycles (M2-102, M2-378).
- Original programs: used to read per-request lists (chip enables, DMA
  addresses).
- Status: confirmed.
- Evidence: M2-05, M2-48, M2-102, M2-108, M2-165, M2-378, M2-432, M2-611, M2-615, M2-616, M2-682, M2-683.

### `04` `ld`

- Read a window register, keeping the bits selected by a mask.
- Syntax: `ld rR, [REG], mask`.
- Encoding: R = destination, arg = window byte offset, imm = AND mask
  (`0xffffffff` keeps the whole word).
- Operation: rR = window[arg] AND imm. Reading has the side effects of the
  selected register: FIFO reads are not ordinary storage reads
  ([chapter 9](09-nand-controller.md)). Read by the program, `C08` returns 5 in
  the tested states (M2-70, M2-384), and `0xc38` returns the instruction's own offset
  (M2-162).
- Register numbers: destination 10 writes the value to all GPRs and not to
  the PC; destination 8 discards it: no jump, no GPR changes and the next
  instruction runs (M2-616), unlike `movi`, `mov` and `add` into r8 (M2-43).
- Timing: from GPR, PRAM, FMC or ECC registers 60.6..61.2 ns, 2 cycles (M2-102, M2-378).
- Original programs: used for status tests and read-modify-write of control
  words.
- Status: confirmed.
- Evidence: M2-43, M2-70, M2-102, M2-162, M2-378, M2-384, M2-616; device ALU, PRAM and indirect cases.

### `05` `movi`

- Load an immediate.
- Syntax: `movi rR, #imm`.
- Encoding: R = destination, imm = value; arg = 0 (1, 0xa, 0x1234, 0xd70,
  0x8000 and 0xffff are ignored, M2-165).
- Operation: rR = imm.
- Register numbers (all 256 tested, M2-43): R = 8 writes the PC and jumps to the
  local target `imm` (M2-38); R = 10 writes all eight GPRs r0..r7 (M2-39, M2-43);
  R = 9 and 11..255 change no GPR.
- Timing: 30.1 ns, 1 cycle (M2-102).
- Original programs: used (constants, markers, loop counters).
- Status: confirmed. Other state effects of the extended destinations and the
  width of an r8 target beyond the local tests are open.
- Evidence: M2-38, M2-39, M2-43, M2-102, M2-165.

### `06` `mov`

- Copy a register.
- Syntax: `mov rR, rA`.
- Encoding: R = destination, arg = source; imm = 0 (the tested non-zero imm
  values are ignored, M2-165). A source field with any of bits 8..15 set
  (0x103, 0x104, 0x108, 0x10a, 0x1ff, 0x803, 0x8003, 0xff03) reads neither the
  low-byte register nor r0 but the value the last source selector before it
  read: r0 after `movi`, r5 after `mov` from r5, the old r3 after
  `add r3,r3,#1`, the instruction offset after a `mov` with source 8, kept over
  a `nop2` and over an `ld`, `st` or `wr` (none of them sets it), and 0 when
  only `ld`, `st` or `wr` ran before it since the reset (M2-364, M2-614,
  M2-618, M2-674..M2-677).
- Operation: rR = rA. A source written by the preceding instruction is used at
  distance 0 (M2-108).
- Register numbers (all 256 tested in each role): destination 8 is the PC,
  10 writes all eight GPRs, 9 and 11..255 change no GPR (M2-43); source 8 returns
  the current instruction offset (M2-38), source 10 the destination register's
  own value (M2-141), sources 9 and 11..255 r0 (M2-40). With the broadcast
  destination 10, source 8 writes the instruction offset to every GPR and
  source 10 makes each GPR its own source (M2-365); destinations 9 and
  11..15 with source 8 or 10 write no GPR and leave the register-bus latch
  (M2-678, M2-679).
- Timing: 30.1 ns, 1 cycle (M2-378).
- Original programs: used.
- Status: confirmed.
- Evidence: M2-38, M2-40, M2-43, M2-108, M2-141, M2-165, M2-364, M2-365, M2-378, M2-614, M2-618, M2-674..M2-679.

### `07` `wait`

- Block until a hardware event is set.
- Syntax: `wait N`.
- Encoding: R = event number. arg and imm are used by `wait 12` as the status
  operand and mask (original programs use 0x40/0x40, and 0x20 for enhanced
  status); for the other events the originals leave them 0; `wait 11` gives the
  same outcome with (0x40, 0x40) and (0xffff, 0xffffffff) (M2-387), and for the
  other events whether they are ignored is open.
- Operation: the engine waits (C08 = 4) until event N is set. If the timeout is
  enabled and expires, a fault is raised (CS_IRQ bit 2) and execution continues
  at the fault vector or at the next instruction according to C58 bit 2; bits 1,
  4, 6 and 7 change neither path (M2-11, M2-612).
  For a wait entered at the start of a run the timeout is
  492.747 us × (count + 1) + 447.5 us for start words `count << 8 | 0xb5`,
  counts 8..256 (M2-81), 2^14 engine cycles per count (inferred, M2-103). The
  fault falls on instants fixed relative to the start of the run (447.5 +
  492.747 k us for n = 0xb), not a fixed time after the wait entry: with count 0
  a wait entered 4..407 us after the start faults at 446..448 us and one entered
  at 477 or 544 us at 940 us (M2-433). Counted from the wait's entry, a wait
  of count c ≥ 1 faults at the (c + 2)-th instant and a wait of count 0 at the
  first (M2-584); the instants are fixed to an event before the start of the
  run, not to the start itself (M2-588): the per-case reset sets the prescaler
  phase, which then runs during the CPU parameter writes (about 0.08 us earlier
  per extra write), and n = 4..6 fit the same phase within the 2-us resolution
  (M2-631);
  start-word bits 7:4 = n select a step of 2^3 cycles for n = 2, 2^5 for n = 3
  and 2^(n+3) for n = 4..11, saturating at n = 11 (M2-115, M2-216). n = 0
  faults at once (0..4 us for every tested count); n = 1 faults after about
  2 engine cycles per count (2 us for count 0x0f, 4 us for 0x40, 8 us for 0x80,
  14..16 us for 0xff; M2-115).
- Events ([chapter 7](07-events-and-waits.md)):
  - 0, 1, 2: command sent, address sent, DMA done (FMC_STATUS bits 1, 2, 20).
    A wait does not consume its flag: repeated and triple waits pass; an
    acknowledgement blocks the next wait; a new trigger re-arms it (M2-66).
  - 3: NAND data transfer done (original programs).
  - 4: ECC done: ECC_STATUS (`0x810`) bit 1 after an encode, bit 2 after a
    decode, both write-1-to-clear; writing 1 clears only bit 0 (M2-67, M2-97,
    M2-99).
  - 5: passes after a status latch, when the chip reports ready (M2-130,
    M2-193); among events 5..10 and 12..15 only event 5 passed in seven
    prepared states (M2-111). The standard latch (`70h`, `wait 0`,
    acknowledge, latch) passes every time (M2-161), and with a waited and
    acknowledged `70h` every tested ordering passes (M2-130). With an unwaited
    `70h`, a latch straight after it passes, and a latch one instruction after
    it (an acknowledge, or one `movi r5,#0` filler) faults; 20 or 200 filler
    instructions before the latch pass (M2-68, M2-178); the two faulting
    orderings fault straight after a reset and after any of eight different
    preceding programs (M2-390). Measured in `nop2` units, a latch 0 or 3..20
    instructions after the unwaited `70h` passes and one 1 or 2 instructions
    after it faults: it sets FMC_STATUS bit 3 instead of bit 23 (M2-593). After
    a passed latch and only the acknowledge of bit 23, a second latch without
    a new `70h` passes (M2-494). The latch rules are in
    [chapter 7, section 7.5](07-events-and-waits.md#wait-event-5).
  - 11: passes at once in every tested state (idle with C58 = 0 or 4, timeout
    disabled, DMA or encode running, command or status latch pending, every
    flag acknowledged, C10 = 0, `0x044` all ones, the chip busy in a page read,
    a reset or a block erase) and costs 37.7 ns, 1.25 cycles, as `nop` (`08`)
    and a passing `wait 0` (M2-10, M2-69, M2-107, M2-387).
  - 12: passes iff status AND arg AND mask ≠ 0 (M2-134); against a busy chip it
    releases only when the chip signals ready (M2-130, M2-137); it passes
    with FMC_STATUS bit 23 cleared (M2-68).
  - 8: never passes, also after a complete page read with DMA out (M2-206).
  - 15: fails in both orders for all five clear-enable stimuli (M2-69).
  - 16..255: never pass in eight states, with no modulo-16 alias (M2-159).
  - 6..10, 13, 14, 15 time out after every further tested stimulus: unacknowledged
    DMA, data and spare reads, a FIFO READ ID, a metadata write, BCH8 slot-1 and
    BCH16 encodes, `0x81c` = 1, commands on CE1..CE3, a pending `sig`, an
    uncorrectable and an erased-sector decode completion, a command with
    `0x044` all ones, CPU writes of C40, C44, C50, C14, C5C and C3C, and all
    VIC0 or VIC1 software lines (M2-386).
  - No event with arg = mask = 0 follows NAND ready/busy (M2-107). CPU writes to
    C0C, C10, C48, C54 and C60..C6C release none of events 6..10, 12..15
    (M2-143).
- Original programs: `wait 0..4` and `wait 12` synchronize peripheral and status
  work (M2-19).
- Status: confirmed for the listed events; events 6..10, 13, 14 have no
  established source.
- Evidence: M2-10, M2-11, M2-19, M2-66..M2-69, M2-81, M2-97, M2-99, M2-107,
  M2-111, M2-115, M2-130, M2-134, M2-137, M2-143, M2-159, M2-161, M2-178,
  M2-193, M2-206, M2-216, M2-386, M2-387, M2-390, M2-433, M2-494, M2-584,
  M2-588, M2-593.

### `08` `nop`

- No operation.
- Syntax: `nop`.
- Encoding: the zero form R = arg = imm = 0; R = 1..7, arg = 1, 2, 0xd60,
  0x8000, 0xffff and imm = 1, 0xd64, 0x80000000, 0xffffffff continue as the zero
  form (for R = 1, 3, 7 and the arg/imm values the GPRs and the whole window
  equal those of the zero form). Every R = 8..0xff (also with arg 0xffff and imm
  0xffffffff) does not continue: the instruction holds at its own offset until
  the timeout fault at the time of an unmet `wait 3` (4388..4390 us with start
  word `0x07b5`, CS_IRQ 5), also with a pending command, address, DMA, ECC or
  status flag for R = 8..15 (M2-377, M2-625). With the timeout disabled
  (start word 1) R = 8 and 0xff hold with C08 = 4, the value of an unmet
  `wait 3`: the hold is a wait state (M2-685, M2-686).
- Operation: execution continues with the following instruction. As a
  predecessor of an extended `st`, it leaves r0 in the register-bus latch
  (M2-47).
- Timing: 37.65 ns, 1.25 cycles (M2-102, M2-378), the cost of a passing wait.
- C14: a loop of `nop` with start word `0xffb5` leaves C14 = 0xff (start-word
  bits 15:8), as a passing `wait 11` does (M2-625). `08` acts as a wait whose
  condition holds for R < 8 and that no tested stimulus satisfies for R ≥ 8
  (M2-625, M2-686).
- Original programs: not used.
- Status: tested forms: zero form (M2-17), the three M1-comparison forms in
  idle, DMA and ECC states (M2-34, M2-65) and every R = 0..0xff (M2-625). The
  condition waited for with R ≥ 8 is unknown
  ([chapter 17](17-open-questions.md), item 4).
- Evidence: M2-17, M2-34, M2-65, M2-102, M2-377, M2-378, M2-625, M2-685, M2-686.

### `09` `wait2`

- Block until an interrupt request line of the CPU is raised.
- Syntax: `wait2 N`.
- Encoding: R = line number N; arg and imm are ignored: (0, 0), (0x40, 0x40),
  (0, 0x40), (1, 0xd64), (0xd60, 0xffffffff) and (0xffff, 0) give the same
  outcome (M2-187).
- Operation: `wait2 N` continues once interrupt request line N is raised; N
  numbers VIC0 lines as on M1 (M1-F174). Otherwise it waits for the timeout and
  faults like an unmet `wait`. A VIC software interrupt (SOFTINT) is visible in the
  raw VIC status but releases none of 59 lines; inferred: `wait2` samples the
  peripheral request lines before the VIC's software OR (M2-140).
- Lines seen to release it:
  - 5: always (M2-74, M2-140).
  - 8: when the timer interrupt fires (M2-140).
  - 16: DMAC0 activity (M2-140, M2-144).
  - 19: the USB interrupt; passes depending on USB traffic (M2-74, M2-144).
  - 20: the NAND controller interrupt. `wait2 20` follows VIC0 line 20, which
    is raised while a flag routed by FMC `0x044` (bits 0, 1, 2, 11, 12) is
    pending, not the sticky CS_IRQ bit 4; with `0x044` all ones it passes
    4–7 us after a DMA, a command or a status latch, with `0x044` = 0 never
    (M2-142, M2-200, M2-204). As on M1, the controller's sources share line 20
    (M1-F175).
- No N in 0..255 tracks a NAND erase, program or read busy period (M2-167).
  Numbers 32..63 (VIC1) never pass, in every state, with a VIC1 software
  interrupt raised, and for 22, 53, 54, 55 although VIC1 raw bit 22 was set
  during some runs (M2-211). Numbers 64..255 never pass (M2-34, M2-74).
- Open: no ECC operation raises a line, so the ECC engine's line and enable are
  not found (M2-212, M2-217); the device behind line 5 is not identified.
- Original programs: not used.
- Status: partial.
- Evidence: M2-74, M2-140, M2-142, M2-144, M2-167, M2-187, M2-200, M2-204,
  M2-211, M2-212, M2-217.

### `0a` `and`

- Bitwise AND.
- Syntax: `and rR, rA, #imm` / `and rR, rA`.
- Encoding: R = destination, arg = source; non-zero imm gives rR = rA AND imm,
  zero imm gives rR = rR AND rA ([chapter 5.5](05-instruction-encoding.md#55-alu-operand-rule)).
- Register numbers: source 10 is the destination itself in both forms: with
  r0 = `11001111` and r1 = `22002222`, register-form `and r1` through source 10
  gives `22002222` (r1 AND r1), where M1 would use r0 (M2-44, M2-141).
- A register-form source with any of bits 8..15 set reads 0 with no source read before it (M2-364, M2-614); after a source read, the value that the last source selector read is measured for `mov`, `add`, `or` and `sub` (M2-618).
- Timing: 1 cycle (ALU, M2-102).
- Original programs: used.
- Status: confirmed.
- Evidence: M2-06, M2-44, M2-52, M2-65, M2-141, M2-364, M2-614, M2-618.

### `0b` `or`

- Bitwise OR.
- Syntax: `or rR, rA, #imm` / `or rR, rA`.
- Encoding: as `and`.
- Register numbers: register-form source 10 gives r1 OR r1 = `22002222` in the
  same test (M2-44, M2-141).
- A register-form source with any of bits 8..15 set reads the value that the last source selector read (r0 after `movi`; M2-364, M2-614, M2-618).
- Timing: 1 cycle (ALU, M2-102).
- Original programs: used.
- Status: confirmed.
- Evidence: M2-44, M2-52, M2-65, M2-141, M2-364, M2-614, M2-618.

### `0c` `add`

- Add modulo 2^32.
- Syntax: `add rR, rA, #imm` / `add rR, rA`.
- Encoding: as `and`; non-zero imm gives rA + imm, zero imm gives rR + rA.
- Operation: wraps modulo 2^32 (M2-06). Operands written by the preceding
  instruction are used at distance 0 (M2-108).
- Register numbers: register-form source 10 gives r1 + r1 = `44004444` (M2-44,
  M2-141). Destination 8 jumps locally and destination 10 writes all eight GPRs,
  tested with immediate 1 (M2-43).
- A register-form source with any of bits 8..15 set reads the value that the last source selector read (r0 after `movi`; M2-364, M2-614, M2-618).
- Timing: 30.1 ns, 1 cycle (M2-102).
- Original programs: used.
- Status: confirmed.
- Evidence: M2-06, M2-43, M2-44, M2-102, M2-108, M2-141, M2-364, M2-614, M2-618.

### `0d` `sub`

- Subtract modulo 2^32.
- Syntax: `sub rR, rA, #imm` / `sub rR, rA`.
- Encoding: as `and`; non-zero imm gives rA - imm, zero imm gives rR - rA.
- Operation: wraps modulo 2^32, including results below zero (M2-06).
- Register numbers: register-form source 10 gives r1 - r1 = 0 (M2-44, M2-141).
  Destination 10 with an immediate writes the result to all GPRs and not to the
  PC (M2-616).
- A register-form source with any of bits 8..15 set reads the value that the last source selector read (r0 after `movi`; M2-364, M2-614, M2-618).
- Timing: `sub` plus a taken `bnz` 67.6 ns, 2.25 cycles (M2-102).
- Original programs: used (equality tests by subtraction).
- Status: confirmed.
- Evidence: M2-06, M2-44, M2-102, M2-141, M2-364, M2-614, M2-616, M2-618.

### `0e` `bnz`

- Branch if non-zero.
- Syntax: `bnz rR, target`.
- Encoding: R = tested register, imm = target byte offset from the program
  start; arg = 0.
- Operation: if rR ≠ 0, pc = imm; otherwise execution continues. The low three
  target bits are ignored (+1, +2, +4 land on the aligned instruction, +8 on the
  next); targets are not reduced modulo 2^15..2^24 and land exactly up to
  2^24 + 0x100 (PC ≥ 25 bits) (M2-105, M2-725). A condition register written by the preceding
  instruction is used at distance 0 (M2-108).
- Register numbers: tested registers 8..255 never branch, for zero and non-zero
  controls (M2-46).
- Timing: `sub` plus a taken `bnz` 67.6 ns, 2.25 cycles (M2-102).
- Original programs: used (loops, status tests).
- arg values 1, 0xff and 0xffff are ignored; the branch is still taken
  (M2-165).
- Status: confirmed.
- Evidence: M2-46, M2-102, M2-105, M2-108, M2-165.

### `0f` `jr`

- Jump through a register, conditionally.
- Syntax: `jr rA` (condition register in R).
- Encoding: R = condition register, arg = register holding the target; imm 0, 1
  and 0xffffffff give the same result.
- Operation: if r[R] ≠ 0, pc = r[arg]; otherwise execution continues (R = 0..7,
  16 of 16 cases, M2-110). With R = 0 and r0 = 0 it never jumps (M2-109). The
  target is a byte offset from the program start with the low three bits
  ignored and no wrap at 2^15..2^20, as for `bnz` (M2-122). A target outside the
  program fetches from the system memory at that offset, so the outcome depends
  on that memory (M2-144). All GPRs keep their values: no link value is saved
  (M2-34, M2-46).
- Register numbers: target selector 8 (the current instruction offset) loops on
  the instruction itself; selectors 9..255 and every selector with any of
  bits 8..15 set (0x100, 0x101, 0x105, 0x1ff, 0x8005) use r0 (M2-42, M2-46,
  M2-673). Extended R values are open.
- Original programs: not used.
- Status: confirmed for R = 0..7 and local targets.
- Evidence: M2-34, M2-42, M2-46, M2-109, M2-110, M2-122, M2-144, M2-673.

### `10` `sig`

- Signal the CPU and continue.
- Syntax: `sig`.
- Encoding: R = arg = imm = 0; R/arg/imm = 3/1/0xd64 and 7/0xd60/0xffffffff,
  R 1 and 0xff, arg 0xffff and imm 1, 0x80000000 and 0xffffffff give the same
  result: the fields are ignored (M2-34, M2-370).
- Operation: CS_IRQ (C0C) bit 1 is set and execution continues; at the following
  END, CS_IRQ = 3. Writing 2 to C0C clears the bit (write 1 to clear) and the
  next `sig` sets it again. No VIC0 or VIC1 raw line rises beyond the run's
  baseline (M2-370).
- Timing: 30.2 ns, 1 cycle (M2-370).
- Original programs: not used.
- Status: tested forms: the M1 expectation matches in the three forms in idle,
  DMA and ECC states (M2-34, M2-65), and in the forms of M2-370.
- Evidence: M2-34, M2-65, M2-370.

### `11` `sti`

- Store a word to system memory.
- Syntax: `sti [rA], rR`.
- Encoding: R = source register, arg = register holding the bus address;
  imm = 0 (the tested non-zero imm values are ignored, M2-165).
- Operation: mem32[rA] = rR, little-endian; the tested unaligned addresses round
  down to the word boundary (M2-05). A data register written by the preceding
  instruction is used at distance 0 (M2-108).
- Register numbers: an address selector with any of bits 8..15 set (0x105)
  uses the value the last source selector read: r0 straight after `movi`
  instructions, r5 after `mov r2` with selector 5 (M2-680, M2-681), the rule of
  the high-bit register sources of `mov` and the ALU (M2-618).
- Timing: to SDRAM 97.8 ns, 3.25 cycles (M2-102, M2-378).
- Original programs: used.
- Status: confirmed. Extended source selectors are not measured.
- Evidence: M2-05, M2-102, M2-108, M2-165, M2-378, M2-680, M2-681.

### `12` `pause`

- Stop and hand control to the CPU.
- Syntax: `pause`.
- Encoding: 20 forms pause: R = 0..15 with arg = imm = 0, and R = 2 with
  arg = 0xd60 and imm = 0, 0xff00 or 0xffffffff, or arg = 0xd64 and
  imm = 0xffffffff ([chapter 16](16-device-findings.md#execution-chronology-and-pause-12));
  the M1-comparison forms 3/1/0xd64 and 7/0xd60/0xffffffff also pause (M2-34).
  Labels of a masked comparison in these forms do not change the behaviour.
- Operation: the engine stops with the PC on the `pause` instruction, C08 = 0x0a
  and CS_IRQ bit 3 (IRQ 8); C00 reads `0x4ffb4`; r6 is unchanged (M2-14, M2-25).
  The engine stays paused until the CPU acts. The CPU resumes it by writing the
  PC of the following instruction, acknowledging CS_IRQ bit 3 (C0C = 8) and
  writing a start word (C00 = 0x4ffb5, 0xfff5 or 1); the program then runs to
  END. With the CPU writes at 100–101 us, END follows at 102–103 us (M2-25,
  M2-34). The acknowledgement is not required: PC = offset + 8 and a start word
  continue the program with or without it (11 cases), and without the PC write
  it stays paused (12 cases) (M2-188). With two start words written straight
  after the PC write the program continued in three of four runs and stayed
  paused in one (M2-512).
- Original programs: no program uses it; the original CPU handler
  (`bl_NandReadOnly` `mem:0x4d08`) advances PC by 8, acknowledges IRQ 8 and sets
  start bit 0.
- Reset from pause: the STOP poll of an ordinary reset did not return a paused
  engine to idle (M2-15). A paused run is recovered by resume to END (3–4 us),
  and an inherited pause by resume plus abort (M2-26, M2-75). See
  [chapter 8](08-sequencer-control-registers.md#pause-12-and-recovery).
- Status: tested forms.
- Evidence: M2-14, M2-25, M2-26, M2-34, M2-75, M2-188, M2-512.

### `13` `shl`

- Logical shift left.
- Syntax: `shl rR, rA, #imm` / `shl rR, rA`.
- Encoding: as `and`; non-zero imm is the count, zero imm shifts rR by the count
  in rA.
- Operation: logical. Counts below 32 shift normally; every count of 32 or more
  gives 0, in the immediate and the register form: counts 32, 33, 63, 64, 255,
  256 and 0xffffffff on 1, 0x80000000 and 0xffffffff, and counts 32..255,
  2^8..2^31, 0x101, 0x11f, 0x1001f, 0x8000001f and 0xffffffe1 on 0x80000001.
  There is no modulo-32, 64 or 256 and no 8-bit-immediate aliasing of the count
  (M2-07, M2-146).
- Register numbers: register-form source 10 uses the destination as the count:
  with r0 = 1 and r1 = 3, `shl r1` through source 10 gives 24 (3 << 3), through
  source 0 it gives 6 (M2-52, M2-141). All eight GPR destinations and eight value
  pairs match.
- A register-form source with any of bits 8..15 set reads 0 with no source read before it (M2-364, M2-614); after a source read, the value that the last source selector read is measured for `mov`, `add`, `or` and `sub` (M2-618).
- Timing: 1 cycle (ALU, M2-102).
- Original programs: used.
- Status: confirmed.
- Evidence: M2-07, M2-52, M2-141, M2-146, M2-364, M2-614, M2-618.

### `14` `shr`

- Logical shift right.
- Syntax: `shr rR, rA, #imm` / `shr rR, rA`.
- Encoding: as `shl`.
- Operation: logical, including the high sign bit; counts below 32 shift
  normally and every count of 32 or more gives 0, with the counts of `shl`
  (M2-07, M2-146).
- Register numbers: register-form source 10 uses the destination as the count:
  with r0 = 1 and r1 = 3, source 10 gives 0 (3 >> 3), source 0 gives 1 (M2-52,
  M2-141).
- A register-form source with any of bits 8..15 set reads 0 with no source read before it (M2-364, M2-614); after a source read, the value that the last source selector read is measured for `mov`, `add`, `or` and `sub` (M2-618).
- Timing: 1 cycle (ALU, M2-102).
- Original programs: used.
- Status: confirmed.
- Evidence: M2-07, M2-52, M2-141, M2-146, M2-364, M2-614, M2-618.

### `15` `tmwait`

- Wait on the timer.
- Syntax: `tmwait`.
- Encoding: the zero form R = arg = imm = 0; the fields are ignored: 24
  operand forms (M2-27) and R 1, 0xff, arg 1, 0xffff, imm 1..0x10000 give the
  zero-form time (M2-373).
- Operation: the engine waits for start-word count (bits 15:8) timeout
  instants, then continues without a fault: counted from its entry it ends at
  the (count + 1)-th instant, one instant before an unmet `wait` of the same
  count, also when entered after the first instant (M2-584). Entered at the
  start of a run it ends 492.747 us × count + 450 us after the start (within
  3 us; 943, 3898, 31986 and 126103 us for counts 1, 7, 64 and 255), one
  2^14-cycle step less than the `wait` timeout of the same start word (M2-27,
  M2-373). With count 0 it does not end: it takes the timeout fault at the
  first instant, as a count-0 `wait` (M2-584). C58 = 0 gives the same
  END, and program writes of C14 (0 .. 0xffffffff) before it do not change the
  time: 126101..126102 us at count 255 against 126099..126101 us without a
  write, and no change with 0x1000 at counts 0x10 and 1; single runs ended 24 us
  (0x1000, two captures) and 13 us (0x800) early without recurring (M2-373,
  M2-629). In ten further count-255 runs, five with a C14 = 0x800 write and
  five without, one run without a write ended 23 us early and the five with it
  at 126101..126102 us: the isolated early ends are run-to-run spread, not an
  effect of the write (M2-689, M2-690). During the
  wait C14 counts down by one per timeout step from start-word bits 15:8, and a
  completed `tmwait` leaves C14 = 0 (M2-13, M2-27, M2-172, M2-373). The forms
  also end in the idle, DMA and ECC states (M2-34, M2-65). Consecutive
  `tmwait`s each wait a full period; one started with C14 = 0 does not reload
  and end at once (M2-150).
- C48: with C48 = 0xffffffff or 0x55555555 a `tmwait` does not end after its
  period (3879..3900 us at count 7) but faults at the `wait` timeout
  (4388..4390 us) in every run; C48 = 0 and the tested presets of C10, C3C and
  C54 leave it unchanged (M2-179). At count 1, each single bit 1..7 and
  0xaaaaaaaa make it fault at the `wait` timeout (1434..1436 us, C14 0x7f),
  bits 8..31 change nothing (941..943 us, C14 0), and bit 0 ends it one instant
  early (449 us, C14 = 1) (M2-627). At count 7, C48 = v = 1..7 ends it v
  instants early (3406 .. 449 us against 3896 us for C48 = 0, 493 us per unit)
  with C14 = v, and C48 = 8 makes it fault at the `wait` timeout (4390 us)
  (M2-687). A `tmwait` ends at the instant at which its C14 count reaches C48
  bits 7:0; a value above the count is never reached and gives the timeout
  fault (M2-688).
- Original programs: not used.
- Status: tested forms.
- Evidence: M2-13, M2-27, M2-34, M2-65, M2-150, M2-172, M2-179, M2-373, M2-584, M2-627, M2-629, M2-687, M2-688, M2-689, M2-690.

### `16` `evjmp`

- Wait for an event, then jump to the fault vector without a fault.
- Syntax: `evjmp`.
- Encoding: R = arg = imm = 0; the fields are ignored: R/arg/imm = 3/1/0xd64
  and 7/0xd60/0xffffffff (M2-34, M2-65), and R = 1, 2, 4, 0xff, arg = 1, 2,
  0xffff, imm = 0xffffffff or a program label give the zero-form outcome for
  every tested source; the target is always C4C, never imm (M2-391).
- Operation: with a command, address, data, DMA, ECC or status event the engine
  continues at the fault vector with CS_IRQ bit 2 clear (CS_IRQ = 1 at the
  vector's END) (M2-58). It catches an event that arrives during the wait (a
  command flag 100, 1000 or 5000 us into it) or is already pending (DMA), jumps
  to C4C for C58 = 0, 2 and 4 with CS_IRQ bit 2 clear and leaves the flag set
  (FMC_STATUS `0xc3002` / `0x1c3000` at the target). The handler sees the
  source's flag: FMC_STATUS bit 1 and C50 bit 0 for a command, bit 20 and C50
  bit 2 for a DMA, ECC_STATUS `0x13` and C50 bit 4 for an encode, bit 23 and C50
  bit 5 for a status latch; the departure comes 3..25 us after the start
  (M2-391). An acknowledged flag is no event, and a `70h` written through
  `+0x408` during the wait raises none. With no event it times out after the
  full period and continues at the next instruction with C58 = 0, through the
  fault vector with C58 = 4 (CS_IRQ bit 2 set, CS_IRQ = 5) (M2-58, M2-186).
- Timeout disabled (start word 1): `evjmp` without an event stays at C08 = 0x0b,
  with the PC on the `evjmp` and CS_IRQ = 0, and survives STOP (M2-57, M2-62). Abort (C00 = 2) gives C08 = 3; a
  forced reset (C00 = 0x80000) before another STOP gives C08 = 0, after which the
  full reset and the next independent program succeed (M2-62, M2-63). This
  recovery is repeatable (16 of 16, M2-75). Other CPU C00 orders on the blocked
  state: STOP leaves 0x0b (C00 `0x40000`), abort gives 3 (C00 `0x40002`), STOP
  then abort 3, abort then STOP 0x0b; `0x80000` alone or after STOP or abort
  gives C08 = 0, C00 `0x7f60` and PC 0, and STOP after it C08 = 0 with C00 = 0
  (M2-392). See
  [chapter 8](08-sequencer-control-registers.md).
- Original programs: not used.
- Status: tested forms.
- Evidence: M2-57, M2-58, M2-62, M2-63, M2-75, M2-186, M2-391, M2-392.

### `17` `bz`

- Branch if zero.
- Syntax: `bz rR, target`.
- Encoding: R = tested register, imm = target byte offset; arg = 0 (1, 0xff
  and 0xffff are ignored, M2-165).
- Operation: if rR = 0, pc = imm; otherwise execution continues.
- Register numbers: tested registers 8..255 never branch, even when r0 is zero
  (M2-46).
- Original programs: used, alone or paired with `bnz`.
- Status: confirmed. The target-bit rule is measured for `bnz` and `jr`
  (M2-105, M2-122); for `bz` it is not measured separately.
- Evidence: M2-46, M2-65, M2-165.

### `18` `pld`

- Load a window word through a register.
- Syntax: `pld rR, [rA]`.
- Encoding: R = destination, arg = register holding a window byte offset;
  imm = 0 (the tested non-zero imm values are ignored, M2-165).
- Operation: rR = window[rA & 0xfff]: only offset bits 11:0 are used. Offsets
  with any of bits 12..31 set (bits 12, 13, 16, 20, each of 17..30, and
  `0x1d64`, `0x2d64`, `0x8d64`, `0xfd64`, `0x10d64`, `0x80000d64`) and the
  aliases `0x1c18`, `0x1c1c`, `0x1c28`, `0x10c28`, `0x1d68`, `0x10d70`, `0xfd6c`
  reach the word at offset & 0xfff. Through `0xc18..0xc34` it reads r0..r7;
  through the pointer register r1 itself it reads the pointer. An unaligned
  offset (low bits 1..3) reads 0 (M2-08, M2-175).
- Register numbers: address selector 8 at instruction offset 0x60 reads 0;
  selectors 9..255 use r0 (M2-48); with a selector with any of bits 8..15 set
  the access behaves as one through r0 (M2-615). Destination 10 writes the loaded word to all GPRs and not to
  the PC; destination 8 discards it, with no jump (M2-616). The other extended
  destinations are open.
- Original programs: not used (none of the ten M2 microcode programs
  contains it).
- Status: confirmed. These M2 measurements supersede M1's "3G only"
  qualification for this opcode.
- Evidence: M2-08, M2-48, M2-165, M2-175, M2-615, M2-616.

### `19` `pst`

- Store a register to a window word through a register.
- Syntax: `pst [rA], rR`.
- Encoding: R = source register, arg = register holding a window byte offset;
  imm = 0 (the tested non-zero imm values are ignored, M2-165).
- Operation: window[rA & 0xfff] = rR, with the offset rule of `pld`: the store
  reaches the word at offset & 0xfff and changes no other word; through
  `0xc18..0xc34` it writes r0..r7 (M2-175); an unaligned
  offset (low bits 1..3) drops the write (M2-08, M2-175). Stores to
  `0xd80..0xdfc` do not persist (M2-147). With an address selector with any of
  bits 8..15 set the store behaves as one through r0: it changes no observed
  word (M2-615).
- Original programs: not used (none of the ten M2 microcode programs contains it).
- Status: confirmed.
- Evidence: M2-08, M2-147, M2-165, M2-175, M2-615.

### `1a..fe` and other values without an established meaning

- Every opcode byte other than the 19 common instructions runs in three operand
  forms under C58 = 0 and C58 = 4, and seven control opcodes run with command,
  address, DMA, ECC and status stimuli; all execute without cleanup loss
  (M2-64, M2-65). The signatures of the control opcodes are consistent with the
  named set (`08`, `0f`, `10`, `12`, `15`, `16`, `ff`).
- Every value `1a..fe` is an undefined opcode that behaves exactly as an unmet
  `wait`. With the timeout enabled it faults (CS_IRQ bit 2) at the same time as
  an unmet `wait 3` (M2-81) and continues at the next instruction or at the
  fault vector according to C58 bit 2 (CS_IRQ = 5 at the END; M2-11, M2-282). With the timeout
  disabled it stays at C08 = 4 without a fault (M2-56). It changes no GPR,
  scratch, PRAM or window word in the three operand forms (M2-282).
- With a pending command, address, DMA, ECC or status flag every value
  `1a..fe` behaves the same: the next instruction is not reached and the
  timeout fault comes at the time of an unmet `wait 3` (M2-583).
- All 228 values `1b..fe` give exactly the outcome of `1a` in every scanned
  state, operand form and C58 setting; only `ff` (`nop2`) differs. No value in
  `1a..fe` acts as a hidden instruction in the tested forms, and none is
  distinguishable from an unmet `wait`: there is no distinct
  illegal-instruction trap (M2-282). M1 undefined opcodes behave the same (M1-F052), with
  the M2 timeout length (M2-81).
- With 64 random R/arg/imm words `1a` gives the zero-form outcome (an unmet
  wait to the timeout fault, GPRs unchanged) in every case: its operand fields
  have no effect (M2-684).
- Status: tested forms.
- Evidence: M2-11, M2-56, M2-64, M2-65, M2-81, M2-282, M2-583, M2-684.

### `ff` `nop2`

- No operation (second encoding).
- Syntax: `nop2`.
- Encoding: R = arg = imm = 0; R/arg/imm = 3/1/0xd64 and 7/0xd60/0xffffffff give
  the same result. An all-ones (erased) word, op 0xff with R 0xff, arg 0xffff
  and imm 0xffffffff, executes as `nop2`: runs of 1, 8 and 64 such words
  continue to the END, as does one word with a command, address, DMA, ECC
  operation or status latch pending (M2-367).
- Operation: execution continues with the following instruction.
- Timing: 30.1 ns, 1 cycle, the same as `movi` (M2-102, M2-378); an all-ones
  word 30.2 ns (M2-367).
- Original programs: not used.
- Status: tested forms, in idle, DMA and ECC states (M2-34, M2-65), and the
  all-ones word (M2-367).
- Evidence: M2-34, M2-65, M2-102, M2-367, M2-378.

## 6.3 Test coverage

| Matrix | Cases | Content |
|---|---:|---|
| confirmation | 531 | the 19 common instructions: explicit results, all eight MOV destinations, source/destination aliasing, immediate-zero forms, wrap and shift edges, taken and untaken branches, RAM and window effects, idle and pending/cleared waits |
| M1 comparison | 78 | M1 hypotheses for `nop`, `nop2`, `wait2`, `jr`, `sig`, `pause`, `tmwait`, `evjmp` |
| opcode scan | 1,422 | every other opcode byte in three operand forms under C58 = 0 and C58 = 4 |
| control-opcode states | 105 | seven control opcodes with command, address, DMA, ECC and status stimuli, three operand forms each |

All 531 confirmation and 78 M1-comparison cases match; source selector 10
differs from the M1 prediction and follows the M2 rule (M2-65, M2-141). The scan and state matrices record
changed registers, scratch bytes, instruction latches, control state and IRQs,
with separate fall-through and vector markers; they are signatures, not
assigned mnemonics. A later full execution of all families reproduces the earlier outcomes except 79 of
31,974 cases whose difference has an identified cause: USB and DMAC line
activity, a shorter fault window turning TIMEOUT into FAULT, `jr` targets in
SDRAM outside the program, and previously unrun CPU-write cases (M2-144).
Cleared-ECC `wait` controls clear all ECC status flags with `ECC_STATUS = 0x1ff`
(writing 1 clears only bit 0, M2-97). The families and acceptance criteria are
in [chapter 13](13-experiments.md#additional-matrices).
