---
layout: default
title: Compatibility
---

# Alphavel Ecosystem - Compatibility Matrix

**Analysis Date:** November 24, 2025  
**Status:** ✅ **ALL DEPENDENCIES COMPATIBLE**

## Current Versions

| Package | Current Version | PHP | alphavel/alphavel | Extensions |
|---------|----------------|-----|-------------------|------------|
| **alphavel/alphavel** | v1.1.0 | ^8.4 | - | psr/container ^2.0, psr/log ^3.0 |
| **alphavel/database** | v2.1.1 | ^8.4 | ^1.0 | ext-pdo, ext-swoole ^5.0 |
| **alphavel/cache** | v1.1.0 | ^8.4 | ^1.0 | - |
| **alphavel/events** | v1.1.0 | ^8.4 | ^1.0 | - |
| **alphavel/logging** | v1.1.0 | ^8.4 | ^1.0 | psr/log ^3.0 |
| **alphavel/support** | v1.1.0 | ^8.4 | ^1.0 | - |
| **alphavel/validation** | v1.1.0 | ^8.4 | ^1.0 | - |
| **alphavel/alpha** | v1.1.0 | ^8.4 | ^1.0 | - (suggest: alphavel/database) |
| **alphavel/skeleton** | v1.0.4 | ^8.4 | ^1.0 | - (suggest: ext-swoole, alpha, database, etc.) |

## Dependency Matrix

### Core Framework (alphavel/alphavel)
```json
"require": {
  "php": "^8.4",
  "psr/container": "^2.0",
  "psr/log": "^3.0"
}
```
✅ **Status:** Independent, no circular dependencies  
✅ **PSR:** Uses PSR-11 (Container) and PSR-3 (Logger)

### Database Package (alphavel/database)
```json
"require": {
  "php": "^8.4",
  "ext-pdo": "*",
  "ext-swoole": "^5.0",
  "alphavel/alphavel": "^1.0"
}
```
✅ **Status:** Compatible with alphavel v1.0.0 and v1.1.0  
✅ **Extensions:** Native PDO, Swoole for performance  
✅ **Replace:** Replaces alphavel/orm (unified)

### Cache Package (alphavel/cache)
```json
"require": {
  "php": "^8.4",
  "alphavel/alphavel": "^1.0"
}
```
✅ **Status:** Compatible with alphavel v1.0.0 and v1.1.0  
✅ **Zero extra dependencies**

### Events Package (alphavel/events)
```json
"require": {
  "php": "^8.4",
  "alphavel/alphavel": "^1.0"
}
```
✅ **Status:** Compatible with alphavel v1.0.0 and v1.1.0  
✅ **Zero extra dependencies**

### Logging Package (alphavel/logging)
```json
"require": {
  "php": "^8.4",
  "psr/log": "^3.0",
  "alphavel/alphavel": "^1.0"
}
```
✅ **Status:** Compatible with alphavel v1.0.0 and v1.1.0  
✅ **PSR-3:** Standard logger interface

### Support Package (alphavel/support)
```json
"require": {
  "php": "^8.4",
  "alphavel/alphavel": "^1.0"
}
```
✅ **Status:** Compatible with alphavel v1.0.0 and v1.1.0  
✅ **Zero extra dependencies**

### Validation Package (alphavel/validation)
```json
"require": {
  "php": "^8.4",
  "alphavel/alphavel": "^1.0"
}
```
✅ **Status:** Compatible with alphavel v1.0.0 and v1.1.0  
✅ **Zero extra dependencies**

### Alpha CLI (alphavel/alpha)
```json
"require": {
  "php": "^8.4",
  "alphavel/alphavel": "^1.0"
},
"suggest": {
  "alphavel/database": "Required for schema-aware code generation"
}
```
✅ **Status:** Compatible with alphavel v1.0.0 and v1.1.0  
✅ **Optional database:** Avoids circular dependency  
✅ **Auto-detection:** Works with or without database

