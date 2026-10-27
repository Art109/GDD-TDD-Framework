# GDD-TDD Framework

Kit para criar, criticar e evoluir GDDs e TDDs de jogos, do protótipo à produção, em Godot ou Unity.

Tem três partes:

- **Modelos** de GDD e TDD, com seções que crescem por estágio (protótipo, vertical slice, produção).
- **Guias de arquitetura** genéricos para Godot e Unity, com os mesmos princípios adaptados a cada engine.
- **Skills** do Claude que geram, sabatinam, revisam e refinam esses documentos.

## Estrutura

```
docs/
├── modelo-gdd.md            ← base de todo GDD, independente de engine
├── modelo-tdd.md            ← como um jogo aplica o guia aos sistemas do GDD
├── arquitetura-godot.md     ← guia de fundação, Godot 4.x (ref. 4.7)
└── arquitetura-unity.md     ← guia de fundação, Unity 6.x (ref. 6.5)
skills/
└── <nome>/SKILL.md          ← 12 skills, uma pasta cada
```

Os documentos em `docs/` são cópias. As versões vivas, editáveis e com diagramas, estão no Claude:

| Documento | Versão viva |
| --- | --- |
| Modelo de GDD | https://claude.ai/artifact/JUy7Z1va5xa7u3mLbagmcp |
| Modelo de TDD | https://claude.ai/artifact/WeAqiznSvqvXFGYzH25cBH |
| Arquitetura Modular para Godot | https://claude.ai/artifact/LgdBwNCGaE5RTLXgrHBzJD |
| Arquitetura Modular para Unity | https://claude.ai/artifact/5zGYdcdfMFWVWF4CaAYZZQ |

## Fluxo

```
ideia → brainstorming-jogo → gerar-gdd → grilling-projeto → revisar-gdd → refinar-projeto
      → gerar-tdd → revisar-tdd → teste-do-construtor → plano-prototipo
```

| Skill | Quando usar |
| --- | --- |
| `brainstorming-jogo` | A ideia ainda está se formando: perguntas abertas, abordagens, documento de design aprovado |
| `grilling-projeto` | Já existe um plano: sabatina em rodadas, com recomendação em cada pergunta |
| `gerar-gdd` | Criar um GDD a partir do modelo |
| `gerar-tdd` | Criar o TDD a partir do GDD, do modelo e do guia da engine |
| `revisar-gdd` | Crítica do GDD: pilares, core loop, MDA, escopo, implementabilidade |
| `revisar-tdd` | Crítica do TDD: rastreabilidade, arquitetura, riscos, performance |
| `refinar-projeto` | Transformar a crítica em documento melhor, com changelog e decisões |
| `teste-do-construtor` | Um leitor sem contexto aponta onde teria de adivinhar |
| `plano-prototipo` | GDD/TDD aprovado vira marcos jogáveis e tarefas verificáveis |
| `observador-skills` | Registra correções e padrões e propõe melhorias nas skills |
| `encontrar-skills` | Procurar skills prontas na comunidade antes de escrever uma |
| `modo-conciso` | Respostas de chat curtas (nunca dentro dos documentos) |

## Instalar as skills

- **Claude Code:** copie as pastas de `skills/` para `~/.claude/skills/` (todos os projetos) ou para `.claude/skills/` do repositório do jogo.
- **claude.ai:** envie cada `SKILL.md` em Configurações → Skills.

## Decisões de arquitetura (os dois guias)

- Duas camadas: `components/` (portável, passa no teste do projeto vazio) e `game/`.
- Sem autoload, singleton ou `DontDestroyOnLoad`.
- Resource / ScriptableObject só guarda dado: campos, sem método nem sinal, sufixo `Data`.
- Save em JSON, com id estável e campo de versão.
- A `Main` cuida só do ciclo de vida. Para ela não virar superclasse: cada cena liga os próprios filhos; as ligações da `Main` ficam em `Wiring` (um domínio cada); os sistemas chegam por um `GameContext` injetado (só em `game/`); eventos com 3+ ouvintes passam por `GameEvents`.
- Sub-cena sobreposta (batalha, loja, minigame) pausa o mundo e devolve um resultado.

## Créditos

Algumas skills são adaptações de trabalhos abertos, com o crédito no topo de cada `SKILL.md`:

| Skill | Origem | Licença |
| --- | --- | --- |
| `grilling-projeto` | `grilling`, de Matt Pocock — [mattpocock/skills](https://github.com/mattpocock/skills) | MIT |
| `brainstorming-jogo`, `teste-do-construtor`, `plano-prototipo` | `brainstorming`, `writing-plans`, `verification-before-completion`, de Jesse Vincent — [obra/superpowers](https://github.com/obra/superpowers) | MIT |
| `encontrar-skills` | `find-skills` — [vercel-labs/skills](https://github.com/vercel-labs/skills) | MIT |
| `modo-conciso` | `caveman`, de Julius Brussee — [JuliusBrussee/caveman](https://github.com/JuliusBrussee/caveman) | Apache-2.0 |
| `observador-skills` | `task-observer`, de Eoghan Henn e colaboradores — [rebelytics/one-skill-to-rule-them-all](https://github.com/rebelytics/one-skill-to-rule-them-all) | CC-BY-4.0 (versão reduzida e modificada) |

As revisões de GDD e TDD se inspiram em [Claude-Code-Game-Studios](https://tessl.io/registry/skills/github/Donchitos/Claude-Code-Game-Studios/design-review), [AlterLab GameForge](https://openskillindex.com/skills/alterlab-ieu-alterlab-gameforge-game-design-review) e no [Architecture Review Checklist](https://archman.dev/docs/checklists-and-templates/architecture-review-checklist-qualities-risks-ops).
