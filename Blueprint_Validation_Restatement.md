# BrickFlow Blueprint — Validation and Restatement

This document applies the following operations to the BrickFlow Blueprint in strict accordance
with the problem statement:

1. Verify internal consistency
2. Identify any ambiguous or conflicting rules
3. Identify any undefined states
4. Identify any circular dependencies
5. Identify any execution-order hazards
6. Identify any conditions that cannot be evaluated deterministically in NinjaTrader
7. Identify any rules that require clarification before implementation
8. Produce a clean, section-by-section restatement using the same structure, without altering
   meaning or intent

No logic has been added, removed, simplified, or altered. No improvements, new indicators,
code, pseudocode, visuals, or examples have been introduced.

---

## PART I — VALIDATION FINDINGS

---

### A. INTERNAL CONSISTENCY

**IC-1 — Correction-count condition duplicated across Section 1 and Section 2**

Section 1 defines a valid uptrend as requiring that "the current bearish correction count is 0,
1, or 2 completed bearish 34/5 bricks." Section 2 restates this as a condition for longs to be
allowed: "The current 34/5 correction is not more than 2 bearish bricks." These two conditions
appear to state the same rule. The same duplication exists symmetrically for the downtrend and
short-allowed conditions. If one is updated in isolation during implementation the two sections
will diverge. Clarify whether they are intentionally redundant or whether one is meant to be the
master definition from which the other is derived.

**IC-2 — Maximum eligible pullback sequences stated in two places**

Section 2 blocks trading when "the setup would be the 3rd or later eligible 13/2 pullback
sequence inside the same 34/5 active leg." Section 7 states "Maximum 2 eligible 13/2 pullback
sequences per 34/5 active leg." These appear to state the same rule in two separate sections.
Confirm whether they are intentionally redundant.

**IC-3 — "Highest high of that pullback" in Section 6 is ambiguous relative to Section 4's
explicit wick language**

Section 4 uses explicit language: the trigger brick must "close above the highest wick high of
the full pullback sequence." Section 6 uses "closes above the highest high of that pullback" and
"1 tick below that confirmed higher-low pullback" without specifying wick or close. The
Definitions section states "All trend/structure comparisons use closes unless a rule explicitly
says wick." If "highest high" in Section 6 is intended to mean wick high, the rule must say so
explicitly, matching Section 4's phrasing. If it means close, the rule contradicts the pattern
established in Section 4. This must be resolved before implementation.

**IC-4 — Structural invalidation condition appears in three separate sections without a defined
evaluation order**

The condition "a completed 34/5 brick closes below the most recent confirmed 34/5 swing low"
appears in:

- Section 1, as a condition making the regime invalid/no-trade.
- Section 2, as a condition that disables longs (implied by the filter deactivating).
- Section 5, as a structural invalidation event requiring an immediate exit.

These three appearances must fire in a consistent and specified order: the exit of any open
position must occur before the regime state and filter states are updated. This sequencing
dependency is not made explicit anywhere in the blueprint.

---

### B. AMBIGUOUS RULES

**AM-1 — "Of the last 3 bullish/bearish impulse bricks" — follow-through brick inclusion**

Section 3 states "Of the last 3 bullish impulse bricks, at least 2 must have no lower wick."
The impulse sequence consists of at least 3 consecutive bullish bricks (including the breakout
brick that crosses the confirmed swing high) followed by one follow-through brick. It is not
stated whether "last 3" is counted from the end of the pre-follow-through impulse or from the
end of the full sequence including the follow-through brick. The two interpretations may
identify different bricks. This must be specified before implementation.

**AM-2 — "The 13/2 breakout brick" in the pullback-floor condition**

Section 3 requires the pullback's lowest wick low to remain "above the close of the 13/2
breakout brick." The impulse sequence includes a breakout brick (the brick whose close first
crosses the prior confirmed 13/2 swing high) and a follow-through brick. The phrase "13/2
breakout brick" must be confirmed as referring specifically to the first brick whose close
crosses the prior confirmed swing high, not the follow-through brick. These two bricks have
different close prices and would produce different floor levels.

