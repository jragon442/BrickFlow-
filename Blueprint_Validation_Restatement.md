# BrickFlow Blueprint — Validation & Clean Restatement

---

## PART I — VALIDATION FINDINGS

### Legend

| Prefix | Category |
|--------|----------|
| IC | Internal Consistency |
| AM | Ambiguous Rule |
| UD | Undefined State |
| CD | Circular Dependency |
| EO | Execution-Order Hazard |
| DT | NinjaTrader Determinism |
| RC | Must Clarify Before Implementation |

---

### IC — Internal Consistency

**IC-1 — Redundant but consistent: valid uptrend and Section 2 trade filter**
Section 1 Valid Uptrend includes "A fresh bullish 34/5 break of structure has occurred" as a condition of being in a valid uptrend. Section 2 Longs Allowed restates "A fresh 34/5 bullish break of structure has already occurred" as a separate gate. Both reference the same event. No conflict; redundancy is harmless.

**IC-2 — 6-brick compression window boundary is implicitly session-scoped**
Section 1 states the rolling window of last 6 completed 34/5 bricks is evaluated on every completed 34/5 brick and that "if fewer than 6 completed 34/5 bricks exist in the current session, this 6-brick compression test is treated as not passed." The phrase "current session" implies the brick count for this test resets at session open along with everything else. The blueprint does not explicitly confirm whether bricks from a prior session contribute to the rolling 6. This is implicitly consistent with the session-reset rule but is not stated outright. See RC-7.

**IC-3 — Section 6 trailing stop reference uses "highest high" without the word "wick"**
Section 4 (Long Entry trigger) explicitly requires the trigger brick to "close above the highest wick high of the full pullback sequence." Section 6 (Long Runner trailing) says "a bullish brick closes above the highest high of that pullback" to confirm a new higher low. The word "wick" is present in Section 4 but absent in Section 6 for the same conceptual reference. The Definitions state "All trend/structure comparisons use closes unless a rule explicitly says wick." Since Section 6 says "highest high" (not "highest wick high"), a strict reading under the Definitions would resolve "highest high" to the highest close, not the highest wick extreme. This is a potential contradiction with the intent of Section 4. See RC-3.

**IC-4 — Structural invalidation appears in three sections with no unified ordering rule**
Section 1 defines conditions that make the regime invalid. Section 2 blocks new entries when the regime is invalid. Section 5 defines structural invalidation for open trades and prescribes exit on the first subsequent tick. No blueprint rule unifies these three sections into a single evaluation-and-action sequence. When a 34/5 brick simultaneously satisfies Section 1 invalidation, Section 2 blocking, and Section 5 structural exit, the order in which those actions are applied is undefined. See EO-3.

**IC-5 — "Stop may never widen" is consistent with breakeven transfer rule**
Section 5 says after entry the stop may only remain where it is or ratchet tighter. Section 6 says after the +1.5R partial fills, move stop on the remaining 50% to breakeven. Moving to breakeven from the initial stop tightens the stop (assuming entry is above initial stop for a long). This is internally consistent: breakeven is numerically higher than the initial stop for a long, which satisfies "tighter means the numerically higher stop price."

**IC-6 — 34/5 active leg ending conditions are consistent with Section 1 invalidity conditions**
The 34/5 active leg ends when: session reset, close back through the most recent confirmed swing in the countertrend direction, or a 3rd consecutive countertrend 34/5 brick. Section 1 lists "Current countertrend correction reaches 3 consecutive opposite-color 34/5 bricks" and "Price closes back through the most recent confirmed 34/5 swing low in an uptrend..." as invalidity conditions. These are consistent: a leg-ending event and a regime-invalidity event are co-occurring.

---

### AM — Ambiguous Rules

**AM-1 — "The last 3 bullish/bearish impulse bricks" counting when the impulse run is exactly 3 bricks long**
The blueprint defines "the last 3 bullish impulse bricks" as "the 3 most recent bullish bricks of the impulse run immediately before the pullback begins, including the required follow-through brick." The minimum impulse run is at least 3 consecutive bullish bricks that break the swing high, followed by 1 follow-through brick — a minimum of 4 total bullish bricks. Therefore, the "last 3" always excludes the first impulse brick when the run is exactly 4 (minimum) bricks long. If the impulse run is longer, it still means the 3 most recent bullish bricks before the pullback, including the follow-through. This reading is internally consistent, but the phrase "including the required follow-through brick" implicitly confirms the follow-through is always counted as one of the 3. See RC-1.

