# 📅 RafaVoucherApp

A web-based **Voucher Attendance Tracker** that helps you monitor your work attendance over a monthly voucher period. Built with a 2-2-2 shift cycle engine, one-tap missed-day toggling, and a clean dashboard summary — all running entirely in your browser with no backend and no sign-up.

> Live demo: open `index.html` in any modern browser. That's it — no install, no build step.

---

## ✨ Features

- **Monthly Voucher Period** — Automatically tracks the 20th of one month through the 19th of the next, based on today's date.
- **2-2-2 Shift Cycle** — Visualizes a repeating 6-day rotation:
  - Days 0–1 → Morning (`M`)
  - Days 2–3 → Night (`N`)
  - Days 4–5 → Off (`OFF`)
- **Configurable Anchor Date** — Pick any day as the "Cycle Start" and the whole calendar recomputes instantly.
- **One-Tap Missed Days** — Click any working day to toggle it as missed (rendered in bold crimson). Toggle off to restore.
- **Live Dashboard** — Big counters for **Days Present**, **Days Missed**, and **Days Off**, scoped to the current voucher period.
- **Offline-First** — Everything is saved in `localStorage`, so your data stays on your device and works without an internet connection.
- **Single-File App** — The entire UI, logic, and styling live in one portable HTML file.

---

## 🚀 Getting Started

### Run Locally
1. Clone the repo:
   ```bash
   git clone https://github.com/EngrDammie/RafaVoucherApp.git
   cd RafaVoucherApp
   ```
2. Open `index.html` in your browser:
   - Double-click the file, **or**
   - Serve it for a cleaner experience:
     ```bash
     python3 -m http.server 8000
     # then visit http://localhost:8000
     ```

### Usage
1. On first open, the dashboard shows `--` for all counters because no cycle is set.
2. Tap **Set Cycle Start** in the Attendance Grid header.
3. Select any past "M" (morning) day — this becomes the anchor for the 2-2-2 rotation.
4. The calendar fills in automatically with `M`, `N`, and `OFF` labels and colors.
5. Tap any working day (`M` or `N`) to mark it **MISSED**. Tap again to undo.
6. The dashboard counters update live with every change.

---

## 🧱 Tech Stack

| Layer       | Technology                          |
|-------------|-------------------------------------|
| Markup      | Semantic HTML5                      |
| Styling     | [Tailwind CSS](https://tailwindcss.com) (via CDN) |
| Font        | Google Poppins                      |
| Logic       | Vanilla JavaScript (no frameworks) |
| Persistence | Browser `localStorage`              |

**Why this stack?** Zero build tooling, no dependencies to install, and the whole app fits in a single file — making it trivial to deploy, share, or embed anywhere.

---

## 🗂️ Project Structure

```
RafaVoucherApp/
├── index.html      # The entire app (HTML + Tailwind + JS)
└── README.md       # You are here
```

---

## 💾 How Data Is Stored

All state lives in your browser's `localStorage`:

| Key           | Type                | Purpose                                            |
|---------------|---------------------|----------------------------------------------------|
| `anchorDate`   | `string` (`YYYY-MM-DD`) | The selected cycle-start day. Drives shift labels. |
| `missedDays`   | `JSON array` of date strings | Voucher days the user marked as missed.           |

Clearing your browser storage will reset the app to first-run state.

---

## ⚙️ How the Core Logic Works

- **Voucher period** — `getVoucherPeriod()` computes start (20th of the current or previous month) and end (19th of the following month) based on today's date.
- **Shift cycle** — `getShiftInfo(targetDate)` computes the day difference from the anchor date, modulo 6, mapping the result to `M`, `N`, or `OFF`.
- **Missed-day toggling** — Each calendar day renders as a button. Clicking a working (`M`/`N`) day flips its membership in the `missedDays` array, persists it, and re-renders.
- **Dashboard counters** — Rerun on every render pass by walking the voucher period and categorizing each day.

---

## 🛣️ Roadmap / Known Limitations

- The voucher period (20th → 19th) and shift cycle (2-2-2) are currently hard-coded and not configurable.
- `missedDays` entries are never cleaned up, so stale markings from past voucher periods can persist in storage.
- Only the current voucher period is visible — no way to browse past or future periods.
- No data export, import, or reset-to-defaults option yet.

---

## 🤝 Contributing

This is a small, focused project — but improvements are welcome.

1. Fork the repo.
2. Create a feature branch: `git checkout -b feat/your-feature`.
3. Commit your changes with a clear message.
4. Push and open a Pull Request describing what and why.

For bug reports or suggestions, please [open an issue](https://github.com/EngrDammie/RafaVoucherApp/issues).

---

## 📄 License

This project is released into the public domain under the **MIT License** — see the intent below. You're free to use, copy, modify, and distribute it.

> A formal `LICENSE` file can be added on request.

---

## 👤 Author

**Engr. Dammie** — [github.com/EngrDammie](https://github.com/EngrDammie)