**AM-3 — "The brick that created the confirmed swing high/low"**

The Definitions section defines Extreme swing high/low as "the wick high/low of the brick that
created the confirmed swing high/low." The confirmed swing high is the highest close of a
bullish same-color run of at least 2 bricks. If multiple bricks in that run share the same
highest close, the identity of "the brick that created" is undefined. Clarify whether this
refers to the first brick to reach that close level, the last brick to reach that close level,
or some other selection rule.

**AM-4 — "Highest high of that pullback" in Section 6 structural trailing (see also IC-3)**

As noted in IC-3, the phrase "highest high of that pullback" in the long-runner trailing
condition and "lowest low of that pullback" in the short-runner trailing condition do not
specify wick or close. Per the Definitions section, the default is close unless explicitly
stated otherwise. If the intent is wick high/low (which would be consistent with Section 4's
entry trigger logic), the rule must say so explicitly.

**AM-5 — "Most recent confirmed 13/2 swing low" in the Section 3 pullback-floor condition**

Section 3 requires the pullback's lowest wick low to remain "above the most recent confirmed
13/2 swing low." It is not specified whether this refers to the confirmed 13/2 swing low that
existed immediately before the current impulse began, or the most recently confirmed 13/2 swing
low at the moment each pullback brick is checked. These could be different levels if a new swing
low was confirmed during or adjacent to the impulse. Clarify which swing low is referenced.

**AM-6 — "That confirmed higher-low pullback" and "that confirmed lower-high pullback" in
Section 6 structural trailing**

Section 6 states the long-runner trailing stop is set to "1 tick below that confirmed
higher-low pullback" and the short-runner trailing stop is set to "1 tick above that confirmed
lower-high pullback." The phrase "that confirmed higher-low pullback" does not specify whether
the reference point is the lowest wick low or the lowest close of the pullback that produced the
confirmed higher low. The same ambiguity applies symmetrically to "that confirmed lower-high
pullback." Per the Definitions section, the default is close unless explicitly stated otherwise,
but given that stops elsewhere in the blueprint (Section 5) are placed 1 tick from wick
extremes, the intended reference must be confirmed explicitly.

---

### C. UNDEFINED STATES

**UD-1 — State of an active leg after structural invalidation and before a new break of
structure**

If a valid uptrend is active with an open long position, and structural invalidation occurs
(e.g., a 3-brick bearish correction on the 34/5 chart), Section 1 declares the regime
invalid/no-trade. Section 5 requires the long to be exited on the next tick. After the
invalidation resolves and the 34/5 chart reasserts directional movement, it is not stated in a
single consolidated location whether a new fresh 34/5 break of structure is required to begin a
new active leg before any new setup is eligible. The blueprint's definition of "34/5 active leg"
implies a new BOS is required, but this is not stated explicitly as the rule governing the
transition out of the invalid state.

**UD-2 — Pullback sequence count at the start of a new 34/5 active leg**

The 2-sequence cap in Sections 2 and 7 presupposes that pullback sequences are being counted
within each active leg. The blueprint does not explicitly state that the count initializes to
zero at the moment a fresh 34/5 break of structure occurs. Confirm that the counter resets to
zero at each new break of structure.

**UD-3 — Swing count initialization after each session reset**

Section 1 states that "session has reset and a new 34/5 trend structure has not yet formed" is
a no-trade regime condition. The blueprint does not state explicitly whether the counts of
confirmed swing highs and swing lows reset to zero at each Break at EOD reset. If they do
reset, then after each session open the system must accumulate 2 confirmed 34/5 swing highs and
2 confirmed 34/5 swing lows within the new session before any trade is eligible. If they do not
reset, prior-session swings may satisfy the count requirement. Confirm which behavior is
intended.

---

### D. CIRCULAR DEPENDENCIES

**CD-1 — None identified**

Confirmed swings are determined directly from completed price data. Trends are determined from
confirmed swings. Breaks of structure are determined from confirmed swings. Active legs begin at
a break of structure. No rule's evaluation depends on itself.

---

### E. EXECUTION-ORDER HAZARDS

