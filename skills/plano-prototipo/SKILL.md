---
name: plano-prototipo
description: Transforma um GDD/TDD aprovado em plano de implementação do protótipo: marcos jogáveis, tarefas pequenas e verificáveis, restrições globais e critério de pronto com evidência.
---

# Plano do protótipo

Adaptado de `writing-plans` e `verification-before-completion` do Superpowers (github.com/obra/superpowers, MIT).

Escreva o plano para alguém competente que **não viu o projeto nem a conversa**. Ele sabe programar na engine; o que não sabe é o que foi decidido: quais sistemas, nomes, valores do GDD, e como provar que cada tarefa funciona. Documente isso. YAGNI: nada que o protótipo não precise para validar suas hipóteses.

## Antes de começar
- Leia GDD e TDD inteiros. Se não estiverem aprovados, sugira `brainstorming-jogo` ou `revisar-gdd` primeiro.
- **Escopo:** se houver subsistemas independentes, proponha um plano por subsistema; cada um deve gerar algo jogável ou testável sozinho.

## Estrutura
Mapeie antes das tarefas quais pastas, cenas, scripts e assets serão criados e a responsabilidade de cada um. Arquivos com uma responsabilidade clara; o que muda junto fica junto; siga os padrões do projeto se já existir.

## Cabeçalho obrigatório
```markdown
# Plano do protótipo – <nome>
**Objetivo:** <uma frase>
**Hipóteses que o protótipo valida:** <do GDD, com como medir>
**Arquitetura:** <2–3 frases>
**Engine/stack:** <versões>
**Documentos de origem:** <GDD e TDD, com versão>

## Restrições globais
<requisitos do projeto todo, um por linha, valores copiados literalmente do GDD/TDD: plataforma, resolução, FPS alvo, controles, nomes de sistemas, números de balanceamento>

## Foco de revisão
<as 5 situações que o GDD sugere mas nenhuma tarefa testa e que mais podem estragar a experiência (ex.: morrer durante a transição de fase, inventario cheio, pausar no meio do pulo). Para cada uma, adicione o teste na tarefa responsável.>
```

## Marcos e tarefas
- Organize em **marcos jogáveis** (ex.: M1 personagem se move e pula; M2 um inimigo e dano; M3 core loop completo uma vez). Cada marco termina com algo que dá para jogar e avaliar.
- **Tarefa** = menor unidade com entrega verificável que um revisor poderia rejeitar sem rejeitar a vizinha. Configuração e documentação entram na tarefa que precisa delas.
- **Passos** em checkbox, cada um com resultado verificável: escrever o teste (ou o roteiro de teste manual), ver falhar, implementar o mínimo, ver passar, commit.
- Para o que só se avalia jogando (sensação, ritmo), escreva um **roteiro de teste manual** com o resultado esperado em termos observáveis.
- Toda tarefa referencia a seção do GDD/TDD de onde vem; tarefa sem origem é corte ou pergunta.

Salve o plano como documento do projeto (ou onde o usuário preferir).

## Pronto só com evidência
Na execução, nenhuma tarefa ou marco é declarado pronto sem evidência nova na mesma mensagem: saída dos testes, build que roda, roteiro manual executado com o resultado. "Deve funcionar", "parece certo" ou o relato de um subagente não são evidência. Se a verificação não pôde ser feita, diga o status real.

## Depois
Rode `teste-do-construtor` no plano (papel: programador sem contexto) antes de entregar.
