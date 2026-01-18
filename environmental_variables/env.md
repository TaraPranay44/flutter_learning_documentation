# Flutter Environment Variables

## Overview

Environment variables in Flutter allow you to configure different values for development, staging, and production environments without changing your code. They help manage API keys, endpoints, feature flags, and other configuration values securely.

## Common Approaches

### 1. Dart Define (Recommended)

Pass variables at compile time using `--dart-define`:

```bash
flutter run --dart-define=API_URL=https://api.example.com --dart-define=API_KEY=abc123
```

Access in code:

```dart
const apiUrl = String.fromEnvironment('API_URL', defaultValue: 'https://dev.api.example.com');
const apiKey = String.fromEnvironment('API_KEY');
```

### 2. Environment-Specific Entry Points

Create separate main files for each environment:

- `main_dev.dart`
- `main_staging.dart`
- `main_prod.dart`

```dart
// main_prod.dart
void main() {
  const environment = Environment(
    apiUrl: 'https://api.production.com',
    enableLogging: false,
  );
  runApp(MyApp(environment: environment));
}
```

Run with: `flutter run -t lib/main_dev.dart`

### 3. .env Files with flutter_dotenv

Add dependency:

```yaml
dependencies:
  flutter_dotenv: ^5.0.2
```

Create `.env` file:

```
API_URL=https://api.example.com
API_KEY=your_secret_key
```

Load and use:

```dart
import 'package:flutter_dotenv/flutter_dotenv.dart';

Future<void> main() async {
  await dotenv.load(fileName: ".env");
  runApp(MyApp());
}

// Access variables
final apiUrl = dotenv.env['API_URL'];
```

## Best Practices

- Never commit sensitive keys to version control (add `.env` to `.gitignore`)
- Use `--dart-define-from-file` for multiple variables (Flutter 3.7+)
- Create a configuration class to centralize environment values
- Provide sensible default values for optional configurations
- Document required environment variables in your README

## Security Considerations

- Environment variables compiled with `--dart-define` are embedded in the app binary
- For highly sensitive data, consider using secure storage solutions
- Use different API keys for each environment
- Implement proper backend security rather than relying solely on client-side protection

---

## Interview Questions

### Question 1: Dart Define vs dotenv Package

**Q: What are the key differences between using `--dart-define` and the `flutter_dotenv` package for managing environment variables in Flutter? When would you choose one approach over the other?**

**Expected Answer:**

`--dart-define` compiles variables at build time, making them constants that can be tree-shaken and optimized by the compiler. They're accessed via `String.fromEnvironment()` and are immutable. This approach is best for configuration that differs between build flavors but doesn't change at runtime.

`flutter_dotenv` loads variables at runtime from a file, providing more flexibility to change values without recompiling. However, this adds a small runtime overhead and the .env file must be included in assets. This approach is better for development when you need to frequently change configurations without rebuilding.

For production, `--dart-define` is generally preferred as it's more performant and doesn't require shipping an additional file. For development convenience, dotenv might be easier.

### Question 2: Build Flavors and Environment Configuration

**Q: You're building a Flutter app that needs to connect to different API endpoints for development, staging, and production. How would you implement this using build flavors, and what would be the advantages of this approach over hardcoding environment checks?**

**Expected Answer:**

I would create build flavors (Android) and schemes (iOS) for each environment, combined with `--dart-define` or separate entry points:

```bash
flutter run --flavor dev -t lib/main_dev.dart --dart-define=ENV=dev
flutter run --flavor prod -t lib/main_prod.dart --dart-define=ENV=prod
```

Configure `android/app/build.gradle` with product flavors and `ios/Runner.xcodeproj` with schemes. Create an environment config class that reads these values.

Advantages: type-safe configuration, compile-time optimization, can't accidentally deploy wrong environment, enables different app IDs for parallel installation, and can customize app icons/names per environment. This is superior to runtime checks (like `if (kDebugMode)`) because it provides true isolation and prevents production code from even including development endpoints.

### Question 3: Security and Environment Variables

**Q: A developer on your team wants to store API keys in environment variables using `--dart-define` for a mobile app. What security concerns would you raise, and what alternative approaches would you suggest for handling sensitive credentials?**

**Expected Answer:**

The main concern is that `--dart-define` embeds values directly in the compiled app binary, where they can be extracted through reverse engineering using tools like apktool or class-dump. Mobile apps should be considered untrusted clients.

For sensitive operations, implement these practices:

1. Never store sensitive keys client-side; use a backend proxy that authenticates users and makes authenticated API calls on their behalf
2. Use OAuth/JWT tokens with short expiration times instead of long-lived API keys
3. Implement certificate pinning to prevent MITM attacks
4. For necessary client-side keys (like analytics), use obfuscation and accept they can be extracted, but limit their permissions on the backend
5. Consider using platform-specific secure storage (Keychain/Keystore) for user-specific tokens
6. Implement backend rate limiting and API key rotation strategies

The architecture should assume the client is compromised and design security accordingly.