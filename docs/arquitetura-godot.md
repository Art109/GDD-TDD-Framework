# Arquitetura de Componentes para Godot

Oct 7, 2026 · @Arthur

> **Template genérico, para copiar e adaptar a cada projeto.** Vale para 2D e 3D, em qualquer gênero, no Godot 4.x (referência atual: 4.7). O que exige versão mínima está marcado. Guia irmão, com os mesmos princípios: [Arquitetura Modular para Unity](arquitetura-unity.md). As regras ainda estão em validação: o que não funcionar no primeiro projeto real muda aqui.

## Resumo

O projeto tem **duas camadas**: `components/`, com peças portáveis que funcionam em qualquer jogo, e `game/`, com todo o resto. Um componente é portável quando funciona plugado numa cena de um projeto vazio. O teste do projeto vazio confirma isso, e uma regra de grep no pre-commit pega os erros mais comuns. A cena `Main` monta tudo, mas só cuida do ciclo de vida: as ligações ficam em nós `Wiring`, e os sistemas chegam a quem precisa por um `GameContext` injetado.

Regras em uma linha cada:

1. Duas camadas: `components/` (portável) e `game/` (o jogo). `core/` só existe quando dois componentes precisam de fato compartilhar um tipo.
2. Componente só referencia a própria pasta. Nada de path, uid ou classe de fora.
3. Nós de componente têm sufixo `Component` (`HealthComponent`, `HitboxComponent`); Resources de dados, sufixo `Data`.
4. O que o componente exige do projeto fica na própria classe: ações do InputMap como `@export`; camadas de física configuradas no Inspector da cena e listadas no doc comment. O que faltar aparece como aviso no editor, pela função `_get_configuration_warnings()`.
5. Comentários em inglês, em `##`. O doc comment no topo do script substitui o README.
6. Sinais sobem, chamadas descem. Cada cena liga os próprios filhos; o jogo liga os componentes.
7. Injeção de um jeito só: `@export` com valor padrão; `setup()` apenas para dependências criadas em runtime.
8. Resource só guarda dado: o script declara campos, sem método nem sinal, e o nome termina em `Data`.
9. Nenhum autoload. Os sistemas do jogo chegam por um `GameContext` injetado, e só em `game/`.
10. A `Main` cuida só do ciclo de vida: montar, trocar de level e abrir sub-cena. As ligações ficam em nós `Wiring`, um por domínio.
11. Save em JSON, com id estável e campo de versão. Nunca `.tres` vindo do jogador.

**O que mudou nesta revisão:** a `Main` ficou neutra de gênero (2D ou 3D, com ou sem Y-sort); entraram a sub-cena sobreposta, as regras contra a `Main` virar superclasse (`Wiring`, `GameContext`, `GameEvents`), a seção de versões e o roteiro de adaptação; o `load_level()` foi corrigido; saiu o apêndice de um jogo específico. Na revisão anterior já tinham saído `modules/`, `module.cfg`, o checador em Python, o README por módulo, o `ConditionEvaluator` e o catálogo longo.

## Duas camadas

`game/` conhece e liga os componentes. Componentes não conhecem o jogo nem uns aos outros. `core/` é uma exceção rara, não uma camada.

*(Diagrama: duas camadas · game e components. Disponível na versão do documento no Claude.)*

| Camada | Contém | Pode referenciar | Portável |
| --- | --- | --- | --- |
| `components/` | Peças que funcionam em qualquer jogo | Só a própria pasta (e `core/`, se existir) | Sim |
| `game/` | Todo o resto: `app/` (Main), `levels/` (cenas de mapa instanciadas em `%World`), `entities/`, `ui/` (telas das CanvasLayers), `systems/` e `data/` | Tudo | Não |
| `core/` (opcional) | Tipos que dois componentes de pastas diferentes precisam dividir | Nada | Sim |

Na dúvida, o código vai para `game/`. Ele só vira componente quando passar no teste do projeto vazio.

## Cena principal e fluxo

A cena `Main` é a `run/main_scene` e nunca sai da árvore. Ela monta os containers, guarda os sistemas, troca de level e abre sub-cenas. Não liga sinais de jogo: isso é dos nós `Wiring` (seção seguinte). Fica em `game/app/`.

```
Main (Node2D ou Node3D)                ← run/main_scene, nunca destruída; só ciclo de vida
├── Systems (Node)                     ← save, relógio, flags, áudio, GameEvents: nós, não autoload
├── Wiring (Node)                      ← HudWiring, AudioWiring, SaveWiring…: um domínio cada
├── World %World                       ← level atual: só ele é trocado
├── Entities %Entities                 ← player, inimigos ativos, pickups, projéteis
├── Effects %Effects                   ← efeitos temporários
├── Overlay %Overlay                   ← sub-cena sobreposta: batalha, loja, minigame, cutscene
├── HUDLayer (CanvasLayer 10)
├── PauseLayer (CanvasLayer 20)
└── TransitionLayer (CanvasLayer 100) %Transition  ← fade (close/open)
```

