# Project-specific wrappers

Read this file when the review depends on understanding the exact implementation details of the project's local rich-text or overlay wrappers.

These classes are project-local helpers, not Flutter SDK components.

## `TwOverlayContextView`

Purpose:
- Build an `Overlay`
- Create one internal `OverlayEntry`
- Expose the overlay-mounted `BuildContext` through a callback
- Remove and dispose the internal `OverlayEntry` in `dispose()`

Implementation:

```dart
import 'package:flutter/material.dart';

typedef TwOverlayContextHandler = Function(BuildContext context);

class TwOverlayContextView extends StatefulWidget {
  final TwOverlayContextHandler contextInitHandler;
  const TwOverlayContextView({
    super.key,
    required this.contextInitHandler,
  });

  @override
  State<TwOverlayContextView> createState() => _TwOverlayContextViewState();
}

class _TwOverlayContextViewState extends State<TwOverlayContextView> {
  OverlayEntry? _overlayEntry;

  @override
  void dispose() {
    _overlayEntry?.remove();
    _overlayEntry?.dispose();
    _overlayEntry = null;
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return _buildPageOverlay(widget.contextInitHandler);
  }

  Widget _buildPageOverlay(
    TwOverlayContextHandler contextInitHandler,
  ) {
    return Overlay(
      initialEntries: [
        _overlayEntry = _overlayEntry ??
            OverlayEntry(
              builder: (context) {
                contextInitHandler(context);
                return const SizedBox.shrink();
              },
            )
      ],
    );
  }
}
```

Review implications:
- Prefer this wrapper when code only needs an overlay-mounted `BuildContext`
- If code bypasses this wrapper and manages raw `OverlayEntry` manually, check for symmetric `remove()` and `dispose()`

## `TwAutoDisposeRichText`, `TwAutoTextSpan`, and `TwAutoWidgetSpan`

Purpose:
- Replace raw `TextSpan` + manual recognizer wiring with a local wrapper that owns recognizer lifecycle
- Build `Text.rich` from a project-defined span model
- Dispose internally created `TapGestureRecognizer` instances in both `didUpdateWidget` and `dispose()`

Implementation:

```dart
import 'package:flutter/gestures.dart';
import 'package:flutter/material.dart';

abstract class AutoRichSpan {}

class TwAutoTextSpan extends AutoRichSpan {
  final String text;
  final TextStyle? style;
  final VoidCallback? onTap;
  final GestureTapDownCallback? onTapDown;
  final GestureTapUpCallback? onTapUp;
  final VoidCallback? onTapCancel;

  TwAutoTextSpan({
    required this.text,
    this.style,
    this.onTap,
    this.onTapDown,
    this.onTapUp,
    this.onTapCancel,
  });
}

class TwAutoWidgetSpan extends AutoRichSpan {
  final Widget child;
  final PlaceholderAlignment alignment;
  final TextStyle? style;

  TwAutoWidgetSpan({
    required this.child,
    this.alignment = PlaceholderAlignment.bottom,
    this.style,
  });
}

class TwAutoDisposeRichText extends StatefulWidget {
  final List<AutoRichSpan> spans;
  final TextAlign textAlign;
  final TextStyle? commonStyle;
  final StrutStyle? commonStrutStyle;
  final TextOverflow overflow;
  final int? maxLines;

  const TwAutoDisposeRichText({
    Key? key,
    required this.spans,
    this.textAlign = TextAlign.left,
    this.commonStyle,
    this.commonStrutStyle,
    this.overflow = TextOverflow.clip,
    this.maxLines,
  }) : super(key: key);

  @override
  State<TwAutoDisposeRichText> createState() => _TwAutoDisposeRichTextState();
}

class _TwAutoDisposeRichTextState extends State<TwAutoDisposeRichText> {
  final List<TapGestureRecognizer?> _recognizers = [];

  @override
  void initState() {
    super.initState();
    _initRecognizers();
  }

  void _initRecognizers() {
    _recognizers.clear();
    for (var span in widget.spans) {
      if (span is TwAutoTextSpan && span.onTap != null) {
        final recognizer = TapGestureRecognizer()
          ..onTap = span.onTap
          ..onTapDown = span.onTapDown
          ..onTapUp = span.onTapUp
          ..onTapCancel = span.onTapCancel;
        _recognizers.add(recognizer);
      } else {
        _recognizers.add(null);
      }
    }
  }

  @override
  void didUpdateWidget(covariant TwAutoDisposeRichText oldWidget) {
    super.didUpdateWidget(oldWidget);
    for (var recognizer in _recognizers) {
      recognizer?.dispose();
    }
    _initRecognizers();
  }

  @override
  void dispose() {
    for (var recognizer in _recognizers) {
      recognizer?.dispose();
    }
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Text.rich(
      TextSpan(
        children: List.generate(
          widget.spans.length,
          (i) {
            final span = widget.spans[i];
            if (span is TwAutoTextSpan) {
              return TextSpan(
                text: span.text,
                style: span.style,
                recognizer: _recognizers[i],
              );
            } else if (span is TwAutoWidgetSpan) {
              return WidgetSpan(
                child: span.child,
                alignment: span.alignment,
                style: span.style,
              );
            }
            return const TextSpan(text: '');
          },
        ),
      ),
      textAlign: widget.textAlign,
      overflow: widget.overflow,
      maxLines: widget.maxLines,
      strutStyle: widget.commonStrutStyle,
      style: widget.commonStyle,
    );
  }
}
```

Review implications:
- `TwAutoDisposeRichText` is specifically safer than hand-written `TextSpan(... recognizer: TapGestureRecognizer() ..onTap = ...)` in `build`
- If code already uses this wrapper correctly, do not flag recognizer disposal issues for the wrapper itself
- If code recreates equivalent recognizer wiring outside this wrapper, treat that as a review target
