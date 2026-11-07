# Chapter 16 — Physical M2 findings through P43


Sections 16.1 to 16.4 are the historical p06 observations; the later rounds follow
in order from [section 16.5](#165-p07-pause-recovery-and-the-next-failure) (p07) to
[section 16.23](#1623-p25-jr-flags-dma-timing-prescaler-parity-registers) (P25), with
[section 16.24](#1624-p43-round-5-and-the-cpu-only-nand-path) (P43). Results of
the later rounds are in [chapter 18](18-ledger.md).

Audited capture: m2-all-p06, 589,189,971 bytes,
SHA-256 `4c804e08dd2e326219266d9eed55f71b489776f51a59b7971202c4920240f8c2`.
The capture started at `20261101T071245.434221Z`, reports S5L8720, probe v14,
bulk USB and resident batches. The image hash was **not recorded**. Do not
attribute it to the current local binary by filename or matching probe version.


## 16.1 What the completed campaign contains

All 159 experiment entries returned a result. `attempt_finished=true` means the
host reached the end of the campaign; `complete=false` records errors. The
summary retains these case records:

| Classification | Records | Meaning |
|---|---:|---|
| Sampled execution | 140 | `nsamples>0` and `polls>0`; includes END, fault and host timeout |
| Setup/preflight did not reach sampled execution | 22,972 | `pre_ok=0`, no samples; all have worker rc `0xfffffffe` |
| Explicit case errors | 6,083 | 2,673 mutation-capability refusals; 3,410 dependency errors |

These are **retained summary records**, not a count of unique native submissions.
Legacy report dictionaries discard some setup probes and overwrite repeated
labels. For example, `startseq2` submitted 18 cases but retains nine final
captures; `startseq3` submitted 16 and retains eight. No p06 JSONL was supplied
with this file. The full native order cannot be reconstructed from the summary
alone. The [coverage table](13-experiments.md) gives retained counts per entry.

The reported capability word is `0x7a0000`: bulk, resident v1/v2 and discovery
are present; mutation bit 18 is absent. The write refusals are harness decisions,
not evidence that M2 rejects writes or that a NAND operation failed. This does
not change the user's authorization to run those experiments.

<a id="execution-chronology-and-pause-12"></a>

## 16.2 Execution chronology and `pause` (`12`)

`state` through `op15` reproduce the earlier architectural observations.
`op12` case `[0xd60] & ffffffff == r2 (1234)` is the first sampled case with a
cleanup error:

| Field | Observed value |
|---|---|
| Instruction latches C40/C44 | `12020d60 / ffffffff` |
| PC (`c38`) | `0x30`, the `pause` (`12`) instruction |
| C08 / CS_IRQ | `0x0a / 0x08` |
| C00 | `0x0004ffb4` |
| Observation interval | 50,001 us, host `TIMEOUT` |
| Worker return | `0xfffffffe` (-2) |

The seven subsequent `op12` records and five `c58ill` records did not execute
their requested instructions. Most retained `startseq` cases also did not
execute; its explicit no-reset/no-preflight case was sampled and timed out.
The retained `startseq2` and `startseq3` cases then reach END. This establishes
recovery in this sequence, not the exact cause: omitted setup captures matter.

Later, `op12wait/idle/op12/n0/arg0/mask0/order0` reproduces the pause signature
with a different encoding (`C40=12000000`, `C44=0`) at PC `0x58`, C08=`0xa`,
IRQ=`8`, and a 20,001 us observation interval. Cleanup again returns -2. Its
remaining 159 cases, and **every later experiment's retained probe captures**,
have no sampled execution.

**Interpretation:** the two forms support a pause/CPU-handoff operation, matching
the original M2 CPU driver's handling of IRQ bit 3. They do not establish all
ignored operand bits or demonstrate successful resume. In particular, the old
test label suggesting a masked comparison is a hypothesis, not the conclusion.
The current reset helper handles C08=4 specially but has no equivalent branch
for C08=`0xa`; bounded polling can return -2 in that state. This is a source-based
explanation of the observed sequence, not an additional silicon measurement.


## 16.3 Positive architectural observations

Paths below are under `results/<experiment>/data` in p06.

| Experiment/case | Observation | Scope |
|---|---|---|
| `regmap`; `control` | r0..r7 at c18..c34; r6 aliases result; CPU-preloaded r3 survives and can be stored | tested register values |
| `clock` | 2,000/20,000 two-instruction loops take 138/1,355 us | slope ≈33.8 ns/instruction in this configuration, not a clock proof |
| `pram` | CPU writes persist d00..d7c; tested sequencer reads agree; d80..dfc writes do not persist | 32 observed parameter words; no claim about other aliases |
| `alu` | logical SHR, arithmetic wraps; tested counts 32/33 produce zero; immediate-zero ADD uses register form; LD masks by AND | nine concrete cases |
| `memory` | little-endian words; tested unaligned loads/stores align down | system-memory operations, not instruction target alignment |
| `indirect` | 18 reads GPR/PRAM window; 19 writes PRAM; 1d64 reads like d64 | this alias only; not all high bits |
| `resume` | C58 bit 0 does not start at the vector; CPU reads it back clear | tested start configurations |
| `wait` | WAIT11 passes in ~4 us; other idle events 0..15 fault in ~126.6 ms | supplied idle state and operands |
| `c58` | bit 2 distinguishes vector/continue after timeout; tested readback fits `value & 0xd6` | low-value sweep plus 40/80/ff/100/ffff |
| `op15` | all 24 forms wait through the observation window; C08=4; C14 changes 7f→35 | release condition and full timer semantics remain unknown |
| `startseq3/no-op (0x08) first` | instruction 08 is followed by marker and END | supports no-op-like behavior for this one encoding only |
| `holdout/shl 1 by 31 / shr by 31` | r2=`80000000`, r3=`1`, END | adds boundary count 31 |
| `holdout/pld 0xc30 reads r6` | r2 and r6 both `6666aaaa` | indirect result-register read |
| `holdout/sti then ldi at scratch+0x200` | aligned store, unaligned reload gives `13572468` | another RAM alignment check |
| `holdout/loop of 100` | marker and END after ~8 us | branch/counter combination |


## 16.4 Results that cannot answer their labels

* `oneshot`'s two fault candidates and `c50`'s three fault candidates were
  observed for only 50 ms. The idle WAIT timeout is ~126.6 ms here. No fault
  within that shorter interval does not establish one-shot suppression or the
  value that a fault handler would eventually read.
* `c50/program jumps to the handler (no fault)` does execute the handler and
  reads `00ff1800` before and after writing 1. It is a **no-fault** control.
* `startseq3` markers finish in 3–4 us, before proposed CPU writes at 2 ms.
  `late_at_us=0`: these rows do not measure the effects of those later writes.
* `opsweep`'s 717 captures all lack execution. Its legacy “stopped fast” labels
  describe zero-initialized output, not 717 opcode behaviors. Likewise zero
  nanoseconds from `opcost`/`optime` is not an instruction timing result.
* All 531 confirmation, 78 M1-hypothesis, 1,422 discovery and 105 state-opcode
  cases are explicitly **inconclusive**. No new opcode aliases or illegal
  ranges are established by those sweeps.
* Later register maps, ECC results and READ ID zeros are not hardware values.
  Preserve the earlier successful READ ID capture; p06 does not contradict it.

## 16.5 p07: pause recovery and the next failure

Capture m2-all-p07, 641,793,213 bytes, SHA-256
`751e38b88144d160fecb9ba2c1de5b0076e98ba2fcf547c8f4ac334bc5130653`.
Capture start: `20261101T161039.840011Z`. All 160 entries use native-case records,
resident batches and bulk USB. The minimum observation window is 160,000 us.
The capability word is `0xfe0000`, including mutation and recovery. The running
image hash is again absent; no adjacent JSONL was supplied. Do not identify the
physical image with a local build merely because their capabilities agree.

| Classification | Native case records |
|---|---:|
| Sampled | 360 |
| Initial setup/preflight did not execute candidate | 25,494 |
| Dependency errors | 3,425 |
| Total retained | 29,279 |

All sampled cases except the first `ev5` case return worker rc=0. These counts
are not successful semantic confirmations: sampled cases include faults and
observation timeouts. The full campaign reaches its end,
but the broad opcode families again have no sampled execution.

### 16.5.1 `pause` cleanup and CPU resume work in the measured controls

All nine `m2_recovery` comparisons match. The two measured operand forms retain
IRQ8/C08=0x0a and r6=0 while paused. Trusted-END cleanup returns rc=0 in 3–4 us
(method 1), and the following controls reach END with r2=0x7265636f.
CPU writes PC to the following instruction, acknowledges IRQ8 and writes
CTRL=0x4ffb5 at 100–101 us; both forms then reach END at 102–103 us with
r6=0x600d600d. These are physical confirmations of those sequences, not merely
accepted register writes.

The first baseline starts with an inherited pause at PC=0x58. Its recovery uses
method 3 (trusted-END attempt plus abort fallback), returns to idle in 1,006 us,
and the baseline executes. This demonstrates the combined fallback sequence in
that starting state; the capture does not isolate abort as the sole cause.
The later direct pause controls use method 1 successfully. Thus recovery depends
on starting state; neither path is established as a universal reset recipe.

`op12` now samples all eight comparison-labelled cases, all pausing with IRQ8.
`op12wait` samples all 160 cases: 80 `pause` cases retain IRQ8 and time out, while
80 `wait` cases yield 9 END and 71 faults. Across these cases, 20 distinct pause encodings are observed: R=0..15 with
arg=imm=0, plus R=2 with arg=0xd60 and imm=0/0xff00/0xffffffff or arg=0xd64
and imm=0xffffffff. The comparison labels do not turn `pause` into a masked comparison. Nor do state labels such as “erase” prove that
the intended peripheral operation actually completed.

### 16.5.2 `tmwait`, faults and event controls

All 24 `tmwait` (`15`, experiment `op15`) forms reach END in 126,101–126,103 us,
with IRQ=1 and C14=0. This resolves the earlier 50 ms observation timeout:
the tested instruction does eventually continue. It supports the timer-wait
name; the exact reload value, divisor and all operand effects still need tests.

The three faulting `c50` controls now reach their handlers at about 126.6 ms.
They read 0x00ff1800 before and after writing 1 to C50, as does the no-fault
handler control. M1's event/acknowledgement interpretation is not established by
these unchanged values. `oneshot` now records two END and two fault outcomes;
full one-shot behavior still needs the longer dedicated sequences.

The `op12wait` controls pass `wait 0` after command stimulus (FMC_STATUS=0x000c3002),
`wait 2` after DMA stimulus (0x001c3000), `wait 4` after ECC stimulus
(ECC_STATUS=0x13), and `wait 5` after status stimulus (FMC_STATUS=0x008c3000).
`wait 11` passes in all five tested stimulus groups; the other selectors fault.
The post-run ECC_STATUS=0x13 does not have bit 2 set: do not promote the current
model's simple bit-2 predicate to a measured hardware equation. These are
qualified state/control pairs, not full event definitions. Idle `wait 0` faults. The paired `pause` cases retain IRQ8. The `ev11` matrix
passes its four `wait 11` forms and faults its four `wait 12` forms. Pending,
acknowledged and consumed event behavior is still only partially measured.

### 16.5.3 First failure: status-latch experiment, then reset

`/results/ev5/data/0000~1latch_matrix` issues a status command, starts the latch
with FMCTRL1=0xca and waits on event 5. It faults after 126,595 us and reaches
its fault-handler marker: r6=0xfa17fa17, IRQ=5, C08=0. Its final pre-cleanup
snapshot has FMCTRL1=0x0000000a, FMC_STATUS=0x008c3000 and C50=0x00ff1820.

Cleanup then fails: stage 4, method 0, reset rc=-3 after 102 us, C08 still 0.
The next case fails at initial reset (stage 1), before preflight; every later
candidate capture is unexecuted. This is different from p06's paused C08=0x0a
failure. A sequencer reaching END does not guarantee its peripherals can reset.

The original reset polls both the c60/c64 buffer handshake and FMCTRL1 bit 30,
each with a 100 us bound in this build. Current diagnostics do not record which
poll failed or its final peripheral registers. The 102 us/idle signature
therefore does **not** identify one of those polls uniquely. Capture the reset
phase and c60/c64/c6c/FMCTRL1 before claiming a cause or a working recovery.

The late 531 confirmation, 78 M1-comparison, 1,422 discovery and 105 state-opcode
checks remain inconclusive. Their zero-filled outputs supply no new instruction
semantics. The next measurement should run pure ISA controls before peripheral
stimuli, and isolate this status-latch/reset transition with phase diagnostics.


## 16.6 Reproduction after p07 (historical model hypothesis)

The first model reproduced FAULT/END and reset failure by making the NSTAT read
consume WAIT5, with either reset handshake blocked. p09's sampled PCs reject
that causal explanation: the first WAIT5 passed. The updated model retains the
persistent transaction, blocks the next command, and defaults to the physically
observed FMC reset phase. Ignored-release and buffer-failure scenarios remain
explicit fault injection. Earlier model matches did not prove that mechanism.

## 16.7 p09: status recovery, opcode controls and communication loss

Capture m2-all-p09, SHA-256
`d4d4215df32a96381c58c6fff8c53f29215762e68d25b83ed278c40900ef4316`;
image hash not recorded. There are 4,657 sampled records in 52 returned families,
then `regnum` loses USB communication. No `regnum` output is retained.
The 33 subsequent USB timeouts supply no further architectural measurements.

All three status-order variants fault at the second command's WAIT0 and recover
by method 4, with subsequent markers reaching END. Reset failure phase 4,
FMCTRL1=2, identifies FMC acknowledgement. NSTAT read ordering does not consume
the first WAIT5 as previously hypothesized.

The 531 common-opcode cases yield 526 matches and five incorrect negative
expectations: immediate acknowledgement before WAIT still reaches END for each
peripheral stimulus. Expectations and asynchronous event modeling are corrected.
All 78 M1-comparison expectations match for the tested encodings/states. The
1,422 discovery, 717 legacy sweep and 105 state-opcode samples are observations,
not automatically decoded semantics.

The old `regnum` writes r8=0x5eed0008. If M2 r8 is PC, it fetches outside mapped
memory. That hypothesis reproduces a worker/USB stall under explicit CPU-bus
fault injection. Physical attribution remains unknown. The corrected probes use
local targets; the p10 `m2_wedge` run isolated each request with durable results
(now selected explicitly with `--isolate-diagnostics`).

## 16.8 p10: M1 register-role comparisons on M2

All 59 `m2_wedge` requests execute and reach END, with IRQ=1, preflight and
cleanup success, and all 38 scored controls matching. Capture SHA-256:
`aa918f288abbf3777afb073fadcda131a487544f785ac757ddca0e2d26a21fb5`.
Capture m2-all-p10 contains only that isolation family.
No image hash or raw journal was supplied. These are confirmations of selected
M1 behaviors on M2, not new meanings discovered for r8/r10. See the
[qualification table](04-sequencer-memory-map.md#43-register-file-r0r7-pc-and-extended-register-roles).
At the end of p10, remaining role
predictions still needed physical comparison; p11 below supplies that evidence.

## 16.9 p11: full selector comparisons and source-10 difference

Capture m2-all-p11: 1,590 sampled records, all worker/preflight/cleanup returns
successful; 1,589 END plus the expected JR r8 self-loop TIMEOUT. There is no
communication loss. Seven whole resident families plus three isolated bus
controls completed. Of 1,569 scored cases, 1,568 match M1 and one differs.

`m2_regsrc/argument-source/010` saves four differing register-form outputs:
AND/OR=`22002222`, ADD=`44004444`, SUB=0. With r0=`11001111` and destination
r1=`22002222`, this is self-source behavior rather than M1's r0 rule. SHL/SHR
large counts give zero under both interpretations. All eight broadcast GPRs,
local MOVI/MOV/ADD PC destinations, BZ/BNZ/JR selectors and ST predecessor
contrasts now have M2 measurements. LDI/PLD at PC=0x60 return `e1a0f00e`/0.

The simulator also needed PRAM clearing across reset to match the untouched
words in successive probes. Its exact placement at CTRL=80000 is inferred,
not a phase-isolated hardware observation.

## 16.10 p12: source 10 and isolated reset phases

All 72 requests return in two resident runs: 64 END/matches and eight expected
observation TIMEOUTs with IRQ=0. Every preflight/worker/cleanup succeeds.
Capture is 693,753 bytes, SHA256
`d6013102ed545f12fcc584ccc0f1144a9a50307cdcb0fe9d8c776cafb8093953`.

All 1,664 MOV/ALU result slots match self-source behavior; 245 distinguish it
from M1's r0 register-form rule, including 42 SHL and 21 SHR results. All eight
GPR destinations and eight value pairs are covered; r0 is a deliberate control.

STOP clears PC but preserves GPRs/PRAM. STOP then C00=80000 clears GPRs, PRAM,
C04/C10/C40/C44/C4C. Separate buffer/FMC/ECC reset sequences preserve GPR/PRAM.
This confirms the reset-before-parameters requirement in the tested states.
Raw C00/C14 observations are retained without a full bit/timer-model claim.

## 16.11 P13: full attempt, state-b reset failure

The 171-family capture retains 6,204 sampled records (one cleanup failure),
21,831 initial-reset failures and 2,959 dependency errors. USB remains alive.
The first failure is ctrlfx/0002: start=1, evjmp at PC0x20, C08=b, IRQ0;
STOP does not clear it, and the ordinary reset only attempts abort for C08=4.
All later initial failures share that same state signature. The four proposed
event/DMA/ECC families never execute. Earlier p11/p12 results remain valid.
All 531 common opcode and 78 M1 comparison checks before this failure match.

## 16.12 P14: no candidates execute; forced reset was skipped

The seven-case resident reset check returns seven initial phase 1 failures,
C08=b/PC20/IRQ0, method 8 only. Abort passed the 0/3 check, so the bit-26 kernel
skipped forced reset before its next STOP. Both possible intermediate values
reproduce all seven failure signatures through the old native ARM binary;
this does not distinguish which intermediate value hardware returned.
The corrected kernel always resets while held, captures those intermediate
values and requires the ordinary reset plus the next preflight to succeed.

## 16.13 P15: effective reset and independent execution

Capture m2-all-p15 contains seven sampled cases in one resident
batch, with no errors. All seven pass END-only preflight and both reset
stages; four marker assessments match. WAIT3, zero-field illegal1a and evjmp
started with C00=1 remain in their expected 2 ms observation TIMEOUTs.

The inherited state on entry and the newly triggered evjmp both recover:
STOP leaves C08=b/PC20/IRQ0, abort reports 3, forced reset before another STOP
reports 0, and the ordinary reset finishes phase 6/rc0. The next marker reaches
END/IRQ1/r2=72657365. Method 01000318 retains both intermediate statuses.
This resolves the p13/p14 failure for the tested sequence. It adds no event
consumption, general opcode-field or physical BCH-parity result.

## 16.14 P16: all families execute; data-path questions remain

All 172 families return 31,001 sampled records with zero reset/cleanup errors.
All 531 core and 78 M1 opcode comparisons match; source 10 repeats its already
qualified M2 difference. No new opcode is identified solely by the discovery
signatures. All 68 C58 CPU/sequencer masks match d6; all five C08 self-reads give 5.

Repeated waits establish non-consuming command/address/DMA flags. Acknowledging
them makes the second wait fault; retriggering restores progress. ECC WAIT4
survives the tested ack 1, and masked status WAIT12 survives clearing bit 23.
WAIT5 remains sensitive to command/latch ordering; WAIT15 fails in all tested
clear-enable sequences. The timeout control words change duration substantially.

All 24 overlap/serial programs end, but none has a fully correct first 512-byte
output. Same-slot cases uniquely set 048 bit 25. All 20 repeated-encode programs
end with 13 completion observations; fresh completion is not yet proved. All
2,080 `miscorr` records report RESULT 1 without valid input parity, so they do
not establish a BCH correction radius.

## 16.15 P16 offline: serial DMA and BCH8 settled from retained records

Serial DMA is byte exact (`dmaloop` 8/8, `dmactl` 24/24 page reads). The overlap
payload loss follows from unacknowledged WAIT2 flags in the p16 generator, not
from the data path (inferred; `m2_dma_ack` tests it). The physical BCH8 code
equals M1's, with sector 0 covering ten metadata bytes: 352/352 raw sectors
match.

## 16.16 P17: acknowledgement confirmed; same-slot overlap corrupts

Acknowledging the WAITs makes the p16 program exact; serial and different-slot
DMA/ECC overlap is exact for BCH8 and BCH16. Writing a buffer during its own ECC
operation drops word-aligned bursts and sets FMC_STATUS bit 25.

## 16.17 P18: timeout rule, +400 words, BCH8 radius

The start-word timeout is linear in the full count field. The +400 storage
words read zero without aliasing. BCH8 corrects up to eight bit errors anywhere
in data, metadata and parity and leaves data unchanged beyond that in the four
tested cases; ECC_RESULT does not report the current sector with the original
read sequence.

## 16.18 P19: BCH16, status byte and ECC status interface

BCH16 is the t = 16 narrow-sense code of the same field. The latched status
byte follows busy/ready; 0x70 is the no-chip echo. Per-sector decode status is
0x810 bits 3/5 and the 0x818 running total; 0x80c is a read-advanced ring.

## 16.19 P20: result ring and BCH16 radius

0x80c is an eight-slot ring advanced by its own reads and realigned by 0x81c;
BCH16 corrects up to 16 errors.

## 16.20 P21: ECC status bits and WAIT4

810: bit 1 encode level (WAIT4), bit 2 decode, bit 3 uncorrectable, bit 5 ≥ 7
corrected, bit 6 erased; ack 1 clears only bit 0. ECC_CTRL bits 15:8 are the
metadata length.

## 16.21 P22: WAIT4 sources, engine clock

WAIT4 follows 810 bit 1 (encode) or bit 2 (decode). Instruction timing and the
timeout step agree on a 33.25 MHz engine clock (inferred, M2-103).

## 16.22 P23: branch targets, JR hazard, no ready/busy WAIT

BNZ targets are byte offsets with the low 3 bits ignored; every JR fell through
with its target register written by the preceding instruction, later explained
by the JR condition r[R] ≠ 0, not by a register hazard (M2-108, M2-110); no WAIT
follows NAND ready/busy (M2-105, M2-107).

## 16.23 P25: JR, flags, DMA timing, prescaler, parity registers

JR jumps iff r[R] ≠ 0; only WAIT 5 passes, after a status latch; the FMC_STATUS
and C50 flag bits; DMA timing; the timeout prescaler; the encode parity
registers (M2-110..M2-116).

## 16.24 P43: round 5 and the CPU-only NAND path

All 18 families executed with every requested reset verified. Round 5 settles
the high-bit selector and latch rules (M2-674..M2-681), the boot-ROM view of
`ldi` (M2-682, M2-683), opcode 0x1a and the `nop` hold (M2-684..M2-686), the
C48 compare and `tmwait` repeatability (M2-687..M2-690), the reset-step origin
of the timeout phase (M2-691, M2-692), the DMA select, overlap, 038 and 0x340
questions (M2-694..M2-705), FMC_STATUS bits 19:12 by length (M2-706, M2-707) and
0x814 under BCH16 (M2-708..M2-710). C08 while running remains a single sample
per run (M2-693). The CPU alone reads, programs and erases both page formats
with hardware or software ECC, equal to the sequencer readers (M2-711..M2-716);
208 of 211 checks match, 1 is an observation, and the 2 mismatches are the
location-order expectation (M2-714).

## 16.25 Reboot sessions: unmapped bus accesses

A DMA from 0x50000000 completes and leaves the device responsive (M2-717); an
`ldi` from the same address hangs the whole system, also in stall-safe mode, and
only a power cycle recovers it (M2-718). Both are the opposite of M1, which
settles M1-F059 (chapter 19 now has no unresolved finding).

## 16.26 P48: round 6

The CPU sampler shows C08, FMC_STATUS bits 19:12 and FMC_DMA_CTRL bits 12:6
changing while the operation runs (M2-719..M2-722). `wait2` 32 and 33 follow
the hardware VIC1 lines 0 and 1 (M2-723). The PC is at least 25 bits wide
(M2-725). Apple's original ERASE and WRITE programs run correctly on the
device (M2-726). The clock-gate family lost USB after four cases and the
CPU-direct family was not analysed.

## 16.27 P49: CPU-issued command sequences

With the corrected FIFO set-up (M2-730) every required round trip reads back exactly; the qualified
command variants are listed in M2-731..M2-740, and the fastest qualified
data cycle is 28.24 ns (M2-731..M2-741).


The gate family returns baseline and gates0..3 with successful cleanup, then
FREE remains queued while USB status polling works (M2-742). Direct worker
dispatch handles injected timer starvation in the native model, and the host
now retains scheduler diagnostics on queued timeouts; the physical cause
remains unresolved (M2-743).

## 16.28 P50: clock gates and VIC0 line 5

Baseline and gates0..108 return END with exact clock restoration and successful
reset/cleanup (110 cases; 67 gates initially enabled, 42 already disabled).
VIC0 line5 is high before and after restoration, but no during-pulse samples
exist, so these results exclude no gate as its source (M2-744). The queued
FREE stall after gate3 does not recur. USB is lost submitting the next RUN;
its acceptance and execution are unknown. Gate109 is the intended next case,
and no gate109..159 result exists (M2-745).


## 16.29 P51: gate3 request and entry history

Four cases complete through gate2, then USB is lost submitting the gate3 RUN.
The journal proves that manifest is byte-identical to p50's successful gate3
request; the lost RUN has no observed acceptance or execution (M2-746).
P50 reports EXECFIRMWARE entry and p51 does not. Image identity and CPU state
were not captured, so this is an entry-history association, not a cause
(M2-747). The new checkpoint build records those missing distinctions
(M2-748).
