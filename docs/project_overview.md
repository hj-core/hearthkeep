# Project Overview: HearthKeep

> **Smart Household Essentials Tracker (Local-First Android)**

---

## 1. Vision

**HearthKeep** is a local-first, highly efficient Android application designed to eliminate household waste and purchase duplication. Instead of acting as a tedious physical tracking map, the app acts as a **smart status dashboard** for your home. It answers two critical everyday questions:

1. **Do we already have this?** (Preventing buying a 4th bottle of dish soap because you forgot there were two backups hidden in the pantry).
2. **Is it still safe to use?** (Notifying you before items like cold medicine, sunscreens, or baby supplies expire).

By stripping away tedious micro-location tracking, HearthKeep focuses on dead-simple inventory states, quick status toggles, and proactive expiration alerts.

---

## 2. Target Audiences & Use Cases

### Audience A: The Household Budgeter & Organizer (The "Mom" Profile)

- **Characteristics:** Manages household shopping and stock. Dislikes wasting money on expired goods or buying duplicates of items the household already owns.
- **Demographics:** Adults 28-55, primarily in multi-person households (families, roommates), responsible for weekly/monthly shopping. Typically the primary shopper — the person who notices when the pantry is bare and the one who gets annoyed at waste.
- **Estimated Addressable Market:** ~45M Android users in the US alone who self-identify as primary household shoppers (Pew, 2025). Narrows significantly to those willing to adopt a dedicated inventory tool rather than sticky notes or memory.
- **Acquisition Path:** Google Play search ("household inventory", "pantry tracker"), word-of-mouth via parenting/mom blogs and subreddits (r/Frugal, r/Organization, r/budgetfood), and productivity app roundups. No paid acquisition in MVP phase.
- **Typical Use Case (Preventing Waste & Duplicates):**
  > **The Problem:** Sarah is at the supermarket and can't remember if they have any backup body wash left. She buys another one. When she gets home, she finds three unopened bottles in the linen closet. Meanwhile, she completely forgets that the allergy medicine in the cabinet expired last month.
  >
  > **The HearthKeep Solution:** Sarah opens HearthKeep at the store. A quick search for "Body Wash" shows her current inventory status:
  >
  > - In Use: 1 bottle
  > - Unopened Backup: 3 bottles
  >
  > She puts the bottle back on the shelf, saving money. When she gets home, a notification alerts her that the allergy medicine is expiring next week, allowing her to replace it before anyone needs it.

### Audience B: The Household Consumer (The Family Member)

- **Characteristics:** Wants to know if an item is available in the house without rummaging through closets, and needs to flag when something is running low without writing it down.
- **Demographics:** Teens to adults (14+), cohabitants who aren't the primary organizer. Less invested in the app long-term — must be reactive usage with near-zero setup friction or they won't engage.
- **Acquisition Path:** Viral within the household — onboarded by the primary organizer (Audience A). No external marketing needed for this segment, but the experience must be good enough that they don't refuse to use it.
- **Retention Hook:** The shared shopping list acts as a passive feedback loop — when Mark marks something empty, Sarah sees it on the list and restocks it. Mark sees his action had real-world impact, reinforcing the behavior.
- **Typical Use Case (The Frictionless Update):**
  > **The Problem:** Mark finishes the last of the hand sanitizer. He intends to tell Sarah to buy more, but forgets 10 seconds later.
  >
  > **The HearthKeep Solution:** Mark opens the app, finds "Hand Sanitizer", and taps **Mark as Empty**. The app automatically adds it to the shared shopping list.

---

## 3. Core Features

### Category 1: Instant Hand-on-Inventory Visibility

- **Broad Static Categorization:** Items are grouped into simple, static categories that never change (e.g., Medicines, Cleaning Supplies, Toiletries, Pantry Backups). No nested shelf-by-shelf mapping.
- **Sub-Second Global Search:** A prominent search bar on the home screen. Typing "Soap" immediately displays the total quantities across the household:
  - Soap (Total: 3) -> 2 x Unopened Backup, 1 x In Use
- **Simple Quantity & State Toggles:** Users manage quantities using a simple state lifecycle:
  - Unopened (Backups in storage)
  - In Use (Currently open and being consumed)
  - Empty (Depleted and needs replacement)

### Category 2: Expiration & Waste Prevention

- **One-Tap Expiration Inputs:** A smart picker with quick presets (e.g., +3 Months, +6 Months, +1 Year, +2 Years) so users can log expiration dates in two taps rather than manual calendar typing.
- **Color-Coded Health Dashboard:**
  - Red (Expired): Needs immediate replacement/disposal.
  - Yellow (Expiring Soon): Expires within 30 days.
  - Green (Stable): Safe to use.
- **Daily Expiry Sweeper (Background Worker):** A lightweight background process that scans the database once a day and sends a single, grouped notification if any items have entered the Expiring Soon or Expired states.

### Category 3: The Automated Shopping List

