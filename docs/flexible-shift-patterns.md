# Flexible Shift Patterns — Feature Proposal

> **Status:** Proposed (not yet implemented)
> **Goal:** Allow the app to support ANY shift arrangement — 2-2-2, 3 days morning + 4 days off, 1-1-1, 6 days morning + 1 day off, or anything else a company can dream up.

---

## 🎯 The Big Idea (for Non-Techies)

Think of a shift pattern like a **song played on repeat**.

Right now, the app only knows ONE song: **"2 mornings, 2 nights, 2 days off"** — and it plays that song over and over, day after day, starting from the anchor date.

This proposal says: **let the user choose the song.** Instead of being stuck with 2-2-2, they could load "3 mornings, 4 days off" and the app would play *that* song on repeat instead — automatically coloring every day correctly, forever, backward and forward.

That's literally the whole feature. Everything else is detail.

**And there's a crucial twist (see "Switching Mid-Period" below):** what if the company changes the song *halfway through a voucher period*? The days already played must stay exactly as they were, and the new song takes over from the switch date onward. The app must keep **both** songs — the old one for the past, the new one for the future — without ever touching your past data.

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

In the future, it would store the anchor date **plus a history of pattern segments**:

```
Segment 1:  starts 2026-07-20   pattern = M, M, N, N, OFF, OFF        (2-2-2)
Segment 2:  starts 2026-08-05   pattern = M, M, M, OFF, OFF, OFF, OFF (3M-4OFF)
```

A **segment** is simply *"from this date onward, this pattern applies."* The very first segment starts on your anchor date. Every time the pattern changes, a **new segment** is added with its own start date — the old segments are never touched or rewritten.

Each day in the pattern also carries useful extra information:

- **Is it a work day?** (M and N = yes, OFF = no) — decides whether you can mark it "missed"
- **Is it a night shift?** — saved now so the future salary feature can add night premium pay later
- **What color is it?** — keeps the calendar's amber / indigo / gray color language

Because all of this is stored as simple data (not baked into the code), the app's other features — stats, streak, export — automatically work with any pattern without needing to be rewritten.

**How the app decides which pattern a day follows:** for any given date, it finds the most recent segment whose start date is on or before that day, and applies *that* segment's pattern. Think of it like timetables: a bus schedule printed in July is history; the one printed in August governs trips from August onward — but the July dates still follow the July schedule.

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

## 🔄 Switching Patterns Mid-Period (Full Flexibility)

This is the most important part of the design.

**The problem:** what if the company changes the shift pattern *in the middle of a voucher period*? For example, you're on 2-2-2 for the first half of July 20 – Aug 19, and the company switches to 3M-4OFF starting August 5.

If the app had a single pattern for everything, switching would recompute *every* day from the anchor — and every day before August 5 would suddenly show the wrong shift. Your past attendance, your missed marks, your streak — all wrong. Unacceptable.

**The solution — the "Rulebook" model (segments):**

The app keeps a **history of segments**. Each segment is:

```
From date X onward, pattern P applies.
```

- The **anchor date** is simply the start of the *first* segment.
- When the pattern changes, the app does **not** rewrite the past. It just **adds a new segment** starting on the switch date.
- For any given day, the app asks: *"Which rulebook was in effect on this date?"* It finds the most recent segment whose start date is on or before that day, and applies *that* segment's pattern.

**Worked example** — anchor July 20, switch to 3M-4OFF on August 5:

```
Segment 1:  Jul 20 onward → M M N N OFF OFF        (2-2-2)
Segment 2:  Aug 05 onward → M M M OFF OFF OFF OFF (3M-4OFF)

Jul 20  → 2-2-2 segment → M        (segment 1)
Jul 25  → 2-2-2 segment → N        (segment 1)
Aug 04  → 2-2-2 segment → N        (segment 1 — last day of the old rules)
Aug 05  → 3M-4OFF segment → M      (segment 2 — first day of the new rules)
Aug 08  → 3M-4OFF segment → OFF    (segment 2)
Aug 12  → 3M-4OFF segment → OFF    (segment 2)
```

Every day before August 5 keeps the exact shifts it had. Every day from August 5 onward follows the new pattern. Nothing in the past is touched.

**How the switch happens in the app (user flow):**

1. User taps **"Change Pattern"** in the grid header.
2. The editor opens with the chip builder / presets.
3. The app asks: **"From which date does the new pattern start?"** — defaults to today, and can be changed (e.g., the company announced it takes effect September 1 — just pick that date).
4. User saves → the app **appends a new segment** and re-renders.
5. Done. The calendar shows old pattern up to the switch date, new pattern after it — perfectly clean.

**What stays untouched:**

- All past days' shift labels
- All missed-day marks (a mark made under the old pattern stays on the same date)
- Stats, streak, percentage — they simply recompute per-day using whichever segment was in effect
- CSV export — same, each day exports with its correct historical shift

**Edge cases handled naturally:**

