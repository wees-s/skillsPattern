---
name: error-memory
description: Memória persistente de erros e soluções aprendidos. Evita retry loops desnecessários consultando soluções já descobertas antes de tentar novamente. Trigger AUTOMÁTICO após 2 falhas consecutivas na mesma operação. Trigger MANUAL via "consultar erros", "buscar solução", "já resolvemos isso?", "error memory", "erros conhecidos", "learned errors", "known errors".
---

# Error Memory — Aprendizado Persistente de Erros

> **Princípio:** nunca repita o mesmo erro duas vezes. Esta skill mantém uma base de conhecimento de problemas encontrados e suas soluções, organizada por categoria.

## Quando acionar

### Trigger automático (obrigatório)
Após **2 falhas consecutivas na mesma operação** (ex.: mesmo comando bash, mesmo erro de build, mesma exceção Python), **PARE** de tentar novamente e:
1. Consulte `errors/INDEX.md` antes de qualquer terceira tentativa.
2. Se houver entrada relevante, aplique a solução documentada.
3. Se não houver, investigue a causa raiz antes de tentar de novo.

> "Mesma operação" = mesma intenção, mesmo comando ou erro de natureza equivalente. Trocar `npm install` por `npm i` continua sendo a mesma operação.

### Trigger manual
O usuário diz qualquer destas frases (PT ou EN):
- "consultar erros"
- "buscar solução"
- "já resolvemos isso?"
- "error memory"
- "erros conhecidos"
- "learned errors"
- "known errors"

Ao receber qualquer um desses gatilhos, leia `errors/INDEX.md` e responda com base no que encontrar.

## Estrutura da base de conhecimento

```
~/.claude/skills/error-memory/
├── SKILL.md                  ← este arquivo
└── errors/
    ├── INDEX.md              ← índice navegável de todos os erros
    ├── build/                ← compilação, lint, type-check
    ├── runtime/              ← exceções em execução
    ├── deploy/               ← CI/CD, Docker, infraestrutura
    ├── dependency/           ← pacotes, versões, conflitos
    ├── config/               ← env vars, settings, tooling
    ├── git/                  ← versionamento, hooks, conflitos
    ├── database/             ← migrations, queries, schema
    ├── network/              ← APIs, fetch, timeouts, CORS
    ├── auth/                 ← tokens, permissões, sessão
    └── other/                ← o que não couber acima
```

Cada categoria é criada sob demanda. Não pré-criar pastas vazias.

## Formato de cada entrada de erro

Arquivo: `errors/<categoria>/<slug-curto>.md`

```markdown
---
id: <slug-curto>
category: <categoria>
title: <título conciso do erro>
first_seen: YYYY-MM-DD
last_seen: YYYY-MM-DD
occurrences: <int>
stack: <ex.: FastAPI, Python 3.12, Docker>
severity: low | medium | high
---

## Sintoma
<mensagem de erro literal, traceback resumido, ou descrição do comportamento observado>

## Contexto
<o que o usuário/agente estava fazendo quando o erro apareceu — comando, arquivo, fluxo>

## Causa raiz
<por que aconteceu, em uma frase ou duas>

## Solução
<passos exatos que resolveram — comandos, edits, configs>

## Como detectar de novo
<sinais que indicam ser este mesmo problema — mensagem chave, padrão de log, pista>

## Não confundir com
<erros parecidos que NÃO se resolvem com esta solução, se houver>
```

## Formato do INDEX.md

```markdown
# Error Memory — Index

Última atualização: YYYY-MM-DD

## Por categoria

### build
- [<slug>](build/<slug>.md) — <título> · <stack> · <ocorrências>x

### runtime
- [<slug>](runtime/<slug>.md) — <título> · <stack> · <ocorrências>x

...
```

Mantenha o INDEX como **índice puro** (uma linha por entrada). Detalhes ficam só no arquivo da entrada.

## Fluxo de uso

### Ao consultar (trigger automático ou manual)
1. Leia `errors/INDEX.md` inteiro (é curto por design).
2. Filtre mentalmente por categoria provável + palavras-chave do erro atual.
3. Abra apenas os arquivos candidatos (1-3 no máximo).
4. Se houver match, aplique a **Solução** e mencione ao usuário: "já vimos isso antes — [link/slug]".
5. Atualize `last_seen` e incremente `occurrences` na entrada correspondente.
6. Se não houver match, siga investigando normalmente.

### Ao registrar um novo erro resolvido
Registre **apenas após confirmação de que a solução funcionou** (build verde, teste passa, comando retorna 0). Critérios para criar uma entrada:

- O erro consumiu pelo menos 2 tentativas para resolver, **ou**
- A causa raiz não é óbvia a partir da mensagem, **ou**
- A solução envolveu uma config/flag/workaround específico que pode reaparecer.

**Não** criar entrada para:
- Typos triviais corrigidos na primeira tentativa.
- Erros cuja mensagem já diz exatamente o que fazer (ex.: "missing semicolon at line 42").
- Problemas de ambiente exclusivos da sessão atual (arquivo aberto em outro processo, etc.).

Passos para registrar:
1. Escolha categoria e crie o arquivo `errors/<categoria>/<slug>.md` no formato acima.
2. Adicione uma linha no `errors/INDEX.md` na seção da categoria.
3. Se a categoria ainda não existir no INDEX, adicione a seção em ordem alfabética.

### Ao reincidir um erro já registrado
1. Atualize `last_seen` para a data atual.
2. Incremente `occurrences`.
3. Se a solução antiga não funcionou mais, **não sobrescreva** — adicione uma seção `## Solução (variante <data>)` documentando a nova rota.

## Regras de qualidade

- **Slugs curtos e estáveis:** `kebab-case`, ≤ 4 palavras (ex.: `alembic-multiple-heads`, `docker-port-already-bound`).
- **Sintoma literal:** copie a mensagem de erro real, não parafraseie. Quem busca depois vai grepar pela string.
- **Solução acionável:** comandos exatos, caminhos absolutos quando relevante, sem "tente algo assim".
- **Sem ruído:** nada de "este erro é frustrante", "espero que ajude", emojis. Apenas fato.
- **Datas absolutas:** sempre `YYYY-MM-DD`, nunca "ontem" ou "semana passada".

## Limites

- Esta skill **não substitui** memórias do sistema padrão (`~/.claude/projects/.../memory/`). Memórias gerais sobre o usuário, projeto e feedback continuam lá.
- Esta skill é **específica para erros e soluções técnicas reproduzíveis**.
- Se um erro for específico de um único projeto e improvável em outros contextos, prefira documentar dentro do próprio projeto (CLAUDE.md ou docs/) em vez de aqui.

## Manutenção

- Quando uma entrada não é vista há mais de 12 meses (`last_seen` antigo) e a stack mudou, marque como `archived: true` no frontmatter em vez de deletar — o histórico ainda pode ser útil.
- Se duas entradas descrevem o mesmo problema, mescle a mais antiga na mais recente e remova a duplicata do INDEX.
