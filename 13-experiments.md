# Chapter 13 — M2 experiment coverage

<a id="m2-experiment-coverage"></a>

All 142 original names, READ ID and 64 follow-up families have runnable entries (207 total). Status counts: 9 applicability, 64 followup, 25 partial, 109 ported.

Every family runs resident by default, including `m2_wedge` and `m2_regbus`. The host uploads manifests, polls progress and collects result pages; a full result arena resumes the same uploaded manifest. `--isolate-diagnostics` explicitly collects those two diagnostic families per case; `--single-probes` opts every family out of batching. The sequence-only `m2_cpu_direct` campaign groups up to 128 jobs per manifest (1,850 jobs in 15 worker RUNs on p48, M2-727). Its batch API also supports map jobs, but the campaign contains no map phase.

`ported` describes the current M2 implementation, not device qualification. `partial` retains explicit missing cases; `applicability` does not reproduce the original operation until the M2 mapping is established. A model run can stop at its first unsupported case. Physical counts below describe retained summary records, not every native submission or successful semantic check. See [device findings](16-device-findings.md) for capture-specific execution qualification and state contamination.

Model column is a historical model report; it is not a fresh model run for the selected physical capture. Physical counts come from capture m2-all-p16.

Original questions: M1 chapter 13.

| Experiment | Migration | M2 scope / gap | Historical model attempt | Physical retained records |
|---|---|---|---|---|
| `state` | ported | M2-specific replacement | emulated | 1 sampled; 0 not executed; 0 case errors |
| `restore` | partial | native M2 reset; power-on defaults from the cold-device capture (`m2_cold_boot`) | emulated | 2 sampled; 0 not executed; 0 case errors |
| `regmap` | ported | shared ISA; M2 reset/start/snapshots | emulated | 1 sampled; 0 not executed; 0 case errors |
| `clock` | ported | shared ISA; M2 reset/start/snapshots | emulated | 2 sampled; 0 not executed; 0 case errors |
| `pram` | ported | shared ISA; M2 reset/start/snapshots | emulated | 1 sampled; 0 not executed; 0 case errors |
| `alu` | ported | shared ISA; M2 reset/start/snapshots | emulated | 9 sampled; 0 not executed; 0 case errors |
| `memory` | ported | shared ISA; M2 reset/start/snapshots | emulated | 1 sampled; 0 not executed; 0 case errors |
| `indirect` | ported | raw opcode discovery; M1 meanings are hypotheses | emulated | 4 sampled; 0 not executed; 0 case errors |
| `resume` | ported | shared ISA; M2 reset/start/snapshots | emulated | 7 sampled; 0 not executed; 0 case errors |
| `control` | ported | shared ISA; M2 reset/start/snapshots | emulated | 1 sampled; 0 not executed; 0 case errors |
| `wait` | ported | shared ISA; M2 reset/start/snapshots | emulated | 16 sampled; 0 not executed; 0 case errors |
| `c58` | ported | shared ISA; M2 reset/start/snapshots | emulated | 37 sampled; 0 not executed; 0 case errors |
| `op15` | ported | shared ISA; M2 reset/start/snapshots | unmodelled: M2 opcode 0x15 at 0x28 | 24 sampled; 0 not executed; 0 case errors |
| `op12` | ported | shared ISA; M2 reset/start/snapshots | unmodelled: M2 opcode 0x12 at 0x30 | 8 sampled; 0 not executed; 0 case errors |
| `c58ill` | ported | shared ISA; M2 reset/start/snapshots | unmodelled: M2 opcode 0x9 at 0x28 | 5 sampled; 0 not executed; 0 case errors |
| `startseq` | ported | raw opcode discovery; M1 meanings are hypotheses | emulated | 14 sampled; 0 not executed; 0 case errors |
| `startseq2` | ported | raw opcode discovery; M1 meanings are hypotheses | emulated | 18 sampled; 0 not executed; 0 case errors |
| `startseq3` | ported | raw opcode discovery; M1 meanings are hypotheses | unmodelled: M2 opcode 0x8 at 0x0 | 16 sampled; 0 not executed; 0 case errors |
| `oneshot` | ported | shared ISA; M2 reset/start/snapshots | emulated | 4 sampled; 0 not executed; 0 case errors |
| `c50` | ported | shared ISA; M2 reset/start/snapshots | unmodelled: M2 opcode 0x9 at 0x30 | 4 sampled; 0 not executed; 0 case errors |
| `holdout` | ported | raw opcode discovery; M1 meanings are hypotheses | unmodelled: M2 opcode 0x15 at 0x20 | 7 sampled; 0 not executed; 0 case errors |
| `op12wait` | ported | M2-specific replacement | unmodelled: M2 opcode 0x12 at 0x58 | 160 sampled; 0 not executed; 0 case errors |
| `ev11` | ported | M2-specific replacement | emulated | 8 sampled; 0 not executed; 0 case errors |
| `ev5` | ported | M2-specific replacement | unmodelled: M2 WAIT event 5 | 4 sampled; 0 not executed; 0 case errors |
| `c50x` | partial | timeout, opcode and unaligned scratch; arbitrary faulting bus addresses omitted | unmodelled: M2 opcode 0x1a at 0x20 | 3 sampled; 0 not executed; 0 case errors |
| `pending` | ported | raw opcode discovery; M1 meanings are hypotheses | emulated | 25 sampled; 0 not executed; 0 case errors |
| `c14` | ported | shared ISA; M2 reset/start/snapshots | unmodelled: M2 opcode 0x15 at 0x28 | 7 sampled; 0 not executed; 0 case errors |
| `op12ack` | ported | shared ISA; M2 reset/start/snapshots | unmodelled: M2 opcode 0x12 at 0x28 | 8 sampled; 0 not executed; 0 case errors |
| `op10` | ported | shared ISA; M2 reset/start/snapshots | unmodelled: M2 opcode 0x10 at 0x28 | 1 sampled; 0 not executed; 0 case errors |
| `unknown` | ported | raw opcode discovery; M1 meanings are hypotheses | unmodelled: M2 opcode 0x8 at 0x60 | 36 sampled; 0 not executed; 0 case errors |
| `opsweep` | ported | M2-specific replacement | unmodelled: M2 opcode 0x8 at 0x60 | 717 sampled; 0 not executed; 0 case errors |
| `optime` | ported | raw opcode discovery; M1 meanings are hypotheses | unmodelled: M2 opcode 0x8 at 0x28 | 16 sampled; 0 not executed; 0 case errors |
| `ctrlbits` | ported | shared ISA; M2 reset/start/snapshots | unmodelled: M2 start/control word 0x1 | 37 sampled; 0 not executed; 0 case errors |
| `fmcmap` | ported | raw opcode discovery; M1 meanings are hypotheses | emulated | 1 sampled; 0 not executed; 0 case errors |
| `evsweep` | ported | M2-specific replacement | unmodelled: M2 WAIT event 5 | 128 sampled; 0 not executed; 0 case errors |
| `rwmask` | ported | M2-specific replacement | emulated | 699 sampled; 0 not executed; 0 case errors |
| `op0f` | ported | shared ISA; M2 reset/start/snapshots | unmodelled: M2 opcode 0xf at 0x60 | 6 sampled; 0 not executed; 0 case errors |
| `ctrlhold` | ported | shared ISA; M2 reset/start/snapshots | unmodelled: M2 start/control word 0x3 | 13 sampled; 0 not executed; 0 case errors |
| `evclear` | ported | M2-specific replacement | emulated | 12 sampled; 0 not executed; 0 case errors |
| `timeregs` | ported | shared ISA; M2 reset/start/snapshots | emulated | 15 sampled; 0 not executed; 0 case errors |
| `ev5rule` | ported | M2-specific replacement | unmodelled: M2 WAIT event 5 | 16 sampled; 0 not executed; 0 case errors |
| `c08` | ported | shared ISA; M2 reset/start/snapshots | unmodelled: M2 opcode 0x9 at 0x20 | 7 sampled; 0 not executed; 0 case errors |
| `chan1` | applicability | candidate +400 set only; second-channel GPIO/event mapping unresolved | emulated | 8 sampled; 0 not executed; 0 case errors |
| `readev` | ported | M2-specific replacement | unmodelled: M2 WAIT event 5 | 128 sampled; 0 not executed; 0 case errors |
| `regnum` | ported | M2-specific replacement | unmodelled: extended register operand | 19 sampled; 0 not executed; 0 case errors |
| `branch` | ported | shared ISA; M2 reset/start/snapshots | unmodelled: unaligned instruction fetch 0x54 | 7 sampled; 0 not executed; 0 case errors |
| `waitf` | ported | shared ISA; M2 reset/start/snapshots | unmodelled: M2 WAIT event 27 | 10 sampled; 0 not executed; 0 case errors |
| `opcost` | ported | raw opcode discovery; M1 meanings are hypotheses | unmodelled: M2 opcode 0x8 at 0x40 | 30 sampled; 0 not executed; 0 case errors |
| `op12rel` | ported | M2-specific replacement | unmodelled: M2 opcode 0x12 at 0x20 | 55 sampled; 0 not executed; 0 case errors |
| `pause` | ported | shared ISA; M2 reset/start/snapshots | emulated | 7 sampled; 0 not executed; 0 case errors |
| `fmc28` | ported | raw opcode discovery; M1 meanings are hypotheses | emulated | 11 sampled; 0 not executed; 0 case errors |
| `blk100` | ported | M2-specific replacement | emulated | 128 sampled; 0 not executed; 0 case errors |
| `irqline` | ported | M2-specific replacement | unmodelled: M2 opcode 0x12 at 0x20 | 8 sampled; 0 not executed; 0 case errors |
| `irqen` | ported | M2-specific replacement | emulated | 264 sampled; 0 not executed; 0 case errors |
| `ev5t` | partial | repeated M2 latch comparison; delay/command-ack timing matrix remains | unmodelled: M2 WAIT event 5 | 6 sampled; 0 not executed; 0 case errors |
| `r8` | partial | M1 register-role comparison with local PC targets; direct LDI r8 low-address bus access remains unqualified | unmodelled: extended register operand | 11 sampled; 0 not executed; 0 case errors |
| `ev8` | applicability | candidate +400 set only; second-channel GPIO/event mapping unresolved | emulated | 8 sampled; 0 not executed; 0 case errors |
| `evcons` | partial | single waits in pending states; ordered repeated-wait combinations remain | unmodelled: M2 WAIT event 5 | 80 sampled; 0 not executed; 0 case errors |
| `dmaloop` | ported | M2-specific replacement | emulated | 8 sampled; 0 not executed; 0 case errors |
| `op0916` | ported | shared ISA; M2 reset/start/snapshots | unmodelled: M2 opcode 0x9 at 0x38 | 55 sampled; 0 not executed; 0 case errors |
| `op16` | ported | M2-specific replacement | unmodelled: M2 opcode 0x16 at 0x58 | 14 sampled; 0 not executed; 0 case errors |
| `opstate` | ported | M2-specific replacement | unmodelled: M2 opcode 0x8 at 0x58 | 49 sampled; 0 not executed; 0 case errors |
| `op16b` | partial | C58/pending states; delayed flag and vector target variants remain | unmodelled: M2 opcode 0x16 at 0x38 | 20 sampled; 0 not executed; 0 case errors |
| `op09` | ported | M2-specific replacement | unmodelled: M2 opcode 0x9 at 0x58 | 320 sampled; 0 not executed; 0 case errors |
| `op09b` | ported | M2-specific replacement | unmodelled: M2 opcode 0x9 at 0x58 | 288 sampled; 0 not executed; 0 case errors |
| `op09c` | partial | condition ranges ported; CE deselect/CE1 comparison remains | unmodelled: M2 opcode 0x9 at 0x58 | 128 sampled; 0 not executed; 0 case errors |
| `chan1pins` | applicability | candidate +400 set only; second-channel GPIO/event mapping unresolved | emulated | 8 sampled; 0 not executed; 0 case errors |
| `pcwrite` | ported | shared ISA; M2 reset/start/snapshots | emulated | 3 sampled; 0 not executed; 0 case errors |
| `op12cont` | ported | shared ISA; M2 reset/start/snapshots | unmodelled: M2 opcode 0x12 at 0x30 | 7 sampled; 0 not executed; 0 case errors |
| `ctrlfx` | partial | timeout, opcode and event-jump cases; enabled command IRQ case remains | unmodelled: M2 start/control word 0x1 | 48 sampled; 0 not executed; 0 case errors |
| `op09d` | applicability | candidate +400 set only; second-channel GPIO/event mapping unresolved | emulated | 8 sampled; 0 not executed; 0 case errors |
| `fwidth` | ported | raw opcode discovery; M1 meanings are hypotheses | unmodelled: register offset field width 0x1d70 | 18 sampled; 0 not executed; 0 case errors |
| `irqsrc` | ported | M2-specific replacement | emulated | 136 sampled; 0 not executed; 0 case errors |
| `eccpar` | ported | M2-specific replacement | emulated | 12 sampled; 0 not executed; 0 case errors |
| `bufsize` | ported | M2-specific replacement | unmodelled: DMA buffer length other than 512 | 10 sampled; 0 not executed; 0 case errors |
| `eccstr` | ported | M2-specific replacement | unmodelled: requires physical BCH parity/correction; model has synthetic layout-only parity | 68 sampled; 0 not executed; 0 case errors |
| `fields0` | ported | raw opcode discovery; M1 meanings are hypotheses | unmodelled: indirect operand alignment/immediate | 21 sampled; 0 not executed; 0 case errors |
| `tmofield` | ported | shared ISA; M2 reset/start/snapshots | unmodelled: M2 start/control word 0x5 | 12 sampled; 0 not executed; 0 case errors |
| `eccstr2` | ported | M2-specific replacement | unmodelled: requires physical BCH parity/correction; model has synthetic layout-only parity | 134 sampled; 0 not executed; 0 case errors |
| `ev5en` | ported | M2-specific replacement | unmodelled: M2 WAIT event 5 | 24 sampled; 0 not executed; 0 case errors |
| `tmobits` | ported | shared ISA; M2 reset/start/snapshots | unmodelled: M2 start/control word 0xff15 | 15 sampled; 0 not executed; 0 case errors |
| `eccmode` | ported | M2-specific replacement | unmodelled: ECC_CTRL candidate 0x302 | 36 sampled; 0 not executed; 0 case errors |
| `eccchunk` | partial | captured on-flash M2 chunks; parity-buffer DMA and host BCH construction remain | emulated | 4 sampled; 0 not executed; 0 case errors |
| `rawpage` | ported | M2-specific replacement | emulated | 2 sampled; 0 not executed; 0 case errors |
| `rawwrite` | ported | M2-specific replacement | emulated | 8 sampled; 0 not executed; 0 case errors |
| `writeseq` | partial | serialized M2 pages; overlapped pipeline and two-plane cases remain | emulated | 11 sampled; 0 not executed; 0 case errors |
| `op09nand` | ported | M2-specific replacement | unmodelled: M2 opcode 0x9 at 0xb8 | 1024 sampled; 0 not executed; 0 case errors |
| `eraseseq` | partial | single-plane/CE/address-cycle cases; two-plane erase remains | emulated | 20 sampled; 0 not executed; 0 case errors |
| `eccchunk2` | partial | captured-codeword corruption; mixed-error and direct parity-buffer cases remain | unmodelled: requires physical BCH parity/correction; model has synthetic layout-only parity | 48 sampled; 0 not executed; 0 case errors |
| `eccmode2` | ported | M2-specific replacement | unmodelled: ECC_CTRL candidate 0xa0a | 132 sampled; 0 not executed; 0 case errors |
| `tmobits2` | ported | shared ISA; M2 reset/start/snapshots | unmodelled: M2 start/control word 0xff55 | 5 sampled; 0 not executed; 0 case errors |
| `ev5ready` | ported | M2-specific replacement | unmodelled: M2 WAIT event 5 | 18 sampled; 0 not executed; 0 case errors |
| `pageorder` | ported | M2-specific replacement | emulated | 33 sampled; 0 not executed; 0 case errors |
| `op09w33` | ported | M2-specific replacement | unmodelled: M2 opcode 0x9 at 0x58 | 48 sampled; 0 not executed; 0 case errors |
| `ev5arm` | partial | M2 rearm candidates; short-read and data-select variants remain | unmodelled: M2 WAIT event 5 | 16 sampled; 0 not executed; 0 case errors |
| `writeseq2` | partial | sequential/repeat programming; overlap/last-DMA comparison remains | emulated | 12 sampled; 0 not executed; 0 case errors |
| `eccmode3` | ported | M2-specific replacement | unmodelled: ECC_CTRL candidate 0x302 | 40 sampled; 0 not executed; 0 case errors |
| `ev5two` | ported | M2-specific replacement | unmodelled: M2 WAIT event 5 | 6 sampled; 0 not executed; 0 case errors |
| `b15ecc` | applicability | M2 alternate-code selector unknown; candidate ECC_CTRL sweep only | unmodelled: ECC_CTRL candidate 0xa0a | 14 sampled; 0 not executed; 0 case errors |
| `eccmode4` | ported | M2-specific replacement | emulated | 24 sampled; 0 not executed; 0 case errors |
| `finale` | ported | M2-specific replacement | unmodelled: ECC_CTRL candidate 0xa0a | 387 sampled; 0 not executed; 0 case errors |
| `startdiff` | ported | M2-specific replacement | unmodelled: M2 opcode 0x1a at 0x20 | 24 sampled; 0 not executed; 0 case errors |
| `startbits` | ported | M2-specific replacement | unmodelled: M2 opcode 0x1a at 0x20 | 420 sampled; 0 not executed; 0 case errors |
| `rsdecode` | applicability | M2 alternate-code selector unknown; candidate ECC_CTRL sweep only | unmodelled: ECC_CTRL candidate 0xa0a | 14 sampled; 0 not executed; 0 case errors |
| `vic20` | ported | M2-specific replacement | emulated | 144 sampled; 0 not executed; 0 case errors |
| `fmc28b` | ported | M2-specific replacement | emulated | 10 sampled; 0 not executed; 0 case errors |
| `seqfirst` | ported | M2-specific replacement | emulated | 1 sampled; 0 not executed; 0 case errors |
| `w2state` | partial | basic M2 states ported; cold-boot/enable/control-word sequence remains | unmodelled: M2 opcode 0x9 at 0x58 | 1792 sampled; 0 not executed; 0 case errors |
| `c50b8` | ported | M2-specific replacement | emulated | 14 sampled; 0 not executed; 0 case errors |
| `c58b1` | ported | raw opcode discovery; M1 meanings are hypotheses | emulated | 28 sampled; 0 not executed; 0 case errors |
| `evsweep2` | partial | forward state sweeps; reverse ordering remains | unmodelled: M2 WAIT event 16 | 3072 sampled; 0 not executed; 0 case errors |
| `c50fault` | ported | M2-specific replacement | unmodelled: M2 opcode 0x1a at 0x28 | 8 sampled; 0 not executed; 0 case errors |
| `ctrlwords` | partial | word/DMA/ECC matrix; nested fault/vector-offset cases remain | emulated | 24 sampled; 0 not executed; 0 case errors |
| `c08self` | partial | M2 operation state captures; pause/continue/vector cases remain | emulated | 5 sampled; 0 not executed; 0 case errors |
| `latchrep` | ported | M2-specific replacement | unmodelled: M2 WAIT event 5 | 2560 sampled; 0 not executed; 0 case errors |
| `irq044` | ported | M2-specific replacement | emulated | 792 sampled; 0 not executed; 0 case errors |
| `fmcbits` | partial | known-page control/DMA variants; READID variants remain | emulated | 9 sampled; 0 not executed; 0 case errors |
| `fmcstat` | ported | M2-specific replacement | emulated | 32 sampled; 0 not executed; 0 case errors |
| `eccregs` | ported | M2-specific replacement | emulated | 15 sampled; 0 not executed; 0 case errors |
| `seqrestart` | partial | M2 resets/acks; within-program three-encode comparison remains | emulated | 12 sampled; 0 not executed; 0 case errors |
| `done5` | ported | M2-specific replacement | unmodelled: ECC_CTRL candidate 0xa0a | 58 sampled; 0 not executed; 0 case errors |
| `progtime` | ported | M2-specific replacement | emulated | 387 sampled; 0 not executed; 0 case errors |
| `startregion` | partial | region reads and scratch writes; reserved-block writes not selected | unmodelled: M2 start/control word 0xffd5 | 56 sampled; 0 not executed; 0 case errors |
| `bufport` | partial | candidate port reads; CPU/sequencer FIFO write experiment remains | emulated | 24 sampled; 0 not executed; 0 case errors |
| `c58b2` | ported | raw opcode discovery; M1 meanings are hypotheses | emulated | 29 sampled; 0 not executed; 0 case errors |
| `stprev` | ported | raw opcode discovery; M1 meanings are hypotheses | unmodelled: extended register operand | 39 sampled; 0 not executed; 0 case errors |
| `tmwait2` | ported | raw opcode discovery; M1 meanings are hypotheses | unmodelled: M2 opcode 0x15 at 0x28 | 7 sampled; 0 not executed; 0 case errors |
| `done5b` | ported | M2-specific replacement | unmodelled: ECC_CTRL candidate 0x200a02 | 24 sampled; 0 not executed; 0 case errors |
| `rsresult` | applicability | M2 alternate-code selector unknown; candidate ECC_CTRL sweep only | unmodelled: ECC_CTRL candidate 0xa0a | 14 sampled; 0 not executed; 0 case errors |
| `rschip` | applicability | M2 alternate-code selector unknown; candidate ECC_CTRL sweep only | unmodelled: ECC_CTRL candidate 0xa0a | 14 sampled; 0 not executed; 0 case errors |
| `miscorr` | ported | M2-specific replacement | unmodelled: requires physical BCH parity/correction; model has synthetic layout-only parity | 2080 sampled; 0 not executed; 0 case errors |
| `fullpage` | ported | M2-specific replacement | emulated | 42 sampled; 0 not executed; 0 case errors |
| `dmactl` | partial | serialized read variants; original overlapped read remains | emulated | 42 sampled; 0 not executed; 0 case errors |
| `fmcstat2` | ported | M2-specific replacement | emulated | 8 sampled; 0 not executed; 0 case errors |
| `ch1buf` | applicability | candidate +400 set only; second-channel GPIO/event mapping unresolved | emulated | 8 sampled; 0 not executed; 0 case errors |
| `evsweep3` | partial | M2 metadata/ECC/program/DMA; candidate second-channel/parity DMA remains | unmodelled: M2 WAIT event 16 | 2560 sampled; 0 not executed; 0 case errors |
| `blk100b` | ported | M2-specific replacement | emulated | 99 sampled; 0 not executed; 0 case errors |
| `straps` | ported | M2-specific replacement | emulated | 1 sampled; 0 not executed; 0 case errors |
| `irqop` | ported | M2-specific replacement | unmodelled: M2 WAIT event 15 | 48 sampled; 0 not executed; 0 case errors |
| `w2vic` | partial | raw VIC comparison; enable-source combinations remain | unmodelled: M2 opcode 0x9 at 0x58 | 1024 sampled; 0 not executed; 0 case errors |
| `regsweep` | ported | raw opcode discovery; M1 meanings are hypotheses | unmodelled: extended register operand | 744 sampled; 0 not executed; 0 case errors |
| `access` | ported | M2-specific replacement | emulated | 3680 sampled; 0 not executed; 0 case errors |
| `readid` | ported | M2 FIFO sequence | emulated | 4 sampled; 0 not executed; 0 case errors |
| `m2_resetcheck` | followup | Start=1 WAIT/illegal/evjmp cleanup, independent END after each reset | not run | 7 sampled; 0 not executed; 0 case errors |
| `m2_status_recovery` | followup | Status-latch failure, read-order controls and following execution | not run | 7 sampled; 0 not executed; 0 case errors |
| `m2_recovery` | followup | Pause cleanup, subsequent execution and explicit CPU resume controls | not run | 9 sampled; 0 not executed; 0 case errors |
| `m2_shifts` | followup | Full-width shift counts and immediate-zero encoding | not run | 168 sampled; 0 not executed; 0 case errors |
| `m2_pram_alias` | followup | Parameter boundary, indirect high-bit alias and alignment | not run | 100 sampled; 0 not executed; 0 case errors |
| `m2_c58_bits` | followup | CPU versus sequencer single-bit write/read masks | not run | 68 sampled; 0 not executed; 0 case errors |
| `m2_waitcons` | followup | Repeated/ordered waits, acknowledge and rearm within one program | not run | 80 sampled; 0 not executed; 0 case errors |
| `m2_status_order` | followup | Command acknowledgement and delay versus status-latch ordering | not run | 33 sampled; 0 not executed; 0 case errors |
| `m2_ce_conditions` | followup | No CE and each CE selection with WAIT/opcode09 operand sweeps | not run | 432 sampled; 0 not executed; 0 case errors |
| `m2_delayed_vector` | followup | Delayed CPU stimuli and alternate vector destinations | not run | 36 sampled; 0 not executed; 0 case errors |
| `m2_nested_fault` | followup | Fault offsets and a second fault inside the handler | not run | 12 sampled; 0 not executed; 0 case errors |
| `m2_pause_state` | followup | State across raw signal/pause/event opcodes and CPU resume candidates | not run | 12 sampled; 0 not executed; 0 case errors |
| `m2_ecc_sequence` | followup | Three encodes per program across reset/ack/slot changes, BCH8 and BCH16 | not run | 20 sampled; 0 not executed; 0 case errors |
| `m2_fifo_access` | followup | CPU and sequencer writes to the candidate FIFO, then DMA readback | not run | 12 sampled; 0 not executed; 0 case errors |
| `m2_timeline` | followup | Sampled register timelines (≈ 188 ns) during DMA, ECC, NAND reads, status latch, command, address | not run | not audited |
| `m2_wait_cpu` | followup | WAIT 6..10, 12..15 with one CPU-written control word 1 ms into the wait | not run | not audited |
| `m2_status_wait` | followup | WAIT5 and masked WAIT12 against a busy chip; WAIT5 latch orderings | not run | not audited |
| `m2_ecc_decode_regs` | followup | 0x820..0x85c after decodes with 1..8 errors | not run | not audited |
| `m2_reset_retention` | followup | Storage words written in one case, read in the next before-map | not run | not audited |
| `m2_jr_cond` | followup | JR as conditional jump: R field 0..7 with the condition register zero or one | not run | not audited |
| `m2_jr_width2` | followup | JR target width and alignment with r0 = 1 | not run | not audited |
| `m2_wait_sources` | followup | WAIT 5..10, 12..15 after encode, decode (erased/uncorrectable), meta latch, unacknowledged DMA, status latch | not run | not audited |
| `m2_fmc_flags` | followup | FMC_STATUS/C50/810/04c/C08 right after each controller operation, before any ack | not run | not audited |
| `m2_dma_cut` | followup | Second DMA started 0..256 nops after the first: bytes delivered (DMA rate) and 038 | not run | not audited |
| `m2_timeout_low` | followup | Start-word bits 7:4 with count 0x07: WAIT3 fault time | not run | not audited |
| `m2_ecc_regs` | followup | Sequencer view of the ECC block 0x800..0x9fc after known encodes and decodes | not run | not audited |
| `m2_jr_hazard` | followup | JR target-register hazard: 0..6 nops between the target write and JR | not run | not audited |
| `m2_jr_width` | followup | JR target width and alignment with the hazard avoided | not run | not audited |
| `m2_alu_hazard` | followup | Register-write hazards: movi/add/ld/ldi producers, add/mov/ldi/sti/bnz consumers at distance 0, 1, 2, 4 | not run | not audited |
| `m2_jump_width` | followup | JR and BNZ target width (bits 0..20) and alignment via landing pads | not run | not audited |
| `m2_wait_rb` | followup | WAIT 0..15 issued right after a block erase starts; status byte shows busy/ready at release | not run | not audited |
| `m2_wait4_decode` | followup | Decode-side WAIT4: which 810 write re-arms it after an erased-sector decode | not run | not audited |
| `m2_bch16_threshold` | followup | 810 bit-5 threshold for BCH16 (4..8, 12 errors) | not run | not audited |
| `m2_ecc_modes2` | followup | ECC_CTRL bits 24..31, encode+decode words, metadata length 13/16/32 | not run | not audited |
| `m2_opcost` | followup | Per-instruction time from 1000 vs 21000-iteration loops, SDRAM and SRAM program | not run | not audited |
| `m2_fmc_misc` | followup | FMC 018 before data reads and 05c before the final DMA on a page read | not run | not audited |
| `m2_ecc_threshold` | followup | BCH8 810 bit-5 threshold (1..8 errors), erased-page status, 818 under 0x81c | not run | not audited |
| `m2_wait4_ack` | followup | Which 0x810 write or 0x81c re-arms WAIT4 after an encode | not run | not audited |
| `m2_ecc_modes` | followup | ECC_CTRL encode words: metadata length and every other single bit, parity by raw read | not run | not audited |
| `m2_ecc_ring` | followup | 0x80c result ring: length, read-pointer advance, reset; 0x810 bit-5 threshold with 1..8 errors | not run | not audited |
| `m2_bch16_correct` | followup | BCH16 decode of host code words with 0..24 errors (seven sectors per page) | not run | not audited |
| `m2_ecc_status_timing` | followup | BCH8 decode: when ECC_RESULT/810/818 report the corrected count and failure, four ack variants | not run | not audited |
| `m2_bch16_encode` | followup | Engine BCH16 parity for known data (BCH8 control), recovered by raw read | not run | not audited |
| `m2_status_busy` | followup | Latched status byte during erase/program/read busy, idle and no-CE | not run | not audited |
| `m2_timeout_window` | followup | Multi-bit start-word timeout counts versus the single-bit linear fit | not run | not audited |
| `m2_bank_regs` | followup | Primary versus +400 storage words: independent, mirrored or inert, CPU and sequencer writers | not run | not audited |
| `m2_bch8_correct` | followup | Host BCH8 code words with 0..16 known bit errors: correction radius and reported counts | not run | not audited |
| `m2_dma_ack` | followup | p16 payload loss: exact p16 serial program versus acknowledged WAIT2, with and without ECC | not run | not audited |
| `m2_dma_ecc_overlap` | followup | Serial controls versus concurrent DMA/ECC on same/different slots; every explicit WAIT now acknowledged (p16 ran the unacknowledged generator) | not run | 24 sampled (p16 generator); 0 not executed; 0 case errors |
| `m2_dma_visibility` | followup | P16 partial payloads: plain DMA/BCH8/BCH16, CPU-timed holds and pre-cleanup checksum | not run | not audited |
| `m2_opcode_confirm` | followup | Expected-result checks of all 19 common ISA opcodes | not run | 531 sampled; 0 not executed; 0 case errors |
| `m2_opcode_m1` | followup | Retest all eight M1 device-discovered opcode meanings as explicit hypotheses | not run | 78 sampled; 0 not executed; 0 case errors |
| `m2_opcode_scan` | followup | All other opcode bytes, three operand forms, continue/vector policies | not run | 1422 sampled; 0 not executed; 0 case errors |
| `m2_opcode_states` | followup | Revisit seven ambiguous control opcodes with five pending stimuli | not run | 105 sampled; 0 not executed; 0 case errors |
| `m2_wedge` | followup | Register/PC/load transitions; per-case collection only with --isolate-diagnostics | not run | 59 sampled; 0 not executed; 0 case errors |
| `m2_regdest` | followup | M1 destination roles: all 256 selectors, MOVI/MOV/ADD, local PC targets | not run | 258 sampled; 0 not executed; 0 case errors |
| `m2_regsrc` | followup | M1 argument sources: all 256 selectors, MOV and every ALU/immediate-zero form | not run | 256 sampled; 0 not executed; 0 case errors |
| `m2_regbranch` | followup | M1 branch-register roles: all 256 selectors, BZ/BNZ, zero/nonzero controls | not run | 256 sampled; 0 not executed; 0 case errors |
| `m2_regaddr` | followup | M1 LDI/PLD address roles with mapped fallback pointers; PC bus role separate | not run | 255 sampled; 0 not executed; 0 case errors |
| `m2_regst` | followup | M1 last-register-bus-value rule: all extended selectors plus predecessor contrasts | not run | 287 sampled; 0 not executed; 0 case errors |
| `m2_regjr` | followup | M1 JR source roles: all 256 selectors, mapped labels and PC self-loop | not run | 256 sampled; 0 not executed; 0 case errors |
| `m2_reg10` | followup | P11 source 10 difference: all eight GPR destinations, distinct small shift counts, zero/large controls | not run | 64 sampled; 0 not executed; 0 case errors |
| `m2_resetpram` | followup | Isolate which CPU reset writes clear PRAM/GPRs; paired bounded-loop controls | not run | 8 sampled; 0 not executed; 0 case errors |
| `m2_regbus` | followup | Isolated PC-as-address applicability; run after collected register comparisons | not run | 3 sampled; 0 not executed; 0 case errors |

