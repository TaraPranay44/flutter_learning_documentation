# Complete Dart OOP & Flutter Usage Documentation

## Table of Contents
1. [Introduction to Dart OOP](#introduction-to-dart-oop)
2. [Classes and Objects](#classes-and-objects)
3. [Constructors](#constructors)
4. [Inheritance](#inheritance)
5. [Abstraction](#abstraction)
6. [Polymorphism](#polymorphism)
7. [Encapsulation](#encapsulation)
8. [Mixins](#mixins)
9. [Interfaces](#interfaces)
10. [Generics](#generics)
11. [Advanced OOP Concepts](#advanced-oop-concepts)
12. [Flutter Built-in Widget Architecture](#flutter-builtin-widget-architecture)
13. [Custom Widgets in Flutter](#custom-widgets-in-flutter)
14. [State Management Patterns](#state-management-patterns)
15. [Best Practices](#best-practices)
16. [Interview Questions](#interview-questions)

---

## 1. Introduction to Dart OOP

Dart is a purely object-oriented language where everything is an object, including numbers, functions, and null. Every object is an instance of a class, and all classes descend from `Object` class.

### Key OOP Principles in Dart
- **Encapsulation**: Bundling data and methods that operate on that data
- **Inheritance**: Creating new classes from existing ones
- **Abstraction**: Hiding implementation details
- **Polymorphism**: Objects taking multiple forms

---

## 2. Classes and Objects

### Basic Class Definition

```dart
class Person {
  // Instance variables (fields)
  String name;
  int age;
  
  // Method
  void introduce() {
    print('Hello, I am $name and I am $age years old');
  }
}

// Creating an object
void main() {
  var person = Person();
  person.name = 'John';
  person.age = 30;
  person.introduce();
}
```

### Class Members

```dart
class BankAccount {
  // Instance variables
  String accountNumber;
  double balance;
  
  // Static variable (shared across all instances)
  static int totalAccounts = 0;
  
  // Static method
  static void displayTotalAccounts() {
    print('Total accounts: $totalAccounts');
  }
  
  // Instance method
  void deposit(double amount) {
    balance += amount;
  }
  
  // Getter
  double get currentBalance => balance;
  
  // Setter
  set updateBalance(double newBalance) {
    if (newBalance >= 0) {
      balance = newBalance;
    }
  }
}
```

### Private Members

In Dart, privacy is at the library level. Use underscore prefix for private members.

```dart
class User {
  String _password; // Private field
  String username;
  
  // Private method
  bool _validatePassword(String pwd) {
    return pwd.length >= 8;
  }
  
  // Public method using private method
  bool setPassword(String pwd) {
    if (_validatePassword(pwd)) {
      _password = pwd;
      return true;
    }
    return false;
  }
}
```

---

## 3. Constructors

### Default Constructor

```dart
class Product {
  String name;
  double price;
  
  // Default constructor
  Product(this.name, this.price);
}
```

### Named Constructor

```dart
class Rectangle {
  double width;
  double height;
  
  // Default constructor
  Rectangle(this.width, this.height);
  
  // Named constructor
  Rectangle.square(double size) : width = size, height = size;
  
  // Named constructor with factory pattern
  Rectangle.fromMap(Map<String, double> map)
      : width = map['width']!,
        height = map['height']!;
}

void main() {
  var rect1 = Rectangle(10, 20);
  var rect2 = Rectangle.square(15);
  var rect3 = Rectangle.fromMap({'width': 5, 'height': 10});
}
```

### Factory Constructor

Factory constructors can return cached instances or subclass instances.

```dart
class Logger {
  static final Map<String, Logger> _cache = {};
  final String name;
  
  // Private constructor
  Logger._internal(this.name);
  
  // Factory constructor
  factory Logger(String name) {
    return _cache.putIfAbsent(name, () => Logger._internal(name));
  }
  
  void log(String message) {
    print('[$name]: $message');
  }
}

void main() {
  var logger1 = Logger('UI');
  var logger2 = Logger('UI');
  print(identical(logger1, logger2)); // true - same instance
}
```

### Constant Constructor

```dart
class ImmutablePoint {
  final double x;
  final double y;
  
  const ImmutablePoint(this.x, this.y);
}

void main() {
  var p1 = const ImmutablePoint(2, 3);
  var p2 = const ImmutablePoint(2, 3);
  print(identical(p1, p2)); // true - compile-time constants
}
```

### Redirecting Constructor

```dart
class Vehicle {
  String brand;
  String model;
  int year;
  
  Vehicle(this.brand, this.model, this.year);
  
  // Redirecting constructor
  Vehicle.thisYear(String brand, String model) : this(brand, model, 2024);
}
```

### Initializer List

```dart
class Circle {
  final double radius;
  final double area;
  final double circumference;
  
  Circle(double r)
      : radius = r,
        area = 3.14159 * r * r,
        circumference = 2 * 3.14159 * r {
    print('Circle created');
  }
}
```

---

## 4. Inheritance

### Basic Inheritance

```dart
class Animal {
  String name;
  
  Animal(this.name);
  
  void eat() {
    print('$name is eating');
  }
}

class Dog extends Animal {
  String breed;
  
  Dog(String name, this.breed) : super(name);
  
  void bark() {
    print('$name is barking');
  }
  
  // Method overriding
  @override
  void eat() {
    print('$name the dog is eating dog food');
  }
}
```

### Calling Parent Constructor

```dart
class Employee {
  String name;
  double salary;
  
  Employee(this.name, this.salary);
  
  void showDetails() {
    print('Employee: $name, Salary: $salary');
  }
}

class Manager extends Employee {
  List<Employee> team;
  
  // Calling parent constructor
  Manager(String name, double salary, this.team) : super(name, salary);
  
  @override
  void showDetails() {
    super.showDetails(); // Call parent method
    print('Team size: ${team.length}');
  }
}
```

### Multi-level Inheritance

```dart
class LivingBeing {
  void breathe() => print('Breathing...');
}

class Animal extends LivingBeing {
  void move() => print('Moving...');
}

class Dog extends Animal {
  void bark() => print('Barking...');
}

void main() {
  var dog = Dog();
  dog.breathe(); // From LivingBeing
  dog.move();    // From Animal
  dog.bark();    // From Dog
}
```

---

## 5. Abstraction

### Abstract Classes

```dart
abstract class Shape {
  // Abstract method (no implementation)
  double calculateArea();
  double calculatePerimeter();
  
  // Concrete method
  void display() {
    print('Area: ${calculateArea()}');
    print('Perimeter: ${calculatePerimeter()}');
  }
}

class Circle extends Shape {
  double radius;
  
  Circle(this.radius);
  
  @override
  double calculateArea() => 3.14159 * radius * radius;
  
  @override
  double calculatePerimeter() => 2 * 3.14159 * radius;
}

class Rectangle extends Shape {
  double width, height;
  
  Rectangle(this.width, this.height);
  
  @override
  double calculateArea() => width * height;
  
  @override
  double calculatePerimeter() => 2 * (width + height);
}
```

### Real-world Example

```dart
abstract class PaymentProcessor {
  String paymentId;
  double amount;
  
  PaymentProcessor(this.paymentId, this.amount);
  
  // Abstract methods
  bool validatePayment();
  void processPayment();
  void sendReceipt();
  
  // Template method pattern
  void executePayment() {
    if (validatePayment()) {
      processPayment();
      sendReceipt();
    } else {
      print('Payment validation failed');
    }
  }
}

class CreditCardPayment extends PaymentProcessor {
  String cardNumber;
  
  CreditCardPayment(String id, double amount, this.cardNumber) 
      : super(id, amount);
  
  @override
  bool validatePayment() {
    return cardNumber.length == 16;
  }
  
  @override
  void processPayment() {
    print('Processing credit card payment: \$${amount}');
  }
  
  @override
  void sendReceipt() {
    print('Receipt sent for payment ID: $paymentId');
  }
}
```

---

## 6. Polymorphism

### Method Overriding (Runtime Polymorphism)

```dart
class Vehicle {
  void start() {
    print('Vehicle is starting');
  }
}

class Car extends Vehicle {
  @override
  void start() {
    print('Car engine starting...');
  }
}

class Bike extends Vehicle {
  @override
  void start() {
    print('Bike engine starting...');
  }
}

void main() {
  List<Vehicle> vehicles = [Car(), Bike(), Vehicle()];
  
  for (var vehicle in vehicles) {
    vehicle.start(); // Polymorphic behavior
  }
}
```

### Method Overloading (Compile-time Polymorphism)

Dart doesn't support traditional method overloading, but we can achieve similar behavior using optional parameters.

```dart
class Calculator {
  // Using optional positional parameters
  int add(int a, [int? b, int? c]) {
    return a + (b ?? 0) + (c ?? 0);
  }
  
  // Using named parameters
  double calculate({
    required double a,
    double b = 0,
    String operation = 'add',
  }) {
    switch (operation) {
      case 'add':
        return a + b;
      case 'subtract':
        return a - b;
      case 'multiply':
        return a * b;
      default:
        return a;
    }
  }
}

void main() {
  var calc = Calculator();
  print(calc.add(5));           // 5
  print(calc.add(5, 10));       // 15
  print(calc.add(5, 10, 15));   // 30
}
```

### Polymorphism with Interfaces

```dart
abstract class Drawable {
  void draw();
}

abstract class Resizable {
  void resize(double factor);
}

class Square implements Drawable, Resizable {
  double size;
  
  Square(this.size);
  
  @override
  void draw() {
    print('Drawing square of size $size');
  }
  
  @override
  void resize(double factor) {
    size *= factor;
    print('Square resized to $size');
  }
}

void drawShape(Drawable shape) {
  shape.draw();
}

void main() {
  var square = Square(10);
  drawShape(square); // Polymorphic call
}
```

---

## 7. Encapsulation

### Complete Encapsulation Example

```dart
class BankAccount {
  // Private fields
  String _accountHolder;
  String _accountNumber;
  double _balance;
  
  BankAccount(this._accountHolder, this._accountNumber, this._balance);
  
  // Getters
  String get accountHolder => _accountHolder;
  String get accountNumber => _accountNumber;
  double get balance => _balance;
  
  // Controlled setters
  set accountHolder(String name) {
    if (name.isNotEmpty) {
      _accountHolder = name;
    }
  }
  
  // Business logic methods
  bool deposit(double amount) {
    if (amount > 0) {
      _balance += amount;
      return true;
    }
    return false;
  }
  
  bool withdraw(double amount) {
    if (amount > 0 && amount <= _balance) {
      _balance -= amount;
      return true;
    }
    return false;
  }
  
  // Computed property
  String get accountSummary {
    return 'Account: $_accountNumber, Holder: $_accountHolder, Balance: \$${_balance}';
  }
}
```

### Property Validation

```dart
class User {
  String _email;
  int _age;
  
  User(this._email, this._age);
  
  // Getter with validation
  String get email => _email;
  
  // Setter with validation
  set email(String value) {
    if (_isValidEmail(value)) {
      _email = value;
    } else {
      throw ArgumentError('Invalid email format');
    }
  }
  
  int get age => _age;
  
  set age(int value) {
    if (value >= 0 && value <= 150) {
      _age = value;
    } else {
      throw ArgumentError('Age must be between 0 and 150');
    }
  }
  
  bool _isValidEmail(String email) {
    return email.contains('@') && email.contains('.');
  }
}
```

---

## 8. Mixins

Mixins are a way to reuse code in multiple class hierarchies without using inheritance.

### Basic Mixin

```dart
mixin Swimming {
  void swim() {
    print('Swimming...');
  }
}

mixin Flying {
  void fly() {
    print('Flying...');
  }
}

class Animal {
  void breathe() => print('Breathing...');
}

class Duck extends Animal with Swimming, Flying {
  void quack() => print('Quack!');
}

void main() {
  var duck = Duck();
  duck.breathe(); // From Animal
  duck.swim();    // From Swimming mixin
  duck.fly();     // From Flying mixin
  duck.quack();   // From Duck
}
```

### Mixin with 'on' Clause

```dart
class Animal {
  void breathe() => print('Breathing');
}

// Mixin that can only be applied to Animal or its subclasses
mixin Walker on Animal {
  void walk() {
    breathe(); // Can access Animal methods
    print('Walking...');
  }
}

class Dog extends Animal with Walker {
  void bark() => print('Barking');
}

// This would cause an error:
// class Robot with Walker {} // Error: Walker requires Animal
```

### Advanced Mixin Example

```dart
mixin Validation {
  bool validate();
}

mixin Serialization {
  Map<String, dynamic> toJson();
  void fromJson(Map<String, dynamic> json);
}

mixin Timestamped {
  DateTime? createdAt;
  DateTime? updatedAt;
  
  void setCreatedAt() {
    createdAt = DateTime.now();
  }
  
  void setUpdatedAt() {
    updatedAt = DateTime.now();
  }
}

class User with Validation, Serialization, Timestamped {
  String username;
  String email;
  
  User(this.username, this.email) {
    setCreatedAt();
  }
  
  @override
  bool validate() {
    return username.isNotEmpty && email.contains('@');
  }
  
  @override
  Map<String, dynamic> toJson() {
    return {
      'username': username,
      'email': email,
      'createdAt': createdAt?.toIso8601String(),
      'updatedAt': updatedAt?.toIso8601String(),
    };
  }
  
  @override
  void fromJson(Map<String, dynamic> json) {
    username = json['username'];
    email = json['email'];
    createdAt = DateTime.parse(json['createdAt']);
    updatedAt = json['updatedAt'] != null 
        ? DateTime.parse(json['updatedAt']) 
        : null;
  }
}
```

---

## 9. Interfaces

In Dart, every class implicitly defines an interface. There's no separate `interface` keyword.

### Implementing Interfaces

```dart
// This class serves as an interface
class Printer {
  void printDocument(String doc) {
    print('Printing: $doc');
  }
}

class Scanner {
  void scanDocument() {
    print('Scanning document');
  }
}

// Implementing multiple interfaces
class AllInOnePrinter implements Printer, Scanner {
  @override
  void printDocument(String doc) {
    print('All-in-one printing: $doc');
  }
  
  @override
  void scanDocument() {
    print('All-in-one scanning');
  }
  
  void fax() {
    print('Faxing document');
  }
}
```

### Abstract Interface Pattern

```dart
abstract class DataRepository {
  Future<List<String>> fetchData();
  Future<bool> saveData(String data);
  Future<bool> deleteData(String id);
}

class LocalDataRepository implements DataRepository {
  @override
  Future<List<String>> fetchData() async {
    // Local storage implementation
    return ['local data 1', 'local data 2'];
  }
  
  @override
  Future<bool> saveData(String data) async {
    // Save to local storage
    print('Saving to local storage: $data');
    return true;
  }
  
  @override
  Future<bool> deleteData(String id) async {
    // Delete from local storage
    print('Deleting from local storage: $id');
    return true;
  }
}

class RemoteDataRepository implements DataRepository {
  @override
  Future<List<String>> fetchData() async {
    // API call implementation
    await Future.delayed(Duration(seconds: 1));
    return ['remote data 1', 'remote data 2'];
  }
  
  @override
  Future<bool> saveData(String data) async {
    // API POST request
    print('Sending to API: $data');
    return true;
  }
  
  @override
  Future<bool> deleteData(String id) async {
    // API DELETE request
    print('Deleting via API: $id');
    return true;
  }
}
```

---

## 10. Generics

### Generic Classes

```dart
class Box<T> {
  T? _value;
  
  void put(T value) {
    _value = value;
  }
  
  T? get() {
    return _value;
  }
  
  bool isEmpty() {
    return _value == null;
  }
}

void main() {
  var intBox = Box<int>();
  intBox.put(42);
  print(intBox.get()); // 42
  
  var stringBox = Box<String>();
  stringBox.put('Hello');
  print(stringBox.get()); // Hello
}
```

### Generic Methods

```dart
T getFirst<T>(List<T> items) {
  return items.first;
}

void printType<T>(T value) {
  print('Type: ${T}, Value: $value');
}

void main() {
  print(getFirst<int>([1, 2, 3])); // 1
  print(getFirst(['a', 'b', 'c'])); // a
  
  printType<String>('Hello'); // Type: String, Value: Hello
  printType<int>(42);         // Type: int, Value: 42
}
```

### Bounded Generics

```dart
abstract class Comparable<T> {
  int compareTo(T other);
}

class Number implements Comparable<Number> {
  final int value;
  
  Number(this.value);
  
  @override
  int compareTo(Number other) {
    return value.compareTo(other.value);
  }
}

// T must implement Comparable
T findMax<T extends Comparable<T>>(List<T> items) {
  T max = items.first;
  for (var item in items) {
    if (item.compareTo(max) > 0) {
      max = item;
    }
  }
  return max;
}

void main() {
  var numbers = [Number(5), Number(2), Number(8), Number(1)];
  var max = findMax(numbers);
  print(max.value); // 8
}
```

### Generic Collections

```dart
class Stack<T> {
  final List<T> _items = [];
  
  void push(T item) {
    _items.add(item);
  }
  
  T? pop() {
    if (_items.isEmpty) return null;
    return _items.removeLast();
  }
  
  T? peek() {
    if (_items.isEmpty) return null;
    return _items.last;
  }
  
  bool get isEmpty => _items.isEmpty;
  int get size => _items.length;
}

void main() {
  var stack = Stack<String>();
  stack.push('first');
  stack.push('second');
  stack.push('third');
  
  print(stack.pop()); // third
  print(stack.peek()); // second
  print(stack.size); // 2
}
```

---

## 11. Advanced OOP Concepts

### Extension Methods

```dart
extension StringExtension on String {
  String capitalize() {
    if (isEmpty) return this;
    return '${this[0].toUpperCase()}${substring(1)}';
  }
  
  bool isValidEmail() {
    return contains('@') && contains('.');
  }
  
  String reverse() {
    return split('').reversed.join('');
  }
}

void main() {
  print('hello'.capitalize()); // Hello
  print('test@email.com'.isValidEmail()); // true
  print('dart'.reverse()); // trad
}
```

### Callable Classes

```dart
class Multiplier {
  final int factor;
  
  Multiplier(this.factor);
  
  // Make the class callable
  int call(int value) {
    return value * factor;
  }
}

void main() {
  var triple = Multiplier(3);
  print(triple(5)); // 15
  print(triple(10)); // 30
}
```

### Cascade Notation

```dart
class Person {
  String? name;
  int? age;
  String? address;
  
  void printInfo() {
    print('Name: $name, Age: $age, Address: $address');
  }
}

void main() {
  // Without cascade
  var person1 = Person();
  person1.name = 'John';
  person1.age = 30;
  person1.address = '123 Street';
  person1.printInfo();
  
  // With cascade
  var person2 = Person()
    ..name = 'Jane'
    ..age = 25
    ..address = '456 Avenue'
    ..printInfo();
}
```

### Operator Overloading

```dart
class Vector {
  final double x, y;
  
  Vector(this.x, this.y);
  
  // Overload + operator
  Vector operator +(Vector other) {
    return Vector(x + other.x, y + other.y);
  }
  
  // Overload - operator
  Vector operator -(Vector other) {
    return Vector(x - other.x, y - other.y);
  }
  
  // Overload * operator
  Vector operator *(double scalar) {
    return Vector(x * scalar, y * scalar);
  }
  
  // Overload == operator
  @override
  bool operator ==(Object other) {
    if (other is! Vector) return false;
    return x == other.x && y == other.y;
  }
  
  @override
  int get hashCode => Object.hash(x, y);
  
  @override
  String toString() => 'Vector($x, $y)';
}

void main() {
  var v1 = Vector(2, 3);
  var v2 = Vector(4, 5);
  
  print(v1 + v2); // Vector(6.0, 8.0)
  print(v1 - v2); // Vector(-2.0, -2.0)
  print(v1 * 2);  // Vector(4.0, 6.0)
  print(v1 == Vector(2, 3)); // true
}
```

### Covariant Keyword

```dart
class Animal {
  void chase(Animal target) {
    print('Animal chasing animal');
  }
}

class Cat extends Animal {
  // Covariant allows overriding with more specific type
  @override
  void chase(covariant Cat target) {
    print('Cat chasing cat');
  }
}

class Mouse extends Animal {}

void main() {
  Animal animal = Cat();
  animal.chase(Cat()); // Cat chasing cat
}
```

---

## 12. Flutter Built-in Widget Architecture

### StatelessWidget

```dart
import 'package:flutter/material.dart';

class MyButton extends StatelessWidget {
  final String text;
  final VoidCallback onPressed;
  final Color? backgroundColor;
  
  const MyButton({
    Key? key,
    required this.text,
    required this.onPressed,
    this.backgroundColor,
  }) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: onPressed,
      style: ElevatedButton.styleFrom(
        backgroundColor: backgroundColor ?? Colors.blue,
      ),
      child: Text(text),
    );
  }
}

// Usage
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: Center(
          child: MyButton(
            text: 'Click Me',
            onPressed: () => print('Button clicked'),
            backgroundColor: Colors.green,
          ),
        ),
      ),
    );
  }
}
```

### StatefulWidget

```dart
class Counter extends StatefulWidget {
  final int initialValue;
  
  const Counter({Key? key, this.initialValue = 0}) : super(key: key);
  
  @override
  State<Counter> createState() => _CounterState();
}

class _CounterState extends State<Counter> {
  late int _count;
  
  @override
  void initState() {
    super.initState();
    _count = widget.initialValue;
  }
  
  void _increment() {
    setState(() {
      _count++;
    });
  }
  
  void _decrement() {
    setState(() {
      _count--;
    });
  }
  
  @override
  Widget build(BuildContext context) {
    return Column(
      mainAxisAlignment: MainAxisAlignment.center,
      children: [
        Text(
          'Count: $_count',
          style: TextStyle(fontSize: 24),
        ),
        SizedBox(height: 20),
        Row(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton(
              onPressed: _decrement,
              child: Text('-'),
            ),
            SizedBox(width: 20),
            ElevatedButton(
              onPressed: _increment,
              child: Text('+'),
            ),
          ],
        ),
      ],
    );
  }
}
```

### InheritedWidget

```dart
class AppTheme extends InheritedWidget {
  final Color primaryColor;
  final Color accentColor;
  
  const AppTheme({
    Key? key,
    required this.primaryColor,
    required this.accentColor,
    required Widget child,
  }) : super(key: key, child: child);
  
  static AppTheme? of(BuildContext context) {
    return context.dependOnInheritedWidgetOfExactType<AppTheme>();
  }
  
  @override
  bool updateShouldNotify(AppTheme oldWidget) {
    return primaryColor != oldWidget.primaryColor ||
           accentColor != oldWidget.accentColor;
  }
}

// Usage
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final theme = AppTheme.of(context);
    
    return Container(
      color: theme?.primaryColor,
      child: Text(
        'Themed Widget',
        style: TextStyle(color: theme?.accentColor),
      ),
    );
  }
}
```

### Widget Lifecycle

```dart
class LifecycleDemo extends StatefulWidget {
  @override
  State<LifecycleDemo> createState() => _LifecycleDemoState();
}

class _LifecycleDemoState extends State<LifecycleDemo> 
    with WidgetsBindingObserver {
  
  @override
  void initState() {
    super.initState();
    print('1. initState called');
    WidgetsBinding.instance.addObserver(this);
  }
  
  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    print('2. didChangeDependencies called');
  }
  
  @override
  Widget build(BuildContext context) {
    print('3. build called');
    return Container();
  }
  
  @override
  void didUpdateWidget(covariant LifecycleDemo oldWidget) {
    super.didUpdateWidget(oldWidget);
    print('4. didUpdateWidget called');
  }
  
  @override
  void deactivate() {
    print('5. deactivate called');
    super.deactivate();
  }
  
  @override
  void dispose() {
    print('6. dispose called');
    WidgetsBinding.instance.removeObserver(this);
    super.dispose();
  }
  
  @override
  void didChangeAppLifecycleState(AppLifecycleState state) {
    print('App lifecycle state: $state');
  }
}
```

---

## 13. Custom Widgets in Flutter

### Custom Painter

```dart
class CirclePainter extends CustomPainter {
  final Color color;
  final double strokeWidth;
  
  CirclePainter({
    required this.color,
    this.strokeWidth = 2.0,
  });
  
  @override
  void paint(Canvas canvas, Size size) {
    final paint = Paint()
      ..color = color
      ..strokeWidth = strokeWidth
      ..style = PaintingStyle.stroke;
    
    final center = Offset(size.width / 2, size.height / 2);
    final radius = size.width / 2 - strokeWidth;
    
    canvas.drawCircle(center, radius, paint);
  }
  
  @override
  bool shouldRepaint(CirclePainter oldDelegate) {
    return oldDelegate.color != color || 
           oldDelegate.strokeWidth != strokeWidth;
  }
}

// Usage
class CustomCircle extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return CustomPaint(
      size: Size(200, 200),
      painter: CirclePainter(
        color: Colors.blue,
        strokeWidth: 4.0,
      ),
    );
  }
}
```

### Render Object Widget

```dart
class CustomBox extends LeafRenderObjectWidget {
  final Color color;
  
  const CustomBox({Key? key, required this.color}) : super(key: key);
  
  @override
  RenderObject createRenderObject(BuildContext context) {
    return RenderCustomBox(color);
  }
  
  @override
  void updateRenderObject(
    BuildContext context,
    RenderCustomBox renderObject,
  ) {
    renderObject.color = color;
  }
}

class RenderCustomBox extends RenderBox {
  Color _color;
  
  RenderCustomBox(this._color);
  
  Color get color => _color;
  set color(Color value) {
    if (_color == value) return;
    _color = value;
    markNeedsPaint();
  }
  
  @override
  void performLayout() {
    size = constraints.constrain(Size(100, 100));
  }
  
  @override
  void paint(PaintingContext context, Offset offset) {
    final paint = Paint()..color = _color;
    context.canvas.drawRect(offset & size, paint);
  }
}
```

### Composite Widget Pattern

```dart
class UserCard extends StatelessWidget {
  final String name;
  final String email;
  final String avatarUrl;
  final VoidCallback? onTap;
  
  const UserCard({
    Key? key,
    required this.name,
    required this.email,
    required this.avatarUrl,
    this.onTap,
  }) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return Card(
      elevation: 4,
      margin: EdgeInsets.all(8),
      child: InkWell(
        onTap: onTap,
        child: Padding(
          padding: EdgeInsets.all(16),
          child: Row(
            children: [
              _buildAvatar(),
              SizedBox(width: 16),
              Expanded(child: _buildInfo()),
              _buildTrailingIcon(),
            ],
          ),
        ),
      ),
    );
  }
  
  Widget _buildAvatar() {
    return CircleAvatar(
      radius: 30,
      backgroundImage: NetworkImage(avatarUrl),
    );
  }
  
  Widget _buildInfo() {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text(
          name,
          style: TextStyle(
            fontSize: 18,
            fontWeight: FontWeight.bold,
          ),
        ),
        SizedBox(height: 4),
        Text(
          email,
          style: TextStyle(
            fontSize: 14,
            color: Colors.grey[600],
          ),
        ),
      ],
    );
  }
  
  Widget _buildTrailingIcon() {
    return Icon(Icons.arrow_forward_ios, size: 16);
  }
}
```

### Builder Pattern for Widgets

```dart
class CustomDialog {
  final String? title;
  final String? message;
  final String? confirmText;
  final String? cancelText;
  final VoidCallback? onConfirm;
  final VoidCallback? onCancel;
  
  CustomDialog._({
    this.title,
    this.message,
    this.confirmText,
    this.cancelText,
    this.onConfirm,
    this.onCancel,
  });
  
  factory CustomDialog.builder() => CustomDialog._();
  
  CustomDialog withTitle(String title) {
    return CustomDialog._(
      title: title,
      message: message,
      confirmText: confirmText,
      cancelText: cancelText,
      onConfirm: onConfirm,
      onCancel: onCancel,
    );
  }
  
  CustomDialog withMessage(String message) {
    return CustomDialog._(
      title: title,
      message: message,
      confirmText: confirmText,
      cancelText: cancelText,
      onConfirm: onConfirm,
      onCancel: onCancel,
    );
  }
  
  CustomDialog withConfirmButton(String text, VoidCallback callback) {
    return CustomDialog._(
      title: title,
      message: message,
      confirmText: text,
      cancelText: cancelText,
      onConfirm: callback,
      onCancel: onCancel,
    );
  }
  
  void show(BuildContext context) {
    showDialog(
      context: context,
      builder: (_) => AlertDialog(
        title: title != null ? Text(title!) : null,
        content: message != null ? Text(message!) : null,
        actions: [
          if (cancelText != null)
            TextButton(
              onPressed: onCancel ?? () => Navigator.pop(context),
              child: Text(cancelText!),
            ),
          if (confirmText != null)
            ElevatedButton(
              onPressed: onConfirm ?? () => Navigator.pop(context),
              child: Text(confirmText!),
            ),
        ],
      ),
    );
  }
}

// Usage
CustomDialog.builder()
  .withTitle('Confirm Action')
  .withMessage('Are you sure you want to proceed?')
  .withConfirmButton('Yes', () => print('Confirmed'))
  .show(context);
```

---

## 14. State Management Patterns

### Provider Pattern

```dart
import 'package:flutter/foundation.dart';

class CounterProvider extends ChangeNotifier {
  int _count = 0;
  
  int get count => _count;
  
  void increment() {
    _count++;
    notifyListeners();
  }
  
  void decrement() {
    _count--;
    notifyListeners();
  }
  
  void reset() {
    _count = 0;
    notifyListeners();
  }
}

// Usage with ChangeNotifierProvider
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (_) => CounterProvider(),
      child: MaterialApp(
        home: CounterScreen(),
      ),
    );
  }
}

class CounterScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Consumer<CounterProvider>(
          builder: (context, provider, child) {
            return Text('Count: ${provider.count}');
          },
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          context.read<CounterProvider>().increment();
        },
        child: Icon(Icons.add),
      ),
    );
  }
}
```

### BLoC Pattern

```dart
import 'dart:async';

// Events
abstract class CounterEvent {}
class IncrementEvent extends CounterEvent {}
class DecrementEvent extends CounterEvent {}

// States
class CounterState {
  final int count;
  CounterState(this.count);
}

// BLoC
class CounterBloc {
  int _count = 0;
  
  final _stateController = StreamController<CounterState>();
  Stream<CounterState> get stateStream => _stateController.stream;
  
  final _eventController = StreamController<CounterEvent>();
  Sink<CounterEvent> get eventSink => _eventController.sink;
  
  CounterBloc() {
    _eventController.stream.listen(_mapEventToState);
  }
  
  void _mapEventToState(CounterEvent event) {
    if (event is IncrementEvent) {
      _count++;
    } else if (event is DecrementEvent) {
      _count--;
    }
    _stateController.sink.add(CounterState(_count));
  }
  
  void dispose() {
    _stateController.close();
    _eventController.close();
  }
}

// Usage
class CounterPage extends StatefulWidget {
  @override
  _CounterPageState createState() => _CounterPageState();
}

class _CounterPageState extends State<CounterPage> {
  final _bloc = CounterBloc();
  
  @override
  void dispose() {
    _bloc.dispose();
    super.dispose();
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: StreamBuilder<CounterState>(
        stream: _bloc.stateStream,
        initialData: CounterState(0),
        builder: (context, snapshot) {
          return Center(
            child: Text('Count: ${snapshot.data!.count}'),
          );
        },
      ),
      floatingActionButton: Column(
        mainAxisAlignment: MainAxisAlignment.end,
        children: [
          FloatingActionButton(
            onPressed: () => _bloc.eventSink.add(IncrementEvent()),
            child: Icon(Icons.add),
          ),
          SizedBox(height: 10),
          FloatingActionButton(
            onPressed: () => _bloc.eventSink.add(DecrementEvent()),
            child: Icon(Icons.remove),
          ),
        ],
      ),
    );
  }
}
```

### GetX Pattern

```dart
import 'package:get/get.dart';

class CounterController extends GetxController {
  var count = 0.obs;
  
  void increment() => count++;
  void decrement() => count--;
  
  @override
  void onInit() {
    super.onInit();
    print('Controller initialized');
  }
  
  @override
  void onClose() {
    print('Controller disposed');
    super.onClose();
  }
}

// Usage
class CounterView extends StatelessWidget {
  final CounterController controller = Get.put(CounterController());
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Obx(() => Text('Count: ${controller.count}')),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: controller.increment,
        child: Icon(Icons.add),
      ),
    );
  }
}
```

### Repository Pattern

```dart
// Data Model
class User {
  final String id;
  final String name;
  final String email;
  
  User({required this.id, required this.name, required this.email});
  
  factory User.fromJson(Map<String, dynamic> json) {
    return User(
      id: json['id'],
      name: json['name'],
      email: json['email'],
    );
  }
  
  Map<String, dynamic> toJson() {
    return {'id': id, 'name': name, 'email': email};
  }
}

// Data Source Interface
abstract class UserDataSource {
  Future<List<User>> getUsers();
  Future<User> getUserById(String id);
  Future<bool> createUser(User user);
}

// Remote Data Source
class RemoteUserDataSource implements UserDataSource {
  @override
  Future<List<User>> getUsers() async {
    // API call
    await Future.delayed(Duration(seconds: 1));
    return [
      User(id: '1', name: 'John', email: 'john@example.com'),
      User(id: '2', name: 'Jane', email: 'jane@example.com'),
    ];
  }
  
  @override
  Future<User> getUserById(String id) async {
    // API call
    await Future.delayed(Duration(seconds: 1));
    return User(id: id, name: 'John', email: 'john@example.com');
  }
  
  @override
  Future<bool> createUser(User user) async {
    // API POST
    await Future.delayed(Duration(seconds: 1));
    return true;
  }
}

// Local Data Source
class LocalUserDataSource implements UserDataSource {
  final List<User> _cache = [];
  
  @override
  Future<List<User>> getUsers() async {
    return _cache;
  }
  
  @override
  Future<User> getUserById(String id) async {
    return _cache.firstWhere((user) => user.id == id);
  }
  
  @override
  Future<bool> createUser(User user) async {
    _cache.add(user);
    return true;
  }
}

// Repository
class UserRepository {
  final RemoteUserDataSource remoteDataSource;
  final LocalUserDataSource localDataSource;
  
  UserRepository({
    required this.remoteDataSource,
    required this.localDataSource,
  });
  
  Future<List<User>> getUsers() async {
    try {
      final users = await remoteDataSource.getUsers();
      // Cache locally
      for (var user in users) {
        await localDataSource.createUser(user);
      }
      return users;
    } catch (e) {
      // Fallback to local cache
      return await localDataSource.getUsers();
    }
  }
  
  Future<User?> getUserById(String id) async {
    try {
      return await remoteDataSource.getUserById(id);
    } catch (e) {
      try {
        return await localDataSource.getUserById(id);
      } catch (e) {
        return null;
      }
    }
  }
}
```

---

## 15. Best Practices

### SOLID Principles in Dart

#### Single Responsibility Principle

```dart
// Bad - Multiple responsibilities
class User {
  String name;
  String email;
  
  User(this.name, this.email);
  
  void save() {
    // Database logic
  }
  
  void sendEmail() {
    // Email logic
  }
  
  void generateReport() {
    // Report logic
  }
}

// Good - Single responsibility
class User {
  final String name;
  final String email;
  
  User(this.name, this.email);
}

class UserRepository {
  Future<void> save(User user) async {
    // Database logic
  }
}

class EmailService {
  void sendEmail(User user, String message) {
    // Email logic
  }
}

class ReportGenerator {
  String generateUserReport(User user) {
    // Report logic
    return 'Report for ${user.name}';
  }
}
```

#### Open/Closed Principle

```dart
// Bad - Need to modify class to add new shapes
class AreaCalculator {
  double calculate(Object shape) {
    if (shape is Circle) {
      return 3.14 * shape.radius * shape.radius;
    } else if (shape is Rectangle) {
      return shape.width * shape.height;
    }
    return 0;
  }
}

// Good - Open for extension, closed for modification
abstract class Shape {
  double calculateArea();
}

class Circle implements Shape {
  final double radius;
  Circle(this.radius);
  
  @override
  double calculateArea() => 3.14 * radius * radius;
}

class Rectangle implements Shape {
  final double width, height;
  Rectangle(this.width, this.height);
  
  @override
  double calculateArea() => width * height;
}

class Triangle implements Shape {
  final double base, height;
  Triangle(this.base, this.height);
  
  @override
  double calculateArea() => 0.5 * base * height;
}

class AreaCalculator {
  double calculate(Shape shape) => shape.calculateArea();
}
```

#### Liskov Substitution Principle

```dart
// Bad - Violates LSP
class Bird {
  void fly() => print('Flying');
}

class Penguin extends Bird {
  @override
  void fly() {
    throw Exception('Penguins cannot fly!');
  }
}

// Good - Follows LSP
abstract class Bird {
  void move();
}

class FlyingBird extends Bird {
  @override
  void move() => print('Flying');
}

class Penguin extends Bird {
  @override
  void move() => print('Swimming');
}
```

#### Interface Segregation Principle

```dart
// Bad - Fat interface
abstract class Worker {
  void work();
  void eat();
  void sleep();
}

// Good - Segregated interfaces
abstract class Workable {
  void work();
}

abstract class Eatable {
  void eat();
}

abstract class Sleepable {
  void sleep();
}

class Human implements Workable, Eatable, Sleepable {
  @override
  void work() => print('Working');
  
  @override
  void eat() => print('Eating');
  
  @override
  void sleep() => print('Sleeping');
}

class Robot implements Workable {
  @override
  void work() => print('Working 24/7');
}
```

#### Dependency Inversion Principle

```dart
// Bad - High-level module depends on low-level module
class MySQLDatabase {
  void save(String data) {
    print('Saving to MySQL: $data');
  }
}

class UserService {
  final MySQLDatabase database = MySQLDatabase();
  
  void saveUser(String user) {
    database.save(user);
  }
}

// Good - Both depend on abstraction
abstract class Database {
  void save(String data);
}

class MySQLDatabase implements Database {
  @override
  void save(String data) {
    print('Saving to MySQL: $data');
  }
}

class MongoDatabase implements Database {
  @override
  void save(String data) {
    print('Saving to MongoDB: $data');
  }
}

class UserService {
  final Database database;
  
  UserService(this.database);
  
  void saveUser(String user) {
    database.save(user);
  }
}

// Usage
void main() {
  var service1 = UserService(MySQLDatabase());
  var service2 = UserService(MongoDatabase());
  
  service1.saveUser('John');
  service2.saveUser('Jane');
}
```

### Design Patterns

#### Singleton Pattern

```dart
class AppConfig {
  static final AppConfig _instance = AppConfig._internal();
  
  factory AppConfig() => _instance;
  
  AppConfig._internal();
  
  String apiUrl = 'https://api.example.com';
  String appVersion = '1.0.0';
}

// Usage
void main() {
  var config1 = AppConfig();
  var config2 = AppConfig();
  
  print(identical(config1, config2)); // true
}
```

#### Factory Pattern

```dart
abstract class Vehicle {
  void drive();
}

class Car implements Vehicle {
  @override
  void drive() => print('Driving a car');
}

class Bike implements Vehicle {
  @override
  void drive() => print('Riding a bike');
}

class VehicleFactory {
  static Vehicle createVehicle(String type) {
    switch (type.toLowerCase()) {
      case 'car':
        return Car();
      case 'bike':
        return Bike();
      default:
        throw ArgumentError('Unknown vehicle type');
    }
  }
}

// Usage
void main() {
  var car = VehicleFactory.createVehicle('car');
  var bike = VehicleFactory.createVehicle('bike');
  
  car.drive();
  bike.drive();
}
```

#### Observer Pattern

```dart
abstract class Observer {
  void update(String message);
}

class Subject {
  final List<Observer> _observers = [];
  
  void attach(Observer observer) {
    _observers.add(observer);
  }
  
  void detach(Observer observer) {
    _observers.remove(observer);
  }
  
  void notify(String message) {
    for (var observer in _observers) {
      observer.update(message);
    }
  }
}

class EmailNotifier implements Observer {
  final String email;
  
  EmailNotifier(this.email);
  
  @override
  void update(String message) {
    print('Email to $email: $message');
  }
}

class SMSNotifier implements Observer {
  final String phone;
  
  SMSNotifier(this.phone);
  
  @override
  void update(String message) {
    print('SMS to $phone: $message');
  }
}

// Usage
void main() {
  var subject = Subject();
  
  subject.attach(EmailNotifier('user@example.com'));
  subject.attach(SMSNotifier('+1234567890'));
  
  subject.notify('New notification!');
}
```

#### Strategy Pattern

```dart
abstract class PaymentStrategy {
  void pay(double amount);
}

class CreditCardPayment implements PaymentStrategy {
  final String cardNumber;
  
  CreditCardPayment(this.cardNumber);
  
  @override
  void pay(double amount) {
    print('Paid \$amount using credit card $cardNumber');
  }
}

class PayPalPayment implements PaymentStrategy {
  final String email;
  
  PayPalPayment(this.email);
  
  @override
  void pay(double amount) {
    print('Paid \$amount using PayPal account $email');
  }
}

class ShoppingCart {
  PaymentStrategy? paymentStrategy;
  
  void setPaymentStrategy(PaymentStrategy strategy) {
    paymentStrategy = strategy;
  }
  
  void checkout(double amount) {
    if (paymentStrategy != null) {
      paymentStrategy!.pay(amount);
    } else {
      print('Please select a payment method');
    }
  }
}

// Usage
void main() {
  var cart = ShoppingCart();
  
  cart.setPaymentStrategy(CreditCardPayment('1234-5678-9012-3456'));
  cart.checkout(100.0);
  
  cart.setPaymentStrategy(PayPalPayment('user@example.com'));
  cart.checkout(50.0);
}
```

---

## 16. Interview Questions

### Beginner Level

**Q1: What is the difference between `final` and `const` in Dart?**

A: `final` means the variable can only be set once and is initialized at runtime. `const` means the variable is a compile-time constant and must be initialized with a constant value.

```dart
final name = 'John'; // Runtime constant
const pi = 3.14159;  // Compile-time constant

final currentTime = DateTime.now(); // Valid
// const currentTime2 = DateTime.now(); // Error - not compile-time constant
```

**Q2: Explain the difference between `extends`, `implements`, and `with`.**

A:
- `extends`: Inheritance - inherit implementation from parent class
- `implements`: Interface - must implement all methods
- `with`: Mixin - add functionality without inheritance

```dart
class Animal { void breathe() {} }
abstract class Runnable { void run(); }
mixin Flying { void fly() {} }

class Bird extends Animal with Flying implements Runnable {
  @override
  void run() => print('Running');
}
```

**Q3: What is the difference between StatelessWidget and StatefulWidget?**

A: StatelessWidget is immutable and doesn't change over time. StatefulWidget has mutable state that can change during the widget's lifetime and trigger rebuilds.

### Intermediate Level

**Q4: Explain the widget tree, element tree, and render tree in Flutter.**

A:
- **Widget Tree**: Immutable configuration for UI elements
- **Element Tree**: Manages lifecycle and holds references to widgets and render objects
- **Render Tree**: Handles layout, painting, and hit testing

**Q5: How does Flutter's `setState()` work internally?**

A: `setState()` marks the widget as dirty and schedules a rebuild. During the next frame, Flutter calls the `build()` method to get the new widget configuration and updates the UI accordingly.

**Q6: What are mixins and when should you use them?**

A: Mixins are a way to reuse code in multiple class hierarchies without using inheritance. Use them when you need to share behavior across unrelated classes.

```dart
mixin Validator {
  bool isValidEmail(String email) => email.contains('@');
}

class User with Validator {
  String email;
  User(this.email);
  
  bool validate() => isValidEmail(email);
}
```

### Advanced Level

**Q7: Explain the differences between `InheritedWidget`, `Provider`, and `Riverpod`.**

A:
- **InheritedWidget**: Low-level Flutter mechanism for passing data down the tree
- **Provider**: Wrapper around InheritedWidget with dependency injection
- **Riverpod**: Compile-safe, testable provider with no dependency on BuildContext

**Q8: How would you implement dependency injection in Flutter?**

A:
```dart
// Service locator pattern
class ServiceLocator {
  static final ServiceLocator _instance = ServiceLocator._internal();
  factory ServiceLocator() => _instance;
  ServiceLocator._internal();
  
  final Map<Type, dynamic> _services = {};
  
  void register<T>(T service) {
    _services[T] = service;
  }
  
  T get<T>() {
    return _services[T] as T;
  }
}

// Usage
void main() {
  ServiceLocator().register<ApiService>(ApiService());
  
  var service = ServiceLocator().get<ApiService>();
}
```

**Q9: Explain the difference between `async`, `async*`, `sync`, and `sync*`.**

A:
- `async`: Returns a `Future`
- `async*`: Returns a `Stream`
- `sync`: Regular synchronous function
- `sync*`: Returns an `Iterable` (synchronous generator)

```dart
Future<int> asyncFunction() async {
  return 42;
}

Stream<int> asyncGenerator() async* {
  for (int i = 0; i < 5; i++) {
    await Future.delayed(Duration(seconds: 1));
    yield i;
  }
}

Iterable<int> syncGenerator() sync* {
  for (int i = 0; i < 5; i++) {
    yield i;
  }
}
```

**Q10: How do you prevent memory leaks in Flutter?**

A:
1. Dispose controllers, streams, and animations
2. Cancel subscriptions in `dispose()`
3. Use weak references when needed
4. Avoid retaining references to widgets
5. Use `AutomaticKeepAliveClientMixin` carefully

```dart
class MyWidget extends StatefulWidget {
  @override
  _MyWidgetState createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> {
  late StreamSubscription _subscription;
  late AnimationController _controller;
  
  @override
  void initState() {
    super.initState();
    _controller = AnimationController(vsync: this);
    _subscription = someStream.listen((data) {
      // Handle data
    });
  }
  
  @override
  void dispose() {
    _controller.dispose();
    _subscription.cancel();
    super.dispose();
  }
  
  @override
  Widget build(BuildContext context) {
    return Container();
  }
}
```

### Expert Level

**Q11: Explain how Flutter's rendering pipeline works.**

A: Flutter's rendering pipeline consists of:
1. **Build Phase**: Widget tree is built/rebuilt
2. **Layout Phase**: Render objects calculate sizes and positions
3. **Paint Phase**: Render objects paint themselves to layers
4. **Composite Phase**: Layers are composited and sent to the GPU

**Q12: How would you optimize a Flutter app with a large list?**

A:
```dart
// Use ListView.builder for lazy loading
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) {
    return ListTile(title: Text(items[index]));
  },
);

// Use const constructors
const ListTile(title: Text('Static item'));

// Implement AutomaticKeepAliveClientMixin when needed
class MyItem extends StatefulWidget {
  @override
  _MyItemState createState() => _MyItemState();
}

class _MyItemState extends State<MyItem> 
    with AutomaticKeepAliveClientMixin {
  @override
  bool get wantKeepAlive => true;
  
  @override
  Widget build(BuildContext context) {
    super.build(context); // Important for keep alive
    return Container();
  }
}

// Use RepaintBoundary for expensive widgets
RepaintBoundary(
  child: ExpensiveWidget(),
);
```

**Q13: Implement a custom RenderObject that creates a circular layout.**

A:
```dart
class CircularLayout extends MultiChildRenderObjectWidget {
  CircularLayout({required List<Widget> children}) : super(children: children);
  
  @override
  RenderObject createRenderObject(BuildContext context) {
    return RenderCircularLayout();
  }
}

class CircularLayoutParentData extends ContainerBoxParentData<RenderBox> {}

class RenderCircularLayout extends RenderBox
    with ContainerRenderObjectMixin<RenderBox, CircularLayoutParentData> {
  
  @override
  void setupParentData(RenderBox child) {
    if (child.parentData is! CircularLayoutParentData) {
      child.parentData = CircularLayoutParentData();
    }
  }
  
  @override
  void performLayout() {
    size = constraints.biggest;
    
    if (childCount == 0) return;
    
    final radius = size.width / 2;
    final center = Offset(size.width / 2, size.height / 2);
    final angleStep = 2 * 3.14159 / childCount;
    
    var child = firstChild;
    var index = 0;
    
    while (child != null) {
      child.layout(constraints.loosen(), parentUsesSize: true);
      
      final angle = angleStep * index;
      final x = center.dx + radius * 0.7 * cos(angle);
      final y = center.dy + radius * 0.7 * sin(angle);
      
      final childParentData = child.parentData as CircularLayoutParentData;
      childParentData.offset = Offset(
        x - child.size.width / 2,
        y - child.size.height / 2,
      );
      
      child = childParentData.nextSibling;
      index++;
    }
  }
  
  @override
  void paint(PaintingContext context, Offset offset) {
    var child = firstChild;
    while (child != null) {
      final childParentData = child.parentData as CircularLayoutParentData;
      context.paintChild(child, childParentData.offset + offset);
      child = childParentData.nextSibling;
    }
  }
}
```

---

## Summary

This comprehensive guide covers:

1. **Core OOP Concepts**: Classes, objects, inheritance, polymorphism, abstraction, encapsulation
2. **Dart-Specific Features**: Mixins, extension methods, generics, factory constructors
3. **Flutter Architecture**: Widget tree, state management, custom widgets
4. **Design Patterns**: Singleton, factory, observer, strategy, repository
5. **Best Practices**: SOLID principles, performance optimization, memory management
6. **Interview Preparation**: Questions ranging from beginner to expert level

### Key Takeaways

- Dart is a purely object-oriented language
- Everything in Dart is an object, including primit