---
name: flutter-coding-standards
description: Comprehensive Flutter and Dart coding standards for building production-quality apps. Use this skill when writing, reviewing, or refactoring Flutter code to ensure consistent project structure, naming conventions, state management patterns, widget composition, testing practices, error handling, internationalization, clean code, SOLID principles, and performance optimization.
license: MIT
metadata:
  author: brenobattaglin
  version: "3.0.0"
  date: July 2026
  abstract: A complete set of Flutter and Dart coding standards organized across 12 categories — multi-package project structure with Dart workspaces, feature-first architecture with public/private module separation, BLoC/Cubit state management with GetIt service locator via core_api, naming conventions, widget composition, testing per package, error handling, internationalization (flutter_localizations + ARB), clean code, SOLID principles, and performance. Each section includes rationale, correct vs. incorrect examples, and actionable rules for AI agents to follow when generating, reviewing, or optimizing Flutter code.
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
  flutter_bloc: ^9.0.0
  hydrated_bloc: ^10.0.0
  auto_route: ^9.0.0
  dio: ^5.0.0
```

> **Rule:** Private feature modules MUST NOT depend on `core`. They depend only on `core_api` (and their own `_api` module). The `core` package is only used by `main.dart` for bootstrap and app-level wiring.

### 1.2 Feature-First Organization

Organize code by feature inside `lib/features/`. Each feature consists of up to **two packages**: a **public API module** (suffix `_api`) and a **private implementation module** (no suffix). The `core` follows the same split pattern. Each package (public and private) is an independent Dart package with its own `test/` folder.

```
project_root/
├── pubspec.yaml                          # Dart workspace definition
├── lib/
│   ├── core/
│   │   ├── core_api/                     # PUBLIC core (shared abstractions)
│   │   │   ├── pubspec.yaml
│   │   │   ├── test/                     # Tests for core_api
│   │   │   │   ├── errors/
│   │   │   │   │   └── result_test.dart
│   │   │   │   └── extensions/
│   │   │   │       └── string_ext_test.dart
│   │   │   └── lib/
│   │   │       ├── core_api.dart         # Barrel: exports shared contracts
│   │   │       └── src/
│   │   │           ├── di/
│   │   │           │   ├── di.dart            # Barrel file
│   │   │           │   └── service_locator.dart  # GetIt instance (`sl`)
│   │   │           ├── errors/
│   │   │           │   ├── errors.dart        # Barrel file
│   │   │           │   ├── app_exception.dart
│   │   │           │   └── result.dart
│   │   │           ├── extensions/
│   │   │           │   ├── extensions.dart    # Barrel file
│   │   │           │   ├── string_ext.dart
│   │   │           │   └── context_l10n_ext.dart
│   │   │           └── utils/
│   │   │               ├── utils.dart         # Barrel file
│   │   │               ├── date_utils.dart
│   │   │               └── app_constants.dart # Constants live here, not in a separate folder
│   │   └── core/                             # PRIVATE core (app implementations)
│   │       ├── pubspec.yaml
│   │       ├── l10n.yaml                     # Flutter gen-l10n config
│   │       ├── test/                         # Tests for core
│   │       │   ├── network/
│   │       │   │   └── dio_client_test.dart
│   │       │   └── services/
│   │       │       └── logger_service_impl_test.dart
│   │       └── lib/
│   │           ├── core.dart                 # Barrel: exports bootstrap + app-specific code
│   │           └── src/
│   │               ├── bootstrap/
│   │               │   ├── bootstrap.dart         # Barrel file
│   │               │   └── core_bootstrap.dart    # GetIt + route registration
│   │               ├── l10n/
│   │               │   ├── arb/
│   │               │   │   ├── app_en.arb         # English (template)
│   │               │   │   ├── app_pt.arb         # Portuguese
│   │               │   │   └── app_es.arb         # Spanish
│   │               │   └── generated/             # Auto-generated by gen-l10n
│   │               │       ├── app_localizations.dart
│   │               │       ├── app_localizations_en.dart
│   │               │       ├── app_localizations_pt.dart
│   │               │       └── app_localizations_es.dart
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
│   │   │   │   ├── test/                 # Tests for auth_api
│   │   │   │   │   └── models/
│   │   │   │   │       └── auth_user_test.dart
│   │   │   │   └── lib/
│   │   │   │       ├── auth_api.dart     # Barrel: exports ONLY public contracts
│   │   │   │       └── src/
│   │   │   │           ├── services/
│   │   │   │           │   └── auth_service.dart       # Abstract class
│   │   │   │           └── models/
│   │   │   │               └── auth_user.dart          # Shared model/entity
│   │   │   └── auth/                     # PRIVATE module (implementation)
│   │   │       ├── pubspec.yaml
│   │   │       ├── test/                 # Tests for auth private module
│   │   │       │   ├── api/
│   │   │       │   │   ├── auth_repository_test.dart
│   │   │       │   │   └── models/
│   │   │       │   │       └── auth_response_model_test.dart
│   │   │       │   ├── blocs/
│   │   │       │   │   └── auth_bloc_test.dart
│   │   │       │   ├── cubits/
│   │   │       │   │   └── session_cubit_test.dart
│   │   │       │   ├── pages/
│   │   │       │   │   └── login_page_test.dart
│   │   │       │   └── widgets/
│   │   │       │       └── login_form_test.dart
│   │   │       └── lib/
│   │   │           ├── auth.dart         # Barrel: exports bootstrap + pages
│   │   │           └── src/
│   │   │               ├── bootstrap/
│   │   │               │   ├── bootstrap.dart               # Barrel file
│   │   │               │   └── auth_bootstrap.dart          # GetIt + route registration
│   │   │               ├── services/
│   │   │               │   ├── services.dart               # Barrel file
│   │   │               │   └── auth_service_impl.dart      # Implements AuthService
│   │   │               ├── api/
│   │   │               │   ├── api.dart                    # Barrel file
│   │   │               │   ├── auth_repository.dart        # Repository
│   │   │               │   └── models/
│   │   │               │       ├── models.dart             # Barrel file
│   │   │               │       └── auth_response_model.dart  # API response model
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
│   │       │   ├── test/                 # Tests for home_api
│   │       │   └── ...                   # Same pattern as auth_api
│   │       └── home/
│   │           ├── test/                 # Tests for home private module
│   │           └── ...                   # Same pattern as auth
│   └── main.dart                         # App entry point
```

### 1.3 Public vs. Private Module Rules

This split applies to **both features and core**.

| Aspect | Public Module (`_api` suffix) | Private Module (no suffix) |
| --- | --- | --- |
| **Purpose** | Expose abstractions for cross-module use | Contain all implementation details |
| **Contains** | Abstract service classes, shared models/entities, extensions, utilities, constants | Service implementations, api (repository + models), blocs, cubits, pages, widgets, bootstrap |
| **Exports** | Only abstract classes and models needed by other modules | Bootstrap function and page widgets (for routing) |
| **Dependencies** | Minimal — Flutter SDK only | Can depend on its own `_api`, `core_api`, external packages |
| **Who depends on it** | Other modules (both public and private) that need the abstraction | The app shell / main.dart (for registration and routing) |

#### Core-specific split

| Aspect | `core_api` | `core` |
| --- | --- | --- |
| **Purpose** | Shared abstractions, models, utilities, constants, extensions, and the service locator instance — used across all features | App-specific implementations, Dio client, root router, core bootstrap, localization (ARB files + generated code) |
| **Contains** | `Result`, `AppException`, extension methods, constants, `sl` (GetIt instance), abstract service contracts (e.g., `LoggerService`), `context.l10n` extension | `CoreBootstrap`, `DioClient`, `AppRouter`, concrete service implementations, ARB files, generated `AppLocalizations` |
| **Who depends on it** | Every other module (both `_api` and private) | Only `main.dart` and the `App` widget |

> **Rule:** Private feature modules MUST NOT import `package:core/core.dart`. If a feature module needs the service locator (`sl`), it imports it from `package:core_api/core_api.dart`.

### 1.4 Barrel File Rules

**Private modules:** Every folder inside `src/` MUST have a barrel file (named after the folder) that exports all files in that folder. The module root barrel file (e.g., `auth.dart`) exports the barrel files for bootstrap and pages only (what the app shell needs).

```dart
// lib/features/auth/auth/lib/src/blocs/blocs.dart
export 'auth_bloc.dart';
export 'auth_event.dart';
export 'auth_state.dart';
```

```dart
// lib/features/auth/auth/lib/src/api/api.dart
export 'auth_repository.dart';
export 'models/models.dart';
```

```dart
// lib/features/auth/auth/lib/src/api/models/models.dart
export 'auth_response_model.dart';
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

