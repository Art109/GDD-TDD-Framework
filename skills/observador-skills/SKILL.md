---
name: observador-skills
description: Observa o trabalho com as skills do projeto (GDD/TDD, revisão, grilling…), registra correções e padrões num log persistente e, numa revisão periódica, propõe melhorias ou skills novas. Use após tarefas com skills, quando o usuário corrigir o Claude ou pedir revisão das skills.
---

# Observador de skills

Adaptado de `task-observer` ("One Skill to Rule Them All"), de Eoghan Henn e colaboradores, github.com/rebelytics/one-skill-to-rule-them-all, licença CC-BY-4.0. Versão reduzida e modificada.

Skills melhoram com o atrito percebido no trabalho real, não sentando para "melhorar uma skill". Esta skill registra esse atrito para que não se perca entre sessões.

## Onde fica o log
- **Padrão:** um documento do Project anexado, `claude/observacoes-skills.md` (persiste entre sessões e aparece no claude.ai).
- Sem Project: uma pasta no computador do usuário, se a sessão estiver ligada a ele (`.skill-observations/observacoes.md` na pasta de projetos). Sem nenhum dos dois: entregue as observações no fim da resposta para o usuário guardar.
- Leia o log antes de escrever. Escreva pouco e em lote: no fim de uma tarefa, não a cada mensagem.

## Quando observar
Ativa durante trabalho que usa skills (revisar, grelhar, refinar, planejar, escrever GDD/TDD) e nas conversas sobre como o trabalho deve ser feito. **A conversa de feedback depois da tarefa costuma ser a de maior valor.** Inativa em conversa casual e perguntas simples.

## O que anotar
**Melhorar uma skill:**
- o usuário corrigiu o resultado de um jeito que vale sempre ("critério de aceitação tem que ter número");
- o Claude descumpriu uma regra escrita na skill;
- apareceu um caso que a skill não previa;
- um jeito melhor de fazer do que a skill manda;
- a skill afirma algo que a evidência da sessão contradiz;
- um princípio que serve para outras skills também (verifique as "irmãs": `revisar-gdd` e `revisar-tdd`, `brainstorming-jogo` e `grilling-projeto`, `gerar-gdd` e `gerar-tdd`, etc.).

**Simplificar uma skill:** seção que nunca é usada, regra que o usuário sempre pula, regras contraditórias.

**Skill nova:** um fluxo de vários passos que se repetiu, ou quando o usuário diz "eu sempre faço assim".

**Não anotar:** correção pontual que não se generaliza (teste: a frase sobrevive sem o nome deste jogo específico?); preferência já escrita numa skill; bug de ferramenta sem relação com método; dados confidenciais do projeto.

**Antes de criar, procure repetição:** se já existe observação aberta igual (mesma regra, mesmo tipo de falha), acrescente o novo exemplo nela em vez de criar outra.

**Anote os próprios erros.** Descumprir uma regra de skill é observação de primeira classe: é a única evidência de que a regra está fraca.

## Formato de cada observação
```markdown
### OBS-007 · Critérios de aceitação vagos passam na revisão
- status: aberta        # aberta | aplicada | recusada | substituída
- skill: revisar-gdd, refinar-projeto   # ou "nova: <nome provisório>"
- data: AAAA-MM-DD · ocorrências: 1
- **Problema:** o que aconteceu, com exemplo concreto.
- **Melhoria:** a mudança proposta na skill (texto, passo ou checklist).
- **Princípio:** a regra geral por trás, em uma frase.
```
IDs sequenciais a partir do maior existente no log.

## Segunda falha da mesma regra
Se uma regra já falhou duas vezes sem correção aplicada no meio, o problema é de proteção, não de redação. Não proponha um texto mais enfático: proponha uma barreira estrutural, como um passo obrigatório com saída visível, um checklist que precisa ser preenchido no relatório, ou uma verificação por outro agente (ex.: `teste-do-construtor`).

## Revisão
Ofereça uma vez por sessão, em uma linha e sem bloquear o trabalho, quando houver 5 ou mais observações abertas ou a última revisão tiver mais de 7 dias. Rode quando o usuário pedir.

1. Agrupe as observações abertas por skill; junte as repetidas.
2. Para cada skill afetada, leia o SKILL.md **atual** inteiro.
3. Proponha as mudanças: para cada uma, a observação de origem, o que muda e por quê. Inclua simplificações, não só acréscimos.
4. Com a aprovação do usuário, gere o SKILL.md completo e atualizado e apresente pelo card de proposta de skill (o card substitui a skill inteira, então nunca envie só o trecho alterado). Skills novas vão no mesmo card.
5. Atualize o status de cada observação (aplicada/recusada, com o motivo) e registre a data da revisão no topo do log.

Nunca altere uma skill sem aprovação do usuário.
