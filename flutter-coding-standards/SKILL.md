---
name: flutter-coding-standards
description: Comprehensive Flutter and Dart coding standards for building production-quality apps. Use this skill when writing, reviewing, or refactoring Flutter code to ensure consistent project structure, naming conventions, state management patterns, widget composition, testing practices, error handling, and performance optimization.
license: MIT
metadata:
  author: brenobattaglin
  version: "2.0.0"
  date: July 2026
  abstract: A complete set of Flutter and Dart coding standards organized across 8 categories — multi-package project structure with Dart workspaces, feature-first architecture with public/private module separation, BLoC/Cubit state management with GetIt service locator, naming conventions, widget composition, testing, error handling, and performance. Each section includes rationale, correct vs. incorrect examples, and actionable rules for AI agents to follow when generating, reviewing, or optimizing Flutter code.
---

# Flutter Coding Standards

Comprehensive coding standards for Flutter and Dart projects. Apply these rules when generating, reviewing, or refactoring Flutter code.

---

## 1. Project Structure

### 1.1 Multi-Package Architecture with Dart Workspaces

The project is a **multi-package monorepo** managed with **Dart workspaces**. Each feature module and the core modules are independent Dart packages, linked together via a root `pubspec.yaml` workspace definition. The core follows the same public/private split as features: `core_api` for shared abstractions and `core` for app-specific implementations.

#### Root `pubspec.yaml` (workspace definition)

```yaml
name: app_workspace
publish_to: none

environment:
  sdk: ^3.7.0

workspace:
  - lib/core/core_api
  - lib/core/core
  - lib/features/auth/auth_api
  - lib/features/auth/auth
  - lib/features/home/home_api
  - lib/features/home/home
  # ... add all feature and core packages here
```

> **Rule:** Every new feature or core package MUST be registered in the root `workspace:` list.

#### Package-level `pubspec.yaml` (each module)

Each package declares `resolution: workspace` to participate in the shared dependency resolution:

```yaml
# lib/features/auth/auth/pubspec.yaml
name: auth
publish_to: none

resolution: workspace

environment:
  sdk: ^3.7.0

dependencies:
  flutter:
    sdk: flutter
  auth_api:
    path: ../auth_api
  core_api:
    path: ../../../core/core_api
  core:
    path: ../../../core/core
  flutter_bloc: ^9.0.0
  hydrated_bloc: ^10.0.0
  get_it: ^8.0.0
  auto_route: ^9.0.0
  dio: ^5.0.0
```

### 1.2 Feature-First Organization

Organize code by feature inside `lib/features/`. Each feature consists of up to **two packages**: a **public API module** (suffix `_api`) and a **private implementation module** (no suffix). The `core` follows the same split pattern.

