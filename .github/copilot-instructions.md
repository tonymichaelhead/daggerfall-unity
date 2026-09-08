# Daggerfall Unity Multiplayer (DFMP) Developer Guidelines

## Architecture & Layer Discipline

We follow a strict 3-layer architecture to keep the fork cleanly rebasable on upstream `Interkarma/daggerfall-unity`:

1. **Layer 1: Upstream DFU (`Assets/Scripts/**`, `Assets/Scenes/**`, etc.)**
   - Tracked against upstream remote `https://github.com/Interkarma/daggerfall-unity.git`.
   - Never write multiplayer logic directly inside core DFU files.

2. **Layer 2: Hook Layer (`Assets/DFMP/Hooks/**`)**
   - Any modifications to upstream DFU files must be **additive only**, lightweight (<5 lines), contain **no networking logic**, and have **no dependency on networking libraries** (e.g. no `using Mirror`).
   - Hooks should invoke static events (e.g. `DaggerfallHooks.OnEnemySpawned`) or call lightweight interfaces.
   - Every core hook edit must be documented in `Assets/DFMP/HOOKS.md` with rationale.
   - Core edits are kept in isolated, rebasable commits on `coop/hooks` or `coop/main`.

3. **Layer 3: Multiplayer Implementation (`Assets/DFMP/Runtime/**`, `Assets/DFMP/Editor/**`)**
   - All networking, synchronization, dedicated server bootstrap, authority, RPCs, and persistence live here.
   - Encapsulated in its own Assembly Definitions (`DFMP.Runtime.asmdef`, etc.).

## Reference Repository (`../dfu-coop-reference/`)

- `../dfu-coop-reference/` is a read-only reference from an earlier co-op fork (`Visty93/Daggerfall-Unity-Co-op`).
- **NEVER** edit files inside `dfu-coop-reference/`.
- **NEVER** copy code verbatim without deliberate redesign. The reference had fatal flaws (e.g. world state tied to player prefab, client-authoritative vitals, no headless support, disabled interest management).
- Use it to understand DFU quirks (terrain floating origin seams, exterior frame coords, dungeon generation parameters, sprite billboard rendering).
- Treat reference implementations as behavioral evidence, not a source of naming or design conventions. Do not inherit its variable names, method names, structure, or abstractions by default; redesign from the requirement using clear, conventional C# names and this project's layer boundaries.
- Prefer simple, readable, maintainable code that another developer can understand locally. Follow established DFU/DFMP patterns and industry-standard practices when they improve clarity, rather than preserving legacy fork terminology or implementation details.

## Coding Standards

- Target Unity: 2019.4 LTS baseline (C# 7.3 / .NET Standard 2.0).
- Keep dedicated-server execution path free of UI, Camera, AudioListener, and `GameManager.Instance.PlayerObject` assumptions.
- World state (time, weather, loot, doors) must live on server-owned singletons, never on player prefabs.

## Testing

- For every DFMP behavior change, add or update focused automated tests when the behavior can be tested without a full Unity player run.
- Prioritize tests for authority boundaries, message validation, lifecycle transitions, coordinate conversion, and persistence over superficial getter/setter coverage.
- Extract pure helpers for deterministic logic so they can be covered by EditMode tests.
- Keep runtime smoke tests for DFU/Mirror integration and document the expected log evidence when an automated test is impractical.
- Before committing, run the focused EditMode tests for touched DFMP code and perform the relevant headless or graphical integration smoke test for networking/world-transition changes.

## Roadmap & Milestone Discipline

- Strictly follow `Assets/DFMP/ROADMAP.md` for milestone scope, architectural rules, MVP boundaries, and server-owned vs client-local responsibilities.
- When planning or implementing features, confirm the target milestone in `Assets/DFMP/ROADMAP.md` and adhere to its design principles.
- Do not jump ahead into out-of-scope milestone features or violate the authoritative/personal state boundaries defined in the roadmap.

