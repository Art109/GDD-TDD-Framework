# TDD — \<Nome do jogo> (\<Estágio> v0.1)

Oct 27, 2026 · @Arthur

> **Modelo de TDD genérico.** Duplique este documento, troque cada `<campo>` e apague as linhas em itálico ao preencher. O TDD diz como **este** jogo é construído. As regras gerais de arquitetura ficam no guia da engine ([Godot](arquitetura-godot.md) ou [Unity](arquitetura-unity.md)) e não se repetem aqui: o TDD registra só como o jogo as aplica e onde se desvia delas.

## Como usar este modelo

Três documentos trabalham juntos: o **GDD** diz o que o jogo faz, o **guia de arquitetura** diz como qualquer jogo é organizado, e este **TDD** diz como este jogo aplica o guia aos sistemas do GDD. Toda seção técnica nasce de algo do GDD; sistema sem origem no GDD é corte ou pergunta.

| Seção | Protótipo | Vertical slice | Produção |
| --- | --- | --- | --- |
| Resumo, decisões e stack | Completa | Completa | Completa |
| Adaptação do guia | Completa | Completa | Completa |
| Rastreabilidade | Completa | Completa | Completa |
| Sistemas, componentes e dados | Os do core loop | Todos do slice | Todos |
| Orçamentos de performance | FPS e plataforma-alvo | Completa | Completa, por plataforma |
| Estado atual e migração | Só se já existe código | Só se já existe código | Só se já existe código |
| Riscos técnicos e spikes | Completa | Completa | Completa |
| Testes e ferramentas | O mínimo do guia | Com build e distribuição | Com CI e telemetria |
| ADRs, pendências, changelog | Sempre | Sempre | Sempre |
| Metas de qualidade, restrições | Completa | Completa | Completa |
| Visão de blocos e cenários | Diagrama + 2 cenários | Completa | Completa |
| Conceitos transversais, dívida técnica, glossário | O que já existe | Completa | Completa |

*Itens que entram só em produção (CI/CD, rede, anti-cheat, localização, migração de save entre versões) ganham a própria subseção em Sistemas quando chegarem.*

**Como manter:** atualize o TDD quando o design mudar, antes de cada marco e antes de qualquer build pública. Uma ADR aceita não é reescrita: se a decisão mudar, uma ADR nova a substitui. Depois do playtest, releia as decisões principais e registre o que se confirmou e o que não.

## Resumo

*Três a cinco frases: como o jogo é montado e o que é mais arriscado tecnicamente.*

\<resumo>

**Documentos de origem:** GDD \<link> v\<n> · Guia de arquitetura \<Godot | Unity> \<link>, revisão de \<data>

**Decisões principais:** *as que mais restringem o resto. Cada uma tem um ADR no fim do documento.*

1. \<ex.: batalha como sub-cena sobreposta, não como troca de level>
2. \<ex.: save só a partir do vertical slice; no protótipo o estado vive em Systems sem persistência>
3. \<decisão>

**Stack:**

| Camada | Escolha | Motivo |
| --- | --- | --- |
| Engine e versão | \<Godot 4.x / Unity 6.x, versão exata fixada> |  |
| Linguagem | \<GDScript / C#> |  |
| Render | \<Forward+, Compatibility / URP, HDRP> |  |
| Input | \<InputMap / Input System> |  |
| Física e navegação | \<o que o jogo usa> |  |
| Dados | \<Resources Data / ScriptableObjects Data> | Regra do guia |
| Save | \<JSON em user:// / persistentDataPath, ou "sem save no protótipo"> |  |
| Testes | \<GUT / Unity Test Framework> | Regra do guia |
| Versionamento | \<Git + LFS> |  |
| Distribuição | \<ex.: build Windows em página privada no itch.io> |  |

**Plataformas-alvo:** \<plataforma, resolução, input, requisito mínimo>

**Fora do escopo técnico:** \<o que não será construído neste estágio: rede, mods, localização…>

**Objetivos e não-objetivos técnicos:** *não-objetivo é algo que poderia razoavelmente ser objetivo e foi excluído de propósito.*

- Objetivo: \<ex.: trocar números de balanceamento sem recompilar nem mexer em código>
- Não-objetivo: \<ex.: suportar mods; rodar em celular neste estágio>