```
project_root/
├── pubspec.yaml                          # Dart workspace definition
├── lib/
│   ├── core/
│   │   ├── core_api/                     # PUBLIC core (shared abstractions)
│   │   │   ├── pubspec.yaml
│   │   │   └── lib/
│   │   │       ├── core_api.dart         # Barrel: exports shared contracts
│   │   │       └── src/
│   │   │           ├── errors/
│   │   │           │   ├── errors.dart        # Barrel file
│   │   │           │   ├── app_exception.dart
│   │   │           │   └── result.dart
│   │   │           ├── extensions/
│   │   │           │   ├── extensions.dart    # Barrel file
│   │   │           │   └── string_ext.dart
│   │   │           ├── constants/
│   │   │           │   ├── constants.dart     # Barrel file
│   │   │           │   └── app_constants.dart
│   │   │           └── utils/
│   │   │               ├── utils.dart         # Barrel file
│   │   │               └── date_utils.dart
│   │   └── core/                             # PRIVATE core (app implementations)
│   │       ├── pubspec.yaml
│   │       └── lib/
│   │           ├── core.dart                 # Barrel: exports bootstrap + app-specific code
│   │           └── src/
│   │               ├── bootstrap/
│   │               │   ├── bootstrap.dart         # Barrel file
│   │               │   └── core_bootstrap.dart    # GetIt + route registration
│   │               ├── di/
│   │               │   ├── di.dart               # Barrel file
│   │               │   └── service_locator.dart  # GetIt instance (`sl`)
│   │               ├── network/
│   │               │   ├── network.dart          # Barrel file
│   │               │   └── dio_client.dart       # Dio HTTP client implementation
│   │               ├── router/
│   │               │   ├── router.dart           # Barrel file
│   │               │   └── app_router.dart       # Root AutoRouter config
│   │               └── services/
│   │                   ├── services.dart          # Barrel file
│   │                   └── logger_service_impl.dart
│   ├── features/
│   │   ├── auth/
│   │   │   ├── auth_api/                 # PUBLIC module (abstractions only)
│   │   │   │   ├── pubspec.yaml
│   │   │   │   └── lib/
│   │   │   │       ├── auth_api.dart     # Barrel: exports ONLY public contracts
│   │   │   │       └── src/
│   │   │   │           ├── services/
│   │   │   │           │   └── auth_service.dart       # Abstract class
│   │   │   │           └── models/
│   │   │   │               └── auth_user.dart          # Shared model/entity
│   │   │   └── auth/                     # PRIVATE module (implementation)
│   │   │       ├── pubspec.yaml
│   │   │       └── lib/
│   │   │           ├── auth.dart         # Barrel: exports bootstrap + pages
│   │   │           └── src/
│   │   │               ├── bootstrap/
│   │   │               │   ├── bootstrap.dart               # Barrel file
│   │   │               │   └── auth_bootstrap.dart          # GetIt + route registration
│   │   │               ├── services/
│   │   │               │   ├── services.dart               # Barrel file
│   │   │               │   └── auth_service_impl.dart      # Implements AuthService
│   │   │               ├── repositories/
│   │   │               │   ├── repositories.dart           # Barrel file
│   │   │               │   └── auth_repository.dart
│   │   │               ├── blocs/
│   │   │               │   ├── blocs.dart                  # Barrel file
│   │   │               │   ├── auth_bloc.dart
│   │   │               │   ├── auth_event.dart
│   │   │               │   └── auth_state.dart
│   │   │               ├── cubits/
│   │   │               │   ├── cubits.dart                 # Barrel file
│   │   │               │   └── session_cubit.dart
│   │   │               ├── routes/
│   │   │               │   ├── routes.dart                 # Barrel file
│   │   │               │   └── auth_routes.dart            # Feature AutoRoute definitions
│   │   │               ├── pages/
│   │   │               │   ├── pages.dart                  # Barrel file
│   │   │               │   └── login_page.dart
│   │   │               └── widgets/
│   │   │                   ├── widgets.dart                # Barrel file
│   │   │                   └── login_form.dart
│   │   └── home/
│   │       ├── home_api/
│   │       │   └── ...                   # Same pattern as auth_api
│   │       └── home/
│   │           └── ...                   # Same pattern as auth
│   └── main.dart                         # App entry point
└── test/
    └── ...                               # Mirrors lib/ structure
```

### 1.3 Public vs. Private Module Rules

This split applies to **both features and core**.

| Aspect | Public Module (`_api` suffix) | Private Module (no suffix) |
| --- | --- | --- |
| **Purpose** | Expose abstractions for cross-module use | Contain all implementation details |
| **Contains** | Abstract service classes, shared models/entities, constants, extensions, utilities | Service implementations, repositories, blocs, cubits, pages, widgets, bootstrap |
| **Exports** | Only abstract classes and models needed by other modules | Bootstrap function and page widgets (for routing) |
| **Dependencies** | Minimal — `core_api`, Flutter SDK | Can depend on its own `_api`, `core_api`, `core`, external packages |
| **Who depends on it** | Other modules (both public and private) that need the abstraction | The app shell / main.dart (for registration and routing) |

#### Core-specific split

| Aspect | `core_api` | `core` |
| --- | --- | --- |
| **Purpose** | Shared abstractions, models, utilities, constants, and extensions used across all features | App-specific implementations, GetIt service locator instance, Dio client, root router, core bootstrap |
| **Contains** | `Result`, `AppException`, extension methods, shared constants, abstract service contracts (e.g., `LoggerService`) | `sl` (GetIt instance), `CoreBootstrap`, `DioClient`, `AppRouter`, concrete service implementations |
| **Who depends on it** | Every other module (both `_api` and private) | Private feature modules and `main.dart` |

### 1.4 Barrel File Rules

**Private modules:** Every folder inside `src/` MUST have a barrel file (named after the folder) that exports all files in that folder. The module root barrel file (e.g., `auth.dart`) exports the barrel files for bootstrap and pages only (what the app shell needs).

```dart
// lib/features/auth/auth/lib/src/blocs/blocs.dart
export 'auth_bloc.dart';
export 'auth_event.dart';
export 'auth_state.dart';
```

```dart
// lib/features/auth/auth/lib/src/pages/pages.dart
export 'login_page.dart';
```