**AM-2 — What constitutes "freshness" when a confirmed swing is immediately broken on the same tick it is confirmed**
A "fresh 34/5 break of structure" requires the confirmed swing to exist before the break occurs. If the same brick that confirms a swing (by being the first opposite-color brick after the run) is simultaneously the brick that closes beyond a different prior swing, the sequence of events matters. The blueprint does not address whether a swing can be confirmed and broken in the same evaluation pass.

**AM-3 — The eligible-pullback counter and multiple same-direction fresh breaks during one active leg**
The blueprint states "A later fresh same-direction 34/5 break that occurs before one of those leg-ending events does not start a new 34/5 active leg and does not reset the pullback-sequence counter." The counter therefore accumulates across the entire active leg, regardless of how many fresh same-direction breaks occur within it. Combined with the 2-pullback maximum, once 2 eligible sequences are counted, all further entries in that leg are blocked even if a new fresh break occurs. This is internally consistent but may need explicit confirmation from the author that this is the intended behavior.

**AM-4 — Section 6 trailing stop: "1 tick below that confirmed higher-low pullback" — reference price undefined**
The long runner trailing formula says: "New stop = the higher of: current stop, 1 tick below that confirmed higher-low pullback." The phrase "that confirmed higher-low pullback" does not specify which price of the pullback is used — the wick low of the pullback or the close of the last pullback brick. The Definitions say "All trend/structure comparisons use closes unless a rule explicitly says wick." However, this is a stop placement rule, not a structure comparison, and the analogous initial stop in Section 5 explicitly says "1 tick below the lowest wick low of the 13/2 pullback sequence." The trailing stop rule does not repeat "wick low" for the runner. See RC-10.

**AM-5 — "No reversal-in-place" combined with "A position is already open in the opposite direction" block**
Section 7 says "No reversal-in-place." Section 2 says block all trades when "A position is already open in the opposite direction." Both rules independently prevent entering a trade in the opposite direction of an open position. These rules are consistent but redundant. No conflict.

**AM-6 — Section 6 Short Runner: "1 tick above that confirmed lower-high pullback" — same reference price ambiguity as AM-4**
Mirrors the ambiguity in AM-4 for the short direction. See RC-10.

---

### UD — Undefined States

**UD-1 — 13/2 qualification state when a 34/5 active leg ends without a stop-out**
Section 7 states: "After any stop-out, any in-progress 13/2 impulse or 13/2 pullback qualification state is discarded." This covers one specific termination path. However, a 34/5 active leg can also end by: session reset, close back through the most recent confirmed swing (without a stop being hit), or a 3rd consecutive countertrend 34/5 brick (without a stop being hit). The blueprint does not explicitly state that the 13/2 qualification state is discarded in these non-stop-out leg-ending cases. Because the momentum qualification rule requires the impulse/pullback to occur "during an active 34/5 long/short leg," any in-progress state would implicitly become invalid when the leg ends. But the explicit discard instruction is absent. See RC-4.

**UD-2 — 13/2 qualification state after force-flat, daily cap, or time window closure (not a stop-out)**
When the daily trade cap, daily loss cap, or time window closure blocks new entries, the blueprint does not specify whether any in-progress 13/2 impulse or pullback state is discarded or retained. If a new entry window opens later in the session (e.g., 13:00 CT after the morning window closes), and the 13/2 state has been retained from before the window closed, it may or may not still be valid relative to the 34/5 active leg state. See RC-5.

**UD-3 — Runner stop logic when the +1.5R partial has not filled**
Section 6 states: "If the +1.5R partial has not filled when any full-exit condition occurs, the entire remaining position exits in full and the breakeven-transfer rule does not activate." This covers the exit condition. It does not address whether the trailing stop management for the runner begins before or only after the partial fills. The runner is defined as "the remainder" after the partial exit. If the partial has not filled, there is no separate runner yet — the full position is still active. The blueprint implies the trailing stop logic for the runner does not apply until after the partial fills, but this is not stated explicitly.

---

### CD — Circular Dependencies

