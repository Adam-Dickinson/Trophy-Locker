# ADR-0020: Keep the overlay toast in full-screen games

- **Status:** Accepted
- **Date:** 2026-11-18

## Context

ADR-0019 switched an unlock to a Windows notification whenever `SHQueryUserNotificationState` reported `QUNS_RUNNING_D3D_FULL_SCREEN`, on the assumption that the overlay is hidden there and that Windows would show its own notification instead. That was built before it was checked with a real game.

The owner's log from 7 to 18 November 2026 (Steam unlocks in Valor Mortis, Phantom Blade Zero, RV There Yet?, Phasmophobia and Crimson Desert Enhanced) showed:

- Every unlock was detected within about 2 seconds, so detection is not the problem.
- The same game is reported as exclusive full-screen at some moments and not at others. In one Phantom Blade Zero session, the 20:08 UTC unlock went to the overlay and was seen; the 21:23 and 21:40 unlocks went to a Windows notification and were not seen.
- Windows accepted the 21:40 notification (`LastNotificationAddedTime` for `electron.app.Trophy Locker` matches it), but no pop-up appeared and nothing was left in Notification Center. Windows 11 holds back notification pop-ups while a full-screen game runs (the Do Not Disturb "when playing a game" rule), so the fallback is suppressed at exactly the moment it is used.
- The overlay was seen over the same game in the same session, so it is not hidden by these games. Most modern games run in a flip-model presentation that Windows still reports as exclusive, but which lets an always-on-top window show.

Pop-ups are the reason people install the app, so a route that hides them by default is the wrong default.

## Options considered

| Option | Result |
|---|---|
| Keep ADR-0019 as is | Toasts go missing in the games tested |
| Tell players to make Trophy Locker a priority app in Windows | Works only for players who find the setting, and gives the plain Windows notification, not the designed toast |
| Remove the Windows notification route and koffi | Simplest, but loses the route for a game that really does hide the overlay |
| Change only the default of `fullscreenFallback` | Does not reach existing users: the whole settings object is saved on any change, so most stored settings already hold `true` |
| **Overlay by default; Windows notification behind a new opt-in setting, `nativeInFullscreen`, default off** | Fixes every install, including existing ones, and keeps the route for the rare game that needs it |

## Decision

The overlay is always used unless the player turns on `nativeInFullscreen`. The old `fullscreenFallback` key is no longer read: no screen ever offered it, so a stored `true` was never a player's choice, and the zod schema drops it on the next save. Detection (koffi, `SHQueryUserNotificationState`, checked only when a toast is due) stays as ADR-0019 describes, and still drives the opt-in route.

## Consequences

**Positive:**

- Existing installs get the overlay in full-screen after updating, with no action from the player.
- The Windows notification route and the full-screen check stay tested and available.

**Negative:**

- A game that truly draws above every window hides the toast unless the player turns the setting on. The unlock is still in Activity.
- The setting has no Settings switch yet, so the opt-in is only reachable once that card is built.

## Revisit if

- A tested game hides the overlay: consider turning the route on per game, or checking whether the overlay was actually drawn.
- Windows changes how it reports or suppresses notifications for full-screen games.
