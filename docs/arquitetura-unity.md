# Arquitetura de Componentes para Unity

Oct 27, 2026 · @Arthur

> **Template genérico, para copiar e adaptar a cada projeto.** Vale para 2D e 3D, em qualquer gênero, no Unity 6.x (referência atual: 6.5). Segue os mesmos princípios da [Arquitetura Modular para Godot](arquitetura-godot.md), aplicados ao jeito da Unity. O que exige versão mínima está marcado.

## Resumo

O projeto tem **duas camadas**: `Components/`, com peças portáveis que funcionam em qualquer jogo, e `Game/`, com todo o resto. Cada componente tem o próprio **Assembly Definition**, então o compilador impede que ele use código do jogo. Uma cena `Bootstrap` monta tudo e nunca é descarregada; os levels entram como cenas aditivas. A `Main` só cuida do ciclo de vida: as ligações ficam em componentes `Wiring`, e os sistemas chegam a quem precisa por um `GameContext` injetado.

Regras em uma linha cada:

1. Duas camadas: `Components/` (portável) e `Game/` (o jogo). `Core/` só existe quando dois componentes precisam de fato compartilhar um tipo.
2. Cada pasta de componente tem um `.asmdef` que só referencia `Core` e pacotes da Unity. O compilador barra o resto.
3. Componentes têm sufixo `Component` (`HealthComponent`); ScriptableObjects de dados, sufixo `Data`.
4. O que o componente exige do projeto fica na própria classe: ações de input como `InputActionReference`, camadas como `LayerMask`, ambos em `[SerializeField]` e listados no doc comment.
5. Comentários em inglês, em `///`. O `<summary>` no topo da classe substitui o README.
6. Eventos sobem, chamadas descem. Cada prefab ou cena liga os próprios filhos; o jogo liga os componentes.
7. Injeção de um jeito só: `[SerializeField]` com valor padrão; `Setup()` apenas para dependências criadas em runtime.
8. ScriptableObject só guarda dado: campos, sem método nem evento, e o nome termina em `Data`.
9. Nenhum singleton, nenhum `DontDestroyOnLoad`. Os sistemas do jogo chegam por um `GameContext` injetado, e só em `Game/`.
10. A `Main` cuida só do ciclo de vida: montar, trocar de level e abrir sub-cena. As ligações ficam em componentes `Wiring`, um por domínio.
11. Save em JSON, com id estável e campo de versão.

## Duas camadas

`Game/` conhece e liga os componentes. Componentes não conhecem o jogo nem uns aos outros. `Core/` é uma exceção rara, não uma camada.

| Camada | Contém | Assembly pode referenciar | Portável |
| --- | --- | --- | --- |
| `Components/` | Peças que funcionam em qualquer jogo, uma pasta e um `.asmdef` cada | Só `Core` (se existir) e pacotes da Unity | Sim |
| `Game/` | Todo o resto: `App/` (Main, Bootstrap), `Levels/`, `Entities/`, `UI/`, `Systems/`, `Data/` | Todos os componentes e `Core` | Não |
| `Core/` (opcional) | Tipos que dois componentes de pastas diferentes precisam dividir | Nada | Sim |

Tudo do projeto fica em `Assets/_Project/`, separado de pacotes e assets de terceiros. Na dúvida, o código vai para `Game/`. Ele só vira componente quando passar no teste do projeto vazio.

Configure cada `.asmdef` de componente com **Auto Referenced desligado**: assim nenhum código o usa sem declarar a referência. Um componente que só calcula (sorteio, dano, crescimento) pode marcar **No Engine References**, e o compilador garante que ele é C# puro.

## Cena Bootstrap e fluxo

A cena `Bootstrap` é a primeira do Build Profile e nunca é descarregada. Ela guarda a `Main`, os sistemas, o player, a câmera e a UI. Cada level é uma cena carregada em modo aditivo e descarregada ao sair. Como a `Bootstrap` nunca sai, `DontDestroyOnLoad` não é necessário. Fica em `Game/App/`.

```
Bootstrap (cena 0, nunca descarregada)
├── Main                         ← Main.cs: só ciclo de vida
├── Systems                      ← SaveSystem, FlagSystem, ClockSystem, AudioSystem, GameEvents
├── Wiring                       ← HudWiring, AudioWiring, SaveWiring…: um domínio cada
├── Entities                     ← o player e o que atravessa levels
├── Effects                      ← pools e efeitos que atravessam levels
├── Camera
└── UI (Canvas)                  ← HUD (sort 10), Pause (20), Transition (100)

Level_Forest (cena aditiva, a ativa enquanto carregada)
└── LevelRoot (Level.cs)         ← mapa, spawn points, inimigos e interagíveis do level

Overlay_Battle (cena aditiva, opcional)
└── OverlayRoot (Overlay.cs)     ← sub-cena sobreposta
```

Regras:

1. `LoadLevel()` na `Main` é o único ponto que troca de level. Saídas e portas disparam evento; `SceneManager.LoadScene` em modo `Single` não é usado.
2. Cada cena de level tem **uma única raiz**, com o componente `Level`. Ele é dono do que é do mapa, liga os próprios filhos e expõe pontos de spawn. Não é dono do player nem da câmera.
3. Os inimigos do level vivem na cena do level e são descarregados com ela. O player vive em `Bootstrap/Entities` e é posicionado por `level.Place(player, spawnId)`, que funciona em 2D e 3D. Inventário, flags e progresso moram em `Systems`, nunca no player.
4. O level carregado vira a cena ativa. Assim, `Instantiate` sem pai cria o objeto no level, e ele some junto ao sair (drops, projéteis). O que precisa atravessar levels é criado com pai explícito em `Entities` ou `Effects`.
5. A `Main` só cuida do ciclo de vida. Ligar um sistema a outro, ou o player à HUD, é trabalho de um `Wiring`.
6. Telas: HUD, pausa e game over ficam em `Game/UI/` e vivem no Canvas da `Bootstrap`. A tela só dispara eventos (`ResumeRequested`) e recebe dados pelo que um `Wiring` liga. Nenhuma tela carrega cena nem conhece o player.
7. Pausa: o menu de pausa põe `Time.timeScale = 0`. UI, transição e animações de interface usam tempo não escalado (`Time.unscaledDeltaTime`); o relógio do jogo usa o escalado e para junto.
8. Uma câmera só, na `Bootstrap`. Levels não trazem câmera nem `AudioListener`.
9. Para testar um level sozinho no editor, um `LevelDebugBoot` (só em `#if UNITY_EDITOR`) carrega a `Bootstrap` se ela não estiver aberta e entrega o level já carregado à `Main`.

```csharp
// Game/App/Main.cs
using UnityEngine;
using UnityEngine.SceneManagement;

/// <summary>
/// Root of the game. Owns the lifecycle only: builds the context,
/// swaps levels and opens overlays. Wiring components do the connections.
/// </summary>
public sealed class Main : MonoBehaviour
{
    [SerializeField] private Player playerPrefab;
    [SerializeField] private Transform entities;
    [SerializeField] private Transition transition;
    [SerializeField] private Wiring[] wirings;
    [SerializeField] private SaveSystem save;
    [SerializeField] private FlagSystem flags;
    [SerializeField] private ClockSystem clock;
    [SerializeField] private GameEvents events;
    [SerializeField] private string firstLevel = "Level_Start";
    [SerializeField] private string firstSpawn = "start";

    private Player player;
    private GameContext context;
    private Level current;
    private bool loading;

    private async void Start()
    {
        context = new GameContext(save, flags, clock, events);
        player = Instantiate(playerPrefab, entities);
        foreach (var wiring in wirings) wiring.Setup(context, player);
        await LoadLevel(firstLevel, firstSpawn);
    }

    /// <summary>Swaps the current level. The only entry point for level changes.</summary>
    public async Awaitable LoadLevel(string sceneName, string spawnId)
    {
        if (loading) return;
        loading = true;
        try
        {
            await transition.Close();
            if (current != null)
            {
                current.ExitRequested -= OnExitRequested;
                await SceneManager.UnloadSceneAsync(current.gameObject.scene);
                current = null;
            }
            await SceneManager.LoadSceneAsync(sceneName, LoadSceneMode.Additive);
            var scene = SceneManager.GetSceneByName(sceneName);
            SceneManager.SetActiveScene(scene);
            current = FindLevel(scene);
            if (current == null)
            {
                Debug.LogError($"LoadLevel: {sceneName} has no root with a Level component");
                return;
            }
            current.ExitRequested += OnExitRequested;
            current.Setup(context);
            current.Place(player, spawnId);
            foreach (var wiring in wirings) wiring.OnLevelLoaded(current);
        }
        finally
        {
            await transition.Open();
            loading = false; // always reached, even when the level failed to load
        }
    }

    private async void OnExitRequested(string sceneName, string spawnId) =>
        await LoadLevel(sceneName, spawnId);

    private static Level FindLevel(Scene scene)
    {
        foreach (var root in scene.GetRootGameObjects())
            if (root.TryGetComponent(out Level level)) return level;
        return null;
    }
}
```

`try/finally` garante que a transição abre e que `loading` volta a `false` mesmo quando o level falha. Referências a cenas por nome são um ponto frágil: centralize os nomes numa `LevelData` (ScriptableObject com o nome da cena e os ids de spawn) em vez de espalhar strings.

## Main sem superclasse

Sem singletons, é fácil a `Main` virar o lugar onde tudo é ligado. Quatro regras evitam isso, e um sinal de alerta diz quando agir.

1. **Cada prefab ou cena liga os próprios filhos.** O level liga o que está dentro dele; o prefab do boss liga os componentes do boss. A `Main` só vê containers, sistemas e telas.
2. **As ligações da `Main` ficam em componentes `Wiring`**, filhos de `Bootstrap/Wiring`, um por domínio (`HudWiring`, `AudioWiring`, `SaveWiring`). Cada um recebe o contexto e o player em `Setup()` e o level novo em `OnLevelLoaded()`.
3. **Os sistemas chegam por um `GameContext` injetado.** É uma classe C# simples, criada uma vez pela `Main`, que só guarda referências aos sistemas. Não é ScriptableObject (guarda objetos vivos da cena) nem estática: só quem a recebe pode usá-la. O level recebe o contexto e repassa a cada filho só o que ele precisa. Componentes nunca recebem o contexto.
4. **Eventos transversais passam por `GameEvents`**, um componente em `Systems` com eventos C# tipados, que chega pelo contexto. Use só quando três ou mais sistemas reagem ao mesmo fato (ex.: "boss derrotado" muda música, save e conquistas). Para o resto, evento local ligado pelo pai.

**Sinal de alerta:** `Main.cs` passou de \~100 linhas, ou um `Wiring` cuida de mais de um domínio. Divida antes de continuar.

```csharp
// Game/App/GameContext.cs
/// <summary>
/// References to the game's systems. Created once by Main and handed down
/// through Setup(). Not a ScriptableObject (it holds live scene objects) and not static.
/// </summary>
public sealed class GameContext
{
    public SaveSystem Save { get; }
    public FlagSystem Flags { get; }
    public ClockSystem Clock { get; }
    public GameEvents Events { get; }

    public GameContext(SaveSystem save, FlagSystem flags, ClockSystem clock, GameEvents events)
    {
        Save = save;
        Flags = flags;
        Clock = clock;
        Events = events;
    }
}
```

