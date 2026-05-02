---
inclusion: always
---

# Documentation Guidelines

## CRITICAL: Centralized Architecture Documentation

**This workspace (`Docs/`) contains the SINGLE SOURCE OF TRUTH for all Clean Architecture documentation.**

### Multi-Workspace Architecture

This documentation can be used across multiple workspaces in your project:

- **`Docs/`** - Centralized architecture documentation (THIS WORKSPACE)
- **Other workspaces** - Your application or package workspaces

### Documentation Reference Rule

**ALWAYS reference documentation from `Docs/` regardless of which workspace you're working in.**

When implementing features in ANY workspace:

1. **Read documentation from**: `Docs/` (centralized)
2. **Create changelogs in**: `{target-workspace}/.kiro/changelogs/` (workspace-specific)
3. **Create specs in**: `{target-workspace}/.kiro/specs/` (workspace-specific)
4. **Implement code in**: `{target-workspace}/lib/` or `{target-workspace}/src/` (workspace-specific)

### Example Workflow

**Scenario**: Create a new UseCase in any workspace

1. **Read**: `Docs/domain/i_usecases.md` (centralized documentation)
2. **Read**: `Docs/architecture-overview.md` (centralized architecture)
3. **Create**: `{workspace}/.kiro/changelogs/YYYY-MM-DD_new-usecase.md` (workspace changelog)
4. **Implement**: `{workspace}/lib/modules/feature/domain/usecases/i_new_usecase.dart` (workspace code)

**Scenario**: Create a new Repository in any workspace

1. **Read**: `Docs/domain/i_repositories.md` (centralized documentation)
2. **Read**: `Docs/infra/repositories.md` (centralized documentation)
3. **Create**: `{workspace}/.kiro/changelogs/YYYY-MM-DD_new-repository.md` (workspace changelog)
4. **Implement**: `{workspace}/lib/domain/repositories/i_new_repository.dart` (workspace code)

### Path References in Documentation

All documentation paths in `Docs/` are relative to the `Docs/` workspace root. When applying to other workspaces, adapt the paths accordingly while following the same architecture structure.

## Documentation Structure

**All documentation is centralized in the `Docs/` workspace.**

### Entry Point

- **[Docs/index.md](../../index.md)** - Main documentation index with navigation to all layers

### Core Architecture

- **[Docs/architecture-overview.md](../../architecture-overview.md)** - Detailed explanation of layer responsibilities, abstractions, SOLID principles, and dependency flow

### Domain Layer (Contracts & Business Rules)

- **[Docs/domain/i_usecases.md](../../domain/i_usecases.md)** - UseCase interface contracts
- **[Docs/domain/i_repositories.md](../../domain/i_repositories.md)** - Repository interface contracts
- **[Docs/domain/entities.md](../../domain/entities.md)** - Business entities with validation rules
- **[Docs/domain/enums.md](../../domain/enums.md)** - Domain enums and constants
- **[Docs/domain/i_failures.md](../../domain/i_failures.md)** - Error type definitions
- **[Docs/domain/i_services.md](../../domain/i_services.md)** - Service interface contracts

### Infrastructure Layer (Implementations & Coordination)

- **[Docs/infra/i_datasources.md](../../infra/i_datasources.md)** - DataSource interface contracts
- **[Docs/infra/models.md](../../infra/models.md)** - Data models with serialization
- **[Docs/infra/repositories.md](../../infra/repositories.md)** - Repository implementations
- **[Docs/infra/usecases.md](../../infra/usecases.md)** - UseCase implementations

### Data Layer (External Communication)

- **[Docs/data/datasources.md](../../data/datasources.md)** - DataSource implementations
- **[Docs/data/drivers.md](../../data/drivers.md)** - External service drivers

### Presentation Layer (UI & State)

- **[Docs/presentation/controllers.md](../../presentation/controllers.md)** - ValueNotifier-based state management
- **[Docs/presentation/modules.md](../../presentation/modules.md)** - Flutter Modular DI configuration
- **[Docs/presentation/validators.md](../../presentation/validators.md)** - Input validation logic
- **[Docs/presentation/tracking.md](../../presentation/tracking.md)** - Tracking mixins for Mixpanel conversion funnels

## Key Documentation Principles

### When Implementing Features

