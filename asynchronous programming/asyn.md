# Flutter Asynchronous Programming: Complete Technical Guide

## Table of Contents

- [Introduction](#introduction)
- [Core Concepts](#core-concepts)
- [The Event Loop Architecture](#the-event-loop-architecture)
- [Future Deep Dive](#future-deep-dive)
- [Async/Await Mechanics](#asyncawait-mechanics)
- [Stream Programming](#stream-programming)
- [Error Handling Patterns](#error-handling-patterns)
- [Advanced Patterns](#advanced-patterns)
- [Performance Considerations](#performance-considerations)
- [Best Practices](#best-practices)
- [Common Pitfalls](#common-pitfalls)

---

## Introduction

Asynchronous programming in Flutter enables developers to build responsive, high-performance applications by managing time-consuming operations without blocking the main UI thread. This comprehensive guide explores the internal mechanics, patterns, and best practices for mastering async programming in Dart/Flutter.

### Why Asynchronous Programming Matters

Flutter apps run on a **single-threaded event loop**. Without asynchronous programming:
- Network requests would freeze the UI
- File I/O operations would block user interactions
- Database queries would cause app stuttering
- The user experience would be severely degraded

### Key Benefits

1. **Improved Performance**: Heavy tasks run in the background while the UI remains responsive
2. **Efficient Resource Utilization**: Concurrent task handling optimizes CPU and memory usage
3. **Better User Experience**: Smooth animations and interactions even during data processing
4. **Scalability**: Handle multiple operations simultaneously without thread management complexity

---

## Core Concepts

### Synchronous vs Asynchronous Execution

#### Synchronous Programming

```dart
void synchronousExample() {
  print('Start');
  
  // This blocks execution for 3 seconds
  sleep(Duration(seconds: 3));
  
  print('End'); // Only prints after 3 seconds
}
```

**Characteristics:**
- Sequential execution
- Blocking operations
- Predictable flow
- Simple to reason about
- **Problem**: UI freezes during long operations

#### Asynchronous Programming

```dart
Future<void> asynchronousExample() async {
  print('Start');
  
  // This doesn't block - execution continues immediately
  await Future.delayed(Duration(seconds: 3));
  
  print('End'); // Prints after 3 seconds, but UI remains responsive
}
```

**Characteristics:**
- Non-blocking execution
- Concurrent operations
- Event-driven
- Requires careful state management
- **Advantage**: UI stays responsive

### Key Terms

| Term | Definition |
|------|------------|
| **Synchronous Operation** | Blocks other operations until completion |
| **Asynchronous Operation** | Allows other operations to execute before completion |
| **Future** | A promise of a value or error that will be available later |
| **Stream** | A sequence of asynchronous events over time |
| **Event Loop** | The mechanism that processes asynchronous events |
| **Microtask Queue** | High-priority queue for internal Dart operations |
| **Event Queue** | Queue for external events (I/O, timers, user input) |

---

## The Event Loop Architecture

### How Dart's Event Loop Works

Dart uses a single-threaded execution model with an event loop that processes tasks from two queues:

```
┌─────────────────────────────────────┐
│         Dart Event Loop             │
├─────────────────────────────────────┤
│                                     │
│  1. Execute synchronous code        │
│                                     │
│  2. Process Microtask Queue         │
│     (scheduleMicrotask)             │
│                                     │
│  3. Process Event Queue              │
│     (Futures, I/O, Timers)          │
│                                     │
│  4. Repeat                          │
│                                     │
└─────────────────────────────────────┘
```

### Execution Order Example

```dart
void main() {
  print('1. Synchronous');
  
  // Event Queue
  Future(() => print('4. Event Queue Future'));
  
  // Microtask Queue (higher priority)
  scheduleMicrotask(() => print('3. Microtask'));
  
  print('2. Synchronous');
}

// Output:
// 1. Synchronous
// 2. Synchronous
// 3. Microtask
// 4. Event Queue Future
```

### Priority Levels

1. **Synchronous code**: Executes immediately
2. **Microtask queue**: High priority, processed before event queue
3. **Event queue**: Normal priority for Futures, I/O, timers

---

## Future Deep Dive

### What is a Future?

A `Future<T>` represents a potential value (or error) of type `T` that will be available at some point in the future.

### Future States

```dart
// State 1: UNCOMPLETED
Future<String> future = fetchData();

// State 2: COMPLETED WITH VALUE
future.then((value) => print(value));

// State 2 (alternative): COMPLETED WITH ERROR
future.catchError((error) => print(error));
```

### Creating Futures

#### Method 1: Future Constructor (Event Queue)

```dart
Future<String> createFuture() {
  return Future(() {
    // Executes in Event Queue
    return 'Data loaded';
  });
}
```

#### Method 2: Future.microtask (Microtask Queue - Higher Priority)

```dart
Future<String> microtaskFuture() {
  return Future.microtask(() {
    // Executes in Microtask Queue (before Event Queue)
    return 'Microtask result';
  });
}

// Example showing priority
void demonstratePriority() {
  Future(() => print('1. Event Queue'));
  Future.microtask(() => print('2. Microtask Queue (runs first)'));
  Future(() => print('3. Event Queue'));
  
  // Output:
  // 2. Microtask Queue (runs first)
  // 1. Event Queue
  // 3. Event Queue
}
```

#### Method 3: Future.delayed

```dart
Future<String> delayedFuture() {
  return Future.delayed(
    Duration(seconds: 2),
    () => 'Delayed result'
  );
}

// Without callback
Future<void> justDelay() {
  return Future.delayed(Duration(seconds: 2));
}
```

#### Method 4: Future.value (Immediate completion as microtask)

```dart
Future<String> immediateFuture() {
  return Future.value('Immediate result');
}

// This completes as a microtask, not synchronously
void demonstrateValue() {
  print('1. Start');
  Future.value('Data').then((value) => print('3. Future.value: $value'));
  print('2. Synchronous');
  
  // Output:
  // 1. Start
  // 2. Synchronous
  // 3. Future.value: Data
}
```

#### Method 5: Future.sync (Synchronous execution)

```dart
Future<String> syncFuture() {
  return Future.sync(() {
    // Executes SYNCHRONOUSLY, returns Future
    return 'Sync result';
  });
}

void demonstrateSync() {
  print('1. Start');
  Future.sync(() => print('2. Sync execution'));
  print('3. After sync');
  
  // Output:
  // 1. Start
  // 2. Sync execution
  // 3. After sync
}
```

#### Method 6: Future.error

```dart
Future<String> errorFuture() {
  return Future.error('Something went wrong');
}

// With stack trace
Future<String> errorWithStack() {
  return Future.error(
    'Error occurred',
    StackTrace.current,
  );
}
```

#### Method 7: Completer (Manual control)

```dart
Future<String> manualFuture() {
  final completer = Completer<String>();
  
  // Simulate async work
  Timer(Duration(seconds: 1), () {
    completer.complete('Manual completion');
  });
  
  return completer.future;
}

// Completer with error
Future<String> completerWithError() {
  final completer = Completer<String>();
  
  Timer(Duration(seconds: 1), () {
    if (someCondition) {
      completer.complete('Success');
    } else {
      completer.completeError('Failed');
    }
  });
  
  return completer.future;
}
```

#### Method 8: Future.doWhile (Loop until condition)

```dart
Future<void> processUntilDone() {
  var count = 0;
  
  return Future.doWhile(() async {
    count++;
    print('Processing: $count');
    await Future.delayed(Duration(milliseconds: 100));
    return count < 5; // Continue while true
  });
}
```

#### Method 9: Future.forEach (Sequential iteration)

```dart
Future<void> processSequentially(List<int> items) {
  return Future.forEach(items, (item) async {
    await processItem(item);
    print('Processed: $item');
  });
}
```

### Working with Futures

#### Using .then() and .catchError()

```dart
void fetchUserData() {
  getUserFromApi()
    .then((user) {
      print('User: ${user.name}');
      return getUserPosts(user.id);
    })
    .then((posts) {
      print('Posts: ${posts.length}');
    })
    .catchError((error) {
      print('Error: $error');
    })
    .whenComplete(() {
      print('Operation complete');
    });
}
```

#### Chaining Futures

```dart
Future<String> processData() {
  return fetchData()
    .then((data) => parseData(data))
    .then((parsed) => validateData(parsed))
    .then((validated) => saveData(validated))
    .then((_) => 'Success');
}
```

### Multiple Futures

#### Future.wait (Parallel execution)

```dart
Future<void> loadMultipleResources() async {
  try {
    final results = await Future.wait([
      fetchUsers(),
      fetchPosts(),
      fetchComments(),
    ]);
    
    final users = results[0];
    final posts = results[1];
    final comments = results[2];
    
    print('All loaded: $users, $posts, $comments');
  } catch (e) {
    // If ANY future fails, this catches the error
    print('Error loading resources: $e');
  }
}
```

#### Future.wait with Error Handling

```dart
Future<void> loadWithIndividualErrors() async {
  final results = await Future.wait(
    [
      fetchUsers(),
      fetchPosts(),
      fetchComments(),
    ],
    eagerError: false, // Don't stop on first error
  );
  
  // Process results even if some failed
}
```

#### Record-based Wait (Different types)

```dart
Future<void> loadDifferentTypes() async {
  try {
    final (users, count, isValid) = await (
      fetchUsers(),      // Future<List<User>>
      getUserCount(),    // Future<int>
      validateSystem(),  // Future<bool>
    ).wait;
    
    print('Users: $users, Count: $count, Valid: $isValid');
  } on ParallelWaitError catch (e) {
    print('Values: ${e.values}');
    print('Errors: ${e.errors}');
  }
}
```

#### Future.any (First to complete)

```dart
Future<String> fastestServer() async {
  return await Future.any([
    fetchFromServer1(),
    fetchFromServer2(),
    fetchFromServer3(),
  ]);
}
```

### Advanced Future Methods

#### Future.whenComplete (Finally equivalent)

```dart
Future<void> fetchWithCleanup() async {
  try {
    await fetchData();
  } catch (e) {
    handleError(e);
  } finally {
    cleanup();
  }
}

// Using whenComplete
Future<void> fetchWithWhenComplete() {
  return fetchData()
    .then((data) => processData(data))
    .catchError((e) => handleError(e))
    .whenComplete(() => cleanup());
}
```

#### Future.asStream (Convert to Stream)

```dart
Stream<String> futureToStream() {
  return Future.value('Single value').asStream();
}

// Usage
await for (final value in futureToStream()) {
  print(value); // Prints once
}
```

#### Future.timeout with onTimeout callback

```dart
Future<String> fetchWithTimeout() {
  return fetchData().timeout(
    Duration(seconds: 5),
    onTimeout: () {
      print('Timeout occurred, returning default');
      return 'Default value';
    },
  );
}

// Without onTimeout (throws TimeoutException)
Future<String> fetchWithTimeoutError() {
  return fetchData().timeout(Duration(seconds: 5));
}
```

### scheduleMicrotask Function

```dart
import 'dart:async';

void demonstrateMicrotask() {
  print('1. Synchronous');
  
  scheduleMicrotask(() {
    print('3. Microtask 1');
  });
  
  scheduleMicrotask(() {
    print('4. Microtask 2');
  });
  
  Future(() => print('5. Event Queue'));
  
  print('2. Synchronous');
  
  // Output:
  // 1. Synchronous
  // 2. Synchronous
  // 3. Microtask 1
  // 4. Microtask 2
  // 5. Event Queue
}

// Use case: Schedule work without blocking
void performWork() {
  // Do some synchronous work
  processDataSync();
  
  // Schedule remaining work as microtask
  scheduleMicrotask(() {
    finalizeProcessing();
  });
  
  // Function returns immediately
}
```

### Zone API for Async Context

```dart
// Creating isolated async context
void demonstrateZones() {
  runZoned(() {
    // All async operations in this zone
    Future(() => print('In custom zone'));
    
    throw Exception('Caught by zone');
  }, onError: (error, stack) {
    print('Zone caught error: $error');
  });
}

// Custom error handling for async operations
void customErrorHandling() {
  runZonedGuarded(() {
    // Your app code
    fetchData().then((data) => processData(data));
  }, (error, stack) {
    // All uncaught async errors come here
    logError(error, stack);
  });
}

// Zone values (context propagation)
void zoneValues() {
  final zone = Zone.current.fork(
    zoneValues: {
      #requestId: 'request-123',
      #userId: 'user-456',
    },
  );
  
  zone.run(() {
    final requestId = Zone.current[#requestId];
    print('Request ID: $requestId');
    
    Future(() {
      // Zone values available in async operations
      final userId = Zone.current[#userId];
      print('User ID: $userId');
    });
  });
}
```

---

## Async/Await Mechanics

### How async/await Works Internally

When you mark a function as `async`, Dart transforms it into a state machine that manages execution across asynchronous boundaries.

### Function Transformation

```dart
// What you write:
Future<String> getData() async {
  final result = await fetchData();
  return result.toUpperCase();
}

// Approximately what Dart generates:
Future<String> getData() {
  final completer = Completer<String>();
  
  fetchData().then((result) {
    try {
      final transformed = result.toUpperCase();
      completer.complete(transformed);
    } catch (e) {
      completer.completeError(e);
    }
  }).catchError((e) {
    completer.completeError(e);
  });
  
  return completer.future;
}
```

### Execution Flow

```dart
Future<void> demonstrateFlow() async {
  print('1. Before await - executes immediately');
  
  await Future.delayed(Duration(seconds: 1));
  
  print('2. After await - executes after Future completes');
}

void main() {
  print('A. Main starts');
  demonstrateFlow();
  print('B. Main continues (doesn\'t wait)');
}

// Output:
// A. Main starts
// 1. Before await - executes immediately
// B. Main continues (doesn't wait)
// 2. After await - executes after Future completes
```

### Async Function Return Types

```dart
// Returns Future<String>
Future<String> getString() async {
  return 'Hello';
}

// Returns Future<int>
Future<int> getNumber() async {
  return 42;
}

// Returns Future<void> (no useful value)
Future<void> doSomething() async {
  print('Doing something');
}

// Returns Future<dynamic>
Future getDynamic() async {
  return 'Could be anything';
}
```

### Await Rules

```dart
// ✅ CORRECT: await in async function
Future<void> correct() async {
  await fetchData();
}

// ❌ WRONG: await in non-async function
void wrong() {
  await fetchData(); // Compile error!
}

// ✅ CORRECT: async in main
Future<void> main() async {
  await fetchData();
}

// ⚠️ ALLOWED but risky: async without await
Future<void> noAwait() async {
  fetchData(); // Starts but doesn't wait
  print('Continues immediately');
}
```

### Multiple Awaits

```dart
Future<void> sequentialOperations() async {
  print('Starting...');
  
  // These execute SEQUENTIALLY
  final user = await fetchUser();
  print('Got user: ${user.name}');
  
  final posts = await fetchPosts(user.id);
  print('Got posts: ${posts.length}');
  
  final comments = await fetchComments(posts.first.id);
  print('Got comments: ${comments.length}');
}

Future<void> parallelOperations() async {
  print('Starting...');
  
  // These execute IN PARALLEL
  final results = await Future.wait([
    fetchUser(),
    fetchPosts(123),
    fetchComments(456),
  ]);
  
  print('All done!');
}
```

### Async Callbacks

```dart
void setupButton() {
  // ✅ Async callback
  button.onClick.listen((event) async {
    final data = await fetchData();
    updateUI(data);
  });
}

// ✅ Async in map
Future<void> processItems(List<int> ids) async {
  final futures = ids.map((id) async {
    return await fetchItem(id);
  });
  
  final items = await Future.wait(futures);
}
```

---

## Stream Programming

### What is a Stream?

A `Stream<T>` is a sequence of asynchronous events that delivers data of type `T` over time.

```
Time →
Stream: [event1]---[event2]------[event3]---[done/error]
```

### Stream vs Future

| Feature | Future | Stream |
|---------|--------|--------|
| Values | Single value | Multiple values |
| Completion | Completes once | Can emit multiple times |
| Use case | One-time async operation | Ongoing async events |
| Example | HTTP request | WebSocket connection |

### Creating Streams

#### Method 1: Stream.periodic

```dart
Stream<int> counterStream() {
  return Stream.periodic(
    Duration(seconds: 1),
    (count) => count,
  ).take(5);
}

// Emits: 0, 1, 2, 3, 4
```

#### Method 2: Stream.fromIterable

```dart
Stream<String> namesStream() {
  return Stream.fromIterable(['Alice', 'Bob', 'Charlie']);
}
```

#### Method 3: Stream.fromFuture

```dart
Stream<String> dataStream() {
  return Stream.fromFuture(fetchData());
}
```

#### Method 4: StreamController (Manual control)

```dart
class DataManager {
  final _controller = StreamController<String>();
  
  Stream<String> get dataStream => _controller.stream;
  
  void addData(String data) {
    _controller.add(data);
  }
  
  void addError(Object error) {
    _controller.addError(error);
  }
  
  void close() {
    _controller.close();
  }
}
```

#### Method 5: async* Generator

```dart
Stream<int> countStream(int max) async* {
  for (int i = 0; i < max; i++) {
    await Future.delayed(Duration(seconds: 1));
    yield i; // Emit value
  }
}
```

### Listening to Streams

#### Method 1: await for Loop

```dart
Future<void> processStream() async {
  final stream = countStream(5);
  
  await for (final value in stream) {
    print('Received: $value');
  }
  
  print('Stream complete');
}
```

#### Method 2: listen() Method

```dart
void setupStreamListener() {
  final stream = dataStream();
  
  stream.listen(
    (data) {
      print('Data: $data');
    },
    onError: (error) {
      print('Error: $error');
    },
    onDone: () {
      print('Stream closed');
    },
    cancelOnError: false,
  );
}
```

### Stream Types

#### Single-Subscription Streams

```dart
// Can only be listened to ONCE
final stream = Stream.fromIterable([1, 2, 3]);

stream.listen((value) => print('Listener 1: $value'));

// ❌ Error: Stream has already been listened to
stream.listen((value) => print('Listener 2: $value'));
```

#### Broadcast Streams

```dart
// Can have MULTIPLE listeners
final controller = StreamController<int>.broadcast();
final stream = controller.stream;

stream.listen((value) => print('Listener 1: $value'));
stream.listen((value) => print('Listener 2: $value'));

controller.add(42); // Both listeners receive it
```

#### Converting Single to Broadcast

```dart
Stream<int> singleStream = getSingleSubscriptionStream();
Stream<int> broadcastStream = singleStream.asBroadcastStream();

// Now multiple listeners can subscribe
broadcastStream.listen((value) => print('Listener 1: $value'));
broadcastStream.listen((value) => print('Listener 2: $value'));
```

### Stream Transformations

#### map()

```dart
Stream<String> upperCaseStream(Stream<String> input) {
  return input.map((str) => str.toUpperCase());
}
```

#### where()

```dart
Stream<int> evenNumbers(Stream<int> input) {
  return input.where((num) => num % 2 == 0);
}
```

#### transform()

```dart
Stream<List<int>> fileStream = file.openRead();

Stream<String> lines = fileStream
  .transform(utf8.decoder)
  .transform(LineSplitter());
```

#### asyncMap()

```dart
Stream<User> enrichUsers(Stream<int> userIds) {
  return userIds.asyncMap((id) async {
    return await fetchUserDetails(id);
  });
}
```

#### asyncExpand() - flatMap equivalent

```dart
Stream<Post> userPosts(Stream<User> users) {
  return users.asyncExpand((user) async* {
    // Returns a stream for each user
    final posts = await fetchUserPosts(user.id);
    for (var post in posts) {
      yield post;
    }
  });
}

// Another example
Stream<int> expandNumbers(Stream<int> numbers) {
  return numbers.asyncExpand((n) async* {
    yield n;
    yield n * 2;
    yield n * 3;
  });
}

// Input: 1, 2, 3
// Output: 1, 2, 3, 2, 4, 6, 3, 6, 9
```

#### expand() - Synchronous version

```dart
Stream<int> expandSync(Stream<int> numbers) {
  return numbers.expand((n) => [n, n * 2, n * 3]);
}
```

#### take() and skip()

```dart
Stream<int> first5(Stream<int> input) {
  return input.take(5);
}

Stream<int> skipFirst3(Stream<int> input) {
  return input.skip(3);
}
```

#### takeWhile() and skipWhile()

```dart
Stream<int> takeWhileLessThan10(Stream<int> input) {
  return input.takeWhile((value) => value < 10);
}

Stream<int> skipWhileLessThan10(Stream<int> input) {
  return input.skipWhile((value) => value < 10);
}
```

#### distinct()

```dart
Stream<int> uniqueValues(Stream<int> input) {
  return input.distinct();
}

// With equality comparison
Stream<User> distinctUsers(Stream<User> input) {
  return input.distinct((u1, u2) => u1.id == u2.id);
}
```

#### handleError()

```dart
Stream<int> safeStream(Stream<int> input) {
  return input.handleError(
    (error) => print('Error: $error'),
    test: (error) => error is FormatException,
  );
}
```

#### timeout()

```dart
Stream<int> timeoutStream(Stream<int> input) {
  return input.timeout(
    Duration(seconds: 5),
    onTimeout: (sink) {
      sink.add(-1); // Default value
      sink.close();
    },
  );
}
```

### Advanced Stream Methods

#### Stream.periodic with take

```dart
Stream<DateTime> clockStream() {
  return Stream.periodic(
    Duration(seconds: 1),
    (_) => DateTime.now(),
  ).take(10); // Only emit 10 times
}
```

#### Stream.empty and Stream.error

```dart
Stream<int> emptyStream() => Stream.empty();

Stream<int> errorStream() => Stream.error('Something went wrong');
```

#### Stream.eventTransformed

```dart
Stream<String> customTransform(Stream<int> input) {
  return input.transform(
    StreamTransformer.fromHandlers(
      handleData: (value, sink) {
        sink.add('Value: $value');
      },
      handleError: (error, trace, sink) {
        sink.addError('Caught: $error');
      },
      handleDone: (sink) {
        sink.close();
      },
    ),
  );
}
```

#### Custom StreamTransformer

```dart
class DuplicateTransformer<T> extends StreamTransformerBase<T, T> {
  @override
  Stream<T> bind(Stream<T> stream) {
    return stream.asyncExpand((value) async* {
      yield value;
      yield value; // Duplicate each value
    });
  }
}

// Usage
final stream = Stream.fromIterable([1, 2, 3]);
final duplicated = stream.transform(DuplicateTransformer<int>());
// Output: 1, 1, 2, 2, 3, 3
```

#### Stream.multi (Multiple subscribers with different behavior)

```dart
Stream<int> createMultiStream() {
  return Stream.multi((controller) {
    var count = 0;
    Timer.periodic(Duration(seconds: 1), (timer) {
      if (count >= 5) {
        controller.close();
        timer.cancel();
      } else {
        controller.add(count++);
      }
    });
  });
}

// Each listener gets independent stream
final stream = createMultiStream();
stream.listen((value) => print('Listener 1: $value'));
stream.listen((value) => print('Listener 2: $value'));
```

### Stream Manipulation

#### Combining Streams

```dart
// Merge multiple streams (interleaved)
import 'package:async/async.dart';

Stream<int> mergedStream = StreamGroup.merge([
  stream1,
  stream2,
  stream3,
]);

// Concatenate streams (one after another)
Stream<int> concatenated = stream1.followedBy(stream2);

// Zip streams (combine corresponding elements)
Stream<List<int>> zipped = StreamZip([stream1, stream2]);
```

#### Stream.first, last, single, length

```dart
Future<void> getStreamValues() async {
  final stream = Stream.fromIterable([1, 2, 3, 4, 5]);
  
  final first = await stream.first;  // 1
  final last = await stream.last;    // 5
  final length = await stream.length; // 5
  
  // Single - throws if stream doesn't have exactly one element
  final singleStream = Stream.value(42);
  final single = await singleStream.single; // 42
}

// firstWhere, lastWhere, singleWhere
Future<void> conditionalStreamValues() async {
  final stream = Stream.fromIterable([1, 2, 3, 4, 5]);
  
  final firstEven = await stream.firstWhere((n) => n % 2 == 0); // 2
  final lastOdd = await stream.lastWhere((n) => n % 2 == 1);   // 5
  
  // With orElse
  final result = await stream.firstWhere(
    (n) => n > 10,
    orElse: () => -1, // Returns -1 if not found
  );
}
```

#### Stream.contains, every, any

```dart
Future<void> streamPredicates() async {
  final stream = Stream.fromIterable([1, 2, 3, 4, 5]);
  
  final contains3 = await stream.contains(3); // true
  final allPositive = await stream.every((n) => n > 0); // true
  final hasEven = await stream.any((n) => n % 2 == 0); // true
}
```

#### Stream.reduce, fold

```dart
Future<void> streamAggregation() async {
  final stream = Stream.fromIterable([1, 2, 3, 4, 5]);
  
  // reduce - combines elements (requires at least one element)
  final sum = await stream.reduce((a, b) => a + b); // 15
  
  // fold - with initial value
  final stream2 = Stream.fromIterable([1, 2, 3, 4, 5]);
  final product = await stream2.fold(1, (prev, element) => prev * element); // 120
  
  // fold with different type
  final stream3 = Stream.fromIterable(['a', 'b', 'c']);
  final result = await stream3.fold<String>('', (prev, element) => prev + element); // 'abc'
}
```

#### Stream.join

```dart
Future<void> joinStream() async {
  final stream = Stream.fromIterable(['Hello', 'World', 'Dart']);
  final joined = await stream.join(' '); // 'Hello World Dart'
  print(joined);
}
```

#### Converting Stream to List/Set

```dart
Future<List<int>> streamToList(Stream<int> stream) {
  return stream.toList();
}

Future<Set<int>> streamToSet(Stream<int> stream) {
  return stream.toSet();
}
```

#### Stream.pipe (Forward to StreamConsumer)

```dart
Future<void> pipeExample() async {
  final file = File('output.txt');
  final stream = Stream.fromIterable(['Line 1\n', 'Line 2\n', 'Line 3\n']);
  
  // Pipe stream to file
  await stream.pipe(file.openWrite());
}
```

### StreamController Advanced Features

#### Sync vs Async Controllers

```dart
// Async controller (default) - events delivered via event queue
final asyncController = StreamController<int>();

// Sync controller - events delivered immediately
final syncController = StreamController<int>(sync: true);

void demonstrateSync() {
  final controller = StreamController<int>(sync: true);
  
  controller.stream.listen((value) {
    print('Received: $value');
  });
  
  print('Before add');
  controller.add(42); // Listener receives immediately
  print('After add');
  
  // Output:
  // Before add
  // Received: 42
  // After add
}
```

#### StreamController callbacks

```dart
StreamController<int> createControllerWithCallbacks() {
  return StreamController<int>(
    onListen: () {
      print('First listener subscribed');
    },
    onPause: () {
      print('All listeners paused');
    },
    onResume: () {
      print('At least one listener resumed');
    },
    onCancel: () {
      print('Last listener cancelled');
    },
  );
}
```

#### Broadcast controller with callbacks

```dart
StreamController<int> createBroadcastController() {
  return StreamController<int>.broadcast(
    onListen: () {
      print('Listener added (may not be first)');
    },
    onCancel: () {
      print('Listener removed (may not be last)');
    },
  );
}
```

#### StreamSink (Write-only interface)

```dart
class DataProcessor {
  final _controller = StreamController<String>();
  
  // Expose only the sink for writing
  StreamSink<String> get input => _controller.sink;
  
  // Expose only the stream for reading
  Stream<String> get output => _controller.stream;
  
  void dispose() {
    _controller.close();
  }
}

// Usage
final processor = DataProcessor();
processor.input.add('Data 1');
processor.input.add('Data 2');

processor.output.listen((data) {
  print('Processed: $data');
});
```

## Isolates for CPU-Intensive Tasks

### What are Isolates?

Isolates are Dart's solution for true parallel execution. Unlike async/await (which is concurrent but not parallel), isolates run on separate threads with their own memory heap.

### When to Use Isolates

- **CPU-intensive computations** (image processing, encryption, parsing large data)
- **Long-running synchronous operations** that would block the UI
- **Parallel processing** of independent tasks

**Don't use isolates for:**
- Network I/O (use async/await instead)
- Simple async operations
- Tasks that require frequent communication with main isolate

### Basic Isolate Usage

```dart
import 'dart:isolate';

// Function to run in isolate (must be top-level or static)
void isolateFunction(SendPort sendPort) {
  // Perform heavy computation
  final result = performHeavyComputation();
  
  // Send result back
  sendPort.send(result);
}

Future<int> runInIsolate() async {
  // Create receive port
  final receivePort = ReceivePort();
  
  // Spawn isolate
  await Isolate.spawn(isolateFunction, receivePort.sendPort);
  
  // Wait for result
  final result = await receivePort.first;
  
  return result as int;
}
```

### Compute Function (Flutter Helper)

```dart
import 'package:flutter/foundation.dart';

// Top-level or static function
int heavyComputation(int value) {
  // Simulate heavy work
  int result = 0;
  for (int i = 0; i < value; i++) {
    result += i;
  }
  return result;
}

// Usage
Future<void> processData() async {
  // Automatically spawns isolate and returns result
  final result = await compute(heavyComputation, 1000000);
  print('Result: $result');
}
```

### Bidirectional Communication

```dart
class IsolateWorker {
  Isolate? _isolate;
  SendPort? _sendPort;
  
  Future<void> init() async {
    final receivePort = ReceivePort();
    
    _isolate = await Isolate.spawn(_isolateEntry, receivePort.sendPort);
    
    _sendPort = await receivePort.first;
  }
  
  Future<dynamic> sendMessage(dynamic message) async {
    final responsePort = ReceivePort();
    
    _sendPort!.send({
      'message': message,
      'responsePort': responsePort.sendPort,
    });
    
    return await responsePort.first;
  }
  
  void dispose() {
    _isolate?.kill(priority: Isolate.immediate);
  }
  
  static void _isolateEntry(SendPort sendPort) {
    final receivePort = ReceivePort();
    sendPort.send(receivePort.sendPort);
    
    receivePort.listen((data) {
      final message = data['message'];
      final responsePort = data['responsePort'] as SendPort;
      
      // Process message
      final result = processMessage(message);
      
      // Send response
      responsePort.send(result);
    });
  }
  
  static dynamic processMessage(dynamic message) {
    // Heavy processing
    return 'Processed: $message';
  }
}

// Usage
final worker = IsolateWorker();
await worker.init();
final result = await worker.sendMessage('Hello');
print(result);
worker.dispose();
```

### Image Processing Example

```dart
import 'dart:typed_data';
import 'dart:ui' as ui;
import 'package:flutter/foundation.dart';

class ImageData {
  final Uint8List bytes;
  final int width;
  final int height;
  
  ImageData(this.bytes, this.width, this.height);
}

// Isolate function for image processing
Uint8List processImage(ImageData data) {
  final pixels = data.bytes;
  
  // Apply grayscale filter
  for (int i = 0; i < pixels.length; i += 4) {
    final gray = (pixels[i] * 0.299 + 
                  pixels[i + 1] * 0.587 + 
                  pixels[i + 2] * 0.114).round();
    
    pixels[i] = gray;
    pixels[i + 1] = gray;
    pixels[i + 2] = gray;
  }
  
  return pixels;
}

// Usage in Flutter
Future<ui.Image> applyGrayscaleFilter(ui.Image image) async {
  final bytes = await image.toByteData(format: ui.ImageByteFormat.rawRgba);
  
  final imageData = ImageData(
    bytes!.buffer.asUint8List(),
    image.width,
    image.height,
  );
  
  // Process in isolate
  final processedBytes = await compute(processImage, imageData);
  
  // Convert back to image
  final codec = await ui.instantiateImageCodec(processedBytes);
  final frame = await codec.getNextFrame();
  
  return frame.image;
}
```

### JSON Parsing in Isolate

```dart
import 'dart:convert';

// Parse large JSON in isolate
List<User> parseUsers(String jsonString) {
  final List<dynamic> jsonList = json.decode(jsonString);
  return jsonList.map((json) => User.fromJson(json)).toList();
}

// Usage
Future<List<User>> loadUsers(String jsonString) async {
  return await compute(parseUsers, jsonString);
}
```

### Isolate Pool Pattern

```dart
class IsolatePool {
  final int size;
  final List<IsolateWorker> _workers = [];
  int _currentIndex = 0;
  
  IsolatePool({required this.size});
  
  Future<void> init() async {
    for (int i = 0; i < size; i++) {
      final worker = IsolateWorker();
      await worker.init();
      _workers.add(worker);
    }
  }
  
  Future<dynamic> execute(dynamic message) async {
    final worker = _workers[_currentIndex];
    _currentIndex = (_currentIndex + 1) % size;
    
    return await worker.sendMessage(message);
  }
  
  void dispose() {
    for (var worker in _workers) {
      worker.dispose();
    }
    _workers.clear();
  }
}

// Usage
final pool = IsolatePool(size: 4);
await pool.init();

// Execute tasks in parallel across pool
final results = await Future.wait([
  pool.execute('Task 1'),
  pool.execute('Task 2'),
  pool.execute('Task 3'),
  pool.execute('Task 4'),
]);

pool.dispose();
```

### Platform-Specific Isolates

```dart
import 'dart:io' show Platform;

Future<int> heavyTask(int input) async {
  if (Platform.isAndroid || Platform.isIOS) {
    // Use compute for mobile
    return await compute(_heavyComputation, input);
  } else {
    // Direct execution for web (no isolate support)
    return _heavyComputation(input);
  }
}

int _heavyComputation(int input) {
  // Heavy work
  return input * 2;
}
```

---

## Complementary Topics

### Timer and Periodic

```dart
import 'dart:async';

// One-time timer
void scheduleTask() {
  Timer(Duration(seconds: 5), () {
    print('Task executed after 5 seconds');
  });
}

// Periodic timer
void startPeriodicTask() {
  Timer.periodic(Duration(seconds: 1), (timer) {
    print('Tick: ${timer.tick}');
    
    if (timer.tick >= 10) {
      timer.cancel();
    }
  });
}

// Timer in async function
Future<void> delayedTask() async {
  print('Starting...');
  await Future.delayed(Duration(seconds: 2));
  print('Done!');
}
```

### Synchronous Iterables vs Asynchronous Streams

```dart
// Synchronous iterable
Iterable<int> generateSync(int max) sync* {
  for (int i = 0; i < max; i++) {
    yield i;
  }
}

// Asynchronous stream
Stream<int> generateAsync(int max) async* {
  for (int i = 0; i < max; i++) {
    await Future.delayed(Duration(milliseconds: 100));
    yield i;
  }
}

// Usage
void main() async {
  // Synchronous - all values available immediately
  for (var value in generateSync(5)) {
    print('Sync: $value');
  }
  
  // Asynchronous - values arrive over time
  await for (var value in generateAsync(5)) {
    print('Async: $value');
  }
}
```

### yield vs yield* in Generators

```dart
// yield - emit single value
Stream<int> simpleGenerator() async* {
  yield 1;
  yield 2;
  yield 3;
}

// yield* - emit all values from another stream/iterable
Stream<int> delegatingGenerator() async* {
  yield* Stream.fromIterable([1, 2, 3]);
  yield* simpleGenerator();
  yield 4;
}

// Output: 1, 2, 3, 1, 2, 3, 4

// Recursive generator
Stream<int> countdown(int n) async* {
  if (n > 0) {
    yield n;
    yield* countdown(n - 1);
  }
}
```

### Async* with Error Handling

```dart
Stream<int> generateWithErrors() async* {
  yield 1;
  yield 2;
  
  throw Exception('Error in stream');
  
  yield 3; // Never reached
}

// Handling errors
Future<void> consumeStream() async {
  try {
    await for (var value in generateWithErrors()) {
      print('Value: $value');
    }
  } catch (e) {
    print('Caught: $e');
  }
}
```

### Future.wait with cleanup (eagerError)

```dart
Future<void> parallelWithCleanup() async {
  try {
    await Future.wait(
      [
        task1(),
        task2(),
        task3(),
      ],
      eagerError: true, // Stop on first error
    );
  } catch (e) {
    print('One task failed: $e');
    // Other tasks continue running in background
  }
}

// Better: Cancel remaining tasks
Future<void> parallelWithCancellation() async {
  final futures = [
    task1(),
    task2(),
    task3(),
  ];
  
  try {
    await Future.wait(futures);
  } catch (e) {
    // Cancel remaining tasks if possible
    print('Failed: $e');
  }
}
```

### Future Error Handling

#### Method 1: try-catch with async/await

```dart
Future<void> fetchDataSafely() async {
  try {
    final data = await fetchData();
    print('Success: $data');
  } catch (e) {
    print('Error: $e');
  } finally {
    print('Cleanup');
  }
}
```

#### Method 2: catchError()

```dart
void fetchDataWithCatchError() {
  fetchData()
    .then((data) => print('Success: $data'))
    .catchError((error) => print('Error: $error'))
    .whenComplete(() => print('Cleanup'));
}
```

#### Method 3: Specific Error Types

```dart
Future<void> handleSpecificErrors() async {
  try {
    await fetchData();
  } on NetworkException catch (e) {
    print('Network error: ${e.message}');
  } on TimeoutException catch (e) {
    print('Timeout: ${e.duration}');
  } on FormatException catch (e) {
    print('Format error: ${e.message}');
  } catch (e, stackTrace) {
    print('Unknown error: $e');
    print('Stack trace: $stackTrace');
  }
}
```

### Stream Error Handling

#### Method 1: try-catch with await for

```dart
Future<void> processStreamSafely() async {
  try {
    await for (final value in dataStream()) {
      print('Value: $value');
    }
  } catch (e) {
    print('Stream error: $e');
  }
}
```

#### Method 2: onError in listen()

```dart
void handleStreamErrors() {
  dataStream().listen(
    (data) => print('Data: $data'),
    onError: (error) => print('Error: $error'),
    cancelOnError: false, // Continue after error
  );
}
```

#### Method 3: handleError()

```dart
Stream<int> safeStream() {
  return dataStream().handleError((error) {
    print('Handled: $error');
    // Can return default value or rethrow
  });
}
```

### Error Recovery Patterns

#### Retry Logic

```dart
Future<T> retry<T>(
  Future<T> Function() operation, {
  int maxAttempts = 3,
  Duration delay = const Duration(seconds: 1),
}) async {
  int attempts = 0;
  
  while (true) {
    try {
      return await operation();
    } catch (e) {
      attempts++;
      if (attempts >= maxAttempts) rethrow;
      
      print('Attempt $attempts failed, retrying...');
      await Future.delayed(delay);
    }
  }
}

// Usage
final data = await retry(() => fetchData(), maxAttempts: 5);
```

#### Timeout Handling

```dart
Future<String> fetchWithTimeout() async {
  try {
    return await fetchData().timeout(
      Duration(seconds: 5),
      onTimeout: () => throw TimeoutException('Request took too long'),
    );
  } catch (e) {
    print('Error: $e');
    return 'Default value';
  }
}
```

#### Fallback Values

```dart
Future<String> fetchWithFallback() async {
  try {
    return await fetchData();
  } catch (e) {
    print('Failed to fetch, using cache');
    return await getCachedData();
  }
}
```

---

## Advanced Patterns

### Completer Pattern

```dart
class AsyncQueue {
  final _queue = <Completer<void>>[];
  
  Future<void> add(Future<void> Function() operation) async {
    final completer = Completer<void>();
    _queue.add(completer);
    
    if (_queue.length == 1) {
      _processQueue();
    }
    
    return completer.future;
  }
  
  Future<void> _processQueue() async {
    while (_queue.isNotEmpty) {
      final completer = _queue.first;
      
      try {
        await operation();
        completer.complete();
      } catch (e) {
        completer.completeError(e);
      }
      
      _queue.removeAt(0);
    }
  }
}
```

### StreamController Advanced Usage

```dart
class RealtimeDataManager {
  final _controller = StreamController<Data>.broadcast();
  Timer? _timer;
  
  Stream<Data> get dataStream => _controller.stream;
  
  void start() {
    _timer = Timer.periodic(Duration(seconds: 1), (_) {
      if (!_controller.isClosed) {
        _controller.add(fetchRealtimeData());
      }
    });
  }
  
  void stop() {
    _timer?.cancel();
    _controller.close();
  }
  
  void dispose() {
    stop();
  }
}
```

### Stream Subscription Management

```dart
class DataWidget extends StatefulWidget {
  @override
  _DataWidgetState createState() => _DataWidgetState();
}

class _DataWidgetState extends State<DataWidget> {
  StreamSubscription<Data>? _subscription;
  
  @override
  void initState() {
    super.initState();
    _subscription = dataStream.listen(
      (data) => setState(() => _data = data),
      onError: (error) => print('Error: $error'),
    );
  }
  
  @override
  void dispose() {
    _subscription?.cancel();
    super.dispose();
  }
  
  @override
  Widget build(BuildContext context) {
    return Text(_data.toString());
  }
}
```

### Debouncing and Throttling

```dart
class Debouncer {
  final Duration delay;
  Timer? _timer;
  
  Debouncer({required this.delay});
  
  void call(void Function() action) {
    _timer?.cancel();
    _timer = Timer(delay, action);
  }
  
  void dispose() {
    _timer?.cancel();
  }
}

// Usage
final debouncer = Debouncer(delay: Duration(milliseconds: 500));

void onSearchChanged(String query) {
  debouncer(() {
    performSearch(query);
  });
}
```

### Stream Throttling

```dart
extension ThrottleExtension<T> on Stream<T> {
  Stream<T> throttle(Duration duration) {
    DateTime? lastEmit;
    
    return where((event) {
      final now = DateTime.now();
      if (lastEmit == null || now.difference(lastEmit!) >= duration) {
        lastEmit = now;
        return true;
      }
      return false;
    });
  }
}

// Usage
stream.throttle(Duration(milliseconds: 500)).listen((data) {
  print('Throttled data: $data');
});
```

### Async Initialization Pattern

```dart
class DatabaseService {
  static DatabaseService? _instance;
  static Future<DatabaseService>? _initFuture;
  
  Database? _database;
  
  DatabaseService._();
  
  static Future<DatabaseService> getInstance() {
    _initFuture ??= _initialize();
    return _initFuture!;
  }
  
  static Future<DatabaseService> _initialize() async {
    final instance = DatabaseService._();
    instance._database = await openDatabase('app.db');
    _instance = instance;
    return instance;
  }
  
  Database get database => _database!;
}

// Usage
final db = await DatabaseService.getInstance();
```

---

## Performance Considerations

### Avoiding Common Performance Issues

#### Issue 1: Unnecessary Async Functions

```dart
// ❌ BAD: Unnecessary async wrapper
Future<int> calculate() async {
  return 42;
}

// ✅ GOOD: Return value directly
int calculate() {
  return 42;
}

// ✅ GOOD: Only use async when needed
Future<int> fetchAndCalculate() async {
  final data = await fetchData();
  return data * 2;
}
```

#### Issue 2: Sequential vs Parallel Execution

```dart
// ❌ BAD: Sequential (slow)
Future<void> loadDataSequential() async {
  final users = await fetchUsers();      // Wait
  final posts = await fetchPosts();      // Wait
  final comments = await fetchComments(); // Wait
  // Total time: ~3 seconds
}

// ✅ GOOD: Parallel (fast)
Future<void> loadDataParallel() async {
  final results = await Future.wait([
    fetchUsers(),
    fetchPosts(),
    fetchComments(),
  ]);
  // Total time: ~1 second
}
```

#### Issue 3: Fire and Forget (Missing await)

```dart
// ❌ BAD: Unawaited future
void saveData() {
  database.save(data); // Fire and forget
  print('Saved!'); // Lies! Might not be saved yet
}

// ✅ GOOD: Await the operation
Future<void> saveData() async {
  await database.save(data);
  print('Saved!'); // Actually saved now
}

// ⚠️ ACCEPTABLE: Intentional fire-and-forget
void logEvent() {
  unawaited(analytics.log(event)); // Explicitly unawaited
}
```

#### Issue 4: Memory Leaks with Streams

```dart
// ❌ BAD: Stream subscription not cancelled
class BadWidget extends StatefulWidget {
  @override
  _BadWidgetState createState() => _BadWidgetState();
}

class _BadWidgetState extends State<BadWidget> {
  @override
  void initState() {
    super.initState();
    dataStream.listen((data) {
      setState(() => _data = data);
    }); // LEAK: Never cancelled
  }
}

// ✅ GOOD: Proper cleanup
class GoodWidget extends StatefulWidget {
  @override
  _GoodWidgetState createState() => _GoodWidgetState();
}

class _GoodWidgetState extends State<GoodWidget> {
  StreamSubscription? _subscription;
  
  @override
  void initState() {
    super.initState();
    _subscription = dataStream.listen((data) {
      setState(() => _data = data);
    });
  }
  
  @override
  void dispose() {
    _subscription?.cancel();
    super.dispose();
  }
}
```

### Optimization Techniques

#### Caching Results

```dart
class CachedDataService {
  String? _cachedData;
  DateTime? _cacheTime;
  final Duration _cacheDuration = Duration(minutes: 5);
  
  Future<String> getData() async {
    if (_cachedData != null && 
        _cacheTime != null &&
        DateTime.now().difference(_cacheTime!) < _cacheDuration) {
      return _cachedData!;
    }
    
    _cachedData = await fetchData();
    _cacheTime = DateTime.now();
    return _cachedData!;
  }
}
```

#### Lazy Loading

```dart
class LazyResource {
  Future<Resource>? _resource;
  
  Future<Resource> get resource {
    _resource ??= _loadResource();
    return _resource!;
  }
  
  Future<Resource> _loadResource() async {
    print('Loading resource...');
    return await expensiveOperation();
  }
}
```

---

## Best Practices

### 1. Always Handle Errors

```dart
// ✅ GOOD
Future<void> fetchData() async {
  try {
    final data = await api.getData();
    processData(data);
  } catch (e) {
    handleError(e);
  }
}
```

### 2. Use Appropriate Return Types

```dart
// ✅ GOOD: Clear intent
Future<User?> findUser(int id) async {
  try {
    return await database.getUser(id);
  } catch (e) {
    return null;
  }
}

// ✅ GOOD: Explicit void for side effects
Future<void> saveSettings(Settings settings) async {
  await database.save(settings);
}
```

### 3. Avoid Mixing async Styles

```dart
// ❌ BAD: Mixing .then() and await
Future<void> badMix() async {
  await fetchData().then((data) {
    return processData(data);
  });
}

// ✅ GOOD: Consistent style
Future<void> goodAsync() async {
  final data = await fetchData();
  await processData(data);
}
```

### 4. Use async/await for Readability

```dart
// ❌ BAD: Callback hell
void callbackHell() {
  fetchUser().then((user) {
    fetchPosts(user.id).then((posts) {
      fetchComments(posts.first.id).then((comments) {
        print('Got comments: $comments');
      });
    });
  });
}

// ✅ GOOD: Linear flow
Future<void> linearFlow() async {
  final user = await fetchUser();
  final posts = await fetchPosts(user.id);
  final comments = await fetchComments(posts.first.id);
  print('Got comments: $comments');
}
```

### 5. Cancel Subscriptions

```dart
// ✅ GOOD: Always dispose
class DataManager {
  StreamSubscription? _subscription;
  
  void start() {
    _subscription = stream.listen(handleData);
  }
  
  void dispose() {
    _subscription?.cancel();
    _subscription = null;
  }
}
```

### 6. Use FutureBuilder and StreamBuilder in Flutter

```dart
// ✅ GOOD: FutureBuilder for one-time async operations
class UserProfile extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return FutureBuilder<User>(
      future: fetchUser(),
      builder: (context, snapshot) {
        if (snapshot.hasError) {
          return ErrorWidget(snapshot.error);
        }
        
        if (!snapshot.hasData) {
          return CircularProgressIndicator();
        }
        
        return UserCard(user: snapshot.data!);
      },
    );
  }
}

// ✅ GOOD: StreamBuilder for continuous updates
class LiveScore extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return StreamBuilder<Score>(
      stream: scoreStream,
      initialData: Score.zero(),
      builder: (context, snapshot) {
        if (snapshot.hasError) {
          return Text('Error: ${snapshot.error}');
        }
        
        final score = snapshot.data!;
        return Text('Score: ${score.value}');
      },
    );
  }
}
```

### 7. Use Linter Rules

Enable these linter rules in `analysis_options.yaml`:

```yaml
linter:
  rules:
    - unawaited_futures
    - discarded_futures
    - avoid_void_async
    - cancel_subscriptions
```

### 8. Document Async Functions

```dart
/// Fetches user data from the API.
///
/// Returns a [User] object on success.
/// Throws [NetworkException] if network fails.
/// Throws [AuthException] if unauthorized.
///
/// Example:
/// ```dart
/// try {
///   final user = await fetchUser(123);
///   print(user.name);
/// } catch (e) {
///   handleError(e);
/// }
/// ```
Future<User> fetchUser(int id) async {
  // Implementation
}
```

### 9. Avoid Blocking Operations

```dart
// ❌ BAD: Blocking operation in async function
Future<void> badAsync() async {
  sleep(Duration(seconds: 5)); // Blocks entire thread!
  await fetchData();
}

// ✅ GOOD: Use Future.delayed
Future<void> goodAsync() async {
  await Future.delayed(Duration(seconds: 5)); // Non-blocking
  await fetchData();
}
```

### 10. Test Async Code Properly

```dart
import 'package:flutter_test/flutter_test.dart';

void main() {
  test('fetchUser returns user data', () async {
    final user = await fetchUser(123);
    expect(user.id, 123);
    expect(user.name, isNotEmpty);
  });
  
  test('fetchUser handles errors', () async {
    expect(
      () => fetchUser(-1),
      throwsA(isA<ArgumentError>()),
    );
  });
  
  test('stream emits correct values', () async {
    final stream = countStream(3);
    expect(stream, emitsInOrder([0, 1, 2, emitsDone]));
  });
}
```

---

## Common Pitfalls

### Pitfall 1: Forgetting async Keyword

```dart
// ❌ WRONG: Returns Future<Future<String>>
Future<String> getData() {
  return fetchData(); // Missing async keyword
}

// ✅ CORRECT
Future<String> getData() async {
  return await fetchData();
}
```

### Pitfall 2: Not Awaiting Futures

```dart
// ❌ BAD: Future not awaited
void saveUser(User user) async {
  database.save(user); // Not awaited!
  print('User saved'); // Might print before save completes
}

// ✅ GOOD
Future<void> saveUser(User user) async {
  await database.save(user);
  print('User saved'); // Prints after save completes
}
```

### Pitfall 3: Using async for Synchronous Code

```dart
// ❌ BAD: Unnecessary async
Future<int> add(int a, int b) async {
  return a + b; // No async operation
}

// ✅ GOOD
int add(int a, int b) {
  return a + b;
}
```

### Pitfall 4: Ignoring Errors

```dart
// ❌ BAD: Silent failure
void loadData() async {
  await fetchData(); // Error might occur silently
}

// ✅ GOOD: Handle errors
Future<void> loadData() async {
  try {
    await fetchData();
  } catch (e) {
    logger.error('Failed to load data: $e');
    showErrorToUser(e);
  }
}
```

### Pitfall 5: Creating New Futures in Loops

```dart
// ❌ BAD: Creates futures but doesn't wait
void processItems(List<int> ids) async {
  for (var id in ids) {
    fetchItem(id); // Fires all requests immediately
  }
}

// ✅ GOOD: Sequential processing
Future<void> processItemsSequential(List<int> ids) async {
  for (var id in ids) {
    await fetchItem(id);
  }
}

// ✅ BETTER: Parallel processing
Future<void> processItemsParallel(List<int> ids) async {
  await Future.wait(ids.map((id) => fetchItem(id)));
}
```

### Pitfall 6: Misusing setState with Async

```dart
// ❌ BAD: setState after dispose
class BadWidget extends StatefulWidget {
  @override
  _BadWidgetState createState() => _BadWidgetState();
}

class _BadWidgetState extends State<BadWidget> {
  String data = '';
  
  @override
  void initState() {
    super.initState();
    loadData();
  }
  
  void loadData() async {
    final result = await fetchData();
    setState(() => data = result); // Might be called after dispose!
  }
}

// ✅ GOOD: Check if mounted
class GoodWidget extends StatefulWidget {
  @override
  _GoodWidgetState createState() => _GoodWidgetState();
}

class _GoodWidgetState extends State<GoodWidget> {
  String data = '';
  
  @override
  void initState() {
    super.initState();
    loadData();
  }
  
  void loadData() async {
    final result = await fetchData();
    if (mounted) {
      setState(() => data = result);
    }
  }
}
```

### Pitfall 7: Race Conditions

```dart
// ❌ BAD: Race condition
class SearchWidget extends StatefulWidget {
  @override
  _SearchWidgetState createState() => _SearchWidgetState();
}

class _SearchWidgetState extends State<SearchWidget> {
  List<Result> results = [];
  
  void onSearchChanged(String query) async {
    final newResults = await searchAPI(query);
    setState(() => results = newResults); // Old searches might finish last!
  }
}

// ✅ GOOD: Cancel previous requests
class SearchWidget extends StatefulWidget {
  @override
  _SearchWidgetState createState() => _SearchWidgetState();
}

class _SearchWidgetState extends State<SearchWidget> {
  List<Result> results = [];
  int _searchVersion = 0;
  
  void onSearchChanged(String query) async {
    final currentVersion = ++_searchVersion;
    final newResults = await searchAPI(query);
    
    if (currentVersion == _searchVersion) {
      setState(() => results = newResults);
    }
  }
}
```

### Pitfall 8: Deadlocks with Completer

```dart
// ❌ BAD: Completer never completes
Future<String> getData() {
  final completer = Completer<String>();
  
  fetchData().then((data) {
    if (data.isValid) {
      completer.complete(data.value);
    }
    // Missing completeError for invalid data!
  });
  
  return completer.future; // Might hang forever
}

// ✅ GOOD: Always complete or error
Future<String> getData() {
  final completer = Completer<String>();
  
  fetchData().then((data) {
    if (data.isValid) {
      completer.complete(data.value);
    } else {
      completer.completeError('Invalid data');
    }
  }).catchError((error) {
    completer.completeError(error);
  });
  
  return completer.future;
}
```

---

## Real-World Examples

### Example 1: Network Request with Cache

```dart
class UserRepository {
  final _cache = <int, User>{};
  final _api = ApiService();
  
  Future<User> getUser(int id) async {
    // Check cache first
    if (_cache.containsKey(id)) {
      return _cache[id]!;
    }
    
    // Fetch from API
    try {
      final user = await _api.fetchUser(id).timeout(
        Duration(seconds: 10),
      );
      
      // Cache the result
      _cache[id] = user;
      
      return user;
    } on TimeoutException {
      throw NetworkException('Request timed out');
    } catch (e) {
      throw NetworkException('Failed to fetch user: $e');
    }
  }
  
  void clearCache() {
    _cache.clear();
  }
}
```

### Example 2: Pagination with Stream

```dart
class PaginatedDataSource {
  final _controller = StreamController<List<Item>>();
  int _currentPage = 0;
  bool _hasMore = true;
  bool _loading = false;
  
  Stream<List<Item>> get itemsStream => _controller.stream;
  
  Future<void> loadMore() async {
    if (_loading || !_hasMore) return;
    
    _loading = true;
    
    try {
      final items = await _api.fetchPage(_currentPage);
      
      if (items.isEmpty) {
        _hasMore = false;
      } else {
        _controller.add(items);
        _currentPage++;
      }
    } catch (e) {
      _controller.addError(e);
    } finally {
      _loading = false;
    }
  }
  
  void dispose() {
    _controller.close();
  }
}
```

### Example 3: Real-time Chat

```dart
class ChatService {
  final _messagesController = StreamController<Message>.broadcast();
  final _api = WebSocketApi();
  StreamSubscription? _subscription;
  
  Stream<Message> get messages => _messagesController.stream;
  
  Future<void> connect() async {
    final socket = await _api.connect();
    
    _subscription = socket.listen(
      (data) {
        final message = Message.fromJson(data);
        _messagesController.add(message);
      },
      onError: (error) {
        _messagesController.addError(error);
      },
      onDone: () {
        _reconnect();
      },
    );
  }
  
  Future<void> sendMessage(String text) async {
    final message = Message(
      text: text,
      timestamp: DateTime.now(),
    );
    
    try {
      await _api.send(message.toJson());
      _messagesController.add(message);
    } catch (e) {
      _messagesController.addError(e);
    }
  }
  
  Future<void> _reconnect() async {
    await Future.delayed(Duration(seconds: 5));
    await connect();
  }
  
  void dispose() {
    _subscription?.cancel();
    _messagesController.close();
  }
}
```

### Example 4: Background Task Queue

```dart
class TaskQueue {
  final _queue = <Future<void> Function()>[];
  bool _processing = false;
  
  Future<void> addTask(Future<void> Function() task) async {
    final completer = Completer<void>();
    
    _queue.add(() async {
      try {
        await task();
        completer.complete();
      } catch (e) {
        completer.completeError(e);
      }
    });
    
    if (!_processing) {
      _processQueue();
    }
    
    return completer.future;
  }
  
  Future<void> _processQueue() async {
    _processing = true;
    
    while (_queue.isNotEmpty) {
      final task = _queue.removeAt(0);
      await task();
    }
    
    _processing = false;
  }
}

// Usage
final queue = TaskQueue();

await queue.addTask(() => saveToDatabase(data1));
await queue.addTask(() => saveToDatabase(data2));
await queue.addTask(() => saveToDatabase(data3));
```

### Example 5: Download Manager with Progress

```dart
class DownloadManager {
  final _progressController = StreamController<DownloadProgress>();
  
  Stream<DownloadProgress> get progressStream => _progressController.stream;
  
  Future<File> downloadFile(String url, String savePath) async {
    final client = HttpClient();
    
    try {
      final request = await client.getUrl(Uri.parse(url));
      final response = await request.close();
      
      final total = response.contentLength;
      var downloaded = 0;
      
      final file = File(savePath);
      final sink = file.openWrite();
      
      await for (var chunk in response) {
        downloaded += chunk.length;
        sink.add(chunk);
        
        final progress = DownloadProgress(
          downloaded: downloaded,
          total: total,
          percentage: (downloaded / total * 100).toInt(),
        );
        
        _progressController.add(progress);
      }
      
      await sink.close();
      client.close();
      
      return file;
    } catch (e) {
      _progressController.addError(e);
      rethrow;
    }
  }
  
  void dispose() {
    _progressController.close();
  }
}

// Usage
class DownloadWidget extends StatefulWidget {
  @override
  _DownloadWidgetState createState() => _DownloadWidgetState();
}

class _DownloadWidgetState extends State<DownloadWidget> {
  final _manager = DownloadManager();
  int _progress = 0;
  
  @override
  void initState() {
    super.initState();
    
    _manager.progressStream.listen((progress) {
      setState(() => _progress = progress.percentage);
    });
    
    _manager.downloadFile(
      'https://example.com/file.zip',
      '/path/to/save/file.zip',
    );
  }
  
  @override
  Widget build(BuildContext context) {
    return LinearProgressIndicator(value: _progress / 100);
  }
  
  @override
  void dispose() {
    _manager.dispose();
    super.dispose();
  }
}
```

---

## Testing Asynchronous Code

### Unit Testing Futures

```dart
import 'package:test/test.dart';

void main() {
  group('User Repository', () {
    late UserRepository repository;
    
    setUp(() {
      repository = UserRepository();
    });
    
    test('fetchUser returns user on success', () async {
      final user = await repository.fetchUser(1);
      
      expect(user.id, 1);
      expect(user.name, isNotEmpty);
    });
    
    test('fetchUser throws on invalid id', () {
      expect(
        () => repository.fetchUser(-1),
        throwsA(isA<ArgumentError>()),
      );
    });
    
    test('fetchUser uses cache on second call', () async {
      final user1 = await repository.fetchUser(1);
      final user2 = await repository.fetchUser(1);
      
      expect(identical(user1, user2), true);
    });
  });
}
```

### Testing Streams

```dart
import 'package:test/test.dart';

void main() {
  group('Counter Stream', () {
    test('emits correct sequence', () {
      final stream = countStream(3);
      
      expect(
        stream,
        emitsInOrder([0, 1, 2, emitsDone]),
      );
    });
    
    test('handles errors', () {
      final stream = errorStream();
      
      expect(
        stream,
        emitsError(isA<Exception>()),
      );
    });
    
    test('can be cancelled', () async {
      final stream = infiniteStream();
      final subscription = stream.listen((_) {});
      
      await Future.delayed(Duration(milliseconds: 100));
      await subscription.cancel();
      
      // Stream should stop emitting
    });
  });
}
```

### Mocking Async Operations

```dart
import 'package:mockito/mockito.dart';
import 'package:test/test.dart';

class MockApiService extends Mock implements ApiService {}

void main() {
  group('User Repository with Mock', () {
    late UserRepository repository;
    late MockApiService mockApi;
    
    setUp(() {
      mockApi = MockApiService();
      repository = UserRepository(api: mockApi);
    });
    
    test('fetchUser calls API', () async {
      final testUser = User(id: 1, name: 'Test');
      
      when(mockApi.getUser(1))
        .thenAnswer((_) async => testUser);
      
      final user = await repository.fetchUser(1);
      
      verify(mockApi.getUser(1)).called(1);
      expect(user, testUser);
    });
    
    test('fetchUser handles API errors', () async {
      when(mockApi.getUser(1))
        .thenThrow(NetworkException('Connection failed'));
      
      expect(
        () => repository.fetchUser(1),
        throwsA(isA<NetworkException>()),
      );
    });
  });
}
```

### Integration Testing

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:integration_test/integration_test.dart';

void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();
  
  testWidgets('Load and display user data', (tester) async {
    await tester.pumpWidget(MyApp());
    
    // Wait for loading indicator
    expect(find.byType(CircularProgressIndicator), findsOneWidget);
    
    // Wait for data to load (with timeout)
    await tester.pumpAndSettle(Duration(seconds: 5));
    
    // Verify data is displayed
    expect(find.text('John Doe'), findsOneWidget);
    expect(find.byType(CircularProgressIndicator), findsNothing);
  });
}
```

---

## Debugging Asynchronous Code

### Using DevTools

```dart
// Add breakpoints in async functions
Future<void> debugExample() async {
  print('Before await'); // Breakpoint 1
  
  final data = await fetchData(); // Breakpoint 2
  
  print('After await: $data'); // Breakpoint 3
}
```

### Logging Async Operations

```dart
class LoggedFuture<T> {
  static Future<T> run<T>(
    String name,
    Future<T> Function() operation,
  ) async {
    print('[$name] Starting...');
    final stopwatch = Stopwatch()..start();
    
    try {
      final result = await operation();
      print('[$name] Completed in ${stopwatch.elapsedMilliseconds}ms');
      return result;
    } catch (e) {
      print('[$name] Failed after ${stopwatch.elapsedMilliseconds}ms: $e');
      rethrow;
    }
  }
}

// Usage
final user = await LoggedFuture.run('fetchUser', () => fetchUser(123));
```

### Timeout Debugging

```dart
Future<T> debugTimeout<T>(
  String name,
  Future<T> future, {
  Duration timeout = const Duration(seconds: 10),
}) {
  return future.timeout(
    timeout,
    onTimeout: () {
      print('[$name] Timeout after ${timeout.inSeconds}s');
      throw TimeoutException('$name timed out');
    },
  );
}

// Usage
final data = await debugTimeout('API Call', fetchData());
```

---

## Performance Profiling

### Measuring Async Performance

```dart
class PerformanceMonitor {
  final _metrics = <String, List<int>>{};
  
  Future<T> measure<T>(
    String operation,
    Future<T> Function() action,
  ) async {
    final stopwatch = Stopwatch()..start();
    
    try {
      return await action();
    } finally {
      stopwatch.stop();
      _metrics.putIfAbsent(operation, () => [])
        .add(stopwatch.elapsedMilliseconds);
    }
  }
  
  void printReport() {
    print('Performance Report:');
    _metrics.forEach((operation, times) {
      final avg = times.reduce((a, b) => a + b) / times.length;
      final min = times.reduce((a, b) => a < b ? a : b);
      final max = times.reduce((a, b) => a > b ? a : b);
      
      print('$operation:');
      print('  Average: ${avg.toStringAsFixed(2)}ms');
      print('  Min: ${min}ms');
      print('  Max: ${max}ms');
      print('  Calls: ${times.length}');
    });
  }
}

// Usage
final monitor = PerformanceMonitor();

await monitor.measure('fetchUser', () => fetchUser(123));
await monitor.measure('fetchPosts', () => fetchPosts(456));

monitor.printReport();
```

### Memory Leak Detection

```dart
class StreamLeakDetector {
  final _activeStreams = <String, StreamController>{};
  
  StreamController<T> createStream<T>(String name) {
    final controller = StreamController<T>();
    _activeStreams[name] = controller;
    
    controller.onCancel = () {
      _activeStreams.remove(name);
    };
    
    return controller;
  }
  
  void checkLeaks() {
    if (_activeStreams.isNotEmpty) {
      print('WARNING: Active streams detected:');
      _activeStreams.keys.forEach((name) {
        print('  - $name');
      });
    }
  }
}
```

---

## Conclusion

Asynchronous programming in Flutter is essential for building responsive, performant applications. Key takeaways:

1. **Understand the Event Loop**: Dart's single-threaded model with microtask and event queues
2. **Use async/await**: Makes async code readable and maintainable
3. **Handle Errors Properly**: Always use try-catch or error handlers
4. **Manage Resources**: Cancel subscriptions and dispose controllers
5. **Optimize Performance**: Use parallel execution when possible
6. **Test Thoroughly**: Write comprehensive tests for async code
7. **Avoid Common Pitfalls**: Watch for race conditions, memory leaks, and silent failures

### Further Resources

- [Dart Language Tour - Asynchrony](https://dart.dev/guides/language/language-tour#asynchrony-support)
- [Flutter Cookbook - Networking](https://flutter.dev/docs/cookbook/networking)
- [Effective Dart - Usage](https://dart.dev/guides/language/effective-dart/usage#asynchrony)
- [dart:async API Reference](https://api.dart.dev/stable/dart-async/dart-async-library.html)

### Quick Reference Card

```dart
// Future basics
Future<String> fetchData() async => await api.getData();

// Error handling
try {
  await fetchData();
} catch (e) {
  handleError(e);
}

// Multiple futures
await Future.wait([future1, future2, future3]);

// Stream basics
Stream<int> countStream() async* {
  for (int i = 0; i < 10; i++) {
    yield i;
  }
}

// Stream listening
await for (final value in stream) {
  print(value);
}

// Timeout
await fetchData().timeout(Duration(seconds: 5));

// Retry
await retry(() => fetchData(), maxAttempts: 3);
```

---

**Version**: 1.0  
**Last Updated**: January 2026  
**Author**: Flutter Development Guide  
**License**: MIT

---