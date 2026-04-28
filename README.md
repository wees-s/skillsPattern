# Claude Skills

Conjunto de skills para padronizar comportamento em execução assistida por IA.

Foco: reduzir retrabalho, evitar repetição de erros e tornar decisões mais determinísticas.

---

## Objetivo

Centralizar regras operacionais que:

* evitam tentativa e erro desnecessária
* reaproveitam soluções já validadas
* impõem fluxo consistente de execução

---

## Estrutura

```text
skills/
├── error-memory/
│   ├── SKILL.md
│   └── errors/
```

Cada skill é isolada e responsável por um tipo específico de problema.

---

## Skills

### error-memory

Mantém um histórico persistente de erros técnicos e suas respectivas soluções.

#### Responsabilidades

* interromper retry após falhas repetidas
* consultar base de erros antes de novas tentativas
* registrar soluções validadas
* atualizar recorrência de erros conhecidos

#### Trigger automático

Após 2 falhas consecutivas na mesma operação:

1. interromper execução
2. consultar `errors/INDEX.md`
3. aplicar solução existente ou investigar causa raiz

#### Trigger manual

Ativado por comandos de consulta explícita (ex: buscar solução, erros conhecidos).

---

## Princípios

* falha repetida sem consulta = erro de processo
* solução não registrada = perda de informação
* execução sem contexto = comportamento não determinístico

---

## Regras gerais

* não insistir após falha recorrente sem consulta
* não registrar erros triviais
* não sobrescrever soluções existentes (usar variações)
* manter histórico incremental

---

## Escopo

* erros técnicos reproduzíveis
* problemas com solução validada
* casos com potencial de recorrência

Fora de escopo:

* erros triviais
* problemas temporários de ambiente
* conhecimento específico de projeto (deve ficar no próprio projeto)

---

## Uso

As skills são acopladas ao fluxo de execução do agente.

Não são utilitários isolados, mas regras que devem ser seguidas durante:

* desenvolvimento
* debugging
* automação

---

## Extensão

Novas skills devem:

* ter escopo bem definido
* resolver um problema recorrente específico
* possuir triggers claros (automático/manual)
* manter baixo acoplamento com outras skills

---

## Manutenção

* revisar entradas antigas periodicamente
* arquivar erros obsoletos
* evitar duplicação de conhecimento

---

## Estado atual

* error-memory implementada
* demais skills ainda não definidas
