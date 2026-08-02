# 📅 RafaVoucherApp

A full-featured **Voucher Attendance Tracker** built as a single-file HTML app. Manage your 2-2-2 shift schedule, mark missed days, track streaks, view attendance percentages, and export data — all offline, in your browser, with no backend or sign-up.

> **Live demo:** open `index.html` in any modern browser. No install, no build step.

---

## ✨ Features

- **Monthly Voucher Period** — Automatically detects the current period (20th–19th) based on today's date.
- **2-2-2 Shift Cycle** — Repeating 6-day rotation: 2 Morning (M), 2 Night (N), 2 Off (OFF).
- **Configurable Anchor Date** — Tap any day as your cycle start; the entire calendar recomputes instantly.
- **One-Tap Missed-Day Toggle** — Tap any M/N day to mark it missed (red). Tap again to unmark (green confirmation popup).
- **Attendance Percentage** — See your present vs. total-working-day percentage below the main counter.
- **Present Streak Counter** — Consecutive present-day streak (backward from today) shown with a fire icon.
- **Period Navigation** — Use arrow buttons or **swipe left/right** on the calendar grid.
- **Period Summary Tooltip** — Tap the period label to see M/N/OFF/Present/Missed counts.
- **Anchor Confirmation Popup** — Green "ANCHOR ✓" animation when setting a new start date.
- **Missed / Un-missed Popups** — Animated MISSED (red) and UNMISSED ✓ (green) feedback on tap.
- **Copy Date** — Double-tap any day button to copy its YYYY-MM-DD date to your clipboard.
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
5. Tap any working day (M or N) to mark it **MISSED** (red). Tap again to unmark.
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
| `theme`       | `"light"` or `"dark"`       | User's theme preference.                      |
| `helpIntroSeen`| `"1"` (set once)           | Marks that the help guide has been opened once.|

> Clearing browser storage / cookies resets the app to first-run state. Export a backup before doing so.

---

## ⚙️ How It Works

### Voucher Period (`getVoucherPeriod`)
Computes a start date (20th of the current or previous month) and end date (19th of the following month) based on today. If today is before the 20th, the period is the previous month's 20th → this month's 19th.

### Shift Engine (`getShiftInfo`)
Calculates the difference in days between the anchor date and the target date, takes `diff % 6`, and maps:
- `0` or `1` → **M** (Morning), amber
- `2` or `3` → **N** (Night), indigo
- `4` or `5` → **OFF**, gray

### Streak Calculation (`calcStreak`)
Walks backward from the period end date (or today, whichever is earlier), skipping OFF days and stopping at the first missed day, counting consecutive present days.

### Rendering (`renderCalendar`)
Clears and rebuilds the calendar grid. Every day is a `<button>` with `data-date`. Future days are muted and not clickable. Past days get shift-based colors. The friendly banner is shown only when no anchor is set and the user is not in cycle-setting mode.

### Event Delegation
A single `onclick` handler on `#calendar-grid` uses `e.target.closest('button[data-date]')` to handle all day clicks, whether toggling missed status or setting the anchor date. A separate `click` listener handles double-tap date copying.

---

## 🖱️ Interactive Features

| Action                        | How                                             |
|-------------------------------|--------------------------------------------------|
| Set / change cycle start      | Tap **Set Cycle Start**, then tap a day.         |
| Mark a day missed             | Tap any M or N day. Red MISSED popup appears.    |
| Unmark a missed day           | Tap the red day again. Green UNMISSED ✓ popup.   |
| Navigate periods              | Arrow buttons or swipe left/right on the grid.   |
| View period summary           | Tap the period label below the date range.       |
| Copy a date                   | Double-tap any day button.                       |
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
- ✅ Missed-day toggling with red highlighting
- ✅ Dashboard counters (present, missed, off)
- ✅ Attendance percentage and streak counter
- ✅ Period navigation (previous / next)
- ✅ Swipe gesture navigation
- ✅ Period summary tooltip
- ✅ Data export (JSON + CSV)
- ✅ Data import with validation
- ✅ Data reset
- ✅ Dark mode toggle
- ✅ Animated popups (MISSED, UNMISSED, anchor)
- ✅ Copy date on double-tap
- ✅ Accessibility (ARIA labels, roles, semantic HTML)
- ✅ Responsive design (mobile / tablet / desktop)
- ✅ Help guide modal
- ✅ Help attention (first-visit pulse + hint bubble)
- ✅ WhatsApp support link

### Known Limitations

- The 2-2-2 cycle, anchor-based calculation, and voucher period boundaries are hard-coded.
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