```dart
// lib/features/auth/auth/lib/auth.dart (root barrel)
export 'src/bootstrap/bootstrap.dart';
export 'src/pages/pages.dart';
```

**Public modules:** The root barrel file exports ONLY the abstractions and models that other modules need.

```dart
// lib/features/auth/auth_api/lib/auth_api.dart
export 'src/services/auth_service.dart';
export 'src/models/auth_user.dart';
```

### 1.5 File Organization Rules

- **One public class per file.** Private helper classes may coexist.
- **File names must match the primary class** using `snake_case`: `login_page.dart` for `LoginPage`.
- **Keep files under 300 lines.** Extract widgets or logic if a file exceeds this.

---

## 2. Naming Conventions

### 2.1 General Rules

| Element | Convention | Example |
| --- | --- | --- |
| Classes | `UpperCamelCase` | `UserProfilePage` |
| Extensions | `UpperCamelCase` | `StringExtension` |
| Mixins | `UpperCamelCase` | `ValidationMixin` |
| Enums | `UpperCamelCase` | `AuthStatus` |
| Enum values | `lowerCamelCase` | `AuthStatus.authenticated` |
| Variables | `lowerCamelCase` | `userName` |
| Constants | `lowerCamelCase` | `defaultPadding` |
| Private members | `_lowerCamelCase` | `_isLoading` |
| Files | `snake_case` | `user_profile_page.dart` |
| Directories | `snake_case` | `user_settings/` |
| Named parameters | `lowerCamelCase` | `isEnabled`, `onPressed` |
| Packages (public) | `snake_case` + `_api` | `core_api`, `auth_api`, `home_api` |
| Packages (private) | `snake_case` | `core`, `auth`, `home` |

### 2.2 Suffix Conventions

Use consistent suffixes for discoverability:

| Type | Suffix | Example |
| --- | --- | --- |
| Pages/Screens | `Page` | `LoginPage` |
| Widgets | (descriptive) | `UserAvatar`, `PriceTag` |
| Blocs | `Bloc` | `AuthBloc` |
| Bloc Events | `Event` (within bloc) | `LoginRequested` |
| Bloc States | `State` (within bloc) | `AuthAuthenticated` |
| Cubits | `Cubit` | `ThemeCubit` |
| Repositories | `Repository` | `AuthRepository` |
| Services (abstract) | `Service` | `AuthService` |
| Services (impl) | `ServiceImpl` | `AuthServiceImpl` |
| Models | `Model` | `UserModel` |
| Entities | (plain) | `User`, `AuthUser` |
| Exceptions | `Exception` | `AuthException` |
| Failures | `Failure` | `ServerFailure` |
| Bootstrap | `Bootstrap` | `AuthBootstrap` |
| Routes | `Routes` | `AuthRoutes` |

### 2.3 Boolean Naming

Prefix booleans with `is`, `has`, `can`, `should`, or `was`:

```dart
// ✅ Correct
bool isLoading = false;
bool hasPermission = true;
bool canSubmit = form.isValid;
bool shouldRefresh = cache.isExpired;

// ❌ Incorrect
bool loading = false;
bool permission = true;
bool submit = form.isValid;
```

---

## 3. Dependency Injection & Service Locator

### 3.1 GetIt as Service Locator

Use **GetIt** as the service locator. The service locator **only injects services** — repositories and blocs are NOT registered in GetIt.

```dart
// lib/core/core/lib/src/di/service_locator.dart
import 'package:get_it/get_it.dart';

/// Global service locator instance.
final sl = GetIt.instance;
```

### 3.2 Bootstrap Pattern

Each private module provides a **bootstrap** file that handles **both** service locator registration (GetIt) **and** route registration (auto_route). This is the single entry point for initializing a feature module.

```dart
// lib/features/auth/auth/lib/src/bootstrap/auth_bootstrap.dart
import 'package:auto_route/auto_route.dart';
import 'package:get_it/get_it.dart';
import 'package:auth_api/auth_api.dart';
import '../services/services.dart';
import '../routes/routes.dart';

/// Bootstraps the auth feature module.
///
/// Registers services into the service locator and
/// provides route definitions for the app router.
class AuthBootstrap {
  const AuthBootstrap._();

  /// Registers all auth services into the service locator.
  static void registerServices(GetIt sl) {
    sl.registerLazySingleton<AuthService>(
      () => AuthServiceImpl(),
    );
  }

  /// Returns the auto_route definitions for this feature.
  static List<AutoRoute> routes() {
    return AuthRoutes.routes;
  }
}
```