**EO-1 — Multi-series bar synchronization**

The strategy uses two data series (13/2 and 34/5). NinjaTrader processes bar-close events
independently per series via OnBarUpdate. A 34/5 brick and a 13/2 brick may close at the same
tick. The order in which NinjaTrader fires OnBarUpdate for each series in that scenario is not
guaranteed. If a 34/5 regime change and a 13/2 trigger close arrive at the same tick, the
system may evaluate the 13/2 trigger against a stale 34/5 regime state. The implementation
must explicitly check the 34/5 state against the most recently completed 34/5 brick before
validating any 13/2 trigger condition, regardless of which series fired the event.

**EO-2 — Partial-exit and stop-movement sequencing**

Section 6 requires: (1) detect that +1.5R has been reached, (2) exit 50% of the position, (3)
move the stop on the remaining 50% to breakeven. If step 3 is processed before step 2 has been
confirmed as filled, the stop may be moved while the full position is still open. The required
fill-confirmation sequence must be made explicit in the implementation.

**EO-3 — Structural exit and trailing stop on the same tick**

If both a structural invalidation condition (Section 5) and a trailing stop trigger (Section 6)
fire on the same tick, both will attempt to close the remaining position. The structural exit
must take priority and the trailing stop order must be cancelled or rendered inactive. The
priority ordering must be specified explicitly before implementation.

**EO-4 — "Next tick after close" for entry submission**

Entry is to be submitted on "the next tick after that close." NinjaTrader's tick-data feed can
deliver multiple ticks carrying the same timestamp. The blueprint does not define whether the
target tick is the first tick whose timestamp is strictly greater than the close-event
timestamp, or the first tick that NinjaTrader processes after the bar-close event fires. This
must be resolved to produce a deterministic entry.

**EO-5 — Stop-may-never-widen rule and the breakeven move**

Section 5 states "Stop may never widen." Section 6 states that after the partial exit, the stop
on the remaining 50% moves to breakeven. Moving to breakeven is always a tightening of the
stop, not a widening, provided the actual filled entry price is used as the reference. However,
if slippage results in a fill price that differs from the anticipated entry price, the definition
of breakeven changes. The blueprint does not specify whether "breakeven" is the actual filled
entry price or the theoretical entry price. This must be resolved before implementation. See
also RC-7.

---

### F. CONDITIONS THAT CANNOT BE EVALUATED DETERMINISTICALLY IN NINJATRADER

**DT-1 — "No lower wick" and "no upper wick" without reference to Open**

The blueprint states that "no open-based rule" is used because ninZaRenko free-version opens
are artificial. In conventional bar arithmetic a bullish brick has no lower wick when its Low
equals its Open — that conventional definition is stated here for context only; it cannot be
used in this implementation because the Open is excluded by design. For the condition "no lower
wick" to be evaluable without the Open, it must be defined in terms of Low and Close only. The
deterministic definition must be: a bullish brick has no lower wick if and only if Low == Close.
Symmetrically, a bearish brick has no upper wick if and only if High == Close. Confirm that
these are the intended definitions.

**DT-2 — Simultaneous brick closes on both series**

As noted in EO-1, if a 13/2 brick and a 34/5 brick close at the same tick, NinjaTrader fires
OnBarUpdate for both series. The order of those calls is indeterminate. When the 13/2
OnBarUpdate is processing a trigger condition and simultaneously reads the 34/5 regime state,
the 34/5 state may or may not yet reflect the simultaneously closed 34/5 brick. This is a
determinism hazard that must be addressed explicitly in implementation.

**DT-3 — Break at EOD reset timing relative to force-flat**

The blueprint specifies force flat at 14:55 CT and Break at EOD = True on both charts. The
exact tick at which NinjaTrader applies a Break at EOD session reset and the order of operations
relative to the force-flat logic at 14:55 CT is implementation-dependent. It is not stated
whether the force-flat event and the Break at EOD reset are the same event, successive events,
or independent events. The interaction must be resolved to ensure positions are always exited
before data is reset.

---

### G. RULES REQUIRING CLARIFICATION BEFORE IMPLEMENTATION

