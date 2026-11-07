# Chapter 17 — Open questions


Summary: every point that the other chapters mark as open, not measured,
unexplained or only inferred, in the form of the M1 book's chapter 17. Each
entry gives what is known so far, the finding IDs of [chapter 18](18-ledger.md)
and the chapter with the details. Nothing here is a new finding, and each open
point is listed once. The comparison with M1, finding by finding, is
[chapter 19](19-m1-m2-differences.md). Points that were open in the first M2 survey and are
now settled are listed in [section 17.8](#178-settled-since-the-first-m2-survey).


## 17.1 Instruction set and sequencer behaviour

| # | open point | known so far | details |
|---|---|---|---|
| 2 | Exact PC width above 2^24; instruction fetch from an invalid or unaligned target | branch and JR targets are byte offsets with the low 3 bits ignored and land exactly up to 2^24 + 0x100, so PC ≥ 25 bits (M2-105, M2-122, M2-725). Not measurable with this harness: a jump beyond 2^20 that does not wrap executes unbounded memory, and a fetch from an unmapped address is inferred to hang the bus, which stops USB before any result reaches the host (M2-611) | [chapter 4](04-sequencer-memory-map.md), [chapter 5](05-instruction-encoding.md) |
| 4 | The condition that `waitc` (`08`) waits for with R = 8..0xff | the hold is a wait state: C08 reads 4 in it, as in an unmet `wait 3`, and it ends only at the timeout fault (M2-625, M2-685, M2-686); R = 0..7 continue at the cost of a passing wait (M2-378); no command, address, DMA, ECC or status flag releases R = 8..15 (M2-625). The condition is unknown because no stimulus the harness can generate releases it, as for the waits of item 9 | [chapter 6](06-opcode-reference.md), [chapter 7](07-events-and-waits.md) |
| 6 | The origin of the 0.25-cycle fractions; the reset steps that together set the prescaler phase; the mechanism that places a wait's fault at the (count + 2)-th and a `tmwait`'s end at the (count + 1)-th instant after the entry | engine clock ≈ 33.25 MHz (inferred, M2-103); costs add linearly per copy: 1.00 cycle for `nop`, `movi`, `mov`, `add`, an untaken and a next-taken `bnz`, 1.25 for `waitc` (`08`), a passing `wait 11` and a passing `wait 0`, 2.03 for `ld`, `st`, `wr` and `pld`, 3.25 for `sti`, 2.25 for `sub` + a backward taken `bnz` (M2-102, M2-378, M2-387); timeout instants are spaced one step apart and fixed to a prescaler phase that the per-case reset sets (M2-433, M2-588, M2-631); of its single steps only C00 = `0x80000` brings the first instant near the reset value, and none reproduces it (M2-691, M2-692); bits 7:4 = n give 2^(n+3) cycles per step for n = 4..11 (M2-115, M2-216). The prescaler and the per-instruction fractions are not readable: the harness sees them only through fault and end instants at 2 us resolution and through loop durations, which show their sums, not their origin | [chapter 5](05-instruction-encoding.md), [chapter 7](07-events-and-waits.md) |
| 7 | Final instruction and stall mechanism of the `regnum` case after which USB communication was lost; whether the reserved-field loss was the `ldi` from 0x11111110 | no output retained (M2-35); an r8 write that sends the PC outside mapped memory reproduces a worker/USB stall only under injected CPU-bus faults (M2-36, emulated); the stall ends communication before any result reaches the host, so the executing instruction cannot be named; a second loss came while the reserved-field forms of item 1 ran: no record reached the host and the device LCD showed partly drawn characters; which case ran is not recorded; run one per probe with valid addresses every form ends normally, and the only form that leaves mapped memory with the register values of the lost run is `ldi` with address selector 0x105, which uses r0 = 0x11111111 (M2-611, M2-615; bus hang inferred) | [chapter 16](16-device-findings.md) |
| 8 | Full bit maps of C00 and C5C; what the C08 values 1, 2, 8 and 9 read while a program runs encode; what the value `0x7f` of C14 is (a constant or a value held elsewhere) | start-word bits 1, 3, 16, 19 and the step field measured, the others without effect (M2-153, M2-157, M2-158, M2-201); C58 stores mask d6, bit 2 selects timeout vectoring, bits 1, 4, 6 and 7 are stored without effect (M2-11, M2-70, M2-148, M2-149, M2-612); C3C = faulting offset + 8; C48 acts on `tmwait` as a compare value (M2-179, M2-627, M2-688); read inside the state, C08 is 4 in every unmet wait, `waitirq`, `tmwait` and `waitc` hold, 0xa in `pause`, 0xb in a blocked `evjmp`, 3 frozen or aborted, 1 held by `C60` (M2-14, M2-56, M2-57, M2-62, M2-70, M2-121, M2-158, M2-384, M2-685); 3 ms into a loop of one instruction kind it reads `nop` 0x9/0x9/0x2, `sti` 0x8/0x1/0x8 and `ld` 0x1/0x1/0x1 in three runs (M2-693); C00 bit 18 is set in every running, waiting, paused or held state; C14 reads `0x7f` after a wait released after one or more steps, after a timeout fault and at power-on (M2-172, M2-177, M2-430). Harness limit: the CPU reads C08 once per run at an instant whose phase within the running loop it does not control, and the value depends on that phase, so the running values cannot be assigned to instructions or pipeline steps; sampled by the CPU while a program runs C08 reads 1, 2 and 9 in ALU and `nop` loops, 7 in an SDRAM-access loop, 5 in a window-access loop and 4 in an unmet wait, changing within one loop (M2-719) | [chapter 8](08-sequencer-control-registers.md) |

## 17.2 Events and the status latch

| # | open point | known so far | details |
|---|---|---|---|
| 9 | Sources of WAIT 6..10 and 13..15 | no tested stimulus releases any of them: not NAND ready/busy (M2-107), not NAND, ECC, DMA, status or metadata states (M2-111), not CPU writes to C0C, C10, C48, C54, C60..C6C (M2-120, M2-125, M2-143), not unacknowledged DMA, data or spare reads, a FIFO READ ID, a metadata write, BCH8 slot-1 and BCH16 encodes, 0x81c = 1, commands on CE1..CE3, a pending `sig`, uncorrectable and erased-sector decode completions, a command routed by a CPU-written 0x044 = all ones, CPU writes of C40, C44, C50, C14, C5C or C3C, or VIC0/VIC1 soft lines (M2-386); WAIT8 not after a page read with DMA out (M2-206); WAIT15 fails after every clear-enable stimulus in both orders (M2-69); event numbers 16..255 never pass (M2-159) | [chapter 7](07-events-and-waits.md) |
| 10 | The condition `wait 11` tests | it passes at once in every tested state (idle with C58 0 or 4, timeout disabled, DMA in flight, encode running, command or status latch pending, every flag acknowledged, C10 = 0, 0x044 all ones) and on every iteration through NAND array busy periods (after 30h, FFh and an erase start), in every operand form, at 1.25 cycles, the cost of a passing `wait 0` (M2-10, M2-69, M2-107, M2-387); no tested state makes it wait | [chapter 7](07-events-and-waits.md) |
| 11 | Why a latch written one or two instructions after an unwaited `70h` sets `FMC_STATUS` bit 3 instead of bit 23 | a latch needs no re-arm: after its `wait 5` and only the bit-23 acknowledge a second latch passes with no operation between and after each of seven operations (M2-494), and after the `0x30020` release after any or no operation (M2-237); after a latch without the release a new command is refused until an `FMCTRL1` write, with bit 26 clear (M2-181, M2-202), and an address operation does not complete and sets bit 26 (M2-610), while a DMA in or out, a data, parity or FIFO read, an encode and a second latch complete as after the release (M2-641); status clear, ECC kick and `FFh` do not release (M2-181); with an unwaited `70h`, a latch with 0, 3..20 `nop`, 20 or 200 instructions between passes, one after one or two `nop`, one `waitc`, one `movi` or the acknowledge faults with bit 3 set and bit 23 never set, after a reset and after any of eight predecessor runs (M2-68, M2-178, M2-390, M2-593); with the `70h` waited for and acknowledged every tested idle ordering passes (M2-130); the standard latch never fails (M2-161) | [chapter 7](07-events-and-waits.md) |

## 17.3 Interrupts

| # | open point | known so far | details |
|---|---|---|---|
| 13 | The device behind VIC0 line 5 (gates0..108 have before/after samples only, with 42 already disabled; none excluded by M2-744; no gate109..159 result, M2-745); VIC1 lines other than 0 and 1 asserted by hardware (line 52 never rose, M2-724); an ECC interrupt source and enable outside the tested words | `waitirq` follows VIC0 hardware lines 5 (always), 8 (timer), 16 (DMAC0), 19; soft interrupts never release it (M2-74, M2-140, M2-167); FMC flags routed by `0x044` bits 0, 1, 2, 11, 12 drive line 20 and CS_IRQ bit 4 (M2-142, M2-200, M2-204); `waitirq` 32 and 33 follow hardware VIC1 lines 0 and 1, each only its own (M2-723); without a hardware line conditions 32..255 never pass (M2-74, M2-211); END sets VIC1 raw bit 22 while C10 ≠ 0 (M2-182); no ECC encode or decode raises a VIC line or CS_IRQ bit 5/6 for any tested enable (`0x800`..`0x9fc`, single bits, `ECC_CTRL` and `0x810` extra bits; M2-212, M2-217); `sig` raises no VIC line (M2-370); 28-us pulses on enabled gates 1/2 leave VIC0 raw 0x20 before/during/after; gate0 was already off, and no record returns from gate3 (M2-729). With the corrected pulse, gate3 also returns with exact restoration and VIC0 raw 0x20 before/after; the following FREE stays queued while USB works (M2-742). The pulse cannot identify a transient line change while gated (M2-730); the scheduler starvation cause is unresolved (M2-743). An identical gate3 request later precedes USB loss without the EXECFIRMWARE entry marker; no RUN acceptance or image hash separates the causes (M2-746, M2-747). The checkpoint harness supplies pre-start USB evidence and CPU/code identity, with native validation only (M2-748) | [chapter 7](07-events-and-waits.md), [chapter 9](09-nand-controller.md), [chapter 10](10-ecc-engine.md) |

## 17.4 NAND controller registers

| # | open point | known so far | details |
|---|---|---|---|
| 15 | Why the buffer-select switch of a running DMA falls at a different word in identical runs (57..63 at 16 `nop`) while the address switch does not, and why the word at the switch is buffer-1 word 120 | an address rewrite takes effect at a multiple of 32 bytes, equal in repeats; a select rewrite at a word boundary no later (M2-671, M2-672); at 16 `nop` the switch falls at word 63, 57 and 58 and the word at it is buffer-1 word 120 in two of three runs, at 32 `nop` at word 114 in all three (M2-694..M2-696); every byte of an overlapped DMA is an input byte, the initial content or zero (M2-697, M2-698). Not measurable with this harness: the program sees only the RAM image after the transfer, one per run, not the buffer read and RAM write order inside it | [chapter 9](09-nand-controller.md) |
| 16 | What FMC_DMA_CTRL (038) bits 5:3 select and why a non-zero preset of bits 12:6 stops a DMA out after 32 bytes; whether bits 2:0 set the burst length and bits 12:6 count words (inferred) | with bits 2:0 = v = 0..7 a DMA completes, more slowly for smaller v, and leaves bits 12:6 = (v + 1) × ceil(128 / (v + 1)) − 128 (M2-79, M2-609); any non-zero preset of bits 12:6 stops a DMA out after 32 bytes (M2-669); of bits 5:3 only 0 and 1 let a DMA out complete, 2..7 stop it after 32 bytes holding source words 0 1 0 1 2 3 4 5 (M2-669, M2-700); with 0xf a DMA out repeats each burst's first word and a DMA in stores buffer word 4g + j from source word 4g + 5j (M2-669, M2-701; four-beat bursts with 20-byte beat steps inferred). Not measurable with this harness: it sees the completed buffer or RAM and the 038 readback, not the bus beats, so the field meanings stay inferred from word orders; bits 12:6 read non-zero while a DMA runs and 0 again after it (M2-722) | [chapter 9](09-nand-controller.md) |
| 18 | Roles of 018 and the `0x100` block; the role of FMC_CTRL0 bit 0 | 018 stores 4 bits and changes no loopback or page read (M2-104, M2-405); `0x1b8..0x1d8` writes change no measured timing or data (M2-160, M2-210); bit 0 is required for a data transfer to complete, its data-path-enable meaning inferred (M2-135). Unknown because no measured transfer, timing or flag depends on these words beyond what is listed | [chapter 9](09-nand-controller.md) |
| 19 | Combined bits of the 004 operation words beyond their measured effects; what `FMC_STATUS` bits 19:12 encode; what else sets bit 26 | single `FMCTRL1` bits: bit 0 starts an address operation, bit 1 a data read, bits 8 and 9 a DMA, the other 28 nothing alone (M2-411); bits 19:12 read `0xc3` after reset, a DMA in, a status latch and the set-up word, `0x41` after a read of 1..224 bytes or a parity read, `0x14` after 255..508 bytes and `0x3c` after 511 or 512 (M2-203, M2-497, M2-642, M2-706, M2-707); bit 26 sets when an address operation does not complete after an unreleased latch (M2-181, M2-610, M2-641). The three values change at length boundaries only, so the bits read as a state code rather than a count, and no other register shows what that state is; combined operation words beyond the original ones start operations that the harness cannot read out bit by bit; sampled during a read the field goes 0xc3 → 0x41 → 0x14 → 0x3c as the read advances (M2-720) | [chapter 9](09-nand-controller.md) |
| 20 | Coherency of the CPU cache with sequencer and DMA accesses | a DMA from the unmapped address 0x50000000 completes and an `ldi` from it hangs the whole system (M2-717, M2-718); `ldi`/`sti` addresses (M2-05) and DMA RAM addresses (M2-664) align down to a multiple of 4; no measured snooping provides coherency | [chapter 4](04-sequencer-memory-map.md), [chapter 9](09-nand-controller.md) |
| 21 | The M1 channel-1 pin mapping on S5L8720; a second controller channel elsewhere than `+0x400` | no register set at `+0x400`: its words read 0 after each of 17 candidate enable writes, and commands, READ ID and DMAs through it do nothing (M2-82, M2-183, M2-185, M2-393); four attached CEs do not imply two channels; a sequencer `ld` of `0x40c` costs the same 2 cycles as one of `0x00c` or of the unused `0x60c`, so access time does not show whether `+0x400` is decoded (M2-596) | [chapter 9](09-nand-controller.md) |

## 17.5 ECC engine

| # | open point | known so far | details |
|---|---|---|---|
| 23 | 810 bits 13..15, 17..23 and 25..30 and the 80c bits outside the error count; the role of 814 | 810 bits 0..3, 5, 6 assigned (M2-88, M2-94, M2-95, M2-97, M2-99, M2-100); bit 4 reads 1, bits 7..12, 16, 24 and 31 are never set (M2-420); only bit 0 of an 81c write acts (M2-91, M2-96, M2-421, M2-645); 808 reads `0x700` (M2-116), 810 bit 5 threshold role inferred (M2-434); 814 stores mask `0x2f` (M2-191) and leaves BCH8 and BCH16 encode parity, decode registers and decoded data unchanged (M2-660, M2-708..M2-710). Unknown because no tested encode, decode, error pattern or format sets the unassigned bits or depends on 814 | [chapter 10](10-ecc-engine.md) |

## 17.6 Original programs and the CPU interface

The CPU-only read/program/erase results of M2-711..716 remain qualified for
their measured paths. The generic sequence path subsequently exposed a
DMA/FIFO mode-selection defect despite successful resets and ready statuses
(M2-727, M2-728). Its correction is validated in the native model, without a
new device readback result. The command reference profile exercises cache,
copyback, multi-plane and discovery templates, but does not establish optional
NAND support on the installed chip (M2-730).

| # | open point | known so far | details |
|---|---|---|---|

## 17.7 Reset stages

No point remains open here: `FMCTRL1` bit 31 zeroes FMC `00c`..`05c` and sets FMC_DMA_CTRL to 7, and `0x81c` bit 0 zeroes `ECC_CTRL`, which accounts for what the complete reset does to these words (M2-123, M2-645; [chapter 8](08-sequencer-control-registers.md)).

## 17.8 Settled since the first M2 survey

These points were open in the first M2 survey or in an earlier version of the
items above; the reference chapters give the results. Former item numbers are
not reused.

| point | findings |
|---|---|
| PC width (former item 2, part): at least 25 bits | M2-725 |
| `waitirq` and VIC1 hardware lines (former item 13, part): condition 32 + k follows VIC1 line k for k = 0, 1 | M2-723 |
| original WRITE/ERASE on hardware (former item 24) | M2-726 |
| unmapped bus accesses (former item 20, first part): a DMA from 0x50000000 completes, an `ldi` from it hangs the system | M2-717, M2-718 |
| Source selector 10 = destination register in every MOV/ALU form | M2-44, M2-52, M2-141 |
| BNZ/BZ and JR target encoding; JR condition (r[R] ≠ 0) | M2-46, M2-105, M2-106, M2-109, M2-110, M2-122 |
| No register-write hazards | M2-108 |
| Undefined opcodes `1a..fe`: same outcome as an unmet wait, no separate trap | M2-56, M2-81, M2-282 |
| `sig` (`10`) and `nop` (`ff`) in the three tested forms, idle, DMA and ECC | M2-34, M2-65 |
| Timeout length rule and start-word bits 7:4 | M2-81, M2-103, M2-115 |
| Instruction costs and engine clock | M2-09, M2-102, M2-103 |
| STOP and reset-before-parameters | M2-53, M2-54, M2-55 |
| Recovery from a held state and from C60 written while running | M2-61, M2-62, M2-63, M2-121, M2-126 |
| Repeated waits, acknowledgement and rearm | M2-66 |
| WAIT4 release: encode 810 bit 1, decode 810 bit 2, W1C | M2-67, M2-97, M2-99 |
| No WAIT follows NAND ready/busy | M2-107 |
| 04c is chip status (0x80 busy, 0xe0 ready, 0x70 no-CE echo); status latch polls until ready | M2-87, M2-130, M2-136 |
| WAIT12 condition: status & arg & mask ≠ 0; release on a busy chip only at ready | M2-68, M2-134, M2-137 |
| FMC and C50 operation flags | M2-112 |
| Serial DMA exact; cause of overlap payload loss | M2-71, M2-76, M2-78, M2-114 |
| NAND read, write, command and address cycle timing | M2-119, M2-127, M2-131, M2-135 |
| FMC_CTRL0 stored and inert bits | M2-135, M2-139 |
| FIFO READ ID | M2-124 |
| BCH8 code and correction radius 8 | M2-73, M2-77, M2-84 |
| BCH16 code and correction radius 16 | M2-86, M2-92 |
| Per-decode results: 80c ring, read advance, 81c reset | M2-85, M2-90, M2-91 |
| 810 status bits (done, uncorrectable, ≥ 7 errors, erased) and 818 total | M2-88, M2-89, M2-93, M2-94, M2-95, M2-96, M2-100 |
| ECC completion after repeated encodes | M2-67, M2-72, M2-97 |
| ECC_CTRL single bits | M2-98, M2-101 |
| Encode parity registers 0x840..0x85c | M2-116 |
| Decode error locations: data, metadata, parity order and numbering | M2-128, M2-132, M2-138 |
| DMA, ECC and decode timelines | M2-117, M2-118, M2-129, M2-133 |
| Shift counts: every count ≥ 32 gives 0, no count aliasing | M2-146 |
| Timeout step for n = 0 (fault at once), n = 2 and n = 3 (2^3, 2^5 cycles) | M2-115, M2-216 |
| `end` and the ignored fields of `wr`, `st`, `ldi`, `mov`, `movi`, `sti`, `pld`, `pst` in the tested values | M2-164, M2-165 |
| `pld`/`pst` offset bits 11:0 and unaligned offsets | M2-175 |
| PC writes through `0xc38` by the program and the CPU | M2-162, M2-278 |
| Start-word bits 1, 3, 16, 19 and the bits without effect; no priming start after a reset | M2-151, M2-153, M2-157, M2-158, M2-201 |
| `C58` bit 0 and END diversion: no value diverts an END | M2-148, M2-149 |
| `C3C` in a fault handler, also nested; `C3C`/`C48`/`C54` widths and retention over a start | M2-154, M2-179 |
| `C14` ignores writes | M2-176 |
| `C50` bits 0..5 mirror pending events | M2-152, M2-209 |
| `wait` event numbers 16..255 and `wait 8` | M2-159, M2-206 |
| WAIT5 orderings: faults only for a latch one or two instructions after an unwaited `70h`; every idle ordering with a waited `70h` passes; the standard latch never fails | M2-68, M2-130, M2-161, M2-178, M2-593 |
| Command refused after a latch until an `FMCTRL1` write; two-chip polls; latch after `90h` | M2-181, M2-198, M2-202, M2-239 |
| Status latch on `FMC_DATA_SEL` `0x100` and `0x200` | M2-169, M2-195, M2-196 |
| `evjmp` events arriving during the wait, pending events and `C58` | M2-186 |
| `waitirq` ignores arg/imm; no NAND busy line; VIC1 conditions never pass | M2-167, M2-187, M2-211 |
| FMC interrupt routing: `0x044` bits, VIC0 line 20, `CS_IRQ` bit 4 | M2-142, M2-200, M2-204, M2-208 |
| END sets VIC1 raw bit 22 while `C10` ≠ 0 | M2-182 |
| No register set at `+0x400`: reads 0, commands and DMAs through it do nothing | M2-82, M2-139, M2-155, M2-183, M2-185 |
| Window masks, offset bit 9 decoding, the `0x100` block and `0xd80..0xdfc` | M2-147, M2-154, M2-155, M2-160, M2-191, M2-210 |
| FMC `0x028` is a 9-bit storage word, not a counter | M2-156, M2-173, M2-199 |
| `FMC_STATUS` bits 12/13/18/19, 24 and 27 | M2-145, M2-163, M2-203 |
| DMA length independent of `FMC_DNUM`; parity buffer 64-byte wrap | M2-189, M2-232 |
| `FMC_CTRL0` bit 24 clear: no DMA completes and, left clear until END, no byte reaches RAM | M2-184, M2-251 |
| Four-cycle erase; programs and erases on CE0 and CE1 | M2-168, M2-194 |
| `ECC_CTRL` combinations: no code other than BCH8 and BCH16 | M2-197 |
| M1 ECC area `0x900..0x9fc` reads 0 on M2 | M2-171 |
| No miscorrection beyond the radius; all-zero chunk decodes clean | M2-174, M2-214 |
| Original WRITE DMA-list prefetch | M2-213 |
| `pld`/`pst` reach: offset bits 11:0 only for every bit 12..31; r0..r7 through `0xc18..0xc34` (former item 5) | M2-175 |
| `evjmp` (`16`) operand forms and sources; no `+0x400` flag; every `C00` order on a blocked start = 1 `evjmp` (former item 12) | M2-62, M2-186, M2-391, M2-392 |
| `FMC_CTRL0` bit 24: a DMA programmed with it clear is held and starts when it is set again, with the address registers at that moment (former item 14) | M2-184, M2-251, M2-338 |
| A DMA writing or reading a buffer under its own encode: kept old bytes, lost output runs, parity and `FMC_STATUS` bit 25 (former item 17) | M2-80, M2-399, M2-401 |
| ECC slot selector `01c`: bit 0 selects the slot; back-to-back kicks; a slot change during an encode (former item 22) | M2-417, M2-418, M2-419 |
| Power-on window: equal to the window after the ordinary reset except `C00` and `C04`; `C14` `0x7f`; the first program runs on its first start (former item 25) | M2-430, M2-431 |
| `sig` routing and cost; `tmwait` time from the start-word count; erased words as `nop` | M2-367, M2-370, M2-373 |
| Instruction costs per copy; `C08` after a fault and in a handler; `C14` load and count rule; `C48` and `tmwait` | M2-172, M2-177, M2-179, M2-378, M2-384 |
| WAIT5 fault orderings independent of the preceding program | M2-390 |
| `FMCTRL1` single bits; 048 and 04c writes; `FMC_STATUS` bits 19:12 | M2-203, M2-411, M2-412 |
| `FMCTRL1` DMA word replaced by a read | M2-332 |
| FMC_DMA_CTRL (038) readback and fault bits | M2-396, M2-398 |
| 810 never-set bits, 81c bit 0, CPU reads of 80c advance the ring | M2-91, M2-420, M2-421 |
| Two-plane program and erase; CPU-only NAND operation | M2-339, M2-435, M2-436 |
| Undefined opcodes `1a..fe` with a pending command, address, DMA, ECC or status flag: unmet wait (former part of item 3) | M2-583 |
| `tmwait` entered later in a run; count-0 `tmwait`; `C48` and `tmwait` in every run (former parts of item 4) | M2-179, M2-584 |
| Timeouts of waits entered late in a run, counted from the entry; the step for start-word bits 7:4 = 1 (former parts of item 6) | M2-115, M2-584, M2-588 |
| `C08` read inside running and waiting states; `C58` bits 6 and 7; `C14` kept after the END (former parts of item 8) | M2-172, M2-384, M2-612 |
| A second status latch needs no re-arm; an address operation after an unreleased latch fails with bit 26; the timing window of a latch after an unwaited `70h` (former parts of item 11) | M2-494, M2-593, M2-610 |
| Register rewrites during a running DMA: address and buffer select followed, `FMC_DNUM` and a repeated `FMCTRL1` write ignored (former part of item 15) | M2-394, M2-603 |
| FMC_DMA_CTRL bits 12:6 during and after a DMA; readback of 2 and 4 as `0x42`/`0x84`; no value of bits 2:0 stops a DMA (former parts of item 16) | M2-398, M2-609 |
| A DMA in with `FMC_DATA_SEL` = 0 writes neither data buffer; CPU writes of 058/05c read back (former part of item 18) | M2-597, M2-599 |
| The set-up word restores `FMC_STATUS` bits 19:12 after a short read (former part of item 19) | M2-203, M2-497 |
| Ring-advancing copy of `0x80c` at `0xa0c`; `0x90c`/`0xb0c` | M2-91 |
| High source- and address-selector bits of `and`, `or`, `sub`, `shl`, `shr`, `mov`, `ldi`, `pld`, `pst`; loads and immediate ALU results with destination r10 or r8 (former parts of item 1) | M2-364, M2-614, M2-615, M2-616 |
| `waitc` (`08`) operand map: R = 0..7 continue, R = 8..0xff stall, no pending flag releases them; the `waitc` stall time (former part of item 4) | M2-377, M2-625 |
| `C48` single bits and `tmwait`; `C14` writes and `tmwait` (former parts of item 4) | M2-373, M2-627, M2-629 |
| Prescaler phase set by the per-case reset; first instants for n = 4..6 (former parts of item 6) | M2-588, M2-631 |
| `C58` bits 1 and 4; `C14` = `0x7f` after a released wait for every count, step field and step number (former parts of item 8) | M2-177, M2-612 |
| Operations that an unreleased status latch refuses: a command and an address operation only (former part of item 11) | M2-641 |
| Switch offsets of address and select rewrites during a DMA (former part of item 15) | M2-671, M2-672 |
| FMC_DMA_CTRL presets of bits 12:6 and 5:3 before a DMA; bits 31:13 not stored (former part of item 16) | M2-669 |
| `058` and `05c` as bytes of `FMC_DATA_SEL`; DMA-in targets of every tested `FMC_DATA_SEL` value; data buffer 0 after `FMCTRL1` = `0x340` and FIFO writes (former parts of item 18) | M2-617, M2-643, M2-668 |
| The `0x550d` readback of `FMC_DATA_SEL`; `FMC_STATUS` bits 19:12 against the transfer length (former parts of item 19) | M2-642, M2-643 |
| DMA RAM address alignment (former part of item 20) | M2-664 |
| Role of 814 in BCH8 decode and encode; parity with metadata length `0xff` (former parts of item 23) | M2-660, M2-662 |
| Reset steps that zero FMC `00c`..`05c` and `ECC_CTRL`: `FMCTRL1` bit 31 and `0x81c` bit 0 (former item 26) | M2-123, M2-645 |
| High-bit source selectors read the last source value; discarded extended destinations and the `st` latch; `jr` with high target-selector bits (former parts of item 1) | M2-618, M2-622, M2-673 |
| Extended destinations 9 and 11..15 with extended sources 8 and 10: no GPR written, latch unchanged; `sti` with high address-selector bits: the last source value; `ld`, `st` and `wr` leave the value a high-bit source reads (former item 1) | M2-674, M2-675, M2-676, M2-677, M2-678, M2-679, M2-680, M2-681 |
| Boot ROM through `ldi` at `0x2000xxxx`: one ROM word per 16-KiB quarter, never the addressed word (former part of item 2) | M2-682, M2-683 |
| Operand fields of the undefined opcode `0x1a`: no effect in 64 random words (former item 3) | M2-684 |
| `waitc` hold for R ≥ 8 is a wait state; C48 compare rule at count 7; the isolated early `tmwait` ends are run-to-run spread (former parts of item 4) | M2-685, M2-686, M2-687, M2-688, M2-689, M2-690 |
| No single reset step sets the prescaler phase alone (former part of item 6) | M2-691, M2-692 |
| Word at a buffer-select switch: buffer-1 word 120; overlapped DMAs contain only input bytes, initial content or zero (former parts of item 15) | M2-694, M2-695, M2-696, M2-697, M2-698 |
| FMC_DMA_CTRL bits 5:3 = 3, 5, 6 and the DMA in with `0xf` (former part of item 16) | M2-699, M2-700, M2-701 |
| DMA-out `FMC_DATA_SEL` values; the transfer after `FMCTRL1` = `0x340` completes by time alone and stops at an `FMC_DATA_SEL` write (former parts of item 18) | M2-702, M2-703, M2-704, M2-705 |
| `FMC_STATUS` bits 19:12 for read lengths 160..508 (former part of item 19) | M2-706, M2-707 |
| 814 and BCH16 encode and decode (former part of item 23) | M2-708, M2-709, M2-710 |
| CPU-only NAND read, program and erase without a sequencer program, with engine and software ECC, both formats; decode location order within a byte | M2-711, M2-712, M2-713, M2-714, M2-715, M2-716 |

References: [chapter 14](14-evidence.md) (outcomes), [chapter 13](13-experiments.md)
(what was done), [chapter 15](15-evidence-interpretation.md) (interpretation).