```dart
// lib/features/auth/auth/lib/src/routes/auth_routes.dart
import 'package:auto_route/auto_route.dart';
import '../pages/pages.dart';

/// Route definitions for the auth feature.
class AuthRoutes {
  const AuthRoutes._();

  static List<AutoRoute> get routes => [
    AutoRoute(page: LoginRoute.page, path: '/login'),
  ];
}
```

The core private module also has a bootstrap that sets up the root router and core services:

```dart
// lib/core/core/lib/src/bootstrap/core_bootstrap.dart
import 'package:get_it/get_it.dart';
import '../services/services.dart';
import '../network/network.dart';

/// Bootstraps the core module.
class CoreBootstrap {
  const CoreBootstrap._();

  /// Registers all core services into the service locator.
  static void registerServices(GetIt sl) {
    sl.registerLazySingleton<DioClient>(
      () => DioClient(),
    );
    // ... other core service registrations
  }
}
```

### 3.3 App-Level Bootstrap

In `main.dart`, call all module bootstraps for services and assemble routes into the root router:

```dart
// lib/main.dart
import 'package:core/core.dart'; // provides `sl`, CoreBootstrap, AppRouter
import 'package:auth/auth.dart';
import 'package:home/home.dart';

void main() {
  _bootstrap();
  runApp(App(router: AppRouter()));
}

void _bootstrap() {
  // Register services (order matters for dependencies)
  CoreBootstrap.registerServices(sl);
  AuthBootstrap.registerServices(sl);
  HomeBootstrap.registerServices(sl);
}
```

### 3.4 Service Locator Rules

- **Only services are registered** in GetIt. Repositories, blocs, and cubits are NOT registered.
- **Abstractions live in the public module** (`_api`). Implementations live in the private module.
- **Always register against the abstract type**, never the implementation type.
- **Use `registerLazySingleton`** for services that should be created once on first access.
- **Use `registerFactory`** for services that need a fresh instance each time.
- **Never call `sl<T>()` from widgets directly.** Access services through repositories or blocs.

### 3.5 Routing with auto_route

Use **auto_route** for declarative, type-safe routing. The root router lives in the core private module. Each feature module contributes its routes via the bootstrap.

#### Root Router

```dart
// lib/core/core/lib/src/router/app_router.dart
import 'package:auto_route/auto_route.dart';
import 'package:auth/auth.dart';
import 'package:home/home.dart';

part 'app_router.gr.dart';

@AutoRouterConfig()
class AppRouter extends RootStackRouter {
  @override
  List<AutoRoute> get routes => [
    ...AuthBootstrap.routes(),
    ...HomeBootstrap.routes(),
  ];

  @override
  RouteType get defaultRouteType => const RouteType.material();
}
```

> **Rule:** Run `dart run build_runner build` after adding or changing `@RoutePage()` annotations to regenerate the `.gr.dart` files.

#### Page Annotation

Every page widget MUST be annotated with `@RoutePage()` for auto_route code generation:

```dart
// lib/features/auth/auth/lib/src/pages/login_page.dart
import 'package:auto_route/auto_route.dart';

@RoutePage()
class LoginPage extends StatelessWidget {
  const LoginPage({super.key});
  ...
}
```

#### App Widget with auto_route

```dart
// lib/app/app.dart
import 'package:auto_route/auto_route.dart';
import 'package:core/core.dart';
import 'package:flutter/material.dart';

class App extends StatelessWidget {
  const App({super.key, required this.router});

  final AppRouter router;

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      routerConfig: router.config(),
    );
  }
}
```

#### Routing Rules

- **Always use `context.router`** for navigation — never use `Navigator` directly.
- **Define routes in the feature's `routes/` folder**, not in pages.
- **Expose routes via the bootstrap** — the root router collects them from all bootstraps.
- **Use path parameters** for dynamic routes: `AutoRoute(page: UserRoute.page, path: '/users/:id')`.

### 3.6 Dio HTTP Client

Use **Dio** for all HTTP/API requests. The Dio client is configured in the core private module and registered via the service locator.