**Restrições:** \<o que não está em discussão: prazo, tamanho do time, plataforma, licenças de assets e de pacotes, hardware de teste>

**Metas de qualidade:** *as três mais importantes, em ordem. Quando duas brigarem, vence a de cima. Cada uma vira um cenário que dá para verificar.*

| Prioridade | Qualidade | Cenário verificável |
| --- | --- | --- |
| 1 | \<ex.: rapidez de iteração> | \<ex.: um designer muda o dano de uma arma e testa em menos de 30 s, sem programador> |
| 2 | \<ex.: performance> | \<ex.: 60 FPS estáveis na cena mais cheia, na máquina de referência> |
| 3 | \<ex.: reaproveitamento> | \<ex.: os componentes de combate passam no teste do projeto vazio> |

## Adaptação do guia de arquitetura

*As respostas ao roteiro "Como adaptar a um projeto" do guia. Cada item vira uma linha aqui.*

**Dimensão e câmera:** <2D lateral, 2D top-down com Y-sort, 3D; uma câmera ou várias>

**Sistemas e `GameContext`:**

| Sistema | Responsabilidade | Campo no `GameContext` |
| --- | --- | --- |
| \<SaveSystem> | \<o que guarda e quando salva> | \<save> |
| \<FlagSystem> |  |  |
| \<ClockSystem> |  |  |

**Wirings:**

| Wiring | Domínio | Liga o quê a quê |
| --- | --- | --- |
| \<HudWiring> | HUD | \<vida do player → HUD> |

**`GameEvents`** (só fatos com 3+ ouvintes):

| Evento | Quem dispara | Quem escuta |
| --- | --- | --- |
| \<boss\_defeated(id)> | \<Boss> | \<música, save, conquistas> |

**Sub-cenas sobrepostas:** \<quais existem (batalha, loja, diálogo), quem as pede, o que recebem e o que devolvem; se o mundo fica visível atrás>

**Taxonomia de estado:**

| Dado | Categoria | Onde mora |
| --- | --- | --- |
| \<inventário> | \<Persistente> | \<InventorySystem, salvo> |
| \<inimigos mortos no mapa> | \<Sessão> |  |

**Desvios do guia:** *toda regra do guia que este jogo não segue, com o motivo. Sem motivo, o desvio não entra.*

| Regra do guia | Desvio | Motivo | Revisar quando |
| --- | --- | --- | --- |
| \<regra> | \<o que fazemos diferente> | \<por quê> | \<gatilho para voltar à regra> |

## Rastreabilidade GDD → TDD

*Uma linha por sistema ou mecânica do GDD. Linha sem sistema técnico = lacuna. Sistema técnico sem linha = algo que o GDD não pediu.*

| Seção do GDD | Mecânica ou sistema | Sistema técnico | Componentes | Dados (`Data`) | Marco |
| --- | --- | --- | --- | --- | --- |
| \<Regras dos sistemas › Combate> | \<prioridade de linha> | \<BattleRules> | \<InitiativeQueue, HealthComponent> | \<UnitData, SkillData> | \<M2> |
|  |  |  |  |  |  |

**Lacunas:** \<mecânicas do GDD ainda sem sistema técnico, cada uma com o motivo ou a pergunta que falta>

## Visão de blocos e cenários

**Blocos:** *um diagrama de como as partes deste jogo se encaixam: Main, sistemas, levels, sub-cenas e os principais componentes, com quem chama quem e quem avisa quem. Um nível de detalhe só; o detalhe de cada sistema fica na próxima seção.*

\<diagrama>

**Cenários de execução:** *de dois a quatro fluxos que mais podem dar errado, passo a passo, com os nós e sinais envolvidos. Escolha os que cruzam vários sistemas.*

1. \<ex.: o jogador encosta num inimigo → o inimigo emite `encounter_requested` → o Wiring chama `open_overlay(batalha)` → o mundo pausa → a batalha devolve o resultado → o Wiring aplica XP e remove o inimigo>
2. \<ex.: troca de level com save automático>
3. \<ex.: morte do player no meio de uma transição>

## Sistemas técnicos

*Uma subseção por sistema da tabela de rastreabilidade. O suficiente para dividir em tarefas sem adivinhar.*

### Sistema: \<nome>