- **One public class per file.** Private classes in the same file are acceptable.
- **File names must match the primary class** using `snake_case`: `login_page.dart` for `LoginPage`.
- **Keep files under 300 lines.** Extract widgets or logic if a file exceeds this.

### 1.6 Anti-Patterns

- **No `helpers/` folder.** "Helper" is an anti-pattern in this architecture — it violates single responsibility and leads to dumping ground classes. Extract logic into properly named services, extensions, or utils instead.
- **No `constants/` folder.** Constants belong in `utils/` alongside other utility code. Group related constants in domain-specific files within `utils/` (e.g., `app_constants.dart`, `api_constants.dart`).
- **No `core` dependency in private feature modules.** Private features access the service locator through `core_api`, never through `core`.

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
| API Response Models | `Model` | `AuthResponseModel` |
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

### 3.1 GetIt as Service Locator (via core_api)

Use **GetIt** as the service locator. The GetIt instance (`sl`) lives in **`core_api`** so that all modules can access it without depending on the private `core` module. The service locator **only injects services** — repositories and blocs are NOT registered in GetIt.

```dart
// lib/core/core_api/lib/src/di/service_locator.dart
import 'package:get_it/get_it.dart';

/// Global service locator instance.
///
/// Lives in core_api so all modules can access registered
/// services without depending on the private core module.
final sl = GetIt.instance;
```