```dart
// lib/core/core/lib/src/network/dio_client.dart
import 'package:dio/dio.dart';

/// Configured Dio HTTP client for all API requests.
class DioClient {
  DioClient({
    String? baseUrl,
  }) : _dio = Dio(
    BaseOptions(
      baseUrl: baseUrl ?? const String.fromEnvironment('API_BASE_URL'),
      connectTimeout: const Duration(seconds: 15),
      receiveTimeout: const Duration(seconds: 15),
      headers: {'Content-Type': 'application/json'},
    ),
  );

  final Dio _dio;

  /// Access the underlying Dio instance for advanced configuration.
  Dio get dio => _dio;

  Future<Response<T>> get<T>(String path, {Map<String, dynamic>? queryParameters}) {
    return _dio.get(path, queryParameters: queryParameters);
  }

  Future<Response<T>> post<T>(String path, {Object? data}) {
    return _dio.post(path, data: data);
  }

  Future<Response<T>> put<T>(String path, {Object? data}) {
    return _dio.put(path, data: data);
  }

  Future<Response<T>> delete<T>(String path) {
    return _dio.delete(path);
  }
}
```

#### Dio Rules

- **Wrap Dio calls in try/catch** at the service layer — convert `DioException` into domain exceptions.
- **Use interceptors** for auth tokens, logging, and retry logic.
- **Never expose `Dio` or `Response` types** outside the service layer — return domain models instead.

---

## 4. State Management

### 4.1 BLoC Pattern Architecture

The state management follows the **BLoC pattern** with a clear layered architecture:

```
Page/Widget → Bloc/Cubit → Repository → Service (from GetIt)
```

- **Pages and Widgets** consume blocs/cubits via `BlocProvider` and `BlocBuilder`/`BlocListener`.
- **Blocs and Cubits** contain business logic and depend on repositories.
- **Repositories** orchestrate data operations and depend on services.
- **Services** are injected via GetIt and handle external communication (API calls, local storage, etc.).

### 4.2 Bloc Implementation

```dart
// lib/features/auth/auth/lib/src/blocs/auth_event.dart
sealed class AuthEvent {
  const AuthEvent();
}

final class LoginRequested extends AuthEvent {
  const LoginRequested({required this.email, required this.password});
  final String email;
  final String password;
}

final class LogoutRequested extends AuthEvent {
  const LogoutRequested();
}
```

```dart
// lib/features/auth/auth/lib/src/blocs/auth_state.dart
sealed class AuthState {
  const AuthState();
}

final class AuthInitial extends AuthState {
  const AuthInitial();
}

final class AuthLoading extends AuthState {
  const AuthLoading();
}

final class AuthAuthenticated extends AuthState {
  const AuthAuthenticated({required this.user});
  final AuthUser user;
}

final class AuthUnauthenticated extends AuthState {
  const AuthUnauthenticated();
}

final class AuthFailure extends AuthState {
  const AuthFailure({required this.message});
  final String message;
}
```

```dart
// lib/features/auth/auth/lib/src/blocs/auth_bloc.dart
import 'package:flutter_bloc/flutter_bloc.dart';
import '../repositories/repositories.dart';
import 'auth_event.dart';
import 'auth_state.dart';

class AuthBloc extends Bloc<AuthEvent, AuthState> {
  AuthBloc({required AuthRepository authRepository})
      : _authRepository = authRepository,
        super(const AuthInitial()) {
    on<LoginRequested>(_onLoginRequested);
    on<LogoutRequested>(_onLogoutRequested);
  }

  final AuthRepository _authRepository;

  Future<void> _onLoginRequested(
    LoginRequested event,
    Emitter<AuthState> emit,
  ) async {
    emit(const AuthLoading());
    final result = await _authRepository.signIn(
      email: event.email,
      password: event.password,
    );
    switch (result) {
      case Success(:final data):
        emit(AuthAuthenticated(user: data));
      case Failure(:final error):
        emit(AuthFailure(message: error.message));
    }
  }

  Future<void> _onLogoutRequested(
    LogoutRequested event,
    Emitter<AuthState> emit,
  ) async {
    await _authRepository.signOut();
    emit(const AuthUnauthenticated());
  }
}
```

### 4.3 Cubit Implementation

Use cubits for simpler state management where events are unnecessary:

```dart
// lib/features/auth/auth/lib/src/cubits/session_cubit.dart
import 'package:flutter_bloc/flutter_bloc.dart';

class SessionCubit extends Cubit<bool> {
  SessionCubit() : super(false);

  void setActive() => emit(true);
  void setInactive() => emit(false);
}
```

### 4.4 Hydrated Bloc

Use `HydratedBloc` or `HydratedCubit` for state that must persist across app restarts:

```dart
// lib/features/settings/settings/lib/src/cubits/theme_cubit.dart
import 'package:hydrated_bloc/hydrated_bloc.dart';

class ThemeCubit extends HydratedCubit<ThemeMode> {
  ThemeCubit() : super(ThemeMode.system);

  void setTheme(ThemeMode mode) => emit(mode);

  @override
  ThemeMode fromJson(Map<String, dynamic> json) {
    return ThemeMode.values[json['theme'] as int];
  }

  @override
  Map<String, dynamic> toJson(ThemeMode state) {
    return {'theme': state.index};
  }
}
```

