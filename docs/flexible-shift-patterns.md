# Flexible Shift Patterns — Feature Proposal

> **Status:** Proposed (not yet implemented)
> **Goal:** Allow the app to support ANY shift arrangement — 2-2-2, 3 days morning + 4 days off, 1-1-1, 6 days morning + 1 day off, or anything else a company can dream up.

---

## 🎯 The Big Idea (for Non-Techies)

Think of a shift pattern like a **song played on repeat**.

Right now, the app only knows ONE song: **"2 mornings, 2 nights, 2 days off"** — and it plays that song over and over, day after day, starting from the anchor date.

This proposal says: **let the user choose the song.** Instead of being stuck with 2-2-2, they could load "3 mornings, 4 days off" and the app would play *that* song on repeat instead — automatically coloring every day correctly, forever, backward and forward.

That's literally the whole feature. Everything else is detail.

---

## 🧩 What Is a Shift Pattern?

A shift pattern is simply **a list of days that repeats itself**.

| Name | The pattern | Total days |
|---|---|---|
| 2-2-2 (current) | M, M, N, N, OFF, OFF | 6 |
| 3 mornings, 4 off | M, M, M, OFF, OFF, OFF, OFF | 7 |
| 1-1-1 | M, N, OFF | 3 |
| 6 mornings, 1 off | M, M, M, M, M, M, OFF | 7 |
| 4 mornings, 2 off | M, M, M, M, OFF, OFF | 6 |
| Something nobody has invented yet | M, N, OFF, M, M, N, ... | any length |

The symbols mean:

- **M** = Morning shift (you work in the morning)
- **N** = Night shift (you work at night)
- **OFF** = You're off duty

Once the pattern is saved, the app walks the calendar day by day, "playing the song" from the anchor date — and every day automatically gets its correct label and color.

---

## 🧮 How the Math Works (Simple Version)

Imagine a pattern with 7 days: **M, M, M, OFF, OFF, OFF, OFF**

Number the days of the pattern 0 to 6:

| Pattern position | 0 | 1 | 2 | 3 | 4 | 5 | 6 |
|---|---|---|---|---|---|---|---|
| Shift | M | M | M | OFF | OFF | OFF | OFF |

Now pick an anchor date — say **July 20**. July 20 is "position 0" (the first M).

To find out what any date is, count how many days it is from the anchor, then find the remainder after dividing by 7 (the pattern length):

| Date | Days from anchor | Remainder (÷7) | Shift |
|---|---|---|---|
| Jul 20 | 0 | 0 | M |
| Jul 22 | 2 | 2 | M |
| Jul 23 | 3 | 3 | OFF |
| Jul 26 | 6 | 6 | OFF |
| Jul 27 | 7 | 0 | M (song restarts) |
| Jul 19 | −1 | 6 | OFF (the day *before* the anchor) |

This "remainder" trick (mathematicians call it **modulo**) works for **any pattern length** — 3 days, 6 days, 7 days, 40 days, whatever. That's why the app only needs one small change to support every possible arrangement.

> **For the curious:** today's code is hard-coded to "divide by 6" because 2-2-2 has 6 days. The proposal is to *store the pattern as data* and "divide by the pattern's own length" instead.

---

## 📦 How the App Would Store It

Today, the app stores only your **anchor date** (the day your cycle starts) and *assumes* 2-2-2 in the code.

In the future, it would store the anchor date **plus the pattern**:

```
Anchor date:  2026-07-20
Pattern:      M, M, M, OFF, OFF, OFF, OFF
```

Each day in the pattern also carries useful extra information:

- **Is it a work day?** (M and N = yes, OFF = no) — decides whether you can mark it "missed"
- **Is it a night shift?** — saved now so the future salary feature can add night premium pay later
- **What color is it?** — keeps the calendar's amber / indigo / gray color language

Because all of this is stored as simple data (not baked into the code), the app's other features — stats, streak, export — automatically work with any pattern without needing to be rewritten.

---

## 🎨 How Users Would Set It Up (the Plan)

Two ways to choose a pattern, both simple:

### Option 1: One-Tap Presets
A menu of ready-made patterns:

| Preset | Pattern |
|---|---|
| 2-2-2 (current) | M M N N OFF OFF |
| 3-3-3 | M M M N N N OFF OFF OFF |
| 1-1-1 | M N OFF |
| 3M / 4OFF | M M M OFF OFF OFF OFF |
| 4M / 4OFF | M M M M OFF OFF OFF OFF |
| 5M / 2OFF | M M M M M OFF OFF |
| 6M / 1OFF | M M M M M M OFF |
| 4M / 2OFF | M M M M OFF OFF |
| 2M / 1OFF | M M OFF |

