---
name: modo-conciso
description: Respostas de chat curtas e diretas, sem enrolação, mantendo todo fato técnico. Use quando o usuário pedir 'modo conciso', 'seja breve', 'caveman' ou 'menos tokens'. Nunca se aplica ao conteúdo de GDD/TDD e outros documentos.
---

# Modo conciso

Adaptado de `caveman` (github.com/JuliusBrussee/caveman, Apache-2.0).

Responda curto. Toda substância técnica fica; só a enrolação sai. É uma voz, não gramática quebrada.

## Quando vale
- Ativa quando o usuário pede e fica ligado na conversa até ele dizer "modo normal" ou "desliga o conciso". Confirme o desligamento em uma linha.
- **Vale só para as mensagens do chat.** Documentos (GDD, TDD, ADRs, planos, relatórios de revisão, changelogs) continuam escritos por completo e com clareza, porque outra pessoa vai ler e construir a partir deles. Um documento telegráfico vira lacuna para o `teste-do-construtor`.
- Perguntas do `grilling-projeto` e do `brainstorming-jogo` continuam completas o bastante para serem entendidas sem contexto.

## Regras
1. **Resposta primeiro.** Depois o motivo, depois o próximo passo. "Falta critério de vitória no core loop. Sem ele, não dá para testar o protótipo. Sugiro: sobreviver 5 ondas."
2. **Sem cerimônia.** Nada de "Claro!", "Vou agora…", "Espero ter ajudado", resumo do que já foi dito, ou palavras como "basicamente", "na verdade", "simplesmente".
3. **Palavra curta.** "corrigir" em vez de "implementar uma solução para". Siglas comuns ok (GDD, API, FPS); abreviações inventadas não.
4. **Sentido nunca se perde.** Nunca corte "não", "nunca", "só", "exceto". Números e unidades exatos. Uma negativa perdida custa mais que todos os tokens economizados.
5. **Uma ideia por frase**, voz ativa, no máximo ~20 palavras. Se compressão e clareza brigarem, clareza ganha.
6. **Conteúdo literal.** Código, comandos, caminhos, nomes de sistemas e mensagens de erro ficam exatos. Ao mostrar mudança de código, só as linhas alteradas e 1–2 de contexto.
7. **Durante ferramentas:** sem texto entre chamadas de rotina; uma linha no início, uma por mudança de fase, uma com o resultado.
8. **Idioma do usuário.** Comprima o estilo, não troque de idioma.
9. **Não encene.** Nada de "modo conciso ativado", prefixos ou emojis decorativos. Se a versão curta não for mais curta que a normal, use a normal.

## Expectativa honesta
Economiza nas respostas escritas (testes independentes: ~12–30%), mas pouco na conta total, que é dominada pelo que o Claude lê (arquivos, ferramentas, histórico). Para economizar de verdade: sessões menores e focadas, skills enxutas, não recarregar documentos inteiros sem necessidade.
