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

*Itens que entram só em produção (CI/CD, rede, anti-cheat, localização, migração de save entre versões) ganham a própria subseção em Sistemas quando chegarem.*

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
| ADR-001 | \<título> | \<proposta / aceita / substituída por ADR-00X> | \<AAAA-MM-DD> |

### ADR-001: \<título>

**Contexto:** \<o problema e as forças em jogo>

**Decisão:** \<o que foi decidido, em uma ou duas frases>

**Alternativas:** \<o que mais foi considerado e por que não>

**Consequências:** \<o que fica mais fácil, o que fica mais difícil, e o que passa a ser obrigatório>

## Pendências do GDD

*Perguntas de design que travam decisões técnicas. Elas também aparecem nas Perguntas em aberto do GDD.*

- [ ] \<ex.: o tempo da fazenda segue o tempo de jogo ou o tempo real?> — trava: \<ClockSystem, save>
- [ ] \<pergunta> — trava:

## Changelog

| Versão | Data | O que mudou |
| --- | --- | --- |
| v0.1 | \<AAAA-MM-DD> | Primeira versão, a partir do GDD v\<n> |
