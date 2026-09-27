# My Wallet — Project Blueprint

*Know your money. Control your money.*

This documents what's actually built, as a reference for extending it, auditing the money logic, or onboarding a contributor.

---

## 1. Concept

My Wallet is a privacy-first personal finance allocator, not a balance tracker. The home screen shows category *cards*, never dollar amounts — you tap in to see numbers. Money is split into buckets on entry (mainly via an automatic salary split) and the app enforces strict rules about which bucket may ever fund which action, so nothing silently moves between them.

## 2. Architecture

- **Single file**: `index.html` — HTML, CSS, and JavaScript, no build step, no external runtime dependencies.
- **Fonts/icons**: system fonts and emoji only — nothing to load, nothing to break offline.
- **State**: one JavaScript object, `S`, holding the entire app. Every screen is a pure function of `S`, re-rendered into `#screen` on change. There is no framework — `render()` just rewrites `innerHTML`.
- **Persistence**: `localStorage` (key `mywallet_v1`) is the source of truth on-device. When running inside a Claude artifact, a second copy syncs to a private per-user cloud document (see §6); outside that environment, this is a no-op and the app runs on `localStorage` alone.

## 3. Data model (the `S` object)

```js
{
  name: string,
  currency: string,               // e.g. "$"
  avatar: string|null,            // data: URL, JPEG, capped ~200x200
  security: {
    enabled: bool,                 // PIN lock on/off
    pin: string|null,              // 4-digit, PLAIN TEXT (see Security notes)
    faceId: bool                   // simulated biometric button toggle
  },
  alloc: { savings, expenses, safeToSpend, tithe },  // percentages, sum to 100
  buckets: { savings, expenses, safeToSpend },        // running balances
  tithe: { due, paid },
  owedToYou: [ { id, name, amount, expected(date) } ],
  youOwe:    [ { id, name, amount, due(date) } ],
  txns: [ { type, desc, amount, sign(1|-1), src?, dest?, date, time } ],
  monthStamp: "YYYY-MM",          // last month the rollover ran for
  lastReminderNotify: "YYYY-MM-DD"|null,
  lastModified: epoch-ms          // used to resolve local-vs-cloud conflicts
}
```

Money owed *to* the user is never added into `buckets` — it's tracked separately in `owedToYou` and only crosses into `savings` when explicitly marked received.

## 4. Screens

| Tab | Contents |
|---|---|
| **Home** | Reminder banner (debts due ≤3 days) + 6 privacy cards: Savings, Expenses, Safe to Spend, Tithe, You Owe, Owed to You. Tapping a card opens its detail sheet with balance + history. |
| **Diary** | Every transaction ever recorded, newest first. |
| **Profile** | Photo, name/currency, allocation percentages, App Security, Backup (export/import), Reset. |

Adding a transaction (the **+** button) is a bottom sheet with a type selector: Salary, Savings, Expense, Safe to Spend, Debt repayment received, Debt payment.

## 5. Core money rules (enforced in code, not just documented)

1. Salary uses the configured percentage split (default 50 / 30 / 10 / 10) — never manual math.
2. A direct "Savings" entry (gifts, bonuses, manual top-ups) goes straight to Savings — no split, no other bucket touched.
3. Money someone owes the user is **not** counted as available money until marked received.
4. Debt repayments received always land in Savings.
5. Debt payments always come from Savings.
6. If Savings can't cover a debt payment, the app shows the exact shortfall and offers to pay the available partial amount — it never silently pulls from another bucket.
7. Normal expenses draw from the Expenses reserve only; they never touch Safe to Spend.
8. Safe to Spend is discretionary — nothing auto-deducts from it except a direct spend entry or a manual rollover the user triggers.
9. Tithe is only ever funded by the salary split; it is never auto-deducted from other income.
10. Unused Expenses reserve auto-rolls into Savings at the start of each new calendar month (`monthStamp` guards against double-running it), and can also be rolled manually at any time — the same manual rollover is available for Safe to Spend.
11. No bucket ever silently funds another. Every movement is explicit and logged.
12. Every money movement creates a Diary entry with source/destination.

## 6. Security (read this before trusting it with real data)

- The PIN lock and "Face ID" button are a **prototype UI flow**, not real security. The PIN is stored in plaintext inside `S.security.pin` in `localStorage`. Anyone with access to browser dev tools or device storage can read it.
- "Face ID" doesn't call any real biometric API — it's a placeholder for what that flow feels like.
- **To make this real**: replace the PIN check with a hashed credential, and wire the Face ID button to the [WebAuthn API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Authentication_API), which does support real Face ID / Touch ID / Windows Hello from a web page.

## 7. Backup & sync

- **Local**: every `save()` writes the full `S` object to `localStorage` immediately.
- **Cloud (Claude-hosted only)**: `save()` also pushes to a private per-account document (`data/users/<id>/wallet`) that only that account can ever read — not even the artifact's owner if it were shared. On load, the app compares `lastModified` between the local copy and the cloud copy and keeps whichever is newer, pushing the other one up/down to match. This requires an internet connection to run; offline edits queue locally until the next successful sync.
- **Manual export/import**: Profile → Export backup file produces a `.json` snapshot (via the platform's file-save prompt, or a fallback alert if unavailable); Import backup file restores from one. This is the only backup method available once self-hosted outside Claude.

## 8. File structure

```
index.html    — the entire application
README.md     — setup, hosting, and usage docs
LICENSE       — MIT
BLUEPRINT.md  — this file
```

## 9. Known gaps / good first contributions

- **People page** — currently "You Owe" and "Owed to You" are separate lists; a unified per-person profile showing both sides doesn't exist yet.
- **Monthly Analysis** — no spending-breakdown/income-vs-allocation report screen yet.
- **Real biometric unlock** — see Security notes above.
- **Cross-device sync outside Claude** — would need a small free backend (Supabase/Firebase both fit) wired into `cloudSave()`/`initCloud()`.
- **Multi-currency / multi-account** — not supported; one currency symbol, one set of buckets.

## 10. License

MIT — see `LICENSE`. Do whatever you want with it.