> **Rule:** The `sl` instance is exported from `core_api`. Private feature modules import it via `package:core_api/core_api.dart`. They never import `package:core/core.dart`.

### 3.2 Bootstrap Pattern

Each private module provides a **bootstrap** file that handles **both** service locator registration (GetIt) **and** route registration (auto_route). This is the single entry point for initializing a feature module.

```dart
// lib/features/auth/auth/lib/src/bootstrap/auth_bootstrap.dart
import 'package:auto_route/auto_route.dart';
import 'package:get_it/get_it.dart';
import 'package:auth_api/auth_api.dart';
import '../services/services.dart';
import '../routes/routes.dart';

class AuthBootstrap {
  const AuthBootstrap._();

  static void registerServices(GetIt sl) {
    sl.registerLazySingleton<AuthService>(
      () => AuthServiceImpl(),
    );
  }

  static List<AutoRoute> routes() {
    return AuthRoutes.routes;
  }
}
```

```dart
// lib/features/auth/auth/lib/src/routes/auth_routes.dart
import 'package:auto_route/auto_route.dart';
import '../pages/pages.dart';

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

class CoreBootstrap {
  const CoreBootstrap._();

  static void registerServices(GetIt sl) {
    sl.registerLazySingleton<DioClient>(
      () => DioClient(),
    );
  }
}
```

### 3.3 App-Level Bootstrap

In `main.dart`, call all module bootstraps for services and assemble routes into the root router:

```dart
// lib/main.dart
import 'package:core/core.dart';
import 'package:core_api/core_api.dart';
import 'package:auth/auth.dart';
import 'package:home/home.dart';

void main() {
  _bootstrap();
  runApp(App(router: AppRouter()));
}

void _bootstrap() {
  CoreBootstrap.registerServices(sl);
  AuthBootstrap.registerServices(sl);
  HomeBootstrap.registerServices(sl);
}
```

> **Note:** `main.dart` is the only place that imports both `core` and `core_api`. Feature modules only import `core_api`.

### 3.4 Service Locator Rules

- **Only services are registered** in GetIt. Repositories, blocs, and cubits are NOT registered.
- **Abstractions live in the public module** (`_api`). Implementations live in the private module.
- **Always register against the abstract type**, never the implementation type.
- **Use `registerLazySingleton`** for services that should be created once on first access.
- **Use `registerFactory`** for services that need a fresh instance each time.
- **Never call `sl<T>()` from widgets directly.** Access services through repositories or blocs.
- **The `sl` instance lives in `core_api`** — private features never depend on `core` to get the service locator.

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

#### App Widget with auto_route and Localization

```dart
// lib/app/app.dart
import 'package:auto_route/auto_route.dart';
import 'package:core/core.dart';
import 'package:flutter/material.dart';
import 'package:flutter_localizations/flutter_localizations.dart';

class App extends StatelessWidget {
  const App({super.key, required this.router});

  final AppRouter router;

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      routerConfig: router.config(),
      localizationsDelegates: const [
        AppLocalizations.delegate,
        GlobalMaterialLocalizations.delegate,
        GlobalWidgetsLocalizations.delegate,
        GlobalCupertinoLocalizations.delegate,
      ],
      supportedLocales: AppLocalizations.supportedLocales,
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
Page/Widget → Bloc/Cubit → Repository → Service (from GetIt via core_api)
```

