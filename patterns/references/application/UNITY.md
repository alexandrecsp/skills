# Unity Game Engineering

## Overview

Unity projects fail in ways general software projects don't: garbage collection spikes that stutter frame time, scene/prefab merge conflicts that corrupt data silently, tight coupling between MonoBehaviours that makes systems untestable, and asset pipelines that balloon build size. This skill applies game-industry-standard patterns — data-driven design with ScriptableObjects, decoupled communication, allocation-conscious hot paths, and disciplined version control for binary/YAML assets — to keep a Unity codebase maintainable as it scales past prototype size.

**Frame budget is the constraint everything else serves.** A 60 FPS game has ~16.6ms per frame; 90 FPS VR has ~11ms. Every gameplay system, every allocation, every `GetComponent` call competes for that budget. Code that would be fine in a web backend can tank a game's frame rate.

**Data should drive behavior, not hardcode it.** Designers and gameplay engineers need to tune values, add content, and iterate without recompiling. ScriptableObjects, not hardcoded constants, are the default for anything a designer might want to touch.

## When to Use

- Writing new MonoBehaviours, systems, or gameplay logic in Unity
- Reviewing Unity C# for performance or architecture issues
- Designing how gameplay systems communicate (events, singletons, dependency injection)
- Working with prefabs, ScriptableObjects, or scene composition
- Profiling frame time, GC allocations, or draw calls
- Setting up project folder structure, `.gitignore`, or Git LFS for a new Unity project
- Configuring build pipelines or addressable/asset bundle strategy
- Debugging stutters, memory growth, or scene load issues

## When Not to Use

- C# with no Unity dependency (a pure rules library); the frame-budget rules don't bind it
- Editor-only tooling and build scripts that never run in play mode

## Core Architecture Patterns

### Favor composition over inheritance-heavy MonoBehaviours

Unity's component model already gives you composition. Don't build deep `PlayerBase → Player → PlayerWithJetpack` inheritance chains — compose small, single-purpose components (`Health`, `Movement`, `Jetpack`) and wire them together on the prefab.

```csharp
// Avoid: monolithic MonoBehaviour doing everything
public class Player : MonoBehaviour
{
    void Update() { HandleInput(); HandleMovement(); HandleHealth(); HandleInventory(); }
}

// Prefer: small components, each with one job
public class PlayerMovement : MonoBehaviour { /* only movement */ }
public class Health : MonoBehaviour { /* only health */ }
public class Inventory : MonoBehaviour { /* only inventory */ }
```

### Decouple systems with events, not direct references

Direct references between gameplay systems (`GameManager.Instance.player.health.TakeDamage()`) create a tangled dependency graph that's hard to test and hard to change. Use ScriptableObject-based event channels or a lightweight event bus instead.

```csharp
// ScriptableObject event channel — decouples publisher from subscribers,
// works across scenes, and is inspectable/wireable in the Editor.
[CreateAssetMenu(menuName = "Events/Game Event")]
public class GameEvent : ScriptableObject
{
    private readonly List<GameEventListener> listeners = new();
    public void Raise() { for (int i = listeners.Count - 1; i >= 0; i--) listeners[i].OnEventRaised(); }
    public void Register(GameEventListener l) => listeners.Add(l);
    public void Unregister(GameEventListener l) => listeners.Remove(l);
}
```

Reserve singletons (`GameManager.Instance`) for true cross-cutting concerns (audio, save system) — not as a substitute for passing references. Every singleton is a hidden dependency; prefer constructor/serialized-field injection where practical.

### ScriptableObjects as data containers, not just config

Use ScriptableObjects for shared, designer-tunable data (weapon stats, enemy configs, dialogue) so content changes don't require code changes or recompiles. Keep them immutable at runtime unless deliberately used as a shared-state pattern (e.g., a runtime `FloatVariable` asset) — and if you do that, document it, since it's a common source of "why did this reset between play sessions" bugs.

### Interfaces for testable gameplay logic

Pull core logic out of MonoBehaviours into plain C# classes behind interfaces where feasible. `MonoBehaviour` cannot be `new`'d in a unit test and drags in Unity's lifecycle — a plain class implementing `IDamageable` or `IMovable` can be tested in isolation with the Unity Test Framework's EditMode tests, no scene required.

## Performance and the Frame Budget

### Eliminate per-frame allocations

The #1 cause of Unity stutter is garbage collection triggered by per-frame allocations in `Update`/`FixedUpdate`. Common offenders:

```csharp
// Avoid — allocates a new array every frame
void Update() {
    var enemies = GameObject.FindGameObjectsWithTag("Enemy"); // allocation + slow
    foreach (var e in GetComponents<Collider>()) { }          // allocation
    string status = "HP: " + health;                          // string concat allocation
}

// Prefer — cache references, reuse buffers, avoid string ops in hot paths
private Collider[] cachedColliders;
void Awake() { cachedColliders = GetComponents<Collider>(); }
void Update() { /* no allocation */ }
```

- Cache `GetComponent`/`Find*` results in `Awake`/`Start`, never call them in `Update`.
- Use object pooling for frequently spawned/destroyed objects (bullets, particles, enemies) instead of `Instantiate`/`Destroy`.
- Prefer `NonAlloc` physics APIs (`Physics.RaycastNonAlloc`, `OverlapSphereNonAlloc`) over their allocating counterparts.
- Avoid LINQ and `foreach` over `List<T>` via `IEnumerable` in hot paths — both allocate on some Unity/.NET versions. Use indexed `for` loops.
- Use the Profiler's memory view to confirm — don't guess. `Deep Profile` for allocation call sites, `GC.Alloc` column in the frame timeline for the actual cost.

### Update method discipline

Every `MonoBehaviour.Update` has per-call overhead. With hundreds of active objects, thousands of empty `Update()` calls add up.

- Disable/remove `Update()` on objects that don't need per-frame logic; drive them from events or coroutines instead.
- Batch logic through a single manager's `Update` for large object counts (a "tick manager" pattern) rather than N independent `Update` calls, when profiling shows it matters.
- Use `FixedUpdate` only for physics-affecting logic; use `Update` for everything else; use `LateUpdate` for camera-follow and anything that must run after all `Update`s.

### Draw calls and batching

- Use SRP Batcher (URP/HDRP) or GPU instancing for objects sharing a material.
- Combine static meshes where possible (`Static` batching flag) for non-moving geometry.
- Keep texture atlases for UI and sprites to reduce material/draw-call count.
- Profile with the Frame Debugger before optimizing — confirm draw calls are actually the bottleneck, not the assumption.

## Project Structure and Asset Organization

```
Assets/
  _Project/            # Team's own content, prefixed to sort above package folders
    Art/
    Audio/
    Prefabs/
    Scenes/
    Scripts/
      <Feature>/        # e.g. Combat, Inventory, Guns — one folder per gameplay feature
        Runtime/
          Interfaces/
          Components/
          Systems/
          Data/
        Editor/
          Components/
          Systems/
          Data/
        Tests/
          Components/
          Systems/
          Data/
        UI/
    ScriptableObjects/
  Plugins/              # Third-party assets, kept isolated from _Project
  StreamingAssets/
```

- Organize `Scripts/` feature-first, not layer-first: everything about one feature — runtime code, its editor tooling, and its tests — lives under one folder, instead of being scattered across separate top-level `Runtime/`, `Editor/`, `Tests/` trees. This makes a feature easy to find, review, or delete as a unit.
- Within each feature's `Runtime/`, `Editor/`, and `Tests/` folder, split by role: `Components/` for MonoBehaviours/data-holding pieces, `Systems/` for the logic that operates on them, `Data/` for ScriptableObjects and plain data classes specific to that feature.
- Keep third-party assets out of your own script folders — makes upgrades and license auditing simpler.
- Unity excludes any folder literally named `Editor` from player builds regardless of nesting depth, so `Scripts/<Feature>/Editor/` is still stripped correctly under this layout.
- If you use assembly definitions, feature-first layout means one asmdef per feature per layer (`Combat.Runtime.asmdef`, `Combat.Editor.asmdef`, `Combat.Tests.asmdef`) rather than one per top-level layer — a real tradeoff of this structure, since you lose a single project-wide Runtime/Editor/Tests assembly boundary in exchange for per-feature isolation.
- One scene per level/area plus an additive "core systems" scene, rather than one giant scene — enables parallel scene work and additive loading.

## Version Control for Unity

Unity projects mix text (YAML scenes/prefabs when configured) and binary assets, and both cause pain if unconfigured.

