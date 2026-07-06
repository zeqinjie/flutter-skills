---
name: widget-split
description: Refactor Flutter widget-building methods into readable, behavior-preserving layers by replacing deeply nested official Flutter widget chains with stepwise `resultWidget` assignments and focused private `_buildXxx()` helpers. Use when a Dart `build()` or `_buildXxx()` method returns nested `Padding`, `Container`, `Align`, `DecoratedBox`, `GestureDetector`, `Visibility`, `Expanded`, `Flexible`, `Positioned`, `Row`, `Column`, `Stack`, `Builder`, `AnimatedBuilder`, `ValueListenableBuilder`, `FutureBuilder`, or `StreamBuilder` trees and Codex needs to split them without changing layout, callbacks, keys, semantics, or state flow.
---

# Widget Split

Refactor for readability, not redesign. Preserve the rendered tree, widget order, keys, callbacks, semantics, and state ownership.

## Public Documentation Guardrails

- Use official Flutter framework widgets and APIs in explanations and examples.
- Do not mention project-specific wrappers, third-party state managers, or internal event widgets unless the user's source code already contains them.
- If the target code contains custom widgets, keep them in the actual edit, but explain the surrounding transformation with Flutter framework terminology.

## Refactoring Workflow

1. Start at the leaf widget and trace outward to the returned subtree.
2. Identify layout-sensitive parents that must stay in legal positions, such as `Expanded`, `Flexible`, `Positioned`, `SliverPadding`, `PreferredSize`, and `Tab`.
3. Replace a direct nested `return` with stepwise wrapping:

```dart
Widget resultWidget = const Text('Details');
resultWidget = Padding(
  padding: const EdgeInsets.all(12),
  child: resultWidget,
);
return resultWidget;
```

4. For `Row`, `Column`, `Stack`, and `Wrap`, prebuild any complex slot as a local variable or `_buildXxx()` helper before placing it in `children: []`.
5. For builder closures such as `Builder`, `LayoutBuilder`, `StatefulBuilder`, `ValueListenableBuilder`, `AnimatedBuilder`, `FutureBuilder`, and `StreamBuilder`, either use the same `resultWidget` pattern inside the closure or extract the whole subtree into a helper.
6. After refactoring, verify that conditions, callbacks, keys, and controller usage are unchanged.

## Core Rules

- Wrap one parent layer per assignment. Do not keep a long `A(child: B(child: C(...)))` chain once refactoring starts.
- Build from the inside out. The first assigned value should be the leaf or completed inner subtree.
- Keep layout widgets under legal parents. `Expanded` and `Flexible` must stay direct children of a `Flex`; `Positioned` must stay under `Stack`; slivers must stay in sliver contexts.
- Preserve child order and conditional branches exactly.
- Extract helpers for meaningful subtrees, reused blocks, or long builder closures. Do not split every simple leaf into its own method.
- Simple `children` entries such as a plain `Text` or a single-wrapper `SizedBox` can stay inline. Lift only the slots that still contain multiple nested layers or are hard to read.

## Preferred Patterns

### Stepwise wrapping

```dart
Widget _buildBody() {
  Widget title = const Text('Featured');
  title = Expanded(child: title);

  Widget action = IconButton(
    onPressed: () {},
    icon: const Icon(Icons.more_horiz),
  );

  Widget resultWidget = Row(
    children: [
      title,
      action,
    ],
  );
  resultWidget = Container(
    padding: const EdgeInsets.all(12),
    decoration: BoxDecoration(
      color: Colors.white,
      borderRadius: BorderRadius.circular(12),
    ),
    child: resultWidget,
  );
  resultWidget = Padding(
    padding: const EdgeInsets.symmetric(horizontal: 16),
    child: resultWidget,
  );
  return resultWidget;
}
```

### Prepare complex Flex children first

```dart
Widget _buildHeader() {
  Widget leading = const Icon(Icons.star);
  leading = Padding(
    padding: const EdgeInsets.only(right: 8),
    child: leading,
  );

  Widget title = const Text(
    'Featured',
    maxLines: 1,
    overflow: TextOverflow.ellipsis,
  );
  title = Expanded(child: title);

  Widget resultWidget = Row(
    children: [
      leading,
      title,
    ],
  );
  return resultWidget;
}
```

### Extract closure-heavy sections

If a `ValueListenableBuilder`, `AnimatedBuilder`, or `LayoutBuilder` closure still contains several nested parents, refactor inside the closure or move the returned subtree into a private helper so the closure stays readable.

Read [references/examples.md](references/examples.md) when the transformation needs more before/after patterns for Flex children, typed returns such as `AppBar`, builder closures, or extraction decisions.

## Final Check

- The outermost widget type matches the original tree.
- Every callback, `Key`, `Semantics`, and condition is preserved.
- Every layout-sensitive widget still has the same legal parent.
- No refactored method returns a deep nested chain when it could return `resultWidget`.
- The diff improves readability without introducing extra state or new abstractions.
