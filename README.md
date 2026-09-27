# My Wallet

*Know your money. Control your money.*

A privacy-first personal finance app. The home screen never shows dollar amounts — just category cards. Salary is automatically split 50/30/10/10 into Savings, Expenses, Safe to Spend, and Tithe, and every rule about which bucket can fund what is enforced in code, not left to memory.

Single-file, no build step, no server, no dependencies. Open it and it runs.

## Features

- **Privacy-first home screen** — categories only, no amounts until you tap in
- **Automatic salary allocation** — configurable split (default 50/30/10/10), applied every time you log a salary
- **Four buckets** — Savings, Expenses, Safe to Spend, Tithe — that never silently fund each other
- **Debt tracking** — "You Owe" and "Owed to You," each with due dates
- **Debt safety rule** — a debt payment can never silently drain the wrong bucket; if Savings is short, you're shown the exact shortfall and asked what to do
- **Expense reserve rollover** — leftover expense budget automatically rolls into Savings at the start of each month (and you can trigger it manually anytime, for Expenses or Safe to Spend)
- **Due-date reminders** — a banner appears when a debt is due within 3 days, plus an optional browser notification
- **App lock** — PIN code, with a simulated Face ID button (this is a prototype flow, not real biometric verification — see Security notes below)
- **Profile photo** — used as your avatar and as the app's background art
- **Full transaction diary** — every money movement is logged with source/destination
- **Export / Import backup** — download a JSON snapshot of your data anytime, and restore from one

## Running it

There's nothing to install. Pick one:

- **Just open it locally** — download `index.html` and double-click it. It runs entirely in your browser.
- **GitHub Pages (free)** — push this repo to GitHub, go to *Settings → Pages*, and point it at the `main` branch. You'll get a free `https://yourname.github.io/mywallet` URL.
- **Netlify / Vercel (free)** — drag the folder onto [netlify.com/drop](https://app.netlify.com/drop), or connect the repo — either gives you a free hosted URL in seconds.
- **Add to your phone's home screen** — open the hosted URL in Safari (iPhone) or Chrome (Android) and use "Add to Home Screen." It launches full-screen, like a native app.

## Data storage & sync — please read

All data lives in your browser's `localStorage`, scoped to whatever URL you open the app from. That means:

- Your data is private to your device and browser by default — nothing is sent anywhere.
- It is **not synced across devices** out of the box. Opening the app on your phone and your laptop gives you two separate, independent datasets.
- Browsers (especially iOS Safari) can clear `localStorage` for sites you haven't opened in a while, or under low storage. **Export a backup regularly** — Profile → Export backup file — since that's the one copy that can't silently disappear.

If you want real cross-device sync, you'll need to add a small free backend yourself — the code is a single `S` (state) object that gets saved via `save()`, so wiring that function to something like [Supabase](https://supabase.com) or [Firebase](https://firebase.google.com) (both have generous free tiers) is a contained change. This isn't included out of the box to keep the project dependency-free and instantly runnable from a single file.

## Security notes (important)

The PIN lock and "Face ID" button are a **prototype-level UI flow**, not real security:
- The PIN is stored in plain text in `localStorage` inside your state object. Anyone with access to the browser's dev tools or storage can read it.
- The "Face ID" button doesn't call any real biometric API — it just unlocks. It's a placeholder for what that flow would feel like.

If you plan to use this for real financial tracking and want real device-level security, the PIN should be replaced with a proper hashed-credential check, and the Face ID button should call the [WebAuthn API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Authentication_API), which does support real platform biometrics (Face ID / Touch ID / Windows Hello) from a web page. That's a good first contribution if you'd like to open a PR.

## Project structure

```
index.html   — the entire app (HTML, CSS, and JS in one file)
LICENSE      — MIT
README.md    — this file
```

## Contributing

PRs welcome. Ideas that would be great additions:
- Real WebAuthn-based biometric unlock
- A "People" page that shows both sides of a relationship (owes you + you owe them) on one profile
- A monthly Analysis/spending-breakdown view
- Optional backend sync (Supabase/Firebase) behind a toggle, so the no-dependency default stays intact

## License

MIT — see [LICENSE](./LICENSE). Do whatever you'd like with it.
