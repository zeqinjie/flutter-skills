# Project-specific conventions

Read this file only when it exists in the installed skill directory. These rules extend the generic checks in `SKILL.md` for a specific Flutter codebase.

## State and Logic ownership

- `State` should own only resources tied directly to the widget lifecycle, such as animation, scroll, and focus controllers.
- Business resources such as requests, timers, and subscriptions should be released centrally in `logic.onDispose()`.
- If a `state` file contains disposal responsibility for business resources, such as `cancel()` or `close()` calls, treat it as a violation.

## Async lifecycle guards

- After `await`, verify `isClosed` is checked before creating a `Timer` or continuing async work tied to a page Logic instance.
- Timer callbacks should also check `isClosed` and cancel themselves when the Logic is closed.

## Project wrappers

These are project-local classes, not Flutter SDK widgets. Do not treat them as official framework components.

If the review depends on exact implementation details, read [project-wrappers.md](project-wrappers.md).

- `TwAutoDisposeRichText` wraps `Text.rich`, creates `TapGestureRecognizer` instances for tappable spans, and disposes them in both `didUpdateWidget` and `dispose()`. Prefer it over manual recognizer wiring in `TextSpan`.
- `TwAutoTextSpan` is the project's span model with tap callbacks. Use it with `TwAutoDisposeRichText`, not raw `TextSpan` plus manual recognizers.
- `TwAutoWidgetSpan` is the inline-widget span model for the same rich-text wrapper.
- `TwOverlayContextView` owns an internal `OverlayEntry`, removes and disposes it in `dispose()`, and exposes overlay-mounted `BuildContext`. Prefer it over handwritten `Overlay(initialEntries: [...])` when the page only needs overlay context.

## Project-specific controllers and packages

- `EasyRefreshController` must be disposed in the matching cleanup callback.
- When using `top_snackbar_flutter`, verify that `_overlayEntry` is manually removed in `dispose()` if the page can be destroyed before the snackbar dismisses itself.
- Do not allow `auto_size_text`. Require `flutter_auto_size_text` instead.
