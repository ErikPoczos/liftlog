# LiftLog

An offline Android workout tracker for intermediate lifters.

## What problem am I solving?

Serious lifters can't log RPE/RIR, unilateral sets, unlimited custom exercises and per-gym PRs in one free tracker. Popular apps like Hevy put these features behind a paywall or don't have them at all.

## Who are my users?

**Intermediate to advanced lifters** (roughly 1+ years of consistent training) who:

- autoregulate their training with **RPE / RIR**, but don't want to be forced to log them every set,
- train at **more than one gym**, where machines and cable stacks aren't comparable, so a PR on one gym's leg press means nothing at another,
- use many accessory or variation exercises and need **unlimited custom exercises**,
- care about **personal records** and want to see them the moment they happen,
- don't want to pay a subscription for basic tracking features.

Beginners who need guided programs and coaching aren't the target audience.

## How do I want to solve it? (MVP)

A single-user, fully offline Android app focused on **fast, detailed workout logging** and **per-gym PR tracking**.

### Gyms
- Create, edit and delete gyms.
- A gym's location is **optional** and can be set by:
  - **using the current location** (GPS), or
  - **address search** (Android `Geocoder`, e.g. "Aupark Bratislava").
- **GPS auto-detection** can be turned on or off in Settings. When it's on, the app does a single location check when a workout starts and pre-selects the nearest gym within its radius (~150 m).
- The gym can always be **selected or changed manually**. If location permission is denied or no gym matches, the app falls back to the last-used gym.
- Every workout is tagged with exactly one gym.

### Exercise library
- About **40 seeded common exercises** (bench press, squat, deadlift, OHP, rows, curls...).
- **Unlimited custom exercises**.
- Each exercise has: name, primary muscle group, equipment type (barbell / dumbbell / machine / cable / bodyweight), and a **default unilateral flag**.
- Seeded exercises can be edited or hidden.
- Exercises are **shared across all gyms**. Only history and PRs are per gym.
- Search and filter by muscle group and equipment.

### Workout logging
- Start an **empty workout** or **repeat a past workout** (copies its exercises and sets as a starting point).
- Only **one active workout** at a time. Every change is **saved immediately**, so the workout survives the app being killed and resumes on reopen.
- Each set contains:
  - **Weight** in kg (decimals allowed; for bodyweight exercises this is *added* weight, 0 allowed)
  - **Reps**
  - **RPE** (optional, 6–10 in 0.5 steps)
  - **RIR** (optional: 0, 1, 2, 3, 4, 5+ or **F = failure**; failure is a separate value from 0)
  - **Set type**: warm-up / working / backoff
  - **Unilateral tag**: pre-filled from the exercise's default, can be changed per set. For unilateral sets, reps mean reps **per side**.
  - **Done** checkbox: only checked sets count.
- A **"last time" hint** for each exercise, showing the previous performance of that exercise at the current gym (e.g. `80 kg × 8 @ RPE 8`).
- **Finish** stores the end time. If there are unchecked sets, the app **asks** whether to remove them.
- **Discard** deletes the active workout (after a confirmation).

### Automatic workout naming
- The name is assigned from the workout's **end time** when you finish:
  - 05:00–11:59 → *Morning workout*
  - 12:00–16:59 → *Afternoon workout*
  - 17:00–21:59 → *Evening workout*
  - 22:00–04:59 → *Night workout*
- While the workout is in progress it shows a placeholder name.
- The user can **rename** a workout at any time. A manual name is never overwritten.

### Personal records
- PRs are tracked **per exercise, per gym, and separately for unilateral vs. bilateral sets**.
- PR types:
  - **Heaviest weight**
  - **Best estimated 1RM** (Epley: `weight × (1 + reps / 30)`)
  - **Rep PRs**: the best weight for each rep count (1–12)
- Only checked **working** and **backoff** sets count. Warm-ups are ignored.
- PRs are **computed from the set history** (queries), not stored separately, so editing or deleting old workouts keeps them correct.
- PRs appear in:
  - a **PR badge** on a set during the workout, as soon as the set is checked,
  - the **workout summary** after finishing (scroll through every exercise with its PR badges; also available for past workouts),
  - the **exercise detail** screen: gym selector, PR values, rep-PR table, and the set history for that exercise at that gym.

### History
- A list of finished workouts (newest first) with name, gym, date and duration.
- Filter by gym.
- Past workouts can be fully **edited and deleted**.

## What will NOT be in the MVP (non-goals) and why

| Non-goal | Why |
|---|---|
| **Statistics and charts** (volume, progress graphs, muscle heatmaps) | A large feature on its own. PR tracking covers the most important "am I progressing?" question for now. |
| **Rest timer** | A reliable timer needs a foreground service, notifications and the notification permission. That's high complexity for a feature you can replace with any phone timer. |
| **Routines / templates** (named plans, target sets, folders, scheduling) | "Repeat a past workout" covers most of the need without a new data model. |
| **Map picker for gym location** | Needs a maps SDK and an API key. Current location + address search is enough. |
| **Background geofencing** | Background location permission and battery cost. A single check when the workout starts is enough. |
| **Accounts, backend, cloud sync** | Offline-only keeps the app simple, private and free to run. Sync can be added later on top of the local DB. |
| **kg / lb switching** | Target users train in kg. Unit conversion touches every screen and every PR calculation. |
| **Drop sets, failure sets and other advanced set types** | Warm-up / working / backoff plus RIR = F covers the real need. More types complicate PR rules. |
| **Exercise images, videos, instructions** | Content work, not engineering. Target users already know the exercises. |
| **iOS** | Native Android only, to keep one codebase and focus on one platform. |
| **Social features** (sharing, feed, followers) | Needs a backend and accounts. Not part of the core problem. |

## What technologies do I need?

| Area | Technology |
|---|---|
| Language | **Kotlin** |
| UI | **Jetpack Compose** + **Material 3** |
| Architecture | MVVM: `ViewModel` + Kotlin **Coroutines** / **Flow**, repository layer |
| Local database | **Room** (SQLite). PRs computed with SQL queries over the set history |
| Settings | **Jetpack DataStore** (Preferences): GPS auto-detect toggle, last-used gym |
| Navigation | **Navigation Compose** |
| Dependency injection | **Hilt** |
| Location | **Fused Location Provider** (Google Play Services Location) for current location; Android **`Geocoder`** for address search |
| Permissions | Runtime `ACCESS_FINE_LOCATION` / `ACCESS_COARSE_LOCATION` (foreground only) |
| Build | **Gradle** (Kotlin DSL) with a version catalog |
| IDE | **Android Studio** |
| Testing | JUnit (unit tests for PR / 1RM / auto-naming logic), Room in-memory DB tests |
| Version control | **Git** + **GitHub** |

- **Package / application ID:** `app.liftlog`
- **Platform:** Android
- **No backend:** all data stays on the device.
