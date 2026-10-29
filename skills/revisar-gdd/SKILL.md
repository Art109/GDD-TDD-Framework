---
name: revisar-gdd
description: Analisa e critica um Game Design Document (GDD): pilares, core loop, MDA, psicologia do jogador, escopo e implementabilidade, com veredito e achados por severidade.
---

# Revisar GDD

Revisão crítica de um GDD antes de virar TDD ou código. O objetivo não é elogiar nem reescrever: é encontrar o que vai quebrar, confundir ou estourar o escopo.

**Referência de estrutura:** Modelo de GDD — https://claude.ai/artifact/JUy7Z1va5xa7u3mLbagmcp (cópia em `docs/modelo-gdd.md` do repositório GDD-TDD-Framework). A tabela "Como usar" do modelo diz o que cada estágio exige. Um GDD em outro formato é revisado pelo conteúdo, não pelos nomes das seções.

## Entrada
- O GDD (arquivo, doc ou texto colado). Ler **inteiro** antes de opinar.
- Pergunte (ou assuma e declare) o **estágio**: `protótipo` (padrão), `vertical slice` ou `produção`. O estágio define o rigor: num protótipo, ausência de monetização/localização é OK; ausência de core loop e pilares não é.
- Se houver contexto (tamanho do time, prazo, plataforma), use-o na análise de escopo.

## Processo

1. **Completude** – tabela de seções: Presente / Fraca / Ausente.
   - Obrigatórias em qualquer estágio: Uma página (resumo), Visão (pitch, fantasia do jogador, **metas de experiência**, um momento de jogo), Pilares (3–5, cada um com o que exclui), Público, Objetivo do protótipo (hipótese + metas com forma de medir), Escopo dentro/fora com o porquê, Core loop, Regras dos sistemas do loop, Controles/plataforma, Plano de playtest, Riscos de design, Perguntas em aberto, Registro de decisões.
   - Vertical slice+: Progressão e economia, Conteúdo, Mundo e narrativa, Fluxo do jogo, Arte e áudio completos, Lista de assets com esforço, Onboarding, Glossário.
   - Produção+: Acessibilidade, Localização, Anexo de negócio e mercado completo, Roadmap.
   - Seção com título mas sem conteúdo útil conta como **Fraca**. Texto marcado "inferido" ou "a definir" no corpo conta como Fraca e vira achado.
2. **Uma página** – bate com o resto do documento? Cabe numa página? Se não cabe, o escopo provavelmente está grande demais.
3. **Metas de experiência** – descrevem sensações, não features? Cada uma tem como ser verificada no playtest? Cada mecânica principal serve a pelo menos uma?
4. **Pilares** – cada mecânica deve servir a pelo menos um pilar. Mecânicas sem pilar = *órfãs*. Pilar sem mecânica = *pilar vazio*. Pilar que não exclui nada não ajuda a decidir. Monte uma matriz pilar × mecânica.
5. **Core loop** – está descrito como ciclo (ação → recompensa → motivo para repetir)? Há loops secundários competindo com o principal? Dá para jogar o loop em 30 segundos num protótipo de papel?
6. **MDA** – as Mecânicas geram as Dinâmicas que produzem as Estéticas declaradas (desafio, descoberta, expressão, comunhão, fantasia…)? Aponte desconexões.
7. **Psicologia do jogador** – Autonomia, Competência, Relação (SDT); curva de flow e dificuldade; onboarding ensina jogando?
8. **Consistência interna** – fórmulas vs. regras descritas, tom vs. arte, dificuldade vs. público, controles vs. plataforma, números repetidos com valores diferentes, dependências entre sistemas citadas dos dois lados, termos usados de forma diferente do glossário.
9. **Casos-limite e estratégias dominantes** – o que acontece se o jogador pular etapas, acumular recurso, ignorar um sistema, explorar uma combinação? Existe uma estratégia que torna o resto irrelevante?
10. **Implementabilidade** – cada ficha de sistema tem regras precisas, fórmulas, parâmetros com faixa, casos-limite, critérios de aceitação testáveis, dependências, alternativas consideradas e status? ("ser divertido" ou "bem balanceado" → reescrever como métrica testável, ex.: "jogador médio vence a fase 1 em 2–4 tentativas").
11. **Escopo** – conte sistemas e volume de conteúdo (use a lista de assets, se houver); compare com time/prazo; aponte complexidade escondida (multiplayer, save, IA, física, procedural, UGC). Item de escopo sem porquê é achado.
12. **Higiene do documento vivo** – mecânica cortada ainda descrita no corpo? Lacunas espalhadas fora de Perguntas em aberto? Ideias sem decisão misturadas ao escopo em vez de Ideias adiadas? Detalhe técnico (engine, arquitetura) que deveria estar no TDD?
13. **Estágio** – o documento cumpre o critério para avançar ao próximo estágio? Se o usuário quer avançar, diga o que falta.
14. **Diferencial** – compare com 2–3 jogos parecidos. O gancho cabe num trailer de 30s? A diferença é de experiência ou só cosmética?

## Saída (relatório)

```
# Revisão do GDD – <nome> (estágio: <protótipo|slice|produção>)
Veredito: APROVADO | PRECISA DE REVISÃO | REVISÃO GRANDE | NÃO AVALIADO
Escopo estimado: P | M | G | GG

## Resumo (3–5 linhas)
## Completude (tabela)
## Pontos fortes (no mínimo 2, específicos)
## Achados
### Crítico – bloqueia o protótipo
### Maior – resolver antes do TDD
### Menor
### Sugestão
## Matriz pilar × mecânica
## Metas de experiência × playtest (cada meta e como será verificada)
## Perguntas em aberto para o designer
## Próximos passos (priorizados, máx. 5)
```

Cada achado: **[ID] Título** — Onde (seção + trecho citado) · Problema · Por que importa · Recomendação concreta.

## Regras
- Cite o texto do documento em todo achado; nada de crítica genérica.
- Justifique a severidade. Não infle: num protótipo, falta de polimento é Menor.
- Separe **qualidade do design** de **qualidade da escrita do documento**.
- Inclua ao menos um achado sobre experiência/psicologia do jogador.
- Aponte contradições, mas não decida sozinho qual versão é a certa — vire pergunta.
- Não edite o GDD durante a revisão. Para aplicar mudanças, use a skill `refinar-projeto`.
- Se o documento não existir ou estiver vazio: veredito NÃO AVALIADO e diga o que falta.
