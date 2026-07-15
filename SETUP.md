# Envelope Budget — Setup Guide

This is a complete SwiftUI + SwiftData iOS app implementing the envelope
budgeting method: variable/fixed/saving envelopes, progress bars everywhere,
a dedicated savings overview, and CSV export that opens cleanly in Google
Sheets.

## Requirements

- Xcode 15.2+ (for SwiftData and Swift Charts)
- iOS 17+ target (SwiftData requires iOS 17)
- No third-party dependencies — everything uses Apple frameworks only
  (SwiftUI, SwiftData, Charts). Nothing to install via SPM/CocoaPods.

## 1. Create the Xcode project

1. Open Xcode → **File → New → Project → iOS → App**.
2. Product Name: `EnvelopeBudget` (or whatever you like — just make sure it
   matches if you rename `EnvelopeBudgetApp.swift`).
3. Interface: **SwiftUI**. Storage: choose "None" (we already wired up
   SwiftData manually in `EnvelopeBudgetApp.swift`, so don't let Xcode
   generate its own).
4. Set **Minimum Deployment** to iOS 17.0 in the project's target settings
   (General tab).

## 2. Add the files

1. Delete the default `ContentView.swift` and the default `Item.swift` that
   Xcode generates (if you chose SwiftData storage) — we don't need them.
2. Drag the entire `EnvelopeBudget` folder from this package into your
   Xcode project navigator (drop it under the top-level project group).
   In the dialog, check **"Copy items if needed"** and **"Create groups"**.
3. Confirm your target membership includes all the new files (Xcode does
   this automatically when you drag them in).

Your file tree inside Xcode should look like:

```
EnvelopeBudget/
  EnvelopeBudgetApp.swift
  Models/
    Envelope.swift
    Transaction.swift
    CategoryData.swift
  Utilities/
    Theme.swift
    Formatters.swift
    CSVExporter.swift
    ShareSheet.swift
  Components/
    ProgressBarView.swift
    EnvelopeRowView.swift
    CategoryIconView.swift  (defined inside EnvelopeRowView.swift)
  Views/
    MainTabView.swift
    Envelopes/
      EnvelopesView.swift
      EnvelopeDetailView.swift
      CreateEnvelopeView.swift
    Savings/
      SavingsView.swift
    Transactions/
      AddTransactionView.swift
      DailyTransactionsView.swift
      TransactionDetailView.swift
    Stats/
      StatView.swift
    Budget/
      BudgetView.swift
    Profile/
      ProfileView.swift
```

## 3. Run it

Build and run on the iOS 17+ simulator (or a device). The app seeds a few
sample envelopes and you can start adding transactions immediately via the
pink **+** button.

## How the pieces fit together

- **Envelope method core**: `Envelope` model tracks `budgetedAmount` vs
  `currentAmount` for spending envelopes, and `currentAmount` vs `goalAmount`
  for savings envelopes. `envelope.progress` (0...1) drives every progress
  bar in the app — one computed property, one visual language everywhere.
- **Adding a transaction** automatically debits/credits the linked
  envelope's `currentAmount` (see `AddTransactionView.saveTransaction()`).
  Editing or deleting a transaction correctly reverses that effect
  (`TransactionDetailView`).
- **Savings tab** (`SavingsView`) totals every saving envelope against its
  goal and shows one big progress bar plus a per-goal breakdown — this is
  the "look at all my savings" screen you asked for.
- **Export**: `CSVExporter` builds a CSV with an ENVELOPES section and a
  TRANSACTIONS section, written to a temp file, then handed to the native
  iOS share sheet (`ShareSheet.swift`). From there:
  - Tap **Save to Files** → later upload to Google Drive → open in Sheets, or
  - If you have the Google Drive app installed, it appears directly in the
    share sheet, and opening the saved file in Drive with "Open with →
    Google Sheets" converts it to a live spreadsheet.
  - This needs zero API keys or OAuth setup — it's the fastest path to
    "clean spreadsheet in Google Sheets."

## Going further (not included, but straightforward next steps)

- **Direct Google Sheets API export** (skip the share-sheet step entirely
  and push rows straight into a live Sheet): add the `GoogleSignIn-iOS` SPM
  package, set up an OAuth client ID in Google Cloud Console, and POST to
  `https://sheets.googleapis.com/v4/spreadsheets/{id}/values/A1:append`
  with the same rows `CSVExporter` builds. Happy to build this out fully if
  you decide you want it — it's maybe another hour of code once you've
  created the Google Cloud project.
- **iCloud sync across devices**: change `ModelConfiguration` in
  `EnvelopeBudgetApp.swift` to use a CloudKit container — SwiftData +
  CloudKit sync is largely automatic once you enable the iCloud capability.
- **Recurring fixed envelopes / auto-reset each month**: currently fixed and
  variable envelope balances persist until you manually adjust them. A
  simple monthly reset job (check `createdAt` vs current month on app
  launch) would automate that.