**CD-1 — No circular dependencies detected**
The evaluation graph flows in one direction: completed bricks produce swing registrations, swing registrations produce break-of-structure events, break-of-structure events produce active legs, active legs gate the 13/2 qualification, qualification gates the entry, entry gates the stop and profit logic. No node in this graph depends on its own output. No circular dependencies are present.

---

### EO — Execution-Order Hazards

**EO-1 — Multi-series OnBarUpdate call order in NinjaTrader**
The blueprint prescribes: "If a 34/5 brick and a 13/2 brick complete on the same incoming tick: process the 34/5 close first, then process the 13/2 close, then evaluate any entry/exit action." In NinjaTrader, a multi-series strategy receives separate `OnBarUpdate` calls for each data series (identified by `BarsInProgress`). When both series close a brick on the same incoming tick, NinjaTrader fires the `OnBarUpdate` calls in the order the data series were registered via `AddDataSeries`. The blueprint's prescribed 34/5-first ordering must be enforced by registering the 34/5 series before the 13/2 series and verifying that the platform respects that ordering. If the platform reverses the order, the 34/5 regime state will be stale when the 13/2 trigger is evaluated on that tick. This must be tested explicitly before relying on it.

**EO-2 — "Updated 34/5 invalidation or block condition governs the 13/2 decision on that same tick"**
The blueprint states: "Any updated 34/5 invalidation or block condition governs the 13/2 decision on that same tick." This rule directly depends on EO-1 being correctly enforced. If 34/5 is processed first and its regime state is fully committed before 13/2 is evaluated, this rule is satisfied. If not, the 13/2 evaluation may use a stale 34/5 state, and a trade might be entered or blocked incorrectly.

**EO-3 — No defined evaluation-and-action sequence when multiple exit conditions trigger simultaneously**
Section 6 lists three full-exit conditions: structural trail stop is hit, 34/5 structural invalidation occurs, session flat time is reached. If two or more of these occur simultaneously (e.g., the trailing stop is at a price that is also a session flat time), the order in which they are processed is undefined. In practice, the outcome is the same (exit the position), but any state updates tied to each specific condition type may differ depending on which is processed first.

**EO-4 — Entry order type is not specified**
Sections 4 and 7 state that entry is submitted "on the first subsequent tick after that close" and that there is "no intrabar anticipation" and "no entry at a projected next-bar open." This describes the timing of the order submission but does not specify the order type (market, stop, limit). In NinjaTrader, the practical difference between a market order and a stop order placed at the last known price on the next tick must be determined before implementation.

**EO-5 — Structural invalidation and open-trade exit timing**
Section 5 says: "At the close of that invalidating 34/5 brick: the regime becomes invalid, new long/short entries are blocked immediately. Exit any remaining long/short on the first subsequent tick." The regime becomes invalid at the close (synchronously with the brick close), but the exit order is submitted on the next tick. Between the close of the invalidating brick and the first subsequent tick, the strategy has an open position in an invalid regime. Any new 13/2 brick that completes on the same tick as the invalidating 34/5 brick must still be processed under the new (invalid) regime state per EO-1/EO-2, but the exit has not yet been submitted. The blueprint's prescribed ordering (34/5 close → 13/2 close → entry/exit action) addresses this: the exit action is evaluated after both closes, and the regime state from the 34/5 close governs the 13/2 decision on that tick.

---

### DT — NinjaTrader Determinism

**DT-1 — "No lower wick" / "no upper wick" field mapping in NinZaRenko**
The blueprint defines: "No lower wick = the completed brick has zero lower shadow length as rendered by the chart series." and "No upper wick = the completed brick has zero upper shadow length as rendered by the chart series." In NinjaTrader's NinZaRenko series, each brick exposes OHLC values. For a bullish brick, the lower wick (shadow below the open) corresponds to `Low[0] < Open[0]`. "No lower wick" would therefore resolve to `Low[0] == Open[0]` for a bullish brick. For a bearish brick, the upper wick (shadow above the open/top of the brick) corresponds to `High[0] > Open[0]`. "No upper wick" would therefore resolve to `High[0] == Open[0]` for a bearish brick. This mapping must be confirmed against the actual NinZaRenko OHLC definition before any wick-check logic is written. See RC-6.

