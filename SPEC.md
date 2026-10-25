# LiftLog – Requirements Specification

LiftLog is a free, fully offline Android app for intermediate and advanced lifters. It logs strength workouts in detail: weight, reps, optional RPE/RIR, set type and unilateral sets. It keeps an unlimited custom exercise library and tracks personal records (PRs) separately for each gym. There is one user per device, with no accounts and no backend. This document says **what** the app must do and how each requirement is tested. The original project pitch is in [project.md](project.md). The architecture, data model and repository contract are in [DESIGN.md](DESIGN.md). Where this document and project.md differ, this document wins. The backlog lives in GitHub issues at https://github.com/ErikPoczos/liftlog.

---

## 1. Problem

Serious lifters can't find one free tracker that has all of these together:

- **optional RPE/RIR** logging,
- **unilateral sets** (reps per side),
- **unlimited custom exercises**,
- **per-gym personal records**. Machines and cable stacks differ between gyms, so a PR on one gym's leg press means nothing at another.

Popular apps such as Hevy charge for these features or don't offer them at all.

**Goal and success metric.** By the end of the winter semester (**2026-12-18**), LiftLog fully replaces Hevy for the author's own training. Success means **100% of workouts are logged in LiftLog for 4 weeks in a row**. This is self-assessed by the author, which is accepted as the metric. The 4-week run starts on 2026-11-16, after milestone M2 (see [Constraints](#4-constraints)).

## 2. Users and roles

There is exactly **one role: Lifter**.

| Role | Description |
|---|---|
| **Lifter** | An intermediate or advanced lifter with 1+ years of steady training. Adjusts training by RPE/RIR but doesn't want to log it on every set. Trains at **several gyms**. Uses many accessory and variation exercises. Cares about seeing PRs the moment they happen. Doesn't want to pay a subscription for basic tracking. |

- One user per device. There are no accounts, no login, no admin and no multi-user features.
- Beginners who need guided programs or coaching are **not** the target audience.

## 3. Features