**RC-1 — Wick-cleanliness count: does "last 3 impulse bricks" include the follow-through?**
(See AM-1.) Specify whether the wick-cleanliness check covers the 3 bricks immediately
preceding the follow-through brick, or whether the follow-through brick is counted as one of the
3.

**RC-2 — Define "13/2 breakout brick" as the breakout brick or the follow-through brick**
(See AM-2.) Confirm that "13/2 breakout brick" in the pullback-floor condition refers to the
first brick whose close crosses the prior confirmed 13/2 swing high, not the follow-through
brick.

**RC-3 — Specify wick or close for "highest high of that pullback" in Section 6**
(See IC-3 and AM-4.) The phrase must be made explicit: wick high or close.

**RC-4 — Define the identity of the brick that "created" a confirmed swing high/low when there
is a tie**
(See AM-3.) If multiple bricks in a same-color run share the highest close, specify which
brick's wick is used as the extreme swing level.

**RC-5 — Specify which "most recent confirmed 13/2 swing low" is used as the pullback floor**
(See AM-5.) Confirm whether this is the confirmed 13/2 swing low that existed immediately before
the current impulse began or the most recently confirmed swing low at the moment each pullback
brick is evaluated.

**RC-6 — Confirm that "no lower wick" means Low == Close for a bullish brick**
(See DT-1.) Given the prohibition on open-based rules, this definition must be confirmed
explicitly.

**RC-7 — Confirm that "breakeven" means the actual filled entry price**
(See EO-5.) If slippage is possible, confirm that the breakeven stop is anchored to the actual
fill price, not a theoretical entry price.

**RC-8 — Confirm that confirmed swing counts reset to zero on each session reset**
(See UD-3.) Specify whether the counts of confirmed 34/5 swing highs and swing lows restart
from zero at each Break at EOD reset.

**RC-9 — "Closes back through" in Section 1 invalid-regime condition: strictly beyond or
touching**
Section 1 states the regime becomes invalid when "price closes back through the most recent
confirmed 34/5 swing low in an uptrend." The Definitions section defines break of structure as
"a completed brick close beyond the prior confirmed swing high/low." It is not stated whether
"closes back through" in the invalidation condition uses the same strictly-beyond semantics or
includes an exact-match (close equals swing level) as invalid. Confirm.

**RC-10 — Confirm whether "that confirmed higher-low / lower-high pullback" in Section 6
trailing refers to wick low/high or close**
(See AM-6.) Section 5 places initial stops 1 tick from wick extremes of the pullback. Confirm
whether the Section 6 structural trailing stop is similarly anchored to the wick extreme (lowest
wick low for long runner, highest wick high for short runner) of the trailing pullback, or to
the close.

---

## PART II — CLEAN RESTATEMENT

The following is a section-by-section restatement of the BrickFlow Blueprint. The structure,
logic, thresholds, counts, and definitions are reproduced without alteration. Informal
conversational asides contained in the original are preserved and marked with brackets. No
meaning or intent has been changed.

---

### Locked Implementation Assumptions

Use tick data only on both series.

Interpret ninZaRenko pairs as Brick Size / Trend Threshold: execution chart = 13/2, structure
chart = 34/5. The mirrored trader manual describes that notation and also describes Break at EOD
as the default/recommended intraday setting.

The free ninZaRenko bar prints real wick highs/lows, but the vendor separately states the free
version uses artificial opens; KingRenko shares the same settings and identical close prices
when matched. For that reason, BrickFlow uses completed-brick closes for structure and completed
wick extremes for stops/triggers, and uses no open-based rule.

Session template is fixed to CME US Index Futures RTH.

Break at EOD = True on both charts.

No EMA/ZLEMA filter is used.

All logic is closed-brick only. The forming brick is ignored.

---

### Definitions

**Completed brick** — any fully closed brick.

**Same-color run** — consecutive completed bricks of the same direction.

**Confirmed swing high** — the highest close of a bullish same-color run of at least 2 bricks,
confirmed only when the first bearish brick closes after that run.

**Confirmed swing low** — the lowest close of a bearish same-color run of at least 2 bricks,
confirmed only when the first bullish brick closes after that run.

