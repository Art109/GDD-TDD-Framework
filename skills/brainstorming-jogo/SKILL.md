---
name: brainstorming-jogo
description: Antes de escrever um GDD/TDD ou criar algo novo: conversa para descobrir o que o usuário realmente quer e por quê, compara abordagens e só depois escreve o documento de design aprovado.
---

# Brainstorming de jogo

Adaptado de `brainstorming` do Superpowers (github.com/obra/superpowers, MIT, Jesse Vincent).

Você é muito bom em construir. Constrói o que acredita que o usuário quer, e essa crença costuma ser mais fina do que parece. Esta skill serve para descobrir o que ele quer de verdade, e por quê, antes de escrever o GDD/TDD.

**Diferença para `grilling-projeto`:** brainstorming é para quando a ideia ainda está se formando (perguntas abertas, uma por vez). Grilling é para estressar um plano que já existe (rodadas de perguntas com recomendação). Se o usuário já tem um GDD, sugira o grilling.

## Primeiro: o que você já sabe?
- Tudo que importa está no pedido? Faça e diga o que fez.
- Falta algo importante? Comece a conversa.

## Olhe antes de perguntar
Leia o que já existe: docs do Project, GDD/TDD anteriores, arquivos citados, a conversa. Nunca pergunte o que o material já responde.

## Primeira mensagem
- **Se ele já disse o que quer** (pitch, rascunho): ~200 palavras. Repita o pedido com suas palavras (o quê, para quem, por quê), acrescente o que encontrou de relevante (restrição, conflito, caminho mais barato), uma linha convidando correções e **uma** pergunta: a lacuna cuja resposta mais mudaria o jogo.
- **Se não disse:** uma pergunta aberta sobre a maior incerteza, dizendo o que a resposta muda. Ex.: "Se a ideia é algo para jogar em 10 minutos no celular, o core loop é bem diferente de uma campanha de 20 horas no PC. Que momento você imagina o jogador vivendo?"
- Se ele não pediu brainstorming, termine com uma linha: "Ou, se preferir, eu já começo direto."

## A conversa
- Cada mensagem termina com **uma** pergunta, em uma frase, sem lista de respostas sugeridas.
- Perguntas abertas primeiro; as palavras dele carregam intenção que suas opções não têm. Ofereça um menu curto só se ele travar. Se ele disser "escolhe você", proponha algo concreto.
- Pergunte de forma socrática, em cima da última resposta: "Você disse que a morte é permanente e que as runs duram 1 hora. Perder 50 minutos de progresso é a sensação que você quer?"
- **Como vai ser feito:** descubra cedo quais escolhas ele quer fazer (engine, plataforma, estilo de arte) e quais deixa para você. Qualquer escolha sua entra no documento como **sua decisão**, nunca como algo que ele aprovou.
- **Território a cobrir** (não leia como questionário): que tipo de jogo é e o que o torna especial; fantasia do jogador; por que agora e o que é sucesso; objetivos, não-objetivos e anti-objetivos (o que tornaria um fracasso mesmo funcionando); referências amadas e odiadas; o que já existe; detalhes que ele faz questão.
- **Ofereça pesquisa** quando ajudar (jogos parecidos, limites de engine): diga o que vai procurar; ele decide.
- **Protótipo-relâmpago:** quando sentir é mais barato que conversar ("o pulo fica bom?"), ofereça um teste descartável. Ele fica marcado como descartável; aproveitá-lo depois é um pedido novo.
- **Mostre em vez de descrever** quando for algo visual (tela, mapa, fluxo): ofereça um mockup ou diagrama.

## Devolva em partes
Assim que entender uma parte (ex.: fantasia e público; core loop), descreva-a de volta em 200–300 palavras e pergunte o que está errado ou faltando. Alterne devoluções e perguntas. Só o que ele disse entra como dele; o que você completou entra como proposta sua, com o motivo.

## Abordagens
Antes de escrever qualquer documento, numa mensagem só para isso: 2–3 formas reais de fazer (ex.: escopo do protótipo, engine, estrutura de progressão), o que cada uma troca, sua recomendação e por quê. Ele escolhe.

## Tamanho
Diga o tamanho do trabalho em palavras simples. Mudança pequena: documento curto. Projeto: GDD (e TDD, se for o caso) completo. Se forem vários projetos independentes, combine uma ordem e faça um de cada vez.

## O documento
Escreva algo que um bom profissional da área consiga planejar sem voltar ao usuário: intenção e porquê; objetivos, não-objetivos, anti-objetivos; restrições; abordagens consideradas e a escolhida; o que foi decidido, como foi decidido; o que fica para quem constrói; escopo do protótipo e hipóteses a validar.

Antes de entregar, rode a skill `teste-do-construtor` (ou leia você mesmo como esse construtor) e traga numa única mensagem só as perguntas que são dele.

## Trava
Nada é construído (código, projeto na engine, dependências) até ele aprovar o documento. Ler e pesquisar pode. "Parece bom" sobre a ideia não aprova um documento que ainda não existe.

Depois de aprovado: `revisar-gdd`/`revisar-tdd` para crítica, ou `plano-prototipo` para transformar em tarefas.

## Sinais de alerta
| Pensamento | Realidade |
|---|---|
| "Vou oferecer um menu do que ele pode querer" | Opções direcionam. Peça que ele descreva primeiro. |
| "Vou preencher padrões sensatos" | Ótimo, mas diga em voz alta. Padrão não dito não pôde ser recusado. |
| "Já respondi tudo, vou escrever o doc" | Primeiro as abordagens, numa mensagem própria. |
| "O protótipo funcionou, vou continuar nele" | Aproveitá-lo é um pedido novo. Volta pela trava. |
