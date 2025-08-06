# CDN Translation Migration Pipeline

Automated repository migration system for adopting `@procore/cdn-translations` package across multiple microfrontends.

## 🏗️ Repository Structure

```
cdn_delivery/
├── README.md                      # This documentation
├── instructions.md                # Detailed migration instructions and workflow
├── packages.json                  # Core dependency versions for migration
├── repos_list.xml                 # Repository manifest for batch processing
├── promt                         # Quick execution commands
├── .gitignore                    # Excludes repositories/ from version control
├── errors/                       # 📋 Error documentation and solutions
│   ├── build_errors.md          # Build-time error fixes
│   └── test_errors.md           # Test environment error solutions
└── repositories/                 # 🚧 WORKING DIRECTORY (gitignored)
    │                             # Clone target repositories here for migration
    └── [repo-name]/              # Each repository cloned individually
        ├── package.json          # Will be updated with CDN dependencies
        ├── src/                  # Source code to be migrated
        └── ...                   # Standard repository structure
```

### 📁 Working Directory Notes

- **`repositories/` folder**: This is a **working directory** where you clone target repositories for migration
- **Temporary**: Repositories are cloned here temporarily during the migration process
- **Clean State**: Start fresh by cloning the repository you want to migrate
- **Example Structure**: The `sidepanel-ui-service/` shown above represents what a repository looks like after successful migration

## Purpose

This repository serves as a **centralized migration system** for implementing CDN-based translation delivery across Procore's microfrontend ecosystem. The system automates the complex process of:

### Core Objectives

1. **Standardized Migration**: Ensures consistent implementation of `@procore/cdn-translations` across all repositories
2. **Feature Flag Integration**: Implements LaunchDarkly-based conditional loading with proper fallbacks
3. **Error Documentation**: Maintains a knowledge base of common issues and their solutions
4. **Automated Validation**: Runs comprehensive lint, test, and build checks to ensure migration success
5. **Batch Processing**: Supports migration of multiple repositories using the manifest system

### Technical Implementation

- **CDN Translation Loading**: Conditionally loads translations from CDN based on LaunchDarkly feature flags
- **Backward Compatibility**: Maintains existing translation functionality as fallback
- **Type Safety**: Resolves TypeScript compatibility issues between packages
- **Test Coverage**: Implements proper mocking strategies for feature toggles and CDN services

## Contribution Guidelines

### Error Documentation Protocol

**CRITICAL**: All developers working with this migration system **MUST** document errors and challenges encountered during adoption.

#### When to Document

Document **every** error or challenge you encounter, including:

- Build failures and compilation errors
- Test failures and missing mocks
- Runtime errors in development/staging
- Type compatibility issues
- Feature flag configuration problems
- LaunchDarkly integration issues
- CDN connectivity or loading problems
- Performance impacts or loading delays

#### How to Document Errors

1. **Search First**: Before creating new documentation, **thoroughly search** all existing `.md` files in the `./errors` directory
2. **File Naming**: Use descriptive names like `feature_flag_context_errors.md` or `cdn_timeout_issues.md`
3. **Required Format**:

```markdown
## Error Description
[Clear description of the error with exact error messages]

## Affected Files/Components
[List specific files, components, or repositories affected]

## Root Cause
[Technical explanation of why the error occurs]

## Solution Applied
[Step-by-step solution with code examples]

## Prevention
[How to avoid this error in future migrations]

## Related Issues
[Links to similar problems or dependencies]

## Environment Context
- Repository: [repo-name]
- Package Versions: [relevant versions]
- Browser/Node Version: [if applicable]
- Migration Step: [which phase of migration]
```

#### Example Error Documentation

```markdown
## Error Description
```
TS2322: Type 'string | undefined' is not assignable to type 'string'
locale: env?.metadata?.i18n?.locale,
```

## Affected Files
- `src/components/EnvironmentLayout/Environment.tsx`
- Any component using `useRequestTranslations` hook

## Root Cause
The `locale` parameter in `useRequestTranslations` expects a `string`, but `env?.metadata?.i18n?.locale` can be `undefined`.

## Solution Applied
```typescript
locale: env?.metadata?.i18n?.locale ?? 'en',
```

## Prevention
Always provide fallback values for optional environment metadata when passing to CDN translation hooks.
```

### Contribution Workflow

1. **Pre-Migration**: Review existing error documentation before starting
2. **During Migration**: Document issues in real-time, don't wait until completion
3. **Post-Migration**: Consolidate learnings and update prevention strategies
4. **Knowledge Sharing**: Include context about edge cases and environment-specific issues

### Quality Standards

- **Completeness**: Include full error messages, stack traces, and context
- **Reproducibility**: Provide enough detail for others to reproduce and fix
- **Solutions**: Always include working solutions, not just problem descriptions
- **Maintenance**: Update existing docs when you find better solutions

## Getting Started

### Initial Setup
1. **Clone This Repository**: `git clone [cdn-delivery-repository-url]`
2. **Review Instructions**: Read `instructions.md` for detailed migration workflow
3. **Check Error Docs**: Familiarize yourself with existing solutions in `./errors/`

### Repository Migration Process
4. **Clone Target Repository**: 
   ```bash
   cd repositories/
   git clone [target-repository-url]
   cd [target-repository-name]/
   ```
5. **Start Migration**: Follow the automated pipeline process from `instructions.md`
6. **Document Issues**: Add any new errors or challenges to the `./errors/` knowledge base

### Important Notes
- **Working Directory**: The `repositories/` folder is your workspace for cloning target repositories
- **Gitignored Content**: Cloned repositories won't be committed to this migration repository
- **Clean Environment**: Always start with a fresh clone of the target repository
- **One at a Time**: Migrate one repository at a time for better error tracking

## 📋 Migration Checklist

- [ ] Review existing error documentation
- [ ] Update package dependencies
- [ ] Implement feature flag integration
- [ ] Add CDN translation hooks
- [ ] Update I18nProvider configuration
- [ ] Configure mock server passthroughs
- [ ] Run validation (lint, test, build)
- [ ] Document any new errors encountered
- [ ] Verify functionality in staging environment

---