- **Switching back to an old pattern?** Just another new segment. The app never merges or rewrites history — accuracy first.
- **Switching several times?** Unlimited segments. Each one is preserved like a timeline.
- **Scheduling a future switch?** Perfectly fine — segments are date-indexed, so a segment starting next month is simply "not in effect yet" until its date arrives.
- **Switching on an OFF day?** No problem — the pattern simply changes at the start of that day.
- **Missed day that becomes OFF after a switch?** The mark stays stored; the day renders as OFF under the new rules. If a future segment ever makes it a work day again, the mark reappears.

**Why this is also correct for salary:** when the salary feature arrives, each day's pay will be computed under the *actual* rules that were in effect that day — the night premium, the shift type, the work day. Pay history stays truthful even across pattern changes.

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

1. It notices there's no pattern history saved.
2. It silently creates **one segment**: the classic **2-2-2** starting on their anchor date — exactly what they have today.
3. Nothing changes for them until they open the pattern editor and pick something new.

No data is lost. Because the app never rewrites history, switching patterns (even switching back) always keeps every past day exactly as it was.

**One small quirk to be aware of:** if a new pattern makes a previously marked *missed* day an *OFF* day, the mark isn't deleted — the day just renders as OFF while that pattern is in effect. Under a later segment (or switching back), the mark reappears. Harmless either way.

---

## ⚠️ Guardrails (Rules the App Will Enforce)

- **Empty pattern?** Not allowed — a pattern needs at least one day.
- **All OFF days?** Allowed, but the app warns: *"No working days — you'll have nothing to mark."*
- **Very long patterns?** Fine mathematically (the remainder trick works at any length), but the editor caps at ~60 days to keep things usable.
- **Repeated days in a row?** Perfectly normal — that's what 2-2-2 already does.
- **One-day pattern (e.g., every day is M)?** Legal and mathematically valid.
- **Changing a pattern?** The whole calendar re-renders instantly. No waiting, no recalculation errors.
- **Changing a pattern mid-period?** Never rewrites the past — it appends a new segment from the chosen switch date (see "Switching Mid-Period" above).

---

## 🎯 Alignment Check — Why It Matters

The anchor date is the *starting point* of the first pattern, and every segment's start date is the starting point of *that* segment's pattern. If you pick a start date that's actually a night day but the pattern begins with M, the whole cycle from that point would be shifted by one day — every label wrong.

To prevent this, the pattern editor will show a **live preview** of the ~14 days following the chosen start date, like:

```
Jul 20   Jul 21   Jul 22   Jul 23   Jul 24   Jul 25   Jul 26
 [M]      [M]      [M]     [OFF]    [OFF]    [OFF]    [OFF]
```

So you can verify "yes, this date really is my morning day" *before* saving — whether it's the original anchor or a mid-period switch. The preview works backward from the start date too.

---

## 🧪 How We'd Test It

1. **Regression test:** confirm 2-2-2 still works exactly as today.
2. **3 mornings + 4 off:** verify the calendar repeats M,M,M,OFF,OFF,OFF,OFF forever.
3. **1-1-1:** verify M,N,OFF repeats every 3 days.
4. **6 mornings + 1 off:** verify the 7-day pattern holds.
5. **Backward dates:** verify days *before* the anchor are correct (the song plays in reverse too).
6. **Mid-period switch:** switch from 2-2-2 to 3M-4OFF on a date in the middle of a period — confirm days before the switch keep their old labels, days after follow the new pattern, and missed marks/stats/streak/CSV all stay correct.
7. **Switch back:** change back to 2-2-2 later — confirm it creates a third segment and history stays accurate.
8. **Scheduled future switch:** set a segment starting next month — confirm it has no effect until its start date arrives.
9. **Multiple switches:** apply 3+ pattern changes and verify the timeline renders correctly for every date.

---

## 🔮 Bonus: This Sets Up the Salary Feature

Remember the salary feature we discussed? Each pattern day already records whether it's a **night shift** and whether it's a **work day**. When the contractors give us the pay rules, we'll be able to attach things like:

- **Night premium** (e.g., +15% for night days)
- **Pay multiplier** per shift type

...without redesigning anything. And because of the segment model, each day's pay is computed under the rules *actually in effect that day* — so even mid-period pattern changes produce a perfectly accurate pay history. The pattern feature isn't just a schedule upgrade — it's the foundation the salary calculator will sit on.

---

## 📋 Summary

| Question | Answer |
|---|---|
| What's the feature? | Let users pick any repeating shift pattern, not just 2-2-2 |
| How does it work? | Store the pattern as data; index into it with the "remainder trick" from the anchor date |
| How do users set it? | One-tap presets or a visual chip builder |
| Can patterns change mid-period? | Yes — a "segment" (rulebook) model: old days keep the old pattern, the switch date onward uses the new one, past data is never rewritten |
| How many switches? | Unlimited — each switch adds a dated segment to a timeline |
| Does it break anything? | No — existing data defaults to 2-2-2; only counters/tooltip need small updates |
| Why now? | It's a small change, and it lays the foundation for the salary calculator |
