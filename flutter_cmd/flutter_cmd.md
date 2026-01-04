# Flutter CLI Commands - Complete Guide

A comprehensive reference for Flutter Command Line Interface commands, suitable for beginners to tech leads.

---

## Table of Contents

1. [Introduction](#introduction)
2. [Getting Started](#getting-started)
3. [Project Management](#project-management)
4. [Building & Running](#building--running)
5. [Package Management](#package-management)
6. [Dart Commands](#dart-commands)
7. [Testing & Analysis](#testing--analysis)
8. [Device Management](#device-management)
9. [Configuration & Maintenance](#configuration--maintenance)
10. [Debugging & Troubleshooting](#debugging--troubleshooting)
11. [Advanced Commands](#advanced-commands)

---

## Introduction

Flutter's CLI provides powerful tools to manage your development workflow directly from the terminal. While IDEs offer graphical interfaces for these operations, understanding CLI commands gives you greater control and flexibility.

### Why Use Flutter CLI?

- **Speed**: Execute commands faster than navigating through IDE menus
- **Automation**: Script repetitive tasks and integrate with CI/CD pipelines
- **Flexibility**: Access all Flutter features regardless of your IDE
- **Debugging**: Better understanding of what happens behind the scenes

---

## Getting Started

### Getting Help

Always start here when you're unsure about a command.

```bash
flutter --help --verbose
```

**What it does**: Lists all available Flutter commands with detailed descriptions.

**When to use**: When you're stuck or exploring new commands.

---

### Check Flutter Installation

```bash
flutter doctor
```

**What it does**: Diagnoses your Flutter installation and shows:

- Flutter SDK status
- Platform-specific tooling (Android SDK, Xcode)
- IDE plugins (VS Code, Android Studio)
- Connected devices compatibility

**Output example**:

```
[✓] Flutter (Channel stable, 3.x.x)
[✓] Android toolchain
[✓] Xcode - develop for iOS and macOS
[✓] Chrome - develop for the web
[✓] Android Studio
[✓] Connected device
```

**When to use**:

- After installing Flutter
- When troubleshooting environment issues
- Before starting a new project

---

### Check Versions

```bash
flutter version
```

**What it does**: Displays version information for Flutter and Dart SDKs.

**When to use**: When you need to verify which version you're running or for bug reports.

---

### Check Available Channels

```bash
flutter channel <CHANNEL_NAME>
```

**What it does**: Lists all Flutter channels (stable, beta, dev, master) and allows switching between them.

**Available channels**:

- **stable**: Most reliable, recommended for production
- **beta**: Preview upcoming features
- **dev**: Latest developments, may be unstable
- **master**: Cutting edge, frequently updated

**Example**:

```bash
flutter channel stable  # Switch to stable
flutter channel         # List all channels
```

**Where channels are established**:

Flutter channels are managed at the **system/machine level** in your Flutter SDK installation, not per project. Here's the scope:

1. **Global System Setting**

   - Location: Your Flutter SDK installation directory
   - Typical paths:
     - macOS/Linux: `~/flutter` or `/usr/local/flutter`
     - Windows: `C:\src\flutter` or `%USERPROFILE%\flutter`
   - Impact: Affects **ALL** Flutter projects on your machine

2. **Git-Based Channels**

   - Flutter SDK is a Git repository
   - Each channel is a Git branch
   - Switching channels = checking out different branches
   - Channel info stored in: `<flutter-sdk>/.git/`

3. **Not Project-Specific**
   - No per-project channel configuration
   - All projects use the same Flutter SDK version/channel
   - To work with different channels, you need:
     - Multiple Flutter SDK installations (different directories)
     - Environment path switching
     - Version management tools (like FVM)

**Using Multiple Channels Simultaneously**:

If you need different projects on different channels:

```bash
# Option 1: Multiple Flutter installations
/flutter-stable/    # Stable channel
/flutter-beta/      # Beta channel
/flutter-dev/       # Dev channel

# Switch PATH variable to use different installation

# Option 2: Use Flutter Version Manager (FVM)
fvm install stable
fvm install beta
fvm use stable --force    # For specific project
fvm use beta --force      # For another project
```

**Check current channel**:

```bash
flutter channel           # Shows active channel with asterisk (*)
```

**Workflow example**:

```bash
# System-wide channel change
flutter channel stable
flutter upgrade           # Get latest stable
cd my-production-app
flutter run               # Uses stable channel

# Switch to beta for another project
flutter channel beta
flutter upgrade           # Get latest beta
cd my-experimental-app
flutter run               # Uses beta channel
```

---

## Project Management

### Create a New Project

```bash
flutter create <APP_NAME>
```

**What it does**: Creates a new Flutter project with a complete skeleton structure including:

- `lib/` folder with main.dart
- `pubspec.yaml` for dependencies
- Platform-specific folders (android, ios, web)
- Test files

**Example**:

```bash
flutter create my_awesome_app
cd my_awesome_app
```

**Best practices**:

- Use lowercase with underscores for app names
- Navigate to your desired directory first using `cd`

---

### Clean Project

```bash
flutter clean
```

**What it does**: Deletes the build directory including:

- `dart-tool/` folder
- `build/android/` folder
- `build/ios/` folder

**When to use**:

- Free up disk space (builds can consume ~200MB+)
- Resolve build issues after updating dependencies
- Before switching branches in version control
- When experiencing mysterious build errors

**Tip**: Run this command on unused projects to save significant disk space.

---

## Building & Running

### Run Your Application

```bash
flutter run <DART_FILE>
```

**What it does**: Launches your app on a connected device or emulator with hot reload enabled.

**Build modes**:

```bash
flutter run --debug     # Debug mode (default, hot reload enabled)
flutter run --profile   # Profile mode (performance profiling)
flutter run --release   # Release mode (optimized for production)
```

**Custom flavors**:

```bash
flutter run --flavor development
flutter run --flavor production
```

**When to use**:

- **Debug**: During active development
- **Profile**: When testing performance
- **Release**: Testing production builds

---

### Build for Deployment

#### Android

```bash
flutter build apk           # Build APK
flutter build appbundle     # Build App Bundle (recommended for Play Store)
```

**Build mode options**:

```bash
flutter build apk --debug      # Debug build
flutter build apk --profile    # Profile build
flutter build apk --release    # Release build (default)
```

**Custom flavors**:

```bash
flutter build apk --flavor production
```

#### iOS

```bash
flutter build ios
```

#### Web

```bash
flutter build web
```

**When to use**:

- **APK**: For direct installation or testing
- **App Bundle**: For Google Play Store distribution (smaller download size)
- **iOS**: For TestFlight or App Store
- **Web**: For web deployment

---

### Install Built Application

```bash
flutter install -d <DEVICE_ID>
```

**What it does**: Installs a previously built app on a specific device.

**Prerequisites**: Must run `flutter build` first.

**Example workflow**:

```bash
flutter build apk --release
flutter install -d emulator-5554
```

---

### Attach to Running Application

```bash
flutter attach -d <DEVICE_ID>
```

**What it does**: Reattaches to an already running Flutter app, restoring hot reload and DevTools functionality.

**When to use**:

- Lost connection during debugging
- Adding Flutter to a pre-existing native app
- Avoid rebuilding after connection loss

**Use case**: Essential when integrating Flutter into existing native applications.

---

## Package Management

### View Dependencies

Your project's dependencies are defined in `pubspec.yaml`.

### Download Dependencies

```bash
flutter pub get
```

**What it does**: Downloads all packages listed in `pubspec.yaml`.

**When to use**:

- After cloning a project
- After manually editing `pubspec.yaml`
- When dependencies are missing

---

### Add a Package

```bash
flutter pub add <package_name>         # Regular dependency
flutter pub add -d <package_name>      # Dev dependency
```

**What it does**: Adds a package to `pubspec.yaml` and downloads it.

**Example**:

```bash
flutter pub add http                   # For production
flutter pub add -d mockito             # For testing only
```

---

### Remove a Package

```bash
flutter pub remove <package_name>
```

**What it does**: Removes a package from `pubspec.yaml`.

**Example**:

```bash
flutter pub remove http
```

---

### Update Packages

```bash
flutter pub update
```

**What it does**: Updates all packages to their latest compatible versions based on constraints in `pubspec.yaml`.

**When to use**: Regularly, to keep dependencies up to date.

---

### Fetch Necessary Packages and Build

```bash
flutter assemble -o <DIRECTORY>
```

**What it does**: Fetches missing packages and builds the app in one command.

---

## Dart Commands

While Flutter provides high-level commands, Dart CLI offers lower-level control and additional functionality. These commands are essential for code generation, compilation, and advanced workflows.

### Run Dart Files Directly

```bash
dart run <DART_FILE>
```

**What it does**: Executes a Dart file directly without Flutter framework.

**Example**:

```bash
dart run bin/main.dart
dart run lib/utils/helper.dart
```

**When to use**:

- Running pure Dart scripts
- Backend/server-side Dart code
- Utility scripts

---

### Code Generation with Build Runner

```bash
dart run build_runner build
```

**What it does**: Generates code for packages that use code generation (annotations).

**Common variations**:

```bash
# One-time build
dart run build_runner build

# Delete conflicting outputs and rebuild
dart run build_runner build --delete-conflicting-outputs

# Watch mode - auto-rebuild on file changes
dart run build_runner watch

# Watch mode with delete conflicts
dart run build_runner watch --delete-conflicting-outputs

# Clean generated files
dart run build_runner clean
```

**When to use**:

- With packages like `json_serializable`, `freezed`, `injectable`, `auto_route`
- After modifying model classes with annotations
- When generated files are out of sync

**Example workflow**:

```bash
# Add dependencies
flutter pub add json_annotation
flutter pub add -d build_runner json_serializable

# Create model with annotations
# Then generate code
dart run build_runner build --delete-conflicting-outputs
```

**Common packages requiring build_runner**:

- `json_serializable` - JSON serialization
- `freezed` - Immutable classes and unions
- `injectable` - Dependency injection
- `auto_route` - Route generation
- `mockito` - Mock generation for testing
- `retrofit` - REST API client generation
- `floor` - SQLite database

---

### Dart Pub Commands

While `flutter pub` is commonly used, `dart pub` provides the same functionality:

```bash
dart pub get              # Get dependencies
dart pub add <package>    # Add package
dart pub remove <package> # Remove package
dart pub upgrade          # Upgrade packages
dart pub outdated         # Check for outdated packages
dart pub deps             # Show dependency tree
```

**Difference between `flutter pub` and `dart pub`**:

- `flutter pub` = `dart pub` + Flutter-specific processing
- For Flutter projects, prefer `flutter pub`
- For pure Dart projects, use `dart pub`

---

### Compile Dart Code

```bash
# Compile to executable
dart compile exe <DART_FILE> -o <OUTPUT_FILE>

# Compile to JavaScript
dart compile js <DART_FILE> -o <OUTPUT_FILE>

# Compile to kernel (AOT)
dart compile kernel <DART_FILE> -o <OUTPUT_FILE>

# Compile to JIT snapshot
dart compile jit-snapshot <DART_FILE> -o <OUTPUT_FILE>
```

**Examples**:

```bash
# Create standalone executable
dart compile exe bin/server.dart -o bin/server

# Compile for web
dart compile js lib/main.dart -o build/main.js
```

**When to use**:

- Creating standalone executables
- Deploying Dart backend applications
- Performance optimization

---

### Analyze Dart Code

```bash
dart analyze [<DIRECTORY>]
```

**What it does**: Analyzes Dart code for issues (similar to `flutter analyze`).

**Example**:

```bash
dart analyze                    # Analyze entire project
dart analyze lib/               # Analyze specific directory
dart analyze lib/main.dart      # Analyze specific file
```

---

### Format Dart Code

```bash
dart format <FILES_OR_DIRECTORIES>
```

**What it does**: Formats Dart code according to Dart style guidelines.

**Options**:

```bash
dart format .                   # Format all files
dart format lib/                # Format directory
dart format lib/main.dart       # Format specific file
dart format --output none .     # Check formatting without changes
dart format --set-exit-if-changed .  # Exit code 1 if changes needed
```

**When to use**:

- Pre-commit hooks
- CI/CD pipelines
- Batch formatting

---

### Run Tests

```bash
dart test [<FILE_OR_DIRECTORY>]
```

**What it does**: Runs Dart tests (alternative to `flutter test`).

**Example**:

```bash
dart test                       # Run all tests
dart test test/unit/            # Run specific directory
dart test test/user_test.dart   # Run specific test file
```

---

### Create New Dart Project

```bash
dart create <PROJECT_NAME>
```

**What it does**: Creates a new Dart project (non-Flutter).

**Project templates**:

```bash
dart create -t console my_app           # Console application
dart create -t package my_package       # Package/library
dart create -t server-shelf my_server   # Web server
dart create -t web my_web_app           # Web application
```

---

### Pub Global Commands

Manage globally installed Dart packages:

```bash
# Activate global package
dart pub global activate <PACKAGE_NAME>

# Run global package
dart pub global run <PACKAGE_NAME>

# Deactivate global package
dart pub global deactivate <PACKAGE_NAME>

# List global packages
dart pub global list
```

**Example - Install and use FVM**:

```bash
dart pub global activate fvm
fvm --version
```

**Common global packages**:

- `fvm` - Flutter Version Management
- `dhttpd` - Simple HTTP server
- `coverage` - Code coverage tools
- `dart_style` - Code formatter
- `stagehand` - Project templates

---

### Dart Fix

```bash
dart fix [<DIRECTORY>]
```

**What it does**: Automatically applies fixes for deprecated APIs and lints.

**Options**:

```bash
dart fix --dry-run              # Preview fixes without applying
dart fix --apply                # Apply all fixes
dart fix lib/                   # Fix specific directory
```

**When to use**:

- After upgrading Flutter/Dart versions
- Migrating to new APIs
- Fixing lint warnings in bulk

---

### Dart Doc

```bash
dart doc [<DIRECTORY>]
```

**What it does**: Generates API documentation from code comments.

**Example**:

```bash
dart doc .                      # Generate docs for project
dart doc lib/                   # Generate docs for specific directory
```

**Output**: Creates documentation in `doc/api/` directory.

---

### Check Dart Version

```bash
dart --version
```

**What it does**: Shows the Dart SDK version.

**Example output**:

```
Dart SDK version: 3.2.0 (stable)
```

---

### Dart REPL (Interactive Mode)

```bash
dart
```

**What it does**: Starts an interactive Dart REPL (Read-Eval-Print Loop).

**Example**:

```dart
>>> print('Hello Dart');
Hello Dart
>>> var x = 10;
>>> print(x * 2);
20
>>> exit()
```

**When to use**: Quick Dart experiments and testing snippets.

---

### Common Dart Workflows

#### Code Generation Workflow

```bash
# 1. Add dependencies
flutter pub add freezed_annotation
flutter pub add -d build_runner freezed

# 2. Create model with annotations
# (write your model class)

# 3. Generate code
dart run build_runner build --delete-conflicting-outputs

# 4. For continuous development
dart run build_runner watch --delete-conflicting-outputs
```

#### JSON Serialization Workflow

```bash
# 1. Add dependencies
flutter pub add json_annotation
flutter pub add -d build_runner json_serializable

# 2. Create model with @JsonSerializable() annotation

# 3. Generate serialization code
dart run build_runner build --delete-conflicting-outputs
```

#### Dependency Injection Setup (Injectable)

```bash
# 1. Add dependencies
flutter pub add get_it injectable
flutter pub add -d build_runner injectable_generator

# 2. Set up dependency injection

# 3. Generate injection code
dart run build_runner build --delete-conflicting-outputs
```

---

## Testing & Analysis

### Run Tests

```bash
flutter test [<DIRECTORY|DART_FILE>]
```

**What it does**: Runs unit tests, widget tests, or integration tests.

**Examples**:

```bash
flutter test                           # Run all tests
flutter test test/widget_test.dart     # Run specific test
flutter test test/                     # Run tests in directory
```

**When to use**:

- Before committing code
- As part of CI/CD pipeline
- When refactoring

---

### Analyze Code

```bash
flutter analyze -d <DEVICE_ID>
```

**What it does**: Performs static analysis to find:

- Potential errors
- Code style issues
- Unused imports
- Missing code

**When to use**:

- Before committing code
- During code reviews
- To maintain code quality

---

## Device Management

### List Connected Devices

```bash
flutter devices -d <DEVICE_ID>
```

**What it does**: Shows all available devices including:

- Physical devices (Android/iOS)
- Emulators/Simulators
- Chrome browser
- Desktop (macOS/Windows/Linux)

**Example output**:

```
3 connected devices:

sdk gphone64 arm64 (mobile) • emulator-5554 • android-arm64 • Android 13
iPhone 14 Pro Max (mobile)  • 12345-67890   • ios          • iOS 16.0
Chrome (web)                • chrome        • web-javascript • Google Chrome 108
```

---

### List Available Emulators

```bash
flutter emulators
```

**What it does**: Lists all installed emulators and provides options to:

- Launch existing emulators
- Create new emulators

**Example**:

```bash
flutter emulators --launch <emulator_id>
```

---

### Test Device Hardware Access

```bash
flutter drive
```

**What it does**: Tests driver integration and hardware access for features like:

- Camera
- Location services
- Sensors

**When to use**: When your app accesses device hardware.

---

## Configuration & Maintenance

### Configure Flutter Settings

```bash
flutter config --build-dir=<DIRECTORY>
```

**What it does**: Configures Flutter functionalities.

**Common configurations**:

```bash
flutter config --enable-web             # Enable web support
flutter config --enable-macos-desktop   # Enable macOS desktop
flutter config --enable-windows-desktop # Enable Windows desktop
flutter config --enable-linux-desktop   # Enable Linux desktop
```

---

### Upgrade Flutter

```bash
flutter upgrade
```

**What it does**: Updates Flutter SDK and Dart SDK to the latest version in your current channel.

**Best practice**: Run this after major Flutter releases.

**Example workflow**:

```bash
flutter channel stable
flutter upgrade
flutter doctor
```

---

### Downgrade Flutter

```bash
flutter downgrade
```

**What it does**: Reverts to the previous active Flutter version.

**When to use**:

- When new version introduces breaking changes
- Compatibility issues with dependencies
- Project requires older Flutter version

---

## Debugging & Troubleshooting

### View Logs

```bash
flutter logs
```

**What it does**: Shows real-time log output from running applications.

**When to use**:

- Debugging runtime errors
- Monitoring app behavior
- Tracking exceptions

---

### Verbose Logging

```bash
flutter run --verbose
```

**What it does**: Shows detailed output including:

- Shell commands executed
- Build process details
- Network requests

**When to use**: Diagnosing complex issues.

---

### Generate Crash Reports

```bash
flutter --bug-report
```

**What it does**: Captures system information for bug reports including:

- Flutter version
- Dart version
- Operating system details
- Device information

**When to use**: When reporting bugs to Flutter team or package maintainers.

---

## Advanced Commands

### Format Code

```bash
flutter format <DART_FILE | DIRECTORY>
```

**What it does**: Formats Dart code according to Flutter style guidelines.

**Examples**:

```bash
flutter format lib/                    # Format all files in lib
flutter format lib/main.dart           # Format specific file
```

**Note**: Most IDEs auto-format on save, but this is useful for:

- CI/CD pipelines
- Batch formatting
- Pre-commit hooks

---

### Generate Localization Files

```bash
flutter gen-l10n <DIRECTORY>
```

**What it does**: Generates localization files from ARB (Application Resource Bundle) files.

**When to use**: Building multi-language apps.

**Get options**:

```bash
flutter gen-l10n -h
```

---

### Symbolicate Stack Traces

```bash
flutter symbolize --input=<STACK_TRACE_FILE>
```

**What it does**: Converts obfuscated stack traces into human-readable format.

**When to use**:

- Debugging release builds
- Analyzing crash reports from production
- Working with obfuscated code

---

### Precache Dependencies

```bash
flutter precache <ARGUMENTS>
```

**What it does**: Pre-downloads Flutter dependencies and artifacts for faster subsequent builds.

**Example**:

```bash
flutter precache --ios       # Precache iOS dependencies
flutter precache --android   # Precache Android dependencies
flutter precache --web       # Precache web dependencies
```

---

## Quick Reference Cheatsheet

| Category     | Command                          | Purpose                  |
| ------------ | -------------------------------- | ------------------------ |
| **Help**     | `flutter --help --verbose`       | List all commands        |
| **Project**  | `flutter create <app_name>`      | Create new project       |
| **Project**  | `flutter clean`                  | Clear build files        |
| **Run**      | `flutter run`                    | Run app with hot reload  |
| **Build**    | `flutter build apk`              | Build Android APK        |
| **Build**    | `flutter build appbundle`        | Build Android App Bundle |
| **Build**    | `flutter build ios`              | Build iOS app            |
| **Build**    | `flutter build web`              | Build web app            |
| **Packages** | `flutter pub get`                | Download dependencies    |
| **Packages** | `flutter pub add <package>`      | Add package              |
| **Packages** | `flutter pub remove <package>`   | Remove package           |
| **Packages** | `flutter pub update`             | Update packages          |
| **Dart**     | `dart run build_runner build`    | Generate code            |
| **Dart**     | `dart run build_runner watch`    | Watch & generate code    |
| **Dart**     | `dart compile exe <file>`        | Compile to executable    |
| **Dart**     | `dart pub global activate <pkg>` | Install global package   |
| **Dart**     | `dart fix --apply`               | Apply automated fixes    |
| **Test**     | `flutter test`                   | Run tests                |
| **Analysis** | `flutter analyze`                | Analyze code             |
| **Devices**  | `flutter devices`                | List devices             |
| **Devices**  | `flutter emulators`              | List emulators           |
| **System**   | `flutter doctor`                 | Check installation       |
| **System**   | `flutter upgrade`                | Update Flutter           |
| **System**   | `flutter version`                | Show version             |
| **System**   | `flutter channel`                | Manage channels          |
| **Debug**    | `flutter logs`                   | View logs                |
| **Format**   | `flutter format <file>`          | Format code              |

---

## Best Practices

### For Beginners

1. **Start with `flutter doctor`**: Ensure your environment is properly set up
2. **Use `flutter create`**: Let Flutter generate the project structure
3. **Master `flutter run`**: Your primary development command
4. **Learn `flutter pub get`**: Essential for dependency management
5. **Try `flutter --help`**: Your built-in documentation

### For Intermediate Developers

1. **Use build modes wisely**: Debug for development, profile for optimization, release for production
2. **Run `flutter analyze`** regularly to maintain code quality
3. **Leverage `flutter test`**: Automated testing saves time
4. **Master `dart run build_runner`**: Essential for code generation packages
5. **Use watch mode**: `dart run build_runner watch` for continuous development
6. **Clean builds**: Run `flutter clean` when troubleshooting
7. **Format code**: Use `flutter format` before committing
8. **Apply fixes**: Use `dart fix --apply` after upgrades

### For Tech Leads

1. **Standardize channels**: Keep team on the same Flutter channel
2. **CI/CD integration**: Script Flutter and Dart commands for automation
3. **Version control**: Document required Flutter version in README
4. **Build flavors**: Use `--flavor` for different environments
5. **Monitor versions**: Regularly run `flutter upgrade` in stable channel
6. **Code generation strategy**: Establish team conventions for build_runner
7. **Global tools**: Standardize globally installed Dart packages (FVM, etc.)
8. **Pre-commit hooks**: Automate `dart format` and `dart analyze`

---

## Common Workflows

### Starting a New Project

```bash
flutter create my_app
cd my_app
flutter pub get
flutter run
```

### Adding a Package

```bash
flutter pub add http
flutter pub get
flutter run
```

### Preparing for Release

```bash
flutter clean
flutter pub get
dart run build_runner build --delete-conflicting-outputs  # If using code gen
flutter test
flutter analyze
flutter build appbundle --release
```

### Code Generation Development

```bash
# One-time generation
dart run build_runner build --delete-conflicting-outputs

# Continuous development (watches for changes)
dart run build_runner watch --delete-conflicting-outputs
```

### Setting Up JSON Serialization

```bash
flutter pub add json_annotation
flutter pub add -d build_runner json_serializable
# Create your model classes with @JsonSerializable()
dart run build_runner build --delete-conflicting-outputs
```

### Troubleshooting Build Issues

```bash
flutter clean
flutter pub get
flutter doctor
flutter run --verbose
```

---

## Conclusion

Mastering Flutter CLI commands enhances your productivity and gives you deeper control over your development workflow. While IDEs provide convenience, CLI commands are essential for automation, troubleshooting, and advanced workflows.

### Next Steps

1. Practice these commands regularly
2. Explore `flutter --help` for more options
3. Integrate commands into your daily workflow
4. Script repetitive tasks
5. Contribute to the Flutter community

**Happy Flutter Development! 🚀**