- **Pages and Widgets** consume blocs/cubits via `BlocProvider` and `BlocBuilder`/`BlocListener`.
- **Blocs and Cubits** contain business logic and depend on repositories.
- **Repositories** live in the `api/` folder, orchestrate data operations, and depend on services.
- **Services** are injected via GetIt (accessed through `core_api`) and handle external communication (API calls, local storage, etc.).

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
import '../api/api.dart';
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
  _bootstrap();
  runApp(const App());
}
```

### 4.5 API Folder & Repository Pattern

The `api/` folder inside a private feature module contains the **repository**, its **barrel file**, and a **`models/` subfolder** for API response/request models.

```
src/
└── api/
    ├── api.dart                       # Barrel file
    ├── auth_repository.dart           # Repository
    └── models/
        ├── models.dart                # Barrel file
        └── auth_response_model.dart   # API response model
```

Repositories depend on services (obtained via GetIt through `core_api`) and are instantiated by blocs or provided via `RepositoryProvider`:

```dart
// lib/features/auth/auth/lib/src/api/auth_repository.dart
import 'package:auth_api/auth_api.dart';
import 'package:core_api/core_api.dart';
import 'models/models.dart';

class AuthRepository {
  AuthRepository({required AuthService authService})
      : _authService = authService;

  final AuthService _authService;

  Future<Result<AuthUser>> signIn({
    required String email,
    required String password,
  }) async {
    try {
      final response = await _authService.signIn(email: email, password: password);
      return Success(response);
    } on Exception catch (e) {
      return Failure(AppException(e.toString()));
    }
  }

  Future<void> signOut() async {
    await _authService.signOut();
  }
}
```

API response models live in the `models/` subfolder and are responsible for serialization/deserialization of API data:

```dart
// lib/features/auth/auth/lib/src/api/models/auth_response_model.dart
import 'package:auth_api/auth_api.dart';

class AuthResponseModel {
  const AuthResponseModel({
    required this.id,
    required this.email,
    this.displayName,
    this.token,
  });

  factory AuthResponseModel.fromJson(Map<String, dynamic> json) {
    return AuthResponseModel(
      id: json['id'] as String,
      email: json['email'] as String,
      displayName: json['display_name'] as String?,
      token: json['token'] as String?,
    );
  }

  final String id;
  final String email;
  final String? displayName;
  final String? token;

  AuthUser toEntity() {
    return AuthUser(
      id: id,
      email: email,
      displayName: displayName,
    );
  }

  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'email': email,
      'display_name': displayName,
      'token': token,
    };
  }
}
```

### 4.6 Page with BlocProvider Wiring

```dart
// lib/features/auth/auth/lib/src/pages/login_page.dart
import 'package:auth_api/auth_api.dart';
import 'package:auto_route/auto_route.dart';
import 'package:core_api/core_api.dart';
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../blocs/blocs.dart';
import '../api/api.dart';
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
      appBar: AppBar(title: Text(context.l10n.loginTitle)),
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

> **Note:** The page imports `sl` from `core_api`, not from `core`. This is the correct dependency direction.

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

abstract class AuthService {
  Future<AuthUser> signIn({
    required String email,
    required String password,
  });

  Future<void> signOut();

  AuthUser? get currentUser;

  Stream<AuthUser?> get authStateChanges;
}
```

### 5.2 Shared Models

Models that need to be shared across features live in the public module:

```dart
// lib/features/auth/auth_api/lib/src/models/auth_user.dart

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
- **Public modules should have zero or minimal dependencies** — ideally only Flutter SDK.
- **Every public API must be documented** with `///` doc comments.
- **The public module barrel file exports only what other features need** — nothing more.

---

## 6. Widget Composition

### 6.1 Composition Over Inheritance

- **Never extend `StatelessWidget` / `StatefulWidget` with custom base classes** to add behavior. Use composition, mixins, or hooks instead.
- **Extract widgets, not methods.** Turn `_buildHeader()` into a `_Header` widget.