**Origem no GDD:** \<seção e regras que ele implementa>

**Responsabilidade:** \<o que ele faz e, também, o que ele não faz>

**Onde vive:** <`game/systems/`, cena do level, sub-cena, componente portável>

**Interface:**

| Tipo | Nome | Descrição |
| --- | --- | --- |
| Método | <`roll(table, rng)`> | \<o que recebe e devolve> |
| Sinal / evento | <`turn_started(unit)`> | \<quando dispara> |

**Estado:** \<o que ele guarda e em que categoria da taxonomia>

**Casos-limite técnicos:** \<ordem de eventos no mesmo frame, entidade destruída no meio da ação, carga assíncrona, save no meio da transição>

**Pronto quando:**

- [ ] \<critério de aceitação do GDD, como teste ou roteiro manual>
- [ ] \<teste automatizado, se o sistema só calcula>

*Copie a subseção para cada sistema.*

## Componentes

| Componente | Origem | Responsabilidade | Exige do projeto |
| --- | --- | --- | --- |
| \<HealthComponent> | \<do guia / novo / existente no projeto> | \<vida e dano> | \<camadas no Inspector> |

*Componente novo nasce em `game/` e só vai para `components/` quando passar no teste do projeto vazio.*

## Dados

| `Data` | Campos principais | Instâncias no protótipo | Parâmetros do GDD que guarda |
| --- | --- | --- | --- |
| \<UnitData> | \<vida, ataque, defesa, velocidade> | <4 unidades> | \<tabela de parâmetros de Combate> |

*Todo número de balanceamento do GDD mora num `Data`. Nenhum valor de gameplay no código.*

## Conceitos transversais

*Decisões que valem para o jogo todo. Uma linha cada; o que o guia já decide, só aponte para ele.*

| Conceito | Como este jogo faz |
| --- | --- |
| Save e versão do save | \<o que salva, quando, formato, migração entre versões> |
| Input e rebinding | \<ações, dispositivos, troca de controle> |
| Pausa e tempo | \<o que para, o que continua, tempo de jogo x tempo real> |
| Erros e falhas | \<o que acontece se um level, um save ou um asset não carregar> |
| Logs e depuração | \<ferramentas de debug, como ligar e desligar> |
| Localização e texto | \<onde ficam os textos, mesmo que só exista um idioma> |
| Acessibilidade | \<opções previstas e onde entram na arquitetura> |

## Orçamentos de performance

*Números, não "deve rodar bem". Medidos na máquina de referência, no pior caso do jogo (a cena mais cheia).*

**Máquina de referência:** \<CPU, GPU, RAM, ou o requisito mínimo da plataforma>

| Métrica | Orçamento | Como medir | Estágio |
| --- | --- | --- | --- |
| Taxa de quadros | <60 FPS estáveis> | \<profiler, na cena mais cheia> | Protótipo |
| Tempo de frame | <16,6 ms; CPU < 10 ms> | \<profiler> | Vertical slice |
| Memória | << 1,5 GB> | \<monitor de memória> | Vertical slice |
| Carregamento de level | << 2 s> | \<log de tempo na transição> | Vertical slice |
| Tamanho do build | \<limite da loja ou do público> | \<tamanho do pacote> | Produção |
| Entidades simultâneas | \<pior caso do GDD> | \<cena de estresse> | Vertical slice |

**Práticas que protegem o orçamento:** \<reaproveitar projéteis e efeitos em pool, exportar texturas no tamanho exibido, carregar levels de forma assíncrona>

## Estado atual e plano de migração

*Só quando já existe código. Num projeto novo, apague esta seção.*

**Problemas encontrados:**

| Onde | Problema | Consequência | Regra do guia que resolve |
| --- | --- | --- | --- |
| \<arquivo ou nó> | \<ex.: troca de mapa com `change_scene_to_file()`> | \<ex.: estado do mapa some ao voltar> | \<ex.: `load_level()` único ponto de troca> |

**Plano de migração:** *cada etapa deixa o jogo jogável. A ordem segue as dependências: o que injeta vem antes do que recebe.*

1. \<etapa> — pronto quando: \<o jogo roda e o problema X sumiu>
2. \<etapa> — pronto quando:
3. \<etapa> — pronto quando:

## Riscos técnicos

