### Description

The SM2s battery goes into a faulty state, where it is unresponsive to communication attempts and the BMS doesn't seem to work any more (no balancing, protections, ...), but can still be charged.

### Root cause

This state is reached if the BMS wants to put itself into sleep mode (due to low cell or battery voltage) while there is still an external voltage source present at it's terminals. For some reason, the BMS firmly expects a drop in voltage at it's terminals and if this can't happen (because PV is still connected) after a while it becomes locked in this faulty, unresponsive state.

This usually doesn't occurr, because if there is PV present, then the battery get's charged and doesn't reach it's sleep / protection threshhold.