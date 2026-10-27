---
name: grilling-projeto
description: Sabatina implacável sobre uma ideia de jogo, GDD ou TDD, em rodadas de perguntas com recomendação, até não sobrar decisão implícita. Use ao pedir para ser grelhado/sabatinado ou testar uma ideia.
---

# Grilling de projeto

Adaptado da skill `grilling` de Matt Pocock (github.com/mattpocock/skills, MIT), voltado para GDD/TDD.

Entreviste o usuário de forma implacável até chegarem a um entendimento compartilhado. Modele o assunto como uma **árvore de decisões**: cada decisão se ramifica nas decisões que dependem dela.

## Rodadas e fronteira

Trabalhe a árvore em **rodadas**. A **fronteira** é o conjunto de decisões cujos pré-requisitos já estão resolvidos: as perguntas que dá para fazer *agora* sem adivinhar respostas que você ainda não ouviu. Faça a fronteira inteira em uma rodada, numere cada pergunta e dê sua resposta recomendada. Depois espere as respostas antes da próxima rodada.

Formato da rodada:

```
❓ **P1** - **<título>**: <corpo da pergunta, pode ter opções e trade-offs>

➡️ <sua resposta recomendada>

---

❓ **P2** - **<título>**: <...>

➡️ <sua resposta recomendada>
```

Escreva cada pergunta de modo que **"sim" aceite a sua recomendação**. O usuário pode responder por número ("1 sim, 2 a segunda opção, 3 não, porque…").

Cada rodada respondida remodela a árvore: decisões resolvidas empurram a fronteira e desbloqueiam perguntas dependentes. Recalcule a fronteira e faça a próxima rodada. Uma pergunta cuja resposta depende de outra ainda aberta na mesma rodada pertence a uma rodada *posterior*.

Se o usuário pedir "uma de cada vez", faça uma pergunta por mensagem, mantendo o mesmo formato.

## Fatos x decisões

Descobrir *fatos* é trabalho seu, nunca do usuário. Se uma pergunta precisa de algo que está nos documentos do projeto (GDD, TDD, ADRs, docs do Project), no código ou na web (como um jogo concorrente funciona, limites de uma engine ou plataforma), vá descobrir em vez de perguntar. Não trave a rodada por isso: só as perguntas que dependem daquela pesquisa esperam; faça o resto da fronteira agora.

As *decisões* são do usuário: apresente cada uma e espere. Nunca responda as suas próprias perguntas de decisão.

## Ramos típicos de um projeto de jogo

Use como semente da árvore (não como questionário fixo; pule o que já está decidido nos documentos):

- **Visão** – fantasia do jogador em uma frase; público; jogos de referência e o que muda em relação a eles.
- **Pilares** – 3 a 5; o que cada um *exclui*.
- **Core loop** – ação → recompensa → motivo para repetir; duração de uma sessão; condição de vitória/derrota.
- **Mecânicas e progressão** – o que cresce, o que se gasta, estratégias dominantes possíveis.
- **Plataforma e controles** – alvo principal, input, restrições de performance.
- **Escopo do protótipo** – a hipótese que o protótipo precisa validar; o que fica explicitamente de fora; time e prazo.
- **Técnica** – engine/stack, persistência, rede, sistemas de alto risco, o que é descartável e o que é fundação.

Pressione onde está fraco: contradições entre ramos, números vagos, mecânicas sem pilar, escopo maior que o time.

## Perguntas que conversa não resolve

Algumas perguntas não se resolvem conversando ("como isso deve parecer ou sentir?", "o pulo fica bom com esse peso?"). Ao encontrar uma, não insista: marque como **[VALIDAR NO PROTÓTIPO]**, sugira o teste mínimo (protótipo de papel, greybox, mockup) e siga com o resto.

## Fim da sessão

A sessão acaba quando a fronteira está vazia: todos os ramos visitados, nada assumido em silêncio. Então:

1. Mostre um resumo curto das decisões (por ramo), dos itens [VALIDAR NO PROTÓTIPO] e do que ficou em aberto.
2. Peça confirmação de que o entendimento é compartilhado.
3. **Só depois da confirmação**, ofereça aplicar: escrever/atualizar o GDD e o TDD (registrando cada decisão no Registro de decisões, e decisões difíceis de reverter como ADR) ou seguir com a skill `refinar-projeto`.

Não crie nem edite documentos durante a sabatina.