<a id="additional-matrices"></a>

## 13.1 Additional matrices

Run `--followups`, or include them via the device runner's `--all`. Each case records errors independently.

| Family | Extends M1 experiments | Question |
|---|---|---|
| `m2_resetcheck` | ctrlfx | Start=1 WAIT/illegal/evjmp cleanup, independent END after each reset |
| `m2_status_recovery` | ev5 | Status-latch failure, read-order controls and following execution |
| `m2_recovery` | op12,startseq | Pause cleanup, subsequent execution and explicit CPU resume controls |
| `m2_shifts` | alu | Full-width shift counts and immediate-zero encoding |
| `m2_pram_alias` | pram,indirect | Parameter boundary, indirect high-bit alias and alignment |
| `m2_c58_bits` | c58 | CPU versus sequencer single-bit write/read masks |
| `m2_waitcons` | evcons | Repeated/ordered waits, acknowledge and rearm within one program |
| `m2_status_order` | ev5t | Command acknowledgement and delay versus status-latch ordering |
| `m2_ce_conditions` | op09c | No CE and each CE selection with WAIT/opcode09 operand sweeps |
| `m2_delayed_vector` | op16b | Delayed CPU stimuli and alternate vector destinations |
| `m2_nested_fault` | ctrlwords,c50x | Fault offsets and a second fault inside the handler |
| `m2_pause_state` | c08self | State across raw signal/pause/event opcodes and CPU resume candidates |
| `m2_ecc_sequence` | seqrestart | Three encodes per program across reset/ack/slot changes, BCH8 and BCH16 |
| `m2_fifo_access` | bufport | CPU and sequencer writes to the candidate FIFO, then DMA readback |
| `m2_timeline` | dmactl,eccregs,fmcstat | Sampled register timelines (≈ 188 ns) during DMA, ECC, NAND reads, status latch, command, address |
| `m2_wait_cpu` | evsweep3,c58 | WAIT 6..10, 12..15 with one CPU-written control word 1 ms into the wait |
| `m2_status_wait` | ev5,ev5ready,latchrep | WAIT5 and masked WAIT12 against a busy chip; WAIT5 latch orderings |
| `m2_ecc_decode_regs` | eccregs | 0x820..0x85c after decodes with 1..8 errors |
| `m2_reset_retention` | restore,ctrlfx | Storage words written in one case, read in the next before-map |
| `m2_jr_cond` | regjr,branch | JR as conditional jump: R field 0..7 with the condition register zero or one |
| `m2_jr_width2` | regjr,fwidth | JR target width and alignment with r0 = 1 |
| `m2_wait_sources` | evsweep,evsweep2,evsweep3 | WAIT 5..10, 12..15 after encode, decode (erased/uncorrectable), meta latch, unacknowledged DMA, status latch |
| `m2_fmc_flags` | fmcstat,fmcstat2,c50 | FMC_STATUS/C50/810/04c/C08 right after each controller operation, before any ack |
| `m2_dma_cut` | dmactl,dmaloop | Second DMA started 0..256 nops after the first: bytes delivered (DMA rate) and 038 |
| `m2_timeout_low` | startbits,tmobits2 | Start-word bits 7:4 with count 0x07: WAIT3 fault time |
| `m2_ecc_regs` | eccregs,eccpar | Sequencer view of the ECC block 0x800..0x9fc after known encodes and decodes |
| `m2_jr_hazard` | regjr,r8 | JR target-register hazard: 0..6 nops between the target write and JR |
| `m2_jr_width` | regjr,fwidth | JR target width and alignment with the hazard avoided |
| `m2_alu_hazard` | alu,regsrc | Register-write hazards: movi/add/ld/ldi producers, add/mov/ldi/sti/bnz consumers at distance 0, 1, 2, 4 |
| `m2_jump_width` | branch,regjr,fwidth | JR and BNZ target width (bits 0..20) and alignment via landing pads |
| `m2_wait_rb` | evsweep,ev5ready | WAIT 0..15 issued right after a block erase starts; status byte shows busy/ready at release |
| `m2_wait4_decode` | evcons,done5b | Decode-side WAIT4: which 810 write re-arms it after an erased-sector decode |
| `m2_bch16_threshold` | eccstr2 | 810 bit-5 threshold for BCH16 (4..8, 12 errors) |
| `m2_ecc_modes2` | eccmode4,b15ecc | ECC_CTRL bits 24..31, encode+decode words, metadata length 13/16/32 |
| `m2_opcost` | opcost,optime,clock | Per-instruction time from 1000 vs 21000-iteration loops, SDRAM and SRAM program |
| `m2_fmc_misc` | fmcbits | FMC 018 before data reads and 05c before the final DMA on a page read |
| `m2_ecc_threshold` | eccregs,done5 | BCH8 810 bit-5 threshold (1..8 errors), erased-page status, 818 under 0x81c |
| `m2_wait4_ack` | evcons,done5b | Which 0x810 write or 0x81c re-arms WAIT4 after an encode |
| `m2_ecc_modes` | eccmode,eccmode2,eccmode3,b15ecc | ECC_CTRL encode words: metadata length and every other single bit, parity by raw read |
| `m2_ecc_ring` | eccregs,rsresult | 0x80c result ring: length, read-pointer advance, reset; 0x810 bit-5 threshold with 1..8 errors |
| `m2_bch16_correct` | eccstr,eccstr2 | BCH16 decode of host code words with 0..24 errors (seven sectors per page) |
| `m2_ecc_status_timing` | eccregs,done5,rsresult | BCH8 decode: when ECC_RESULT/810/818 report the corrected count and failure, four ack variants |
| `m2_bch16_encode` | eccpar,eccstr2 | Engine BCH16 parity for known data (BCH8 control), recovered by raw read |
| `m2_status_busy` | ev5,latchrep,ev5ready | Latched status byte during erase/program/read busy, idle and no-CE |
| `m2_timeout_window` | startbits,tmobits | Multi-bit start-word timeout counts versus the single-bit linear fit |
| `m2_bank_regs` | chan1,chan1pins | Primary versus +400 storage words: independent, mirrored or inert, CPU and sequencer writers |
| `m2_bch8_correct` | eccstr,eccstr2,miscorr,rawwrite | Host BCH8 code words with 0..16 known bit errors: correction radius and reported counts |
| `m2_dma_ack` | dmaloop,dmactl | p16 payload loss: exact p16 serial program versus acknowledged WAIT2, with and without ECC |
| `m2_dma_ecc_overlap` | dmactl,writeseq2 | Serial controls versus concurrent DMA/ECC on same/different slots |
| `m2_dma_visibility` | dmaloop,dmactl | P16 partial payloads: plain DMA/BCH8/BCH16, CPU-timed holds and pre-cleanup checksum |
| `m2_opcode_confirm` | alu,branch,regmap,memory,indirect,wait | Expected-result checks of all 19 common ISA opcodes |
| `m2_opcode_m1` | op0f,op10,op12,op15,op16,w2vic | Retest all eight M1 device-discovered opcode meanings as explicit hypotheses |
| `m2_opcode_scan` | unknown,opsweep,fields0 | All other opcode bytes, three operand forms, continue/vector policies |
| `m2_opcode_states` | opstate,op12wait,op16 | Revisit seven ambiguous control opcodes with five pending stimuli |
| `m2_wedge` | regnum | Register/PC/load transitions; per-case collection only with --isolate-diagnostics |
| `m2_regdest` | regnum,r8,regsweep | M1 destination roles: all 256 selectors, MOVI/MOV/ADD, local PC targets |
| `m2_regsrc` | regsweep | M1 argument sources: all 256 selectors, MOV and every ALU/immediate-zero form |
| `m2_regbranch` | regsweep | M1 branch-register roles: all 256 selectors, BZ/BNZ, zero/nonzero controls |
| `m2_regaddr` | regsweep | M1 LDI/PLD address roles with mapped fallback pointers; PC bus role separate |
| `m2_regst` | stprev,regsweep | M1 last-register-bus-value rule: all extended selectors plus predecessor contrasts |
| `m2_regjr` | r8,regsweep | M1 JR source roles: all 256 selectors, mapped labels and PC self-loop |
| `m2_reg10` | regsweep | P11 source 10 difference: all eight GPR destinations, distinct small shift counts, zero/large controls |
| `m2_resetpram` | restore,ctrlfx | Isolate which CPU reset writes clear PRAM/GPRs; paired bounded-loop controls |
| `m2_regbus` | r8,regsweep | Isolated PC-as-address applicability; run after collected register comparisons |
| `m2_r5_latch_clear` | stprev,regsweep | `ld`, `st` or `wr` between a latch predecessor and a high-bit source selector: kept or cleared |
| `m2_r5_ext_dest` | regnum,regsweep | `mov` to r9, r11..r15 with source selector 8 or 10: GPR dump and extended-`st` latch |
| `m2_r5_sti_high` | regsweep | `sti` with address selector 0x105, every GPR the address of its own marker, with and without a preceding `mov` |
| `m2_r5_rom_ldi` | memory,r8 | `ldi` of 0x20000000 + k × 0x1000 (k = 0..15) against the boot-ROM image |
| `m2_r5_op1a_operands` | unknown,fields0 | Undefined opcode 0x1a with 64 random R/arg/imm words against the zero form |
| `m2_r5_nop_stall_c08` | opsweep,c08self | `nop` R = 8 / 0xff with the timeout disabled: C08 inside the hold against an unmet `wait 3` |
| `m2_r5_c48_count7` | op15 | Count-7 `tmwait` with C48 = 0..8: end instant, C14 and fault |
| `m2_r5_tmwait_repeat` | op15 | Count-255 `tmwait` with and without a C14 = 0x800 write, five runs each |
| `m2_r5_prescaler_step` | startbits,restore | Count-0 `wait 3` first fault instant after one CPU reset step versus full reset and none |
| `m2_r5_c08_long` | c08self | C08 3 ms into a 400 ms loop of `nop2`, `ld` or `sti`, three runs each |
| `m2_r5_select_tag` | dmactl | FMC_DATA_SEL rewrite 16 / 32 `nop2` into a DMA out with both buffers tagged per word |
| `m2_r5_overlap_tags` | dmactl,dmaloop | Out-then-out and in-then-in DMA overlaps (gap 0 / 16) with tagged inputs |
| `m2_r5_dma_ctrl_sel` | dmactl | DMA out after 038 = 0x1f, 0x2f, 0x37; DMA in after 0xf; control 7 |
| `m2_r5_out_select` | bufport,dmactl | DMA out with FMC_DATA_SEL = 0, 1, 0x300, 0x400, 0x1100 (control 0x100), four tagged buffers |
| `m2_r5_x340_window` | bufport,writeseq2 | FMCTRL1 0x340 second transfer: delay alone or an 014 write after 0/8/16 `nop2`, then a page program and raw read |
| `m2_r5_status_lengths` | fmcstat | FMC_STATUS bits 19:12 after data reads of 160..508 bytes |
| `m2_r5_bch16_814` | eccregs,eccstr2 | BCH16 buffer encode and 12-error decode with 0x814 = 0x2f against 0 |
| `m2_cpu_stack` | — | CPU-only NAND read/program/erase (no sequencer program), hardware and software ECC, boot and main formats, against the sequencer readers |