```csharp
// Game/App/Wiring/Wiring.cs
/// <summary>Base for the components that connect Main's parts, one domain each.</summary>
public abstract class Wiring : MonoBehaviour
{
    public virtual void Setup(GameContext context, Player player) { }
    public virtual void OnLevelLoaded(Level level) { }
}

// Game/App/Wiring/HudWiring.cs
/// <summary>Connects the player's health to the HUD.</summary>
public sealed class HudWiring : Wiring
{
    [SerializeField] private Hud hud;

    public override void Setup(GameContext context, Player player) =>
        player.Health.HealthChanged += hud.ShowHealth; // Player exposes Health via [SerializeField]
}
```

```csharp
// Game/Systems/GameEvents.cs
/// <summary>
/// Events for facts that three or more systems react to.
/// Everything else uses a local event connected by the parent.
/// </summary>
public sealed class GameEvents : MonoBehaviour
{
    public event System.Action<string> BossDefeated;
    public event System.Action<int> DayStarted;

    public void RaiseBossDefeated(string bossId) => BossDefeated?.Invoke(bossId);
    public void RaiseDayStarted(int day) => DayStarted?.Invoke(day);
}
```

`GameContext` e `GameEvents` são código do jogo e ficam em `Game/`. Cada projeto define os próprios campos e eventos; os acima são exemplos. Quem assina um evento cancela a assinatura em `OnDestroy`, para não deixar referência a um objeto destruído.

## Sub-cena sobreposta

Uma sub-cena abre por cima do mundo, pausa o mundo e devolve um resultado ao fechar. Serve para batalha em cena separada, loja, minigame, diálogo longo ou cutscene. O level não é descarregado, então tudo volta exatamente como estava.

1. Quem pede a sub-cena (um inimigo, um NPC, um baú) dispara um evento com o nome da cena e os dados. Um `Wiring` liga esse evento a `Main.OpenOverlay()`.
2. A sub-cena é uma cena aditiva com uma única raiz que estende `Overlay`. Ela recebe o contexto e os dados em `Setup()` e dispara `Finished(result)` ao terminar.
3. Enquanto ela está aberta, a raiz do level fica inativa (`SetActive(false)`). Isso guarda o estado, mas chama `OnDisable` e para corrotinas: componentes do level retomam o que precisam em `OnEnable`. Se o mundo precisa continuar visível atrás (um diálogo), o `Level` expõe `SetPaused(bool)` e desliga só os próprios comportamentos.
4. Uma sub-cena por vez. Para empilhar (loja dentro de diálogo), a própria sub-cena abre a seguinte.

```csharp
// Game/App/Overlay.cs
/// <summary>Root of any scene opened on top of the world by Main.OpenOverlay().</summary>
public abstract class Overlay : MonoBehaviour
{
    public event System.Action<object> Finished;

    public virtual void Setup(GameContext context, object payload) { }
    protected void Finish(object result) => Finished?.Invoke(result);
}

// Game/App/Main.cs (continuação)
private bool overlayOpen;

/// <summary>Opens a scene on top of the world and pauses the world until it finishes.</summary>
public async Awaitable<object> OpenOverlay(string sceneName, object payload = null)
{
    if (overlayOpen || current == null) return null;
    overlayOpen = true;
    current.gameObject.SetActive(false);
    try
    {
        await SceneManager.LoadSceneAsync(sceneName, LoadSceneMode.Additive);
        var scene = SceneManager.GetSceneByName(sceneName);
        SceneManager.SetActiveScene(scene);
        var overlay = FindRoot<Overlay>(scene);
        if (overlay == null)
        {
            Debug.LogError($"OpenOverlay: {sceneName} has no root with an Overlay component");
            await SceneManager.UnloadSceneAsync(scene);
            return null;
        }
        var done = new AwaitableCompletionSource<object>();
        overlay.Finished += done.SetResult;
        overlay.Setup(context, payload);
        var result = await done.Awaitable;
        await SceneManager.UnloadSceneAsync(scene);
        return result;
    }
    finally
    {
        SceneManager.SetActiveScene(current.gameObject.scene);
        current.gameObject.SetActive(true);
        overlayOpen = false;
    }
}
```

`FindRoot<T>` é o mesmo `FindLevel` da seção anterior, genérico: percorre as raízes da cena e devolve a primeira com o componente.

## Componente portável

### O teste

Copie a pasta do componente (com o `.asmdef` e os `.meta`) para um projeto Unity vazio, coloque o componente num GameObject e rode. Se compila e funciona sem mais nada além dos pacotes que o `.asmdef` declara, o componente é independente e portátil. Se pede algo de fora, ainda é código do jogo.

### As regras

1. Tudo dentro da própria pasta: scripts, `.asmdef`, prefabs e assets de exemplo, ícone e testes. Nenhuma referência a código ou asset de fora (exceção: `Core/`).
2. Sufixo `Component` em todo MonoBehaviour que dá comportamento a uma entidade (`HurtboxComponent`, `LockComponent`). Classes base mantêm o nome simples (`State`, `StateMachine`); ScriptableObjects de dados terminam em `Data` (`HealthData`, `DropEntryData`).
3. Input pelo Input System: ações entram como `InputActionReference` em `[SerializeField]`. Quem usa escolhe a ação no Inspector. O `.asmdef` referencia `Unity.InputSystem`.
4. Camadas de física entram como `LayerMask` em `[SerializeField]`, nunca como número ou nome no código. O doc comment diz quais camadas o componente espera.
5. O `/// <summary>` no topo da classe, em inglês, explica o componente: o que faz, eventos e o que exige do projeto.
6. O ícone fica na própria pasta e é atribuído no script (guardado no `.meta`, viaja junto).
7. Testes ficam em `Tests/` dentro da pasta, com um `.asmdef` de testes próprio.
8. Cada componente vive num `namespace` próprio (`Components.Health`), para não colidir com código do jogo nem de pacotes.