**Extreme swing high/low** — the wick high/low of the brick that created the confirmed swing
high/low.

**Break of structure** — a completed brick close beyond the prior confirmed swing high/low.

**34/5 active leg** — the period after a fresh 34/5 break of structure until invalidation.

**13/2 pullback sequence** — a 1-brick or 2-brick countertrend correction with no interleaved
same-direction brick.

All trend/structure comparisons use closes unless a rule explicitly says wick.

---

### Section 1 — Market Regime Definition (34/5)

**Valid uptrend** — all of the following must be true:

- At least 2 confirmed 34/5 swing highs and 2 confirmed 34/5 swing lows exist.
- The latest confirmed 34/5 swing high is above the prior confirmed 34/5 swing high.
- The latest confirmed 34/5 swing low is above the prior confirmed 34/5 swing low.
- A completed bullish 34/5 brick has already closed above the most recent confirmed 34/5 swing
  high.
- Since that fresh bullish break, the current bearish correction count is 0, 1, or 2 completed
  bearish 34/5 bricks.

**Valid downtrend** — all of the following must be true:

- At least 2 confirmed 34/5 swing highs and 2 confirmed 34/5 swing lows exist.
- The latest confirmed 34/5 swing high is below the prior confirmed 34/5 swing high.
- The latest confirmed 34/5 swing low is below the prior confirmed 34/5 swing low.
- A completed bearish 34/5 brick has already closed below the most recent confirmed 34/5 swing
  low.
- Since that fresh bearish break, the current bullish correction count is 0, 1, or 2 completed
  bullish 34/5 bricks.

**Invalid / no-trade regime** — any one of the following is sufficient:

- Fewer than 2 confirmed swing highs or swing lows exist.
- Latest swings are structurally mixed: higher high with lower low, or lower high with higher
  low.
- The last 6 completed 34/5 bricks contain no same-color run of at least 3 bricks.
- Current countertrend correction reaches 3 consecutive opposite-color 34/5 bricks.
- Price closes back through the most recent confirmed 34/5 swing low in an uptrend or through
  the most recent confirmed 34/5 swing high in a downtrend.
- Session has reset and a new 34/5 trend structure has not yet formed.

---

### Section 2 — Trade Direction Filter

**Longs allowed only when all of the following are true:**

- 34/5 regime is a valid uptrend.
- A fresh 34/5 bullish break of structure has already occurred.
- The current 34/5 correction is not more than 2 bearish bricks.
- No completed 34/5 brick has closed below the most recent confirmed 34/5 swing low.
- All Section 7 risk/time filters are open.

**Shorts allowed only when all of the following are true:**

- 34/5 regime is a valid downtrend.
- A fresh 34/5 bearish break of structure has already occurred.
- The current 34/5 correction is not more than 2 bullish bricks.
- No completed 34/5 brick has closed above the most recent confirmed 34/5 swing high.
- All Section 7 risk/time filters are open.

**Block all trades when any of the following is true:**

- Neither long nor short filter is active.
- 34/5 is in transition, compression, or a 3-brick countertrend correction.
- Daily trade cap or daily loss cap is hit.
- Time window is closed.
- A position is already open in the opposite direction.
- The setup would be the 3rd or later eligible 13/2 pullback sequence inside the same 34/5
  active leg.

---

### Section 3 — Momentum Qualification (13/2)

**Long momentum qualification** — all of the following must be satisfied in sequence:

- Must occur during an active 34/5 long leg.
- A bullish 13/2 impulse must print at least 3 consecutive bullish bricks.
- That impulse must break above the most recent confirmed 13/2 swing high by close.
- After that breakout close, one additional bullish 13/2 close must occur before any pullback
  begins.
- Of the last 3 bullish impulse bricks, at least 2 must have no lower wick.
- The pullback that follows must be exactly 1 or 2 consecutive bearish bricks.
- The lowest wick low of that pullback must remain:
  - above the close of the 13/2 breakout brick,
  - above the most recent confirmed 13/2 swing low,
  - above the most recent confirmed 34/5 swing low.