```dart
// ❌ Incorrect — build methods are an anti-pattern
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

## 8. Internationalization (i18n)

### 8.1 Approach

Use the **official Flutter localization stack**: `flutter_localizations` + `intl` + ARB files with `flutter gen-l10n`. This is the market standard, supported by all major Translation Management Systems (Lokalise, Crowdin, Phrase, Weblate), and produces compile-time type-safe localization classes with zero runtime overhead.

### 8.2 Architecture in Multi-Package Monorepo

Localization follows the **centralized core** strategy:

- **ARB files and generated code** live in the **private `core`** module (`lib/core/core/lib/src/l10n/`).
- **The `context.l10n` extension** lives in **`core_api`** so all feature modules can access localized strings without depending on `core`.
- **`AppLocalizations`** is re-exported from `core.dart` for use in `MaterialApp` configuration.

### 8.3 Configuration

#### `l10n.yaml` (in `core` package root)

```yaml
arb-dir: lib/src/l10n/arb
template-arb-file: app_en.arb
output-localization-file: app_localizations.dart
output-dir: lib/src/l10n/generated
output-class: AppLocalizations
synthetic-package: false
nullable-getter: false
```

#### `core/pubspec.yaml` additions

```yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_localizations:
    sdk: flutter
  intl: ^0.20.0

flutter:
  generate: true
```

### 8.4 ARB File Format

ARB files use the ICU MessageFormat standard. The English file (`app_en.arb`) is the **template** — all keys must be defined here first.

```json
{
  "@@locale": "en",

  "appTitle": "My App",
  "@appTitle": {
    "description": "The title of the application"
  },

  "loginTitle": "Welcome Back",
  "@loginTitle": {
    "description": "Header title on the login screen"
  },

  "loginButton": "Sign In",
  "@loginButton": {
    "description": "Label for the login button"
  },

  "welcomeMessage": "Welcome, {name}!",
  "@welcomeMessage": {
    "description": "Greeting with user name",
    "placeholders": {
      "name": {
        "type": "String",
        "example": "John"
      }
    }
  },

  "itemCount": "{count, plural, =0{No items} =1{1 item} other{{count} items}}",
  "@itemCount": {
    "description": "Label showing the number of items",
    "placeholders": {
      "count": {
        "type": "int"
      }
    }
  },

  "lastUpdated": "Last updated: {date}",
  "@lastUpdated": {
    "description": "Shows when data was last updated",
    "placeholders": {
      "date": {
        "type": "DateTime",
        "format": "yMMMd"
      }
    }
  },

  "totalPrice": "Total: {price}",
  "@totalPrice": {
    "description": "Shows the total price",
    "placeholders": {
      "price": {
        "type": "double",
        "format": "currency",
        "optionalParameters": {
          "symbol": "$",
          "decimalDigits": 2
        }
      }
    }
  }
}
```

#### Translation file (`app_pt.arb`)

```json
{
  "@@locale": "pt",
  "appTitle": "Meu App",
  "loginTitle": "Bem-vindo de volta",
  "loginButton": "Entrar",
  "welcomeMessage": "Bem-vindo, {name}!",
  "itemCount": "{count, plural, =0{Nenhum item} =1{1 item} other{{count} itens}}",
  "lastUpdated": "Última atualização: {date}",
  "totalPrice": "Total: {price}"
}
```

### 8.5 The `context.l10n` Extension

Defined in `core_api` for convenient, concise access across all modules:

```dart
// lib/core/core_api/lib/src/extensions/context_l10n_ext.dart
import 'package:core/core.dart';
import 'package:flutter/widgets.dart';

extension BuildContextL10nX on BuildContext {
  AppLocalizations get l10n => AppLocalizations.of(this);
}
```

### 8.6 Usage in Widgets

```dart
import 'package:core_api/core_api.dart';
import 'package:flutter/material.dart';

@RoutePage()
class LoginPage extends StatelessWidget {
  const LoginPage({super.key});

