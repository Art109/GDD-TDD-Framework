---
name: gerar-gdd
description: Cria um GDD novo a partir do Modelo de GDD do usuário, com as seções do estágio (protótipo, vertical slice, produção), usando o que saiu de brainstorming, grilling ou de um pitch.
---

# Gerar GDD

Cria o GDD de um jogo seguindo o modelo do usuário. O modelo é a fonte da verdade da estrutura: leia-o sempre, não reconstrua de memória.

**Modelo:** Modelo de GDD — https://claude.ai/artifact/JUy7Z1va5xa7u3mLbagmcp (cópia no repositório: `docs/modelo-gdd.md`) (doc do Claude; ler com as ferramentas de docs). Mapa do kit: documento `claude/kit-gdd-tdd.md` do Project.

## Entrada
- O material do jogo: resultado de `brainstorming-jogo` ou `grilling-projeto` na conversa, um pitch, ou um GDD antigo em outro formato (PDF, export de ferramenta, doc).
- Estágio: `protótipo` (padrão), `vertical slice` ou `produção`.
- Se o material for só uma ideia vaga, sugira `brainstorming-jogo` antes. Se for um plano que ainda não foi pressionado, ofereça `grilling-projeto` (o usuário decide).

## Processo
1. **Ler o modelo inteiro** e a tabela "Como usar": ela diz o que cada seção exige no estágio escolhido.
2. **Mapear o material para as seções.** Para cada seção: o que o material já responde, o que falta. Num GDD antigo, aproveite o conteúdo bom e reorganize; não copie texto marcado como "inferido" ou "a definir" como se fosse decisão.
3. **Perguntar só o que bloqueia.** Se faltar algo obrigatório no estágio (hipótese central, pilares, escopo dentro/fora, core loop, regras do sistema principal), pergunte em uma rodada, no formato do grilling (numerado, com recomendação, "sim" aceita). O resto vai direto para Perguntas em aberto.
4. **Criar o documento** como doc novo do Claude, com o título `GDD — <Nome> (<Estágio> v0.1)`, seguindo a ordem e os nomes de seção do modelo. Preencha seção por seção.
5. **Regras de preenchimento:**
   - Apague as linhas em itálico de orientação e todos os `<campos>`; seção sem conteúdo no estágio fica com uma linha ("Fora do protótipo") em vez de placeholder.
   - Números, não adjetivos. Valor que o usuário não deu entra como valor inicial marcado `[proposta]` ou vira pergunta.
   - Nada de "a definir" no corpo: toda lacuna vai para Perguntas em aberto, com o que ela bloqueia.
   - Toda mecânica liga a um pilar. Mecânica sem pilar: pergunte se sai do escopo.
   - Nada técnico (engine, arquitetura, stack): anote para o TDD.
   - Regras dos sistemas com regras numeradas, fórmulas, parâmetros, casos-limite e critérios de aceitação testáveis.
   - Decisões já tomadas na conversa entram no Registro de decisões, com alternativas e motivo.
   - Changelog com a v0.1 e a origem do material.
6. **Conferir** contra a tabela do estágio: toda seção obrigatória preenchida ou com pergunta explícita.

## Saída
- O GDD como doc do Claude, aberto para o usuário.
- Uma linha com o que ficou em Perguntas em aberto e o próximo passo sugerido: `revisar-gdd`, ou `gerar-tdd` se o usuário quiser seguir.

## Regras
- Nunca altere o Modelo de GDD; o GDD é um documento novo.
- Se o modelo não puder ser lido, diga isso e peça o link, em vez de improvisar a estrutura.