- If a 3rd bearish 13/2 brick prints, momentum qualification is lost.
- If a bullish brick appears before the trigger but the pullback rules are no longer intact, the
  setup resets.

**Short momentum qualification** — all of the following must be satisfied in sequence:

- Must occur during an active 34/5 short leg.
- A bearish 13/2 impulse must print at least 3 consecutive bearish bricks.
- That impulse must break below the most recent confirmed 13/2 swing low by close.
- After that breakout close, one additional bearish 13/2 close must occur before any pullback
  begins.
- Of the last 3 bearish impulse bricks, at least 2 must have no upper wick.
- The pullback that follows must be exactly 1 or 2 consecutive bullish bricks.
- The highest wick high of that pullback must remain:
  - below the close of the 13/2 breakout brick,
  - below the most recent confirmed 13/2 swing high,
  - below the most recent confirmed 34/5 swing high.
- If a 3rd bullish 13/2 brick prints, momentum qualification is lost.
- If a bearish brick appears before the trigger but the pullback rules are no longer intact, the
  setup resets.

**Brick speed / expansion logic:**

- Momentum is defined by uninterrupted directional printing, not time.
- The required breakout brick plus one follow-through brick is the minimum expansion requirement.
- Any impulse that breaks structure but does not print a follow-through brick is not qualified.

**Wick and overlap rules:**

- Long impulse: at least 2 of last 3 impulse bricks must have no lower wick.
- Short impulse: at least 2 of last 3 impulse bricks must have no upper wick.
- A qualified pullback may contain only countertrend bricks.
- Any color alternation inside the pullback sequence = overlap/chop = no trade.

---

### Section 4 — Entry Logic

**Long entry:**

- Wait for a qualified long pullback sequence from Section 3.
- Only the first bullish brick after that pullback may act as the continuation trigger.
- That trigger brick must:
  - close above the highest wick high of the full pullback sequence,
  - be bullish,
  - have no lower wick,
  - close while the 34/5 long filter remains valid.
- Entry timing:
  - signal is confirmed only at the trigger brick close,
  - submit the long entry on the next tick after that close,
  - no intrabar anticipation,
  - no entry at a projected next-bar open.
- If that first bullish brick fails any trigger condition, the setup expires and a new 13/2
  impulse must form.

**Short entry:**

- Wait for a qualified short pullback sequence from Section 3.
- Only the first bearish brick after that pullback may act as the continuation trigger.
- That trigger brick must:
  - close below the lowest wick low of the full pullback sequence,
  - be bearish,
  - have no upper wick,
  - close while the 34/5 short filter remains valid.
- Entry timing:
  - signal is confirmed only at the trigger brick close,
  - submit the short entry on the next tick after that close,
  - no intrabar anticipation,
  - no entry at a projected next-bar open.
- If that first bearish brick fails any trigger condition, the setup expires and a new 13/2
  impulse must form.

---

### Section 5 — Stop Placement

**Long stop logic:**

- Initial stop = 1 tick below the lowest wick low of the 13/2 pullback sequence that produced
  the entry.
- If that pullback low is at or below the most recent confirmed 34/5 swing low, the trade is
  invalid before entry.
- After entry, stop may only remain where it is or ratchet tighter. Stop may never widen.
- Structural invalidation for any open long:
  - a completed 34/5 brick closes below the most recent confirmed 34/5 swing low, or
  - the 34/5 chart prints a 3rd consecutive bearish correction brick.
- On structural invalidation, exit any remaining long on the next tick.

**Short stop logic:**

- Initial stop = 1 tick above the highest wick high of the 13/2 pullback sequence that produced
  the entry.
- If that pullback high is at or above the most recent confirmed 34/5 swing high, the trade is
  invalid before entry.
- After entry, stop may only remain where it is or ratchet tighter. Stop may never widen.
- Structural invalidation for any open short:
  - a completed 34/5 brick closes above the most recent confirmed 34/5 swing high, or
  - the 34/5 chart prints a 3rd consecutive bullish correction brick.
- On structural invalidation, exit any remaining short on the next tick.

---

