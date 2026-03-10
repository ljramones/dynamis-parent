# Graphics Cluster Architecture Synthesis

Date: 2026-03-10
Scope: `DynamisLightEngine`, `DynamisGPU`, `DynamisVFX`, `DynamisSky`, `DynamisTerrain`

This is a synthesis pass based on completed repo-level ratification reviews. It defines cross-repo ownership and boundary tightening priorities. It does not refactor code.

## 1. Cluster Ownership Map

### 1.1 Render Planning / Orchestration
- Owner: `DynamisLightEngine`
- Includes:
  - frame/pass ordering authority
  - render policy and feature scheduling policy
  - host-facing runtime contracts for rendering lifecycle
- Must not be owned by: `DynamisVFX`, `DynamisSky`, `DynamisTerrain`, `DynamisGPU`

### 1.2 GPU Execution / Resource Lifecycle
- Owner: `DynamisGPU`
- Includes:
  - ingestion/validation/upload seams
  - buffer/image/resource lifecycle
  - backend execution capabilities (pull-compatible)
- Must not be owned by: `DynamisLightEngine` policy layer, feature repos

### 1.3 Feature-Local Runtime + Render Data Generation
- Owner: feature repos
  - `DynamisVFX`: effect lifecycle, effect-local simulation, effect render-data generation
  - `DynamisSky`: sky/atmosphere/celestial modeling, sky-state derivation, sky render inputs
  - `DynamisTerrain`: terrain state, terrain-local LOD/streaming logic, terrain render inputs
- Must not include:
  - global frame graph ownership
  - generic GPU orchestration ownership
  - global world authority

### 1.4 Geometry Preparation / Shaping
- Owner: `MeshForge` (outside this cluster but relevant boundary)
- LightEngine and feature repos should consume prepared geometry outputs; they should not become canonical geometry shaping authorities.

### 1.5 Debug / Diagnostics Rendering
- Owner split:
  - feature-local debug data: feature repos
  - global debug pass ordering/integration: `DynamisLightEngine`
  - execution primitives: `DynamisGPU`

## 2. Dependency-Direction Model

Intended direction:

1. Feature repos (`VFX`, `Sky`, `Terrain`) -> depend on `DynamisGPU` **API-level execution seams**
2. `DynamisLightEngine` -> depends on feature APIs and `DynamisGPU` execution seams
3. `DynamisGPU` -> does not depend on feature repos or LightEngine policy
4. Feature repos -> do not own LightEngine pass policy; they provide feature-local update/record hooks

Practical rule set:

- Allowed:
  - `LightEngine` -> `DynamisGPU` (`dynamis-gpu-api` preferred)
  - `VFX/Sky/Terrain` -> `DynamisGPU` substrate
  - `LightEngine` -> feature repo APIs
- Disallowed target state:
  - feature repos and LightEngine broad reliance on `dynamis-gpu-vulkan` internals for stable contracts
  - feature repos encoding global pass-order policy as authoritative
  - `DynamisGPU` accepting render-policy ownership

## 3. Repeated Cross-Repo Pathology Patterns

### Pattern A: Direct `dynamis-gpu-vulkan` Coupling
Repeated in:
- `DynamisVFX`
- `DynamisSky`
- `DynamisTerrain`
- `DynamisLightEngine` backend implementations

Risk:
- execution internals become de facto stable API
- boundary between policy/features and execution substrate erodes

### Pattern B: Raw Backend Handle Leakage in Public Feature APIs
Repeated in:
- `VfxFrameContext.commandBuffer` / draw context types
- `Sky` and `Terrain` frame/resource contracts with raw handles
- `TerrainGpuResources` handle-heavy public shape

Risk:
- public contracts freeze backend-specific assumptions
- harder backend substitution and seam hardening

### Pattern C: Backend Package Over-Export
Repeated in:
- `dynamissky-vulkan` exports many internal backend packages
- `dynamisterrain-vulkan` exports many backend packages

Risk:
- implementation details become externally consumable contract accidentally

### Pattern D: Weakly Typed Integration Seams
Repeated in:
- `TerrainService.setSkySource(Object skySource)`

Risk:
- integration authority and ownership become ambiguous
- drift into ad-hoc policy wiring

### Pattern E: Phase-Ordering Assumptions Outside Authority Layer
Observed in:
- feature integration docs/code with strong order assumptions

Risk:
- hidden render policy leaks into feature repos
- LightEngine orchestration authority weakens

## 4. Authority vs Feature Distinction (Cluster Rule)

### Engine authority repos
- `DynamisLightEngine`: render planning authority
- `DynamisGPU`: execution/orchestration substrate authority

### Feature repos
- `DynamisVFX`, `DynamisSky`, `DynamisTerrain`

Feature repos are allowed to:
- own feature-local state/simulation/modeling
- generate render-facing feature data
- consume world/physics/sky inputs via typed boundaries

Feature repos are not allowed to:
- own frame graph / global pass planning
- own generic GPU resource orchestration policy
- expose backend internals as stable public contract surfaces

## 5. Boundary-Tightening Priority List

### Highest Priority
1. **Constrain direct `dynamis-gpu-vulkan` coupling** across LightEngine/VFX/Sky/Terrain to narrower seams where feasible.
2. **Remove raw GPU handles from public feature APIs** (or isolate behind typed abstractions) for VFX/Sky/Terrain stable surfaces.
3. **Reassert LightEngine as sole render-planning authority** with explicit contract that feature repos provide feature phases, not global pass policy.

### Medium Priority
4. **Narrow backend package exposure** in Sky/Terrain Vulkan modules (reduce accidental public surface).
5. **Replace weakly typed feature seams** (e.g., Terrain sky source typing) with explicit typed integration contracts.
6. **Clarify feature-to-LightEngine phase contracts** so ordering assumptions are declarative and host-controlled.

### Watch Only
7. Keep Sky “environment authority” language aligned with WorldEngine world-authority model (wording + docs consistency).
8. Watch Terrain meshforge/physics integration modules to ensure they remain terrain-local adapters, not generalized ownership layers.
9. Watch LightEngine mesh/geometry shaping behavior to prevent continued overlap with MeshForge.

## 6. Cluster-Level Conclusions

- `DynamisGPU` remains the clean execution substrate anchor.
- `DynamisLightEngine` is the primary boundary hotspot and still needs tightening against execution and shaping overlap.
- `DynamisVFX`, `DynamisSky`, and `DynamisTerrain` are in the correct feature role, but share the same recurring coupling/exposure issues.
- The cluster now has enough evidence to shift from isolated reviews to a targeted tightening plan.

## 7. Recommended Next Phase

**Recommendation: targeted graphics-cluster tightening plan** (not more isolated repo reviews yet).

Plan focus should be limited and pragmatic:
1. define target seam contracts between LightEngine <-> GPU and Feature <-> GPU
2. prioritize API-surface tightening (handle leakage + weak typing)
3. sequence implementation in small non-breaking slices

This should be done before resuming broad repo-by-repo review work outside the graphics cluster.