Initialize `HydratedBloc` storage in `main.dart`:

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  HydratedBloc.storage = await HydratedStorage.build(
    storageDirectory: kIsWeb
        ? HydratedStorageDirectory.web
        : HydratedStorageDirectory((await getApplicationDocumentsDirectory()).path),
  );
  _registerServices();
  runApp(const App());
}
```

### 4.5 Repository Pattern

Repositories depend on services (obtained via GetIt) and are instantiated by blocs or provided via `RepositoryProvider`:

```dart
// lib/features/auth/auth/lib/src/repositories/auth_repository.dart
import 'package:auth_api/auth_api.dart';
import 'package:core_api/core_api.dart';

class AuthRepository {
  AuthRepository({required AuthService authService})
      : _authService = authService;

  final AuthService _authService;

  Future<Result<AuthUser>> signIn({
    required String email,
    required String password,
  }) async {
    try {
      final user = await _authService.signIn(email: email, password: password);
      return Success(user);
    } on Exception catch (e) {
      return Failure(AppException(e.toString()));
    }
  }

  Future<void> signOut() async {
    await _authService.signOut();
  }
}
```

### 4.6 Page with BlocProvider Wiring

```dart
// lib/features/auth/auth/lib/src/pages/login_page.dart
import 'package:auth_api/auth_api.dart';
import 'package:auto_route/auto_route.dart';
import 'package:core/core.dart'; // for `sl`
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../blocs/blocs.dart';
import '../repositories/repositories.dart';
import '../widgets/widgets.dart';

@RoutePage()
class LoginPage extends StatelessWidget {
  const LoginPage({super.key});

  @override
  Widget build(BuildContext context) {
    return RepositoryProvider(
      create: (_) => AuthRepository(authService: sl<AuthService>()),
      child: BlocProvider(
        create: (context) => AuthBloc(
          authRepository: context.read<AuthRepository>(),
        ),
        child: const _LoginView(),
      ),
    );
  }
}

class _LoginView extends StatelessWidget {
  const _LoginView();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Login')),
      body: BlocListener<AuthBloc, AuthState>(
        listener: (context, state) {
          if (state is AuthAuthenticated) {
            context.router.replaceNamed('/home');
          } else if (state is AuthFailure) {
            ScaffoldMessenger.of(context).showSnackBar(
              SnackBar(content: Text(state.message)),
            );
          }
        },
        child: const LoginForm(),
      ),
    );
  }
}
```

### 4.7 State Management Rules

- **Separate UI state from business logic.** Widgets should not contain business rules.
- **Keep state as local as possible.** Use `setState` or `ValueNotifier` for widget-local state.
- **Lift state only when shared** between sibling or distant widgets.
- **Never store UI-only state (scroll position, animation state) in blocs.**
- **All state objects must be immutable.** Use `const` constructors, `final` fields, and `copyWith` methods.
- **Use `sealed class` for events and states** to enable exhaustive pattern matching.
- **Events should be past-tense or imperative** (`LoginRequested`, `LogoutRequested`).
- **States should be nouns describing the current condition** (`AuthLoading`, `AuthAuthenticated`).

---

## 5. Public Module Design

### 5.1 Abstract Service Definition

Public modules (`_api`) define abstract service contracts that other features can depend on:

```dart
// lib/features/auth/auth_api/lib/src/services/auth_service.dart
import '../models/auth_user.dart';

/// Contract for authentication operations.
///
/// Implementations are registered in the service locator
/// by the auth private module.
abstract class AuthService {
  /// Signs in a user with [email] and [password].
  Future<AuthUser> signIn({
    required String email,
    required String password,
  });

  /// Signs out the current user.
  Future<void> signOut();

  /// Returns the currently authenticated user, or `null`.
  AuthUser? get currentUser;

  /// A stream of authentication state changes.
  Stream<AuthUser?> get authStateChanges;
}
```

### 5.2 Shared Models

Models that need to be shared across features live in the public module:

```dart
// lib/features/auth/auth_api/lib/src/models/auth_user.dart

/// Represents an authenticated user.
///
/// This model is shared across features that depend on auth.
class AuthUser {
  const AuthUser({
    required this.id,
    required this.email,
    this.displayName,
    this.photoUrl,
  });

