# Daggerfall Unity Multiplayer (DFCoop) Developer Guidelines

## Architecture & Layer Discipline

We follow a strict 3-layer architecture to keep the fork cleanly rebasable on upstream `Interkarma/daggerfall-unity`:

1. **Layer 1: Upstream DFU (`Assets/Scripts/**`, `Assets/Scenes/**`, etc.)**
   - Tracked against upstream remote `https://github.com/Interkarma/daggerfall-unity.git`.
   - Never write multiplayer logic directly inside core DFU files.

2. **Layer 2: Hook Layer (`Assets/DFCoop/Hooks/**`)**
   - Any modifications to upstream DFU files must be **additive only**, lightweight (<5 lines), contain **no networking logic**, and have **no dependency on networking libraries** (e.g. no `using Mirror`).
   - Hooks should invoke static events (e.g. `DaggerfallHooks.OnEnemySpawned`) or call lightweight interfaces.
   - Every core hook edit must be documented in `Assets/DFCoop/HOOKS.md` with rationale.
   - Core edits are kept in isolated, rebasable commits on `coop/hooks` or `coop/main`.

3. **Layer 3: Multiplayer Implementation (`Assets/DFCoop/Runtime/**`, `Assets/DFCoop/Editor/**`)**
   - All networking, synchronization, dedicated server bootstrap, authority, RPCs, and persistence live here.
   - Encapsulated in its own Assembly Definitions (`DFCoop.Runtime.asmdef`, etc.).

## Reference Repository (`../dfu-coop-reference/`)

- `../dfu-coop-reference/` is a read-only reference from an earlier co-op fork (`Visty93/Daggerfall-Unity-Co-op`).
- **NEVER** edit files inside `dfu-coop-reference/`.
- **NEVER** copy code verbatim without deliberate redesign. The reference had fatal flaws (e.g. world state tied to player prefab, client-authoritative vitals, no headless support, disabled interest management).
- Use it to understand DFU quirks (terrain floating origin seams, exterior frame coords, dungeon generation parameters, sprite billboard rendering).

## Coding Standards

- Target Unity: 2019.4 LTS baseline (C# 7.3 / .NET Standard 2.0).
- Keep dedicated-server execution path free of UI, Camera, AudioListener, and `GameManager.Instance.PlayerObject` assumptions.
- World state (time, weather, loot, doors) must live on server-owned singletons, never on player prefabs.
