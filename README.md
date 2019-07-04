# Dart and Flutter tips & tricks

One of the things I like about Dart & Flutter is how I keep finding interesting ways to use it in various situations, and when I do - I share them on [Twitter](https://twitter.com/minikin). Here's a collection of all the tips & tricks that I've shared so far.
Each entry has a link to the original tweet if you want to respond with some feedback or question, which is always welcome!

## Table of contents

[#1 Reduce boilerplates for operator == and hashCode](https://github.com/minikin/FlutterDartTips#1-reduce-boilerplates-for-operator-==-and-hashCode)

### [#1 Reduce boilerplates for operator == and hashCode](https://twitter.com/minikin/status/1128939893760172033?s=20)

:fire: If you don't use a library like `built_value` in your project you probably need to write a lot of
boilerplates to override operator `==` and `hashCode` like so:

_BEFORE:_

```dart
/// item.dart

class Item {
  final String id;
  final String title;
  final int numberOfparts;
  final double weight;
  final bool composable;

  const Item({
    @required this.id,
    @required this.title,
    @required this.numberOfparts,
    @required this.weight,
    @required this.composable,
  });

  @override
  bool operator ==(Object other) =>
      identical(this, other) ||
      other is Item &&
          other.runtimeType == runtimeType &&
          other.id == id &&
          other.title == title &&
          other.numberOfparts == numberOfparts &&
          other.weight == weight &&
          other.composable == composable;

  @override
  int get hashCode =>
      id.hashCode ^
      title.hashCode ^
      numberOfparts.hashCode ^
      weight.hashCode ^
      composable.hashCode;
}

```

To reduce boilerplates we can define a free function `hashValues` and use a shorter syntax in
`operator ==` method.

```dart
/// hash_values.dart

/// Generates a hash code for multiple [objects].
int hashValues(Iterable objects) => _finish(
    objects.fold(0, (hash, element) => _combine(hash, element.hashCode)));

// Jenkins hash functions
int _combine(int hash, int value) {
  hash = 0x1fffffff & (hash + value);
  hash = 0x1fffffff & (hash + ((0x0007ffff & hash) << 10));
  return hash ^ (hash >> 6);
}

int _finish(int hash) {
  hash = 0x1fffffff & (hash + ((0x03ffffff & hash) << 3));
  hash = hash ^ (hash >> 11);
  return 0x1fffffff & (hash + ((0x00003fff & hash) << 15));
}

```

_AFTER:_

```dart
/// item.dart

class Item {
  final String id;
  final String title;
  final int numberOfparts;
  final double weight;
  final bool composable;

  const Item({
    @required this.id,
    @required this.title,
    @required this.numberOfparts,
    @required this.weight,
    @required this.composable,
  });

  @override
  bool operator ==(dynamic other) =>
      identical(other, this) ||
      other.id == id &&
          other.title == title &&
          other.numberOfparts == numberOfparts &&
          other.weight == weight &&
          other.composable == composable;

  @override
  int get hashCode =>
      hashValues([id, title, numberOfparts, weight, composable]);
}
```

_USAGE:_

```dart
void main() {
  final item1 = Item(
      id: '1',
      title: 'Item 1',
      numberOfparts: 5,
      weight: 10.45,
      composable: true);

  final item2 = Item(
      id: '2',
      title: 'Item 2',
      numberOfparts: 5,
      weight: 5.95,
      composable: true);

  item1 == item2 ? print('Equal') : print('Not Equal');
}
```

:boom: __[TRY IT IN DARTPAD](https://dartpad.dartlang.org/9a7bbcb6dc07a40546743a3b0958966f)__

-----------
This project has been inspired by [Swift tips & tricks](https://github.com/JohnSundell/SwiftTips)