### Skeleton (alphavel/skeleton)
```json
"require": {
  "php": "^8.4",
  "alphavel/alphavel": "^1.0"
},
"suggest": {
  "ext-swoole": "For high-performance (520k+ req/s)",
  "alphavel/alpha": "CLI tools",
  "alphavel/database": "Database operations",
  "alphavel/cache": "Caching",
  "alphavel/events": "Events",
  "alphavel/logging": "Logging"
}
```
✅ **Status:** Compatible with alphavel v1.0.0 and v1.1.0  
✅ **All optional:** User chooses features  
✅ **Zero conflicts:** class_exists() in configs

## Dependency Graph

```
┌─────────────────────┐
│  alphavel/alphavel  │ ← Core framework (v1.1.0)
│      (PHP ^8.4)     │
└──────────┬──────────┘
           │
           │ depends on (^1.0)
           │
   ┌───────┴────────────────────────────────────┐
   │                                            │
   ▼                                            ▼
┌────────────────┐                    ┌──────────────────┐
│ Core Packages  │                    │ Optional Packages│
├────────────────┤                    ├──────────────────┤
│ - cache        │                    │ - alpha (CLI)    │
│ - events       │                    │ - database       │
│ - logging      │                    │ - skeleton       │
│ - support      │                    │                  │
│ - validation   │                    │                  │
└────────────────┘                    └──────────────────┘
     (v1.1.0)                              (v1.1.0/v2.1.1)
```

## Backward Compatibility

### Breaking Changes (v1.0.0 → v1.1.0)
- **PHP Requirement:** 8.1/8.2 → 8.4
- **Reason:** +10-15% performance, better JIT
- **Affected:** All core packages
- **Migration:** Update PHP to 8.4

### API Backward Compatibility
✅ **100% compatible** - No API changes  
✅ **No functional breaking changes**  
✅ **Only runtime update (PHP)**

## Conflict Verification

### ❌ Conflicts Found: NONE

#### Verification Performed:
1. ✅ **PHP Version:** All 9 packages require ^8.4
2. ✅ **alphavel/alphavel:** All use ^1.0 (compatible with 1.0.0 and 1.1.0)
3. ✅ **PSR Standards:** psr/log ^3.0, psr/container ^2.0 (consistent)
4. ✅ **Circular Dependencies:** ZERO (database and alpha decoupled)
5. ✅ **PHP Extensions:** ext-pdo (native), ext-swoole ^5.0 (database only)

## Recommended Installation

### Minimal Installation (Framework Only)
```bash
composer require alphavel/alphavel:^1.1
```

### Installation with Database
```bash
composer require alphavel/alphavel:^1.1
composer require alphavel/database:^2.1
```

### Complete Installation
```bash
composer create-project alphavel/skeleton:^1.0 my-project
cd my-project
composer require alphavel/database:^2.1  # optional
composer require alphavel/cache:^1.1     # optional
composer require alphavel/events:^1.1    # optional
```

### Development (with CLI)
```bash
composer require --dev alphavel/alpha:^1.1
```

## Compatibility Tests Performed

### 1. Composer Validate ✅
All composer.json files are valid (warnings only about version field).

### 2. Dependency Resolution ✅
```bash
# Tested on fresh installation (alphavel_z)
composer create-project alphavel/skeleton
composer require alphavel/database
# ✅ All dependencies resolved without conflicts
```

### 3. Runtime Compatibility ✅
- PHP 8.4-cli tested on Docker
- Swoole 5.0+ working
- All extensions available

## Recommendations

### For New Users
1. Use `alphavel/skeleton` v1.0.4 as base
2. Add packages as needed
3. Database is optional but recommended

### For Updating from v1.0.0
1. Update PHP to 8.4:
   ```bash
   sudo apt install php8.4-cli php8.4-swoole
   ```
2. Update packages:
   ```bash
   composer require alphavel/alphavel:^1.1
   composer require alphavel/database:^2.1  # if using
   composer require alphavel/cache:^1.1     # if using
   # etc...
   ```
3. Test application (zero breaking changes in API)

## Version Support

| Version | PHP | Status | Support |
|---------|-----|--------|---------|
| 1.0.x | ^8.1 | ⚠️ Old | Security only |
| 1.1.x | ^8.4 | ✅ Current | Full support |
| 2.x.x | TBD | 🔮 Future | Planned |

---

**Conclusion:** ✅ All dependencies are 100% compatible.  
**Zero conflicts detected.**  
**Ecosystem ready for production.**
