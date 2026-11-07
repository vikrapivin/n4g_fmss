# Chapter 1 — Overview and terms


Summary: what the M2 FMSS engine of the S5L8720 (iPod nano 4G) is, the evidence
classes and numbering used in this book, the terms, the three hardware blocks
(sequencer, NAND controller (FMC), ECC engine) and the named parts of the data
path.

The M2 FMSS engine is the NAND hardware of the S5L8720: a small sequencer
coprocessor that runs programs of 8-byte instructions, the NAND controller (FMC)
it drives, and the ECC engine. All three share one register window at
`mem:0x38a00000`.


## 1.1 Evidence classes

The statements in the chapters rest on three sources:

- static analysis of the original programs and of the original CPU driver in
  the bootloader (`bl_NandReadOnly`), the restore OS (`fw_restore`) and the
  operating system (`osos`);
- device tests: small programs run on the sequencer of one nano 4G unit
  ([chapter 13](13-experiments.md), outcomes in [chapter 14](14-evidence.md));
- the M2 simulator, which runs the sequencer programs and the actual ARM driver
  and probe code.

Each statement carries one of the following evidence classes:

- **observed** (source-observed): seen in the original programs or in the
  original CPU driver, with image and address.
- **device** (device-observed): measured on the unit, with execution and
  initial state qualified for the case.
- **inferred**: an interpretation of observed use or of device results.
- **3p**: a third-party driver, reported by its authors as tested.
- **open**: not settled by the available M2 evidence.
- **emulated**: behaviour of the named model and build; it makes no claim about
  unobserved silicon behaviour or calibrated hardware timing.

The finding list ([chapter 18](18-ledger.md)) abbreviates these as D (device),
S (original-code use), I (interpretation) and M (model validation). An M1
measurement does not become an M2 device result because its name is retained:
M1 results are hypotheses on M2 until an M2 measurement confirms them (for
example, the 78 M1-comparison expectations confirmed only for their tested
forms and states, M2-34, M2-65).


## 1.2 Numbering, names and notation

Chapters are numbered 1 to 19 and headings inside chapter N are numbered N.1,
N.2, N.2.1 and so on, as in the M1 book. The entries of chapter 6 (opcode
reference) are the exception: each heading is the hexadecimal opcode and its
lowercase mnemonic, without a section number. A section number identifies the
M2 organisation; it does not imply that the section has the same contents as
the M1 section with the same number.

Established mnemonics are retained for matching operations: `end`, `wr`, `st`,
`ldi`, `ld`, `movi`, `mov`, `wait`, `nop`, `and`, `or`, `add`, `sub`, `bnz`,
`sti`, `pause`, `shl`, `shr`, `bz`, `pld` and `pst`. `wait2`, `jr`, `sig`,
`tmwait`, `evjmp` and `nop2` are the names of their M1 comparison candidates;
unresolved M2 behaviour is stated separately. A mnemonic is not a claim that
every operand, timing or side effect is identical across generations.

Conventions used throughout:

- Opcode values are hexadecimal (`12` means `0x12`), matching the M1 tables.
  Event numbers, register indices and bit numbers are decimal unless prefixed.
- Bits are written "bit N" or "bits H:L", high bit first. Register offsets and
  values are hexadecimal. `C58`, `C4C` and `C14` abbreviate the window offsets
  `0xc58`, `0xc4c` and `0xc14`; they are not absolute CPU addresses.
- `FMSS_CTRL`, `FMSS_PROG`, `CS_STATUS`, `CS_IRQ`, `SEQ_RESULT` and `FMSS_PC`
  name the corresponding control registers ([chapter 8](08-sequencer-control-registers.md)).
- Instruction fields are `R`, `arg` and `imm`. `rR` names register R, `rA` the
  register selected by arg, `[REG]` a window word and `#imm` an immediate.
  Branch targets and the PC are byte offsets from the program start.
