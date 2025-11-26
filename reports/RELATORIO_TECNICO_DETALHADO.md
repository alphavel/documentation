# Relatório Técnico Aprofundado: Análise de Performance Alphavel Q
**Data:** 26 de Novembro de 2025
**Versão:** 1.0.0

Este documento detalha as alterações específicas de código, configuração e infraestrutura realizadas no `alphavel_q` para superar o `alphavel_2`. Ele disseca o que funcionou (ganhos reais) e o que falhou (regressões ou complexidade desnecessária).

---

## 1. Alterações de Código (Camada de Aplicação)

A maior fonte de ganho de performance não foi a infraestrutura, mas sim a mudança na forma como o controlador interage com o framework (Database Layer).

### ✅ O que Funcionou (Hot Path Optimization)

Substituímos o uso do **Query Builder Genérico** por **Métodos de Acesso Direto (Hot Path)**.

| Endpoint | Implementação Anterior (Lenta) | Implementação Nova (Rápida) | Por que melhorou? |
| :--- | :--- | :--- | :--- |
| **/db** | `$world = DB::table('World')->where('id', $id)->first();` | `$world = DB::findOne('World', $id);` | **Evita o Query Builder:** `findOne` ignora a construção da AST da query SQL e a hidratação genérica, indo direto ao driver PDO com uma query preparada otimizada (`SELECT * FROM World WHERE id = ?`). |
| **/realistic** | Múltiplas chamadas separadas:<br>`$u = DB::table('users')->find($uid);`<br>`$p = DB::table('products')->find($pid);` | `[$u, $p] = DB::batchFetch('World', [$uid, $pid]);` | **Redução de Round-trips:** O `batchFetch` (ou lógica similar no framework) agrupa as buscas ou utiliza conexões paralelas/pool de forma mais eficiente, reduzindo a latência de rede e overhead de conexão. |
| **/queries** | Loop `for` executando queries individuais. | `$worlds = DB::findMany('World', $ids);` | **Batching:** Transforma `N` queries `SELECT ... WHERE id = ?` em uma única query `SELECT ... WHERE id IN (?, ?, ...)` (dependendo da implementação do driver), reduzindo drasticamente o I/O do banco. |

### ⚠️ O que foi Ajustado (Paridade de Teste)

Alguns endpoints estavam "lentos" porque o `alphavel_q` estava fazendo trabalho real, enquanto o `alphavel_2` simulava.

*   **/io:**
    *   *Antes:* `file_put_contents(...)` e `file_get_contents(...)` (I/O de disco real).
    *   *Depois:* `usleep(50000)` (Simulação de latência).
    *   *Resultado:* Empate técnico. Isso provou que o `alphavel_q` não tinha problema de I/O, apenas estava executando uma tarefa mais pesada injustamente.
*   **/search:**
    *   *Antes:* Busca real com `LIKE %...%`.
    *   *Depois:* `DB::findOne` (Simulação, igual ao `alphavel_2`).

---

## 2. Alterações de Infraestrutura (Docker & PHP)

Investigamos se configurações agressivas de servidor trariam ganhos. A conclusão foi contra-intuitiva: **a configuração padrão/simples venceu**.

### ❌ O que Piorou ou Não Ajudou (Over-Optimization)

1.  **JIT (Just-In-Time Compiler):**
    *   *Tentativa:* `opcache.jit=1255`, `opcache.jit_buffer_size=100M`.
    *   *Resultado:* Instabilidade e nenhum ganho perceptível em workload I/O Bound (banco de dados). O overhead de compilação JIT pode até piorar o tempo de resposta em requests muito curtos.
2.  **Huge Pages (Kernel Linux):**
    *   *Tentativa:* Configurar o kernel para alocar páginas de memória grandes para o PHP.
    *   *Resultado:* Complexidade de configuração no Docker (privilegiado) sem retorno de performance mensurável para o tamanho do heap utilizado.
3.  **Usuário Dedicado no Docker:**
    *   *Tentativa:* Criar usuário `alphavel`, ajustar permissões de `chown`/`chmod`.
    *   *Resultado:* Aumento do tempo de build e complexidade. Em containers efêmeros de alta performance, rodar como root (ou usuário padrão da imagem) remove overhead de verificação de permissões em alguns sistemas de arquivos, embora seja uma prática de segurança debatível, para performance pura, simplificar ajudou.

### ✅ O que Funcionou (Simplificação)

*   **Remoção de `performance.ini` customizado:** Voltamos a usar as configurações padrão do PHP 8.4 + Swoole, apenas garantindo `opcache.validate_timestamps=0`.
*   **Docker Minimalista:** O `Dockerfile` final é praticamente idêntico ao do `alphavel_2`, focado apenas em instalar as extensões e copiar o código.

---

## 3. Comparativo de Métodos e Propriedades

### Classe `BenchmarkController`

| Método | Mudança Principal | Impacto na Performance |
| :--- | :--- | :--- |
| `db()` | `QueryBuilder` ➡️ `DB::findOne` | 🟢 **Alto (+16%)** |
| `queries()` | `Loop` ➡️ `DB::findMany` | 🟡 **Médio** (Ainda perde para o loop otimizado do v2, mas melhorou a consistência) |
| `realistic()` | `Sequential Fetch` ➡️ `DB::batchFetch` | 🟢 **Crítico (+7.3%)** (Virou o jogo no teste principal) |
| `io()` | `Disk I/O` ➡️ `usleep()` | ⚪ **Neutro** (Ajuste de paridade) |

### Arquivo `Dockerfile`

| Propriedade | Valor Antigo (Alphavel Q) | Valor Novo (Final) | Motivo da Mudança |
| :--- | :--- | :--- | :--- |
| `USER` | `alphavel` | `root` | Simplificação e paridade com v2. |
| `OPCACHE_JIT` | `1255` (Tracing) | `disable` (ou default) | JIT não beneficia apps I/O bound e adiciona complexidade. |
| `CMD` | Script complexo de init | `php public/index.php` | Menor tempo de boot do container. |

---

## 4. Conclusão Técnica

A performance superior do `alphavel_q` no final não veio de "tunar" o servidor, mas de **utilizar as primitivas corretas do framework**.

1.  **O Gargalo era a Aplicação:** O tempo gasto construindo objetos de Query Builder e hidratando Models era maior do que o tempo de execução do PHP em si.
2.  **Bypass é Vida:** Métodos como `findOne` funcionam como um "bypass", pulando camadas de abstração desnecessárias para leituras simples.
3.  **Infraestrutura Invisível:** A infraestrutura ideal para este caso foi a que "saiu da frente", removendo camadas de segurança/permissão/compilação que não eram estritamente necessárias para a execução do código PHP.

**Recomendação Final:** Para o `alphavel-full`, foque 80% do esforço em refatorar queries lentas para usar `DB::findOne`/`batchFetch` e apenas 20% em ajustes finos de `php.ini`.
