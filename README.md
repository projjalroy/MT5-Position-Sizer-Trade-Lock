# Position Sizer Locked (PS Locked)

A MetaTrader 5 Expert Advisor that adds two discipline tools to the free [EarnForex Position Sizer](https://www.earnforex.com/metatrader-expert-advisors/Position-Sizer/) panel:

1. **Trade Lock Guard.** After you break your own rules (too many stop-losses, too much loss, too many losing trades in a row), new trading is blocked for a set time. While the lock is active you cannot loosen the rules.
2. **Partial take-profits after entry.** Open the trade first. TP1 / TP2 / TP3 lines then appear on the chart, and a share of your position closes at each one.

Everything is set from the **Trading** tab of the Position Sizer panel. The position-sizing, risk and order features of the original Position Sizer work as before.

> **Disclaimer:** This software is provided as is, with no warranty. Trading leverage products carries a high risk of loss. Test on a demo account before using it live.

---

## Features

### 1. Trade Lock Guard

The lock starts automatically when any enabled rule is hit:

| Rule | What triggers it | Default |
|---|---|---|
| **Max SL hits/day** | Number of stop-losses hit | 2 |
| **Max loss, %** | Loss as % of the day's starting balance (floating loss included) | 2.00 |
| **Max losses in row** | Losing trades closed one after another (a win or breakeven resets the count) | 3 |
| **Lock hours** | How long trading stays blocked (1 to 720) | 24 |

Set any of the first three to `0` to turn that rule off.

**While locked**
- The rule fields turn grey and cannot be edited. If you try, you get an alert and the old values come back. Changing them is only possible after the lock ends.
- The Trade button, the Shift+T hotkey and the outside trade button refuse to place orders.
- Any position opened during the lock is closed within about a second, and pending orders are deleted.
- The lock survives restarting MT5. It is stored both in terminal global variables and in a file in the common data folder, and whichever copy has the later lock wins.
- A status panel in the top-right corner of the chart shows the lock state, the time left, and your current SL count, loss % and losses in a row.

**How the numbers are counted**
- The period starts at server midnight, or when the last lock ended, whichever is later.
- Only stop-losses that closed at a loss count as SL hits (option `TLG: Count only SL hits that closed at a loss`).
- A trade closed in several parts counts as **one** trade. Commission and swap are included in the result.

### 2. Partial take-profits after entry

1. Place the trade normally, with a stop-loss and in one order.
2. Within about a second, three horizontal lines appear at set R multiples of your stop-loss distance (defaults **2R, 3R, 4R**).
3. When price reaches a line, the EA closes that share of the **original** volume. TP3 closes whatever is left.
4. The position's real take-profit is set to the last target that will close something, so the broker holds a backstop.

**Panel settings (Trading tab)**
- **Partial TPs after entry (lines)** is the on/off checkbox.
- **TP1/2/3 at R** sets the distance of each target. The values must increase from TP1 to TP3.
- **Close share, %** sets the share for TP1 and TP2. The TP3 share is the remainder and is shown read-only. Example: `50 / 30` gives `20` at TP3. Setting TP1 to `100` closes everything at TP1.

**Behaviour**
- Lines can be dragged after the trade opens. Double-click a line to select it, then drag.
- Editing the R values or shares updates the lines of open positions straight away. Targets that were already hit stay closed.
- Each line has a label in the style of the Position Sizer's TP label: lots, % of balance, money and R, for the part closed at that target.
- Switching the checkbox on also takes over positions that are already open and have a stop-loss.
- A position without a stop-loss is not managed, because the lines are measured from it.
- If the lot size is too small to split at a target, that target is skipped. The last target always closes the rest.

---

## Installation

1. Install the original Position Sizer from EarnForex so that you have its support files in `MQL5/Experts/Position Sizer/`: `Position Sizer Trading.mqh`, `Defines.mqh`, `errordescription.mqh`, the `Translations` folder and the icon.
2. Copy these two files into the same folder:
   - `PS Locked.mq5`
   - `Position Sizer Locked.mqh`
3. Open `PS Locked.mq5` in MetaEditor and press **F7**. It should compile with 0 errors.
4. In MT5, drag **PS Locked** onto a chart and tick **Allow Algo Trading**. Remove the original Position Sizer from that chart first, because MT5 allows one EA per chart.

**Name length:** keep the EA file name short. The panel names its controls after the file name, and MT5 limits object names to 63 characters. With a name such as "Position Sizer Locked", the panel can fail to start.

---

## Inputs (EA settings dialog)

All original Position Sizer inputs are still there. Added by this project:

| Input | Meaning |
|---|---|
| `TLG: Count only SL hits that closed at a loss` | Ignore stop-losses that closed at breakeven or in profit |
| `TLG: Delete ALL pending orders while locked` | Remove pending orders during a lock |
| `TLG: Also close trades opened BEFORE the lock started` | Close older positions too, not only those opened during the lock |

The lock rules and the partial-TP settings are not inputs. They are edited on the panel.

---

## Limits to know

- **It is a self-discipline tool, not a hard block.** MT5 offers no way to stop an order from being sent, so the EA closes a trade a moment after it opens. You may pay the spread.
- Removing the EA, turning off Algo Trading or deleting the saved lock data ends the lock. For a stronger setup, run it on a VPS and keep your trading password only there.
- Orders from MT5's own order window, another EA or a phone app are not stopped at the click. They are closed shortly after.
- The EA must stay attached and running for partial closes to happen. Only the final take-profit is held by the broker.
- Partial closes need a **hedging** account.
- Prices are checked every second and on each tick. A fast spike can fill slightly past a line.
- Time is the broker server's time, so changing your PC clock has no effect.

---

## Files

| File | Purpose |
|---|---|
| `PS Locked.mq5` | The Expert Advisor, including the Trade Lock Guard and partial take-profit logic |
| `Position Sizer Locked.mqh` | The panel code, a modified copy of the original `Position Sizer.mqh` |

## Credits

Built on the **Position Sizer** by [EarnForex.com](https://www.earnforex.com/). Check the original project's license terms before publishing or redistributing this derived work.