  @override
  Widget build(BuildContext context) {
    final l10n = context.l10n;

    return Scaffold(
      appBar: AppBar(title: Text(l10n.loginTitle)),
      body: Column(
        children: [
          Text(l10n.welcomeMessage('John')),
          Text(l10n.itemCount(5)),
          ElevatedButton(
            onPressed: () {},
            child: Text(l10n.loginButton),
          ),
        ],
      ),
    );
  }
}
```

### 8.7 Adding a New Language

1. Create a new ARB file: `app_es.arb` with `"@@locale": "es"` and all translated keys.
2. Run `flutter gen-l10n` (or `flutter pub get` if `generate: true` is set).
3. The generated `AppLocalizations.supportedLocales` automatically includes the new locale — no manual `MaterialApp` changes needed.

For regional variants (e.g., `pt_BR` vs `pt_PT`), create separate ARB files with `"@@locale": "pt_BR"` and `"@@locale": "pt_PT"`.

### 8.8 Internationalization Rules

- **Never hardcode user-facing strings.** Always use `context.l10n.keyName`.
- **Use descriptive ARB key names** in `lowerCamelCase` (e.g., `loginButton`, not `login_btn`).
- **Always include `@key` metadata** with descriptions in the template ARB file.
- **Use ICU MessageFormat** for plurals, gender (select), dates, and currencies.
- **Never pass `BuildContext` or `AppLocalizations` to BLoC/Cubit layers.** Blocs emit domain objects (enums, error types). Map to localized strings in the UI layer only.
- **Run `flutter gen-l10n`** after any ARB file change.

---

## 9. Clean Code

### 9.1 No Comments in Code

Code MUST be self-documenting. Do not write inline comments or block comments to explain what the code does. If code needs a comment to be understood, it needs to be refactored.

```dart
// ❌ Incorrect — commented code
class AuthRepository {
  Future<Result<AuthUser>> signIn({
    required String email,
    required String password,
  }) async {
    try {
      // Call the auth service to sign in the user
      final user = await _authService.signIn(
        email: email,
        password: password,
      );
      // Return success result with the user
      return Success(user);
    } on Exception catch (e) {
      // Return failure result with the exception
      return Failure(AppException(e.toString()));
    }
  }
}

// ✅ Correct — self-documenting code, no comments needed
class AuthRepository {
  Future<Result<AuthUser>> signIn({
    required String email,
    required String password,
  }) async {
    try {
      final user = await _authService.signIn(
        email: email,
        password: password,
      );
      return Success(user);
    } on Exception catch (e) {
      return Failure(AppException(e.toString()));
    }
  }
}
```

**Exceptions:** `///` doc comments on public APIs are required (they are documentation, not comments). Comments explaining *why* (not *what*) a non-obvious design decision was made are acceptable in rare cases.

### 9.2 No Magic Numbers

Never use raw numeric or string literals in code. Extract them into well-named constants in the `utils/` folder or as static class members.

```dart
// ❌ Incorrect — magic numbers
class DioClient {
  DioClient() : _dio = Dio(
    BaseOptions(
      connectTimeout: Duration(seconds: 15),
      receiveTimeout: Duration(seconds: 15),
    ),
  );
}

EdgeInsets.all(16.0)
if (password.length < 8) { ... }
opacity: 0.87

// ✅ Correct — named constants
// In utils/app_constants.dart
class NetworkConstants {
  const NetworkConstants._();

  static const connectionTimeout = Duration(seconds: 15);
  static const receiveTimeout = Duration(seconds: 15);
}

class LayoutConstants {
  const LayoutConstants._();

  static const defaultPadding = EdgeInsets.all(16.0);
}

class ValidationConstants {
  const ValidationConstants._();

  static const minPasswordLength = 8;
}

class OpacityConstants {
  const OpacityConstants._();

  static const highEmphasis = 0.87;
}

// Usage
DioClient() : _dio = Dio(
  BaseOptions(
    connectTimeout: NetworkConstants.connectionTimeout,
    receiveTimeout: NetworkConstants.receiveTimeout,
  ),
);

Padding(padding: LayoutConstants.defaultPadding)
if (password.length < ValidationConstants.minPasswordLength) { ... }
Opacity(opacity: OpacityConstants.highEmphasis)
```

### 9.3 Self-Documenting Code

Write code that communicates its intent through:

- **Descriptive variable names:** `remainingAttempts` not `n` or `count`.
- **Descriptive method names:** `fetchActiveUsersByOrganization()` not `getUsers()`.
- **Small, focused functions:** Each function does one thing. If you need to describe a function with "and", split it.
- **Consistent abstraction levels:** A method should not mix high-level orchestration with low-level details.

```dart
// ❌ Incorrect — unclear intent
Future<void> process(List<Map<String, dynamic>> d) async {
  for (final item in d) {
    if (item['s'] == 1 && item['a'] > 18) {
      await _repo.save(item);
    }
  }
}

// ✅ Correct — self-documenting
Future<void> saveEligibleUsers(List<UserData> candidates) async {
  final eligibleUsers = candidates.where(
    (user) => user.isActive && user.isAdult,
  );

  for (final user in eligibleUsers) {
    await _userRepository.save(user);
  }
}
```

### 9.4 Clean Code Rules

- **Never leave commented-out code** in the codebase. Delete it — version control preserves history.
- **Never use abbreviations** in names unless they are universally understood (`id`, `url`, `http`).
- **Avoid deep nesting.** Use early returns and guard clauses to flatten control flow.
- **Prefer immutability.** Use `final` for all variables that are not reassigned.
- **Keep methods short.** If a method exceeds 20 lines, consider extracting logic.
- **Minimize function parameters.** If a function has more than 3-4 parameters, consider grouping them into a class.