**DT-2 — Break at EOD behavior and session-open brick state**
The blueprint states "Break at EOD = True on both charts." In NinjaTrader, when a session break occurs with an open brick, the behavior of that brick (whether it is forced-closed, discarded, or carries over) depends on the Renko/NinZaRenko configuration. The strategy must correctly identify the first completed brick of a new session versus any artifact of the EOD break mechanism. The session reset rule (registry and counter reset at each session open) depends on accurately detecting the session open event.

**DT-3 — "Next tick after close" in NinjaTrader tick-data mode**
The blueprint defines: "'Next tick after close' means the first tick received after the relevant completed-brick close has been fully processed." In NinjaTrader with tick data, `OnBarUpdate` is called at brick completion. The "first tick received after" the close is the first `OnEachTick` or next `OnBarUpdate` call after the `IsFirstTickOfBar == false` check following the bar completion. The exact mechanism for detecting "first tick after close" in NinjaTrader's event model must be confirmed before entry submission logic is written.

---

### RC — Must Clarify Before Implementation

**RC-1 — Confirm the counting of "last 3 impulse bricks" when the impulse run is longer than 4 bricks**
The blueprint says "the last 3 bullish impulse bricks" includes the follow-through brick. If the impulse run before the pullback is 6 bullish bricks (3 initial + breakout + follow-through + 1 more), are the "last 3" the 3 most recent bullish bricks immediately before the pullback starts (bricks 4, 5, 6), regardless of how many bricks preceded the breakout brick? Confirmation is needed that "last 3" always means the 3 most recent in the run, not a fixed slice anchored to the breakout brick.

**RC-2 — Confirm the "freshness" behavior when the same confirmed swing is broken by the confirming brick itself**
If the brick that confirms a bearish swing low (i.e., the first bullish brick after a bearish run) simultaneously closes above the prior confirmed swing high (which would be a bullish break of structure), is that a fresh bullish break of structure? Or does the swing confirmation and the break occupy different evaluation states that cannot both be active simultaneously? The blueprint's processing order rule may imply an answer, but it is not explicit.

**RC-3 — Resolve "highest high of that pullback" in Section 6 Long Runner trailing**
Clarify whether "a bullish brick closes above the highest high of that pullback" uses the wick high of the pullback bricks or the close of the highest pullback brick. The Definitions default to closes for structure comparisons; however, the analogous Section 4 entry trigger uses the wick high. Confirm which applies here before coding.

**RC-4 — Define 13/2 qualification state disposal when a 34/5 leg ends without a stop-out**
When the 34/5 active leg ends due to: (a) close back through the confirmed swing level, or (b) a 3rd consecutive countertrend brick — with no open trade and therefore no stop-out — must any in-progress 13/2 impulse or pullback qualification state be explicitly discarded? Or is it implicitly invalid because the "must occur during an active 34/5 leg" condition is no longer satisfied? Confirm whether an explicit discard instruction is intended.

**RC-5 — Define 13/2 qualification state across time window boundaries**
When the entry time window closes (e.g., end of the 08:35–11:30 CT window) and a 34/5 active leg remains valid with an in-progress 13/2 state, clarify whether that 13/2 state is retained or discarded when the next window opens (13:00 CT). If retained, the leg and state may have aged significantly. If discarded, a new impulse must form.

**RC-6 — Confirm "no lower wick" and "no upper wick" NinZaRenko OHLC mapping**
For a bullish NinZaRenko brick: does "no lower wick" mean `Low[0] == Open[0]`? For a bearish NinZaRenko brick: does "no upper wick" mean `High[0] == Open[0]`? This must be confirmed against the NinZaRenko series OHLC specification before any wick-check logic is implemented.

**RC-7 — Confirm the 6-brick compression window is strictly session-scoped**
Confirm that the rolling window of "last 6 completed 34/5 bricks" for the compression test uses only bricks from the current session and that bricks from a prior session do not carry over into this count.

**RC-8 — Confirm the definition of "losing trade" for the 2-losing-trade cap**
The blueprint says "Maximum 2 losing trades per session." Clarify: (a) is a trade losing if it closes with any negative net P&L (including commissions)? (b) Is a trade stopped out at breakeven counted as a losing trade? (c) If the runner is stopped out at breakeven after the +1.5R partial has filled, is that trade losing or breakeven?

