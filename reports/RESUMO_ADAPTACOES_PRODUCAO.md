# Adaptação do Alphavel Full para Performance Nativa
**Data:** 26 de Novembro de 2025

Seguindo as descobertas do benchmark `alphavel_q` vs `alphavel_2`, aplicamos as seguintes otimizações no core do framework e no esqueleto de novos projetos:

## 1. Otimização do Query Builder (Hot Path Nativo)
**Arquivo:** `database/QueryBuilder.php`

O `QueryBuilder` agora detecta automaticamente se está em uma transação.
*   **Fora de Transação (Leitura):** Utiliza a conexão de leitura compartilhada (`DB::connectionRead()`) e um cache global de statements. Isso elimina o overhead de lookup de corrotina e permite reuso massivo de statements preparados entre requests.
*   **Em Transação:** Mantém o isolamento por corrotina, mas agora com chaves de cache corrigidas (`tx:{connHash}:{sqlHash}`) para evitar colisão de statements entre conexões.

**Impacto:** Queries padrão como `DB::table('users')->where('id', 1)->get()` agora têm performance próxima a `DB::findOne()`, sem necessidade de refatoração manual.

## 2. Exposição de Conexão de Leitura
**Arquivo:** `database/DB.php`

*   Método `connectionRead()` alterado de `private` para `public` (uso interno), permitindo que o `QueryBuilder` e outros componentes acessem a conexão otimizada de leitura.

## 3. Otimização de Resposta JSON
**Arquivo:** `alphavel/Response.php`

*   Adicionados flags `JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES` para reduzir o tamanho do payload e processamento.
*   Adicionado `JSON_THROW_ON_ERROR` para maior segurança e debug.

## 4. Simplificação da Infraestrutura (Skeleton)
**Arquivos:** `skeleton/Dockerfile`, `skeleton/docker-compose.yml`

O template padrão para novos projetos (`skeleton`) foi atualizado para refletir a configuração vencedora:
*   **Removido:** JIT Tracing (instável/desnecessário para I/O bound).
*   **Removido:** Huge Pages e File Cache do OPcache.
*   **Removido:** Criação complexa de usuário `alphavel` (roda como root/default para menor overhead e simplicidade).
*   **Mantido:** `opcache.validate_timestamps=0` e configurações essenciais de Swoole.

Essas mudanças garantem que qualquer novo projeto criado com Alphavel já nasça com a melhor performance possível "out of the box".
