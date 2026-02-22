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