```csharp
// Components/Movement/Runtime/PlayerInputComponent.cs
using UnityEngine;
using UnityEngine.InputSystem;

namespace Components.Movement
{
    /// <summary>
    /// Reads player input and raises the desired direction.
    /// Events: DirectionChanged. Requires from the project: a Vector2 move action.
    /// </summary>
    public sealed class PlayerInputComponent : MonoBehaviour
    {
        public event System.Action<Vector2> DirectionChanged;

        [SerializeField] private InputActionReference move;

        private void OnEnable() => move.action.Enable();
        private void OnDisable() => move.action.Disable();

        private void Update() => DirectionChanged?.Invoke(move.action.ReadValue<Vector2>());
    }
}
```

### Exigências visíveis no editor

O doc comment diz o que o componente exige; a Unity pode garantir parte disso e avisar o resto. Três ferramentas, da mais forte para a mais fraca:

1. **`[RequireComponent(typeof(T))]`**: a Unity adiciona o componente exigido e impede que ele seja removido.
2. **`Reset()`**: roda quando o componente é adicionado no editor; serve para deixar a configuração padrão certa (ex.: marcar o collider como trigger).
3. **`OnValidate()`**: roda a cada mudança no Inspector; avisa no Console, com o objeto clicável, quando algo está errado.

```csharp
// Components/Health/Runtime/HurtboxComponent.cs
using UnityEngine;

namespace Components.Health
{
    /// <summary>Trigger that receives hits. Raises HitReceived.</summary>
    [RequireComponent(typeof(Collider))] // Collider2D num jogo 2D
    public sealed class HurtboxComponent : MonoBehaviour
    {
        public event System.Action<int> HitReceived;

        private void Reset() => GetComponent<Collider>().isTrigger = true;

        private void OnValidate()
        {
            if (TryGetComponent(out Collider hurtbox) && !hurtbox.isTrigger)
                Debug.LogWarning($"{name}: HurtboxComponent needs a trigger collider.", this);
        }

        public void ReceiveHit(int damage) => HitReceived?.Invoke(damage);
    }
}
```

Campos de configuração levam `[Tooltip("...")]` com a mesma explicação do doc comment, para quem só olha o Inspector.

### Estrutura de uma pasta

```
Components/Health/
├── Runtime/
│   ├── Components.Health.asmdef            ← só Core e pacotes; Auto Referenced desligado
│   ├── HealthComponent.cs                  ← /// <summary> no topo, em inglês
│   ├── HitboxComponent.cs
│   ├── HurtboxComponent.cs
│   └── HealthData.cs                       ← tipo de ScriptableObject (só campos)
├── Editor/                                 ← só se houver inspector ou ferramenta de editor
│   ├── Components.Health.Editor.asmdef     ← plataforma: só Editor; referencia o Runtime
│   └── HealthComponentEditor.cs
├── Examples/
│   └── DefaultHealth.asset
├── Icons/
│   └── HealthIcon.png
└── Tests/
    ├── Editor/                             ← testes EditMode (cálculo)
    │   └── Components.Health.Tests.Editor.asmdef
    └── Runtime/                            ← testes PlayMode (física, cena), só se necessário
        └── Components.Health.Tests.asmdef
```

