# Documentation Validation Checklist

## ✅ Completed

### 1. PHP Version Requirements
- ✓ Fixed: `core/getting-started.md` - PHP 8.1+ → 8.4+
- ✓ Fixed: `packages/circuit-breaker/README.md` - PHP 8.1+ → 8.4+
- ✓ Fixed: `packages/websocket/README.md` - PHP 8.1+ → 8.4+
- ✓ Verified: `alphavel/composer.json` requires `php: ^8.4`

## 🔍 Requires Review

### 2. System Requirements
**Files to check:**
- [ ] Swoole version requirements (mentioned as 5.0+ in some files)
- [ ] Extension requirements (redis, pdo, etc.)
- [ ] OS compatibility mentions

**Search command:**
```bash
grep -r "Swoole" documentation/ --include="*.md"
```

### 3. Command Examples
**Potential issues:**
- [ ] `php alpha` commands - verify they exist in CLI tool
- [ ] `composer create-project alphavel/skeleton`
- [ ] `php alpha package:add database` - confirm syntax
- [ ] Docker commands in getting-started

**Files to validate:**
- `core/getting-started.md`
- `core/cli-commands.md`
- `deployment/local-development.md`

### 4. Internal Links
**Found broken link patterns:**
- Links to `README.md` should use directory format (`packages/database/`)
- Links to `.md` should be `.html` for GitHub Pages

**Files with link issues:**
- `SUMMARY.md` - 20+ links to `README.md`
- `README.md` - Multiple package links

**Fix command:**
```bash
# Replace README.md links with directory format
sed -i 's|packages/\([^/]*\)/README\.md|packages/\1/|g' documentation/*.md
```

### 5. Performance Numbers
**Inconsistencies to verify:**
- Core framework: "520k+ req/s" or "520,000+ req/s"?
- Database: "6,700+ req/s" or "6.7k+ req/s"?
- WebSocket: "500k+ msgs/s"
- Circuit Breaker: "<0.1ms overhead"

**Files mentioning performance:**
- `introduction/benchmarks.md`
- `core/performance.md`
- `core/raw-routes.md`
- Package README files

### 6. Package Names & Installation
**Commands to verify:**
```bash
composer require alphavel/database
composer require alphavel/cache
composer require alphavel/validation
# ... etc
```

**Check against:**
- Packagist.org listings
- composer.json in each package directory

### 7. Code Examples
**Validation needed:**
- [ ] Namespace correctness (`Alphavel\Framework\`, `Alphavel\Database\`)
- [ ] Class names (Controller, Router, Response, etc.)
- [ ] Method names and signatures
- [ ] Configuration array structures

**Critical files:**
- `core/routing.md`
- `core/controllers.md`
- `core/autowiring.md`
- All package README files

### 8. Docker Configuration
**Files to review:**
- [ ] `deployment/local-development.md`
- [ ] Docker commands match actual `docker-compose.yml`
- [ ] Port numbers (9501, 9999, etc.)
- [ ] Environment variables

### 9. Version Numbers
**Check consistency:**
- Alphavel Framework version mentions
- Package version compatibility
- Changelog version references

### 10. External Links
**Verify these work:**
- GitHub organization: `https://github.com/alphavel`
- Packagist URLs: `https://packagist.org/packages/alphavel/`
- Discord/community links (if any)

## 🚨 Critical Issues Found

1. ✓ **FIXED: PHP 8.1 → 8.4** (3 files corrected)

## 📝 Notes

- Many files still reference `README.md` which should be converted to directory links for GitHub Pages
- Need to verify all `php alpha` commands against actual CLI implementation
- Performance benchmarks should be verified with latest tests
- Code examples need syntax validation

## 🔧 Quick Fixes Available

### Fix README.md links globally:
```bash
cd documentation
find . -name "*.md" -type f -exec sed -i 's|/README\.md)|/)|g' {} \;
find . -name "*.md" -type f -exec sed -i 's|/README\.html)|/)|g' {} \;
```

### Find all PHP version mentions:
```bash
grep -rn "PHP 8\." documentation/ --include="*.md"
```

### Find all broken internal links:
```bash
find documentation/ -name "*.md" -exec grep -l "\.md)" {} \;
```