---

## 10. SOLID Principles

### 10.1 Single Responsibility Principle (SRP)

Each class should have one reason to change. In this architecture:

- **Services** handle external communication (API, storage).
- **Repositories** orchestrate data operations and error handling.
- **Blocs/Cubits** manage state transitions.
- **Widgets** render UI.

```dart
// ❌ Incorrect — bloc doing too much
class AuthBloc extends Bloc<AuthEvent, AuthState> {
  Future<void> _onLogin(LoginRequested event, Emitter<AuthState> emit) async {
    final response = await Dio().post('/auth/login', data: {
      'email': event.email,
      'password': event.password,
    });
    final user = AuthUser.fromJson(response.data);
    await SharedPreferences.getInstance()
      ..setString('token', response.data['token']);
    emit(AuthAuthenticated(user: user));
  }
}

// ✅ Correct — each layer has one responsibility
class AuthBloc extends Bloc<AuthEvent, AuthState> {
  AuthBloc({required AuthRepository authRepository})
      : _authRepository = authRepository,
        super(const AuthInitial()) {
    on<LoginRequested>(_onLogin);
  }

  final AuthRepository _authRepository;

  Future<void> _onLogin(LoginRequested event, Emitter<AuthState> emit) async {
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
}
```

### 10.2 Open/Closed Principle (OCP)

Classes should be open for extension, closed for modification. Use abstract classes and sealed types.

```dart
// ✅ New error types can be added without modifying existing code
sealed class AppException implements Exception {
  const AppException(this.message);
  final String message;
}

class NetworkException extends AppException { ... }
class ServerException extends AppException { ... }
class CacheException extends AppException { ... }
```

### 10.3 Liskov Substitution Principle (LSP)

Subtypes must be substitutable for their base types. Service implementations must honor the full contract of their abstract class.

```dart
// ✅ Any AuthService implementation can be swapped without breaking consumers
abstract class AuthService {
  Future<AuthUser> signIn({required String email, required String password});
  Future<void> signOut();
}

class FirebaseAuthServiceImpl implements AuthService { ... }
class MockAuthServiceImpl implements AuthService { ... }
```

### 10.4 Interface Segregation Principle (ISP)

Clients should not be forced to depend on interfaces they don't use. Split large interfaces into focused ones.

```dart
// ❌ Incorrect — fat interface
abstract class UserService {
  Future<User> getUser(String id);
  Future<void> updateUser(User user);
  Future<void> deleteUser(String id);
  Future<List<User>> searchUsers(String query);
  Future<void> uploadAvatar(String userId, File file);
  Future<UserStats> getUserStats(String userId);
}

// ✅ Correct — segregated interfaces
abstract class UserReadService {
  Future<User> getUser(String id);
  Future<List<User>> searchUsers(String query);
}

abstract class UserWriteService {
  Future<void> updateUser(User user);
  Future<void> deleteUser(String id);
}

abstract class UserMediaService {
  Future<void> uploadAvatar(String userId, File file);
}
```

### 10.5 Dependency Inversion Principle (DIP)

High-level modules should not depend on low-level modules. Both should depend on abstractions.

This is enforced by the public/private module split:

- **Public modules** define abstractions (`AuthService`).
- **Private modules** provide implementations (`AuthServiceImpl`).
- **Consumers depend on the abstraction** (`auth_api`), never on the implementation (`auth`).
- **The service locator** registers implementations against abstract types.

```dart
// ✅ Repository depends on abstraction, not implementation
class AuthRepository {
  AuthRepository({required AuthService authService})
      : _authService = authService;

  final AuthService _authService;
}

// Registration in bootstrap (the only place that knows the implementation)
sl.registerLazySingleton<AuthService>(() => AuthServiceImpl());
```

---

## 11. Testing

### 11.1 Test Folder Per Package

Each package (both public and private) MUST have its own `test/` directory at the package root, mirroring the `lib/src/` structure. Tests are part of the package.

```
lib/features/auth/auth/                # Private module package
├── pubspec.yaml
├── lib/
│   └── src/
│       ├── api/
│       ├── blocs/
│       ├── cubits/
│       ├── pages/
│       └── widgets/
└── test/                              # Tests for this package
    ├── api/
    │   ├── auth_repository_test.dart
    │   └── models/
    │       └── auth_response_model_test.dart
    ├── blocs/
    │   └── auth_bloc_test.dart
    ├── cubits/
    │   └── session_cubit_test.dart
    ├── pages/
    │   └── login_page_test.dart
    └── widgets/
        └── login_form_test.dart
```