A separação `Runtime/`, `Editor/` e `Tests/` segue a [recomendação da Unity](https://docs.unity3d.com/6/Documentation/Manual/cus-asmdef.html). Código de editor fora de um assembly só de Editor quebra o build do jogo.

### Convenções e controle de versão

As da [Unity](https://unity.com/how-to/organizing-your-project), que evitam conflitos e erros de importação:

- Pastas e arquivos em PascalCase, sem espaços (as ferramentas de linha de comando da Unity têm problemas com espaços).
- Namespaces espelham as pastas: `Components.Health` em `Components/Health/`. Em projetos de estúdio, prefixe com o nome do estúdio (`Estudio.Components.Health`).
- Assets de terceiros fora de `Assets/_Project/`, nunca misturados com os seus.
- `.meta` sempre no controle de versão, e mover ou renomear assets só de dentro do editor, para o `.meta` ir junto.
- Em Project Settings → Editor: **Asset Serialization = Force Text** e **Version Control = Visible Meta Files**.
- Dividir trabalho em prefabs (inclusive aninhados) em vez de editar a cena: diffs menores e menos conflitos. Para os conflitos que sobrarem, configure o merge YAML da Unity (UnityYAMLMerge) no Git.
- Um modelo de script em `Assets/ScriptTemplates/` com o cabeçalho padrão do projeto.

## Comunicação e injeção

**Eventos sobem, chamadas descem.** Um objeto chama métodos dos próprios filhos e avisa o pai por evento. Componentes não se conhecem: quem liga um ao outro é o jogo, no prefab ou na cena que contém os dois.

```csharp
// Game/Entities/Boss/Boss.cs
public sealed class Boss : MonoBehaviour
{
    [SerializeField] private HealthComponent health;
    [SerializeField] private DropTableData drops;

    private System.Random random;

    public void Setup(System.Random random) => this.random = random; // the level owns the seed

    private void Awake() => health.Died += OnDied;
    private void OnDestroy() => health.Died -= OnDied;

    private void OnDied()
    {
        foreach (var result in Drops.Roll(drops, random))
            SpawnPickup(result);
    }
}
```

Por padrão, eventos C# (`event Action<T>`). `UnityEvent` só quando a ligação precisa ser feita no Inspector por quem monta a cena (um botão de UI, um gatilho de level). Os dois juntos no mesmo componente confundem: escolha um por evento.

### Injeção: um jeito só

| Caso | Como |
| --- | --- |
| Dependência conhecida na edição (um asset de dados, um filho, uma ação de input) | `[SerializeField] private` com valor padrão |
| Dependência que só existe em runtime (algo criado pelo jogo depois) | Método `Setup(...)`, chamado por quem cria o objeto logo após o `Instantiate` |
| Sistemas do jogo (save, flags, relógio, eventos), só em `Game/` | `GameContext` entregue por `Setup(context)`; quem recebe repassa só o campo que o filho usa |

Valor padrão de um asset: defina em **Default References** no Inspector do script, para que ele venha preenchido ao adicionar o componente. Campos `[SerializeField]` sempre `private`; o que precisa ser lido de fora vira propriedade só de leitura.

**O `Data` é só leitura.** Um ScriptableObject é o mesmo objeto para todas as instâncias: se um inimigo alterar o `HealthData`, todos mudam juntos. No editor, a mudança feita em Play Mode ainda fica gravada no asset. O que muda em runtime (vida atual, cooldown) fica num campo do próprio componente ou numa classe C# simples criada pelo dono.

### Proibido

- Singletons (`static Instance`) e `DontDestroyOnLoad`.
- `GameObject.Find`, `FindAnyObjectByType`, `FindObjectsByType` e busca por tag para achar dependências.
- `SendMessage` e `BroadcastMessage`.
- `transform.parent` ou `GetComponentInParent` para alcançar algo fora do próprio prefab.
- Passar o `GameContext` inteiro para um componente.

**Por que não usar event channels em ScriptableObject.** A Unity [recomenda esse padrão](https://unity.com/how-to/scriptableobjects-event-channels-game-code) como alternativa a singletons: um asset com um evento, que objetos de qualquer cena assinam. Este guia não o adota por dois motivos: ScriptableObject aqui só guarda dado, e a própria Unity reconhece que encadear canais deixa o fluxo difícil de rastrear. O `GameEvents` resolve o mesmo problema com eventos que chegam pelo contexto. Um projeto em que designers precisem ligar eventos entre cenas sem código pode adotar os canais e registrar o desvio no TDD.

### Portão e outras regras de jogo

Regra de negócio é do jogo. O `LockComponent` só guarda o estado e expõe um campo que descreve a condição; quem decide quando ela vale é o jogo.

```csharp
// Components/Level/Runtime/LockComponent.cs
namespace Components.Level
{
    /// <summary>
    /// Lock for a passage. Raises Unlocked and BlockedAttempt.
    /// The game decides when unlockFlag is met and calls Unlock().
    /// </summary>
    public sealed class LockComponent : MonoBehaviour
    {
        public event System.Action Unlocked;
        public event System.Action BlockedAttempt;

        [SerializeField] private bool locked = true;
        [SerializeField] private string unlockFlag = "";

        public bool IsLocked => locked;
        public string UnlockFlag => unlockFlag;

        public void Unlock()
        {
            if (!locked) return;
            locked = false;
            Unlocked?.Invoke();
        }

        public void TryPass()
        {
            if (locked) BlockedAttempt?.Invoke();
        }
    }
}
```

Um avaliador genérico de condições só vale a pena quando houver muitas regras diferentes. Até lá, o jogo checa a flag e chama `Unlock()`.

## Dados, estado e save

### ScriptableObject guarda dados

Um ScriptableObject de dados tem um script que define o **tipo** (só campos `[SerializeField]` e propriedades de leitura) e cada `.asset` é uma **instância** com valores, criada pelo menu `Create`. O script não tem método nem evento, e todo ScriptableObject de dados termina em `Data` (`HealthData`, `DropTableData`). O asset pode apontar para outros assets, prefabs e cenas, mas não tem comportamento: quem executa é um componente ou uma função estática. Objetos de runtime que não são asset nem são salvos (como o `DropResult`) não levam o sufixo.

```csharp
// Components/Drops/Runtime/DropEntryData.cs
namespace Components.Drops
{
    /// <summary>One possible drop. Fields only, no rules.</summary>
    [CreateAssetMenu(menuName = "Components/Drops/Drop Entry")]
    public sealed class DropEntryData : ScriptableObject
    {
        [SerializeField] private ScriptableObject payload;
        [SerializeField, Range(0f, 1f)] private float chance = 1f;
        [SerializeField] private int minAmount = 1;
        [SerializeField] private int maxAmount = 1;

        public ScriptableObject Payload => payload;
        public float Chance => chance;
        public int MinAmount => minAmount;
        public int MaxAmount => maxAmount;
    }
}

// Components/Drops/Runtime/DropTableData.cs
namespace Components.Drops
{
    /// <summary>A list of possible drops. Fields only, no rules.</summary>
    [CreateAssetMenu(menuName = "Components/Drops/Drop Table")]
    public sealed class DropTableData : ScriptableObject
    {
        [SerializeField] private DropEntryData[] entries = System.Array.Empty<DropEntryData>();

        public System.Collections.Generic.IReadOnlyList<DropEntryData> Entries => entries;
    }
}
```

Propriedades só de leitura não são comportamento: elas expõem o campo sem deixar ninguém alterá-lo. Os dados do jogo ficam em `Game/Data/` (`Boss01Drops.asset`). Um componente só traz assets de exemplo, em `Examples/`.

### Aleatoriedade vem de fora

Toda função que sorteia recebe um `System.Random` (ou `Unity.Mathematics.Random`). O jogo controla a semente, e os testes ficam determinísticos. O `UnityEngine.Random` global não é usado em componentes.

### Taxonomia de estado

| Categoria | Vive | Onde mora | Exemplos |
| --- | --- | --- | --- |
| Configuração | Sempre (arquivo) | ScriptableObject `Data`, só leitura | Itens, tabelas de drop, stats |
| Persistente | Entre sessões | Classes C# simples em `Systems`, salvas no save | Inventário, bosses derrotados |
| Sessão | Até sair ou um evento de reset | Classes C# simples em `Systems`, fora do save | Inimigos mortos desde o último descanso |
| Transitório | Enquanto o objeto existir | Campos do próprio componente | Velocidade, timer de ataque |

Cada jogo decide em que categoria cada dado cai.

### Regras de save

- JSON, num arquivo em `Application.persistentDataPath`. `JsonUtility` para estruturas simples; o pacote Newtonsoft Json (`com.unity.nuget.newtonsoft-json`) quando houver dicionários ou herança.
- Referências por id estável: um campo `id` (string) no `Data`, nunca o caminho do asset, o nome do arquivo nem o `GetInstanceID()`, que muda a cada execução.
- Campo de versão desde o primeiro save.
- Nunca salvar ScriptableObjects nem MonoBehaviours diretamente: o save guarda classes C# simples, marcadas `[System.Serializable]`.
- Configurações do jogador (volume, controles, idioma) num arquivo separado do save. `PlayerPrefs` só para preferências pequenas, nunca para progresso.

## Máquina de estados em classes C#

Decisão fechada: **uma classe C# por estado**, guardadas por uma `StateMachine<TOwner>`. A Unity não tem a árvore de nós do Godot, e um MonoBehaviour por estado traria custo de GameObject e ordem de `Update` imprevisível. A máquina e a base `State<TOwner>` são um componente portável (`Components/StateMachine`, com No Engine References). O dono chama `Tick()` no próprio `Update`, e os estados recebem o dono tipado, sem cast.

```csharp
// Components/StateMachine/Runtime/State.cs
namespace Components.StateMachine
{
    /// <summary>A single state. Owned by a StateMachine; requests changes through Machine.</summary>
    public abstract class State<TOwner>
    {
        protected TOwner Owner { get; private set; }
        protected StateMachine<TOwner> Machine { get; private set; }

        internal void Bind(TOwner owner, StateMachine<TOwner> machine)
        {
            Owner = owner;
            Machine = machine;
        }

        public virtual void Enter() { }
        public virtual void Exit() { }
        public virtual void Tick(float deltaTime) { }
    }
}

// Components/StateMachine/Runtime/StateMachine.cs
namespace Components.StateMachine
{
    /// <summary>Switches between states. Raises StateChanged.</summary>
    public sealed class StateMachine<TOwner>
    {
        public event System.Action<State<TOwner>, State<TOwner>> StateChanged;

        private readonly TOwner owner;

        public State<TOwner> Current { get; private set; }

        public StateMachine(TOwner owner) => this.owner = owner;

        public void TransitionTo(State<TOwner> next)
        {
            if (next == null || next == Current) return;
            var previous = Current;
            previous?.Exit();
            next.Bind(owner, this);
            Current = next;
            next.Enter();
            StateChanged?.Invoke(previous, next);
        }

        public void Tick(float deltaTime) => Current?.Tick(deltaTime);
    }
}
```

```csharp
// Game/Entities/Boss/Boss.cs (trecho)
private StateMachine<Boss> machine;
internal readonly BossIdle Idle = new();
internal readonly BossChase Chase = new();
internal readonly BossAttack Attack = new();

private void Start()
{
    machine = new StateMachine<Boss>(this);
    machine.TransitionTo(Idle);
}

private void Update() => machine.Tick(Time.deltaTime);


// Game/Entities/Boss/States/BossChase.cs
public sealed class BossChase : State<Boss>
{
    public override void Tick(float deltaTime)
    {
        if (Owner.DistanceToTarget() <= Owner.Data.AttackRange)
            Machine.TransitionTo(Owner.Attack);
    }
}
```

Os estados são criados uma vez pelo dono e reaproveitados, então não geram lixo a cada troca. Para ver o estado atual no Inspector, o dono expõe `machine.Current?.GetType().Name` num campo só de depuração.

## Testes e verificação

### Testes com Unity Test Framework

| Tipo de componente | Teste |
| --- | --- |
| Só calcula (sorteio de drop, cálculo de dano, regra de crescimento) | Obrigatório. Teste EditMode em `Tests/` dentro da pasta do componente |
| Depende de física, animação ou input | Opcional. O teste do projeto vazio e o playtest cobrem; PlayMode quando já quebrou uma vez |
| Código de `Game/` | Opcional. Vale para regras que já quebraram uma vez |

```csharp
// Components/Drops/Tests/Editor/DropsTests.cs
using NUnit.Framework;
using UnityEngine;

namespace Components.Drops.Tests
{
    public sealed class DropsTests
    {
        [Test]
        public void EntryWithFullChanceAlwaysDrops()
        {
            var entry = ScriptableObject.CreateInstance<DropEntryData>();
            var table = ScriptableObject.CreateInstance<DropTableData>();
            DropsTestUtil.SetChance(entry, 1f);
            DropsTestUtil.SetEntries(table, entry);

            var results = Drops.Roll(table, new System.Random(42));

            Assert.AreEqual(1, results.Count);
        }
    }
}
```

Com a semente fixa, o resultado é sempre o mesmo, e o teste nunca falha por acaso. Como os campos do `Data` são privados, o teste preenche os valores por um utilitário de teste (`SerializedObject` no editor) ou usa um asset de `Examples/`. Nunca abra os campos só para o teste.

### O que o compilador garante

Na Unity, a independência dos componentes é checada pelo compilador, não por grep. Um componente cujo `.asmdef` não referencia `Game` simplesmente não compila se usar uma classe do jogo.

| Pega | Não pega |
| --- | --- |
| Uso de qualquer classe de `Game/` ou de outro componente sem referência declarada | Um prefab ou asset de exemplo arrastando um asset de fora da pasta |
| Código de engine num componente marcado No Engine References | `GameObject.Find`, busca por tag e singletons |
| Ciclos entre assemblies | Nomes de cena ou de camada escritos como string |

O que o compilador não pega, duas buscas no pre-commit pegam, e o teste do projeto vazio cobre o resto:

```yaml
- repo: local
  hooks:
    - id: components-sem-busca-global
      name: "Components: sem Find, singleton ou DontDestroyOnLoad"
      language: pygrep
      entry: 'GameObject\.Find|Find(Any|First)?Objects?(By|Of)Type|static\s+\w+\s+Instance\b|DontDestroyOnLoad|SendMessage'
      files: ^Assets/_Project/Components/.*\.cs$
      exclude: /Tests/
    - id: components-asmdef-isolado
      name: "Components: asmdef não referencia Game"
      language: pygrep
      entry: '"Game"|"GUID:'
      files: ^Assets/_Project/Components/.*\.asmdef$
```

A segunda regra exige referências por nome nos `.asmdef` de componente (desligue **Use GUIDs** no Inspector do asmdef), para que a busca enxergue o que está sendo referenciado.

## Exemplo: componente de drops

O componente de drops responde a uma pergunta: dada uma tabela, o que cai? Ele não sabe quem morreu, onde, nem o que é um item. Por só calcular, o `.asmdef` dele pode ficar quase sem engine: só usa `ScriptableObject` dos dados.

```csharp
// Components/Drops/Runtime/Drops.cs
using System.Collections.Generic;

namespace Components.Drops
{
    /// <summary>
    /// Rolls what drops from a DropTableData.
    /// Requires from the project: nothing. The caller provides the System.Random.
    /// </summary>
    public static class Drops
    {
        public static List<DropResult> Roll(DropTableData table, System.Random random)
        {
            var results = new List<DropResult>();
            foreach (var entry in table.Entries)
            {
                if (random.NextDouble() > entry.Chance) continue;
                var amount = random.Next(entry.MinAmount, entry.MaxAmount + 1);
                results.Add(new DropResult(entry.Payload, amount));
            }
            return results;
        }
    }

    /// <summary>One roll result. Runtime only, never saved, so no Data suffix.</summary>
    public readonly struct DropResult
    {
        public readonly UnityEngine.ScriptableObject Payload;
        public readonly int Amount;

        public DropResult(UnityEngine.ScriptableObject payload, int amount)
        {
            Payload = payload;
            Amount = amount;
        }
    }
}
```

Fluxo de um drop, com quatro componentes:

1. O `HealthComponent` do boss dispara `Died`.
2. O script do boss (jogo) chama `Drops.Roll(drops, random)`.
3. Recebe os resultados.
4. Para cada um, decide como aparece: aqui, instancia um prefab com `PickupComponent` e o payload.
5. Quando o player coleta, o `PickupComponent` dispara `Collected(payload, collector)`.
6. O jogo confere o tipo e chama `inventory.Add(item, amount)` no `InventoryComponent`.

Cada componente passa no teste do projeto vazio sozinho. O único código específico são as poucas linhas do jogo nos passos 2, 4 e 6.

## Exemplos de componentes e o que fica no jogo

Exemplos de candidatos, tirados de um jogo de ação. Servem de referência de tamanho e de fronteira, não de lista obrigatória: cada gênero tem os seus (um jogo por turnos teria fila de iniciativa e cálculo de dano; um 3D, sensores e movimento 3D). Crie cada um só quando o jogo precisar dele.

| Pasta | Componentes | O que faz | Exige do projeto |
| --- | --- | --- | --- |
| `Health/` | `HealthComponent`, `HitboxComponent`, `HurtboxComponent` | Vida e dano | Camadas e colliders (Inspector) |
| `Drops/` | `Drops.Roll()`, `DropTableData`, `DropEntryData`, `DropResult` | Sorteia o que cai | Nada |
| `Inventory/` | `InventoryComponent`, `InventoryData`, `ItemData` | Guarda itens | Nada |
| `Interaction/` | `InteractableComponent`, `InteractorComponent`, `PickupComponent`, `CollectorComponent` | Interagir e coletar | Ação de interagir (`InputActionReference`), camadas |
| `Sensing/` | `TargetSensorComponent` | Detecta alvos numa área | Camadas (`LayerMask`) |
| `Movement/` | `MovementComponent`, `PlayerInputComponent` | Move o corpo | Ação de movimento (`InputActionReference`) |
| `Level/` | `MapExitComponent`, `SpawnPointComponent`, `LockComponent` | Saídas e travas | Camada do player |
| `StateMachine/` | `StateMachine<T>`, `State<T>` | Estados em classes C# | Nada |

**`Core/` hoje fica vazio.** Se `HitboxComponent` e `HurtboxComponent` passarem a trocar um tipo de golpe (ex.: `HitInfo`), ele fica em `Health/`, porque os dois estão na mesma pasta. `Core/` só aparece quando dois componentes de pastas diferentes precisarem do mesmo tipo.

**Fica em `Game/`, até outro jogo precisar:** a `Main`, os `Wiring`, o `GameContext`, as sub-cenas, o save, o relógio, as flags, as entidades e seus estados, os levels e a interface.

```
Assets/_Project/
├── Components/                ← uma pasta e um .asmdef por componente
├── Core/                      ← só quando necessário
└── Game/                      ← Game.asmdef
    ├── App/                   ← Main, GameContext, Overlay, Transition, Bootstrap.unity
    │   └── Wiring/            ← Wiring base e um script por domínio
    ├── Data/                  ← todos os assets de dados do jogo
    ├── Entities/              ← player, inimigos e seus estados
    ├── Levels/                ← cenas de level e o Level base
    ├── Overlays/              ← sub-cenas: batalha, loja, minigame
    ├── Systems/               ← save, relógio, flags, áudio, GameEvents
    └── UI/                    ← HUD, pausa, game over
```

## Trade-offs

| Decisão | Ganho | Custo |
| --- | --- | --- |
| Duas camadas com um `.asmdef` por componente | O compilador garante a independência; compilação incremental mais rápida | Mais arquivos de assembly para manter |
| `Bootstrap` persistente + levels aditivos | Player, câmera e UI únicos; sem `DontDestroyOnLoad` | Gestão manual de cena ativa e de descarregar |
| Exigências como `[SerializeField]` | Componente se explica no Inspector | Mais campos para configurar ao plugar |
| Jogo decide as regras (ex.: `LockComponent`) | Sem sistema genérico para manter | Com muitas regras, a camada do jogo cresce; aí vale generalizar |
| `Main` só com ciclo de vida, ligações em `Wiring` | A `Main` não vira superclasse; cada domínio num arquivo pequeno | Procurar "quem liga isto" exige abrir o `Wiring` certo |
| `GameContext` injetado | Dependências explícitas, sem singleton | Repasse manual de referências pelos levels |
| `GameEvents` para eventos transversais | Muitos sistemas reagem sem se conhecer | Quem lê um `Raise` não vê quem escuta; por isso só com 3+ ouvintes |
| Sub-cena sobreposta | Volta exata ao mundo, sem salvar e recarregar | `OnDisable`/`OnEnable` no level; mundo e sub-cena na memória ao mesmo tempo |
| Estados em classes C# | Sem GameObject por estado; ordem de execução clara | Estados não aparecem na hierarquia; depuração por campo de debug |
| Teste obrigatório só para cálculo | Cobre o que dá para testar barato | O resto depende de playtest |

**Regra para crescer:** algo nasce em `Game/` e só vira componente quando um segundo uso aparecer e ele passar no teste do projeto vazio.

## Como adaptar a um projeto

Copie este documento para o projeto e decida o que segue. O que não for decidido aqui vai para o TDD do jogo.

- [ ] 2D ou 3D, e o pipeline de render (URP ou HDRP).
- [ ] Quais sistemas existem em `Systems` e quais campos o `GameContext` tem.
- [ ] Quais eventos são transversais o bastante para `GameEvents` (3+ ouvintes).
- [ ] O jogo usa sub-cena sobreposta? Para quê, e se o mundo fica visível atrás.
- [ ] Em que categoria da taxonomia de estado cai cada dado do jogo.
- [ ] Se há save no protótipo, o formato do JSON com versão e a biblioteca (`JsonUtility` ou Newtonsoft).
- [ ] Lista inicial de componentes, criados só quando usados.
- [ ] Versão da Unity e dos pacotes (Input System, Test Framework) fixadas no README do projeto.

## Versões

As regras valem para qualquer Unity 6.x. Alguns detalhes dependem da versão; se o projeto usar uma mais antiga, troque só esses pontos.

| Recurso usado aqui | Versão mínima | Sem ele |
| --- | --- | --- |
| `Awaitable`, `AwaitableCompletionSource`, `await` em `LoadSceneAsync` | 2023.1 (Unity 6.0) | Corrotinas com `IEnumerator` e um callback no lugar do retorno |
| Build Profiles | Unity 6.0 | Build Settings: a `Bootstrap` como cena 0 |
| `FindAnyObjectByType` (citado nos proibidos) | 2021.3 | `FindObjectOfType`, igualmente proibido |
| Input System (`InputActionReference`) | Pacote 1.x | Input Manager antigo: nomes de eixo como `[SerializeField] string` |
| Unity Test Framework | Pacote 1.x | — |

Referência atual: Unity 6.5. Ao migrar de versão, revise esta tabela e o pre-commit.

## Referências

- [Organizing your project](https://unity.com/how-to/organizing-your-project): convenções de nomes, cenas aditivas, prefabs aninhados, `.meta`, namespaces por pasta.
- [Assembly definitions (Unity 6)](https://docs.unity3d.com/6/Documentation/Manual/cus-asmdef.html): assemblies `Runtime`, `Editor` e de testes.
- [ScriptableObject event channels](https://unity.com/how-to/scriptableobjects-event-channels-game-code) e o e-book [Create modular game architecture with ScriptableObjects](https://unity.com/resources/create-modular-game-architecture-scriptableobjects-unity-6): o padrão que este guia troca pelo `GameEvents`, e por quê.
- [Arquitetura Modular para Godot](arquitetura-godot.md): o guia irmão, com os mesmos princípios.
