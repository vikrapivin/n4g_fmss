# Chapter 3 — The register window as the ARM CPU sees it


Summary: the offsets and names of the register window, the memory map with the
evidence for each word, the rules that apply only to CPU accesses, the
CPU-visible interrupt lines, how the original CPU driver starts and finishes a
run, and the instruction and wait behaviour visible through the window.

## 3.1 Offsets and names

The register window is at `mem:0x38a00000`; addresses in this book are offsets
from that base unless stated. The original CPU driver is observed in
`bl_NandReadOnly` (reset `_resetFMSS` at `mem:0x502c`, start `_startFMSS` at
`mem:0x51b8`, interrupt clear at `mem:0x4cbc`, interrupt wait at
`mem:0x51f4`, pause handling at `mem:0x4d08`), in `osos` (reset at
`mem:0x08069ba4`, start at `mem:0x080daf1c`, read setup at `mem:0x0808d050`)
and in `fw_restore` (FIFO reads at `mem:0x0802a640`)
([chapter 14, section 14.3](14-evidence.md#original-code-observations)).
Register names follow [chapter 1, section 1.2](01-overview.md#12-numbering-names-and-notation).


## 3.2 Memory map: the register window

| Offset | Role | Evidence and limits |
|---|---|---|
| `000` | `FMC_CTRL0`: controller enable bit 0, CE bits 8:1, DMA bit 24, cycle timing bits 14:12 and 18:16 | bit 0 is required for transfers; cycle = (bits 14:12 + bits 18:16 + 2) × 7.06 ns per byte; bits 18:16 ≤ 1 shift read data by one byte; bits 30:28, 9, 10, 25, 26 are stored with no measured effect; bits 15, 19..23, 27, 31 read 0; storage mask `0x77077fff` (M2-127, M2-131, M2-135, M2-139, M2-191); bits 30:28 do not move the command, latch or read completions (M2-407); a DMA programmed with bit 24 clear is held and starts when bit 24 is set, with the DMA address of that moment (M2-184, M2-338) |
| `004` | `FMCTRL1`: command, address, data, DMA and reset trigger | observed command words; written alone, bit 0 starts an address operation, bit 1 a data read, bits 8 and 9 a DMA, and the other 28 bits start no operation (M2-411; [chapter 9](09-nand-controller.md)); bit 31 zeroes `00c..05c` and sets `038` to 7, `0x800` changes none of them (M2-645) |
| `008` | NAND command byte | observed |
| `00c`, `010`, `02c` | address bytes and address-cycle count minus one | observed; `00c`/`010` store 32 bits, `02c` 3 bits (M2-83) |
| `014` | data/parity buffer selection | inferred from transfer sequences; stores 16 bits, its bits 7:0 are `058` and bits 15:8 read at `05c` (M2-83, M2-643); a DMA in writes data buffer 0 only with 1 and parity buffer 0 only with `0x10`, no buffer with 0 or the other tested values (M2-668); a DMA in flight follows a rewrite of it from a word boundary on (M2-603, M2-672) |
| `018` | written 1 before data reads by the original program | stores 4 bits (M2-83, M2-191); no effect on page-read data, ECC results or a buffer loopback (M2-104, M2-405) |
| `01c` | ECC slot index | stores bit 0 (M2-83, M2-417); an encode uses the slot that bit selects, a second kick replaces the first, and a change during an encode gives a parity of neither slot (M2-417, M2-418, M2-419) |
| `024` | transfer count | reads 8 after an 8-byte READ ID (M2-336) |
| `028` | storage word | 9 bits; does not count operations, `pld`/`sti` loops or FIFO reads (M2-156, M2-173, M2-199) |
| `030` | NAND transfer byte count minus one | stores 13 bits (M2-83); a DMA moves 512 bytes whatever it holds (M2-189) |
| `034` | DMA bus address | stores 32 bits (M2-83); a DMA uses it aligned down to a multiple of 4 (M2-664); a DMA in flight follows a rewrite of it at the same byte offsets from the next 32-byte boundary on (M2-603, M2-671) |
| `038` | `FMC_DMA_CTRL` | reads 7 after reset and at power-on (M2-123, M2-430); `0x10..0x80` fault (M2-79); `0x40..0x1000` and `0x1fc7` read back unchanged, and any of bits 12:6 set, or bits 5:3 = 2, 4 or 7, makes the next DMA out fault after delivering only its first 32 bytes, while with bits 5:3 = 1 a DMA out writes each group of four words as four copies of its first word; bits 31:13 are not stored (M2-396, M2-669); bits 12:6 count up from 0 during a DMA and return to 0 at its completion (M2-113, M2-117, M2-398); with bits 2:0 = 0..6 a DMA completes more slowly than with 7, and with 2, 4, 5 or 6 it leaves a residual count in bits 12:6 (2 reads back `0x42`, 4 reads `0x84`; M2-79, M2-609) |
| `040` | FIFO/metadata latch command | observed; `0x52` latches the chip ID into `060`/`064` (M2-124) |
| `044` | FMC interrupt routing | bits 0, 1, 2, 11, 12 route the command, address, transfer, DMA and status-latch flags onto VIC0 line 20 and into `CS_IRQ` bit 4 (M2-142, M2-200) |
| `048` | `FMC_STATUS` completion flags | bits 1/2/3/20/23 = command/address/data/DMA/status latch (M2-112); bits 19:12 read `0xc3` (bits 12, 13, 18, 19) after reset and are not cleared by writing 1, `0x41` after a short chip read, `0x3c` (bits 14..17) after a 512-byte read; a data read of 1..128 bytes leaves `0x41`, of 256 or 384 bytes `0x14`, of 511 or 512 bytes `0x3c`, and 32 or 128 `FMC_FIFO` writes leave `0x41` or `0x3c` (M2-642); a DMA in, a status latch and the set-up word restore `0xc3` from `0x41` and from `0x3c` (M2-203, M2-497); writing all ones clears a pending DMA flag and nothing else, writing 0 changes nothing (M2-412); bit 24 never set by a status latch (M2-163); bit 25 set by a DMA into a buffer under its own ECC operation (meaning inferred, M2-80); bit 26 set by an address operation after a latch without the `0x30020` release, not by a refused command (M2-181, M2-610); bit 27 set after reading a programmed sector, clear after an erased one, cleared by writing `0x0ff00ffe` (M2-145); the completion flags are acknowledged by writing the bit |
| `04c` | latched NAND chip status | `0x80` busy, `0xe0` ready, `0x70` with no CE (M2-87, M2-136); ignores writes (M2-412) |
| `058` | transfer word of the original programs: `014` bits 7:0 | stores 8 bits (M2-83); a write replaces `014` bits 7:0 and a later `014` write replaces it (M2-643); it reads 0 after a buffer loopback, whose DMA out leaves `014` = `0x100` (M2-405); CPU writes before the start read back unchanged (M2-599); written before a DMA in that writes `014` = 0 it has no effect (M2-597) |
| `05c` | written `0x200` before the final DMA: `014` bits 15:8 | stores bits 15:8, which are `014` bits 15:8 in place (M2-83, M2-643); no effect on page reads (M2-104); reads `0x100` after a buffer loopback, whose DMA out leaves `014` = `0x100` (M2-405); a CPU write of `0x200` before the start reads back unchanged (M2-599) |
| `060..068` | FIFO/metadata data words | observed; ID repeats after five bytes (M2-124) |
| `080` | CPU FIFO read port | consuming read (observed, `fw_restore` `mem:0x0802a640`); after `FMCTRL1` `0x1c2` and `0x340` CPU reads return the page data in 32-byte transfers (M2-436); sequencer reads return 0 after a READ ID and from DMA-filled buffers (M2-124, M2-207); writes never reach a data buffer: without `0x340` the buffer stays unchanged, and after `0x340` buffer 0 receives RAM from `FMC_DMA_ADDR` + `0x200` on, never a written value (M2-207, M2-341, M2-617) |
| `100..1fc` | block of unknown function | sequencer-writable words only at `1b8..1d8`; the other words keep nothing and read `0x12` from the sequencer; writes change no measured timing (M2-160, M2-210) |
| `200..3fc` | copy of `000..1fc` | offset bit 9 not decoded below `400`; the `100` block appears only at `100` and `300` (M2-155) |
| `400..7fc` | no register set | read 0 (M2-155); storage words read 0 after sequencer or CPU writes, without aliasing the primary words (M2-82); no `FMC_CTRL0` bit enables them (M2-139); commands and DMAs through `+0x400` do nothing (M2-183, M2-185); no tested enable write makes them hold a value (M2-393); a sequencer `ld` costs the same as at `00c` or `60c` (M2-596) |
| `800` | ECC identification word | reads `0x20070618` (M2-116) |
| `804` | `ECC_CTRL`: operation, BCH16 bit 2, metadata length bits 13:8 | storage mask `0x3f04`; no code other than BCH8/BCH16 (M2-98, M2-101, M2-191, M2-197); lengths above 32 encode the data alone (M2-662); `81c` = 1 zeroes it (M2-645) |
| `808`, `814` | no established role | `808` reads `0x700`; storage masks `0x1f81` and `0x2f` (M2-116, M2-191); with `808` bits 12:8 = 0, `810` bit 5 is set after every completion (threshold role inferred, M2-434); `814` values change no BCH8 decode or encode result (M2-660) |
| `80c` | `ECC_RESULT`: 8-slot ring of per-decode results | each `80c` read, by the sequencer or by the CPU, also through the copy at `a0c`, advances the read position; reads of `90c` and `b0c` read 0 and do not (M2-90, M2-91) |
| `810` | `ECC_STATUS` | bit 0 done, bit 1 encode done, bit 2 decode done, bit 3 uncorrectable, bit 5 ≥ 7 corrected bits, bit 6 erased; bits 1, 2, 6 are write-1-to-clear; writing 1 clears only bit 0; bit 4 reads 1, bits 7..12, 16, 24 and 31 never set (M2-88, M2-94, M2-95, M2-97, M2-99, M2-100, M2-420) |
| `818` | corrected-bit running total << 11 \| decode count | M2-89 |
| `81c` | ECC reset | a write with bit 0 set clears `818`, sets `810` to `0x10`, resets the ring positions and zeroes `804`; other bits do nothing (M2-91, M2-96, M2-421, M2-645) |
| `820..` | decode error locations, 13-bit fields, ascending by byte, higher bit first within a byte | M2-128, M2-132, M2-138, M2-714 |
| `840..85c` | encode parity | word order reversed, big-endian words (M2-116); `844` keeps no written bits (M2-191) |
| `900..9fc` | none | read 0 after encodes and decodes; `940..950` drop writes (M2-171, M2-191) |
| `a00..bfc` | copy of `800..9fc` | offset bit 9 not decoded in `800..bff` (M2-155) |
| `c00` | `FMSS_CTRL`: start word, STOP, abort, clearing step | [chapter 2](02-architecture.md#21-how-the-sequencer-is-started-and-stopped), [chapter 8](08-sequencer-control-registers.md) |
| `c04` | `FMSS_PROG`: program bus address | observed; cleared by the clearing step (M2-54) |
| `c08` | `CS_STATUS`: state code | 0, 3, 4, `a`, `b` read by the CPU in the states of chapter 2; 1, 2 or 9 while a program runs, varying between identical runs; 5 read by the sequencer (M2-70, M2-384) |
| `c0c` | `CS_IRQ`: bit 0 END, bit 1 `sig`, bit 2 fault, bit 3 pause, bit 4 routed FMC flag | `sig` sets bit 1 and continues; bit 1 is write-1-to-clear (M2-34, M2-65, M2-370); bit 4 sticky (M2-200) |
| `c10` | initialization word | original prologue writes 1; stores `0xdf`; non-zero makes an END set VIC1 raw bit 22 (M2-182, M2-191) |
| `c14` | counter/timer | `0x7f` at power-on and after reset (M2-281, M2-430); start-word bits 15:8 after a wait that passes before the first timeout step, counting down by one per step while a wait stalls, `0x7f` after a wait that ends after one or more steps, also when the CPU releases it (M2-172, M2-177); 0 after a completed `tmwait` (M2-27); ignores writes (M2-176) |
| `c18..c34` | r0..r7; r6 at `c30` is `SEQ_RESULT` | device (M2-02) |
| `c38` | `FMSS_PC` | program-relative byte offset; a CPU or program write jumps (M2-162, M2-278) |
| `c3c` | fault information | faulting offset + 8 in the handler, 0 after the run; 32 bits (M2-154, M2-179) |
| `c40`, `c44` | instruction word latches | device; cleared by the clearing step (M2-54) |
| `c48`, `c54` | unassigned storage | 32 bits / mask `0x83f`, kept by a start (M2-154); CPU writes release no `wait` (M2-143); any of `C48` bits 1..7 makes a `tmwait` fault at the wait timeout instead of ending, bit 0 ends a count-1 `tmwait` one instant early, bits 8..31 change nothing (M2-179, M2-627) |
| `c4c` | fault vector | M2-11 |
| `c50` | fault-handler state and completion bits | bits 0..5 mirror pending events 0..5 (M2-18, M2-28, M2-112, M2-209) |
| `c58` | fault policy | reads back AND `0xd6`; bit 2 vector/continue; bits 1, 4, 6 and 7 stored without effect on the timeout path; no value diverts an END (M2-11, M2-70, M2-148, M2-149, M2-612) |
| `c5c` | vector-associated state | 1 after a timeout vector, 0 after continuation ([chapter 8](08-sequencer-control-registers.md)) |
| `c60`, `c64` | buffer reset request / completion | observed handshake (M2-20); `c60` = 1 while running holds the engine (M2-121) |
| `c68` | no established role | stores 3 bits (mask 7) (M2-191) |
| `c6c` | buffer configuration | 0 normal path, `0x10` alternate path (observed) |
| `d00..d7c` | parameter RAM, 32 words | CPU read/write and sequencer reads (M2-03); cleared by the clearing step (M2-54) |
| `d80..dfc` | none | reads 0; CPU and program writes do not persist; no alias into `d00..d7c` (M2-04, M2-147) |
| `e00..ffc` | none | offset bit 9 is decoded in `c00..fff`: no word of `c00..dff` appears at +`200` (M2-155) |

The bit maps of `FMC_STATUS`, `ECC_STATUS` and `ECC_CTRL` are in chapters 9 and
10; the control block is in chapter 8.


## 3.3 CPU-only rules

- **Consuming reads:** `080` and the FIFO data words `060/064/068` are FIFO
  ports; a CPU read consumes data (observed). Reading them is part of an
  operation, not an observation of state.
- **`ECC_RESULT` reads:** a CPU read of `80c`, also through its copy at `a0c`,
  advances the ring's read position like a sequencer read, while reads of
  `810`/`818` and writes of `80c` do not (M2-91). A CPU snapshot of the window
  that includes `80c` or `a0c` therefore moves the slot from which the next
  program's `80c` reads start.
- **`C60` while running:** a CPU write `C60` = 1 during a run holds the engine
  at `C08` = 1 beyond STOP and the buffer handshake; `C60` = 0 releases it
  (M2-121, M2-126).
- **No CPU-written event:** CPU writes to `C0C`, `C10`, `C48`, `C54` and
  `C60..C6C` release none of `wait` 6..10 and 12..15 (M2-120, M2-125, M2-143),
  nor do writes of `C40`, `C44`, `C50`, `C14`, `C5C` or `C3C` into a waiting
  `wait` 6..10 or 13..15 (M2-386).
- **`C58`:** CPU and sequencer writes both read back masked by `0xd6` (M2-70).
- **`C14`:** CPU writes (`0x55`) and program writes are ignored (M2-176;
  [chapter 8, section 8.7](08-sequencer-control-registers.md#87-fmss_c14-0xc14-countertimer)).
- **`C38`:** a CPU write of the PC takes effect outside a pause: before the
  start the program starts at the written offset, while a program runs it jumps
  there (M2-278).
- **`C3C`, `C48`, `C54`:** a start keeps the values the CPU wrote (M2-154).
- **Storage masks:** CPU and sequencer writes give the same masks over the
  window (M2-191).
- **`FMC_FIFO` (`080`):** the CPU reads page data through it after `FMCTRL1`
  `0x1c2` (data read into the buffer) and `0x340`, 32 bytes per transfer
  (M2-436); sequencer writes to it reach no data buffer, without `0x340` and
  after it (M2-207, M2-617).
- **CPU-driven NAND:** the programs' register sequences written by the CPU,
  with each `wait` replaced by a poll of its flag, read, erase and program
  pages and run decodes and encodes with the same results as the sequencer
  (M2-435, M2-436; [chapter 12](12-cpu-interface.md)).
- **`C5C`:** its M2 role is not established; the M1 cleanup write to it has no
  M2 counterpart in the original driver.
- **GPRs:** values the CPU writes to r0..r7 before a start survive into the
  program (M2-02, M2-157).
- **`C08`:** the CPU reads the state codes of chapter 2; the sequencer reads 5
  (M2-70).
- **Reset and power-on values:** the complete reset leaves FMC `00c..05c`,
  `ECC_CTRL` and the parameter RAM 0 and `FMC_DMA_CTRL` 7 (M2-123); the FMC
  words and `FMC_DMA_CTRL` come from `FMCTRL1` bit 31 and `ECC_CTRL` from
  `0x81c` = 1, written by the CPU or the sequencer alike (M2-645). The
  power-on window, and the window left by the kernel boot or by the previous
  cleanup, differ from the one after the ordinary reset only in `C00` (`0x7f60`
  instead of the last start word) and `C04` (0 instead of the program address)
  (M2-281, M2-430). Power-on values include `004` `0x40000000`, `020`
  `0x7f167f`, `038` 7, `048` `0xc3000`, `1cc` `0x8060`, `1d0` `0x3010d0`, `800`
  `0x20070618`, `808` `0x700`, `810` `0x10`, `C08` 0, `C14` `0x7f`, `C50`
  `0xff1800`; the GPRs, the parameter RAM and the other FMC and ECC storage
  words read 0 (M2-430). The first program after power-on runs on its first
  start (M2-431).

## 3.4 CPU-visible interrupt lines

- The original CPU driver polls `CS_IRQ & 0xd` for END, fault and pause; it
  does not wait on a VIC line (observed, `mem:0x51f4`).
- NAND controller flags drive VIC0 line 20 when routed in FMC `044` (bits 0,
  1, 2, 11, 12: command, address, transfer, DMA, status latch); the line
  follows a routed flag while it is pending, and `CS_IRQ` bit 4 records every
  routed flag of the run (M2-142, M2-200). An END with a routed flag still
  pending leaves USB working (M2-208).
- An END raises VIC1 raw bit 22 while `C10` is non-zero, a fault-handler END
  included; after an END with `C10` = 0 and after a run that stops without an
  END (held, paused, looping or faulted without a handler END) it is clear, and
  it is clear while a program runs (M2-182).
- `sig` raises no VIC0 or VIC1 line beyond the run's baseline (M2-370).
- `waitirq` conditions are VIC0 hardware lines. Line 5 is always asserted, line 8
  is the timer interrupt, line 16 is DMAC0 and line 20 is the FMC interrupt
  (M2-142, M2-204); condition 19 passes in the DMA and ECC contexts (M2-167). Conditions 32..255 never pass, VIC1 included (M2-74,
  M2-140, M2-211).
- A VIC software interrupt is visible in the raw VIC status but releases
  `waitirq` on none of 59 lines (M2-140). Inferred: `waitirq` samples the
  peripheral request lines before the VIC's software OR.
- No ECC interrupt is found: after encodes and decodes no VIC line rises for
  any tested enable candidate (M2-212, M2-217). The source behind line 5 and
  the ECC interrupt are open
  ([chapter 17, section 17.3](17-open-questions.md#173-interrupts)).

CPU interrupt numbers are distinct from `wait` event numbers.


## 3.5 Starting and finishing a run

The original CPU driver (`bl_NandReadOnly`; observed):

1. **Reset** (`mem:0x502c`): `C00` = 8 (STOP), poll `C08` = 0; if `C08` = 4,
   `C00` = 2 (abort) and poll for 0 or 3. Unless `c6c` = `0x10`: `c60` = 1,
   wait for `c64` = 1, `c60` = 0. Then `C00` = `0x80000`, `004` = `0x800`,
   poll `004` bit 30, `004` = `0x80000000`, `81c` = 1 (ECC reset). This is the
   M2 sequence; M1's `C00` = `0x30000` reset does not occur.
2. **Program:** `c04` = the program's bus address; clear `CS_IRQ` (`mem:0x4cbc`:
   write `0xd`, poll until bits 0, 2, 3 are clear).
3. **Start** (`mem:0x51b8`): `CS_IRQ` = `0xff`; `c6c` = 0, `c60` = 1, wait for
   `c64` ≠ 0, `c60` = 0 (or `c6c` = `0x10` on the alternate path); `c00` =
   `0xffb5` (bootloader) or `0xfff5` (restore OS).
4. **Wait** (`mem:0x51f4`): poll `CS_IRQ` bits 0, 2, 3. Bit 3 (pause): PC += 8,
   acknowledge bit 3, set `C00` bit 0 (`mem:0x4d08`). Bit 0 alone: success.
   Bit 2: failed run, followed by the reset of step 1.

The parameters are written after the reset, because the clearing step erases
the parameter RAM (M2-54, M2-55). The recovery from states the original reset
does not handle (pause inherited from an earlier run, blocked `evjmp`, active
status latch, `C60` hold) is described in
[chapter 2, section 2.1](02-architecture.md#21-how-the-sequencer-is-started-and-stopped).




## 3.6 Instruction and wait behaviour seen through the window

Instructions are eight bytes, little-endian: `arg:u16, reg:u8, opcode:u8,
immediate:u32` (M2-01; [chapter 5](05-instruction-encoding.md)). The `wait`
events used by the original programs, with their final measured sources
([chapter 7](07-events-and-waits.md)):

| `wait` event | source |
|---|---|
| 0 | command completion, `FMC_STATUS` bit 1 (observed, M2-19) |
| 1 | address completion, bit 2 |
| 2 | DMA completion, bit 20 |
| 3 | data transfer completion, bit 3 |
| 4 | ECC completion: `810` bit 1 after an encode, bit 2 after a decode, write-1-to-clear (M2-97, M2-99) |
| 5 | status latch, `FMC_STATUS` bit 23 when the chip reports ready (M2-111, M2-130, M2-193); a latch written one or two instructions after an unwaited `70h` sets bit 3 instead and the wait faults (M2-593) |
| 11 | passes at once in every measured state and operand form, also on every iteration through NAND busy periods (M2-10, M2-107, M2-387) |
| 12 | masked NAND status: passes iff status & arg & mask ≠ 0 (M2-134); after a latch on a busy chip it releases only at ready (M2-137) |
| 6..10, 13..15 | no source found among NAND, ECC, DMA, status, CPU-write and VIC soft-line stimuli (M2-107, M2-111, M2-143, M2-206, M2-386) |
| 16..255 | never pass (M2-159) |

`wait` 0, 1, 2 do not consume their flags; acknowledging blocks the next wait
and a new trigger re-arms it (M2-66). After one status latch the controller
keeps polling the chip status until ready; without a latch, or with
`wait 12` arg = 0, the wait never passes (M2-130). A passing `wait` 12 with
mask `0x40` also passes on the no-CE echo `0x70` (M2-87, M2-134); it is not by itself
evidence of a valid chip status byte.

Measured controls of individual instructions:

- `pause` (`12`): two forms stop with `CS_IRQ` = 8, `C08` = `a` and PC at the
  instruction; CPU resume continues to the following instruction (M2-14,
  M2-25).
- `waitc` (`08`): with R = 0..7 the following marker and END execute (M2-17,
  M2-625); with R = 8..`0xff` it holds at its own offset until the timeout
  fault of an unmet wait, also with a pending flag (M2-377, M2-625).
- Shifts by 31: SHL of 1 gives `0x80000000`, SHR gives 1 (M2-07); every
  count ≥ 32 gives 0 (M2-146).
- `pld` (`18`) of `0xc30` reads r6 (M2-08); `pld`/`pst` use offset bits 11:0
  and reach r0..r7 through `0xc18..0xc34` (M2-175).
- `tmwait` (`15`): with count = start-word bits 15:8 ≥ 1 it ends at the
  (count + 1)-th timeout instant after its entry, 492.747 us × count + 450 us
  after the start when entered at the start (126.1 ms for `0xff`), whatever
  its fields, with `C14` = 0; with count 0 it takes the timeout fault (M2-27,
  M2-373, M2-584); consecutive `tmwait`s each wait a full period (M2-150);
  `C48` bits 7:0 change its end (M2-627).
- Undefined opcodes `1a..fe` act as an unmet wait, also with a pending flag
  (M2-282, M2-583).
- `end` with non-zero R, arg or imm stops like the zero form (M2-164).
- `jr` jumps to r[arg] iff r[R] ≠ 0 (M2-110); a target selector with any of
  bits 8..15 set takes r0 (M2-673).

Opcodes `09`, `0f`, `10`, `16` and the extended register operands are
described with their measured forms in [chapter 6](06-opcode-reference.md);
the M1 meaning of a retained mnemonic is not assumed beyond those forms.
