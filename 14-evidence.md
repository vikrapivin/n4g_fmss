# Chapter 14 — Evidence: outcomes


This chapter records M2 outcomes, following the nano3g evidence chapter.
Interpretation belongs in [chapter 15](15-evidence-interpretation.md), experiment
scope in [chapter 13](13-experiments.md), and outstanding questions in
[chapter 17](17-open-questions.md).

<a id="physical-capture"></a>

## 14.1 Physical capture: p06

Capture m2-all-p06, 589,189,971 bytes, SHA-256
`4c804e08dd2e326219266d9eed55f71b489776f51a59b7971202c4920240f8c2`.
The running image hash was not recorded. The capture contains 159 experiment
entries, 140 retained sampled records, 22,972 setup/preflight non-executions and
6,083 explicit errors. These counts describe retained records, not unique
native submissions: some legacy reports omit setup probes or overwrite labels.

The following table includes every returned entry. A host TIMEOUT is an
observation-window outcome; it does not identify an illegal opcode. A record
without sampled execution supplies no instruction or register measurement.

| Experiment | Sampled outcomes | Not executed | Case errors |
|---|---|---:|---:|
| `state` | END: 1 | 0 | 0 |
| `restore` | END: 2 | 0 | 0 |
| `regmap` | END: 1 | 0 | 0 |
| `clock` | END: 2 | 0 | 0 |
| `pram` | END: 1 | 0 | 0 |
| `alu` | END: 9 | 0 | 0 |
| `memory` | END: 1 | 0 | 0 |
| `indirect` | END: 4 | 0 | 0 |
| `resume` | END: 6 | 0 | 0 |
| `control` | END: 1 | 0 | 0 |
| `wait` | FAULT: 15, END: 1 | 0 | 0 |
| `c58` | FAULT: 37 | 0 | 0 |
| `op15` | TIMEOUT: 24 | 0 | 0 |
| `op12` | TIMEOUT: 1 | 7 | 0 |
| `c58ill` | none | 5 | 0 |
| `startseq` | TIMEOUT: 1 | 6 | 0 |
| `startseq2` | END: 9 | 0 | 0 |
| `startseq3` | END: 8 | 0 | 0 |
| `oneshot` | END: 2, TIMEOUT: 2 | 0 | 0 |
| `c50` | TIMEOUT: 3, END: 1 | 0 | 0 |
| `holdout` | TIMEOUT: 3, END: 4 | 0 | 0 |
| `op12wait` | TIMEOUT: 1 | 159 | 0 |
| `ev11` | none | 4 | 4 |
| `ev5` | none | 2 | 2 |
| `c50x` | none | 3 | 0 |
| `pending` | none | 20 | 0 |
| `c14` | none | 7 | 0 |
| `op12ack` | none | 8 | 0 |
| `op10` | none | 1 | 0 |
| `unknown` | none | 36 | 0 |
| `opsweep` | none | 717 | 0 |
| `optime` | none | 10 | 0 |
| `ctrlbits` | none | 37 | 0 |
| `fmcmap` | none | 1 | 0 |
| `evsweep` | none | 112 | 16 |
| `rwmask` | none | 233 | 466 |
| `op0f` | none | 6 | 0 |
| `ctrlhold` | none | 13 | 0 |
| `evclear` | none | 12 | 0 |
| `timeregs` | none | 15 | 0 |
| `ev5rule` | none | 16 | 0 |
| `c08` | none | 7 | 0 |
| `chan1` | none | 8 | 0 |
| `readev` | none | 128 | 0 |
| `regnum` | none | 19 | 0 |
| `branch` | none | 7 | 0 |
| `waitf` | none | 10 | 0 |
| `opcost` | none | 30 | 0 |
| `op12rel` | none | 55 | 0 |
| `pause` | none | 7 | 0 |
| `fmc28` | none | 11 | 0 |
| `blk100` | none | 128 | 0 |
| `irqline` | none | 8 | 0 |
| `irqen` | none | 264 | 0 |
| `ev5t` | none | 6 | 0 |
| `r8` | none | 11 | 0 |
| `ev8` | none | 8 | 0 |
| `evcons` | none | 80 | 0 |
| `dmaloop` | none | 8 | 0 |
| `op0916` | none | 55 | 0 |
| `op16` | none | 14 | 0 |
| `opstate` | none | 49 | 0 |
| `op16b` | none | 20 | 0 |
| `op09` | none | 320 | 0 |
| `op09b` | none | 192 | 96 |
| `op09c` | none | 128 | 0 |
| `chan1pins` | none | 8 | 0 |
| `pcwrite` | none | 3 | 0 |
| `op12cont` | none | 7 | 0 |
| `ctrlfx` | none | 48 | 0 |
| `op09d` | none | 8 | 0 |
| `fwidth` | none | 18 | 0 |
| `irqsrc` | none | 136 | 0 |
| `eccpar` | none | 12 | 0 |
| `bufsize` | none | 10 | 0 |
| `eccstr` | none | 68 | 0 |
| `fields0` | none | 21 | 0 |
| `tmofield` | none | 12 | 0 |
| `eccstr2` | none | 134 | 0 |
| `ev5en` | none | 24 | 0 |
| `tmobits` | none | 15 | 0 |
| `eccmode` | none | 36 | 0 |
| `eccchunk` | none | 2 | 2 |
| `rawpage` | none | 2 | 0 |
| `rawwrite` | none | 5 | 3 |
| `writeseq` | none | 6 | 5 |
| `op09nand` | none | 512 | 512 |
| `eraseseq` | none | 4 | 16 |
| `eccchunk2` | none | 16 | 32 |
| `eccmode2` | none | 132 | 0 |
| `tmobits2` | none | 5 | 0 |
| `ev5ready` | none | 16 | 2 |
| `pageorder` | none | 14 | 19 |
| `op09w33` | none | 24 | 24 |
| `ev5arm` | none | 16 | 0 |
| `writeseq2` | none | 9 | 3 |
| `eccmode3` | none | 40 | 0 |
| `ev5two` | none | 0 | 6 |
| `b15ecc` | none | 14 | 0 |
| `eccmode4` | none | 24 | 0 |
| `finale` | none | 387 | 0 |
| `startdiff` | none | 24 | 0 |
| `startbits` | none | 396 | 0 |
| `rsdecode` | none | 14 | 0 |
| `vic20` | none | 144 | 0 |
| `fmc28b` | none | 10 | 0 |
| `seqfirst` | none | 1 | 0 |
| `w2state` | none | 1792 | 0 |
| `c50b8` | none | 14 | 0 |
| `c58b1` | none | 19 | 0 |
| `evsweep2` | none | 2048 | 1024 |
| `c50fault` | none | 8 | 0 |
| `ctrlwords` | none | 24 | 0 |
| `c08self` | none | 5 | 0 |
| `latchrep` | none | 2560 | 0 |
| `irq044` | none | 792 | 0 |
| `fmcbits` | none | 7 | 2 |
| `fmcstat` | none | 32 | 0 |
| `eccregs` | none | 15 | 0 |
| `seqrestart` | none | 12 | 0 |
| `done5` | none | 58 | 0 |
| `progtime` | none | 0 | 387 |
| `startregion` | none | 52 | 4 |
| `bufport` | none | 24 | 0 |
| `c58b2` | none | 21 | 0 |
| `stprev` | none | 39 | 0 |
| `tmwait2` | none | 5 | 0 |
| `done5b` | none | 24 | 0 |
| `rsresult` | none | 14 | 0 |
| `rschip` | none | 14 | 0 |
| `miscorr` | none | 2080 | 0 |
| `fullpage` | none | 42 | 0 |
| `dmactl` | none | 40 | 2 |
| `fmcstat2` | none | 8 | 0 |
| `ch1buf` | none | 8 | 0 |
| `evsweep3` | none | 2048 | 512 |
| `blk100b` | none | 99 | 0 |
| `straps` | none | 0 | 0 |
| `irqop` | none | 48 | 0 |
| `w2vic` | none | 1024 | 0 |
| `regsweep` | none | 743 | 0 |
| `access` | none | 736 | 2944 |
| `readid` | none | 4 | 0 |
| `m2_shifts` | none | 168 | 0 |
| `m2_pram_alias` | none | 100 | 0 |
| `m2_c58_bits` | none | 68 | 0 |
| `m2_waitcons` | none | 80 | 0 |
| `m2_status_order` | none | 33 | 0 |
| `m2_ce_conditions` | none | 432 | 0 |
| `m2_delayed_vector` | none | 36 | 0 |
| `m2_nested_fault` | none | 12 | 0 |
| `m2_pause_state` | none | 12 | 0 |
| `m2_ecc_sequence` | none | 20 | 0 |
| `m2_fifo_access` | none | 12 | 0 |
| `m2_dma_ecc_overlap` | none | 24 | 0 |
| `m2_opcode_confirm` | none | 531 | 0 |
| `m2_opcode_m1` | none | 78 | 0 |
| `m2_opcode_scan` | none | 1422 | 0 |
| `m2_opcode_states` | none | 105 | 0 |


