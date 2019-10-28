# Coding Styles

## name libraries and source files using lowercase\_with\_underscores.

```dart
//GOOD
library peg_parser.source_scanner;

import 'file_system.dart';
import 'slider_menu.dart';
```

```dart
// BAD
library pegparser.SourceScanner;

import 'file-system.dart';
import 'SliderMenu.dart';
```

## PREFER using lowerCamelCase for constant names.

```dart
//GOOD
const pi = 3.14;
const defaultTimeout = 1000;
final urlScheme = RegExp('^([a-z]+):');
```

```dart
// BAD
const PI = 3.14;
const DefaultTimeout = 1000;
final URL_SCHEME = RegExp('^([a-z]+):');
```

## Prescribed order for the imports

```dart
// GOOD

// DO sort sections alphabetically

// 'dart'
import 'dart:async';
import 'dart:html';

// 'package'
import 'package:bar/bar.dart';
import 'package:foo/foo.dart';

// 'others imports'
import ...

```

## Avoid using cast\(\)

```dart
//GOOD
List<int> singletonList(int value) {
 var list = <int>[];
 list.add(value);
 return list;
}

```

```dart
// BAD
List<int> singletonList(int value) {
 var list = []; // List<dynamic>.
 list.add(value);
 return list.cast<int>();
}

```

## DO use = to separate a named parameter from its default value.

```dart
//GOOD
void insert(Object item, {int at = 0}) { ... }
```

```dart
// BAD
void insert(Object item, {int at : 0}) { ... }
```

## PREFER relative paths when importing libraries

```dart
//GOOD
import 'src/utils.dart';
```

```dart
// BAD
import 'package:my_package/src/utils.dart';
```



