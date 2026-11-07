# nano 4G FMSS ("M2"): sequencer, NAND controller and ECC engine

These documents describe the FMSS engine of the iPod nano 4G (S5L8720): the
sequencer coprocessor inside the NAND controller, the NAND controller (FMC) it
drives, and the ECC engine. They cover the engine's behaviour, registers,
instruction set, memory maps, ECC, events, timing and interfaces, and how the
original firmware uses it. Each statement about the engine carries its
evidence class and a finding ID.

The documents form a book of numbered chapters with the same chapter numbers,
section structure, mnemonics and operand notation as the
M1 book of the nano 3G (S5L8702). Chapter 1
defines the evidence classes, the numbering and the terms
([conventions](01-overview.md#12-numbering-names-and-notation)). Chapters 2 to
12 are the reference chapters that describe the engine. Chapter 13 describes
the experiments. Chapters 14 to 16 hold the evidence: the outcomes, their
interpretation and the device findings. Chapter 17 lists the open questions,
chapter 18 is the finding list with the `M2-nn` IDs cited throughout, and
chapter 19 compares M2 with M1 finding by finding.

The compiled book [FMSS_M2.pdf](FMSS_M2.pdf) contains the same 19 chapters with
a linked contents page and chapter/section bookmarks.

## Chapters

| chapter | title | content |
|---|---|---|
| 1 | [Overview and terms](01-overview.md) | evidence classes, numbering, terms, the three blocks, the named parts of the data path, scope of the device evidence |
| 2 | [Architecture: execution model, start, stop and fault handling](02-architecture.md) | start, pause, hold, STOP, abort and reset, state codes, execution model, END and fault, start word, timing, completion versus data |
| 3 | [The register window as the ARM CPU sees it](03-cpu-memory-map.md) | offsets and names, memory map, CPU-only rules, interrupt lines, how the original CPU driver runs a program, wait sources |
| 4 | [The register window as the sequencer sees it](04-sequencer-memory-map.md) | window offsets and widths, segments and mirrors, register file and extended register roles, parameter RAM, system memory, program fetch |
| 5 | [Instruction encoding](05-instruction-encoding.md) | instruction word layout, operand map, opcode summary table, ALU operand rule, byte order |
| 6 | [Opcode reference](06-opcode-reference.md) | one entry per established or candidate mnemonic, plus unclassified opcode values |
| 7 | [Events, waits and the timeout](07-events-and-waits.md) | `wait` events, `wait2` conditions, `tmwait` and `evjmp`, `C14`, the timeout length |
| 8 | [Sequencer control registers](08-sequencer-control-registers.md) | the control block `0xc00..0xc6c` and the start-word bits |
| 9 | [NAND controller (FMC)](09-nand-controller.md) | `FMC_CTRL0`, the `FMCTRL1` codes, the other registers, `FMC_STATUS`, buffers, DMA, the `+0x400` range, the `0x100` block |
| 10 | [ECC engine](10-ecc-engine.md) | codeword layout, `ECC_CTRL`, the BCH8 and BCH16 codes, correction, status, result ring and error locations |
| 11 | [NAND operations](11-nand-operations.md) | how the original firmware performs reset, ID read, page read, page program, block erase and fault handling |
| 12 | [CPU interface: parameter words, result words and timing word](12-cpu-interface.md) | parameter words and their lifetime, completion and coherency, result layers, reset verification and recovery |
| 13 | [Experiments: what was done and why](13-experiments.md) | the experiment families, their purposes and their M1-to-M2 coverage |
| 14 | [Evidence: outcomes](14-evidence.md) | the outcomes of the device experiments and of the reverse engineering of the original firmware |
| 15 | [Evidence interpretation](15-evidence-interpretation.md) | what the outcomes are taken to show, by topic |
| 16 | [Device findings: risks and what only a device can settle](16-device-findings.md) | the risks and their status, and the results settled on a device |
| 17 | [Open questions](17-open-questions.md) | every point marked open, not measured or unexplained |
| 18 | [Finding list](18-ledger.md) | M2 findings with evidence and class |
| 19 | [Differences between M1 and M2](19-m1-m2-differences.md) | every M1 finding: on M1 / on M2, verdict and evidence |

## Evidence labels

* **Device / device-observed:** a physical capture with execution and initial
  state qualified for the case.
* **Observed / source-observed:** an original-code finding with image and
  address, from the Ghidra analyses; not a physical measurement.
* **Inferred:** an explanation or mapping supported by the stated observations.
* **3p:** third-party driver evidence, with its reported testing scope.
* **Open:** not settled by the available M2 evidence.
* **Emulated:** behaviour of the named model and build; no claim about
  unobserved silicon behaviour or calibrated hardware timing.

M1 results are comparison hypotheses on M2 until an M2 measurement confirms
them; the M1 channel-1 routing, power-on masks and RS modes are not imported.
