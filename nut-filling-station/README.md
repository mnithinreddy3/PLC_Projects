# Nut Filling Station

## Overview
A conveyor-based sorting and filling system that routes boxes to the correct hopper based on a colored label, fills them, then sends them on. Simulates a real packaging line process where product identification and fill control happen inline on a moving conveyor.

## Problem
Boxes with either a red or blue label move down a conveyor. Red-labeled boxes need to be filled with pecans, blue-labeled boxes with walnuts. The system has to:
- Detect when a box arrives and stop the conveyor to fill it
- Identify which label is present and open the correct hopper
- Detect when the box is full and close the hopper
- Restart the conveyor to send the filled box on its way — continuously, with no product wasted and no delays

The core challenge: this is a repeating cycle, not a one-shot sequence, so the logic has to correctly reset and restart itself for the next box without operator intervention.

## Approach
Treated the process as a cycle with distinct states — conveyor running / box present & filling / box full & releasing — controlled by the proximity switch, photo eyes, and level switch working together rather than independently.

Key design decision: the conveyor motor logic needed to seal in "stopped" while a box is present, but explicitly restart once the level switch confirms the box is full — this was the main point I got stuck on, since simply de-energizing the hoppers on level switch doesn't restart the conveyor by itself. Used [latching/seal-in logic / an internal bit — *fill in what you actually used*] to track "box present and being filled" as a state, separate from the raw sensor inputs, so the conveyor restart condition could be tied to box-full rather than just sensor state.

Also handled the bonus edge case: hoppers must never energize unless a box (proximity switch) is actually present under them, even if a photo eye is triggered — prevents dumping product with no box to catch it.

## I/O List
| Tag | Type | Description |
|---|---|---|
| I:0/0 | Input | Proximity switch — closes when a box is near |
| I:0/1 | Input | Level switch — closes when the box is full |
| I:0/2 | Input | Red photo eye — closes when a red label is detected |
| I:0/3 | Input | Blue photo eye — closes when a blue label is detected |
| O:0/0 | Output | Conveyor motor — runs conveyor forward |
| O:0/1 | Output | Walnut hopper solenoid — opens when energized |
| O:0/2 | Output | Pecan hopper solenoid — opens when energized |

## Test Criteria (verified in Emulate)
- Conveyor runs by default; both hoppers off
- Box arrives (proximity switch closed) → conveyor stops, hoppers stay off
- Red label detected → pecan hopper energizes, walnut stays off, conveyor stays off
- Label switches to blue → pecan de-energizes, walnut energizes
- Box full (level switch closed) → both hoppers de-energize, conveyor restarts
- No box present → both hoppers stay off regardless of photo eye state, conveyor keeps running

## What I Learned
The trickiest part wasn't the sorting logic itself — it was realizing the conveyor restart couldn't just be "not filling anymore," it needed to be explicitly tied to the level switch confirming a complete fill, using an internal state bit rather than relying on the momentary sensor inputs alone.


## Acknowledgments
Projects in this repo are based on assignments from Paul (PLC Dojo)'s **Applied Logic** course series on Udemy. Used with permission from the instructor. All logic implementations and solutions are my own work.
