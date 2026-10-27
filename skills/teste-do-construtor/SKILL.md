---
name: teste-do-construtor
description: Testa se um GDD, TDD ou plano está pronto para ser construído: um leitor sem contexto (subagente) lê como programador/designer e lista só as perguntas em que teria de adivinhar.
---

# Teste do construtor

Adaptado do *builder check* do `brainstorming` do Superpowers (github.com/obra/superpowers, MIT).

Um documento parece completo para quem escreveu, porque o autor preenche as lacunas de cabeça. Este teste entrega o documento a alguém que **não viu a conversa** e pergunta onde ele teria de chutar.

## Como rodar

1. Escolha o papel do leitor conforme o documento: GDD → "game designer e programador de gameplay"; TDD → "engenheiro de software de jogos experiente em <engine>"; plano → "programador júnior competente sem contexto do projeto".
2. Dispare um subagente **novo** (sem o contexto da conversa) com o prompt abaixo. Passe o documento colado ou o caminho. Se não houver ferramenta de subagente, leia você mesmo nesse papel, ignorando o que sabe da conversa.

```
Você é um(a) [PAPEL]. Recebeu este documento e precisa montar um plano
para construir o que ele descreve. Você não poderá falar com quem escreveu.

Leia [DOCUMENTO].

Liste apenas as perguntas que ainda precisaria ver respondidas para planejar
bem: pontos em que teria de adivinhar e um chute errado faria diferença.
Para cada uma: a pergunta e por que a resposta muda o plano.
Ordene por importância.

Ignore o que o documento deixa explicitamente para quem constrói, o que
você poderia decidir sozinho de forma razoável e detalhes de redação.

Se não faltar nada importante, responda "nada importante faltando".
```

## Triagem do resultado
Separe as perguntas em três pilhas:
- **Já respondida** pela conversa ou por outro documento do projeto: escreva a resposta no documento.
- **Menor:** um construtor sensato resolveria sem mudar o que é construído. Decida e marque no documento como `[decisão do autor]`.
- **Do usuário:** só ele pode responder e a resposta muda o que é construído. Leve numa única mensagem, a mais importante primeiro (o formato de rodada da `grilling-projeto` funciona bem aqui).

Atualize o documento com as respostas e entregue com uma lista curta das decisões que você tomou, para ele conferir.

## Regras
- Uma rodada só de perguntas por teste; não vire interrogatório.
- O subagente não edita nada; só reporta.
- Rode de novo depois de mudanças grandes.