**RC-9 — Confirm entry order type**
The blueprint states entry is submitted on the first subsequent tick after the trigger brick closes, with no intrabar anticipation and no projected open entry. Confirm whether this is a market order, a stop order placed at the last known price, or a limit order. Confirm what happens if the first tick after the trigger close represents a gap beyond the trigger price.

**RC-10 — Resolve "1 tick below that confirmed higher-low pullback" reference price for runner trailing stop**
Section 6 Long Runner: "New stop = the higher of: current stop, 1 tick below that confirmed higher-low pullback." Clarify whether "that confirmed higher-low pullback" references the lowest wick low of the pullback sequence (matching the Section 5 initial stop language) or the closing price of the last pullback brick. The same clarification applies to the Short Runner: "1 tick above that confirmed lower-high pullback."

---

## PART II — CLEAN RESTATEMENT

*The following is a faithful section-by-section restatement of the BrickFlow Blueprint. No logic, thresholds, counts, or definitions have been altered. Formatting has been regularized. Meaning and intent are unchanged.*

---

### BrickFlow — Locked Implementation Assumptions

- Use tick data only on both series.
- Interpret NinZaRenko pairs as Brick Size / Trend Threshold:
  - Primary execution chart: 13/2
  - Higher-timeframe structure chart: 34/5
- Use completed-brick closes for structure and completed wick extremes for stops and triggers.
- Use no open-based structure rule.
- Session template is fixed to CME US Index Futures RTH.
- Break at EOD = True on both charts.
- No EMA/ZLEMA filter is used.
- All logic is closed-brick only. The forming brick is ignored.
- At each session open, both the 34/5 and 13/2 confirmed swing registries reset. The 34/5 active leg, the eligible 13/2 pullback-sequence counter, and any in-progress 13/2 impulse/pullback qualification state also reset.
- "Next tick after close" means the first tick received after the relevant completed-brick close has been fully processed.
- If a 34/5 brick and a 13/2 brick complete on the same incoming tick:
  - process the 34/5 close first,
  - then process the 13/2 close,
  - then evaluate any entry/exit action.
- Any updated 34/5 invalidation or block condition governs the 13/2 decision on that same tick.

---

### Definitions

- **Completed brick**: any fully closed brick.
- **Same-color run**: consecutive completed bricks of the same direction.
- **Confirmed swing high**: the highest close of a bullish same-color run of at least 2 bricks, confirmed only when the first bearish brick closes after that run.
- **Confirmed swing low**: the lowest close of a bearish same-color run of at least 2 bricks, confirmed only when the first bullish brick closes after that run.
- The confirmed swing high/low definition applies identically to both the 34/5 chart and the 13/2 chart, using only completed bricks on the relevant chart.
- **Latest confirmed swing high/low**: the most recently confirmed one as of the current completed brick close on that chart, with no lookahead.
- **Extreme swing high/low**: the wick high/low of the brick that created the confirmed swing high/low.
- **Break of structure**: a completed brick close beyond the prior confirmed swing high/low on the same chart.
- **Fresh 34/5 break of structure**: the first completed 34/5 close beyond a newly confirmed 34/5 swing high/low that has not already been broken by close since that swing was confirmed. Repeated closes beyond the same already-broken level are not fresh.
- **34/5 active leg**: the period beginning at a fresh 34/5 break of structure and ending only when the earliest of the following occurs:
  - session reset,
  - a completed 34/5 close back through the most recent confirmed 34/5 swing low in an up-leg,
  - a completed 34/5 close back through the most recent confirmed 34/5 swing high in a down-leg,
  - a 3rd consecutive countertrend 34/5 brick.
  - A later fresh same-direction 34/5 break that occurs before one of those leg-ending events does not start a new 34/5 active leg and does not reset the pullback-sequence counter.
- **13/2 pullback sequence**: a 1-brick or 2-brick countertrend correction with no interleaved same-direction brick.
- **Eligible 13/2 pullback sequence**: a 13/2 pullback sequence that fully satisfies all Section 3 pullback requirements and is complete at the close of its final countertrend brick while the corresponding 34/5 active leg remains valid.
  - The eligible-pullback counter increments exactly once, at the close of the final countertrend brick that completes that eligible sequence, whether or not a later trigger passes or a trade executes.
  - A sequence that violates any Section 3 pullback requirement before completion never becomes eligible and does not increment the counter.
