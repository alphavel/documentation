# Relatório de Performance para Produção: Lições do Alphavel Q
**Data:** 26 de Novembro de 2025
**Contexto:** Análise comparativa entre `alphavel_q` (nova arquitetura) e `alphavel_2` (legado otimizado).

Este documento consolida os aprendizados técnicos obtidos durante as sessões de benchmark, definindo as diretrizes para a implementação do `alphavel-full` em ambiente de produção.

---

## 1. Otimização de Código: O Fator Decisivo 🚀

A maior lição deste experimento foi que **a arquitetura de código supera a configuração de infraestrutura**. O `alphavel_q` só superou o `alphavel_2` quando adotou os padrões de acesso a dados otimizados.

### Ações Recomendadas para Produção:
*   **Adotar "Hot Path Methods":** Em endpoints de alto tráfego, **não utilize** o Query Builder genérico (`DB::table()->where()->get()`). Utilize os métodos especializados que evitam overhead de hidratação e construção de query:
    *   Use `DB::findOne($tabela, $id)` para buscas por chave primária.
    *   Use `DB::findMany($tabela, $ids)` para buscas em lote.
    *   Use `DB::batchFetch($tabela, $ids)` para carregar múltiplos recursos distintos em uma única ida ao banco (quando suportado).
*   **Evitar I/O Bloqueante:** Mesmo em ambientes Swoole/Async, operações de disco ou rede síncronas matam a performance. Onde possível, use drivers assíncronos ou delegue para filas.

---

## 2. Infraestrutura: Menos é Mais 🛠️

Tentamos aplicar "micro-otimizações" agressivas no `alphavel_q` (JIT Tracing, Huge Pages, usuários dedicados, flags de compilação), mas elas provaram ser desnecessárias e, por vezes, instáveis.

### Ações Recomendadas para Produção:
*   **Simplificar o Dockerfile:** A configuração vencedora foi a mais simples (baseada na do `alphavel_2`).
    *   Não complique a gestão de usuários dentro do container a menos que seja uma exigência estrita de segurança (compliance). Permissões mal configuradas consomem ciclos de CPU.
    *   Remova scripts de inicialização complexos.
*   **OPcache Conservador:**
    *   `opcache.validate_timestamps=0` (Essencial para produção).
    *   `opcache.jit=disable` ou `tracing` (com cautela). Para aplicações Web (I/O Bound), o JIT do PHP 8 tráz ganhos marginais e pode dificultar o debug. O benchmark provou que o código bem escrito vence sem JIT.
*   **Network Mode:** O uso da rede `host` ou configurações de bridge otimizadas no Docker Compose ajudam, mas a aplicação deve ser agnóstica a isso.

---

## 3. Pontos de Atenção e Melhoria Contínua ⚠️

Apesar da vitória geral, o `alphavel_q` ainda perdeu em cenários específicos que devem ser monitorados.

*   **Serialização JSON:** O `alphavel_q` foi ~17% mais lento em serialização de JSON simples (`/json`).
    *   *Recomendação:* Revisar a classe `Response` e o serializador JSON do framework novo. Para payloads gigantescos, considerar serialização manual ou bibliotecas de alta performance (ex: `simdjson` bindings se disponível).
*   **Loops de Queries:** Em cenários de muitas queries pequenas sequenciais (`/queries`), o overhead do novo driver pareceu ligeiramente maior.
    *   *Recomendação:* Reforçar o uso de `findMany` para transformar N queries em 1 query `IN (...)`.

---

## 4. Resumo da Estratégia de Deploy

Para o `alphavel-full` em produção, a estratégia vencedora é:

1.  **Base de Código:** Migrar Controllers críticos para usar `DB::findOne`/`batchFetch`.
2.  **Container:** Usar a imagem Docker simplificada (sem user setup complexo).
3.  **Configuração:** `opcache.validate_timestamps=0`, preloading ativado se possível.
4.  **Monitoramento:** Focar em latência de banco de dados (que foi onde ganhamos 16% de performance).

> **Conclusão:** Não precisamos de "mágica" no servidor. Precisamos de código que respeite o caminho crítico do banco de dados e uma infraestrutura que não atrapalhe.