```
lib/features/auth/auth_api/           # Public module package
├── pubspec.yaml
├── lib/
│   └── src/
│       ├── models/
│       └── services/
└── test/                              # Tests for public module
    └── models/
        └── auth_user_test.dart
```

> **Rule:** No `helpers/` folder in tests. Test setup code goes in `setUp()` / `tearDown()`. Shared mocks and fakes are organized in a `test/fixtures/` folder with files like `mocks.dart`, `fakes.dart`, and `pump_app.dart`.

### 11.2 Test Fixtures Folder

Instead of a `helpers/` folder, use a `fixtures/` folder for shared test utilities:

```
test/
├── fixtures/
│   ├── mocks.dart         # Mock class declarations
│   ├── fakes.dart         # Fake implementations
│   └── pump_app.dart      # Widget test helpers (pumpApp extension)
├── api/
│   └── auth_repository_test.dart
└── blocs/
    └── auth_bloc_test.dart
```

### 11.3 Test Naming

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

### 11.4 Test Principles

- **Unit test all business logic** — repositories, blocs/cubits, models.
- **Widget test critical UI flows** — form validation, navigation, error display.
- **Mock external dependencies** — use `mocktail` or `mockito`.
- **Never test framework code** — don't test that `setState` rebuilds a widget.
- **Use `setUp` and `tearDown`** for common initialization.
- **Prefer `pumpWidget` over `pump`** when testing widget behavior.
- **Test edge cases**: empty lists, null values, max-length strings.
- **Test public modules** — model serialization, equality, and any utility/extension logic.

### 11.5 Golden Tests

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

## 12. Performance

### 12.1 Build Optimization

- **Use `const` widgets** wherever possible to prevent unnecessary rebuilds.
- **Use `const` constructors** on custom widgets.
- **Avoid calling `setState` in `build`** — this triggers infinite rebuilds.
- **Use `RepaintBoundary`** around expensive-to-paint widgets (complex animations, canvases).

### 12.2 List Performance

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

### 12.3 Image Best Practices

- **Cache network images** using `cached_network_image`.
- **Use appropriate `fit` modes** — prefer `BoxFit.cover` for thumbnails.
- **Specify `width` and `height`** to avoid layout shifts.
- **Precache critical images** with `precacheImage`.

### 12.4 Async Best Practices

- **Cancel subscriptions in `dispose()`** to prevent memory leaks.
- **Use `CancelableOperation`** from `async` package for cancellable futures.
- **Debounce** search inputs and other high-frequency events.
- **Never use `Timer`** without cancelling it in `dispose`.
- **Cancel bloc subscriptions** — blocs handle this automatically via `close()`.

### 12.5 Avoid Unnecessary Rebuilds

- **Use `BlocSelector`** to listen to specific state slices.
- **Avoid nesting `BlocBuilder` inside `BlocBuilder`** — combine into one `BlocSelector` or `MultiBlocListener`.
- **Use `AnimatedBuilder` / `ListenableBuilder`** to scope rebuilds to animated sections.

---

## 13. General Dart Rules

### 13.1 Null Safety

- **Never use `!` (bang operator)** unless the non-null guarantee is provably true.
- **Prefer `?.` and `??`** over null checks with `if`.
- **Use late only when initialization is guaranteed** before access.

### 13.2 Collections

```dart
// ✅ Prefer collection literals
final list = <String>[];
final map = <String, int>{};
final set = <int>{};

// ❌ Avoid constructor calls
final list = List<String>();  // lint warning
```

### 13.3 String Interpolation

```dart
// ✅ Correct
final greeting = 'Hello, $name!';
final info = 'User ${user.name} has ${user.age} years';

// ❌ Incorrect
final greeting = 'Hello, ' + name + '!';
```

### 13.4 Imports

- **Always use relative imports** within the same package.
- **Use package imports** when importing from other packages in the workspace.
- **Sort imports**: dart core → dart libraries → package imports → relative imports.
- **Never use `show` / `hide`** unless resolving naming conflicts.

### 13.5 Documentation

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

> **Note:** `///` doc comments on public APIs are documentation, not code comments. They are required. Inline `//` comments explaining *what* code does are prohibited — the code should be self-explanatory.