## 14.2 Selected observed values

| Case | Returned observation |
|---|---|
| `regmap` | r0..r7 at c18..c34 match; result equals r6 |
| `clock` | 2,000 / 20,000 two-instruction iterations: 138 / 1,355 us |
| `pram` | CPU writes retained d00..d7c; tested d80..dfc writes not retained |
| `alu` | logical right shift; wrap at 32 bits; tested counts 32/33 give zero; LD masks by AND |
| `memory` | little-endian RAM words; tested unaligned word accesses align down |
| `indirect` | op18 reads GPR/PRAM; op19 writes PRAM; 1d64 reads the d64 value |
| `wait` | WAIT11 passes in approximately 4 us; other idle 0..15 events fault at approximately 126.6 ms |
| `c58` | bit 2 distinguishes vector/continue; tested readback fits mask d6 |
| `op15` | 24 forms time out within their observation windows; C08=4, C14=35 |
| `op12` first case | C40/C44=12020d60/ffffffff; PC=30; C08=a; IRQ=8; worker=-2 |
| `op12wait` first case | C40/C44=12000000/0; PC=58; C08=a; IRQ=8; worker=-2 |
| `startseq3`, `waitc` (`08`) first | following marker and END reached for zero-field encoding |
| `holdout`, shifts | SHL 1 by 31 gives 80000000; SHR by 31 gives 1 |
| `holdout`, indirect r6 | r2 and r6 both 6666aaaa |
| `holdout`, RAM | store then unaligned reload gives 13572468 |

All numbers in register/encoding columns are hexadecimal. The full qualification
and chronology are retained in [device findings](16-device-findings.md). None of
the 2,136 new confirmation/discovery/state-opcode cases produced a usable
instruction measurement in p06.

<a id="original-code-observations"></a>

## 14.3 Original-code observations

| Source and address | Observation | Detailed reference |
|---|---|---|
| `bl_NandReadOnly`, mem:0x502c | reset writes C00=8, handles C08=4, resets controller and ECC | [CPU map](03-cpu-memory-map.md) |
| `bl_NandReadOnly`, mem:0x51b8 | start includes c60/c64 handshake and writes the start word | [CPU map](03-cpu-memory-map.md) |
| `bl_NandReadOnly`, mem:0x4d08 | pause IRQ handler advances PC by 8, acknowledges IRQ8, sets CTRL bit 0 | [control registers](08-sequencer-control-registers.md) |
| `bl_NandReadOnly`, mem:0x4cbc / mem:0x51f4 | interrupt clear / interrupt wait | [CPU map](03-cpu-memory-map.md) |
| `fw_restore`, mem:0x0802a640 | repeated reads of controller +080 consume FIFO data | [NAND controller](09-nand-controller.md) |
| Original READ/WRITE/ERASE programs | WAIT0..4 synchronize peripheral work; WAIT12 tests masked status | [NAND operations](11-nand-operations.md) |