  final String id;
  final String email;
  final String? displayName;
  final String? photoUrl;
}
```

### 5.3 Public Module Rules

- **NEVER put implementations in a public module.** Only abstractions and shared models.
- **Public modules should have zero or minimal dependencies** — ideally only `core_api` and Flutter SDK.
- **Every public API must be documented** with `///` doc comments.
- **The public module barrel file exports only what other features need** — nothing more.

---

## 6. Widget Composition

### 6.1 Composition Over Inheritance

- **Never extend `StatelessWidget` / `StatefulWidget` with custom base classes** to add behavior. Use composition, mixins, or hooks instead.
- **Extract widgets, not methods.** Turn `_buildHeader()` into a `_Header` widget.

```dart
// ❌ Incorrect — build helper methods
class ProfilePage extends StatelessWidget {
  Widget _buildHeader() => ...;
  Widget _buildBody() => ...;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [_buildHeader(), _buildBody()],
    );
  }
}

// ✅ Correct — extracted widgets
class ProfilePage extends StatelessWidget {
  const ProfilePage({super.key});

  @override
  Widget build(BuildContext context) {
    return const Column(
      children: [_ProfileHeader(), _ProfileBody()],
    );
  }
}

class _ProfileHeader extends StatelessWidget { ... }
class _ProfileBody extends StatelessWidget { ... }
```

### 6.2 Const Constructors

Always use `const` constructors when possible:

```dart
// ✅ Correct
class AppLogo extends StatelessWidget {
  const AppLogo({super.key, this.size = 48});
  final double size;
  ...
}

// Usage with const
const AppLogo(size: 32)
```

### 6.3 Widget Parameters

- Use **named parameters** for widgets with more than one parameter.
- Mark required parameters with `required`.
- Provide sensible defaults.
- Accept **callbacks** (`VoidCallback`, `ValueChanged<T>`) — not controllers — for parent-child communication.

```dart
// ✅ Correct
class ActionButton extends StatelessWidget {
  const ActionButton({
    super.key,
    required this.label,
    required this.onPressed,
    this.isLoading = false,
    this.icon,
  });

  final String label;
  final VoidCallback onPressed;
  final bool isLoading;
  final IconData? icon;
  ...
}
```

### 6.4 Keys

- Use `ValueKey` or `ObjectKey` for items in lists.
- Use `GlobalKey` sparingly — only when you need to access state or context across the tree.
- Always provide keys for items in `ListView.builder`, `AnimatedList`, and reorderable lists.

---

## 7. Error Handling

### 7.1 Result Types

Prefer returning result types over throwing exceptions for expected errors:

```dart
// ✅ Preferred — sealed class result type (in core_api package)
sealed class Result<T> {
  const Result();
}

class Success<T> extends Result<T> {
  const Success(this.data);
  final T data;
}

class Failure<T> extends Result<T> {
  const Failure(this.error);
  final AppException error;
}

// Usage in a repository
Future<Result<User>> getUser(String id) async {
  try {
    final response = await _api.fetchUser(id);
    return Success(response.toEntity());
  } on DioException catch (e) {
    return Failure(AppException.fromDioError(e));
  }
}
```

### 7.2 Exception Hierarchy

Define a structured exception hierarchy in the `core_api` package:

```dart
sealed class AppException implements Exception {
  const AppException(this.message);
  final String message;
}

class NetworkException extends AppException {
  const NetworkException([super.message = 'Network error occurred']);
}

class ServerException extends AppException {
  const ServerException(super.message, {this.statusCode});
  final int? statusCode;
}

class CacheException extends AppException {
  const CacheException([super.message = 'Cache error occurred']);
}

class ValidationException extends AppException {
  const ValidationException(super.message, {this.field});
  final String? field;
}
```

### 7.3 Error Handling Rules

- **Never silently catch exceptions.** At minimum, log them.
- **Never catch `Exception` or `Object` generically** without re-throwing unknown types.
- **Show user-facing error messages** — never expose raw stack traces or technical details.
- **Retry transient failures** (network timeouts) with exponential backoff.

---

## 8. Testing

### 8.1 Test File Structure

Each package has its own `test/` directory mirroring its `lib/src/` structure:

```
lib/features/auth/auth/
├── lib/
│   └── src/
│       ├── blocs/
│       ├── repositories/
│       └── pages/
└── test/
    ├── blocs/
    │   └── auth_bloc_test.dart
    ├── repositories/
    │   └── auth_repository_test.dart
    ├── pages/
    │   └── login_page_test.dart
    └── helpers/
        ├── mocks.dart
        ├── fakes.dart
        └── pump_app.dart
```

