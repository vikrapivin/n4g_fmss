# Chapter 12 — CPU interface and software ABI

<a id="cpu-interface-and-software-abi"></a>

Summary: The parameter words the original CPU driver passes to READ and READID,
the lifetime of parameters across the reset, cache coherency and pause
handling, the separate layers of result codes, the reset-verification and
recovery records of the probe, and controller access by the CPU without a
program.

## 12.1 Parameter words (READ / READID)

Hardware registers provide the execution interface; parameter RAM contents are
defined by each microcode program. The table is the original bootloader
READ/READID ABI from `bl_NandReadOnly` read setup `mem:0x4d9c`, mode setup
`mem:0x4fcc` and ID setup `mem:0x5ab0`
([chapter 14, section 14.3](14-evidence.md#original-code-observations)). These are not
hardwired meanings of the parameter words.

| Offset | Original meaning |
|---|---|
| `d08` | READID destination address |
| `d0c` | row-list pointer; READID instead uses the CE count |
| `d10` | CE-mask-list pointer |
| `d14` | per-sector status output pointer, or zero |
| `d18` | page count |
| `d1c` | metadata output pointer, 12-byte CPU record per page |
| `d20` | DMA pointer list, one pointer per 2 KiB chunk |
| `d24` | ECC histogram pointer, or zero |
| `d28` | chunks per page = sectors per page / 4 |
| `d2c` / `d30` | subsequent-sector / first-sector ECC command |
| `d34` / `d38` | first-sector metadata+parity / parity-only transfer length minus one |
| `d3c` / `d40` | program scratch: page summary / chunk counter |
| `d4c` | NAND timing word ([chapter 9, section 9.11](09-nand-controller.md#911-bus-timing)) |
| `d7c` | program scratch: last ECC result |

The eight GPRs can also be CPU-preloaded: the boot-page programs use a
register-based ABI instead. Neither a NAND command byte nor a USB monitor
command is an FMSS opcode.

## 12.2 Parameter lifetime and reset

The original program advances several pointers in place, so parameters are
rewritten for each invocation. `C00` = `0x80000`, alone or with STOP or abort,
clears all 32 PRAM words and all eight GPRs, while STOP and abort alone keep
them (M2-54, M2-422); no `C00` sequence changes FMC `0x00c..0x05c` or
`ECC_CTRL`, which the per-case reset zeroes (M2-123, M2-422): `FMCTRL1` bit 31
zeroes the FMC words and `0x81c` = 1 zeroes `ECC_CTRL` (M2-645). A reset after
setup therefore destroys the inputs: parameters are written after the reset
(M2-55). The emCORE adapter resets before writing parameters (M2-50). A start
keeps the CPU-written GPRs and the CPU-written `C3C`, `C48` and `C54` (M2-154,
M2-157), and the first start word after the ordinary reset runs the program
(M2-151).

At power-on the window equals the window after the ordinary reset except
`C00` (`0x7f60` at power-on, the last start word after a reset) and `C04` (0
at power-on, the program address after a reset); the GPRs and PRAM read 0,
`C14` reads `0x7f`, `FMC_DMA_CTRL` 7 and `FMC_STATUS` `0xc3000` (M2-430). The
first program after power-on runs on its first start, so no priming start is
needed after a power cycle either (M2-431).

A CPU read of `ECC_RESULT` (`0x80c`, also through its copy at `0xa0c`)
advances the result ring's read position like a sequencer read (M2-91): a CPU
snapshot of the ECC window between decodes shifts the slot from which the
program's own `0x80c` reads start; writing `0x81c` = 1 resets the ring.

## 12.3 Completion and coherency

Program and input memory are cleaned from the CPU cache before the sequencer or
DMA reads them; DMA-written output is invalidated before the CPU consumes it.
The actual allocation and bus address are used: an arbitrary uncached alias is
not interchangeable with the validated address path. END, fault and pause are
separate completions.

The original pause handler skips the pausing instruction with PC += 8,
acknowledges the pause and resumes. Both measured `pause` forms give the pause
signature (M2-14), and an explicit CPU resume reaches the following marker and
END (M2-25). The PC write and the start word continue the program with or
without the acknowledgement; without the PC write it stays paused (M2-188).
With the PC write followed by two start words the program continued in three
of four runs and stayed paused in one (M2-512). A
CPU PC write outside a pause moves a starting or running program to the
written offset (M2-278); see
[chapter 8](08-sequencer-control-registers.md#pause-12-and-recovery).

A CPU write of `C60` = 1 during a run holds the engine (`C08` = 1) beyond STOP
and the reset handshake (M2-121); `C60` = 0 releases it, and an engine still
running then finishes normally (M2-126).

## 12.4 Result layers

r6 is visible at `c30`. A microcode result, an ECC per-sector result, a FIL
return code and an emCORE worker return are separate layers. Worker
`0xfffffffe` is the probe's bounded setup/start/cleanup failure, not a hardware
opcode or NAND status; it can accompany a valid preceding sample or an entirely
unexecuted candidate.

A successful worker return, setup and cleanup do not establish CPU-visible DMA
data: programs that leave a DMA completion unacknowledged reach END with every
wait passing and a cut payload (M2-71, M2-78). Results therefore carry full
payload comparisons and pre-cleanup observations. CPU writes scheduled within a
case run on the device without a per-case USB exchange. Compressed host
summaries change storage only, not firmware execution or the native result
layout.

## 12.5 Reset verification and recovery

Issuing STOP does not prove a reset. A timeout-disabled `evjmp` started with
`C00` = 1 stays in `C08` = `b` (PC `0x20`, IRQ 0) after STOP, and later
candidates fail the initial reset (M2-57, M2-59). For zero-field `evjmp` the
measured sequence is `b` → 3 after abort → 0 after a forced sequencer reset,
followed by a successful full reset and independent execution (M2-62, M2-63).
Of the other `C00` orders on a blocked state `b`, STOP leaves `b` (`C00`
`0x40000`), abort then STOP returns to `b`, and every order containing
`C00` = `0x80000` reaches `C08` = 0 (M2-392).
An intermediate `C08` = 0 or 3 does not mark completion: when recovering a
captured state-`b` failure the forced reset is still issued (M2-60). See
[chapter 8](08-sequencer-control-registers.md#sequencer-control-faults-and-cpu-handoff).

```mermaid
flowchart LR
  B[C08 = b after STOP] -->|abort| T[C08 = 3]
  T -->|forced reset| Z[C08 = 0]
  Z -->|full reset| R[next END-only preflight]
```

With this recovery, a full execution of all families (31,001 sampled records) has zero setup,
reset, cleanup or USB failures (M2-64). Its recorded recoveries are forced state-`b` recovery 16 times, status
release 1342 times, pause resume 197 times and one pause resume plus abort
(M2-75).

## 12.6 Recovery diagnostic records

The probe retains a reset failure and attempts bounded abort and forced
sequencer reset before checking the full reset and the next END-only
preflight. Recovery method bits are software diagnostics, not hardware status
bits:

| Field | Meaning |
|---|---|
| firmware capability bit 27 | reset while held is supported |
| method low byte `0x08` / `0x10` | abort / forced reset attempted |
| method `0x20` | `C60` hold recovery: `C60` = 0, wait for `C08` 0/3, then the complete ordinary reset repeated |
| method `0x40` (with `0x20`) | the `C60` recovery needed the abort and sequencer-reset fallback (`C00` = 2, then `0x80000`) |
| method bit 24 | validity: status bytes 15:8 and 23:16 are present |
| status byte 15:8 | `recovery_status.after_abort` (host name) |
| status byte 23:16 | `recovery_status.after_forced_reset` (host name) |

Records with method 8 alone contain no intermediate status measurement.

## 12.7 Controller access by the CPU alone

The CPU can drive the NAND controller and the ECC engine without a sequencer
program: the programs' register sequences written by the CPU, with each `wait`
replaced by a bounded poll of the flag the wait tests, give the same results
as the sequencer path (M2-435). Raw page reads by DMA and through `FMC_FIFO`
return the bytes of the sequencer raw read; CPU-kicked decodes in the boot-ROM
format (three sectors of 26 parity and 512 data bytes, BCH16, `0x808` = `0x80`)
and in the main BCH8 format (metadata, parity, data) return the raw data and
metadata; a CPU erase leaves the pages all `0xff`, and a CPU program with host
BCH8 parity reads back exactly by the raw paths, a CPU decode and the
sequencer READ (M2-435).

| step | CPU access |
|---|---|
| FIFO read of 32 bytes | select FIFO with `FMC_CTRL0` bit 24 clear; `FMC_DNUM` = 31, `FMC_DATA_SEL` = 1, `FMCTRL1` = `0x1c2`, poll `FMC_STATUS` bit 3, then `FMC_DATA_SEL` = `0x100`, `FMCTRL1` = `0x340` and eight reads of `0x080` (M2-436, M2-728) |
| encode | the kicked encode gives the host BCH8 parity in slots 0 and 1 (metadata command `0xa02`) and the host BCH16 parity (`0xa06`) (M2-436) |

The reads of `0x080` return the transferred page data only in this sequence;
sequencer reads of `0x080` from DMA-filled buffers return 0 (M2-207).

Reset and ready-status success do not establish payload correctness. A generic
CPU sequence that kept DMA mode enabled for FIFO access completed its status
polls and resets while all 12 round-trip controls failed. Select the requested
data path for every read, preserving CE and timing fields; a sequence can
switch DMA → FIFO → DMA without another controller reset. The corrected
implementation is validated in the compiled-kernel model (M2-727, M2-728,
M2-730); the earlier device-qualified operations below remain separate evidence.

The CPU path covers the whole page life cycle in both formats, with the
engine or software BCH, and agrees with the sequencer path in every
comparison:

| operation | result | evidence |
|---|---|---|
| raw read, DMA and FIFO | boot-ROM reserved-region pages (CE0 pages 0, 1, 2 and 100, three BCH16 sectors each) and main-format BCH8 pages (rows 128, 129, 256000, 383872 and a sequencer-written page) equal the sequencer raw read | M2-711 |
| decode, engine and software BCH | data, metadata and sector status equal those of the sequencer readers in both formats | M2-711 |
| erase and program | engine-encoded, software-encoded and raw images in the main and boot formats read back exactly through raw DMA/FIFO, the engine decoder, the software decoder and the sequencer readers; the CPU-kicked encode equals host BCH8 and BCH16 | M2-712 |
| injected errors | with 1, 2, 4, 7 and 8 data errors, 3 errors in metadata/parity and one 9-error sector per page, engine and software decode return the same data, metadata, correction counts and location sets; the 9-error sector is uncorrectable in both and left as stored | M2-713 |
| error locations | the `0x820` fields list a byte's higher bit first ([chapter 10, section 10.8](10-ecc-engine.md#108-error-locations-0x820)) | M2-714 |
| closing state | every job that touched the controller (89) leaves `C08` = 0 and `FMC_CTRL0` = 0 after its closing reset | M2-716 |

Job times, each including the reset before and after it (M2-715):

| job | time |
|---|---|
| raw page read, DMA / FIFO | 586 us / 795 us |
| engine-decoded page, main / boot format | 648 us / 270 us |
| erase | 2274 us |
| program | 899..1831 us |
| software BCH8 decode of a main page | 1956 us clean, 18.4 ms with 33 corrections |
| software BCH16 decode of a boot page | 1370 us |

Beyond the drivers' own sequences, CPU-issued command sequences read back
exactly for the random column read and write, partial-page programming,
overwriting a programmed page without an erase, out-of-order page programming
within a block, copyback inside a block, and the two-plane program and erase
(M2-731..M2-740). In the FIFO input path the CPU must clear `FMC_CTRL0`
bit 24 as the drivers do; a sequence that left it set lost earlier DMA buffer
content (M2-728, M2-730). Reads and programs remain exact down to a data
cycle of 28.24 ns (timing fields 0/2) on the measured unit, a qualification
specific to that unit and its operating conditions (M2-741).
