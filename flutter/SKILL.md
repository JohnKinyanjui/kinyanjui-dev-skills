---
name: kinyanjui-flutter-skill
description: Implement and review Flutter mobile app changes with clear feature-first folder organization for domain usecases, Riverpod providers, screens, widgets, shared API clients, security utilities, and reusable UI components.
---

# Flutter Feature-First Code Structure Skill

Use this skill when arranging or modifying a Flutter mobile app that follows a
feature-first architecture.

The main goal is to keep the codebase easy to navigate by grouping each product
area into its own feature folder, while keeping shared app-wide code in
`shared`.

## Top-Level Structure

Use this general structure:

```text
lib/
  features/
    feature_name/
      domain/
        provider/
          providers.dart
        usecases/
          get.dart
          post.dart
          put.dart
          delete.dart
          models.dart
      ui/
        screen/
          feature_screen.dart
        widgets/
          feature_widget.dart
  shared/
    http/
      api_client.dart
    security/
      token_manager.dart
    ui/
      theme/
        app_colors.dart
      widgets/
        common/
          shared_widget.dart
```

## Features Folder

The `features` folder contains all user-facing product areas.

Each feature should own its own:

- API/usecase logic
- models
- providers
- screens
- widgets

Examples of feature folders:

```text
features/
  auth/
  profile/
  home/
  events/
  sessions/
  surveys/
  navigation/
```

Each feature should be understandable without searching across unrelated
features.

If a screen belongs only to one feature, keep it inside that feature.
If a widget is only used by one feature, keep it inside that feature.
If code is reused across many features, move it to `shared`.

## Feature Folder Layout

Each feature should usually look like this:

```text
feature_name/
  domain/
    provider/
      providers.dart
    usecases/
      get.dart
      post.dart
      put.dart
      delete.dart
      models.dart
  ui/
    screen/
      feature_screen.dart
    widgets/
      feature_widget.dart
```

Not every feature needs every file. Only create what the feature needs.

For example, a read-only feature may only need:

```text
feature_name/
  domain/
    provider/
      providers.dart
    usecases/
      get.dart
      models.dart
  ui/
    screen/
      feature_screen.dart
```

## Domain Folder

The `domain` folder contains non-UI logic for a feature.

It should contain:

- API usecases
- request/response models
- parsing helpers
- Riverpod providers
- feature-specific business logic

The UI should call providers or usecases, not contain raw API calls directly.

## Domain Provider Folder

Use:

```text
domain/
  provider/
    providers.dart
```

`providers.dart` wires Riverpod providers for the feature.

Example:

```dart
final getItemsUseCaseProvider = Provider<GetItemsUseCase>(
  (_) => GetItemsUseCase(),
);

final itemsProvider =
    FutureProvider.autoDispose.family<List<Item>, int>((ref, id) {
      final uc = ref.watch(getItemsUseCaseProvider);
      return uc(id);
    });
```

Keep provider names clear and feature-specific.

## Domain Usecases Folder

Use:

```text
domain/
  usecases/
    get.dart
    post.dart
    put.dart
    delete.dart
    models.dart
```

Usecase files should contain feature actions.

Recommended file responsibilities:

- `get.dart`: fetch/list/detail actions
- `post.dart`: create/submit/upload actions
- `put.dart`: update actions
- `delete.dart`: delete/remove actions
- `models.dart`: models, DTOs, parsing helpers

Usecase classes should be small and focused.

Example:

```dart
class GetItemsUseCase {
  GetItemsUseCase({ApiClient? client})
      : _client = client ?? ApiClient.instance;

  final ApiClient _client;

  Future<List<Item>> call() async {
    final res = await _client.dio.get<dynamic>('/items');
    return parseItems(res.data);
  }
}
```

## Models

Keep feature models in:

```text
domain/
  usecases/
    models.dart
```

Models should parse data defensively because API responses can change.

Useful helpers:

```dart
Map<String, dynamic>? asMap(Object? value) {
  if (value is Map<String, dynamic>) return value;
  if (value is Map) return Map<String, dynamic>.from(value);
  return null;
}

String? stringValue(Object? value) {
  final text = value?.toString().trim();
  if (text == null || text.isEmpty) return null;
  return text;
}

int? intValue(Object? value) {
  if (value is int) return value;
  if (value is num) return value.toInt();
  if (value is String) return int.tryParse(value);
  return null;
}
```

## UI Folder

The `ui` folder contains all feature presentation code.

Use:

```text
ui/
  screen/
    feature_screen.dart
  widgets/
    feature_widget.dart
```

Screens should live in `ui/screen`.
Feature-only widgets should live in `ui/widgets`.

Keep screens focused on layout, user interaction, and provider watching.
Move repeated UI pieces into widgets.

### Widget Extraction Policy

Always keep small one-off UI fragments inline instead of extracting them into
private widget classes or widget-builder functions. If the extracted private UI
only wraps a few lines of layout and is not reused, inline it directly into the
screen's `build` method so the local UI flow is easier to read.

Do not leave big widgets inside screen files. If a widget is large, reused,
stateful, visually distinct, or has meaningful behavior, move it into the
feature's `ui/widgets` folder with a clear public class name. Screen files should
own route-level state and composition, while substantial UI pieces should live in
widgets.

## Screen Folder

Use:

```text
ui/
  screen/
    profile_screen.dart
    event_detail_screen.dart
```

Screens are route-level widgets.

A screen can:

- watch providers
- handle loading/error/empty states
- own local screen state
- navigate to other screens
- compose feature widgets

Avoid putting large reusable widgets inside screen files when they can be
moved to `ui/widgets`.

## Widgets Folder

Use:

```text
ui/
  widgets/
    item_card.dart
    section_header.dart
    empty_state.dart
```

Widgets in a feature folder should be specific to that feature.

If the widget becomes useful across multiple features, move it to:

```text
shared/
  ui/
    widgets/
      common/
```

## Shared Folder

The `shared` folder contains app-wide code used by multiple features.

Use it for:

- HTTP client
- token/session storage
- app theme/colors
- common widgets
- shared utilities
- reusable UI components

Example:

```text
shared/
  http/
    api_client.dart
  security/
    token_manager.dart
  ui/
    theme/
      app_colors.dart
    widgets/
      common/
        app_button.dart
        app_feedback_dialog.dart
```

Do not place feature-specific business logic in `shared`.

## Naming Rules

Use clear, predictable names.

Files:

```text
profile_screen.dart
event_card.dart
get.dart
post.dart
models.dart
providers.dart
```

Classes:

```dart
ProfileScreen
EventCard
GetEventsUseCase
CreateEventUseCase
EventModel
```

Providers:

```dart
eventsProvider
eventDetailsProvider
getEventsUseCaseProvider
```

## Import Direction

Keep dependencies flowing in one direction:

```text
ui -> domain -> shared
```

Allowed:

- UI imports domain providers/models.
- Domain imports shared API/security utilities.
- Any layer can import shared UI/theme only when appropriate.

Avoid:

- domain importing UI
- shared importing feature code
- feature A depending on feature B unless it is a deliberate navigation or
  integration point

## When To Create A New Feature

Create a new feature folder when the code represents a distinct app area, such
as:

- auth
- profile
- events
- surveys
- sessions
- settings
- notifications

Do not create a new feature for a tiny reusable widget. Put reusable widgets in
`shared`.

## When To Move Code To Shared

Move code to `shared` when:

- two or more features use it
- it is app infrastructure
- it is theme/design-system code
- it is a common widget
- it is not tied to one product area

Keep code inside the feature when:

- only that feature uses it
- it depends on that feature's models
- it represents feature-specific behavior

## Development Rules

Before editing, inspect the existing feature structure and follow the local
pattern.

When adding code:

- keep edits scoped to the relevant feature
- prefer existing helper classes and shared widgets
- do not introduce a new architecture style unless asked
- avoid unrelated refactors
- avoid moving files unless it improves the structure clearly

When checking changes:

```text
git diff --check
```

Run heavier checks only when appropriate for the task.