- **Auto-Generation:** Any item marked as Empty or Expired automatically populates a dynamic shopping list.
- **Quick Restock:** When an item is purchased, a single tap on the shopping list transitions it back to Unopened backup stock, resetting the cycle.

---

## 4. Technical Architecture (Local-First Android Stack)

To ensure maximum speed, complete user privacy, and zero server hosting costs, the app is built on a modern, local-first Android architecture.

### Application Layer Architecture

1. **User Interface Layer:**
   - Built with Jetpack Compose for a clean, modern, and reactive declarative UI dashboard.
2. **State & Business Logic Layer:**
   - Implemented via Android ViewModels and Repository patterns to handle stream-based data updates.
3. **Local Storage Layer:**
   - Powered by Room Database (an official Google abstraction layer over SQLite) to maintain categories, item details, quantities, and expiration timestamps strictly on the device.

### Framework Selection

- **UI Framework:** Jetpack Compose
- **Database:** Room Database (SQLite)
- **Language:** Kotlin
- **Scheduler:** WorkManager (Used to execute the daily background calculation for expiring items efficiently without draining the device battery)

---

## 5. Execution & Scope Control (2-Month Timeline)

### Development Milestone Breakdown

#### Month 1: Database & Core Mechanics

- Establish the local database configurations and Room DB setup.
- Implement Category and Item database schemas along with data relationship models.
- Write core state transfer logic (handling state changes between Unopened, In Use, and Empty).
- Develop foundational Create, Read, Update, and Delete (CRUD) operations.

#### Month 2: UI, Notifications & Polish

- Build out the responsive Jetpack Compose user interface screens.
- Connect visual health indicators and expiration threshold color-coding to the UI.
- Integrate the WorkManager background engine for daily automated expiration alerts.
- Conduct end-to-end internal testing, bug fixing, and execute the Google Play Store launch.

### Phase Schedule

| Phase       | Duration     | Core Deliverables          | Focus                                                                                                                           |
| :---------- | :----------- | :------------------------- | :------------------------------------------------------------------------------------------------------------------------------ |
| **Phase 1** | Weeks 1 to 4 | Database & Core Mechanics  | Setup local schemas, database models (Room DB), data relationships, and basic Create-Read-Update-Delete (CRUD) state logic.     |
| **Phase 2** | Weeks 5 to 8 | UI, Notifications & Polish | Design Jetpack Compose screens, visual expiration cues, integrate WorkManager background scheduling, and deploy to Google Play. |

---

## 6. Risk Identification & Mitigation

| Risk                                                                                                                                                                          | Severity | Likelihood | Mitigation                                                                                                                                                                                                                                                                                                    |
| :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------- | :--------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Android Fragmentation** — OS version differences, manufacturer-specific quirks (e.g., WorkManager battery optimization on Xiaomi/Huawei), and varying notification behavior | High     | Medium     | Target API 26+ (Android 8.0) covering ~97% of active devices. Test early on low-end hardware and aggressive OEM ROMs (Xiaomi, Samsung, Huawei). Use `ActivityManager.isBackgroundRestricted()` checks and guide users through battery optimization exemptions.                                                |
| **Data Loss (No Cloud Backup)** — Phone reset, app uninstall, or device failure destroys all inventory data with no recovery path                                             | Critical | Medium     | Implement local backup/restore via `Room.databaseBuilder().addCallback()` with a JSON export/import feature accessible through Settings. Consider Android's `BackupAgent` or `Auto Backup` for automatic cloud backup to Google Drive (user must still opt in to a Google account on device).                 |
| **User Onboarding Abandonment** — Users install, face a cold empty state with no items, and churn before seeing value                                                         | High     | High       | Ship with a curated set of sample items and categories pre-populated (e.g., "Cleaning Supplies: Dish Soap, All-Purpose Cleaner"). Provide a one-tap "Import Starter Pack" option. Show an interactive walkthrough that demonstrates the search bar and state toggles on sample data.                          |
| **Low Long-Term Retention** — Users enter their inventory once, then forget the app exists, returning to old habits                                                           | Medium   | High       | Leverage the Daily Expiry Sweeper as a recurring touchpoint — even if users aren't actively adding items, the expiration notification brings them back. Explore optional weekly digest notification ("5 items expiring this month"). The shopping list auto-population creates a natural re-engagement cycle. |
| **Manual Data Entry Fatigue** — Users tire of typing item names and expiry dates, leading to incomplete inventory                                                             | Medium   | Medium     | Prioritize quick preset pickers for expiry (+3M, +6M, +1Y) to minimize typing. As a post-MVP enhancement, consider barcode scanning via ML Kit for automatic item name and expiry lookup, but defer in v1 to keep scope tight.                                                                                |
| **Google Play Store Rejection** — WorkManager background processing may trigger Play policy reviews, or app may be rejected for insufficient functionality                    | Low      | Low        | Ensure compliance with Play Store background-restriction policies. WorkManager's `PeriodicWorkRequest` with a minimum 15-minute interval is within guidelines. Avoid any deceptive ads, data collection, or permission overreach (no internet, location, or camera permissions in v1).                        |