*Todo risco alto ganha um spike: um teste descartável, com prazo curto, que responde se a abordagem funciona antes de o resto depender dela.*

| Risco | Probabilidade | Impacto | Mitigação ou spike | Contingência |
| --- | --- | --- | --- | --- |
| \<ex.: IA tática dos inimigos> | \<alta / média / baixa> | \<alto: sem ela o combate não é testável> | \<spike de 2 dias com regras fixas> | \<inimigos com comportamento scriptado> |
|  |  |  |  |  |

**Dívida técnica aceita:** *atalhos tomados de propósito no protótipo. Registrada, ela é uma decisão; esquecida, vira bug.*

| Atalho | Por que agora | Custo de pagar | Pagar quando |
| --- | --- | --- | --- |
| \<ex.: sem save, estado só em memória> | \<ex.: o playtest dura uma sessão> | \<ex.: 2 dias> | \<ex.: antes do vertical slice> |

## Testes e ferramentas

**Testes:** \<o que é obrigatório pela regra do guia (componentes de cálculo) e o que este jogo acrescenta, ex.: "regras de linha com teste para cada caso-limite do GDD">

**Roteiros manuais:** \<o que só se avalia jogando, com o resultado esperado em termos observáveis>

**Ferramentas de depuração:** \<cena de teste, atalhos de debug num nó próprio fora do gameplay, logs>

**Telemetria do playtest:** \<o que o jogo grava para medir as metas do GDD, e onde: arquivo local CSV/JSON por sessão>

**Build e distribuição:** \<plataformas, como gerar a build, onde publicar para testadores>

**Versionamento:** \<branches, LFS para binários, o que não vai para o repositório>

## Decisões (ADRs)

*Um registro curto por decisão difícil de reverter, surpreendente sem contexto ou com troca real. As outras decisões ficam no texto das seções.*

| Nº | Decisão | Status | Data |
| --- | --- | --- | --- |
| ADR-001 | \<título> | \<proposta / aceita / obsoleta / substituída por ADR-00X> | \<AAAA-MM-DD> |

### ADR-001: \<título>

**Contexto:** \<o problema e as forças em jogo>

**Decisão:** \<o que foi decidido, em uma ou duas frases>

**Alternativas:** \<o que mais foi considerado e por que não>

**Consequências:** \<todas, inclusive as negativas e as neutras: o que fica mais fácil, o que fica mais difícil, e o que passa a ser obrigatório>

## Pendências do GDD

*Perguntas de design que travam decisões técnicas. Elas também aparecem nas Perguntas em aberto do GDD.*

- [ ] \<ex.: o tempo da fazenda segue o tempo de jogo ou o tempo real?> — trava: \<ClockSystem, save>
- [ ] \<pergunta> — trava:

## Glossário técnico

*Os termos do jogo ficam no glossário do GDD; aqui, só os que existem no código. O nome de uma classe ou sistema segue o termo do GDD.*

| Termo | Significado | Termo do GDD equivalente |
| --- | --- | --- |
| \<ex.: `InitiativeQueue`> | \<fila que decide quem age a seguir> | \<ex.: ordem de turno> |

## Changelog

| Versão | Data | O que mudou |
| --- | --- | --- |
| v0.1 | \<AAAA-MM-DD> | Primeira versão, a partir do GDD v\<n> |

## Referências do modelo

*Em que este modelo se apoia. Apague ao usar o modelo num jogo.*

- [arc42](https://docs.arc42.org/home/): metas de qualidade com cenários, restrições, visão de blocos, cenários de execução, conceitos transversais, riscos e dívida técnica, glossário.
- [Design Docs at Google](https://www.industrialempathy.com/posts/design-docs-at-google/), Malte Ubl: objetivos e não-objetivos, alternativas consideradas, documento atualizado antes de lançar e revisitado depois.
- [Formato de ADR de Michael Nygard](https://faq.arc42.org/questions/C-9-3/): contexto, decisão, status (proposta, aceita, obsoleta, substituída) e todas as consequências.
- [The Anatomy of a Design Document](https://www.gamedeveloper.com/design/the-anatomy-of-a-design-document-part-1-documentation-guidelines-for-the-game-concept-and-proposal), Tim Ryan: análise técnica com riscos, impacto no cronograma e alternativas.
