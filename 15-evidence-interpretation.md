# Chapter 15 — Interpreting M2 evidence


Use the instruction-level capture before the human-readable legacy summary.
Some report labels and comments came from M1 hypotheses. An experiment called
“illegal opcode”, “one-shot”, “conditional wait” or “timer” does not establish
that interpretation on M2.


## 15.1 Required distinctions

| Evidence | Valid conclusion | Invalid shortcut |
|---|---|---|
| Recorded manifest | the program was planned/submitted | every instruction ran |
| Worker completed with rc -2 | bounded native path failed; inspect capture | all output is invalid, or all output is a hardware result |
| Zero samples, pre_ok=0 | candidate execution not established | opcode ended instantly; register reads are all zero |
| Sampled TIMEOUT | no requested completion during the host interval | illegal instruction or infinite hardware stall |
| Fault followed by END | fault path completed | clean success |
| END with expected sentinel | that path reached sentinel/end | all fields/operands have confirmed semantics |
| `late_at_us=0` | scheduled write was not recorded as delivered | candidate write had no effect |
| Two operands alias | those operands agree in that context | all high bits are ignored |
| Source uses one register word | that word is used for that operation | each bit is independently understood |
| Model passes | model/code contract holds | physical silicon is validated |

p06 illustrates all of these distinctions. The first `pause` (`12`) sample is useful
even though cleanup fails. Later opcode sweeps are inconclusive even though USB
and the worker keep delivering records. Failure continuation is valuable only
if the analysis preserves this difference.


## 15.2 Order and state contamination

Treat experiments as a stateful sequence. A pause or timeout can leave state
that reset does not clear. p06 recovers between `op12` and `startseq2`, then loses
sampled execution permanently after the first `op12wait` case. A simulator that
boots fresh for each family cannot discover this by construction; include
persistent-session/native-batch tests as well as isolated instruction tests.

Full-map zeros are especially ambiguous: the capture may never have been filled,
the snapshot window may have been skipped, or a consuming port may be represented
by a placeholder. Do not turn such values into an access-mask table.


## 15.3 M1 comparison discipline

The M1 book supplies the questions and candidate programs. M2 has its own CPU
reset/start handshake, ECC/status layout and source-defined WAIT12 status path.
The tested indirect instructions `pld` (`18`) and `pst` (`19`) exist on M2 despite old “3G only” descriptions.
Conversely, M1's JR/SIG/WAIT2/EVJMP/NOP2, timer scaling, one-shot behavior, mirrored
channel map, power-on defaults and RS selection were unconfirmed on M2 at p06.
Their later M2 measurements and differences from M1 are in
[chapter 19](19-m1-m2-differences.md); the power-on values are in
[chapter 8, section 8.21](08-sequencer-control-registers.md#821-reset), and what is
still open is in [chapter 17](17-open-questions.md).

Distinguish a physical observation from the smallest explanatory model. In the
p06 simulator update, pause IRQ/state/latches are measured; STOP preserving that
pause is an inferred failure scenario consistent with the native return sequence.
Neither establishes all reset paths or an automatic recovery algorithm.

## 15.4 P16 examples of interpretation boundaries

P16 is a complete execution attempt with one known comparison mismatch, not
an incomplete run. Its source-10 mismatch deliberately retains M1's prediction;
64 dedicated M2 source-10 controls match the measured alternative.

C58=0 programs can acknowledge their own timeout IRQ and then return END.
Stored before/after IRQ words distinguish that from a wait that passed.
Likewise, all 24 p16 DMA/ECC programs reach END but fail payload comparison;
their guarding WAIT2 tested an unacknowledged flag ([section 9.5](09-nand-controller.md#95-dma)). Neither an END
count nor RESULT 80c=0 proves complete DMA or a valid codeword, and a passing
WAIT proves an event only if its flag was clear before the stimulus.
WAIT2's initial VIC snapshot can miss a later transient; snapshot disagreement
alone does not establish a different interrupt mapping.

## 15.5 P43 examples of interpretation boundaries

The two `m2_cpu_stack` mismatches compare the 0x820 location list with the
injected locations in ascending order; the sets are equal and the engine lists
the bits of one byte higher first (M2-714). A check that fixes an order the
hardware does not follow fails without any wrong data.

The boot-ROM words that `ldi` returns are identified in the ROM image, but
which earlier ROM access left each one is inferred (M2-683). The 20-byte beat
step of a DMA in with 038 = 0xf is inferred from the word order alone (M2-701).
One C08 read per run samples a loop at an uncontrolled phase, so differing
values between runs of one instruction kind are not a state difference
(M2-693). The BCH16 0x814 comparison first counted the 0x814 readback word as a
decode difference; the decode registers and data are identical (M2-709, M2-710).