1. **Start with Domain** - Read entity and interface docs first
2. **Follow the Flow** - Domain → Infrastructure → Data → Presentation
3. **Check Examples** - Each doc has real-world examples from the codebase
4. **Verify Contracts** - Ensure interfaces follow I\_ prefix and Either pattern

### When Answering Questions

1. **Reference Docs** - Always cite specific documentation sections
2. **Use Real Examples** - Pull examples from the docs (UserEntity, AuthProviderType, etc.)
3. **Explain Layers** - Clarify which layer is responsible for what
4. **SOLID Principles** - Emphasize DIP, ISP, SRP in explanations

### Documentation Standards

- All interfaces use `I_` prefix (e.g., `IUserRepository`, `IUserUsecase`)
- All operations return `Either<Failure, Success>` for error handling
- Entities are immutable with `const` constructors
- Models extend Entities and add serialization
- Controllers use ValueNotifier for reactive state management

## Quick Reference

### Layer Responsibilities

- **Domain**: Defines WHAT (contracts, entities, rules) - zero dependencies
- **Infrastructure**: Implements HOW (coordination, transformation)
- **Data**: Executes communication (HTTP, DB, cache)
- **Presentation**: Manages UI state (ValueNotifier + Either pattern)

### Dependency Direction

```
Presentation → Infrastructure → Data
     ↓              ↓
   Domain ← ← ← ← ← ←
```

All layers depend on Domain abstractions, never implementations.

## Creating New API Integrations

### CRITICAL: Complete Integration Workflow

When creating new API endpoints or integrations, follow this **EXACT** order to ensure Clean Architecture compliance and avoid rework.

### Step-by-Step Integration Flow

#### 1. **Read Documentation First** (MANDATORY)

Before writing ANY code, read these docs in order:

1. **[Docs/architecture-overview.md](../../architecture-overview.md)** - Understand layer responsibilities
2. **[Docs/domain/entities.md](../../domain/entities.md)** - Learn entity patterns
3. **[Docs/infra/models.md](../../infra/models.md)** - Learn model patterns with safe conversions
4. **[Docs/domain/i_repositories.md](../../domain/i_repositories.md)** - Repository interface contracts
5. **[Docs/infra/repositories.md](../../infra/repositories.md)** - Repository implementation patterns
6. **[Docs/domain/i_usecases.md](../../domain/i_usecases.md)** - UseCase interface contracts
7. **[Docs/infra/usecases.md](../../infra/usecases.md)** - UseCase implementation patterns
8. **[Docs/infra/i_datasources.md](../../infra/i_datasources.md)** - DataSource interface contracts
9. **[Docs/data/datasources.md](../../data/datasources.md)** - DataSource implementation patterns
10. **[Docs/presentation/controllers.md](../../presentation/controllers.md)** - Controller patterns

#### 2. **Create Changelog** (Before Implementation)

```bash
# Location: {workspace}/.kiro/changelogs/
# File: YYYY-MM-DD_feature-name.md
```

Start with status "In Progress" and update as you go.

#### 3. **Domain Layer** (Contracts & Entities)

**Order matters! Follow this sequence:**

##### 3.1. Create Request/Response Entities

```dart
// Location: {workspace}/lib/src/domain/entities/{module}/

// Example: send_phone_code_request_entity.dart
class SendPhoneCodeRequestEntity {
  const SendPhoneCodeRequestEntity({
    required this.phoneNumber,
  });

  final String phoneNumber;
}

// Example: send_phone_code_response_entity.dart
class SendPhoneCodeResponseEntity {
  const SendPhoneCodeResponseEntity({
    required this.sessionId,
  });

  final String sessionId;
}
```

**Rules:**