- **No lower wick**: the completed brick has zero lower shadow length as rendered by the chart series.
- **No upper wick**: the completed brick has zero upper shadow length as rendered by the chart series.
- All trend/structure comparisons use closes unless a rule explicitly says wick.

---

### 1. Market Regime Definition (34/5)

#### Valid Uptrend

1. At least 2 confirmed 34/5 swing highs and at least 2 confirmed 34/5 swing lows exist.
2. The latest confirmed 34/5 swing high is above the prior confirmed 34/5 swing high.
3. The latest confirmed 34/5 swing low is above the prior confirmed 34/5 swing low.
4. A fresh bullish 34/5 break of structure has occurred: a completed bullish 34/5 brick has closed above the most recent confirmed 34/5 swing high.
5. Since the most recent fresh bullish 34/5 break, the current bearish correction count is 0, 1, or 2 completed bearish 34/5 bricks.

#### Valid Downtrend

1. At least 2 confirmed 34/5 swing highs and at least 2 confirmed 34/5 swing lows exist.
2. The latest confirmed 34/5 swing high is below the prior confirmed 34/5 swing high.
3. The latest confirmed 34/5 swing low is below the prior confirmed 34/5 swing low.
4. A fresh bearish 34/5 break of structure has occurred: a completed bearish 34/5 brick has closed below the most recent confirmed 34/5 swing low.
5. Since the most recent fresh bearish 34/5 break, the current bullish correction count is 0, 1, or 2 completed bullish 34/5 bricks.

#### Invalid / No-Trade Regime

- Any state with fewer than 2 confirmed 34/5 swing highs or fewer than 2 confirmed 34/5 swing lows is a single unified no-trade state.
- Latest swings are structurally mixed:
  - higher high with lower low, or
  - lower high with higher low.
- The last 6 completed 34/5 bricks are evaluated as a rolling window on every completed 34/5 brick. If they contain no same-color run of at least 3 bricks, the regime is invalid.
  - If fewer than 6 completed 34/5 bricks exist in the current session, this 6-brick compression test is treated as not passed.
- Current countertrend correction reaches 3 consecutive opposite-color 34/5 bricks.
- Price closes back through the most recent confirmed 34/5 swing low in an uptrend or through the most recent confirmed 34/5 swing high in a downtrend.
- Session has reset and a new 34/5 trend structure has not yet formed.

---

### 2. Trade Direction Filter

#### Longs Allowed Only When

1. 34/5 regime is a valid uptrend.
2. A fresh 34/5 bullish break of structure has already occurred.
3. The current 34/5 correction is not more than 2 bearish bricks.
4. No completed 34/5 brick has closed below the most recent confirmed 34/5 swing low.
5. All Section 7 risk/time filters are open.

#### Shorts Allowed Only When

1. 34/5 regime is a valid downtrend.
2. A fresh 34/5 bearish break of structure has already occurred.
3. The current 34/5 correction is not more than 2 bullish bricks.
4. No completed 34/5 brick has closed above the most recent confirmed 34/5 swing high.
5. All Section 7 risk/time filters are open.

#### Block All Trades When

- Neither long nor short filter is active.
- 34/5 is in transition, compression, or a 3-brick countertrend correction.
- Daily trade cap or daily loss cap is hit.
- Time window is closed.
- A position is already open in the opposite direction.
- The setup would be the 3rd or later eligible 13/2 pullback sequence inside the same 34/5 active leg.

---

### 3. Momentum Qualification (13/2)

#### Long Momentum Qualification

1. Must occur during an active 34/5 long leg.
2. A bullish 13/2 impulse must print at least 3 consecutive bullish bricks.
3. That impulse must break above the most recent confirmed 13/2 swing high by close.
4. After that breakout close, one additional bullish 13/2 close must occur before any pullback begins.
5. "The last 3 bullish impulse bricks" means the 3 most recent bullish bricks of the impulse run immediately before the pullback begins, including the required follow-through brick.
6. Of those last 3 bullish impulse bricks, at least 2 must have no lower wick.
7. The pullback that follows must be exactly 1 or 2 consecutive bearish bricks.
8. The lowest wick low of that pullback must remain:
   - above the close of the 13/2 breakout brick,
   - above the most recent confirmed 13/2 swing low,
   - above the most recent confirmed 34/5 swing low.
