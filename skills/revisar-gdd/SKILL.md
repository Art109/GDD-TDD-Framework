---
name: revisar-gdd
description: Analisa e critica um Game Design Document (GDD): pilares, core loop, MDA, psicologia do jogador, escopo e implementabilidade, com veredito e achados por severidade.
---

# Revisar GDD

Revisão crítica de um GDD antes de virar TDD ou código. O objetivo não é elogiar nem reescrever: é encontrar o que vai quebrar, confundir ou estourar o escopo.

## Entrada
- O GDD (arquivo, doc ou texto colado). Ler **inteiro** antes de opinar.
- Pergunte (ou assuma e declare) o **estágio**: `protótipo` (padrão), `vertical slice` ou `produção`. O estágio define o rigor: num protótipo, ausência de monetização/localização é OK; ausência de core loop e pilares não é.
- Se houver contexto (tamanho do time, prazo, plataforma), use-o na análise de escopo.

## Processo

1. **Completude** – tabela de seções: Presente / Fraca / Ausente.
   - Obrigatórias em qualquer estágio: Visão/Pitch, Pilares de design (3–5), Público-alvo, Core loop, Mecânicas principais, Controles/plataforma, Escopo do protótipo (o que entra e o que NÃO entra).
   - Vertical slice+: Progressão, Economia/fórmulas, Level/mundo, Arte e áudio (direção), UI/UX, Onboarding, Critérios de aceitação.
   - Produção+: Narrativa, Monetização, Acessibilidade, Localização, Análise competitiva, Riscos.
   - Seção com título mas sem conteúdo útil conta como **Fraca**.
2. **Pilares** – cada mecânica deve servir a pelo menos um pilar. Mecânicas sem pilar = *órfãs*. Pilar sem mecânica = *pilar vazio*. Monte uma matriz pilar × mecânica.
3. **Core loop** – está descrito como ciclo (ação → recompensa → motivo para repetir)? Há loops secundários competindo com o principal? Dá para jogar o loop em 30 segundos num protótipo de papel?
4. **MDA** – as Mecânicas geram as Dinâmicas que produzem as Estéticas declaradas (desafio, descoberta, expressão, comunhão, fantasia…)? Aponte desconexões.
5. **Psicologia do jogador** – Autonomia, Competência, Relação (SDT); curva de flow e dificuldade; onboarding ensina jogando?
6. **Consistência interna** – fórmulas vs. regras descritas, tom vs. arte, dificuldade vs. público, controles vs. plataforma, números repetidos com valores diferentes.
7. **Casos-limite e estratégias dominantes** – o que acontece se o jogador pular etapas, acumular recurso, ignorar um sistema, explorar uma combinação? Existe uma estratégia que torna o resto irrelevante?
8. **Implementabilidade** – regras precisas o bastante para programar? Critérios de aceitação mensuráveis? ("ser divertido" ou "bem balanceado" → reescrever como métrica testável, ex.: "jogador médio vence a fase 1 em 2–4 tentativas").
9. **Escopo** – conte sistemas e volume de conteúdo; compare com time/prazo; aponte complexidade escondida (multiplayer, save, IA, física, procedural, UGC).
10. **Diferencial** – compare com 2–3 jogos parecidos. O gancho cabe num trailer de 30s? A diferença é de experiência ou só cosmética?

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