`World`, `Entities`, `Effects` e `Overlay` são do mesmo tipo da `Main`: `Node2D` num jogo 2D, `Node3D` num 3D. Cada um tem seu ciclo de vida: o level é trocado, só o player atravessa a troca (o resto de `%Entities` e todo `%Effects` é limpo), os efeitos somem sozinhos e a sub-cena dura até emitir `finished`.

É a estrutura de ponto de entrada que a [documentação oficial](https://docs.godotengine.org/en/stable/tutorials/best_practices/scene_organization.html) sugere (`Main` → `World` + GUI, trocando os filhos de `World` ao mudar de level), com três acréscimos: `Entities` e `Effects` separados do mapa, `Overlay` para sub-cenas e `Wiring` para as ligações.

Regras:

1. `load_level()` na `Main` é o único ponto que troca de level. Saídas e portas emitem sinal; `change_scene_to_file()` não é usado.
2. O level, dentro de `%World`, é dono do que é do mapa (tilemap ou geometria, cenário, interagíveis), liga os próprios filhos e expõe pontos de spawn. Não é dono do player, por isso roda sozinho com F6.
3. Tudo que se move e age vive em `%Entities`. O level declara os inimigos (marcadores de spawn) e os cria quando a `Main` chama `level.spawn_entities(%Entities)`. O level posiciona o player com `level.place(player, spawn_id)`, que funciona em 2D e 3D. Inventário, flags e progresso moram em `Systems`, nunca no player.
4. Objetos criados em runtime vão para `%Entities` (drops, projéteis) ou `%Effects` (partículas), nunca para `current_scene`.
5. A `Main` só cuida do ciclo de vida. Ligar um sistema a outro, ou o player à HUD, é trabalho de um `Wiring`.
6. Telas: HUD, pausa e game over ficam em `game/ui/` e vivem nas CanvasLayers. A tela só emite sinais (`resume_requested`, `restart_requested`) e recebe dados pelo que um `Wiring` liga. Nenhuma tela troca cena nem conhece o player.
7. Y-sort, só em 2D com profundidade (top-down, isométrico): `Main`, `World`, `Entities` e o nó de props do level têm `y_sort_enabled`, para o Godot ordenar entidades e cenário juntos. Em 2D lateral e em 3D, não se usa.
8. Pausa: a `Main` fica em Always, e todo filho direto dela tem o `process_mode` definido, nunca Inherit: `World`, `Entities`, `Effects`, `Overlay` e `Systems` em Pausable (o relógio para junto); `Wiring`, `HUDLayer`, `PauseLayer` e `TransitionLayer` em Always.

```gdscript
# game/app/main.gd
extends Node2D  # Node3D num jogo 3D
## Root of the game. Owns the lifecycle only: builds the context,
## swaps levels and opens overlays. Wiring nodes do the connections.

@export var player_scene: PackedScene
@export var first_level: PackedScene
@export var first_spawn: StringName = &"start"

var _player: Node
var _context: GameContext
var _loading: bool = false


func _ready() -> void:
	_context = GameContext.new($Systems/Save, $Systems/Flags, $Systems/Clock, $Systems/GameEvents)
	_player = player_scene.instantiate()
	%Entities.add_child(_player)
	for wiring: Wiring in $Wiring.get_children():
		wiring.setup(_context, _player)
	load_level(first_level, first_spawn)


## Swaps the current level. The only entry point for level changes.
func load_level(scene: PackedScene, spawn_id: StringName) -> void:
	if _loading or scene == null:
		return
	_loading = true
	await %Transition.close()
	_clear_level()
	var level: Level = scene.instantiate() as Level
	if level == null:
		push_error("load_level: scene root does not extend Level")
	else:
		level.exit_requested.connect(load_level)
		level.setup(_context)
		%World.add_child(level)
		level.spawn_entities(%Entities)
		level.place(_player, spawn_id)
		for wiring: Wiring in $Wiring.get_children():
			wiring.on_level_loaded(level)
	await %Transition.open()
	_loading = false  # always reached, even when the level failed to load


func _clear_level() -> void:
	for container: Node in [%World, %Entities, %Effects]:
		for child in container.get_children():
			if child == _player:
				continue
			container.remove_child(child)  # leaves the tree now, not at the end of the frame
			child.queue_free()
```

`remove_child()` antes de `queue_free()` tira o level antigo da árvore na hora, para que ele não conviva um frame com o novo. Se o level falhar, a transição abre de novo e `_loading` volta a `false`, em vez de travar o jogo.

## Main sem superclasse

Sem autoload, é fácil a `Main` virar o lugar onde tudo é ligado. Quatro regras evitam isso, e um sinal de alerta diz quando agir.

1. **Cada cena liga os próprios filhos.** O level liga o que está dentro dele; a cena do boss liga os componentes do boss. A `Main` só vê containers, sistemas e telas.
2. **As ligações da `Main` ficam em nós `Wiring`**, filhos de `Main/Wiring`, um por domínio (`HudWiring`, `AudioWiring`, `SaveWiring`). Cada um recebe o contexto e o player em `setup()` e o level novo em `on_level_loaded()`.
3. **Os sistemas chegam por um `GameContext` injetado.** É uma classe simples (`RefCounted`), criada uma vez pela `Main`, que só guarda referências aos sistemas. Não é Resource (guarda nós vivos) nem global: só quem o recebe pode usá-lo. O level recebe o contexto e repassa a cada filho só o que ele precisa. Componentes nunca recebem o contexto.
4. **Eventos transversais passam por `GameEvents`**, um nó em `Systems` com sinais tipados, que chega pelo contexto. Use só quando três ou mais sistemas reagem ao mesmo fato (ex.: "boss derrotado" muda música, save e conquistas). Para o resto, sinal local ligado pelo pai.

**Sinal de alerta:** o script da `Main` passou de \~100 linhas, ou um `Wiring` cuida de mais de um domínio. Divida antes de continuar.

```gdscript
# game/app/game_context.gd
class_name GameContext
extends RefCounted
## References to the game's systems. Created once by Main and handed down
## through setup(). Not a Resource (it holds live nodes) and not global.

var save: SaveSystem
var flags: FlagSystem
var clock: ClockSystem
var events: GameEvents


func _init(p_save: SaveSystem, p_flags: FlagSystem, p_clock: ClockSystem, p_events: GameEvents) -> void:
	save = p_save
	flags = p_flags
	clock = p_clock
	events = p_events
```

```gdscript
# game/app/wiring/wiring.gd
class_name Wiring
extends Node
## Base for the nodes that connect Main's parts, one domain each.


func setup(_context: GameContext, _player: Node) -> void: pass
func on_level_loaded(_level: Level) -> void: pass


# game/app/wiring/hud_wiring.gd
extends Wiring
## Connects the player's health to the HUD.

@export var hud: Hud


func setup(_context: GameContext, player: Node) -> void:
	var health: HealthComponent = (player as Player).health  # Player exposes it with @export
	health.health_changed.connect(hud.show_health)
```

```gdscript
# game/systems/game_events.gd
class_name GameEvents
extends Node
## Signals for facts that three or more systems react to.
## Everything else uses a local signal connected by the parent.

signal boss_defeated(boss_id: StringName)
signal day_started(day: int)
```

`GameContext` e `GameEvents` são código do jogo e ficam em `game/`. Cada projeto define os próprios campos e sinais; os acima são exemplos.

## Sub-cena sobreposta

Uma sub-cena abre por cima do mundo, pausa o mundo e devolve um resultado ao fechar. Serve para batalha em cena separada, loja, minigame, diálogo longo ou cutscene. O mundo não é destruído, então tudo volta exatamente como estava.

1. Quem pede a sub-cena (um inimigo, um NPC, um baú) emite um sinal com a cena e os dados. Um `Wiring` liga esse sinal a `Main.open_overlay()`.
2. A raiz da sub-cena estende `Overlay`, recebe o contexto e os dados em `setup()` e emite `finished(result)` ao terminar.
3. Enquanto ela está aberta, `World`, `Entities` e `Effects` ficam em `PROCESS_MODE_DISABLED` e, se a sub-cena ocupa a tela, invisíveis. O menu de pausa continua funcionando por cima dela.
4. Uma sub-cena por vez. Para empilhar (loja dentro de diálogo), a própria sub-cena abre a seguinte como filha.

```gdscript
# game/app/overlay.gd
class_name Overlay
extends Node
## Root of any scene opened on top of the world by Main.open_overlay().

signal finished(result: Variant)


func setup(_context: GameContext, _payload: Dictionary) -> void: pass


# game/app/main.gd (continuação)
@export var hide_world_under_overlay: bool = true


## Opens a scene on top of the world and pauses the world until it finishes.
func open_overlay(scene: PackedScene, payload: Dictionary = {}) -> Variant:
	var overlay: Overlay = scene.instantiate() as Overlay
	if overlay == null or %Overlay.get_child_count() > 0:
		return null
	_set_world_active(false)
	overlay.setup(_context, payload)
	%Overlay.add_child(overlay)
	var result: Variant = await overlay.finished
	%Overlay.remove_child(overlay)
	overlay.queue_free()
	_set_world_active(true)
	return result


func _set_world_active(active: bool) -> void:
	var mode: ProcessMode = PROCESS_MODE_PAUSABLE if active else PROCESS_MODE_DISABLED
	for container: Node in [%World, %Entities, %Effects]:
		container.process_mode = mode
		if hide_world_under_overlay:
			container.visible = active
```

## Componente portável

### O teste

Copie a pasta do componente para um projeto Godot vazio, plugue o nó numa cena e rode. Se funciona sem mais nada, o componente é independente e portável. Se pede algo de fora, ainda é código do jogo.

### As regras

1. **Tudo dentro da própria pasta.** Scripts, cenas, `.tres` de exemplo, ícone e testes. Nenhuma referência a path, uid ou `class_name` fora dela (exceção: `core/`).
2. **Sufixo `Component` em todo nó que dá comportamento a uma entidade** (`HurtboxComponent`, `LockComponent`). Bases de nó mantêm o nome simples (`State`, `StateMachine`); Resources de dados terminam em `Data` (`HealthData`, `DropEntryData`).
3. **Ações do InputMap viram `@export` com valor padrão.** Quem usa ajusta no Inspector.
4. **Camadas de física são configuradas no Inspector da cena, nunca em script.** O doc comment do componente diz quais camadas ele espera.
5. **O doc comment `##` no topo, em inglês, explica o componente:** o que faz, sinais e o que exige do projeto. É o lugar que todo mundo lê, e o editor mostra no Inspector.
6. **`@icon` aponta para um arquivo da própria pasta.** Ícone apontando para fora quebra a cópia.
7. **Testes ficam em `tests/` dentro da pasta**, para viajarem junto.

```gdscript
@icon("res://components/movement/movement_icon.svg")
class_name PlayerInputComponent
extends Node
## Reads player input and emits the desired direction.
##
## Signals: [signal direction_changed].
## Requires from the project: the InputMap actions below.

signal direction_changed(direction: Vector2)

@export var action_left: StringName = &"move_left"
@export var action_right: StringName = &"move_right"
@export var action_up: StringName = &"move_up"
@export var action_down: StringName = &"move_down"
```

### Exigências visíveis no editor

O doc comment diz o que o componente exige; o editor pode mostrar isso sozinho. Com `@tool` e `_get_configuration_warnings()`, o nó ganha o ícone de aviso na árvore de cena quando falta algo, como o `Area2D` sem `CollisionShape2D`. É a recomendação oficial para dependências de cena: a cena se documenta pelo próprio código.

```gdscript
@tool
@icon("res://components/health/health_icon.svg")
class_name HurtboxComponent
extends Area2D
## Area that receives hits. Emits [signal hit_received].

signal hit_received(damage: int)


func _get_configuration_warnings() -> PackedStringArray:
	var warnings: PackedStringArray = []
	if collision_layer == 0:
		warnings.append("Set collision_layer to the owner's hurtbox layer.")
	if get_children().filter(func(c: Node) -> bool: return c is CollisionShape2D).is_empty():
		warnings.append("Add a CollisionShape2D child.")
	return warnings
```

Regras para `@tool`: o script roda no editor, então lógica de jogo em `_process` ou `_physics_process` começa com `if Engine.is_editor_hint(): return`. Quando um `@export` muda o que é válido, o setter chama `update_configuration_warnings()`. Use em componentes com exigências de cena (camadas, filhos, ações de input); componentes que só calculam não precisam.

### Estrutura de uma pasta

```
components/health/
├── health_component.gd     ← ## doc comment no topo, em inglês
├── hitbox_component.gd
├── hurtbox_component.gd
├── health_data.gd          ← tipo de Resource (só campos)
├── health_icon.svg         ← @icon aponta para cá
├── examples/
│   └── default_health.tres
└── tests/
    └── test_health_component.gd
```

### Convenções de nomes e pastas

As da [documentação oficial](https://docs.godotengine.org/en/stable/tutorials/best_practices/project_organization.html), que evitam erros só visíveis no build exportado:

- Pastas e arquivos em `snake_case` (o sistema de arquivos do `.pck` diferencia maiúsculas, o Windows não). Exceção: scripts C# levam o nome da classe, em PascalCase.
- Nós em PascalCase, como os nós nativos.
- Sinais com verbo no passado: `died`, `item_collected`, `exit_requested`.
- Código e assets de terceiros em `addons/` na raiz, mesmo que não sejam plugins.
- Assets ficam perto da cena que os usa: os sprites do boss em `game/entities/boss/`, não numa pasta global de sprites.
- Pastas que o Godot não deve importar (como `docs/`) recebem um arquivo vazio `.gdignore`.

## Comunicação e injeção

**Sinais sobem, chamadas descem.** Um nó chama métodos dos próprios filhos e avisa o pai por sinal. Componentes não se conhecem: quem liga um ao outro é o jogo, na cena que contém os dois.

```gdscript
# game/ — cena do boss
func _ready() -> void:
    $HealthComponent.died.connect(_on_died)

func _on_died() -> void:
    for result in Drops.roll(boss_drops, rng):
        _spawn_pickup(result)
```

### Injeção: um jeito só

| Caso | Como |
| --- | --- |
| Dependência conhecida na edição (um `.tres`, um nó filho, um nome de ação) | `@export` com valor padrão |
| Dependência que só existe em runtime (algo criado pelo jogo depois) | Método `setup(...)`, chamado por quem cria o nó antes do `add_child` |
| Sistemas do jogo (save, flags, relógio, eventos), só em `game/` | `GameContext` entregue por `setup(context)`; quem recebe repassa só o campo que o filho usa |

O valor padrão faz a cena rodar sozinha com F6. Exemplo: `@export var data: HealthData = preload("res://components/health/examples/default_health.tres")`.

**O `Data` é só leitura.** Um `.tres` carregado é o mesmo objeto para todas as instâncias: se um inimigo alterar o `HealthData`, todos mudam juntos. O que muda em runtime (vida atual, cooldown) fica numa `var` do próprio nó. Estado que precisa morar num Resource (como o `InventoryData` do player) é criado pelo dono com `.new()` ou `duplicate()`, nunca usado direto do `preload`.

### Proibido

- Autoloads, e singletons disfarçados (`static var` guardando um nó ou um sistema).
- `get_node()` para fora do próprio nó, caminhos `/root/...` e `get_tree().current_scene`.
- Achar dependência por grupo global (`get_first_node_in_group("player")`).
- Passar o `GameContext` inteiro para um componente.

**Por que mais estrito que a documentação oficial.** A [documentação do Godot](https://docs.godotengine.org/en/stable/tutorials/best_practices/autoloads_versus_regular_nodes.html) aceita autoload para sistemas que guardam só os próprios dados, precisam ser globais e funcionam isolados (ex.: quest, diálogo), e desaconselha os que mexem nos dados de outros. Este guia vai além e proíbe todos, pelos mesmos motivos que ela aponta: estado global, acesso de qualquer lugar e bug sem dono. O custo é repassar referências pelo `GameContext`. Um projeto que precisar de autoload registra o desvio no TDD, com o motivo.

### Portão e outras regras de jogo

Regra de negócio é do jogo. O `LockComponent` só guarda o estado e expõe um campo que descreve a condição; quem decide quando ela vale é o jogo.

```gdscript
class_name LockComponent
extends Node
## Lock for a passage. Emits [signal unlocked] and [signal blocked_attempt].
## The game decides when [member unlock_flag] is met and calls [method unlock].

signal unlocked
signal blocked_attempt

@export var locked: bool = true
@export var unlock_flag: StringName = &""

func unlock() -> void:
    if locked:
        locked = false
        unlocked.emit()
```

Um avaliador genérico de condições só vale a pena quando houver muitas regras diferentes. Até lá, o jogo checa a flag e chama `unlock()`.

## Dados, estado e save

### `.tres` guarda dados

Um Resource personalizado tem dois arquivos: o `.gd` define o **tipo** (só `class_name` e campos `@export`) e cada `.tres` é uma **instância** com valores, criada no Inspector. O `.gd` não tem nenhum método nem sinal, e todo Resource de dados termina em `Data` (`HealthData`, `DropTableData`). O `.tres` pode apontar para outros Resources, cenas e scripts, mas não tem comportamento: quem executa é um nó ou uma função estática. Objetos de runtime que não são Resource nem são salvos (como o `DropResult`, um `RefCounted`) não levam o sufixo.

```gdscript
# components/drops/drop_entry_data.gd
class_name DropEntryData
extends Resource
## One possible drop. Fields only, no rules.

@export var payload: Resource
@export_range(0.0, 1.0) var chance: float = 1.0
@export var min_amount: int = 1
@export var max_amount: int = 1

# components/drops/drop_table_data.gd
class_name DropTableData
extends Resource
## A list of possible drops. Fields only, no rules.

@export var entries: Array[DropEntryData] = []
```

Os dados do jogo ficam em `game/data/` (`boss_01_drops.tres`, `bean.tres`). Um componente só traz `.tres` de exemplo, em `examples/`.

### Aleatoriedade vem de fora

Toda função que sorteia recebe um `RandomNumberGenerator`. O jogo controla a semente, e os testes ficam determinísticos.

### Taxonomia de estado

| Categoria | Vive | Onde mora | Exemplos |
| --- | --- | --- | --- |
| Configuração | Sempre (arquivo) | `.tres`, só leitura | Itens, tabelas de drop, stats |
| Persistente | Entre sessões | Dados do jogo, salvos no save | Inventário, bosses derrotados |
| Sessão | Até sair ou um evento de reset | Dados do jogo, fora do save | Inimigos mortos desde o último descanso |
| Transitório | Enquanto o nó existir | No próprio nó | Velocidade, timer de ataque |

Cada jogo decide em que categoria cada dado cai.

### Regras de save

- Referências por **id estável**, nunca por caminho de arquivo.
- **Campo de versão** desde o primeiro save.
- **Nunca carregar `.tres` vindo do jogador:** um `.tres` pode conter script e executar código. Use JSON ou `ConfigFile`.
- Configurações do jogador (volume, controles, idioma) num arquivo separado do save.

## Máquina de estados por nós

Decisão fechada: **um nó por estado**, filhos de uma `StateMachine`. A máquina e a base `State` são componentes portáveis. Cada dono (player, boss) tem uma base tipada no jogo, para os estados acessarem o dono sem cast espalhado.

```
Boss (CharacterBody2D)
└── StateMachine          ← components/state_machine
    ├── Idle              ← game/entities/boss/states, extends BossState
    ├── Chase
    └── Attack
```

```gdscript
class_name State
extends Node
## A single state. Direct child of a [StateMachine].
## Request a change by emitting [signal transition_requested] with the next state's name.

signal transition_requested(to: StringName)

func enter(_previous: StringName) -> void: pass
func exit() -> void: pass
func update(_delta: float) -> void: pass
func physics_update(_delta: float) -> void: pass
```

```gdscript
class_name StateMachine
extends Node
## Switches between its child states. Emits [signal state_changed].

signal state_changed(from: StringName, to: StringName)

@export var initial_state: State
var current: State

func _ready() -> void:
    for child in get_children():
        if child is State:
            child.transition_requested.connect(transition_to)
    # Children run _ready before the owner: wait so states can use the owner's @onready vars.
    if owner and not owner.is_node_ready():
        await owner.ready
    current = initial_state
    if current:
        current.enter(&"")

func transition_to(to: StringName) -> void:
    var next: State = get_node_or_null(NodePath(to)) as State
    if next == null or next == current:
        return
    var from: StringName = current.name if current else &""
    if current:
        current.exit()
    current = next
    current.enter(from)
    state_changed.emit(from, to)

func _process(delta: float) -> void:
    if current:
        current.update(delta)

func _physics_process(delta: float) -> void:
    if current:
        current.physics_update(delta)
```

```gdscript
# game/entities/boss/states/boss_state.gd
class_name BossState
extends State
## Base for boss states: typed access to the boss.

var boss: Boss:
    get: return owner as Boss

# game/entities/boss/states/chase.gd
extends BossState

func physics_update(_delta: float) -> void:
    if boss.distance_to_target() <= boss.data.attack_range:
        transition_requested.emit(&"Attack")
```

O `owner` é a raiz da cena onde o estado foi salvo (`boss.tscn`), por isso a base tipada funciona sem `get_parent()`.

## Testes com GUT

| Tipo de componente | Teste |
| --- | --- |
| **Só calcula** (sorteio de drop, cálculo de dano, regra de crescimento) | **Obrigatório.** Teste GUT em `tests/` dentro da pasta do componente |
| Depende de física, animação ou input | Opcional. O teste do projeto vazio e o playtest cobrem |
| Código de `game/` | Opcional. Vale para regras que já quebraram uma vez |

```gdscript
# components/drops/tests/test_drops.gd
extends GutTest

func test_entry_with_full_chance_always_drops() -> void:
    var table: DropTableData = DropTableData.new()
    var entry: DropEntryData = DropEntryData.new()
    entry.chance = 1.0
    table.entries = [entry]
    var rng: RandomNumberGenerator = RandomNumberGenerator.new()
    rng.seed = 42
    assert_eq(Drops.roll(table, rng).size(), 1)
```

Com a semente fixa, o resultado é sempre o mesmo, e o teste nunca falha por acaso.

## Verificação no pre-commit

A independência dos componentes é checada por duas buscas por regex (`pygrep`), sem script próprio. Funcionam igual em Windows, Linux e macOS.

```yaml
- repo: local
  hooks:
    - id: components-scripts-isolados
      name: "components: scripts só referenciam a própria pasta"
      language: pygrep
      entry: 'res://(?!components/|core/)|uid://|\.\./'
      files: ^components/.*\.gd$
      exclude: /tests/
    - id: components-cenas-isoladas
      name: "components: cenas e .tres só apontam para components/ ou core/"
      language: pygrep
      entry: 'path="res://(?!components/|core/)'
      files: ^components/.*\.(tscn|tres)$
```

| Pega | Não pega |
| --- | --- |
| `preload("res://game/...")`, `uid://` em script, `../` | Um componente usando o `class_name` de outro componente |
| Cena de componente usando textura ou script de `game/` ou `assets/` | Um componente apontando para a pasta de outro componente |
|  | Grupos, autoloads e caminhos montados em runtime |

O que o grep não pega, o teste do projeto vazio pega. Faça esse teste ao criar um componente e quando ele mudar de forma importante. A pasta `tests/` fica fora da checagem porque os testes dependem do GUT (`addons/gut`).

## Exemplo: componente de drops

O componente de drops responde a uma pergunta: dada uma tabela, o que cai? Ele não sabe quem morreu, onde, nem o que é um item.

```gdscript
# components/drops/drops.gd
class_name Drops
## Rolls what drops from a [DropTableData].
## Requires from the project: nothing. The caller provides the [RandomNumberGenerator].

static func roll(table: DropTableData, rng: RandomNumberGenerator) -> Array[DropResult]:
    var results: Array[DropResult] = []
    for entry in table.entries:
        if rng.randf() <= entry.chance:
            var result: DropResult = DropResult.new()
            result.payload = entry.payload
            result.amount = rng.randi_range(entry.min_amount, entry.max_amount)
            results.append(result)
    return results

# components/drops/drop_result.gd
class_name DropResult
extends RefCounted
## One roll result. Runtime only, never saved, so no Data suffix.

var payload: Resource
var amount: int = 1
```

*(Diagrama: fluxo de um drop · 4 componentes, 6 passos. Disponível na versão do documento no Claude.)*

1. O `HealthComponent` do boss emite `died`.
2. A cena do boss (jogo) chama `Drops.roll(boss_drops, rng)`.
3. Recebe os resultados.
4. Para cada um, decide como aparece: aqui, cria um `PickupComponent` com o payload.
5. Quando o player coleta, o `PickupComponent` emite `collected(payload, collector)`.
6. O jogo confere o tipo e chama `inventory.add(item, amount)` no `InventoryComponent`.

Cada componente passa no teste do projeto vazio sozinho. O único código específico são as poucas linhas do jogo nos passos 2, 4 e 6.

## Componentes base e o que fica no jogo

Exemplos de candidatos, tirados de um jogo de ação 2D. Servem de referência de tamanho e de fronteira, não de lista obrigatória: cada gênero tem os seus (um jogo por turnos teria fila de iniciativa e cálculo de dano; um 3D, sensores e movimento 3D). Crie cada um só quando o jogo precisar dele.

| Pasta | Componentes | O que faz | Exige do projeto |
| --- | --- | --- | --- |
| `health/` | `HealthComponent`, `HitboxComponent`, `HurtboxComponent` | Vida e dano | Camadas de física (Inspector) |
| `drops/` | `Drops.roll()`, `DropTableData`, `DropEntryData`, `DropResult` | Sorteia o que cai | Nada |
| `inventory/` | `InventoryComponent`, `InventoryData`, `ItemData` | Guarda itens | Nada |
| `interaction/` | `InteractableComponent`, `InteractorComponent`, `PickupComponent`, `CollectorComponent` | Interagir e coletar | Ação de interagir (`@export`), camadas (Inspector) |
| `sensing/` | `TargetSensorComponent` | Detecta alvos numa área | Camadas (Inspector) |
| `movement/` | `MovementComponent`, `PlayerInputComponent` | Move o corpo | Ações do InputMap (`@export`) |
| `level/` | `MapExitComponent`, `SpawnPointComponent`, `LockComponent` | Saídas e travas | Camada do player (Inspector) |
| `state_machine/` | `StateMachine`, `State` | Estados por nós | Nada |

**`core/` hoje fica vazio.** Se `HitboxComponent` e `HurtboxComponent` passarem a trocar um tipo de golpe (ex.: `HitData`), ele fica em `health/`, porque os dois estão na mesma pasta. `core/` só aparece quando dois componentes de pastas diferentes precisarem do mesmo tipo.

**Fica em `game/`, até outro jogo precisar:** a `Main`, os `Wiring`, o `GameContext`, as sub-cenas, o save, o relógio, as flags, as entidades e seus estados, os levels e a interface.

```
game/
├── app/                       ← Main, GameContext, Overlay, troca de level, transição
│   └── wiring/                ← Wiring base e um script por domínio
├── data/                      ← todos os .tres do jogo
├── entities/                  ← player, inimigos e seus estados
├── levels/                    ← cenas de level (Level base em app/ ou aqui)
├── overlays/                  ← sub-cenas: batalha, loja, minigame
├── systems/                   ← save, relógio, flags, áudio, GameEvents
└── ui/                        ← HUD, pausa, game over
```

## Trade-offs

| Decisão | Ganho | Custo |
| --- | --- | --- |
| Duas camadas | Fácil de explicar e de seguir | `game/` concentra muito código; organize por pasta |
| Grep em vez de checador | Zero ferramenta própria | Não pega uso de `class_name` de outro componente; o teste do projeto vazio cobre |
| Exigências como `@export` | Componente se explica no Inspector | Mais campos para configurar ao plugar |
| Jogo decide as regras (ex.: `LockComponent`) | Sem sistema genérico para manter | Com muitas regras, a camada do jogo cresce; aí vale generalizar |
| `Main` só com ciclo de vida, ligações em `Wiring` | A `Main` não vira superclasse; cada domínio num arquivo pequeno | Mais arquivos; procurar "quem liga isto" exige abrir o `Wiring` certo |
| `GameContext` injetado | Dependências explícitas, sem autoload | Repasse manual de referências pelos levels |
| `GameEvents` para eventos transversais | Muitos sistemas reagem sem se conhecer | Quem lê um `emit()` não vê quem escuta; por isso só com 3+ ouvintes |
| Sub-cena sobreposta | Volta exata ao mundo, sem salvar e recarregar | Mundo e sub-cena na memória ao mesmo tempo |
| State machine por nós | Estados visíveis na cena, fáceis de depurar | Mais nós e arquivos que um `match` |
| Teste obrigatório só para cálculo | Cobre o que dá para testar barato | O resto depende de playtest |

**Regra para crescer:** algo nasce em `game/` e só vira componente quando um segundo uso aparecer e ele passar no teste do projeto vazio.

## Como adaptar a um projeto

Copie este documento para o projeto e decida o que segue. O que não for decidido aqui vai para o TDD do jogo.

- [ ] 2D ou 3D: tipo da `Main` e dos containers; Y-sort só em 2D com profundidade.
- [ ] Quais sistemas existem em `Systems` e quais campos o `GameContext` tem.
- [ ] Quais eventos são transversais o bastante para `GameEvents` (3+ ouvintes).
- [ ] O jogo usa sub-cena sobreposta? Para quê (batalha, loja, diálogo)?
- [ ] Em que categoria da taxonomia de estado cai cada dado do jogo.
- [ ] Se há save no protótipo, e o formato do JSON com versão.
- [ ] Lista inicial de componentes, criados só quando usados.
- [ ] Versão do Godot e do GUT fixadas no README do projeto.

## Versões

As regras valem para qualquer Godot 4.x. Alguns detalhes dependem da versão; se o projeto usar uma mais antiga, troque só esses pontos.

| Recurso usado aqui | Versão mínima | Sem ele |
| --- | --- | --- |
| Nomes únicos `%World`, arrays tipados `Array[T]` | 4.0 | — |
| Tipo na variável do `for` (`for w: Wiring in ...`) | 4.2 | Tire o tipo do `for` |
| `uid://` (checado no pre-commit) | 4.4 | A regra de `uid://` não tem efeito, não atrapalha |
| GUT | 9.x para Godot 4 | — |

Referência atual: Godot 4.7. Ao migrar de versão, revise esta tabela e o pre-commit.

## Referências

- [Scene organization](https://docs.godotengine.org/en/stable/tutorials/best_practices/scene_organization.html): cenas sem dependências, injeção pelo pai, sinais no passado, `_get_configuration_warnings()`, estrutura `Main` → `World` + GUI.
- [Autoloads versus regular nodes](https://docs.godotengine.org/en/stable/tutorials/best_practices/autoloads_versus_regular_nodes.html): os problemas de estado e acesso globais, e quando o autoload é aceitável.
- [Project organization](https://docs.godotengine.org/en/stable/tutorials/best_practices/project_organization.html): `snake_case`, PascalCase nos nós, `addons/`, `.gdignore`, maiúsculas no build exportado.
- [Arquitetura Modular para Unity](arquitetura-unity.md): o guia irmão, com os mesmos princípios.