9. Pullback validity is evaluated and locked at the close of the final bearish pullback brick.
10. The next bullish brick is then evaluated independently as the trigger candidate against that locked pullback state.
11. If a 3rd bearish 13/2 brick prints, momentum qualification is lost.
12. Violation of any single pullback condition listed above means the pullback rules are no longer intact and the setup resets.
13. If a bullish brick appears before the trigger and the pullback rules are no longer intact, the setup resets.

#### Short Momentum Qualification

1. Must occur during an active 34/5 short leg.
2. A bearish 13/2 impulse must print at least 3 consecutive bearish bricks.
3. That impulse must break below the most recent confirmed 13/2 swing low by close.
4. After that breakout close, one additional bearish 13/2 close must occur before any pullback begins.
5. "The last 3 bearish impulse bricks" means the 3 most recent bearish bricks of the impulse run immediately before the pullback begins, including the required follow-through brick.
6. Of those last 3 bearish impulse bricks, at least 2 must have no upper wick.
7. The pullback that follows must be exactly 1 or 2 consecutive bullish bricks.
8. The highest wick high of that pullback must remain:
   - below the close of the 13/2 breakout brick,
   - below the most recent confirmed 13/2 swing high,
   - below the most recent confirmed 34/5 swing high.
9. Pullback validity is evaluated and locked at the close of the final bullish pullback brick.
10. The next bearish brick is then evaluated independently as the trigger candidate against that locked pullback state.
11. If a 3rd bullish 13/2 brick prints, momentum qualification is lost.
12. Violation of any single pullback condition listed above means the pullback rules are no longer intact and the setup resets.
13. If a bearish brick appears before the trigger and the pullback rules are no longer intact, the setup resets.

#### Brick Speed / Expansion Logic

- Momentum is defined by uninterrupted directional printing, not time.
- The required breakout brick + one follow-through brick is the minimum expansion requirement.
- Any impulse that breaks structure but does not print a follow-through brick is not qualified.

#### Wick and Overlap Rules

- Long impulse: at least 2 of the last 3 bullish impulse bricks, as defined above, must have no lower wick.
- Short impulse: at least 2 of the last 3 bearish impulse bricks, as defined above, must have no upper wick.
- A qualified pullback may contain only countertrend bricks.
- Any color alternation inside the pullback sequence = overlap/chop = no trade.

---

### 4. Entry Logic

#### Long Entry

1. Wait for a qualified long pullback sequence from Section 3.
2. Only the first bullish brick after that pullback may act as the continuation trigger.
3. That trigger brick must:
   - close above the highest wick high of the full pullback sequence,
   - be bullish,
   - have no lower wick,
   - close while the 34/5 long filter remains valid.
4. Signal is confirmed only at the trigger brick close.
5. Submit the long entry on the first subsequent tick after that close.
6. No intrabar anticipation.
7. No entry at a projected next-bar open.
8. If that first bullish brick fails any trigger condition, the setup expires and a new 13/2 impulse must form.

#### Short Entry

1. Wait for a qualified short pullback sequence from Section 3.
2. Only the first bearish brick after that pullback may act as the continuation trigger.
3. That trigger brick must:
   - close below the lowest wick low of the full pullback sequence,
   - be bearish,
   - have no upper wick,
   - close while the 34/5 short filter remains valid.
4. Signal is confirmed only at the trigger brick close.
5. Submit the short entry on the first subsequent tick after that close.
6. No intrabar anticipation.
7. No entry at a projected next-bar open.
8. If that first bearish brick fails any trigger condition, the setup expires and a new 13/2 impulse must form.

---

### 5. Stop Placement

#### Long Stop Logic

1. Initial stop = 1 tick below the lowest wick low of the 13/2 pullback sequence that produced the entry.
2. If that pullback low is at or below the most recent confirmed 34/5 swing low, the trade is invalid before entry.
3. After entry, stop may only:
   - remain where it is, or
   - ratchet tighter.
4. Stop may never widen.
5. Structural invalidation for any open long:
   - a completed 34/5 brick closes below the most recent confirmed 34/5 swing low, or
   - the 34/5 chart prints a 3rd consecutive bearish correction brick.
6. At the close of that invalidating 34/5 brick:
   - the regime becomes invalid,
   - new long entries are blocked immediately.
7. Exit any remaining long on the first subsequent tick.

