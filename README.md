# 📅 RafaVoucherApp

A full-featured **Voucher Attendance Tracker** built as a single-file HTML app. Manage any shift schedule (2-2-2 by default, but fully customizable), mark missed days, track streaks, view attendance percentages, record the payment you receive (always for the previous voucher period), and export data — all offline, in your browser, with no backend or sign-up.

> **Live demo:** open `index.html` in any modern browser. No install, no build step.

---

## ✨ Features

- **Monthly Voucher Period** — Automatically detects the current period (20th–19th) based on today's date.
- **Customizable Shift Patterns** — Works with any repeating schedule (2-2-2 default, or presets like 3M/4OFF, 5M/2OFF, 1-1-1, or a fully custom pattern built chip by chip).
- **Mid-Period Pattern Switching** — Change your pattern from any chosen date; days before it keep their old pattern, so history is never rewritten.
- **Live Pattern Preview** — See the first 14 days of a new pattern before you save it.
- **Configurable Anchor Date** — Tap any day as your cycle start; the entire calendar recomputes instantly.
- **Day Action Dialog** — Tap any day to open an action dialog (titled with the current voucher period) with a drop-down: **Missed**, **Unmiss** (if already missed), **Paid**, or **Copy Date**.
- **Record Payment (PAID)** — Your company always pays **for the previous voucher period** (one payment per period). Choose **Paid** and enter the amount (e.g. ₦82,850.43). The day turns green **PAID** (a marker only — it never changes the current period's attendance), and a summary card shows the amount, the **previous voucher period** it pays for, and the **days-to-pay** count from the previous period's start up to (and including) the pay day.
- **Attendance Percentage** — See your present vs. total-working-day percentage below the main counter.
- **Present Streak Counter** — Consecutive present-day streak (backward from today) shown with a fire icon.
- **Period Navigation** — Use arrow buttons or **swipe left/right** on the calendar grid.
- **Period Summary Tooltip** — Tap the period label to see M/N/OFF/Present/Missed counts (capped at the paid day if the period is paid).
- **Anchor Confirmation Popup** — Green "ANCHOR ✓" animation when setting a new start date.
- **Missed / Un-missed / PAID Popups** — Animated MISSED (red), UNMISSED ✓ (green) and PAID ✓ (green) feedback on actions.
- **Copy Date** — Via the day action dialog, copy any day's YYYY-MM-DD date to your clipboard.
- **Export / Import / Reset** — Export as JSON or CSV, import JSON backups, or reset all data.
- **Help Attention** — On first visit, the help button pulses with a hint bubble until the guide is opened once.
- **WhatsApp Contact** — One-tap WhatsApp link in the footer for support.
- **Dark Mode** — Toggle at the top-left corner; persisted across sessions.
- **Accessibility** — ARIA labels, roles, and semantic HTML for screen readers.
- **Fully Responsive** — Works on phones, tablets, and desktops.
- **Offline-First** — Everything stored in `localStorage`. No internet required after first load.

---

## 🚀 Getting Started

### Run Locally

```bash
git clone https://github.com/EngrDammie/RafaVoucherApp.git
cd RafaVoucherApp
```

Then open `index.html` in your browser. That's it.

For best results (to avoid CORS issues with some browsers when importing files), serve with a local server:

```bash
python3 -m http.server 8000
# → http://localhost:8000
```

### First-Time Usage

1. On first open, you'll see a friendly welcome message and `--` for all counters.
2. Tap **Set Cycle Start** above the calendar.
3. The message disappears and day buttons appear. Tap any past day to set it as your **anchor date**.
4. The calendar instantly fills with color-coded M (Morning), N (Night), and OFF labels.
5. Tap any day to open the **action dialog** and choose **Missed** to mark a working day **MISSED** (red). Choose **Paid** to record your payment for the period.
6. Watch your stats, percentage, and streak update in real time.

---

## 🧱 Tech Stack

| Layer       | Technology                              |
|-------------|------------------------------------------|
| Markup      | Semantic HTML5                           |
| Styling     | Tailwind CSS (CDN) + custom CSS overrides|
| Font        | Google Poppins                           |
| Logic       | Vanilla JavaScript (ES6)                 |
| Persistence | Browser `localStorage`                   |

Zero build tools, no package manager, no framework — just one portable HTML file.

---

## 📁 Project Structure

```
RafaVoucherApp/
├── index.html      # The entire app (HTML + Tailwind + CSS + JS)
└── README.md       # This file
```

---

## 💾 Data Storage

All state is saved in your browser's `localStorage`:

| Key          | Type                        | Purpose                                       |
|--------------|-----------------------------|-----------------------------------------------|
| `anchorDate`  | `string` (`YYYY-MM-DD`)     | The cycle-start day that drives shift labels. |
| `missedDays`  | `JSON` array of date strings| Days the user marked as missed.               |
| `shiftSegments`| `JSON` array of segments   | `[{startDate, pattern:[{label, work, night, color}]}]` — every pattern switch adds a dated segment. |
| `paidDays`    | `JSON` array of payments  | `[{date:'YYYY-MM-DD', amount:number}]` — the day the user was paid. One per voucher period, always **for the previous period**. |
| `theme`       | `"light"` or `"dark"`       | User's theme preference.                      |
| `helpIntroSeen`| `"1"` (set once)           | Marks that the help guide has been opened once.|

> Clearing browser storage / cookies resets the app to first-run state. Export a backup before doing so.

---

## ⚙️ How It Works

### Voucher Period (`getVoucherPeriod`)
Computes a start date (20th of the current or previous month) and end date (19th of the following month) based on today. If today is before the 20th, the period is the previous month's 20th → this month's 19th.

### Shift Engine (`getShiftInfo`)
Patterns are stored as data — each entry has a label plus `work` / `night` / `color` flags, so any repeating schedule works. Segments are `{startDate, pattern}` pairs (the first segment's start is the anchor date). For any target date, the engine picks the most recent segment whose start date is on or before it, then walks the pattern via modulo indexing: `((diffDays % len) + len) % len`. Dates before the first segment wrap backward around the first pattern. Changing your pattern simply appends a new dated segment — history is never rewritten.

### Pattern Editor
Tap **Shift Pattern** to open the editor: choose a preset or build a custom pattern with `+ M` / `+ N` / `+ OFF` chips (tap a chip to remove it), pick the start date, check the live 14-day preview, then save. A saved switch applies from the chosen date onward (future dates allowed).

### Streak Calculation (`calcStreak`)
Walks backward from the period end date (or today, whichever is earlier), skipping OFF days and stopping at the first missed day, counting consecutive present days.

### Rendering (`renderCalendar`)
Clears and rebuilds the calendar grid. Every day is a `<button>` with `data-date`. Future days are muted and not clickable. Past days get shift-based colors. When a voucher period is paid, a green **PAID** summary card shows the amount, the **previous voucher period** it pays for, and days-to-payment (measured from the previous period's start to the pay day). The friendly banner is shown only when no anchor is set and the user is not in cycle-setting mode.

### Day Action Dialog
A single `onclick` handler on `#calendar-grid` uses `e.target.closest('button[data-date]')` to handle day clicks. Outside cycle-setting mode, tapping any past day opens the **Day Action** dialog titled with the current voucher period. Its drop-down offers **Missed** / **Unmiss** (if already missed) / **Paid** / **Copy Date**. Choosing **Paid** opens a second dialog to enter the amount (labelled as paying the **previous voucher period**); on submit the day is stored in `paidDays`, marked green **PAID**, and a `PAID ✓` popup fires. Copy date is handled inside this dialog (it replaced the old double-tap shortcut).

---

## 🖱️ Interactive Features

| Action                        | How                                             |
|-------------------------------|--------------------------------------------------|
| Set / change cycle start      | Tap **Set Cycle Start**, then tap a day.         |
| Change your shift pattern     | Tap **Shift Pattern**, build/select a pattern, pick a start date, Save. |
| Mark a day missed             | Tap a day → dialog → **Missed**. Red MISSED popup. |
| Unmark a missed day           | Tap the day → dialog → **Unmiss**. Green UNMISSED ✓ popup. |
| Record payment                | Tap a day → dialog → **Paid** → enter amount → **Mark Paid**. Green PAID ✓ popup + summary card (shows the previous voucher period it pays for). |
| Navigate periods              | Arrow buttons or swipe left/right on the grid.   |
| View period summary           | Tap the period label below the date range.       |
| Copy a date                   | Tap a day → dialog → **Copy Date**.              |
| Open the help guide           | Tap the `?` button (pulses on first visit).      |
| Contact support               | Tap the WhatsApp number in the footer.           |
| Toggle dark mode              | Sun/moon toggle at top-left corner.              |
| Export data                   | **Export JSON** or **Export CSV** at the bottom. |
| Import data                   | **Import Data** → select a `.json` backup file.  |
| Reset all data                | **Reset All** → confirm in the dialog.          |

---

## 🗺️ Roadmap (Implemented)

- ✅ Voucher period (20th → 19th) auto-detection
- ✅ 2-2-2 shift engine with configurable anchor
- ✅ Customizable shift patterns (presets + chip builder + live preview)
- ✅ Mid-period pattern switching via dated segments
- ✅ Missed-day toggling with red highlighting (via day action dialog)
- ✅ Paid-day tracking — record the amount (always **for the previous voucher period**), mark PAID, stop counting, show the previous period's days-to-payment summary
- ✅ Dashboard counters (present, missed, off)
- ✅ Attendance percentage and streak counter
- ✅ Period navigation (previous / next)
- ✅ Swipe gesture navigation
- ✅ Period summary tooltip
- ✅ Data export (JSON + CSV)
- ✅ Data import with validation
- ✅ Data reset
- ✅ Dark mode toggle
- ✅ Animated popups (MISSED, UNMISSED, anchor, PAID)
- ✅ Copy date (via the day action dialog)
- ✅ Accessibility (ARIA labels, roles, semantic HTML)
- ✅ Responsive design (mobile / tablet / desktop)
- ✅ Help guide modal
- ✅ Help attention (first-visit pulse + hint bubble)
- ✅ WhatsApp support link

### Known Limitations

- Shift labels are currently M / N / OFF only — custom labels and night-shift pay multipliers are planned.
- One paid record is kept per voucher period (always for the previous period); re-recording payment on the same day updates the amount, but there is no explicit "unmark paid" action.
- `missedDays` entries persist across periods — they are not automatically cleaned up (but unused entries have no visible effect in periods where they don't fall within the date range).
- Future days are rendered but not clickable.
- CSV export only covers the currently viewed period.

---

## 🤝 Contributing

Contributions are welcome:

1. Fork the repo.
2. Create a feature branch: `git checkout -b feat/your-feature`.
3. Commit with a clear message.
4. Push and open a Pull Request.

For bugs or suggestions, [open an issue](https://github.com/EngrDammie/RafaVoucherApp/issues).

---

## 📄 License

MIT License — free to use, copy, modify, and distribute.

---

## 👤 Author

**Engr. Dammie** — [github.com/EngrDammie](https://github.com/EngrDammie)

Need help? Chat with the team on **WhatsApp**: [0705 333 1253](https://wa.me/2347053331253)
