# GDD — \<Nome do jogo> (\<Estágio> v0.1)

Oct 27, 2026 · @Arthur

> **Modelo de GDD genérico, independente de engine.** Duplique este documento, troque cada `<campo>` e apague as linhas em itálico ao preencher. O que é técnico (engine, arquitetura, stack, performance) não entra aqui: vai para o TDD.

## Como usar este modelo

O GDD cresce com o estágio. No protótipo, ele responde a uma pergunta e corta todo o resto; no vertical slice e na produção, ganha as seções que antes eram curtas. Mude o estágio no título ao avançar, e registre a mudança no Changelog.

| Seção | Protótipo | Vertical slice | Produção |
| --- | --- | --- | --- |
| Visão | Completa | Completa | Completa |
| Pilares | Completa | Completa | Completa |
| Objetivo do protótipo | Completa | Vira "Objetivo do slice" | Vira "Metas de qualidade" |
| Escopo | Completa | Completa | Completa |
| Core loop | Completa | Completa | Completa |
| Regras dos sistemas | Só os do loop | Todos do slice | Todos |
| Conteúdo | O mínimo do teste | O do slice | Lista completa |
| Progressão e economia | Se o loop depende dela | Completa | Completa, com balanceamento |
| Mundo e narrativa | 3 linhas ou o mínimo testado | Completa | Completa |
| Controles e UX | Completa | Completa | Com acessibilidade |
| Arte e áudio | 3 linhas | Completa | Completa, com guia de estilo |
| Plano de playtest | Completa | Completa | Completa |
| Cronograma | Marcos até o playtest | Marcos do slice | Roadmap |
| Riscos de design | Completa | Completa | Completa |
| Perguntas, decisões, changelog | Sempre | Sempre | Sempre |
| Anexo: negócio e mercado | 3 linhas | Completa | Completa |

Três regras valem em todo estágio:

1. **Números, não adjetivos.** "Divertido", "bem balanceado" e "rápido" viram métricas que dão para medir.
2. **O que não está decidido vira pergunta**, na seção Perguntas em aberto. Nada de "a definir" espalhado pelo texto.
3. **Toda mecânica serve a um pilar.** Mecânica sem pilar sai do escopo ou ganha um pilar.

## Visão

*Uma página no máximo. Quem lê só esta seção precisa saber que jogo é este e por que ele existe.*

**Pitch:** \<uma ou duas frases: o que o jogador faz e o que torna o jogo diferente>

**Fantasia do jogador:** \<o que o jogador sente ou se torna, em uma frase. Ex.: "comandar um grupo pequeno que sobrevive por esperteza, não por força">

**Referências:**

| Jogo ou obra | O que pegamos | O que fazemos diferente |
| --- | --- | --- |
| \<referência 1> | \<mecânica, estrutura ou tom> | \<a diferença de experiência> |
| \<referência 2> |  |  |
| \<referência 3> |  |  |

**Plataforma e formato:** \<plataforma principal, controles, single ou multiplayer; o que é desejo futuro e não compromisso>

**Público:** \<quem joga, que jogos essa pessoa já joga, quanto tempo tem por sessão>

**Tema e tom:** \<cenário, época, tom (aventureiro, sombrio, cômico) e o que ele permite ou exclui>

## Pilares

*De 3 a 5. Cada pilar diz o que exclui: um pilar que não exclui nada não ajuda a decidir.*

| Pilar | O que significa no jogo | O que exclui |
| --- | --- | --- |
| \<Pilar 1> | \<como aparece nas mecânicas> | \<o que não fazemos por causa dele> |
| \<Pilar 2> |  |  |
| \<Pilar 3> |  |  |

*Ex.: "Morrer faz a história andar" significa que cada derrota destrava algo; exclui game over sem recompensa.*

## Objetivo do protótipo

*O protótipo existe para responder uma pergunta. Tudo que não serve a ela sai.*

**Hipótese central:** \<uma frase que o playtest prova ou derruba. Ex.: "o loop run → morte → progressão faz o jogador querer jogar de novo">

**Perguntas secundárias** (medidas, mas não decisivas):

- \<pergunta 1>
- \<pergunta 2>

**Metas de sucesso** (valores iniciais, revisáveis antes do teste):

| Métrica | Meta | Como medir |
| --- | --- | --- |
| \<o que se mede> | \<número ou porcentagem> | \<telemetria, questionário, observação> |
|  |  |  |

**O que o protótipo não valida:** \<o que fica para playtests seguintes: arte, volume de conteúdo, multiplayer…>

## Escopo

*Uma linha por área. Se uma área não entra no protótipo, a coluna "Dentro" diz "nada".*

| Área | Dentro do protótipo | Fora do protótipo |
| --- | --- | --- |
| Personagens / unidades | \<quantos, com o quê> | \<o resto> |
| Inimigos / desafios |  |  |
| Níveis / mapas |  |  |
| Progressão |  |  |
| Narrativa |  |  |
| Interface |  |  |
| Arte e áudio | \<greybox, placeholder ou assets prontos> |  |
| Modo de jogo |  |  |