- ✅ Use `const` constructors
- ✅ All fields are `final`
- ✅ Organize parameters by line length (shortest → longest)
- ✅ Add `copyWith` for complex entities
- ✅ No serialization logic (that's for Models)

##### 3.2. Create Failure Types (if needed)

```dart
// Location: {workspace}/lib/src/domain/failures/i_{module}_failures.dart

abstract class IRegisterFailure extends Failure {
  IRegisterFailure({required super.message});
}

class RegisterInvalidDataError extends IRegisterFailure {
  RegisterInvalidDataError({required super.message});
}
```

##### 3.3. Define Repository Interface

```dart
// Location: {workspace}/lib/src/domain/repositories/i_{module}_repository.dart

abstract class IRegisterRepository {
  Future<Either<IRegisterFailure, SendPhoneCodeResponseEntity>> sendPhoneCode({
    required SendPhoneCodeRequestEntity data,
  });
}
```

**Rules:**

- ✅ Use `I_` prefix
- ✅ Return `Either<Failure, Success>`
- ✅ Use entities, not models

##### 3.4. Define UseCase Interface

```dart
// Location: {workspace}/lib/src/domain/usecases/i_{module}_usecase.dart

abstract class IRegisterUsecase {
  Future<Either<IRegisterFailure, SendPhoneCodeResponseEntity>> sendPhoneCode({
    required SendPhoneCodeRequestEntity data,
  });
}
```

#### 4. **Infrastructure Layer** (Models & Implementations)

##### 4.1. Create Request/Response Models

```dart
// Location: {workspace}/lib/src/infra/models/{module}/

// Example: send_phone_code_request_model.dart
class SendPhoneCodeRequestModel extends SendPhoneCodeRequestEntity {
  const SendPhoneCodeRequestModel({required super.phoneNumber});

  factory SendPhoneCodeRequestModel.fromEntity(
    SendPhoneCodeRequestEntity entity,
  ) {
    return SendPhoneCodeRequestModel(phoneNumber: entity.phoneNumber);
  }

  Map<String, dynamic> get toMap {
    return {'phoneNumber': phoneNumber.replaceAll(RegExp('[^0-9]'), '')};
  }
}

// Example: send_phone_code_response_model.dart
class SendPhoneCodeResponseModel extends SendPhoneCodeResponseEntity {
  const SendPhoneCodeResponseModel({required super.sessionId});

  factory SendPhoneCodeResponseModel.fromJson(Map<String, dynamic> json) {
    return SendPhoneCodeResponseModel(
      sessionId: json['sessionId']?.toString() ?? '',
    );
  }

  Map<String, dynamic> get toMap {
    return {'sessionId': sessionId};
  }
}
```

**CRITICAL Rules for Models:**

- ✅ Extend corresponding Entity
- ✅ Use **SAFE conversions** (no `as` casts):
  - Strings: `json['field']?.toString() ?? ''`
  - Booleans: `json['field'] == true`
  - Integers: `int.tryParse(json['field']?.toString() ?? '0') ?? 0`
  - Doubles: `double.tryParse(json['field']?.toString() ?? '0.0') ?? 0.0`
  - Lists: Validate with `is List` and use try-catch
  - Maps: Validate with `is Map` before passing
- ✅ Clean data in `toMap` (remove special chars from CPF, phone, etc.)
- ✅ Organize parameters by line length

##### 4.2. Define DataSource Interface

```dart
// Location: {workspace}/lib/src/infra/datasources/i_{module}_datasource.dart

abstract class IRegisterDatasource {
  Future<Either<HttpErrorResponse, HttpDriverResponse>> sendPhoneCode({
    required SendPhoneCodeRequestEntity data,
  });
}
```

**Rules:**

- ✅ Return `Either<HttpErrorResponse, HttpDriverResponse>`
- ✅ Use entities in parameters

##### 4.3. Implement Repository

```dart
// Location: {workspace}/lib/src/infra/repositories/{module}_repository.dart

class RegisterRepository extends IRegisterRepository {
  RegisterRepository({required this.datasource, required this.crashLog});

  final IRegisterDatasource datasource;
  final CrashLog crashLog;

  @override
  Future<Either<IRegisterFailure, SendPhoneCodeResponseEntity>> sendPhoneCode({
    required SendPhoneCodeRequestEntity data,
  }) {
    return datasource.sendPhoneCode(data: data).then((value) {
      try {
        return value.fold(
          (l) => Left(RegisterServerError(message: l.message)),
          (r) => Right(
            SendPhoneCodeResponseModel.fromJson(r.data as Map<String, dynamic>),
          ),
        );
      } catch (e, s) {
        crashLog.capture(exception: e, stackTrace: s);
        return Left(RegisterUnknownError(message: '$e'));
      }
    });
  }
}
```

**Rules:**

- ✅ Use `extends` not `implements`
- ✅ Inject `datasource` and `crashLog`
- ✅ Use `.then()` with `fold`
- ✅ Transform `HttpErrorResponse` → Domain `Failure`
- ✅ Use `Right(unit)` for void returns

##### 4.4. Implement UseCase

```dart
// Location: {workspace}/lib/src/infra/usecases/{module}_usecase.dart

class RegisterUsecase extends IRegisterUsecase {
  RegisterUsecase({
    required this.crashLog,
    required this.repository,
    required this.storageService,
  });

  final CrashLog crashLog;
  final IRegisterRepository repository;
  final IPreferencesStorageService storageService;

  @override
  Future<Either<IRegisterFailure, SendPhoneCodeResponseEntity>> sendPhoneCode({
    required SendPhoneCodeRequestEntity data,
  }) {
    // Validate required fields
    if (data.phoneNumber.isEmpty) {
      return Future.value(
        Left(
          RegisterInvalidDataError(
            message: 'O número de telefone deve ser informado.',
          ),
        ),
      );
    }

    return repository.sendPhoneCode(data: data);
  }
}
```

**Rules:**

- ✅ Use `extends` not `implements`
- ✅ Validate required fields
- ✅ Return `Left(Failure)` for validation errors
- ✅ Delegate to repository

#### 5. **Data Layer** (External Communication)

##### 5.1. Implement DataSource

```dart
// Location: {workspace}/lib/src/data/datasources/{module}_datasource.dart

class RegisterDatasource extends IRegisterDatasource {
  RegisterDatasource({required this.httpDriver});

  final IHttpDriver httpDriver;

  @override
  Future<Either<HttpErrorResponse, HttpDriverResponse>> sendPhoneCode({
    required SendPhoneCodeRequestEntity data,
  }) {
    return httpDriver
        .post(
          '/v1/sign-up/create-session',
          data: SendPhoneCodeRequestModel.fromEntity(data).toMap,
        )
        .then((value) => value.fold(HttpErrorResponse.left, Right));
  }
}
```

**Rules:**

- ✅ Use `extends` not `implements`
- ✅ Inject `httpDriver`
- ✅ Convert entity → model → map
- ✅ Use `.then()` with `fold`

#### 6. **Presentation Layer** (Controllers & UI)

##### 6.1. Create Controller

```dart
// Location: {workspace}/lib/src/presentation/controllers/{module}/

class SendPhoneCodeController extends CustomController<
    IRegisterFailure,
    SendPhoneCodeResponseEntity> {
  SendPhoneCodeController({required this.usecase});

  final IRegisterUsecase usecase;

  Future<void> call(SendPhoneCodeRequestEntity data) async {
    await execute(() => usecase.sendPhoneCode(data: data));
  }
}
```

**Rules:**

- ✅ Extend `CustomController<Failure, Entity>`
- ✅ Inject usecase
- ✅ Use `execute()` method
- ✅ Organize parameters by line length

##### 6.2. Register in Module

```dart
// Location: {workspace}/lib/src/presentation/modules/{module}_module.dart

class RegisterModule extends Module {
  @override
  void binds(Injector i) {
    // DataSources
    i.add<IRegisterDatasource>(RegisterDatasource.new);

    // Repositories
    i.add<IRegisterRepository>(RegisterRepository.new);

    // UseCases
    i.add<IRegisterUsecase>(RegisterUsecase.new);

    // Controllers
    i.add<SendPhoneCodeController>(SendPhoneCodeController.new);
  }
}
```

**Rules:**

- ✅ Use `.new` syntax for bindings
- ✅ Order: DataSources → Repositories → UseCases → Controllers

#### 7. **Update Exports** (If in Package)

If working in a package, update export files:

```dart
// lib/src/domain/entities/entities.dart
export 'send_phone_code_request_entity.dart';
export 'send_phone_code_response_entity.dart';

// lib/src/infra/models/models.dart
export 'send_phone_code_request_model.dart';
export 'send_phone_code_response_model.dart';
```

#### 8. **Run Diagnostics**

```bash
# Check for errors
flutter analyze {workspace}/lib/src/domain/
flutter analyze {workspace}/lib/src/infra/
flutter analyze {workspace}/lib/src/data/
flutter analyze {workspace}/lib/src/presentation/
```

#### 9. **Update Changelog**

Mark status as "Completed" and add all created files.

### Integration Checklist

Use this checklist for every new integration:

- [ ] Read all relevant documentation
- [ ] Create changelog (status: In Progress)
- [ ] **Domain Layer**:
  - [ ] Request/Response entities created
  - [ ] Failure types defined (if needed)
  - [ ] Repository interface defined
  - [ ] UseCase interface defined
- [ ] **Infrastructure Layer**:
  - [ ] Request/Response models created with safe conversions
  - [ ] DataSource interface defined
  - [ ] Repository implemented
  - [ ] UseCase implemented with validation
- [ ] **Data Layer**:
  - [ ] DataSource implemented
- [ ] **Presentation Layer**:
  - [ ] Controller created
  - [ ] Module bindings updated
- [ ] **Exports** (if package):
  - [ ] Entity exports updated
  - [ ] Model exports updated
- [ ] **Quality**:
  - [ ] All diagnostics pass
  - [ ] Parameters organized by line length
  - [ ] Safe type conversions in models
  - [ ] Data cleaning in models (CPF, phone, etc.)
- [ ] **Documentation**:
  - [ ] Changelog updated (status: Completed)
  - [ ] All files documented in changelog

### Common Mistakes to Avoid

❌ **DON'T**:

- Skip reading documentation
- Use `as` casts in models (use safe conversions)
- Use `implements` for Repository/UseCase (use `extends`)
- Forget to validate in UseCases
- Use lambda functions in module bindings (use `.new`)
- Create entities without `const` constructors
- Put serialization logic in entities
- Forget to clean data in models (CPF, phone, etc.)
- Skip changelog creation

✅ **DO**:

- Follow the exact order above
- Use safe type conversions everywhere
- Validate all required fields in UseCases
- Organize parameters by line length
- Create changelog before starting
- Update exports in packages
- Run diagnostics before finishing

## File References in Documentation

Documentation files use the `#[[file:<relative_path>]]` syntax to reference actual code files. When you see this, it means the documentation is directly tied to real implementation examples in the codebase.

## AI Change Tracking

### Changelog Documentation

**CRITICAL**: Changelogs are workspace-specific, but follow centralized documentation.

When making changes via AI in ANY workspace, always document them in that workspace's `.kiro/changelogs/` directory for traceability.

#### Changelog Structure

- **Location**: `{workspace}/.kiro/changelogs/`
- **Naming**: `YYYY-MM-DD_feature-name.md` (e.g., `2026-02-22_roles-permissions.md`)
- **Format**: Markdown with structured sections

#### Required Changelog Sections

```markdown
# [Feature/Change Name]

**Date**: YYYY-MM-DD
**Type**: [Feature | Bugfix | Refactor | Documentation]
**Status**: [In Progress | Completed | Rolled Back]

## Summary

Brief description of what was changed and why.

## Files Modified

- `path/to/file1.dart` - Description of changes
- `path/to/file2.dart` - Description of changes

## Files Created

- `path/to/new_file.dart` - Purpose of new file

## Files Deleted

- `path/to/old_file.dart` - Reason for deletion

## Architecture Impact

- Which layers were affected (Domain/Infrastructure/Data/Presentation)
- Any new dependencies or contracts introduced
- Breaking changes or migration notes

## Testing

- Tests added or modified
- Manual testing steps performed

## Related Documentation

- Links to updated docs in `Docs/` or specs
- References to related changelogs in the same workspace
- Cross-workspace references if applicable
```

#### Changelog Best Practices

1. **Create Before Implementation** - Start the changelog when beginning work in the target workspace
2. **Update Continuously** - Add entries as you make changes
3. **Be Specific** - Include file paths relative to the workspace root and describe what changed
4. **Link Context** - Reference `Docs/` documentation, specs, or issues
5. **Mark Status** - Update status as work progresses (In Progress → Completed)
6. **Cross-Reference** - When changes span multiple workspaces, create changelogs in each affected workspace

#### When to Create Changelogs

- Implementing new features
- Refactoring existing code
- Fixing bugs that affect multiple files
- Making architectural changes
- Updating dependencies or contracts

#### Changelog Retention

- Keep changelogs indefinitely for historical reference
- Use git history in conjunction with changelogs for full traceability
- Reference changelogs in PR descriptions when applicable

## Documentation File Policy

### CRITICAL: No Loose .md Files

**NEVER create standalone `.md` documentation files outside of designated folders.**

#### Allowed Locations for .md Files

1. **`{workspace}/.kiro/changelogs/`** - Change documentation (REQUIRED for all changes, workspace-specific)
2. **`Docs/`** - Architecture and design documentation (CENTRALIZED, structured folders only)
3. **`{workspace}/.kiro/specs/`** - Feature specifications (workspace-specific, structured spec folders only)

#### Forbidden Locations

- ❌ Any workspace root (loose `.md` files)
- ❌ `lib/` folders in any workspace
- ❌ `src/` folders in any workspace
- ❌ Any other location not explicitly listed above

#### Why This Rule Exists

- **Consistency**: All documentation follows the same structure
- **Discoverability**: Developers know exactly where to find documentation
- **Maintenance**: Centralized docs are easier to keep up-to-date
- **Traceability**: Changelogs provide historical context for all changes

#### What to Do Instead

- **For implementation guides**: Add to workspace-specific changelog with detailed examples
- **For usage documentation**: Add to `Docs/` in appropriate layer folder (centralized)
- **For feature specs**: Create in `{workspace}/.kiro/specs/{feature-name}/` (workspace-specific)
- **For change tracking**: Always use `{workspace}/.kiro/changelogs/` (workspace-specific)

#### Example: Wrong vs Right

```
❌ WRONG:
{workspace}/                           # Any workspace
├── CANCEL_TOKEN_USAGE.md              # Loose file in workspace root
├── lib/
│   └── IMPLEMENTATION_GUIDE.md        # Loose file in lib
└── modules/
    └── HOW_TO_USE.md                  # Loose file in modules

✅ RIGHT:
{workspace}/                           # Any workspace
├── .kiro/
│   ├── changelogs/
│   │   └── YYYY-MM-DD_cancel-token-support.md  # Workspace-specific changelog
│   └── specs/
│       └── cancel-token-feature/
│           ├── requirements.md
│           ├── design.md
│           └── tasks.md
└── lib/
    └── modules/
        └── feature/
            └── domain/
                └── usecases/
                    └── i_cancel_token_usecase.dart  # Implementation

Docs/                                  # Centralized documentation workspace
├── domain/
│   └── i_usecases.md                  # Centralized architecture documentation
└── presentation/
    └── controllers.md
```

#### Enforcement

- AI assistants MUST NOT create loose `.md` files
- Code reviews MUST reject PRs with loose `.md` files
- Existing loose `.md` files MUST be moved or deleted

## Export Standardization

### CRITICAL: Hierarchical Export Structure

**All packages MUST follow a standardized, recursive export structure for external usage.**

#### Export Pattern Rules

1. **Each folder MUST have an export file** named `<folder_name>.dart`
2. **Export files MUST export all files** within their folder
3. **Export files MUST export all subfolders** via their export files
4. **Structure MUST be recursive** from deepest to shallowest folders
5. **Root export file** (`lib/<package_name>.dart`) MUST export `src/src.dart`

#### Export File Naming Convention

- Folder: `lib/src/domain/entities/`
- Export file: `lib/src/domain/entities/entities.dart`
- Content: Exports all `.dart` files in the folder

#### Hierarchical Structure Example

```
lib/
├── paypay_core.dart                    # Root export (exports src/src.dart)
└── src/
    ├── src.dart                        # Exports all layer folders
    ├── core/
    │   ├── core.dart                   # Exports all core subfolders
    │   ├── constants/
    │   │   └── constants.dart          # Exports all constant files
    │   ├── validators/
    │   │   └── validators.dart         # Exports all validator files
    │   └── utils/
    │       └── utils.dart              # Exports all util files
    ├── domain/
    │   ├── domain.dart                 # Exports all domain subfolders
    │   ├── entities/
    │   │   └── entities.dart           # Exports all entity files
    │   ├── repositories/
    │   │   └── repositories.dart       # Exports all repository files
    │   └── usecases/
    │       └── usecases.dart           # Exports all usecase files
    ├── infra/
    │   ├── infra.dart                  # Exports all infra subfolders
    │   └── ...
    ├── data/
    │   ├── data.dart                   # Exports all data subfolders
    │   └── ...
    └── presentation/
        ├── presentation.dart           # Exports all presentation subfolders
        ├── widgets/
        │   ├── widgets.dart            # Exports all widget subfolders + files
        │   ├── buttons/
        │   │   └── buttons.dart        # Exports all button files
        │   └── inputs/
        │       └── inputs.dart         # Exports all input files
        └── ...
```

#### Export File Content Pattern

**Leaf folder** (no subfolders, only files):

```dart
// lib/src/domain/entities/entities.dart
export 'user_entity.dart';
export 'company_entity.dart';
export 'address_entity.dart';
```

**Branch folder** (has subfolders):

```dart
// lib/src/domain/domain.dart
export 'entities/entities.dart';
export 'repositories/repositories.dart';
export 'usecases/usecases.dart';
export 'enums/enums.dart';
export 'failures/failures.dart';
export 'services/services.dart';
```

**Mixed folder** (has both subfolders and files):

```dart
// lib/src/presentation/widgets/widgets.dart
export 'buttons/buttons.dart';
export 'inputs/inputs.dart';
export 'alerts/alerts.dart';
export 'address_form_view.dart';
export 'avatar_initials.dart';
export 'custom_error_builder.dart';
```

**Root src export**:

```dart
// lib/src/src.dart
export 'core/core.dart';
export 'domain/domain.dart';
export 'infra/infra.dart';
export 'data/data.dart';
export 'presentation/presentation.dart';
export 'main.dart';
export 'helper_file.dart';
```

**Package root export**:

```dart
// lib/paypay_core.dart
// External packages
export 'package:base_core/base_core.dart';
export 'package:flutter_modular/flutter_modular.dart';

// Internal exports
export 'src/src.dart';
```

#### Usage Examples

**Import entire layer**:

```dart
import 'package:paypay_core/src/domain/domain.dart';
// Access: UserEntity, IUserRepository, IUserUsecase, etc.
```

**Import specific category**:

```dart
import 'package:paypay_core/src/domain/entities/entities.dart';
// Access: UserEntity, CompanyEntity, AddressEntity, etc.
```

**Import specific subfolder**:

```dart
import 'package:paypay_core/src/presentation/widgets/buttons/buttons.dart';
// Access: PaypayButton, PaypayFloatingButton, etc.
```

**Import everything** (via package root):

```dart
import 'package:paypay_core/paypay_core.dart';
// Access: Everything exported by the package
```

#### Conflict Resolution

When there are naming conflicts between packages:

```dart
// lib/paypay_core.dart
export 'package:base_core/base_core.dart';
export 'src/src.dart' hide DeviceInfoEntity; // Hide duplicate from src
```

Use `hide` or `show` to resolve conflicts at the export level.

#### Benefits

1. **Organized Imports**: Clear, hierarchical import structure
2. **Flexibility**: Import at any level (layer, category, or specific)
3. **Discoverability**: Easy to find what's available
4. **Maintainability**: Adding new files only requires updating one export file
5. **Performance**: Allows tree-shaking with specific imports
6. **Consistency**: Same pattern across all packages

#### Maintenance Workflow

**Adding a new file**:

1. Create file: `lib/src/domain/entities/payment_entity.dart`
2. Add export: `export 'payment_entity.dart';` in `entities.dart`

**Creating a new folder**:

1. Create folder: `lib/src/domain/validators/`
2. Create export file: `lib/src/domain/validators/validators.dart`
3. Add files and export them in `validators.dart`
4. Add export in parent: `export 'validators/validators.dart';` in `domain.dart`

**Removing a file**:

1. Delete file: `lib/src/domain/entities/old_entity.dart`
2. Remove export: Delete `export 'old_entity.dart';` from `entities.dart`

#### Enforcement

- **AI assistants MUST** follow this pattern when creating new packages
- **Code reviews MUST** verify export structure compliance
- **New folders MUST** have corresponding export files
- **Export files MUST** be updated when files are added/removed

#### Example: paypay-core Export Structure

See `paypay-core/.kiro/changelogs/2026-02-23_export-standardization.md` for complete implementation example.

## Git Commit Message Guidelines

### Commit Message Format

Follow the [Conventional Commits](https://www.conventionalcommits.org/) specification for all commits:

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Commit Types

- **feat**: New feature
- **fix**: Bug fix
- **docs**: Documentation only changes
- **style**: Code style changes (formatting, missing semi-colons, etc.)
- **refactor**: Code refactoring (neither fixes a bug nor adds a feature)
- **perf**: Performance improvements
- **test**: Adding or updating tests
- **chore**: Changes to build process, dependencies, or auxiliary tools

### Scope

The scope should indicate the module or area affected:

- `crashlytics`, `analytics`, `auth`, `commission`, `funnel`, `core`, etc.

### Subject

- Use imperative mood ("add" not "added" or "adds")
- Don't capitalize first letter
- No period at the end
- Maximum 72 characters

### Body (Detailed Commits)

For significant changes, include a detailed body with:

1. **Features/Changes**: List of new features or changes
2. **Architecture**: Architecture impact and layer changes
3. **Files**: Key files modified/created/deleted
4. **Platform Support**: Platform compatibility notes
5. **Breaking Changes**: Any breaking changes (use `BREAKING CHANGE:` footer)

### Examples

#### Simple Commit

```bash
git commit -m "fix(auth): resolve login timeout issue"
```

#### Detailed Commit (Preferred for Features)

```bash
git commit -m "feat(crashlytics): add advanced Firebase Crashlytics tracking and Web support

Features:
- Add IFirebaseCrashlyticsDriver interface extending ICrashLogDriver
- Add IFirebaseCrashlyticsService with comprehensive tracking methods
- Implement user info tracking (setUserInfo, clearUserInfo)
- Implement breadcrumbs (logBreadcrumb, logNavigation, logApiCall, logUserAction)
- Implement session context tracking (setSessionContext)
- Implement crash management (sendUnsentReports, checkForUnsentReports)
- Add Web platform support (skip Crashlytics on Web)

Architecture:
- IFirebaseCrashlyticsDriver extends ICrashLogDriver (Infrastructure)
- IFirebaseCrashlyticsService extends ICrashLogService (Domain)
- FirebaseCrashlyticsDriver implements IFirebaseCrashlyticsDriver (Data)
- Proper Clean Architecture compliance (Domain ← Infrastructure ← Data)

Platform Support:
- ✅ Android, iOS, macOS - Full support
- ❌ Web - Gracefully skipped (uses Sentry/Datadog instead)

BREAKING CHANGE: None - all changes are backward compatible

Closes #123"
```

#### Refactoring Commit

```bash
git commit -m "refactor(funnel): reorganize enrollment flow and forms

- Reorganize bottom sheets into bsheets/ folder
- Update enrollment page and form views
- Improve address and personal data forms
- Update payment options and offer selector
- Remove deprecated resume view"
```

#### Chore Commit

```bash
git commit -m "chore: update navigation and dependencies

- Update navigation components
- Update Firebase and iOS configurations
- Update dependencies (pubspec.lock, Podfile.lock)"
```

### Breaking Changes

If introducing breaking changes, add a `BREAKING CHANGE:` footer:

```bash
git commit -m "feat(api): change authentication flow

BREAKING CHANGE: Authentication now requires email verification before login.
Users must update their login flow to handle email verification."
```

### Multiple Changes

If you have multiple unrelated changes, create separate commits:

```bash
# Commit 1: Feature
git add lib/src/domain/usecases/
git commit -m "feat(domain): add new user management usecases"

# Commit 2: Fix
git add lib/src/presentation/pages/
git commit -m "fix(ui): resolve navigation issue on iOS"

# Commit 3: Chore
git add pubspec.lock ios/Podfile.lock
git commit -m "chore: update dependencies"
```

### Commit Message Best Practices

1. **Be Descriptive**: Explain WHAT and WHY, not just WHAT
2. **Use Detailed Body**: For features and significant changes, always include a detailed body
3. **Reference Issues**: Use `Closes #123` or `Fixes #123` in footer
4. **Group Related Changes**: Commit related changes together
5. **Atomic Commits**: Each commit should represent a single logical change
6. **Test Before Commit**: Ensure code compiles and tests pass
7. **Review Changes**: Use `git diff` before committing

### Lefthook Integration

If using Lefthook for git hooks, ensure your commit messages pass validation:

```yaml
# lefthook.yml
commit-msg:
  commands:
    conventional-commit:
      run: |
        # Validate commit message format
        grep -qE '^(feat|fix|docs|style|refactor|perf|test|chore)(\(.+\))?: .{1,72}' {1}
```

### Tools

Consider using tools to help with commit messages:

- **commitizen**: Interactive commit message builder
- **commitlint**: Lint commit messages
- **conventional-changelog**: Generate changelogs from commits