### Section 6 — Profit Management

- Define 1R as the distance from actual filled entry price to initial stop price.
- Position must be divisible into 2 equal parts.

**Partial exit:**

- Exit 50% at +1.5R.

**Runner logic:**

- After the +1.5R partial fills, move stop on the remaining 50% to breakeven.
- The remainder is the runner.

**Structural trailing:**

Long runner:

- Trail to 1 tick below each newly confirmed 13/2 higher low formed after entry.
- A new higher low is confirmed only after:
  - a 1-brick or 2-brick bearish pullback forms, then
  - a bullish brick closes above the highest high of that pullback.
- New stop = the tighter of:
  - current stop,
  - 1 tick below that confirmed higher-low pullback.

Short runner:

- Trail to 1 tick above each newly confirmed 13/2 lower high formed after entry.
- A new lower high is confirmed only after:
  - a 1-brick or 2-brick bullish pullback forms, then
  - a bearish brick closes below the lowest low of that pullback.
- New stop = the tighter of:
  - current stop,
  - 1 tick above that confirmed lower-high pullback.

**Full exit conditions** — exit any remainder when:

- structural trail stop is hit,
- 34/5 structural invalidation occurs,
- session flat time is reached.

---

### Section 7 — Risk Controls

- Session template: CME US Index Futures RTH.
- Allowed entry windows: 08:35–11:30 CT and 13:00–14:45 CT.
- No new positions outside those windows.
- Force flat at 14:55 CT.
- Maximum 3 trades per session.
- Maximum 2 losing trades per session.
- Stop trading for the day at -2R net realized.
- Maximum 2 eligible 13/2 pullback sequences per 34/5 active leg.
- No add-ons.
- No pyramiding.
- No reversal-in-place.
- After a stop-out, the next trade must come from a new qualified 13/2 impulse and new qualified
  pullback.
- If a trade is open and a force-flat condition occurs, exit on the next tick.

---

### Section 8 — Failure Modes

**Common false signals:**

- 13/2 breaks a local swing but prints no follow-through brick.
- Pullback retraces back into the 13/2 breakout brick close.
- Pullback becomes 3 countertrend bricks instead of 1–2.
- First resumption brick after pullback fails to clear the entire pullback extreme.
- First resumption brick prints an opposite-side wick.
- Entry appears on the 3rd or later eligible pullback sequence of the same 34/5 leg.

**Structural conditions that degrade edge:**

- 34/5 has no confirmed HH/HL or LL/LH sequence.
- 34/5 last 6 bricks lack any 3-brick same-color run.
- 34/5 is already in a 3-brick countertrend correction.
- 13/2 pullback overlaps with alternating colors before trigger.
- 13/2 pullback reaches the most recent 34/5 swing level.

**Explicit do-not-trade scenarios:**

- No confirmed 34/5 trend after session reset.
- Mixed 34/5 structure.
- 13/2 impulse without breakout follow-through.
- 13/2 pullback deeper than allowed.
- Trigger brick not clean enough.
- Daily loss cap hit.
- Daily trade cap hit.
- Outside time windows.
- Any open trade in the opposite direction.

**Tweaks / additions that should be locked before coding:**

- Keep tick data fixed. That is required for the non-repainting intent of this contract.
- Keep session template and Break at EOD fixed. Without that, swing counts and daily resets
  change. The mirrored trader manual describes Break at EOD as default/recommended for intraday
  use.
- Do not use ninZaRenko opens in the strategy. The vendor states the free bar uses artificial
  opens; the close-based/open-agnostic design above is intentional.
- [Note in original: "Your chosen pairs are structurally usable but unconventional relative to
  the mirrored manual's published best-practice note that Brick Size should ideally be a
  multiple of Trend Threshold. I left them unchanged because you fixed them in the prompt." This
  is an informal conversational note, not a rule. It has been preserved verbatim within
  brackets. No pre-implementation decision is required from it beyond accepting the 13/2 and
  34/5 pairs as fixed.]
- If partial exits are mandatory, live size must be at least 2 contracts. If live size will be
  1 contract, the exit model must be changed before implementation.