**Regra de escopo:** \<ex.: "qualquer item novo só entra se tirar outro do mesmo tamanho, ou se for necessário para responder à hipótese central">

## Core loop

*Escreva como ciclo: a ação, a recompensa e o motivo para repetir. Se o jogo tem um loop de sessão e um de longo prazo, descreva os dois e como um alimenta o outro.*

1. \<ação principal do jogador>
2. \<o que acontece em resposta: desafio, encontro, resultado>
3. \<a recompensa, ou a perda>
4. \<o que essa recompensa muda na próxima volta>
5. Volta ao passo 1 com \<o que é diferente agora>

**Duração:** \<de uma volta do loop e de uma sessão típica, em minutos>

**Vitória e derrota:** \<como se ganha, como se perde, e o que a derrota custa ou dá>

## Regras dos sistemas

*Uma subseção por sistema do core loop. É a parte que o programador mais usa: regras precisas, números iniciais, casos-limite e como saber que está pronto. Os números são valores iniciais, calibrados em playtest; no TDD eles viram dados, nunca código.*

### Sistema: \<nome>

**Para que serve:** \<o problema de design que resolve e o pilar que atende>

**Regras:**

1. \<regra precisa o bastante para programar sem perguntar. Ex.: "ataques físicos só atingem a primeira linha inimiga com unidades vivas">
2. \<regra>
3. \<regra>

**Fórmulas:** *uma por cálculo que o sistema faz. Ex.:*

```latex
\text{dano} = \max(1,\ \text{ataque} \times \text{multiplicador} - \text{defesa})
```

**Parâmetros:**

| Parâmetro | Valor inicial | Faixa de ajuste | O que muda ao subir |
| --- | --- | --- | --- |
| \<nome> | \<valor + unidade> | \<mín. – máx.> | \<efeito na experiência> |
|  |  |  |  |

**Casos-limite:** *o que acontece quando o jogador faz o inesperado ou o sistema chega ao extremo.*

- \<ex.: a linha da frente morre no meio do turno; quem atacava ela?>
- \<ex.: recurso zerado, inventário cheio, dois efeitos no mesmo frame>

**Critérios de aceitação:** *cada um dá para testar sozinho, com resultado observável.*

- [ ] \<ex.: com o tanque vivo na linha 1, nenhum ataque físico atinge a linha 2>
- [ ] \<critério>

*Copie a subseção "Sistema" para cada sistema do loop.*

## Conteúdo

*Personagens, unidades, inimigos, itens: o que existe no jogo e por quê. Cada peça de conteúdo existe para criar um problema ou dar uma resposta. Se duas peças fazem o mesmo papel, uma sobra.*

**Personagens ou unidades jogáveis:**

| Nome | Papel no jogo | Como causa efeito | Como sobrevive ou se defende | No protótipo? |
| --- | --- | --- | --- | --- |
| \<nome provisório> | \<estilo de jogo, não função> |  |  | Sim / Não |

**Inimigos ou desafios:**

| Nome | Comportamento | Problema que cria | Respostas esperadas do jogador |
| --- | --- | --- | --- |
| \<nome> | \<o que faz> | \<o que exige do jogador> | \<quais habilidades ou decisões resolvem> |

**Itens, equipamentos ou cartas:** \<tabela no mesmo formato, se o jogo tiver>

## Progressão e economia

*Um papel por recurso. Recurso novo só entra se fizer algo que nenhum outro faz.*

| Recurso | Ganho | Gasto | Persiste? | Tipo de progressão |
| --- | --- | --- | --- | --- |
| \<nome> | \<de onde vem> | \<onde é usado> | \<sim, não, até quando> | \<poder, variedade, acesso, estética> |

**Curva:** \<como a progressão acelera ou desacelera: quando o jogador desbloqueia o quê, em sessões ou horas>

**Sinais de alerta a observar no playtest:** \<ex.: "jogadores acumulam moeda sem gastar = a loja não oferece escolhas interessantes">

## Mundo e narrativa

*No protótipo, três linhas bastam, a menos que a narrativa faça parte da hipótese.*

**Mundo:** \<onde e quando, o que é extraordinário nele, quem está em conflito>

**Como a história chega ao jogador:** \<diálogos, ambiente, cenas, texto; entre partidas ou durante>

**No protótipo:** \<o mínimo de narrativa testado, ex.: "6 falas no hub por personagem, variando com o resultado da run">

## Controles e UX

**Esquema de controle:**

| Ação | Teclado e mouse | Controle | Toque |
| --- | --- | --- | --- |
| \<mover, atacar, confirmar…> |  |  |  |

*Apague as colunas das plataformas que não são alvo.*

**Interface:** \<o que a HUD mostra sempre, o que aparece só quando importa, e as telas principais>

**Onboarding:** \<como o jogador aprende jogando: o que é ensinado primeiro, o que fica para depois, e o que nunca precisa de tutorial>

