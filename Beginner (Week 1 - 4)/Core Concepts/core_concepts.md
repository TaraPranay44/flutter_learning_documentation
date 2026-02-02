# Dart Core Concepts - Complete Expert Guide

## Table of Contents
1. [Variables & Data Types](#variables--data-types)
2. [Functions & Arrow Functions](#functions--arrow-functions)
3. [Named & Optional Parameters](#named--optional-parameters)
4. [Typedef & Function Types](#typedef--function-types)
5. [Metadata & Annotations](#metadata--annotations)
6. [Library Directives](#library-directives)
7. [Operators](#operators)
8. [Control Flow Statements](#control-flow-statements)

---

## 1. Variables & Data Types

### Understanding `var`, `final`, `const`, `static`, and `late`

#### `var` - Type Inference

```dart
var name = 'John';  // Inferred as String
var age = 25;       // Inferred as int
var height = 5.9;   // Inferred as double
var isActive = true; // Inferred as bool

name = 'Jane';  // ✓ OK
// name = 42;   // ✗ Error: Can't assign int to String
```

**Key Points:**
- Type is inferred at compile-time
- Cannot change type after initialization
- Use when type is obvious from the initializer

#### `final` - Runtime Constant

```dart
final String name = 'John';
final city = 'New York';  // Type inferred

// Cannot reassign
// name = 'Jane';  // ✗ Error

// Runtime evaluation
final currentTime = DateTime.now();
final randomNum = Random().nextInt(100);

class Person {
  final String name;
  final int age;
  
  Person(this.name, this.age);
}

final List<int> numbers = [1, 2, 3];
numbers.add(4);  // ✓ OK - list content is mutable
// numbers = [5, 6];  // ✗ Error - reference is immutable
```

**Key Points:**
- Value set once at runtime
- Must be initialized before use
- Reference is immutable, content can be mutable

#### `const` - Compile-time Constant

```dart
const String appName = 'MyApp';
const int maxUsers = 100;
const double pi = 3.14159;

// Must be compile-time constant
// const time = DateTime.now();  // ✗ Error

const List<String> colors = ['red', 'green', 'blue'];
// colors.add('yellow');  // ✗ Error: Unsupported on const

// Const objects are canonicalized
const p1 = Point(1, 2);
const p2 = Point(1, 2);
print(identical(p1, p2));  // true - same object

class Point {
  final int x, y;
  const Point(this.x, this.y);
}
```

**final vs const:**

```dart
final list1 = [1, 2, 3];
final list2 = [1, 2, 3];
print(identical(list1, list2));  // false

const list3 = [1, 2, 3];
const list4 = [1, 2, 3];
print(identical(list3, list4));  // true
```

#### `static` - Class-level Members

```dart
class MathUtils {
  static const double pi = 3.14159;
  static int calculationCount = 0;
  
  static double circleArea(double radius) {
    calculationCount++;
    return pi * radius * radius;
  }
}

// Access via class name
print(MathUtils.pi);
double area = MathUtils.circleArea(5);
print(MathUtils.calculationCount);

// Singleton pattern
class Database {
  static final Database _instance = Database._internal();
  Database._internal();
  factory Database() => _instance;
}
```

#### `late` - Lazy Initialization

```dart
late String description;

void initialize() {
  description = 'Initialized later';
}

// Lazy evaluation
late String expensiveOperation = _computeValue();

String _computeValue() {
  print('Computing...');
  return 'Result';
}

// _computeValue() called only when accessed
print(expensiveOperation);

// Late final
late final String config;

void loadConfig() {
  config = 'Production';
  // config = 'Dev';  // ✗ Error: Can't set twice
}

class DataLoader {
  late List<String> data;
  
  Future<void> loadData() async {
    await Future.delayed(Duration(seconds: 1));
    data = ['item1', 'item2'];
  }
}
```

### Summary Table

| Keyword | Mutable | Set Time | Use Case |
|---------|---------|----------|----------|
| `var` | Yes | First assignment | Type inference |
| `final` | No | Runtime (once) | Runtime constants |
| `const` | No | Compile-time | Compile-time constants |
| `static` | Depends | N/A | Class-level members |
| `late` | Depends | Deferred | Lazy initialization |

---

## 2. Functions & Arrow Functions

### Regular Functions

```dart
// Basic function
String greet(String name) {
  return 'Hello, $name!';
}

// Multiple parameters
int add(int a, int b) {
  return a + b;
}

// Void function
void printMessage(String message) {
  print(message);
}

// Returning multiple values (Dart 3.0+)
(int, String) getUserInfo() {
  return (25, 'John');
}

var (age, name) = getUserInfo();
```

### Arrow Functions

```dart
// Arrow syntax
String greetArrow(String name) => 'Hello, $name!';
int square(int x) => x * x;
bool isEven(int n) => n % 2 == 0;

// In collections
List<int> numbers = [1, 2, 3, 4, 5];
var doubled = numbers.map((n) => n * 2).toList();
var evens = numbers.where((n) => n % 2 == 0).toList();

// Getters
class Rectangle {
  final double width, height;
  Rectangle(this.width, this.height);
  
  double get area => width * height;
  double get perimeter => 2 * (width + height);
  bool get isSquare => width == height;
}

// Conditional in arrow function
String getGrade(int score) => 
  score >= 90 ? 'A' :
  score >= 80 ? 'B' :
  score >= 70 ? 'C' : 'F';
```

### Anonymous Functions

```dart
var multiply = (int a, int b) => a * b;

List<String> fruits = ['apple', 'banana', 'cherry'];

fruits.forEach((fruit) {
  print('I like $fruit');
});

var upperFruits = fruits.map((fruit) => fruit.toUpperCase()).toList();

var longFruits = fruits.where((fruit) => fruit.length > 5).toList();

List<int> numbers = [1, 2, 3, 4, 5];
var sum = numbers.reduce((value, element) => value + element);
```

### Closures

```dart
Function makeAdder(int addBy) {
  return (int value) => value + addBy;
}

var add2 = makeAdder(2);
var add5 = makeAdder(5);

print(add2(3));  // 5
print(add5(3));  // 8

Function makeCounter() {
  int count = 0;
  return () => ++count;
}

var counter = makeCounter();
print(counter());  // 1
print(counter());  // 2
```

### Higher-Order Functions

```dart
void executeOperation(int a, int b, Function(int, int) operation) {
  print(operation(a, b));
}

executeOperation(10, 5, (a, b) => a + b);  // 15

Function multiplier(int factor) {
  return (int value) => value * factor;
}

// Function composition
T Function(T) compose<T>(T Function(T) f, T Function(T) g) {
  return (T x) => f(g(x));
}

int addOne(int x) => x + 1;
int double_(int x) => x * 2;

var addOneThenDouble = compose(double_, addOne);
print(addOneThenDouble(5));  // 12
```

---

## 3. Named & Optional Parameters

### Positional Parameters

```dart
String greet(String firstName, String lastName) {
  return 'Hello, $firstName $lastName!';
}

print(greet('John', 'Doe'));
```

### Optional Positional Parameters

```dart
String greet(String firstName, [String? lastName]) {
  if (lastName != null) {
    return 'Hello, $firstName $lastName!';
  }
  return 'Hello, $firstName!';
}

print(greet('John'));
print(greet('John', 'Doe'));

// With default values
int add(int a, [int b = 0, int c = 0]) {
  return a + b + c;
}

print(add(5));        // 5
print(add(5, 3));     // 8
print(add(5, 3, 2));  // 10
```

### Named Parameters

```dart
void printUserInfo({String? name, int? age, String? city}) {
  print('Name: $name, Age: $age, City: $city');
}

printUserInfo(name: 'Alice', age: 30, city: 'NYC');
printUserInfo(age: 25, name: 'Bob');  // Order doesn't matter

// Required named parameters
void createUser({
  required String username,
  required String email,
  String? phoneNumber,
}) {
  print('User: $username, Email: $email');
}

// With default values
void configureServer({
  String host = 'localhost',
  int port = 8080,
  bool secure = false,
}) {
  print('Server: ${secure ? "https" : "http"}://$host:$port');
}

configureServer();
configureServer(port: 3000);
configureServer(secure: true, port: 443);
```

### Combining Parameters

```dart
void sendEmail(
  String recipient,
  String subject, {
  String? body,
  List<String>? attachments,
  bool urgent = false,
}) {
  print('To: $recipient, Subject: $subject');
}

sendEmail('alice@example.com', 'Meeting');
sendEmail('bob@example.com', 'Report', body: 'Review', urgent: true);
```

### Super Parameters (Dart 2.17+)

```dart
class Animal {
  final String name;
  final int age;
  Animal({required this.name, required this.age});
}

class Cat extends Animal {
  final String color;
  Cat({required super.name, required super.age, required this.color});
}

var cat = Cat(name: 'Whiskers', age: 3, color: 'orange');
```

---

## 4. Typedef & Function Types

### Basic Typedef

```dart
typedef IntOperation = int Function(int a, int b);

IntOperation add = (a, b) => a + b;
IntOperation multiply = (a, b) => a * b;

int performOperation(int x, int y, IntOperation operation) {
  return operation(x, y);
}

print(performOperation(5, 3, add));       // 8
print(performOperation(5, 3, multiply));  // 15

// Generic typedef
typedef Transformer<T, R> = R Function(T input);

Transformer<String, int> stringLength = (s) => s.length;
Transformer<int, String> intToString = (i) => i.toString();
```

### Complex Signatures

```dart
typedef ApiCallback = void Function(
  Map<String, dynamic> response,
  String? error,
);

void fetchData(String url, ApiCallback callback) {
  Future.delayed(Duration(seconds: 1), () {
    callback({'data': 'success'}, null);
  });
}

// Event handler
typedef EventHandler<T> = void Function(T event);

class EventEmitter<T> {
  final List<EventHandler<T>> _listeners = [];
  
  void addListener(EventHandler<T> handler) {
    _listeners.add(handler);
  }
  
  void emit(T event) {
    for (var handler in _listeners) {
      handler(event);
    }
  }
}
```

### Typedef for Return Types

```dart
typedef Factory<T> = T Function();

class ServiceLocator {
  final Map<Type, Factory> _factories = {};
  
  void register<T>(Factory<T> factory) {
    _factories[T] = factory;
  }
  
  T get<T>() {
    return (_factories[T] as Factory<T>)();
  }
}

var locator = ServiceLocator();
locator.register<String>(() => 'Hello');
print(locator.get<String>());
```

### Type Aliases for Classes

```dart
typedef StringList = List<String>;
typedef IntMap = Map<String, int>;
typedef JsonMap = Map<String, dynamic>;

StringList names = ['Alice', 'Bob'];
IntMap scores = {'Alice': 95, 'Bob': 87};
```

---

## 5. Metadata & Annotations

### Built-in Annotations

#### @override

```dart
class Animal {
  void makeSound() => print('Some sound');
  String get type => 'Animal';
}

class Dog extends Animal {
  @override
  void makeSound() => print('Woof!');
  
  @override
  String get type => 'Dog';
}
```

#### @deprecated

```dart
class OldApi {
  @deprecated
  void oldMethod() {
    print('Deprecated');
  }
  
  @Deprecated('Use newMethod() instead')
  void anotherOldMethod() {
    print('Old');
  }
  
  void newMethod() {
    print('New');
  }
}
```

#### @pragma

```dart
class PerformanceCritical {
  @pragma('vm:prefer-inline')
  int fastComputation(int x) => x * x;
  
  @pragma('vm:never-inline')
  void debugHelper() => print('Debug');
}
```

#### @protected, @visibleForTesting, @immutable, @sealed

```dart
import 'package:meta/meta.dart';

@immutable
class Point {
  final int x, y;
  const Point(this.x, this.y);
}

class Base {
  @protected
  void protectedMethod() {}
}

class DataProcessor {
  @visibleForTesting
  String cleanData(String data) => data.trim();
}

@sealed
class Result {}
class Success extends Result {}
class Failure extends Result {}
```

### Custom Annotations

```dart
class Todo {
  final String description;
  final String assignee;
  const Todo(this.description, {this.assignee = 'unassigned'});
}

@Todo('Implement feature X')
void featureX() {}

@Todo('Fix bug', assignee: 'Alice')
int calculate() => 0;

// Complex annotation
class ApiEndpoint {
  final String method;
  final String path;
  final bool requiresAuth;
  
  const ApiEndpoint({
    required this.method,
    required this.path,
    this.requiresAuth = false,
  });
}

class UserController {
  @ApiEndpoint(method: 'GET', path: '/users')
  void getUsers() {}
  
  @ApiEndpoint(method: 'POST', path: '/users', requiresAuth: true)
  void createUser() {}
}
```

---

## 6. Library Directives

### Import

```dart
// Dart SDK libraries
import 'dart:math';
import 'dart:async';
import 'dart:convert';

// Package imports
import 'package:http/http.dart';
import 'package:flutter/material.dart';

// Local files
import 'models/user.dart';
import '../utils/helpers.dart';
```

### Import with Prefix

```dart
import 'dart:html' as html;
import 'dart:io' as io;

void handleFile() {
  html.window.alert('Browser');
  io.File('test.txt').writeAsStringSync('data');
}

import 'dart:math' as math;
var result = math.sqrt(16);
```

### Show and Hide

```dart
// Import only specific members
import 'dart:math' show Random, pi, sqrt;

// Import everything except
import 'dart:math' hide sin, cos, tan;

// With prefix
import 'package:http/http.dart' as http show get, post;
```

### Deferred Loading

```dart
import 'heavy_library.dart' deferred as heavy;

Future<void> useHeavy() async {
  await heavy.loadLibrary();
  heavy.doSomething();
}
```

### Export

```dart
// lib/models.dart
export 'models/user.dart';
export 'models/product.dart';

// Selective export
export 'src/utils.dart' show capitalize;
export 'src/helpers.dart' hide internalHelper;

// Public API
export 'src/widgets/button.dart';
export 'src/models/user.dart';
```

### Part and Part Of

```dart
// lib/geometry.dart
library geometry;

part 'src/point.dart';
part 'src/line.dart';

int _nextId = 0;

// lib/src/point.dart
part of geometry;

class Point {
  final int id = _nextId++;
  Point();
}
```

### Conditional Imports

```dart
import 'stub.dart'
  if (dart.library.io) 'mobile.dart'
  if (dart.library.html) 'web.dart';

class PlatformInfo {
  static String get platform => /* implementation */;
}
```

---

## 7. Operators

### Arithmetic

```dart
int a = 10, b = 3;

print(a + b);   // 13  Addition
print(a - b);   // 7   Subtraction
print(a * b);   // 30  Multiplication
print(a / b);   // 3.333  Division
print(a ~/ b);  // 3   Integer division
print(a % b);   // 1   Modulo

int x = 5;
print(-x);    // -5
print(++x);   // 6  Pre-increment
print(x++);   // 6  Post-increment
```

### Relational

```dart
print(a == b);   // false
print(a != b);   // true
print(a > b);    // true
print(a < b);    // false
print(a >= b);   // true
print(a <= b);   // false
```

### Logical

```dart
bool a = true, b = false;

print(!a);      // false
print(a && b);  // false
print(a || b);  // true

// Short-circuit
if (true || expensiveCheck()) {
  // expensiveCheck() not called
}
```

### Bitwise

```dart
int a = 5;  // 0101
int b = 3;  // 0011

print(a & b);   // 1  AND
print(a | b);   // 7  OR
print(a ^ b);   // 6  XOR
print(~a);      // -6 NOT
print(a << 1);  // 10 Left shift
print(a >> 1);  // 2  Right shift

// Flags
class Permissions {
  static const int read = 1 << 0;     // 0001
  static const int write = 1 << 1;    // 0010
  static const int execute = 1 << 2;  // 0100
}

int perms = Permissions.read | Permissions.write;
bool canRead = (perms & Permissions.read) != 0;
```

### Null-aware Operators

```dart
// ?. Null-aware access
String? name;
print(name?.length);  // null instead of error

Person? person;
print(person?.address?.city);  // Safe navigation

// ?? Null coalescing
String display = name ?? 'Guest';

String result = first ?? second ?? third ?? 'default';

// ??= Null assignment
String? value;
value ??= 'Default';  // Assigns only if null

// ! Null assert
String nonNull = nullableString!;  // Runtime error if null

Map<String, String> config = {'key': 'value'};
String val = config['key']!;  // Assert it exists
```

### Type Test Operators

```dart
dynamic value = 'Hello';

if (value is String) {
  print(value.length);  // Smart cast
}

if (value is! int) {
  print('Not an integer');
}

// as - Type cast
String str = value as String;
```

### Cascade Operator

```dart
class Person {
  String? name;
  int? age;
  void printInfo() => print('$name, $age');
}

var person = Person()
  ..name = 'Bob'
  ..age = 25
  ..printInfo();

List<int> numbers = []
  ..add(1)
  ..add(2)
  ..addAll([3, 4]);

// Null-aware cascade
Person? maybePerson;
maybePerson
  ?..name = 'Charlie'
  ..age = 35
  ..printInfo();

// Builder pattern
var query = QueryBuilder()
  ..table('users')
  ..select(['id', 'name'])
  ..where({'active': true});
```

### Spread Operator

```dart
List<int> list1 = [1, 2, 3];
List<int> list2 = [4, 5, 6];
List<int> combined = [...list1, ...list2];

// Null-aware spread
List<int>? nullableList;
List<int> safe = [1, 2, ...?nullableList, 3];

// In maps
Map<String, int> map1 = {'a': 1};
Map<String, int> map2 = {'b': 2};
Map<String, int> combined2 = {...map1, ...map2};

// Conditional spreading
bool showExtra = true;
List<int> items = [
  1,
  2,
  if (showExtra) ...[3, 4, 5],
  6,
];
```

### Operator Overloading

```dart
class Vector {
  final double x, y;
  Vector(this.x, this.y);
  
  Vector operator +(Vector other) =>
    Vector(x + other.x, y + other.y);
  
  Vector operator *(double scalar) =>
    Vector(x * scalar, y * scalar);
  
  @override
  bool operator ==(Object other) =>
    other is Vector && x == other.x && y == other.y;
  
  @override
  int get hashCode => Object.hash(x, y);
  
  Vector operator -() => Vector(-x, -y);
  
  double operator [](int index) => index == 0 ? x : y;
}

var v1 = Vector(3, 4);
var v2 = Vector(1, 2);
print(v1 + v2);  // Vector(4, 6)
print(v1 * 2);   // Vector(6, 8)
print(-v1);      // Vector(-3, -4)
```

---

## 8. Control Flow Statements

### If-Else

```dart
int age = 20;

if (age >= 18) {
  print('Adult');
}

if (age >= 18) {
  print('Adult');
} else {
  print('Minor');
}

int score = 85;
if (score >= 90) {
  print('A');
} else if (score >= 80) {
  print('B');
} else if (score >= 70) {
  print('C');
} else {
  print('F');
}

// Type check with smart cast
dynamic value = 'Hello';
if (value is String) {
  print(value.toUpperCase());
}

// Multiple conditions
if (x > 0 && y > 0) {
  print('Both positive');
}

// Null check
String? username;
if (username != null && username.isNotEmpty) {
  print('Hello, $username');
}
```

### Switch

```dart
String day = 'Monday';
switch (day) {
  case 'Monday':
    print('Start of week');
    break;
  case 'Friday':
    print('End of week');
    break;
  case 'Saturday':
  case 'Sunday':
    print('Weekend');
    break;
  default:
    print('Midweek');
}

// With enum
enum Status { pending, approved, rejected }

void handleStatus(Status status) {
  switch (status) {
    case Status.pending:
      print('Awaiting');
      break;
    case Status.approved:
      print('Approved');
      break;
    case Status.rejected:
      print('Rejected');
      break;
  }
}
```

### Switch Expressions (Dart 3.0+)

```dart
String season = month switch {
  'December' || 'January' || 'February' => 'Winter',
  'March' || 'April' || 'May' => 'Spring',
  'June' || 'July' || 'August' => 'Summer',
  'September' || 'October' || 'November' => 'Fall',
  _ => 'Unknown',
};

// Pattern matching
Object obj = 42;
String result = switch (obj) {
  int() => 'Integer',
  String() => 'String',
  List() => 'List',
  _ => 'Other',
};

// Guards
int value = 15;
String category = switch (value) {
  < 0 => 'Negative',
  0 => 'Zero',
  > 0 && < 10 => 'Small',
  >= 10 && < 100 => 'Medium',
  _ => 'Large',
};
```

### For Loops

```dart
// Basic for
for (int i = 0; i < 5; i++) {
  print(i);
}

// For-in
List<String> fruits = ['apple', 'banana', 'cherry'];
for (var fruit in fruits) {
  print(fruit);
}

// Indexed
for (var i = 0; i < fruits.length; i++) {
  print('$i: ${fruits[i]}');
}

// forEach
fruits.forEach((fruit) => print(fruit));

// Map
Map<String, int> ages = {'Alice': 30, 'Bob': 25};
for (var entry in ages.entries) {
  print('${entry.key}: ${entry.value}');
}

// Pattern for loop (Dart 3.0+)
var points = [(1, 2), (3, 4)];
for (var (x, y) in points) {
  print('x=$x, y=$y');
}
```

### While Loops

```dart
int count = 0;
while (count < 5) {
  print(count);
  count++;
}

// Sentinel-controlled
String? input;
while (input != 'quit') {
  // Process
  input = 'quit';
}

// Retry logic
int attempts = 0;
bool success = false;
while (!success && attempts < 3) {
  attempts++;
  success = tryOperation();
}
```

### Do-While

```dart
int count = 0;
do {
  print(count);
  count++;
} while (count < 5);

// Executes at least once
int n = 10;
do {
  print('Runs once: $n');
} while (n < 5);

// Menu pattern
String choice;
do {
  print('Menu');
  choice = '3';
} while (choice != '3');
```

### Break and Continue

```dart
// Break
for (int i = 0; i < 10; i++) {
  if (i == 5) break;
  print(i);  // 0, 1, 2, 3, 4
}

// Labeled break
outerLoop:
for (int i = 0; i < 3; i++) {
  for (int j = 0; j < 3; j++) {
    if (i == 1 && j == 1) {
      break outerLoop;
    }
    print('i: $i, j: $j');
  }
}

// Continue
for (int i = 0; i < 10; i++) {
  if (i % 2 == 0) continue;
  print(i);  // 1, 3, 5, 7, 9
}

// Skip invalid data
for (var item in data) {
  if (item == null || item.isEmpty) continue;
  process(item);
}
```

### Assert

```dart
int age = 25;
assert(age >= 0);
assert(age >= 18, 'Must be adult');

class Rectangle {
  final double width, height;
  
  Rectangle(this.width, this.height) {
    assert(width > 0, 'Width must be positive');
    assert(height > 0, 'Height must be positive');
  }
}

// Only runs in debug mode
// Use exceptions for production errors
```

### Exception Handling

```dart
// Try-catch
try {
  int result = 10 ~/ 0;
} catch (e) {
  print('Error: $e');
}

// Specific exception
try {
  var list = [1, 2, 3];
  print(list[10]);
} on RangeError {
  print('Index out of range');
} catch (e) {
  print('Other error');
}

// Multiple catch
try {
  throw FormatException('Bad format');
} on FormatException catch (e) {
  print('Format: ${e.message}');
} on Exception catch (e) {
  print('General: $e');
} catch (e) {
  print('Unknown: $e');
}

// With stack trace
try {
  throw Exception('Error');
} catch (e, stackTrace) {
  print('Error: $e');
  print('Stack: $stackTrace');
}

// Finally
try {
  print('Opening');
  throw Exception('Error');
} catch (e) {
  print('Error: $e');
} finally {
  print('Closing');  // Always runs
}

// Rethrow
void validate(int age) {
  try {
    if (age < 0) {
      throw ArgumentError('Negative age');
    }
  } catch (e) {
    print('Validation failed');
    rethrow;
  }
}

// Custom exception
class InsufficientFundsException implements Exception {
  final String message;
  final double required, available;
  
  InsufficientFundsException(this.message, this.required, this.available);
  
  @override
  String toString() => 'InsufficientFundsException: $message';
}

void withdraw(double amount, double balance) {
  if (amount > balance) {
    throw InsufficientFundsException('Not enough', amount, balance);
  }
}
```

### Pattern Matching (Dart 3.0+)

```dart
// If-case
Object value = 42;
if (value case int n) {
  print('Integer: $n');
}

if (value case int n when n > 0) {
  print('Positive: $n');
}

// Destructuring
var point = (3, 4);
if (point case (var x, var y)) {
  print('Point: x=$x, y=$y');
}

// List patterns
var list = [1, 2, 3];
if (list case [var first, ...var rest]) {
  print('First: $first, Rest: $rest');
}

// Map patterns
var map = {'name': 'Alice', 'age': 30};
if (map case {'name': var name, 'age': var age}) {
  print('$name is $age');
}

// Sealed classes
sealed class Result {}
class Success extends Result {
  final String data;
  Success(this.data);
}
class Error extends Result {
  final String message;
  Error(this.message);
}

String handleResult(Result result) {
  return switch (result) {
    Success(data: var d) => 'Success: $d',
    Error(message: var m) => 'Error: $m',
  };
}
```

---

## Best Practices

### Variables
- Use `const` for compile-time constants
- Use `final` for runtime constants
- Use `var` when type is obvious
- Use `late` for lazy initialization

### Functions
- Use arrow syntax for simple expressions
- Use named parameters for clarity
- Document complex functions
- Avoid deep nesting

### Control Flow
- Use guard clauses to reduce nesting
- Use switch expressions for cleaner code
- Use early returns
- Handle exceptions appropriately

### Code Organization
- Group related imports
- Use meaningful names
- Keep functions small and focused
- Write self-documenting code

---

## Conclusion

This guide covers all essential Dart core concepts with detailed explanations and practical examples. Master these concepts to write professional, maintainable Dart code.

**Key Takeaways:**
1. Understand variable modifiers for proper state management
2. Use functions effectively with closures and higher-order patterns
3. Leverage named parameters for readable APIs
4. Use typedefs for complex function signatures
5. Apply annotations for metadata and tooling
6. Organize code with library directives
7. Master all operators including null-safety
8. Use appropriate control flow for clean logic

