# Flutter Widget Split Examples

Use these examples when the target refactor needs a concrete before/after pattern. Keep examples in official Flutter terms even if the user's project contains custom widgets.

## Simple nested chain

Before:

```dart
Widget _buildBody() {
  return Padding(
    padding: const EdgeInsets.all(16),
    child: Container(
      decoration: BoxDecoration(
        color: Colors.white,
        borderRadius: BorderRadius.circular(12),
      ),
      child: Row(
        children: const [
          Icon(Icons.info_outline),
          SizedBox(width: 8),
          Text('Details'),
        ],
      ),
    ),
  );
}
```

After:

```dart
Widget _buildBody() {
  Widget resultWidget = Row(
    children: const [
      Icon(Icons.info_outline),
      SizedBox(width: 8),
      Text('Details'),
    ],
  );
  resultWidget = Container(
    decoration: BoxDecoration(
      color: Colors.white,
      borderRadius: BorderRadius.circular(12),
    ),
    child: resultWidget,
  );
  resultWidget = Padding(
    padding: const EdgeInsets.all(16),
    child: resultWidget,
  );
  return resultWidget;
}
```

## Prepare complex `children` entries first

Before:

```dart
Widget _buildHeader() {
  return Row(
    children: [
      Padding(
        padding: const EdgeInsets.only(right: 8),
        child: const Icon(Icons.star),
      ),
      Expanded(
        child: Container(
          padding: const EdgeInsets.symmetric(vertical: 4),
          child: const Text(
            'Featured item',
            maxLines: 1,
            overflow: TextOverflow.ellipsis,
          ),
        ),
      ),
    ],
  );
}
```

After:

```dart
Widget _buildHeader() {
  Widget leading = const Icon(Icons.star);
  leading = Padding(
    padding: const EdgeInsets.only(right: 8),
    child: leading,
  );

  Widget title = const Text(
    'Featured item',
    maxLines: 1,
    overflow: TextOverflow.ellipsis,
  );
  title = Container(
    padding: const EdgeInsets.symmetric(vertical: 4),
    child: title,
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

## Refactor builder closures in place

Before:

```dart
Widget _buildCounter(ValueNotifier<int> counter) {
  return ValueListenableBuilder<int>(
    valueListenable: counter,
    builder: (context, value, child) {
      return Align(
        alignment: Alignment.centerRight,
        child: DecoratedBox(
          decoration: BoxDecoration(
            color: Colors.blue.shade50,
            borderRadius: BorderRadius.circular(10),
          ),
          child: Padding(
            padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 6),
            child: Text('$value'),
          ),
        ),
      );
    },
  );
}
```

After:

```dart
Widget _buildCounter(ValueNotifier<int> counter) {
  Widget resultWidget = ValueListenableBuilder<int>(
    valueListenable: counter,
    builder: (context, value, child) {
      Widget resultWidget = Text('$value');
      resultWidget = Padding(
        padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 6),
        child: resultWidget,
      );
      resultWidget = DecoratedBox(
        decoration: BoxDecoration(
          color: Colors.blue.shade50,
          borderRadius: BorderRadius.circular(10),
        ),
        child: resultWidget,
      );
      resultWidget = Align(
        alignment: Alignment.centerRight,
        child: resultWidget,
      );
      return resultWidget;
    },
  );
  return resultWidget;
}
```

## Typed returns stay typed

```dart
AppBar _buildAppBar() {
  AppBar resultWidget = AppBar(
    title: const Text('Details'),
    centerTitle: true,
  );
  return resultWidget;
}
```

## Extraction checklist

- Keep `Expanded`, `Flexible`, `Positioned`, slivers, and other placement-sensitive widgets in legal parent contexts.
- Preserve `Key`, callbacks, semantics, and conditional branches exactly.
- Extract a helper only when it names a meaningful subtree or shortens a long closure.
- Leave very simple leaf children inline; lift only the slots that are still nested or noisy.