> **Numbering:** feature IDs match the GitHub story IDs (`Epic.Story`, e.g. 4.7 = "Story 4.7"). Epic 1 holds developer-facing foundation stories (1.1–1.4), so it is listed under [Constraints](#4-constraints). The exception is 1.5 (theme), which users see and is listed here.

Each feature is one sentence from the Lifter's point of view, followed by acceptance criteria. Together, the criteria are the test plan.

### Settings / appearance (Epic 1)

**1.5** As a lifter, I want to choose a light or dark theme, so that the app looks the way I like regardless of my phone's system theme.

Acceptance criteria
- [ ] Given a fresh install, when I launch the app for the first time, then it uses the **dark** theme, even if the phone's system theme is light.
- [ ] Given I'm on the Gyms tab, when I tap the gear icon, then the Settings screen opens with a **Light / Dark** switch showing the current theme.
- [ ] Given the Settings screen, when I switch from Dark to Light (or back), then the new theme applies to the whole app **immediately**, with no restart.
- [ ] Given I chose a theme, when I close and reopen the app (or restart the phone), then my chosen theme is still used.
- [ ] Given I chose a theme, when the phone's system theme changes, then the app's theme does **not** change.

### Gyms (Epic 2)

**2.1** As a lifter, I want to create, edit and delete gyms, so that I can tag workouts with where I trained.

Acceptance criteria
- [ ] Given I'm on the Gyms tab with no gyms, then I see an empty state ("No gyms yet – add your first gym").
- [ ] Given I'm on the Gyms tab, when I add a gym with a name, then it appears in the list.
- [ ] Given an existing gym, when I edit its name, then the new name is shown in the gym list, the gym picker, the active workout header, history rows and the history gym filter, workout summaries and the exercise detail gym selector.
- [ ] Given several gyms, then wherever gyms are listed (gym list, gym picker, history gym filter, exercise detail gym selector), they are sorted alphabetically by name, ignoring case. "The first gym" always means the first gym in this order.
- [ ] Given I try to save a gym with an empty name, then saving is blocked.
- [ ] Given a gym with no workouts, when I delete it and confirm, then it is removed for good.
- [ ] Given a gym with workouts, when I delete it and confirm, then it is **archived**. It is hidden from the gym list and the gym picker, but its workouts and PRs are kept, and it still appears in the history gym filter and on the exercise detail gym selector.
- [ ] Given I cancel the delete confirmation, then nothing changes.

**2.2** As a lifter, I want to pick the gym for my workout manually, so that my workout is always tagged with the right gym.

Acceptance criteria
- [ ] Given no gyms exist, when I tap Start workout, then I'm asked to create a gym first and taken to the gym form. No gym is ever created automatically.
- [ ] Given gyms exist and GPS auto-detect is off, when I start a workout, then the last-used gym is pre-selected (or the first gym in the gym list, by name ignoring case, if there is no last-used gym or it no longer exists or is archived).
- [ ] Given an active workout, when I tap the gym in the header, then I can pick a different non-archived gym from a bottom sheet.
- [ ] Given I start a workout (empty or repeated, Story 4.6) or change the gym of the active workout, then that gym becomes the last-used gym.
- [ ] Given I change the gym of a past workout in edit mode (Story 6.3), then the last-used gym does **not** change.
- [ ] Given any saved workout, then its header and summary show exactly one gym, and a workout can't be started without a gym (see the first criterion).

**2.3** As a lifter, I want to set a gym's location from my current position, so that auto-detection can recognize it.

Acceptance criteria
- [ ] Given location permission is granted, when I tap "Use current location" in the gym form, then the gym's coordinates are saved and shown.
- [ ] Given permission isn't granted yet, when I tap it, then the system permission dialog is shown, with an explanation if I denied it before.
- [ ] Given I granted only approximate location, when I tap it, then the approximate position is used.
- [ ] Given permission is permanently denied, when I tap it, then I see a message with a link to app settings, and I can still save the gym without a location.
- [ ] Given a gym has a location, when I tap "Remove location", then the coordinates are cleared.
- [ ] Location is optional. A gym without a location is fully usable, except for auto-detection.
- [ ] Only foreground location is requested, never background location.

**2.4** As a lifter, I want to set a gym's location by searching an address, so that I can add a gym without being there.

Acceptance criteria
- [ ] Given I'm online, when I search e.g. "Aupark Bratislava", then I see up to 5 matching addresses, and selecting one saves its coordinates.
- [ ] Given I'm offline, nothing matches, or the device has no address search service, when I search, then I see a distinct message for each case ("offline", "no results", "address search unavailable"), each with a **Retry** button. The app never crashes.

**2.5** As a lifter, I want the app to detect my gym automatically when I start a workout, so that I don't have to select it every time.

Acceptance criteria
- [ ] Given the Settings screen from Story 1.5, then it also has an "Auto-detect gym (GPS)" switch, which is **off** after a fresh install.
- [ ] Given I turn auto-detect on, then location permission is requested. If I deny it, the switch turns back off.
- [ ] Given auto-detect is off, when I start a workout, then the last-used gym is selected and no location is requested.
- [ ] Given auto-detect is on and I'm within **150 m** of a non-archived gym that has a location, when I start a workout, then the nearest such gym is pre-selected.
- [ ] Given auto-detect is on but no gym is in range, the location request fails or does not return a location within 5 s, or permission is denied, when I start a workout, then the last-used gym is selected.
- [ ] Given a gym was auto-selected, when I open the gym picker, then I can still change it.
- [ ] The detection radius is fixed at 150 m and can't be edited. Location is checked once per workout start, never in the background.

### Exercise library (Epic 3)

**3.1** As a lifter, I want common exercises available on first launch, so that I can start logging immediately.

Acceptance criteria
- [ ] Given a fresh install, when I open the Exercises tab, then every exercise in the seed list (about 40 common exercises; the exact list is Open question 1) is listed, with no duplicates. They cover chest, back, shoulders, arms, legs and core across barbell, dumbbell, machine, cable and bodyweight.
- [ ] Each seeded exercise has a name, a primary muscle group, an equipment type (barbell / dumbbell / machine / cable / bodyweight) and a default unilateral flag.
- [ ] Given the app is reopened, then the seed data is not inserted again (no duplicates).

**3.2** As a lifter, I want to browse and search exercises, so that I can find the one I need quickly.

Acceptance criteria
- [ ] Given the Exercises tab, then exercises are listed alphabetically.
- [ ] Given the Exercises tab, when I type in the search field, then the list filters by name (ignoring case) as I type.
- [ ] Given the filter chips, when I select a muscle group and/or an equipment type, then only matching exercises are shown, combined with the search text.
- [ ] Given I'm adding exercises to a workout, when I select several in the picker and confirm with "Add (n)", then all of them are returned in the order I selected them.
- [ ] Hidden exercises don't appear in the picker.

**3.3** As a lifter, I want to create unlimited custom exercises, so that I can log any variation I do.

Acceptance criteria
- [ ] Given I'm on the Exercises tab, when I create an exercise with a name, muscle group, equipment and unilateral default, then it appears in the list and the picker.
- [ ] Given an empty name, when I save, then saving is blocked.
- [ ] Given a name that already exists (ignoring case, e.g. "bench press" vs "Bench Press"), when I save, then I see an error.
- [ ] Given 200 custom exercises exist, when I create another one, then it is saved and listed. There is no limit on the number of custom exercises.

**3.4** As a lifter, I want to edit or hide exercises, including seeded ones, so that the library fits my training.

Acceptance criteria
- [ ] Given any exercise, seeded or custom, when I edit it, then the changes are saved. The same validation as Story 3.3 applies.
- [ ] Given an exercise, when I hide it, then it disappears from the list and the picker, but its history and PRs remain.
- [ ] Given "Show hidden" is on, when I view the list, then hidden exercises are visible and can be unhidden.
- [ ] Given a custom exercise that has never been added to any workout, when I delete it and confirm, then it is removed.
- [ ] Given a custom exercise that appears in any workout, finished or active, even with only empty or unchecked sets, when I delete it and confirm, then it is hidden instead of deleted.
- [ ] Given a seeded exercise, then it can only be hidden, never deleted.

### Workout logging (Epic 4)

**4.1** As a lifter, I want to start an empty workout, so that I can log a session.

Acceptance criteria
- [ ] Given no active workout, when I open the Workout tab, then I see **Start workout** and a "Repeat a workout" section (Story 4.6).
- [ ] Given no active workout, when I tap Start workout, then a workout is created with the current time and the gym selected as in Stories 2.2 / 2.5, and the active workout is shown in the Workout tab.
- [ ] Given an active workout exists, when I open the Workout tab, then the active workout is shown instead of Start. The bottom navigation bar stays visible.
- [ ] The header shows the gym, the elapsed time (counting up from the start time) and the workout name. That is "Workout in progress", unless I gave it a name myself.
- [ ] Given an active workout, then Start workout is not offered and repeating is blocked (Story 4.6), so a second active workout can't be created.

**4.2** As a lifter, I want to add, remove and reorder exercises in my workout, so that it matches what I actually do.

Acceptance criteria
- [ ] Given an active workout, when I tap Add exercise and pick exercises, then they are added at the end of the workout in the order I selected them, each with one empty set.
- [ ] Given an exercise is already in the workout, when I add it again, then it is added as another separate entry with its own sets. The same exercise may appear in one workout more than once.
- [ ] Given an exercise in the workout, when I remove it (confirming first if it has sets), then it and its sets are removed.
- [ ] Given several exercises, when I tap move up / move down, then the order changes and is saved. The first exercise has no "up" button and the last has no "down" button.

**4.3** As a lifter, I want to log sets with weight, reps, set type and a unilateral tag, so that my training is recorded accurately.

Acceptance criteria
- [ ] Given an exercise in the workout, when I tap Add set, then a new set row appears.
- [ ] Weight is entered in kg, from **0 to 999.99**, with **at most 2 decimal places** (e.g. 22.5, 1.25). For bodyweight exercises, weight means *added* weight. Reps accept whole numbers from **1 to 999**.
- [ ] Given I enter a negative weight, a weight above 999.99, a weight with more than 2 decimal places (e.g. 22.555) or text, then it is not saved and the field shows an error.
- [ ] Given I enter reps of 0, reps above 999, a decimal number (e.g. 2.5) or text, then it is not saved and the field shows an error.
- [ ] Each set has a type: warm-up, working or backoff. The default is working.
- [ ] Each set has a unilateral tag. It starts from the exercise's default and can be changed per set. Unilateral sets show "per side", and their reps mean reps per side.
- [ ] Given weight and reps are filled, when I tick the done checkbox, then the set is marked done: the checkbox shows as checked, the row is styled differently from unchecked rows (e.g. a tinted background), and the set is saved immediately together with the time it was checked (used by Story 4.10).
- [ ] Given I uncheck a set, then its check time is cleared.
- [ ] Given weight or reps is empty, then the set can't be marked done.
- [ ] Given a checked set, when I clear its weight or reps, then the set is unchecked automatically and its check time is cleared.
- [ ] Given a set, when I delete it, then it is removed.

**4.4** As a lifter, I want to optionally log RPE and RIR, so that I can track effort when I care to.

Acceptance criteria
- [ ] Given a set, when I open RPE, then I can pick a value from 6 to 10 in steps of 0.5, or leave it empty.
- [ ] Given a set, when I open RIR, then I can pick 0, 1, 2, 3, 4, 5+ or **F** (failure), or leave it empty. F and 0 are stored as different values.
- [ ] RPE and RIR don't depend on each other. A set can be marked done with neither.

**4.5** As a lifter, I want to see what I did last time on this exercise at this gym, so that I know what to aim for.

Acceptance criteria
- [ ] Given I've done this exercise before at the current gym, when it's in my workout, then each set row shows a "Previous" value, e.g. `80 × 8 @8`. It comes from the set with the same number in the most recent finished workout that had this exercise at this gym.
- [ ] Given that previous workout had fewer sets, then the extra rows show "–".
- [ ] Given that previous workout had this exercise more than once (Story 4.2), then its done sets are numbered across all its entries in workout order: the first entry's sets, then the second entry's, and so on.
- [ ] Given I tap a Previous value, then its weight and reps are copied into that set.
- [ ] Given there's no previous workout with this exercise at this gym, then the column shows "–". Workouts at other gyms are not used.

**4.6** As a lifter, I want to repeat a past workout, so that I don't rebuild the same session every week.

Acceptance criteria
- [ ] Given finished workouts exist and none is active, then the Workout tab shows a "Repeat a workout" list of the **5 most recent** finished workouts (name, gym, date).
- [ ] Given I pick a workout from that list, or tap **Repeat** on a workout summary (Story 5.3), then a new active workout is created. It has the same exercises in the same order, the same number of sets, and the same weights, reps, set types and unilateral tags.
- [ ] RPE, RIR and the done state are not copied. All sets start unchecked.
- [ ] The gym is not copied. It is chosen the same way as on a normal start (last-used or auto-detected; asked to create a gym if none exist).
- [ ] Given a workout is already active, when I try to repeat, then I'm told to finish or discard it first, and nothing is created.

**4.7** As a lifter, I want to finish my workout and have it named automatically, so that history stays tidy without effort.

Acceptance criteria
- [ ] Given an active workout with all sets checked, when I tap Finish, then the end time is saved, the workout is finished, and its summary opens (Story 5.3).
- [ ] Given unchecked sets exist, when I tap Finish, then I'm asked "Remove N unchecked sets and finish?". **Remove & finish** deletes those sets, also removes every exercise that is left with no sets, and finishes the workout. **Cancel** leaves the workout active and unchanged.
- [ ] Given a finished workout without a manual name, then its name comes from its **end time** in local time: 05:00–11:59 *Morning workout*, 12:00–16:59 *Afternoon workout*, 17:00–21:59 *Evening workout*, 22:00–04:59 *Night workout*. This holds at the boundaries (e.g. 04:59 → Night, 05:00 → Morning, 11:59 → Morning, 12:00 → Afternoon, 16:59 → Afternoon, 17:00 → Evening, 21:59 → Evening, 22:00 → Night, 00:00 → Night).
- [ ] The automatic name is calculated whenever it is shown and is never saved. Only a manual name is saved.
- [ ] Given a workout, active or finished, when I tap its name and enter a new one, then that name is saved and shown instead of the automatic name. It is never overwritten automatically, even if the end time changes.
- [ ] Given a workout with a manual name, when I rename it to an empty name, then it goes back to the automatic name (or "Workout in progress" while active).

**4.8** As a lifter, I want to discard a workout, so that accidental or abandoned sessions don't pollute my history.

Acceptance criteria
- [ ] Given an active workout, when I tap Discard and confirm, then the workout and all its exercises and sets are deleted, and the Workout tab shows Start workout again.
- [ ] Given I cancel the dialog, then nothing changes.

**4.9** As a lifter, I want my workout in progress to be saved continuously, so that an app kill or phone restart never costs me data.

Acceptance criteria
- [ ] Given I'm logging a workout, when the app is killed (e.g. `adb shell am kill app.liftlog` or "Don't keep activities") or the phone restarts, then on reopening the active workout is restored exactly: exercises, order, sets, values and done states.
- [ ] Given an active workout exists, when I launch the app, then the active workout is shown.
- [ ] There is no Save button. Every change is saved immediately.
- [ ] Given I typed a value into a set and waited 1 s, when the app is killed, then on reopening the value is restored.

**4.10** As a lifter, I want the app to notice when I forgot to finish a workout, so that it doesn't end up hours long with a wrong end time and name.

Acceptance criteria
- [ ] Every edit to the active workout, including checking or unchecking a set, counts as activity and records the time of the last activity.
- [ ] Given an active workout with **more than 3 hours** since its last activity, when I open the app or come back to it, then a dialog appears: "Looks like you forgot to finish. Finish at HH:MM (last checked set)?". HH:MM is the latest check time among the sets that are **currently** checked (a set that was checked and later unchecked doesn't count).
- [ ] Given no set in the workout is currently checked, then the suggested time is the time of the last activity.
- [ ] Given the suggested time equals the workout's start time (a workout that was never edited), then **Finish at HH:MM** is not offered. Only **Pick time…** and **Keep going** are offered.
- [ ] Given the workout has had 3 hours or less of inactivity, when I open the app or come back to it, then no dialog is shown. Boundary: last activity exactly 3 h 00 min ago → no dialog; 3 h 01 min ago → dialog.
- [ ] Given the dialog, when I tap **Finish at HH:MM**, then the normal finish flow (Story 4.7) runs with that suggested end time.
- [ ] Given the dialog, when I tap **Pick time…**, then a time picker opens. It selects a time of day only. The end time is the **first occurrence** of that time of day strictly after the workout's start time. Example: start 23:10, picked 00:30 → 00:30 on the next day.
- [ ] Given I pick a time whose end time is not after the workout's start time, or is **later than now**, then an error message is shown, the picker stays open and the workout stays active. Given a valid time (after the start and not later than now), then the normal finish flow runs with that end time.
- [ ] Given the dialog, when I tap **Keep going**, then the workout stays active and its exercises and sets are unchanged. Keep going counts as activity: the time of the last activity becomes now, so the dialog isn't shown again until another 3 hours pass with no activity. If no set is checked when the dialog appears again, then the suggested time is the moment I tapped Keep going.
- [ ] Given I finish through this dialog and unchecked sets exist, then the "Remove N unchecked sets and finish?" dialog from Story 4.7 is shown as usual.
- [ ] Given I dismiss the forgotten-workout dialog (Back or a tap outside), or press **Cancel** in the unchecked-sets dialog that follows it, then nothing is saved, the workout stays active, and the forgotten-workout dialog appears again the next time the app comes to the foreground.
- [ ] Given I finish through this dialog, then the automatic name (if there is no manual name) and the duration come from the chosen end time, not from the moment I tapped. Example: last set checked at 19:40, dialog answered the next morning → *Evening workout*.

### Personal records (Epic 5)

**5.1** As a lifter, I want my PRs calculated from my history, so that they're always correct even after I edit old workouts.

Acceptance criteria
- [ ] PRs are tracked separately for each combination of exercise, gym, and unilateral vs. bilateral.
- [ ] Four PR types are calculated, each with its own badge label:
  - **heaviest weight**, badge "Weight";
  - **best estimated 1RM** (e1RM), using the Epley formula `weight × (1 + reps / 30)`, where a 1-rep set's e1RM is its weight, badge "e1RM";
  - **rep PRs**, the best weight for each rep count, badge "*n*RM" (e.g. "5RM"). Rep PRs cover **any** rep count, with no upper limit (e.g. 1, 3, 5, 8, 12, 20, 30);
  - **most reps at a weight**, the highest rep count for each exact weight, badge "PR".
- [ ] A set earns an "*n*RM" badge only if an earlier qualifying set with exactly *n* reps exists **and** the set's weight is strictly greater than the best weight at *n* reps. The first set ever at a rep count earns **no** "*n*RM" badge, but it becomes the best for that rep count and appears in the rep-PR table (Story 5.4).
- [ ] A set earns a "PR" badge only if an earlier qualifying set with exactly the same weight exists **and** the set's reps are strictly greater than the most reps ever done at that weight. The first set ever at a weight earns **no** "PR" badge.
- [ ] The four PR types are checked independently, so one set can earn several badges.
- [ ] Only sets that are done **and** of type **working** or **backoff** count. Warm-ups and unchecked sets are ignored.
- [ ] Sets are compared in chronological order: workout start time, then exercise position, then set position. The active workout's sets come last.
- [ ] A set earns a PR type only if it is **strictly better** than the best of the earlier qualifying sets for that type. A tie is not a PR.
- [ ] The first workout with a qualifying set for an exercise at a gym (counted separately for unilateral and bilateral) is a **baseline**. Its sets earn no PR badges, but they do count as the bests that later sets must beat. A workout that had the exercise with only warm-up or unchecked sets is not the baseline.
- [ ] Example: given a baseline workout with 100 kg × 5 (working, done), when a later workout at the same gym has 100 kg × 5 checked, then it gets no badge (tie); when 102.5 kg × 5 is checked, then it gets the badges Weight, e1RM and 5RM, but no "PR" because 102.5 kg was never lifted before; a warm-up 110 kg × 1 gets no badge.
- [ ] Example: given the most reps done so far at 40 kg is 7 (40 kg × 7), when a later workout at the same gym has 40 kg × 8 checked, then it gets the badge "PR". It gets no "8RM" badge if no 8-rep set was done before. "Weight" and "e1RM" are added only if 40 kg × 8 also beats those bests. A further 40 kg × 8 gets no "PR" (tie).
- [ ] PRs are calculated from history and never stored. Given I edit or delete an old workout, then all PRs reflect the change without any manual recalculation.

**5.2** As a lifter, I want to see a PR badge on a set as soon as I check it, so that I get instant feedback.

Acceptance criteria
- [ ] Given I check a set that beats a PR, then a trophy badge appears on that set right away, naming the PR type(s) with the labels "Weight", "e1RM", "*n*RM" (e.g. "5RM", "20RM") and "PR".
- [ ] Given I uncheck the set, or edit it so it no longer beats the PR, then the badge disappears right away. Given I edit it so it now beats a PR, then the badge appears.
- [ ] The check uses finished workouts plus earlier sets of the active workout.
- [ ] Warm-up sets and sets in a baseline workout never get a badge.

**5.3** As a lifter, I want to see a summary after finishing a workout, so that I can review what I did and which PRs I hit.

Acceptance criteria
- [ ] Given I finish a workout, then the summary screen opens. It shows the name, gym, date, duration and every exercise with its sets (weight × reps, RPE/RIR, set type, "per side" for unilateral sets).
- [ ] Sets that hit PRs show their PR badges, and the summary header shows the total number of PRs in the workout. The total counts **sets** with at least one PR badge, so a set with Weight and 5RM counts as 1.
- [ ] Given a past workout, when I open it from history, then I see the same summary. PRs are shown as they were at that time, compared only with earlier workouts.
- [ ] The summary offers **Repeat** (Story 4.6), **Edit** (Story 6.3) and **Delete** (Story 6.4).

**5.4** As a lifter, I want to see an exercise's PRs and history per gym, so that I can track progress where I train.

Acceptance criteria
- [ ] Given I tap an exercise in the library, then the detail screen opens with the last-used gym selected, if that gym still exists (archived gyms included).
- [ ] Given there is no last-used gym, or it was deleted, then the first gym is selected, taken from all gyms (archived included) sorted by name, ignoring case.
- [ ] Given no gyms exist, then no gym is selected, the PR values show "–" and the history is empty.
- [ ] I can switch the gym, including archived gyms, and toggle between bilateral and unilateral.
- [ ] I see the heaviest weight and the best e1RM, each with the date it was achieved, or "–" if there is none.
- [ ] I see a rep-PR table with a row for **every rep count I have ever logged** in a qualifying set for this exercise, gym and side mode (e.g. 1, 3, 5, 8, 12, 20, 30). Each row shows the best weight and its date.
- [ ] Below that, I see the history of done sets (all set types, including warm-ups) for this exercise at this gym, filtered by the selected side mode, grouped by workout (date + name), newest first.
- [ ] Given no qualifying sets exist for the selected gym and side mode, then the PR values show "–" and the rep-PR table is empty.
- [ ] Given no done sets exist for the selected gym and side mode, then the history is empty.

### History (Epic 6)

**6.1** As a lifter, I want to see a list of my past workouts, so that I can look back on my training.

Acceptance criteria
- [ ] The History tab lists finished workouts, newest first, with name, gym, date, duration and number of exercises. The active workout is not listed.
- [ ] Given no finished workouts, then an empty state is shown.
- [ ] Tapping a workout opens its summary (Story 5.3).

**6.2** As a lifter, I want to filter my history by gym, so that I can focus on one location.

Acceptance criteria
- [ ] Given the History tab, when I select a gym in the filter, then only workouts at that gym are shown.
- [ ] The filter lists all gyms, including archived ones.
- [ ] Selecting "All gyms" clears the filter.

**6.3** As a lifter, I want to edit a past workout, so that I can fix mistakes after the fact.

Acceptance criteria
- [ ] Given a past workout, when I tap Edit, then it opens in the workout screen in edit mode, with no elapsed timer and no Finish or Discard. I can change its name, gym, exercises, sets (all fields) and start/end date and time.
- [ ] Changes are saved immediately, as in Story 4.9.
- [ ] Given I set an end time that is not after the start time, then an error message is shown and the saved start and end times stay unchanged.
- [ ] Given I change the workout's gym, then the last-used gym (Story 2.2) stays unchanged.
- [ ] Given the workout has no manual name and I change the end time, then the displayed automatic name updates to match the new end time. Given it has a manual name, then the name stays the same.
- [ ] PRs (Story 5.1) and "Previous" hints (Story 4.5) reflect the edits automatically.

**6.4** As a lifter, I want to delete a past workout, so that wrong or test entries don't distort my PRs.

Acceptance criteria
- [ ] Given a past workout, when I tap Delete (on the summary or in edit mode) and confirm, then the workout and all its exercises and sets are deleted, PRs recalculate, and I'm taken back to History.
- [ ] Given I cancel the dialog, then nothing changes.

---

## 4. Constraints

### Platform and technology

| Area | Constraint |
|---|---|
| Platform | Android only, **phones only, portrait only**. |
| SDK levels | compileSdk / targetSdk = newest stable Android API level; **minSdk 26**. |
| Package / applicationId | `app.liftlog` |
| Language & UI | Kotlin, Jetpack Compose, Material 3. |
| Architecture | MVVM with Coroutines / Flow and a repository layer (details in [DESIGN.md](DESIGN.md)). |
| Persistence | Room (SQLite) for all domain data; Jetpack DataStore (Preferences) for settings. |
| Dependency injection | Hilt. |
| Navigation | Navigation Compose with type-safe `@Serializable` routes; bottom bar with 4 tabs: **Workout, History, Exercises, Gyms**. Settings is opened from a gear icon on the Gyms tab. |
| Location | Fused Location Provider for the current position; Android `Geocoder` for address search. **Foreground location permission only, never background.** |
| Build | Gradle Kotlin DSL with a version catalog (`libs.versions.toml`). |
| CI | GitHub Actions running `./gradlew build test` on every push and pull request. |

### Product constraints

- **Fully offline.** No backend, no accounts, no network calls. The only exception is address search (Story 2.4), which uses `Geocoder` and needs internet.
- **UI language is English.** Weight is **kg only**.
- **Theme** is set by hand (Light / Dark) and is **Dark by default**. It never follows the system theme (Story 1.5).
- **GPS auto-detect is off by default.** The gym radius is **fixed at 150 m**.
- **Single user per device**, with one active workout at a time.
- **Accepted risk – data loss:** there is no export/import and no backup feature. Uninstalling the app or resetting the phone may lose all data. This risk is accepted for the MVP. Manual export is a v2 candidate (see [Out of scope](#5-out-of-scope)).

### Schedule

| Milestone | Due | Content |
|---|---|---|
| M1 – Foundation & data | 2026-11-01 | Epic 1 (incl. 1.5), Story 2.1, Epic 3 |
| M2 – Workout logging | 2026-11-15 | Story 2.2, Epic 4 (incl. 4.10). The 4-week Hevy replacement run starts on 2026-11-16. |
| M3 – PRs, history & GPS | 2026-12-06 | Stories 2.3–2.5, Epic 5, Epic 6. The 4-week run ends on 2026-12-13. |
| Deadline | 2026-12-18 | End of semester; time left over for fixes |

### Engineering constraints (Epic 1 developer stories)

These stories are for the developer, not the user. They are not features, but each one must be met.

- **Story 1.1 – Runnable project skeleton.** Given a fresh clone, `./gradlew assembleDebug` succeeds. The installed app shows the bottom bar with Workout, History, Exercises and Gyms and can switch between them. The repo has an Android `.gitignore` and a README with a description, a link to project.md and build instructions. Hilt is set up. *(The skeleton's "follow system dark mode" criterion is replaced by Story 1.5.)*
- **Story 1.2 – Room database schema.** On first start, the database has tables for gyms, exercises, workouts, workout exercises and sets. Schema export is on (schema version 1), and the schema JSON in `app/schemas/` is committed. DAO tests on an in-memory database cover insert, query, update and delete for every entity. Gyms and exercises that are still in use are never deleted for good; they are archived or hidden instead. Only the manual workout name is stored. The displayed name is calculated (Story 4.7). The time of the last activity on the active workout and the check time of each checked set are stored (Story 4.10). The gym radius is a constant, not a stored value.
- **Story 1.3 – Persistent app settings.** Settings live in Preferences DataStore and survive restarts. Fresh-install defaults: theme **Dark**, GPS auto-detect **off**, no last-used gym.
- **Story 1.4 – CI on every push.** A push or pull request to `main` builds the app and runs the unit tests. A failing test turns the check red.
- **Testing.** Unit tests cover the pure logic: the Epley formula, PR rules (ties, warm-ups, baseline, unilateral/bilateral split, different gyms, any rep count, no badge for a new rep count or a new weight, most reps at a weight), the summary PR count, weight and reps input limits, automatic naming at the time boundaries, nearest-gym matching and forgotten-workout detection. Room DAOs are tested on an in-memory database. Restoring a workout after the app is killed (Story 4.9) is tested by hand.

## 5. Out of scope

| Not in the MVP | Reason |
|---|---|
| Statistics and charts (volume, progress graphs, muscle heatmaps) | A large feature on its own. PR tracking already answers the main "am I progressing?" question. |
| Rest timer | A reliable timer needs a foreground service, notifications and the notification permission. That is a lot of work for something any phone timer can do. |
| Routines / templates (named plans, target sets, folders, scheduling) | "Repeat a workout" (Story 4.6) covers most of the need without a new data model. |
| Map picker for gym location | Needs a maps SDK and an API key. Current location and address search are enough. |
| Background geofencing | Needs background location permission and costs battery. One check at workout start is enough. |
| Accounts, backend, cloud sync | Staying offline keeps the app simple, private and free to run. Sync could be added later on top of the local database. |
| Data export / import and backup | Kept out to hold MVP scope down. The data-loss risk is accepted (see [Constraints](#4-constraints)). **Manual export is a v2 candidate.** |
| kg / lb switching | Target users train in kg. Unit conversion would touch every screen and every PR calculation. |
| Drop sets, failure sets and other advanced set types | Warm-up / working / backoff plus RIR = F covers the real need. More set types would complicate the PR rules. |
| Exercise images, videos, instructions | This is content work, not engineering, and target users already know the exercises. |
| iOS | One native Android codebase keeps the focus on a single platform. |
| Tablets and landscape orientation | The app is used one-handed on a phone in the gym. Supporting other layouts adds work and testing with little benefit. |
| Social features (sharing, feed, followers) | Needs a backend and accounts, and isn't part of the core problem. |
| Following the system theme | A manual Light/Dark choice (default Dark) is simpler and predictable. Following the system would add a third mode and more state to test. |
| Import from Hevy | Needs parsing of an outside export format and mapping its exercises. Not needed to start logging from now on. |
| Editable gym radius | A fixed 150 m works for typical gyms and keeps the form and the matching logic simple. |

## 6. Open questions

This question is not decided yet. Acceptance criteria that depend on it point to it by number. All other questions raised while writing this spec and [DESIGN.md](DESIGN.md) have been decided and are written into the acceptance criteria in §3.

| # | Question | Affects |
|---|---|---|
| 1 | Which exact ~40 exercises are seeded, with their muscle group, equipment and default unilateral flag? | Story 3.1 (task 3.1.1), DESIGN.md `SeedExercises` |