Source use establishes that these sequences occur in the original firmware; it
does not prove every bit's meaning or all possible operand forms.


## 14.4 Emulated checks

The p09 revision tests actual ARM probe/worker code, packet-level USB transfers,
cache visibility, persistent peripheral state and explicit CPU bus-stall
injection. The native resident sequence completes nine pause
controls, seven status controls and 531 common-opcode checks. Status-before-ISA
ordering now works: clearing FMC/NSTAT on reset and deferring completions fixes
the earlier inherited-state WAIT9 model gap.

p09 corrects an earlier inferred mechanism: the physical status programs stop
at the second WAIT0, not the first WAIT5. The model now reproduces that location,
FMC reset phase 4, release and following execution. Buffer-blocked and ignored
release scenarios remain fault injection. The invalid-PC/CPU-stall test is also
explicitly hypothetical; no regnum result identifies the physical wedging case.

Manifest validation checks requests without executing unknown instructions.
BCH parity is synthetic and timing is uncalibrated; complete FMSS/VIC routing
is still open.

## 14.5 Physical capture: p07

The [p07 findings](16-device-findings.md#165-p07-pause-recovery-and-the-next-failure)
give provenance and qualification. Source SHA-256:
`751e38b88144d160fecb9ba2c1de5b0076e98ba2fcf547c8f4ac334bc5130653`.
All 29,279 native records are retained: 360 sampled, 25,494 not executed and
3,425 dependency errors. Nine recovery comparisons match. All 24 `tmwait` forms
reach END at 126,101–126,103 us, C14=0. `ev5`'s first case faults and reaches END,
then reset fails with C08=0; subsequent candidates have no execution.

| Experiment | Sampled outcomes | Not executed | Case errors |
|---|---|---:|---:|
| `m2_recovery` | END: 7, TIMEOUT: 2 | 0 | 0 |
| `state` | END: 1 | 0 | 0 |
| `restore` | END: 2 | 0 | 0 |
| `regmap` | END: 1 | 0 | 0 |
| `clock` | END: 2 | 0 | 0 |
| `pram` | END: 1 | 0 | 0 |
| `alu` | END: 9 | 0 | 0 |
| `memory` | END: 1 | 0 | 0 |
| `indirect` | END: 4 | 0 | 0 |
| `resume` | END: 7 | 0 | 0 |
| `control` | END: 1 | 0 | 0 |
| `wait` | FAULT: 15, END: 1 | 0 | 0 |
| `c58` | FAULT: 37 | 0 | 0 |
| `op15` | END: 24 | 0 | 0 |
| `op12` | TIMEOUT: 8 | 0 | 0 |
| `c58ill` | FAULT: 5 | 0 | 0 |
| `startseq` | END: 14 | 0 | 0 |
| `startseq2` | END: 18 | 0 | 0 |
| `startseq3` | END: 16 | 0 | 0 |
| `oneshot` | END: 2, FAULT: 2 | 0 | 0 |
| `c50` | FAULT: 3, END: 1 | 0 | 0 |
| `holdout` | FAULT: 2, END: 4, TIMEOUT: 1 | 0 | 0 |
| `op12wait` | TIMEOUT: 80, FAULT: 71, END: 9 | 0 | 0 |
| `ev11` | END: 4, FAULT: 4 | 0 | 0 |
| `ev5` | FAULT: 1 | 3 | 0 |
| `c50x` | none | 3 | 0 |
| `pending` | none | 25 | 0 |
| `c14` | none | 7 | 0 |
| `op12ack` | none | 8 | 0 |
| `op10` | none | 1 | 0 |
| `unknown` | none | 36 | 0 |
| `opsweep` | none | 717 | 0 |
| `optime` | none | 10 | 0 |
| `ctrlbits` | none | 37 | 0 |
| `fmcmap` | none | 1 | 0 |
| `evsweep` | none | 128 | 0 |
| `rwmask` | none | 233 | 466 |
| `op0f` | none | 6 | 0 |
| `ctrlhold` | none | 13 | 0 |
| `evclear` | none | 12 | 0 |
| `timeregs` | none | 15 | 0 |
| `ev5rule` | none | 16 | 0 |
| `c08` | none | 7 | 0 |
| `chan1` | none | 8 | 0 |
| `readev` | none | 128 | 0 |
| `regnum` | none | 19 | 0 |
| `branch` | none | 7 | 0 |
| `waitf` | none | 10 | 0 |
| `opcost` | none | 30 | 0 |
| `op12rel` | none | 55 | 0 |
| `pause` | none | 7 | 0 |
| `fmc28` | none | 11 | 0 |
| `blk100` | none | 128 | 0 |
| `irqline` | none | 8 | 0 |
| `irqen` | none | 264 | 0 |
| `ev5t` | none | 6 | 0 |
| `r8` | none | 11 | 0 |
| `ev8` | none | 8 | 0 |
| `evcons` | none | 80 | 0 |
| `dmaloop` | none | 8 | 0 |
| `op0916` | none | 55 | 0 |
| `op16` | none | 14 | 0 |
| `opstate` | none | 49 | 0 |
| `op16b` | none | 20 | 0 |
| `op09` | none | 320 | 0 |
| `op09b` | none | 288 | 0 |
| `op09c` | none | 128 | 0 |
| `chan1pins` | none | 8 | 0 |
| `pcwrite` | none | 3 | 0 |
| `op12cont` | none | 7 | 0 |
| `ctrlfx` | none | 48 | 0 |
| `op09d` | none | 8 | 0 |
| `fwidth` | none | 18 | 0 |
| `irqsrc` | none | 136 | 0 |
| `eccpar` | none | 12 | 0 |
| `bufsize` | none | 10 | 0 |
| `eccstr` | none | 68 | 0 |
| `fields0` | none | 21 | 0 |
| `tmofield` | none | 12 | 0 |
| `eccstr2` | none | 134 | 0 |
| `ev5en` | none | 24 | 0 |
| `tmobits` | none | 15 | 0 |
| `eccmode` | none | 36 | 0 |
| `eccchunk` | none | 4 | 0 |
| `rawpage` | none | 2 | 0 |
| `rawwrite` | none | 8 | 0 |
| `writeseq` | none | 11 | 0 |
| `op09nand` | none | 1024 | 0 |
| `eraseseq` | none | 20 | 0 |
| `eccchunk2` | none | 33 | 15 |
| `eccmode2` | none | 132 | 0 |
| `tmobits2` | none | 5 | 0 |
| `ev5ready` | none | 18 | 0 |
| `pageorder` | none | 33 | 0 |
| `op09w33` | none | 48 | 0 |
| `ev5arm` | none | 16 | 0 |
| `writeseq2` | none | 12 | 0 |
| `eccmode3` | none | 40 | 0 |
| `ev5two` | none | 6 | 0 |
| `b15ecc` | none | 14 | 0 |
| `eccmode4` | none | 24 | 0 |
| `finale` | none | 387 | 0 |
| `startdiff` | none | 24 | 0 |
| `startbits` | none | 420 | 0 |
| `rsdecode` | none | 14 | 0 |
| `vic20` | none | 144 | 0 |
| `fmc28b` | none | 10 | 0 |
| `seqfirst` | none | 1 | 0 |
| `w2state` | none | 1792 | 0 |
| `c50b8` | none | 14 | 0 |
| `c58b1` | none | 28 | 0 |
| `evsweep2` | none | 3072 | 0 |
| `c50fault` | none | 8 | 0 |
| `ctrlwords` | none | 24 | 0 |
| `c08self` | none | 5 | 0 |
| `latchrep` | none | 2560 | 0 |
| `irq044` | none | 792 | 0 |
| `fmcbits` | none | 9 | 0 |
| `fmcstat` | none | 32 | 0 |
| `eccregs` | none | 15 | 0 |
| `seqrestart` | none | 12 | 0 |
| `done5` | none | 58 | 0 |
| `progtime` | none | 387 | 0 |
| `startregion` | none | 56 | 0 |
| `bufport` | none | 24 | 0 |
| `c58b2` | none | 29 | 0 |
| `stprev` | none | 39 | 0 |
| `tmwait2` | none | 7 | 0 |
| `done5b` | none | 24 | 0 |
| `rsresult` | none | 14 | 0 |
| `rschip` | none | 14 | 0 |
| `miscorr` | none | 2080 | 0 |
| `fullpage` | none | 42 | 0 |
| `dmactl` | none | 42 | 0 |
| `fmcstat2` | none | 8 | 0 |
| `ch1buf` | none | 8 | 0 |
| `evsweep3` | none | 2560 | 0 |
| `blk100b` | none | 99 | 0 |
| `straps` | none | 1 | 0 |
| `irqop` | none | 48 | 0 |
| `w2vic` | none | 1024 | 0 |
| `regsweep` | none | 744 | 0 |
| `access` | none | 736 | 2944 |
| `readid` | none | 4 | 0 |
| `m2_shifts` | none | 168 | 0 |
| `m2_pram_alias` | none | 100 | 0 |
| `m2_c58_bits` | none | 68 | 0 |
| `m2_waitcons` | none | 80 | 0 |
| `m2_status_order` | none | 33 | 0 |
| `m2_ce_conditions` | none | 432 | 0 |
| `m2_delayed_vector` | none | 36 | 0 |
| `m2_nested_fault` | none | 12 | 0 |
| `m2_pause_state` | none | 12 | 0 |
| `m2_ecc_sequence` | none | 20 | 0 |
| `m2_fifo_access` | none | 12 | 0 |
| `m2_dma_ecc_overlap` | none | 24 | 0 |
| `m2_opcode_confirm` | none | 531 | 0 |
| `m2_opcode_m1` | none | 78 | 0 |
| `m2_opcode_scan` | none | 1422 | 0 |
| `m2_opcode_states` | none | 105 | 0 |

## 14.6 Physical capture: p09

Capture m2-all-p09: 4,657 sampled records across 52 returned families,
SHA-256
`d4d4215df32a96381c58c6fff8c53f29215762e68d25b83ed278c40900ef4316`.
There are no retained setup/worker failures. All three faulting status controls
recover and permit following execution; the first reset failure is phase 4.
531 common-opcode cases yield 526 matches and five wrong immediate-ack FAULT
expectations, corrected to observed END. 78 M1-comparison expectations match.
Broad opcode scans now have execution, with meanings still qualified per form.
Communication disappears during regnum, with no returned case; 33 subsequent
timeouts are transport failures. See [device findings](16-device-findings.md#167-p09-status-recovery-opcode-controls-and-communication-loss).

## 14.7 p10: M1 register-role comparisons on M2

All 59 `m2_wedge` requests reach END with IRQ=1; all 38 scored controls match.
Capture SHA-256 `aa918f288abbf3777afb073fadcda131a487544f785ac757ddca0e2d26a21fb5`.
Qualification: [section 16.8](16-device-findings.md#168-p10-m1-register-role-comparisons-on-m2).

## 14.8 p11: full selector comparisons and source-10 difference

Capture m2-all-p11: 1,590 sampled records, 1,589 END plus the expected JR r8
self-loop TIMEOUT; of 1,569 scored cases, 1,568 match M1. The one difference,
`m2_regsrc/argument-source/010`, gives AND/OR=`22002222`, ADD=`44004444`, SUB=0
(self-source, not M1's r0 rule). LDI/PLD at PC=0x60 return `e1a0f00e`/0.
Qualification: [section 16.9](16-device-findings.md#169-p11-full-selector-comparisons-and-source-10-difference).

## 14.9 p12: source 10 and isolated reset phases

72 requests: 64 END/matches and eight expected observation TIMEOUTs with IRQ=0.
Capture 693,753 bytes, SHA-256
`d6013102ed545f12fcc584ccc0f1144a9a50307cdcb0fe9d8c776cafb8093953`.
All 1,664 MOV/ALU result slots match self-source behaviour (245 distinguish it
from M1's r0 rule). STOP clears PC but preserves GPRs/PRAM; STOP then
C00=80000 clears GPRs, PRAM, C04/C10/C40/C44/C4C.
Qualification: [section 16.10](16-device-findings.md#1610-p12-source-10-and-isolated-reset-phases).

## 14.10 P13: full attempt, state-b reset failure

6,204 sampled records (one cleanup failure), 21,831 initial-reset failures and
2,959 dependency errors over 171 families. First failure ctrlfx/0002: start=1,
evjmp at PC 0x20, C08=b, IRQ0; all later initial failures share that signature.
All 531 common opcode and 78 M1 comparison checks before it match.
Qualification: [section 16.11](16-device-findings.md#1611-p13-full-attempt-state-b-reset-failure).

## 14.11 P14: no candidates execute; forced reset was skipped

Seven initial phase 1 failures, C08=b/PC20/IRQ0, method 8 only.
Qualification: [section 16.12](16-device-findings.md#1612-p14-no-candidates-execute-forced-reset-was-skipped).

## 14.12 P15: effective reset and independent execution

Capture m2-all-p15: seven sampled cases, no errors; four marker
assessments match; WAIT3, zero-field illegal1a and evjmp started with C00=1 stay
in their 2 ms observation TIMEOUTs. STOP leaves C08=b/PC20/IRQ0, abort reports
3, forced reset before another STOP reports 0, the ordinary reset finishes
phase 6/rc0 and the next marker reaches END/IRQ1/r2=72657365 (method 01000318).
Qualification: [section 16.13](16-device-findings.md#1613-p15-effective-reset-and-independent-execution).

## 14.13 P16: full campaign with effective recovery

Capture m2-all-p16, 407,282,777 bytes, SHA-256
`56c8e1c81c4deee4994ed28b0e97412818ea8e802f64085b0f22a191e81e7534`.
All 172 families use resident batches; 31,001 native records are sampled.
There are no setup/reset/cleanup/transport failures. Preflight is 1 in 30,978
cases and explicitly skipped (2) in 23. `attempt_finished=true`; `complete=false`
reflects the one known M1 source-10 hypothesis mismatch, not incomplete execution.

| Scoring class | Records |
|---|---:|
| match | 2296 |
| mismatch against retained M1 hypothesis | 1 |
| discovery observation | 1527 |
| unscored native observation | 27177 |

The 531 core and 78 M1 opcode-comparison cases all match. All 157 previously
deferred event/ECC/DMA cases now execute. The detailed findings are in
chapters 7–10 and 16.
No image hash or raw journal was supplied; bit 27 is present. The compressed
copy retains the exact original bytes after decompression (23023680 bytes on disk).

## 14.14 P16 offline re-analysis

Offline re-analysis of the retained p16 records:

| Record set | Outcome |
|---|---|
| `dmaloop` (8) | all outputs byte exact |
| `dmactl` page reads reaching END (24) | all 4 KiB pages byte exact; 0x10..0x80 controls fault |
| `m2_dma_ecc_overlap` (24, p16 generator) | first output exact for 32..64 bytes, second differs from byte 0; 0x038 mostly 0x187 |
| raw sectors (`rawpage`, `rawwrite`, `fullpage`) | 352/352 match M1 BCH8 with message data‖meta10 (sector 0) or data |
| erased raw sectors | 8, all 0xff |

## 14.15 P17: DMA acknowledgement and overlap

Capture m2-dma-p17, 48,343 bytes; 34 cases, all resets
verified.

| Family / cases | Outcome |
|---|---|
| `m2_dma_ack` acknowledged (6) | all byte exact (0x038 = 7 or 0) |
| `m2_dma_ack` exact p16 program (2) | first output exact for 32 / 36 bytes |
| `m2_dma_ack` plain stale (2) | first output exact for 256 bytes; 0x038 = 0x207 |
| `m2_dma_ecc_overlap` serial + different-slot (16) | all byte exact; 048 = 0xc3000 |
| `m2_dma_ecc_overlap` same-slot (8) | 96..140 old bytes in 4..32-byte runs; 048 = 0x020c3000 |

## 14.16 P18: timeout, +400 words, BCH8 correction

Capture m2-dma-p18, 32,635 bytes; 34 cases, no case errors.

| Family | Outcome |
|---|---|
| `m2_timeout_window` (14) | fault_at = 492.747 × (count + 1) + 447.5 us, residual ≤ 1.1 us; WAIT3 = illegal opcode |
| `m2_bank_regs` (6) | every +400 word reads 0; primary values identical in both orders; masks in [section 9.1](09-nand-controller.md#91-register-map) |
| `m2_bch8_correct` (14) | raw writes exact; ≤ 8 errors corrected (25 sectors); > 8 left unchanged (4); ECC_RESULT not per-sector |

## 14.17 P19: ECC status, BCH16, status during busy

Capture m2-p19, 47,756 bytes; 19 cases, all END.

| Family | Outcome |
|---|---|
| `m2_bch16_encode` (7) | BCH16 14/14 narrow-sense t = 16; BCH8 control 7/7; repeat identical |
| `m2_status_busy` (5) | 04c 0x80 busy / 0xe0 ready; program 15 busy polls, erase ≥ 48, read 1; no CE 0x70 |
| `m2_ecc_status_timing` (7) | 810 bit 3 uncorrectable, bit 5 at 8; 818 corrected total << 11 \| decodes; 80c ring of 8 advanced by reads |

## 14.18 P20: result ring and BCH16 correction

Capture m2-p20, 26,727 bytes; 16 cases, all END.

| Family | Outcome |
|---|---|
| `m2_ecc_ring` (9) | period-8 ring; 80c reads advance it, 810/818 reads and 80c writes do not; 0x81c aligns to slot 0 |
| `m2_bch16_correct` (7) | ≤ 16 errors corrected (10 sectors), > 16 unchanged (4); 80c per sector correct after 0x81c |

## 14.19 P21: ECC status bits, WAIT4 release, ECC_CTRL

Capture m2-p21, 44,789 bytes; 56 cases.

| Family | Outcome |
|---|---|
| `m2_ecc_threshold` (4) | 810 0x15 for 1..6, 0x35 for 7..8, 0x55 erased; 81c clears 818 |
| `m2_wait4_ack` (13) | only 810 writes containing bit 1, and 81c, re-arm WAIT4 |
| `m2_ecc_modes` (39) | bits 15:8 metadata length, bit 2 BCH16; bits 3..7, 16..23 no parity effect |

## 14.20 P22: decode WAIT4, BCH16 bit 5, ECC_CTRL, timing, FMC 018/05c

Capture m2-p22, 152,023 bytes; 89 cases.

| Family | Outcome |
|---|---|
| `m2_wait4_decode` (6) | only writes with bit 2 (4, 0x1ff) re-arm WAIT4 after a decode |
| `m2_bch16_threshold` (3) | 810 bit 5 from 7 errors |
| `m2_ecc_modes2` (27) | bits 24..31 no effect; 0x3/0x7 decode; metadata 13/16/32 pad with 0xff |
| `m2_opcost` (48) | ALU 30.1 ns, window access 60.6 ns, ldi SDRAM 211 ns; SRAM = SDRAM |
| `m2_fmc_misc` (5) | 018 / 05c: no data or ECC effect |

## 14.21 P23: jump targets and WAIT vs ready/busy

Capture m2-p23, 11,281 bytes; 58 cases.

| Family | Outcome |
|---|---|
| `m2_jump_width` BNZ (21) | low 3 bits ignored; pads 2^4..2^14 hit; beyond program: SDRAM executed, PC 0x40200 once |
| `m2_jump_width` JR (21) | all fell through (target register written by the preceding instruction; cause r[R] = 0, not a hazard: M2-108, M2-110) |
| `m2_wait_rb` (16) | only WAIT 11 passes, immediately while busy |

## 14.22 P24: register hazards and JR

Capture m2-p24, 8,218 bytes; 37 cases, all END.

| Family | Outcome |
|---|---|
| `m2_alu_hazard` (8) | new value at every distance 0..4 (32/32) |
| `m2_jr_hazard` (8), `m2_jr_width` (21) | JR fell through in all 29 (r0 = 0) |

## 14.23 P25 bundle

Capture m2-p25, 47,361 bytes; 165 cases, all resets
verified.

| Family | Outcome |
|---|---|
| `m2_jr_cond` (16) | jump iff r[R] ≠ 0 |
| `m2_jr_width2` (21) | consistent with exact jumps after an 8-byte pad shift (generator error) |
| `m2_wait_sources` (71) | only status latch → WAIT 5 |
| `m2_fmc_flags` (15) | flag map ([section 9.7](09-nand-controller.md#97-fmc_status-flags), [section 8.14](08-sequencer-control-registers.md#814-fmss_c50-0xc50-completion-bits)) |
| `m2_dma_cut` (18) | gap ≥ 1.9 us exact; shorter: TIMEOUT, 8..48 bytes |
| `m2_timeout_low` (16) | count 7: n = 0, 1 fault within 0..2 us; step 2^(n+3) cycles for n = 4..11, saturating at n = 11; n = 2, 3 (4, 8 us) do not fit 2^(n+3) (steps 2^3, 2^5: M2-115, M2-216) |
| `m2_ecc_regs` (8) | parity at 0x840..0x85c; 0x800 = 0x20070618 |

## 14.24 P26 bundle (partial)

Capture m2-p26, 18,245 bytes. Valid: `m2_timeline` (19),
`m2_wait_cpu` C0C/C10/C48/C54 (36). C60 = 1 by CPU held the engine; 66 later
cases not executed.

## 14.25 P43 bundle

Capture m2-p43, 183,161 bytes, SHA-256
`46e9e6fade291332008c5384fb5716ca1cc0fadaef6c874612ed63936ec08f31`; 18 families
(17 `m2_r5_*` and `m2_cpu_stack`), 0 case errors,
every requested reset verified (the eight no-reset cases of
`m2_r5_prescaler_step` skip the reset by design).

| Family | Outcome |
|---|---|
| `m2_r5_latch_clear` (8) | high-bit source keeps the predecessor's value across `ld`/`st`/`wr` (M2-674..M2-677) |
| `m2_r5_ext_dest` (12) | no GPR written, latch unchanged (M2-678, M2-679) |
| `m2_r5_sti_high` (3) | r0 after `movi`, r5 after `mov` with selector 5: the last source value (M2-680, M2-681) |
| `m2_r5_rom_ldi` (16) | one ROM word per 16-KiB quarter, never the addressed word (M2-682, M2-683) |
| `m2_r5_op1a_operands` (65) | zero-form outcome for all 64 operand words (M2-684) |
| `m2_r5_nop_stall_c08` (4) | C08 = 4 for R = 8, 0xff and unmet `wait 3` (M2-685, M2-686) |
| `m2_r5_c48_count7` (9) | ends 493 us earlier per C48 unit with C14 = C48; C48 = 8 faults (M2-687, M2-688) |
| `m2_r5_tmwait_repeat` (10) | 126079..126108 us; one 23-us early end, without a C14 write (M2-689, M2-690) |
| `m2_r5_prescaler_step` (10) | reset 76/78 us, none 24/34, C00 82/82, FMCTRL1 30/32, 0x81c 28/32 (M2-691, M2-692) |
| `m2_r5_c08_long` (9) | `nop` 9/9/2, `ld` 1/1/1, `sti` 8/1/8 (M2-693) |
| `m2_r5_select_tag` (6) | switch at word 63/57/58 (gap 16, buffer-1 word 120 at the switch twice), 114/114/114 (gap 32) (M2-694..M2-696) |
| `m2_r5_overlap_tags` (4) | every word an input word, initial content or zero (M2-697, M2-698) |
| `m2_r5_dma_ctrl_sel` (5) | 038 bits 5:3 = 3/5/6 stall after 32 bytes; DMA in 0xf: word 4g + j from 4g + 5j (M2-699..M2-701) |
| `m2_r5_out_select` (6) | only 0x100 delivers buffer 0, all others 512 zero bytes (M2-702) |
| `m2_r5_x340_window` (9) | delay alone completes; 014 write after 0/8/16 `nop` leaves 0/16/32 RAM words (M2-703..M2-705) |
| `m2_r5_status_lengths` (11) | 0x41 for 160..224, 0x14 for 255..508 (M2-706, M2-707) |
| `m2_r5_bch16_814` (4) | encode and decode identical with 0x814 = 0x2f (M2-708..M2-710) |
| `m2_cpu_stack` (93 sequencer-harness records, 107 CPU jobs) | 211 checks: 208 match, 1 observation, 2 mismatches (both the location-order check: the engine lists the bits of one byte higher first, M2-714); controller idle after 89/89 jobs (M2-711..M2-716) |

## 14.26 Reboot sessions: unmapped bus accesses

Captures m2-reboot-ldi (2,466 bytes, SHA-256
`44e95cc6b5738f2ea96f9e386a04fa91fbd502efb36e1400dae756852690e753`) and
m2-reboot-dma (2,615 bytes, SHA-256
`f047be85b0cfb10b2049bfbf0018fc9efb3b164386e68c7c6da0eefa89ce2f9e`), each run alone with
`--single-probes` after a power cycle.

| Family | Outcome |
|---|---|
| `m2_reboot_ldi_hole` (4) | both controls END; `ldi/hole/stall-safe` lost USB (no record), the ordinary hole then timed out on a dead device (M2-718) |
| `m2_reboot_dma_hole` (4) | both controls END; `dma/hole` and `dma/hole/stall-safe` END after 9 us / at the 50-ms window, 034 = 0x50000000, device alive (M2-717) |

## 14.27 P48 bundle (round 6)

Capture m2-p48, 11,778,983 bytes; the attempt did not finish (USB lost in
`m2_r6_gate5`). Analysed families: `m2_r6_c08` (15), `m2_r6_read_status` (12), `m2_r6_latch` (12),
`m2_r6_dma` (30), `m2_r6_vic1` (33), `m2_r6_pc` (36), `m2_r6_apple` (8).
The capture SHA-256 is
`190c5f1404c0a2b7d9506d54ece388e3c1747f220ac17a30651f84cadf6ccf5e`;
the device image hash was not recorded.

| Family | Outcome |
|---|---|
| `m2_r6_c08` | CPU-sampled C08 1/2/9, 7 (SDRAM), 5 (window), 4 (wait) (M2-719) |
| `m2_r6_read_status` | bits 19:12 0xc3 → 0x41 → 0x14 → 0x3c during reads (M2-720) |
| `m2_r6_latch` | bits 3/23 change in every order within sample resolution (M2-721) |
| `m2_r6_dma` | exact transfers; 038 bits 12:6 live during the DMA (M2-722) |
| `m2_r6_vic1` | `waitirq` 32/33 released by hardware VIC1 lines 0/1 only; line 52 never rose (M2-723, M2-724) |
| `m2_r6_pc` | `bnz`/`jr` exact to 2^24 + 0x100, no alias (M2-725) |
| `m2_r6_apple` | original ERASE/WRITE erase and program blocks 3000/3001 exactly (M2-726) |
| `m2_cpu_direct` | 1,850 jobs in 15 worker RUNs, all resets verified, 58.06 s execution; all 12 required round trips fail their erased/DMA/FIFO payload checks (M2-727, M2-728) |
| `m2_r6_gate5` | baseline and gate0..2 return; gate0 already off, gate1/2 initially on; VIC0 raw 0x20 and VIC1 raw 0 throughout each 28-us pulse, clocks restored; gate3 loses USB with no result (M2-729) |

The CPU executor left DMA enabled during FIFO reads. The inferred stray-DMA
corruption prevents using the 44 FIFO-containing cases to establish optional
NAND responses. Separate DMA-only observations retain six mismatching
cache/copyback/multi-plane hypotheses and 48 mismatching timing comparisons; passing a
reference model does not supersede these device outcomes (M2-728).

### 14.27.1 Harness correction and model evidence

The corrected reader selects the path per read and restores CE/timing control.
The clock pulse loads its state into registers before gating and restores
clocks before any stack, SDRAM or peripheral read. GATE_MARK|2 fields
r0/r1/r3/r4/r5 are `0xffffffff`, meaning unavailable; its timestamp is measured
after restoration. Neither a stopped timer nor an interrupt state during the
window is inferred from those fields (M2-730).

Payload SHA-256
`e9a79a1d12a274a9526b48938dbb477546e50ebceffd5167e0241838c62e286f`
(89,600 bytes) passes 75 regressions and native preflight of all 1,850 CPU
templates plus 161 gate cases. The CPU logs retain every descriptor/status
marker, use at most 429 of 512 entries and drop no details. Four compact-map
controls also pass. CPU preflight explicitly uses
`nand-command-reference-v1`, including synthetic parameter/features/alternate-ID
responses; it does not establish optional Hynix support or physical timing.
There is no post-fix device capture in this evidence set (M2-730).

## 14.28 P49 bundle

Capture m2-p49: `m2_cpu_direct` (1,850 CPU jobs in resident CPU batches,
0 required-control failures, every reset verified; M2-731..M2-741) and `m2_r6_gate5` (five completed cases: baseline and gates0..3;
the following `BATCH_FREE` remains QUEUED, jobs done/started 62/63; M2-742).


All five gate records contain exact PWRCON restoration and successful reset/cleanup.
The finite probe TIMEOUT is distinct from the host timeout on FREE. USB status
polling remains live; gate4 is not submitted. Before/after VIC0 raw is `0x20`,
VIC1 raw zero; no interrupt sample is taken while clocks are disabled.
The physical
cause of the dispatch stall is not established. A native fault injection
(timer IRQ8 masked, non-idle thread current) reproduces queued FREE on the old
image and completes on the new direct worker dispatch path, which also adds
live timeout diagnostics (M2-743).

## 14.29 P50: clock-gate scan

Capture m2-p50 (964,738 bytes), SHA-256
`25570c99ced692b925d0a6b2318e9cd7c6a89ecb9dc7f087a79039224cc5ec11`:
`m2_r6_gate5`, 110 cases (baseline and gates0..108), all END, worker rc0,
initial reset and cleanup successful, exact restoration. Of the gate entries,
67 were enabled and 42 already off. All have VIC0 raw0x20 / VIC1 raw0 before
and after restoration; during-pulse interrupt fields are unavailable, not
measurements. No gate identity follows (M2-744).

USB is lost submitting the following RUN; the retained status is allocation
job331 DONE, not the failed RUN. Gate109 is the next intended case by order,
but acceptance, execution and restoration are unknown (M2-745). The image
hash is absent. The subsequently supplied raw journal verifies all 110 result
records and confirms the gate109 plan (see the following section). The five shared p49/p50 cases change from
TIMEOUT to END because the observation floor rises from 3,000 to 160,000 µs;
p50's loop ends in 13,525–13,548 µs. The queued FREE stall after gate3 does
not recur.

P51 (capture m2-p51, 36,531 bytes, SHA-256 `c386c0f055942e1e9af17657c52c211cdcb03705cda16f977d0c978faefb2a4d`, with its raw journal): baseline and gates 0..2 return END with exact restoration; USB is lost submitting the next RUN, whose last journal `batch-plan` names gate 3, a case that completed in p50 (M2-745).


## 14.30 P51 and the p50/p51 journals

P51 (capture m2-p51 and its raw journal) returns four cases:
baseline and gates0..2, all END with successful reset/cleanup and exact clock
restoration. It loses USB submitting the next RUN. The journal records the
gate3 plan and allocation13 DONE; no acceptance or RUNNING status follows
(M2-746). The p50 journal additionally confirms gate108 FREE330 DONE and the
gate109 plan preceding its loss (M2-745).

All 110 p50 and four p51 journal case records equal their summary records;
all raw request/output hashes verify. The five shared manifests are identical,
including gate3. P50 reports the EXECFIRMWARE entry marker; p51 does not. That
is the only capability difference. No image hash or CPU cache/MMU state was
recorded, so the entry-history difference does not establish causality
(M2-747).

The next 53-case family collects a pre-start checkpoint over USB and records
running-code identity; native packet/cache tests and preflight pass, without
modelling physical gate effects (M2-748).


## 14.31 Shared-driver native comparison

The resident comparison executes the same page/block API with CPU and
M2 physical backends. BCH8 and BCH16 native tests cover four CEs, cross-writes,
wrong-page detection and failure handling; the public reader's fallback and
boot-page semantics are shared. Per-operation timings exclude USB and byte
verification. These are native validation results, not device speed measurements
(M2-749).


## 14.32 P53 shared-driver measurements and gate checkpoint

P53 completes all 2,328 shared-driver operations and 4,405 required checks, with
zero mismatches. Independent reconstruction checks every read CRC and metadata
value. Programmed-page reads measure 10.7939 MiB/s M2 versus 8.9343 MiB/s CPU;
programs measure 2.8030 versus 2.5798 MiB/s (M2-750, M2-751). The first erase
on each CE is slow regardless of the alternating backend order (M2-752).

The subsequent gate campaign returns baseline and gates110..159, 51 END cases.
Gate3 is accepted, observed RUNNING, and produces pre-start checkpoint token52.
USB is lost during its ACK submission; pulse completion is unobserved.
Gate109 is never submitted. The reload entry marker is present (M2-753).

The CPU read optimization is separately qualified by native BCH8/BCH16 tests,
corrected/error paths, phase accounting and actual USB packet/worker transfers.
The baseline remains available in the same resident workload, with instrumented
and plain read controls. This is functional validation, not a measured speedup
(M2-754).