- A **`wait` event N** and a **`wait2` condition N** belong to separate
  selector spaces; the same N in both names unrelated things.
- Times are in us and ms; "about" or "≈" marks a measured approximation. Raw
  capture values keep their original notation.
- "Original firmware", "original programs" and "original CPU driver" name
  Apple's firmware, its FMSS programs and its ARM NAND driver. FMSS is the
  firmware's own name; an expansion of the acronym is not established.
- Source locations distinguish `file:<image>+0xOFF` from `mem:0xADDR`.
- M2 findings are cited by their stable `M2-nn` IDs from
  [chapter 18](18-ledger.md); M1 findings by `M1-Fnnn`. Experiment outcomes are
  in [chapter 14](14-evidence.md).


## 1.3 Terms

| term | meaning | source of the name |
|---|---|---|
| **NAND controller (FMC)** | the SoC's flash memory controller at `0x38a00000`. It produces command, address and data cycles on the NAND bus and contains the data buffers and the DMA ([chapter 9](09-nand-controller.md)). `0x400..0x7fc` reads 0 (M2-155) and holds no second register set: writes are not stored (M2-82), no `FMC_CTRL0` bit enables it (M2-139), and commands and DMAs through it do nothing (M2-183, M2-185) | register names of the SoC family (`FMCTRL0`, `FMCTRL1`, ...) |
| **FMSS engine** | the whole NAND hardware described here: sequencer, NAND controller and ECC engine (section 1.4) | the original firmware's own name |
| **sequencer** | the coprocessor inside the NAND controller that runs programs of 8-byte instructions ([chapter 5](05-instruction-encoding.md)) | name used in these documents |
| **M1 engine** | the engine of the nano 3G (S5L8702), described in the M1 book | name used in these documents |
| **M2 engine** | the engine of the nano 4G (S5L8720) described here; the nano 5G firmware carries the same programs | the 4G/5G firmware |
| **ECC engine** | error-correcting code hardware at offset `0x800` that generates BCH8/BCH16 parity for 512-byte sectors and corrects them ([chapter 10](10-ecc-engine.md)) | name used in these documents |
| **program** | M2 microcode, not ARM code | name used in these documents |
| **window offset** | a register address relative to `0x38a00000` | name used in these documents |
| **CE** | chip enable: the signal that selects one NAND chip (`FMC_CTRL0` bits 8:1) | NAND standard |
| **CS_IRQ** | the sequencer's flag word at `0xc0c`: bit 0 END, bit 1 `sig` (M2-34, M2-65), bit 2 fault, bit 3 pause, bit 4 routed FMC flag (M2-200) | the original firmware calls it `FC_CS_IRQ`; "CS" is not expanded |
| **CS_STATUS (`C08`)** | the sequencer's state code at `0xc08` as the CPU reads it (0 idle, 4 waiting, `a` paused, 1, 2 or 9 while a program runs, others in [chapter 2](02-architecture.md#21-how-the-sequencer-is-started-and-stopped)) | name used in these documents |
| **event** | one of the status flags that `wait` waits for, selected by a number: "`wait` event N" ([chapter 7](07-events-and-waits.md)) | name used in these documents |
| **condition** | the number N of `wait2`: VIC0 hardware interrupt line N ([chapter 3, section 3.4](03-cpu-memory-map.md#34-cpu-visible-interrupt-lines); M2-140) | name used in these documents |
| **VIC0, VIC1** | the CPU's two vectored interrupt controllers | SoC register names |
| **parameter RAM** | 32 words at `0xd00..0xd7c` that the CPU and the program both access; they hold a program's inputs and outputs ([chapter 4, section 4.4](04-sequencer-memory-map.md#44-parameter-ram); M2-03) | name used in these documents |
| **sector** | the 512-byte ECC and data unit of the original programs | name used in these documents |
| **slot** | one of the two ECC/buffer selections the original programs alternate (section 1.5.4) | name used in these documents |
| **chunk** | the 2 KiB DMA grouping of the original driver (section 1.5.5) | name used in these documents |
| **page** | the NAND transaction unit | NAND standard |

Metadata, parity, the corrected-sector status, the sequencer IRQ flags and a
software return value are different objects; none is another name for the
same status.

## 1.4 The three blocks

The NAND hardware at `0x38a00000` has three parts that share one register
window ([chapter 3, section 3.2](03-cpu-memory-map.md#32-memory-map-the-register-window)):

| block | registers | what it does | who drives it |
|---|---|---|---|
| **sequencer** | control block `0xc00..0xc6c`, registers `0xc18..0xc34`, parameter RAM `0xd00..0xd7c` ([chapter 8](08-sequencer-control-registers.md)) | runs a program of 8-byte instructions fetched from system memory | the CPU starts and stops it through `FMSS_CTRL` (`0xc00`) |
| **NAND controller** (FMC) | `0x000..0x0fc` ([chapter 9](09-nand-controller.md)) | puts commands, addresses and data on the NAND bus and moves data by DMA | whoever writes its registers: the sequencer program or the CPU |
| **ECC engine** | `0x800..0x8fc` ([chapter 10](10-ecc-engine.md)) | generates BCH parity for 512-byte sectors, checks and corrects them, and reports error counts and locations | the same: the sequencer program or the CPU |

The sequencer does no NAND work itself. Its program writes the NAND
controller's and the ECC engine's registers and waits for their completion
flags (`wait` events; M2-19). How the sequencer is started and stopped, and how
the CPU and the sequencer interact, is in [chapter 2](02-architecture.md). The
original programs (RESET, READID and READ in the bootloader; WRITE and ERASE in
the restore OS) are described in [chapter 11](11-nand-operations.md).

## 1.5 Named parts of the data path

Data passes through the following parts between system memory and a NAND chip.
"Device" marks what was measured and "inferred" what follows from it.

The data and parity buffers are storage inside the NAND controller. M2
evidence establishes no window word that reads or writes a data buffer
directly. Transfers fill and empty the buffers: chip transfers and DMAs of the
NAND controller (`FMCTRL1`), ECC operations, and the FIFO/metadata latch path
(`0x040`). The ECC engine exposes two results as registers: the parity of the
last encode at `0x840..0x85c` (M2-116) and the error locations of the last
decode at `0x820..` (M2-128).

### 1.5.1 System memory

- **What:** the SoC's SDRAM and internal SRAM on the CPU bus, addressed with
  32-bit bus addresses.
- **Used by the FMSS engine for:** the program (fetched from the bus address
  in `FMSS_PROG`, `0xc04`), `ldi`/`sti` data, and the data of every DMA (bus
  address in `FMC_DMA_ADDR`, `0x034`).
- **Details:** words are little-endian and `ldi`/`sti` addresses align down to
  a multiple of 4 (device, M2-05), as do DMA RAM addresses in both directions
  (device, M2-664). SRAM and SDRAM accesses cost the same in the
  measured forms (M2-102). [Chapter 4, section 4.5](04-sequencer-memory-map.md#45-system-memory-and-the-program).

### 1.5.2 Data buffer 0 and data buffer 1

- **What:** the 512-byte sector buffers between the NAND bus, the ECC engine
  and the DMA.
- **Selected by:** `0x014`; the original READ program alternates the selections
  `0x102` and `0x201` and uses `0x10` for the FIFO (observed; meaning of the
  fields inferred). `0x014` stores 16 bits; `0x058` is its bits 7:0 and
  `0x05c` reads its bits 15:8 in place, so a later `0x058` write replaces the
  low byte (device, M2-83, M2-643). A DMA in writes data buffer 0 only with
  `0x014` = 1; with 0 and the other tested non-source values it completes and
  writes no buffer (device, M2-668).
- **Filled and emptied by:** NAND data transfers and DMA. Memory → buffer →
  memory DMA is byte exact for both selections (device, M2-76). A DMA moves
  512 bytes whatever `FMC_DNUM` holds (device, M2-189). A DMA in flight follows
  a rewrite of `0x014` and of `FMC_DMA_ADDR` at the same byte offsets (device,
  M2-603).

### 1.5.3 Parity buffer 0 and parity buffer 1

- **What:** the storage for metadata and parity that travels with a sector,
  separate from the 512-byte data buffer (observed in the original programs).
- **Selected by:** `0x014` = `0x10` for a DMA (device, M2-232) and the ECC slot
  (section 1.5.4). The original programs also write the transfer word `0x058`
  (values 1, `0x10`, `0x20`; observed); `0x058` is `0x014` bits 7:0, so such a
  write sets the low byte of the selection, and a later `0x014` write replaces
  it (device, M2-597, M2-643).
- **Contents:** after an encode, the parity is readable at `0x840..0x85c`
  with the word order reversed and big-endian words (device, M2-116).
- **Size:** a 512-byte DMA into parity buffer 0 (`0x014` = `0x10`) leaves the
  source's last 64 bytes, which a DMA out returns repeated over 512 bytes and a
  128-byte program writes twice (device, M2-232): the buffer wraps at 64 bytes
  (inferred).

### 1.5.4 Slot

- **What:** the ECC buffer index 0 or 1 in bit 0 of `0x01c`; `0x01c` stores
  only that bit, and an encode uses the data buffer of the slot it selects
  (device, M2-83, M2-417).
- **Behaviour:** the original pipeline uses two sector/ECC selections (observed,
  M2-21). Of two encode kicks back to back the second replaces the first, and
  changing `0x01c` during an encode gives a parity that matches neither slot
  (device, M2-418, M2-419). Serial DMA/ECC sequences and overlap on different
  slots are byte exact for BCH8 and BCH16. A DMA into or out of a buffer while
  its own ECC operation is running loses bytes in runs of 4..32 bytes and sets
  `FMC_STATUS` bit 25 (device; meaning of the flag inferred, M2-80, M2-399,
  M2-401).

### 1.5.5 Chunk

- **What:** the original driver's 2 KiB DMA grouping: one DMA pointer per 2 KiB
  in parameter word `d20`, and `d28` = sectors per page divided by four
  ([chapter 12](12-cpu-interface.md)). The ECC engine works on 512-byte sectors.
  A chunk is a software transfer grouping, not a physical NAND page.

### 1.5.6 ECC area

- **What:** the metadata and parity bytes stored with the sectors of a page.
- **Layout used by the original driver:** for sector 0, ten metadata bytes,
  the parity and the 512 data bytes; for each further sector, parity and data.
  BCH8 uses 13 parity bytes per sector and BCH16 26 (observed, M2-22): 114
  metadata/parity bytes per 4 KiB page with BCH8, 218 with BCH16
  ([chapter 10, section 10.4](10-ecc-engine.md#physical-stream-used-by-the-driver)).
- **Codeword:** the sector-0 message is the data followed by the metadata
  bytes, MSB first (device, M2-77, M2-86). `ECC_CTRL` bits 13:8 set the
  metadata length; bits 15:14 are ignored (device, M2-98, M2-101, M2-191,
  M2-197). A length up to 32 covers that many stream bytes (the twelve bytes
  of `0x060..0x068`, then `0xff`); 33 and above give the parity of the data
  alone, as length 0 (device, M2-662).
- **Not a window range:** the M1 ECC register area `0x900..0x9fc` reads 0 on
  M2 (device, M2-171).

### 1.5.7 Metadata registers

- **What:** `0x060`, `0x064` and `0x068`, the FIFO/metadata data words, loaded
  by latch commands written to `0x040`: `0x52` ID latch, `0xc2` corrected
  metadata latch, `0xc1` metadata input (observed).
- **Device:** a READ ID through latch `0x52` appears in `0x060`/`0x064`; the ID
  repeats after five bytes (M2-124).

### 1.5.8 FIFO port (`FMC_FIFO`)

- **What:** `0x080`, a CPU read port. The restore OS reads it repeatedly and
  each read consumes FIFO data (observed, `fw_restore` `mem:0x0802a640`).
- **Device:** after a READ ID through latch `0x52`, `0x080` reads 0 (M2-124).
  Sequencer reads return 0 from DMA-filled data and parity buffers (M2-207).
  CPU reads return page data after `FMCTRL1` = `0x1c2` (chip read into the
  buffer) and `0x340`, 32 bytes per transfer (M2-436). Sequencer writes to it
  never reach a data buffer: without `0x340` they leave the buffer unchanged
  (M2-207); `0x340` with `0x014` = 1 writes 512 zero bytes to `FMC_DMA_ADDR`
  and is followed by a transfer of the RAM after that output into data buffer 0,
  which a page programmed from the buffer shows, and no written value appears
  (M2-341, M2-617).

### 1.5.9 Status register

- **What:** `0x04c`, the latched NAND chip status, distinct from the
  sequencer's `CS_IRQ` and from the controller's completion flags at `0x048`.
- **Device:** after one status latch, `0x04c` follows the chip: `0x80` while
  busy, `0xe0` when ready; without a latch it is not updated. With no CE it
  reads `0x70`, the command echo (M2-87, M2-136).

### 1.5.10 Chip page

- **What:** the NAND page as stored on the chip: data sectors and the ECC area
  (section 1.5.6). The measured chip uses 4 KiB pages with a 128-byte spare
  (section 1.6).

### 1.5.11 What the ECC engine touches

- **Encode:** reads the selected buffer's data and metadata and produces parity,
  readable at `0x840..0x85c` (M2-116).
- **Decode:** corrects data, metadata and parity: up to 8 bit errors with BCH8
  and up to 16 with BCH16; beyond that the data is left unchanged (device,
  M2-84, M2-92). Error locations are numbered over data (0..4095), then
  metadata and parity (M2-132, M2-138); per-decode results go to the 8-slot
  ring at `0x80c` (M2-91).



## 1.6 Scope of the device evidence

The device results come from one nano 4G unit with the Hynix chip `AD D5 14 B6`
(M2-124), 4 KiB pages with a 128-byte spare and BCH8 in the original format.
The 4 KiB/eight-sector programs and the CE0 and BCH8 defaults of the tests are a
workload; they do not establish the maximum page size, CE count, address
width, supported ECC modes or pipeline capacity of the engine.

Three kinds of behaviour are kept apart:

| layer | covers |
|---|---|
| sequencer | operand rules, register widths and aliases, arithmetic, branches, RAM access, PC, faults, pause and resume, reset and start state |
| SoC integration | WAIT sources, interrupt routing and acknowledgement, DMA ordering, buffers, ECC commands, clocks |
| attached NAND | ID, address cycles, ready/status behaviour, page layout, program and erase response |

NAND chip latency is not instruction latency. Shared Apple programs do not by
themselves make the nano 4G and nano 5G register maps, clocks, interrupts or
ISA edge cases identical; each finding names the SoC it was observed on.

A full execution of all 172 families (31,001 sampled native records) had no
setup, reset or cleanup failure (M2-64), and a later full execution of all
families reproduces the earlier captures with 79 of 31,974 outcomes differing (M2-144). One
capture begins at a power cycle, so its first access reads the power-on window
(M2-430); every other case starts from the ordinary reset. Execution
does not decode every tested word; hardware faults and observation timeouts are
expected outcomes of discovery cases. A program that reaches END does not
certify the bytes its peripherals moved (M2-71, M2-78). The device findings and
their limits are in [chapter 16](16-device-findings.md), their interpretation
in [chapter 15](15-evidence-interpretation.md) and the open points in
[chapter 17](17-open-questions.md).