### 8.2 Test Naming

Use descriptive three-part names: `group` → `test` → `expectation`:

```dart
group('AuthRepository', () {
  group('signIn', () {
    test('returns User when credentials are valid', () async { ... });
    test('throws AuthException when password is wrong', () async { ... });
    test('throws NetworkException when offline', () async { ... });
  });
});
```

### 8.3 Test Principles

- **Unit test all business logic** — repositories, blocs/cubits.
- **Widget test critical UI flows** — form validation, navigation, error display.
- **Mock external dependencies** — use `mocktail` or `mockito`.
- **Never test framework code** — don't test that `setState` rebuilds a widget.
- **Use `setUp` and `tearDown`** for common initialization.
- **Prefer `pumpWidget` over `pump`** when testing widget behavior.
- **Test edge cases**: empty lists, null values, max-length strings.

### 8.4 Golden Tests

Use golden tests for design-critical widgets:

```dart
testWidgets('AppButton matches golden', (tester) async {
  await tester.pumpWidget(
    const MaterialApp(
      home: Scaffold(
        body: AppButton(label: 'Submit'),
      ),
    ),
  );

  await expectLater(
    find.byType(AppButton),
    matchesGoldenFile('goldens/app_button.png'),
  );
});
```

---

## 9. Performance

### 9.1 Build Optimization

- **Use `const` widgets** wherever possible to prevent unnecessary rebuilds.
- **Use `const` constructors** on custom widgets.
- **Avoid calling `setState` in `build`** — this triggers infinite rebuilds.
- **Use `RepaintBoundary`** around expensive-to-paint widgets (complex animations, canvases).

### 9.2 List Performance

```dart
// ❌ Incorrect — builds all items at once
ListView(
  children: items.map((item) => ItemTile(item: item)).toList(),
);

// ✅ Correct — lazily builds visible items only
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) => ItemTile(
    key: ValueKey(items[index].id),
    item: items[index],
  ),
);
```

### 9.3 Image Best Practices

- **Cache network images** using `cached_network_image`.
- **Use appropriate `fit` modes** — prefer `BoxFit.cover` for thumbnails.
- **Specify `width` and `height`** to avoid layout shifts.
- **Precache critical images** with `precacheImage`.

### 9.4 Async Best Practices

- **Cancel subscriptions in `dispose()`** to prevent memory leaks.
- **Use `CancelableOperation`** from `async` package for cancellable futures.
- **Debounce** search inputs and other high-frequency events.
- **Never use `Timer`** without cancelling it in `dispose`.
- **Cancel bloc subscriptions** — blocs handle this automatically via `close()`.

### 9.5 Avoid Unnecessary Rebuilds

- **Use `BlocSelector`** to listen to specific state slices.
- **Avoid nesting `BlocBuilder` inside `BlocBuilder`** — combine into one `BlocSelector` or `MultiBlocListener`.
- **Use `AnimatedBuilder` / `ListenableBuilder`** to scope rebuilds to animated sections.

---

## 10. General Dart Rules

### 10.1 Null Safety

- **Never use `!` (bang operator)** unless the non-null guarantee is provably true.
- **Prefer `?.` and `??`** over null checks with `if`.
- **Use late only when initialization is guaranteed** before access.

### 10.2 Collections

```dart
// ✅ Prefer collection literals
final list = <String>[];
final map = <String, int>{};
final set = <int>{};

// ❌ Avoid constructor calls
final list = List<String>();  // lint warning
```

### 10.3 String Interpolation

```dart
// ✅ Correct
final greeting = 'Hello, $name!';
final info = 'User ${user.name} has ${user.age} years';

// ❌ Incorrect
final greeting = 'Hello, ' + name + '!';
```

### 10.4 Imports

- **Always use relative imports** within the same package.
- **Use package imports** when importing from other packages in the workspace.
- **Sort imports**: dart core → dart libraries → package imports → relative imports.
- **Never use `show` / `hide`** unless resolving naming conflicts.

### 10.5 Documentation

- **Document all public APIs** with `///` doc comments.
- **Start doc comments with a single-sentence summary.**
- **Use `@param` annotations sparingly** — prefer self-documenting parameter names.

```dart
/// Authenticates a user with the given [email] and [password].
///
/// Returns a [User] on success, or throws [AuthException] on failure.
/// The [rememberMe] flag persists the session across app restarts.
Future<User> signIn({
  required String email,
  required String password,
  bool rememberMe = false,
}) async { ... }
```