Tap one, done.

### Option 2: Custom Builder (Build Your Own)
A row of chips shows the pattern growing as you build it:

```
[M] [M] [N] [N] [OFF] [OFF]
```

With buttons beneath it:

- **+M** — add a morning day
- **+N** — add a night day
- **+OFF** — add an off day
- **⌫ Delete last** — remove the most recent day
- **Clear** — start over

You tap until the chip row matches your real schedule, check a live preview of the next two weeks, then hit **Save Pattern**.

*(For power users, a text box could also accept shorthand like `2M 2N 2OFF`.)*

---

## ✅ What Happens to the Rest of the App?

Surprisingly little needs to change, because the app already works in a pattern-agnostic way:

| Feature | Change needed |
|---|---|
| Calendar colors | None — colors come from the pattern itself |
| Missed-day marking | None — only "work days" (M/N) are tappable, whatever they're called |
| Attendance % | None — same formula |
| Streak counter | None — it already skips OFF days |
| CSV export | Tiny — status column reads from the pattern automatically |
| Period summary tooltip | Small — counts each label dynamically instead of assuming M/N/OFF |

The two spots that *do* need updating are the **pattern editor** (new) and the **summary tooltip/counters** (generalized to count any labels).

---

## 🔄 What About People Who Already Use the App?

Existing users have an anchor date saved but no pattern. When the app loads:

1. It notices there's no pattern saved.
2. It silently assumes the classic **2-2-2** — exactly what they have today.
3. Nothing changes for them until they open the pattern editor and pick something new.

No data is lost. If someone switches patterns and switches back, their old missed-day marks still behave correctly.

**One small quirk to be aware of:** if you change your pattern, a day you previously marked *missed* might now be an *OFF* day under the new pattern. The mark isn't deleted — the day just renders as OFF instead of red. Switch back to the old pattern and it reappears. Harmless either way.

---

## ⚠️ Guardrails (Rules the App Will Enforce)

- **Empty pattern?** Not allowed — a pattern needs at least one day.
- **All OFF days?** Allowed, but the app warns: *"No working days — you'll have nothing to mark."*
- **Very long patterns?** Fine mathematically (the remainder trick works at any length), but the editor caps at ~60 days to keep things usable.
- **Repeated days in a row?** Perfectly normal — that's what 2-2-2 already does.
- **One-day pattern (e.g., every day is M)?** Legal and mathematically valid.
- **Changing a pattern?** The whole calendar re-renders instantly. No waiting, no recalculation errors.

---

## 🎯 Alignment Check — Why It Matters

The anchor date is the *starting point* of the pattern. If you pick an anchor that's actually a night day but your pattern starts with M, the entire calendar would be shifted by one day — every label wrong.

To prevent this, the pattern editor will show a **live preview** of the next ~14 days beneath the anchor, like:

```
Jul 20   Jul 21   Jul 22   Jul 23   Jul 24   Jul 25   Jul 26
 [M]      [M]      [M]     [OFF]    [OFF]    [OFF]    [OFF]
```

So you can verify "yes, July 20 really is my morning day" *before* saving. The app also works backward from the anchor, so the preview shows both directions.

---

## 🧪 How We'd Test It

1. **Regression test:** confirm 2-2-2 still works exactly as today.
2. **3 mornings + 4 off:** verify the calendar repeats M,M,M,OFF,OFF,OFF,OFF forever.
3. **1-1-1:** verify M,N,OFF repeats every 3 days.
4. **6 mornings + 1 off:** verify the 7-day pattern holds.
5. **Backward dates:** verify days *before* the anchor are correct (the song plays in reverse too).
6. **Pattern switching:** change patterns and confirm stats, streak, CSV, and missed marks all stay sane.

---

## 🔮 Bonus: This Sets Up the Salary Feature

Remember the salary feature we discussed? Each pattern day already records whether it's a **night shift** and whether it's a **work day**. When the contractors give us the pay rules, we'll be able to attach things like:

- **Night premium** (e.g., +15% for night days)
- **Pay multiplier** per shift type

...without redesigning anything. The pattern feature isn't just a schedule upgrade — it's the foundation the salary calculator will sit on.

---

## 📋 Summary

| Question | Answer |
|---|---|
| What's the feature? | Let users pick any repeating shift pattern, not just 2-2-2 |
| How does it work? | Store the pattern as data; index into it with the "remainder trick" from the anchor date |
| How do users set it? | One-tap presets or a visual chip builder |
| Does it break anything? | No — existing data defaults to 2-2-2; only counters/tooltip need small updates |
| Why now? | It's a small change, and it lays the foundation for the salary calculator |