**Legibilidade:** \<como o jogador entende o que está acontecendo: aviso antes de ataques fortes, silhuetas e cores distintas por tipo, feedback de acerto>

**Acessibilidade** (a partir do vertical slice): \<remapeamento, legendas, opções de daltonismo, dificuldade>

## Arte e áudio

*No protótipo, três linhas: estilo-alvo, referências e de onde vêm os assets.*

**Estilo:** <2D, 3D, pixel art, low poly… e a sensação que ele passa>

**Referências visuais:** <2 ou 3 jogos ou artistas, e o que pegar de cada>

**Paleta e tom:** \<cores dominantes e o que elas comunicam>

**Música e som:** \<estilo, referências, prioridades de SFX>

**Fonte dos assets:** \<produção própria, pacotes prontos, greybox; licenças>

**Risco de consistência:** \<se misturar assets de fontes diferentes, como unificar: paleta, color grading, escala de pixel>

## Plano de playtest

*Mede as metas do Objetivo do protótipo. Cada meta tem de onde vem o número.*

**Participantes:** \<quantos, que perfil, e quantos não conhecem o projeto. Amigos próximos tendem a ser gentis demais: misture perfis>

**Formato da sessão:**

1. \<instrução inicial, o mínimo possível> (\<min>)
2. \<parte obrigatória> (\<min>)
3. \<tempo livre: o testador decide se continua; a decisão é um dado> (\<min>)
4. Questionário curto (\<min>)

**Observação:** \<presencial ou por chamada; o que anotar: onde hesita, se confunde, comenta em voz alta>

**Telemetria:** \<o que o jogo registra por partida, sem backend: duração, causa de derrota, escolhas, recursos ganhos e gastos>

**Questionário:**

1. \<pergunta ligada à hipótese central>
2. \<pergunta aberta: o que te fez continuar ou parar?>
3. \<pergunta de 1 a 5 sobre a sensação principal>
4. O que você mudaria primeiro?

## Cronograma

*Marcos jogáveis: cada um termina com algo que dá para jogar e avaliar. Se atrasar, corta-se escopo, nunca a folga. O detalhamento em tarefas fica no plano do protótipo.*

| Marco | Entrega jogável | Semanas | Pronto quando |
| --- | --- | --- | --- |
| M1 | \<ex.: personagem se move, ataca e recebe dano numa sala de teste> | \<n> | \<critério observável> |
| M2 |  |  |  |
| M3 | \<core loop completo uma vez> |  |  |
| Folga | Buffer para atrasos | \<n> | — |
| Playtest | Sessões e consolidação dos dados | \<n> | Metas comparadas com os resultados |

**Prazo:** \<data alvo ou "sem prazo fixo, estimativa de N semanas">

## Riscos de design

*Riscos de experiência e de escopo. Riscos técnicos vão para o TDD.*

| Risco | Impacto se acontecer | Mitigação | Contingência (se a mitigação falhar) |
| --- | --- | --- | --- |
| \<ex.: escopo cresce> | \<ex.: o playtest nunca acontece> | \<ex.: regra de escopo> | \<ex.: cortar o marco seguinte, nunca a folga> |
|  |  |  |  |

## Perguntas em aberto

*Tudo que ainda não foi decidido mora aqui, e só aqui. Quando uma pergunta é respondida, a resposta vai para a seção certa e para o Registro de decisões.*

- [ ] \<pergunta> — bloqueia: \<o que não pode avançar sem a resposta, ou "nada">
- [ ] \<pergunta> — bloqueia:

## Registro de decisões

*Decisões que alguém pode querer rediscutir. A mais recente em cima.*

| Data | Decisão | Alternativas consideradas | Motivo |
| --- | --- | --- | --- |
| \<AAAA-MM-DD> | \<ex.: batalha em cena separada> | \<ex.: combate no mapa, estilo Chrono Trigger> | \<ex.: 3x mais trabalho na transição, sem ganho para a hipótese> |

## Changelog

| Versão | Data | O que mudou |
| --- | --- | --- |
| v0.1 | \<AAAA-MM-DD> | Primeira versão |

## Anexo: negócio e mercado

*No protótipo, só as três primeiras linhas. O resto passa a ser exigido a partir do vertical slice. Nada aqui entra "inferido": se não foi decidido, vira pergunta em aberto.*

**Modelo de receita:** \<premium, free-to-play, demo gratuita… e por que combina com o jogo>

**Mercado:** \<onde o jogo se posiciona e o que o público do gênero rejeita>

**Concorrentes:** <3 a 5 jogos, e a diferença de experiência em relação a cada um>

*A partir do vertical slice:*

**Preço:** \<faixa e justificativa>

**Canais:** \<lojas e onde divulgar>

**Orçamento:**

| Item | Custo estimado |
| --- | --- |
| \<licenças, assets, música, taxas de loja, marketing> | \<valor + moeda> |

**Lançamento:** \<fases (demo, acesso antecipado, lançamento) e o que dispara cada uma>
