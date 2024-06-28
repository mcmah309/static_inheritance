# static_inheritance

A static inheritance macro.

Workaround for inheriting/overriding static methods since for a generic `T` that extends `X`, `T.method()` is not possible. But with this macro we can do `X.static<T>().method()`

## Code Generation Example

Input:
```dart
@StaticInheritance
abstract class X {
  @static
  int method() => 1; 
  @static 
  int method2() => 2;
}
@StaticInheritance
class Y implements X {
  @static
  int method() => 9999;
  @static
  int method3() => 3;
}
```
Output:
```dart
class XStaticInheritedMethods {
  const XStaticInheritedMethods();

  int method() => 1;
  int method2() => 2;
}

sealed class X {
  static const Map<Type, XStaticInheritedMethods> _staticInheritedMethods = {
    X: XStaticInheritedMethods(),
    Y: YStaticInheritedMethods()
  };

  static XStaticInheritedMethods static<T extends X>() => _staticInheritedMethods[T] as XStaticInheritedMethods;
}

final class Y implements X {
  static const Map<Type, YStaticInheritedMethods> _staticInheritedMethods = {
    Y: YStaticInheritedMethods()
  };

  static YStaticInheritedMethods static<T extends Y>() => _staticInheritedMethods[T] as YStaticInheritedMethods;
}

class YStaticInheritedMethods implements XStaticInheritedMethods {
  const YStaticInheritedMethods();
  
  @override
  int method() => 9999;
  @override
  int method2() => const XStaticInheritedMethods().method2();
  int method3() => 3;
}
```
Test
```dart
int testIt<T extends X>() {
  return X.static<T>().method();
}

void main() {
  print(testIt<X>());
  print(testIt<Y>());
}
```

### Implementation Discusssion

We cannot have abstract `@static` methods since you can pass `X` into e.g. `testIt<T extends X>()` and call `X.static<T>()`. If users want to not implement on the base class, just implement with `throw Unimplemented()` and make sure not pass it. We cannot try to get around this by trying to restrict `X` (like only accepting an `Marker` interface), since then you cannot use functions like `testIt<T extends X>()` because you would need to pass the marker instead.

`YStaticInheritedMethods` implements rather than extends since `Y` could implement more than one `@StaticInheritance` class.