- **Force YAML/text serialization:** Project Settings → Editor → Asset Serialization → Force Text. Makes scenes/prefabs diffable and mergeable instead of opaque binary blobs.
- **Git LFS for binaries:** track `*.png`, `*.psd`, `*.fbx`, `*.wav`, `*.mp4`, and other large binary assets via `.gitattributes` so the repo doesn't bloat and diffs stay usable.
- **`.gitignore` the generated folders:** `Library/`, `Temp/`, `Obj/`, `Build/`, `Logs/`, `.vs/`, `*.csproj`, `*.sln` — these regenerate from source and shouldn't be versioned.
- **Enable "Smart Merge" (UnityYAMLMerge)** and configure it as the merge driver for `.unity`/`.prefab` files so scene/prefab conflicts get semantic merging instead of manual YAML surgery.
- **Scene ownership:** when multiple people must edit the same scene, split work into separate additive scenes or prefabs where possible — YAML merge reduces pain but doesn't eliminate it, and simultaneous edits to the same GameObject hierarchy still risk data loss.
- **Lock binary assets you can't merge** (Unity Teams/Plastic SCM locking, or a Git LFS lock convention) so two people don't edit the same `.fbx` or large texture simultaneously.

## Testing

Use the Unity Test Framework (UTF, formerly Unity Test Runner):

- **EditMode tests** for pure logic — anything not requiring the player loop (damage calculations, inventory logic, save-data serialization). Fast, no scene load, run these in CI.
- **PlayMode tests** for behavior that needs the Unity runtime (physics interactions, coroutines, `Update` timing). Slower — reserve for what EditMode genuinely can't cover.
- Structure gameplay logic to be EditMode-testable by keeping it in plain C# classes (see interfaces above) rather than baked directly into `MonoBehaviour.Update`.
- Wire UTF into CI (`Unity -runTests -batchmode`) as part of the quality gate pipeline.

## Build Pipeline and Content Delivery

- Use **Addressables** over legacy Resources/AssetBundles for anything loaded dynamically — supports remote content updates, memory-conscious loading/unloading, and avoids the flat, unmanaged `Resources` folder that bloats build size and load time.
- Strip unused code via Managed Stripping Level and confirm no reflection-dependent code breaks under stripping (test IL2CPP builds, not just Mono/Editor).
- Keep platform-specific build settings (texture compression, quality tiers) per-platform rather than one-size-fits-all — mobile and console/PC have very different memory and bandwidth budgets.
- Automate builds via Unity's command-line batch mode in CI so builds are reproducible and not dependent on one person's local Editor state.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "It's just a prototype, performance doesn't matter yet" | Allocation patterns and architecture set in prototypes calcify into production. Cheap fixes now are expensive refactors later. |
| "Singletons are simpler than events" | Simpler to write, harder to test and reason about. The coupling cost shows up as the project grows past a few systems. |
| "We'll fix the GC spikes later with a big optimization pass" | Allocations are cheapest to avoid at the call site. A "later" pass means auditing every `Update` in the codebase instead of a habit at write time. |
| "Binary scene files are fine, we're a small team" | Merge conflicts in binary YAML are unrecoverable without Force Text + Smart Merge. Small teams still collide on shared scenes. |
| "We don't need tests, it's all visual/gameplay feel" | Damage formulas, inventory logic, save/load, and economy math are exactly the bugs that silently corrupt player progress — and are fully unit-testable. |
| "Resources folder is easier than Addressables" | Easier at first, then every asset in `Resources/` ships in every build regardless of use, bloating size and load time. |

## Verification

After implementing or reviewing Unity code:

- [ ] No allocations in `Update`/`FixedUpdate` hot paths (verified via Profiler, not assumption)
- [ ] `GetComponent`/`Find*` calls are cached, not called per-frame
- [ ] Behaviour is composed from small components, not deep MonoBehaviour inheritance chains
- [ ] Systems communicate via events/interfaces, not a tangle of direct singleton references
- [ ] Designer-tunable values live in ScriptableObjects, not hardcoded constants
- [ ] Core gameplay logic is testable outside the Editor (EditMode tests exist and pass)
- [ ] Scene/prefab serialization is Force Text with Smart Merge configured
- [ ] `.gitignore`/Git LFS correctly exclude generated folders and track binary assets
- [ ] Frequently spawned objects use pooling, not raw `Instantiate`/`Destroy`
- [ ] Content loads through Addressables, not the `Resources` folder
- [ ] The game is composed of additive scenes, not one monolithic scene
- [ ] Profiler (CPU + Memory) and Frame Debugger were used to confirm any performance claim
