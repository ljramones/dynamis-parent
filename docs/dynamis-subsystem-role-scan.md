# Dynamis Subsystem Role Scan

## Purpose

This document confirms the likely architectural role of several Dynamis subsystems whose exact authority boundaries still need repo-grounded verification before producing the engine-wide authority map.

It is a **role-confirmation scan**, not a full boundary ratification review.

---

## DynamisScripting

### What the repo appears to do

`DynamisScripting` is an 11-module runtime stack (`scripting-api`, `spi`, `dsl`, `canon`, `oracle`, `chronicler`, `percept`, `society`, `economy`, `runtime`, `ashford`).

Key public/runtime surfaces indicate a world-rule and narrative simulation host:
- `WorldOracle` contract with `validate/shape/commit`
- `DefaultWorldOracle` implementation with validate-shape-commit phases
- `ScriptingRuntime` tick runtime, intent/canon/event wiring, and patch application
- `CanonLog`, `PerceptBus`, DSL compile/evaluation, and chronicler graph scheduling surfaces

The package and module structure points to scripting plus canonical simulation policy execution, not just lightweight script embedding.

### Likely intended architectural role

Scripting host/runtime plus rule-policy simulation layer (oracle/chronicler/canon/percept pipeline).

### Should exclusively own

* Script/runtime contracts and execution boundaries (intent/percept/canon APIs)
* Deterministic rule evaluation and script-pipeline hosting
* Script-level extension/plugin seams (SPI)

### Should clearly not own

* Global world lifecycle orchestration authority (WorldEngine ownership)
* ECS/SceneGraph/Physics substrate ownership
* Session persistence ownership
* Content catalog/asset authority

### Closest subsystem relationships

* WorldEngine
* Event
* Expression
* AI
* Session
* Content

### Confidence

Medium

### Later full review needed

Yes

---

## DynamisContent

### What the repo appears to do

`DynamisContent` is a 3-module runtime content layer (`content-api`, `content-core`, `content-runtime`) centered on asset identity, manifest resolution, loader registration, and cache.

Key surfaces:
- `AssetManager`, `AssetLoader`, `AssetResolver`, `AssetCache`
- `AssetId`/`AssetType` identity contracts
- `DefaultAssetManager` resolution and loader dispatch
- `ContentRuntime` wiring builder (manifest + loaders + cache)

No world tick/orchestration code appears in the repo; structure is runtime content lookup/resolve focused.

### Likely intended architectural role

Runtime content identity/manifest/loader/cache authority.

### Should exclusively own

* Runtime content identity and type contracts
* Runtime manifest lookup/resolution and loader dispatch
* Runtime content cache/invalidation behavior

### Should clearly not own

* Build/import/bake pipeline orchestration (AssetPipeline ownership)
* Session save/load authority
* World lifecycle orchestration
* Rendering/GPU authority

### Closest subsystem relationships

* AssetPipeline
* Session
* WorldEngine
* ECS
* SceneGraph

### Confidence

High

### Later full review needed

Probably

---

## DynamisSession

### What the repo appears to do

`DynamisSession` is a 3-module save/load stack (`session-api`, `session-core`, `session-runtime`) with explicit ECS snapshot export/import and file IO serialization.

Key surfaces:
- `SessionManager` (`newGame`, `save`, `load`)
- `WorldSnapshotter`
- `SaveGame`/`EcsSnapshot` model contracts
- `DefaultWorldSnapshotter`, `SaveGameReader/Writer`, `DefaultSessionManager`

Repo clearly centers on persistence/session state roundtrip for ECS world data.

### Likely intended architectural role

Session/persistence authority for save/load and world snapshot restoration.

### Should exclusively own

* Save/load lifecycle and slot-file persistence boundaries
* Snapshot serialization/deserialization and format/version boundaries
* ECS state export/import codec registry boundary

### Should clearly not own

* World runtime orchestration/tick authority
* Gameplay simulation policy authority
* Content runtime authority
* Rendering/UI/input authority

### Closest subsystem relationships

* ECS
* WorldEngine
* Content
* Event

### Confidence

High

### Later full review needed

Probably

---

## DynamisUI

### What the repo appears to do

`DynamisUI` is a 5-module UI framework (`ui-api`, `ui-core`, `ui-widgets`, `ui-debug`, `ui-runtime`) with retained scene graph, renderer SPI, widgets, debug overlays, and runtime wiring.

Key surfaces:
- `UIRenderer` SPI (renderer-agnostic drawing contract)
- `UIStage` / `UINode` retained hierarchy and event dispatch
- widget packages (HUD/dialogue/inventory/menu)
- `DebugOverlay` and performance/AI inspector overlays
- `UIRuntime` integration with `EventBus` and localization rebind

Structure indicates presentation-layer ownership with external renderer implementations.

### Likely intended architectural role

Renderer-agnostic UI framework/runtime (presentation and UI-local interaction).

### Should exclusively own

* UI scene graph/layout/widget composition
* UI-local event dispatch and debug overlay presentation
* UI runtime frame/update/render orchestration at presentation scope
* Renderer abstraction contracts for UI drawing

### Should clearly not own

* Raw input authority (Input subsystem ownership)
* World/simulation authority
* Scripting policy authority
* Render planning/GPU execution authority

### Closest subsystem relationships

* Input
* Event
* Localization
* WorldEngine
* LightEngine
* AI

### Confidence

High

### Later full review needed

Probably

---

## Summary

| Subsystem | Likely Role | Confidence | Full Review Needed? |
|---|---|---|---|
| DynamisScripting | Scripting host/runtime plus rule-policy simulation layer | Medium | Yes |
| DynamisContent | Runtime content identity/manifest/loader/cache layer | High | Probably |
| DynamisSession | Session persistence/save-load and world snapshot layer | High | Probably |
| DynamisUI | Renderer-agnostic UI framework/runtime and widgets/overlays | High | Probably |