#### Short Stop Logic

1. Initial stop = 1 tick above the highest wick high of the 13/2 pullback sequence that produced the entry.
2. If that pullback high is at or above the most recent confirmed 34/5 swing high, the trade is invalid before entry.
3. After entry, stop may only:
   - remain where it is, or
   - ratchet tighter.
4. Stop may never widen.
5. Structural invalidation for any open short:
   - a completed 34/5 brick closes above the most recent confirmed 34/5 swing high, or
   - the 34/5 chart prints a 3rd consecutive bullish correction brick.
6. At the close of that invalidating 34/5 brick:
   - the regime becomes invalid,
   - new short entries are blocked immediately.
7. Exit any remaining short on the first subsequent tick.

---

### 6. Profit Management

- Define 1R as the distance from actual filled entry price to initial stop price.
- Position must be divisible into 2 equal whole-number parts.

#### Partial Exit

- Exit 50% at +1.5R.

#### Runner Logic

- After the +1.5R partial fills, move stop on the remaining 50% to breakeven.
- The remainder is the runner.

#### Structural Trailing

- For longs, tighter means the numerically higher stop price.
- For shorts, tighter means the numerically lower stop price.

#### Long Runner

- Trail to 1 tick below each newly confirmed 13/2 higher low formed after entry.
- A new higher low is confirmed only after:
  - a 1-brick or 2-brick bearish pullback forms,
  - then a bullish brick closes above the highest high of that pullback.
- New stop = the higher of:
  - current stop,
  - 1 tick below that confirmed higher-low pullback.

#### Short Runner

- Trail to 1 tick above each newly confirmed 13/2 lower high formed after entry.
- A new lower high is confirmed only after:
  - a 1-brick or 2-brick bullish pullback forms,
  - then a bearish brick closes below the lowest low of that pullback.
- New stop = the lower of:
  - current stop,
  - 1 tick above that confirmed lower-high pullback.

#### Full Exit Conditions

- Exit any remainder when:
  - structural trail stop is hit,
  - 34/5 structural invalidation occurs,
  - session flat time is reached.
- If the +1.5R partial has not filled when any full-exit condition occurs, the entire remaining position exits in full and the breakeven-transfer rule does not activate.

---

### 7. Risk Controls

- Session template: CME US Index Futures RTH.
- Allowed entry windows:
  - 08:35–11:30 CT
  - 13:00–14:45 CT
- No new positions outside those windows.
- Force flat at 14:55 CT.
- Maximum 3 trades per session.
- Maximum 2 losing trades per session.
- Stop trading for the day at -2R net realized.
- Maximum 2 eligible 13/2 pullback sequences per 34/5 active leg.
- No add-ons.
- No pyramiding.
- No reversal-in-place.
- Configured entry size must be an even whole-number contract size of at least 2. If not, block all entries.
- After any stop-out, any in-progress 13/2 impulse or 13/2 pullback qualification state is discarded. The next trade must begin from a new 13/2 impulse and a new qualified pullback.
- If a trade is open and a force-flat condition occurs, exit on the first subsequent tick.

---

### 8. Failure Modes

#### Common False Signals

- 13/2 breaks a local swing but prints no follow-through brick.
- Pullback retraces back into the 13/2 breakout brick close.
- Pullback becomes 3 countertrend bricks instead of 1–2.
- First resumption brick after pullback fails to clear the entire pullback extreme.
- First resumption brick prints an opposite-side wick.
- Entry appears on the 3rd or later eligible pullback sequence of the same 34/5 leg.

#### Structural Conditions That Degrade Edge

- 34/5 has no confirmed HH/HL or LL/LH sequence.
- 34/5 last 6 bricks lack any 3-brick same-color run.
- 34/5 is already in a 3-brick countertrend correction.
- 13/2 pullback overlaps with alternating colors before trigger.
- 13/2 pullback reaches the most recent 34/5 swing level.

#### Explicit Do-Not-Trade Scenarios

- No confirmed 34/5 trend after session reset.
- Mixed 34/5 structure.
- 13/2 impulse without breakout follow-through.
- 13/2 pullback deeper than allowed.
- Trigger brick not clean enough.
- Daily loss cap hit.
- Daily trade cap hit.
- Outside time windows.
- Any open trade in the opposite direction.
