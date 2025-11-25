# Alphavel Ecosystem - Compatibility Matrix

**Data da Análise:** 24 de novembro de 2025  
**Status:** ✅ **TODAS AS DEPENDÊNCIAS COMPATÍVEIS**

## Versões Atuais

| Pacote | Versão Atual | PHP | alphavel/alphavel | Extensões |
|--------|--------------|-----|-------------------|-----------|
| **alphavel/alphavel** | v1.1.0 | ^8.4 | - | psr/container ^2.0, psr/log ^3.0 |
| **alphavel/database** | v2.1.1 | ^8.4 | ^1.0 | ext-pdo, ext-swoole ^5.0 |
| **alphavel/cache** | v1.1.0 | ^8.4 | ^1.0 | - |
| **alphavel/events** | v1.1.0 | ^8.4 | ^1.0 | - |
| **alphavel/logging** | v1.1.0 | ^8.4 | ^1.0 | psr/log ^3.0 |
| **alphavel/support** | v1.1.0 | ^8.4 | ^1.0 | - |
| **alphavel/validation** | v1.1.0 | ^8.4 | ^1.0 | - |
| **alphavel/alpha** | v1.1.0 | ^8.4 | ^1.0 | - (suggest: alphavel/database) |
| **alphavel/skeleton** | v1.0.4 | ^8.4 | ^1.0 | - (suggest: ext-swoole, alpha, database, etc.) |

## Matriz de Dependências

### Core Framework (alphavel/alphavel)
```json
"require": {
  "php": "^8.4",
  "psr/container": "^2.0",
  "psr/log": "^3.0"
}
```
✅ **Status:** Independente, sem dependências circulares  
✅ **PSR:** Usa PSR-11 (Container) e PSR-3 (Logger)

### Database Package (alphavel/database)
```json
"require": {
  "php": "^8.4",
  "ext-pdo": "*",
  "ext-swoole": "^5.0",
  "alphavel/alphavel": "^1.0"
}
```
✅ **Status:** Compatível com alphavel v1.0.0 e v1.1.0  
✅ **Extensões:** PDO nativo, Swoole para performance  
✅ **Replace:** Substitui alphavel/orm (unificado)

### Cache Package (alphavel/cache)
```json
"require": {
  "php": "^8.4",
  "alphavel/alphavel": "^1.0"
}
```
✅ **Status:** Compatível com alphavel v1.0.0 e v1.1.0  
✅ **Zero dependências extras**

### Events Package (alphavel/events)
```json
"require": {
  "php": "^8.4",
  "alphavel/alphavel": "^1.0"
}
```
✅ **Status:** Compatível com alphavel v1.0.0 e v1.1.0  
✅ **Zero dependências extras**

### Logging Package (alphavel/logging)
```json
"require": {
  "php": "^8.4",
  "psr/log": "^3.0",
  "alphavel/alphavel": "^1.0"
}
```
✅ **Status:** Compatível com alphavel v1.0.0 e v1.1.0  
✅ **PSR-3:** Logger interface padrão

### Support Package (alphavel/support)
```json
"require": {
  "php": "^8.4",
  "alphavel/alphavel": "^1.0"
}
```
✅ **Status:** Compatível com alphavel v1.0.0 e v1.1.0  
✅ **Zero dependências extras**

### Validation Package (alphavel/validation)
```json
"require": {
  "php": "^8.4",
  "alphavel/alphavel": "^1.0"
}
```
✅ **Status:** Compatível com alphavel v1.0.0 e v1.1.0  
✅ **Zero dependências extras**

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
✅ **Status:** Compatível com alphavel v1.0.0 e v1.1.0  
✅ **Database opcional:** Evita dependência circular  
✅ **Auto-detecção:** Funciona com ou sem database

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
✅ **Status:** Compatível com alphavel v1.0.0 e v1.1.0  
✅ **Todos opcionais:** Usuário escolhe features  
✅ **Zero conflitos:** class_exists() em configs

## Grafo de Dependências

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

## Compatibilidade com Versões Anteriores

### Breaking Changes (v1.0.0 → v1.1.0)
- **PHP Requirement:** 8.1/8.2 → 8.4
- **Motivo:** +10-15% performance, melhor JIT
- **Afetados:** Todos os pacotes core
- **Migração:** Atualizar PHP para 8.4

### Retrocompatibilidade API
✅ **100% compatível** - Nenhuma mudança de API  
✅ **Sem breaking changes funcionais**  
✅ **Apenas atualização de runtime (PHP)**

## Verificação de Conflitos

### ❌ Conflitos Encontrados: NENHUM

#### Verificações Realizadas:
1. ✅ **Versão PHP:** Todas as 9 packages requerem ^8.4
2. ✅ **alphavel/alphavel:** Todos usam ^1.0 (compatível com 1.0.0 e 1.1.0)
3. ✅ **PSR Standards:** psr/log ^3.0, psr/container ^2.0 (consistente)
4. ✅ **Dependências circulares:** ZERO (database e alpha desacoplados)
5. ✅ **Extensões PHP:** ext-pdo (nativo), ext-swoole ^5.0 (só database)

## Instalação Recomendada

### Instalação Mínima (Framework Only)
```bash
composer require alphavel/alphavel:^1.1
```

### Instalação com Database
```bash
composer require alphavel/alphavel:^1.1
composer require alphavel/database:^2.1
```

### Instalação Completa
```bash
composer create-project alphavel/skeleton:^1.0 meu-projeto
cd meu-projeto
composer require alphavel/database:^2.1  # opcional
composer require alphavel/cache:^1.1     # opcional
composer require alphavel/events:^1.1    # opcional
```

### Desenvolvimento (com CLI)
```bash
composer require --dev alphavel/alpha:^1.1
```

## Testes de Compatibilidade Realizados

### 1. Composer Validate ✅
Todos os composer.json são válidos (warnings apenas sobre version field).

### 2. Resolução de Dependências ✅
```bash
# Testado em instalação fresh (alphavel_z)
composer create-project alphavel/skeleton
composer require alphavel/database
# ✅ Todas as dependências resolvidas sem conflitos
```

### 3. Runtime Compatibility ✅
- PHP 8.4-cli testado no Docker
- Swoole 5.0+ funcionando
- Todas as extensões disponíveis

## Recomendações

### Para Usuários Novos
1. Use `alphavel/skeleton` v1.0.4 como base
2. Adicione pacotes conforme necessidade
3. Database é opcional mas recomendado

### Para Atualização de v1.0.0
1. Atualizar PHP para 8.4:
   ```bash
   sudo apt install php8.4-cli php8.4-swoole
   ```
2. Atualizar pacotes:
   ```bash
   composer require alphavel/alphavel:^1.1
   composer require alphavel/database:^2.1  # se usar
   composer require alphavel/cache:^1.1     # se usar
   # etc...
   ```
3. Testar aplicação (zero breaking changes na API)

## Suporte a Versões

| Versão | PHP | Status | Suporte |
|--------|-----|--------|---------|
| 1.0.x | ^8.1 | ⚠️ Old | Security only |
| 1.1.x | ^8.4 | ✅ Atual | Full support |
| 2.x.x | TBD | 🔮 Futuro | Planejado |

---

**Conclusão:** ✅ Todas as dependências estão 100% compatíveis.  
**Zero conflitos detectados.**  
**Ecossistema pronto para produção.**
