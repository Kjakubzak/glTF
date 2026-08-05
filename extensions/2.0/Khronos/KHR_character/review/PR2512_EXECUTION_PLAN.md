# KHR Character Phase 1: proposal and implementation execution plan (Informative)

Plan snapshot: 2026-08-04
Starting proposal head: [`bf38201bc5f4ed5de815c79ad43f4fc281bee83f`](https://github.com/KhronosGroup/glTF/commit/bf38201bc5f4ed5de815c79ad43f4fc281bee83f)

> Non-normative execution proposal; not an adopted Khronos Working Group decision.

This plan implements the work identified in the [pre-RC assessment](./PR2512_ASSESSMENT.md), which records the starting implementation commit hashes. It deliberately freezes the extension wire contract before migrating testbeds. Experimental branches may compare alternatives, but prototype behavior must not become the normative source of truth.

## Success criteria

The work is complete when:

- each advancing extension has one coherent normative meaning, fallback, validity model, and support contract;
- no required behavior exists only in the PR body, an external working document, a prototype, or a test;
- proposal schemas/examples and procedural diagnostics match the selected semantics;
- a public, pinned corpus distinguishes every formerly ambiguous model;
- independent Three.js and Unity evaluators agree on deterministic driver vectors and pre-mix outputs;
- validator positive/negative fixtures report stable expected issue codes and paths;
- reference-pose and per-view visibility outputs match public expected results;
- round trips preserve normalized structural/semantic wire meaning, stable references, and unknown/custom tokens and companion payloads;
- required CI has no skipped conformance fixtures or golden files;
- status, registry, branch synchronization, checks, and legal requirements are ready for the selected extensions to enter RC independently.

## Critical sequencing

```text
WG design decisions and asset audit
    -> normative READMEs, schemas, names, and support rules
    -> public distinguishing corpus and expected results
    -> validator rules and recognition sentinel
    -> pure Three.js and Unity evaluators in parallel
    -> converter/exporter migration and asset regeneration
    -> host mixer/render adapters and Unity sandbox integration
    -> RC-entry evidence and repository checkpoint
    -> official Validator and Sample Renderer/Viewer integration
```

Do not support multiple expression interpretations through runtime auto-detection. Do not migrate production fixtures until the corresponding wire decision is pinned.

# Part I — Extension proposal changes

## Phase 0 — Synchronize and audit before wire decisions

### Tasks

- [ ] Synchronize the proposal branch with target glTF `main` at execution time.
- [ ] Rerun repository schema, link, license, and formatting checks before semantic edits.
- [ ] Pin every public expression asset used by the Three.js and Unity testbeds.
- [ ] Audit those assets for:
  - one-key samplers;
  - first samples that differ from static/default values;
  - different sampler time ranges and intentional channel delays;
  - typed-child filtering assumptions;
  - one animation referenced by several expression entries;
  - mixed-purpose response animations;
  - first-key-relative or in-place additive conversion;
  - mapping direction and vocabulary assumptions.
- [ ] Publish model-neutral raw counts and asset hashes first, then report incompatibilities and candidate migrations separately for global phase, per-sampler phase, and fixed-target weighting.

### Acceptance

- The audit is public and commit-pinned.
- No design choice is justified only by uncounted implementation assumptions.
- Experimental implementations are explicitly not treated as evidence of conformance until the wire decision is final.

## Phase 1 — Record the blocking design resolutions

The Working Group should record one resolution for each row. The recommendation column is a proposed starting point, not current normative meaning.

| Decision | Recommended resolution | Alternative/tradeoff |
|---|---|---|
| Expression coefficient | One animation-wide response phase | Per-sampler transfer curves are viable only as a complete replacement if migration cost warrants them. Fixed-target weighting is a separate complete model. |
| Global time | `t = e * T`, where `T` is the maximum last input time among all channel-referenced samplers | Canonical `t=e` is simpler but constrains the timeline to `[0,1]`. |
| Sample meaning | Ordinary absolute core samples | Deltas require exact scalar, vector, scale, morph, texture, and quaternion rules. |
| Zero input | No property contribution; clear retained response influence | Writing the static neutral sample may overwrite a live underlay. |
| One-key expression response | Prohibit; author explicit inactive/active keys | Synthetic base-to-target sampling creates non-core interpolation behavior. |
| Typed expression children | Classify/validate; parent owns the whole response animation | Filtering requires strict partial-support and `extensionsRequired` rules and makes parent-only examples inert. |
| Driver source | Current scalar supplied by host; no Phase 1 mutable driver property | Portable animation-to-driver wiring needs a mutable Asset Object Model (AOM) property, order, and cycle rules. |
| Mapping | Add an explicitly named endpoint-command→native-driver input operation | Silently reversing existing forward native→endpoint bytes is a breaking reinterpretation. |
| Unknown custom mask | Preserve token; numeric evaluator factor `1.0`; companion vendor extension for portable behavior | Any fallback may lose author intent; unknown→`blend` changes meaning unpredictably. |
| Reference pose | Nonempty, static, one-key, ordinary node TRS animation | Arbitrary animations cannot be validated as static pose storage. |
| Visibility | Pure per-render-view predicate | Mutating node/global visibility fails descendant overrides and simultaneous views. |
| Required support | Feature-specific minimum behavior | “May ignore even when required” makes `extensionsRequired` meaningless. |

### Acceptance

- Exactly one expression interpretation is selected.
- Every mapping operation has a name, direction, input domain, and output domain.
- The six gate families—expressions, mapping, custom masks, reference poses, visibility, and required support—have recorded resolutions.
- The resolution record cites the audit and proposal commit.
- No resolution depends solely on an implementation or PR description.

## Phase 2 — Establish shared proposal conventions and scope

### Primary files

- `extensions/2.0/Khronos/KHR_character/README.md`
- `extensions/2.0/Khronos/KHR_character/schema/glTF.KHR_character.schema.json`
- all 13 proposal READMEs and affected schemas

### Tasks

- [ ] Add common normative/informative boilerplate to every advancing README.
- [ ] Mark background, known implementations, and intended limitations `(Informative)`.
- [ ] Move required BCP 14 statements into normative sections before removing contradictory language from Notes, Implementation Notes, and Examples.
- [ ] State the one-character-per-glTF-2.0-asset limitation.
- [ ] Define `rootNode` as the designated character root without inventing a universal skin/scene ownership model.
- [ ] Remove unconditional `KHR_xmp_json_ld` dependency when no XMP data is used, or define the behavior that justifies it.
- [ ] Publish one extension placement/dependency and `extensionsUsed`/`extensionsRequired` matrix.
- [ ] Freeze wire names and token spelling:
  - `morphtarget` versus `morph_target`;
  - `lookat` versus `look_at`;
  - joint versus node-transform terminology;
  - texture versus texture-transform terminology;
  - standard visibility/camera/pose tokens.

### Acceptance

- An asset with no XMP data does not declare XMP unless the WG records a concrete reason.
- Every extension has one placement and dependency contract.
- Informative material cannot be mistaken for normative behavior.
- Any rename updates proposal directories, schemas, READMEs, examples, registry draft, PR description, and a downstream migration manifest atomically. Validator/Unity/Three types update later against the pinned proposal commit.

## Phase 3 — Freeze the expression response contract

### Primary files

- `extensions/2.0/Khronos/KHR_character_expression/README.md`
- `extensions/2.0/Khronos/KHR_character_expression/schema/glTF.KHR_character_expression.schema.json`
- `extensions/2.0/Khronos/KHR_character_expression_joint/README.md`
- `extensions/2.0/Khronos/KHR_character_expression_joint/schema/KHR_character_expression.KHR_character_expression_joint.schema.json`
- `extensions/2.0/Khronos/KHR_character_expression_morphtarget/README.md`
- `extensions/2.0/Khronos/KHR_character_expression_morphtarget/schema/KHR_character_expression.KHR_character_expression_morphtarget.schema.json`
- `extensions/2.0/Khronos/KHR_character_expression_texture/README.md`
- `extensions/2.0/Khronos/KHR_character_expression_texture/schema/KHR_character_expression.KHR_character_expression_texture.schema.json`

### Normative work for the recommended global-phase package

- [ ] Define each expression entry as a stateless response with a current host-supplied driver whose conceptual default is `0`.
- [ ] Require a supplied driver to be finite, use `0` when no driver is supplied, and clamp finite values to `[0, 1]`.
- [ ] Keep driver acquisition, arbitration, smoothing, cadence, and final nonzero mixing host-defined.
- [ ] Let `C` be every channel in the referenced response animation.
- [ ] Define `T` as the maximum final decoded input key among all samplers referenced by `C`, excluding truly unused samplers; accessor metadata alone does not establish this value.
- [ ] Require `T > 0`; sample supported channels at common time `t=e*T` using core interpolation and endpoint clamping.
- [ ] Keep unsupported optional target samplers in `T`; reject missing required support.
- [ ] Return ordinary absolute core samples before host mixing.
- [ ] Require the isolated sample at `t=0` to equal the resolved static/default property value after accessor decoding. Resolve node TRS from authored values or core defaults; morph weights from node weights, then mesh weights, then zero; pointer targets from their defined/default AOM values; and texture transforms from their extension defaults. Publish comparison tolerances and treat normalized quaternions `q` and `-q` as equivalent.
- [ ] At `e=0`, submit no property response and clear prior retained influence; outgoing masks remain a separate pre-mask-vector effect.
- [ ] Prohibit one-key samplers referenced by expression response channels.
- [ ] Make typed children classifiers/validators rather than execution filters; unclassified channels remain part of `C`, and duplicate classification is invalid.
- [ ] Give repeated animation references independent driver/evaluation identity.
- [ ] Do not mutate or redefine ordinary playback of the referenced animation.
- [ ] Explicitly exclude portable in-file driver animation from Phase 1, unless the AOM/order/cycle work is added.

### Schema work

- [ ] Make expression entries proper `glTFProperty` objects.
- [ ] Require nonempty expression identifiers.
- [ ] Decide duplicate-label validity; keep array indices authoritative.
- [ ] Require nonempty channel lists and structurally unique indices within each individual list.
- [ ] Align defaults, extension placement, and required-child rules with the chosen package.

### Required distinguishing cases to specify now and materialize in Phase 10

- `e=0`, `.25`, `.5`, `1`;
- direct `.5` versus history `0,1,.5`;
- different update rates with identical current input;
- sampler ranges `[0,1]` and `[0,4]`;
- delayed ranges `[0,.25]` and `[.75,1]`;
- nonuniform `STEP`, `LINEAR`, and `CUBICSPLINE`;
- static nonzero node, mesh-weight, and texture-transform defaults;
- one-key rejection;
- unsupported optional long-duration target without rescaling;
- two entries sharing an animation at different simultaneous drivers;
- `e=1` with no loop/wrap;
- zero response over a live underlay.

### Per-sampler alternative

If the WG selects per-sampler response curves instead, replace the global rules rather than supporting both:

```text
tau = firstInput + e * (lastInput - firstInput)
```

The spec must then define typed filtering, unsupported-child fallback, untyped entries, independent timestamp meaning, exact one-key `LINEAR` base-to-target synthesis, and rejection of one-key `STEP`/`CUBICSPLINE`.

### Fixed-target alternative

Selecting fixed-target weighting stops this phase-sampling plan until an expanded design defines immutable base/target extraction, scalar/vector/scale/morph/texture/quaternion math, one-key semantics, and the boundary with a live underlay. It must not reuse either phase package implicitly.

### Acceptance

- The phase-package equations derive unambiguous expected values distinguishing global phase, canonical `t=e`, and per-sampler curves. Fixed-target cases demonstrate why a separate contract is required rather than claiming values before that contract exists.
- The text defines an identity property response at `e=0`; it does not claim continuity against a live underlay for nonzero input, whose final composition remains host-defined.
- No expression behavior is normative only in discussion or implementation notes.

## Phase 4 — Define mapping and masks as one scalar pipeline

### Primary files

- `extensions/2.0/Khronos/KHR_character_expression_mapping/README.md`
- `extensions/2.0/Khronos/KHR_character_expression_mapping/schema/glTF.KHR_character_expression_mapping.schema.json`
- `extensions/2.0/Khronos/KHR_character_expression_mask/README.md`
- `extensions/2.0/Khronos/KHR_character_expression_mask/schema/KHR_character_expression.KHR_character_expression_mask.schema.json`
- `extensions/2.0/Khronos/KHR_character_expression_mask/schema/KHR_character_expression_mask.mask.schema.json`

### Pipeline

```text
host source arbitration
    -> endpoint-command or direct-native input surface
    -> input normalization
    -> reverse input mapping, when selected
    -> final native clamp
    -> immutable pre-mask native vector
    -> mask factors
    -> effective expression drivers
    -> response sampling
    -> host-defined nonzero property composition
```

### Mapping tasks

- [ ] Preserve and name the existing forward native→endpoint operation, or deliberately migrate/remove it.
- [ ] Prefer a separately named reverse endpoint-command→native-driver operation for playback.
- [ ] Define version/collision-safe mapping-set identity or an external-profile boundary.
- [ ] Define finite command/weight domains, repeated contributions, accumulation, unmapped values, and final clamp.
- [ ] Keep mapping-set selection and combining separate direct/mapped surfaces outside the format.
- [ ] Require nonempty keys and mapping sets.

### Mask tasks

- [ ] Consolidate the current immutable-snapshot and multiplicative `blend`/`block` equations.
- [ ] State that duplicates, self-masks, and cycles use those equations without recursive evaluation.
- [ ] Preserve unknown custom tokens.
- [ ] Give an unknown type numeric identity factor `1.0`.
- [ ] Require vendor qualification and a companion vendor extension on the same mask object for portable custom semantics.
- [ ] List that extension in `extensionsUsed`, and in `extensionsRequired` when its behavior is essential.
- [ ] Replace exact “VRM-compatible” claims with “inspired by VRM” unless a conversion profile exists.

### Acceptance

- One stored mapping cannot be interpreted in opposite directions.
- Mapping and masks have executable equations and one ordering.
- Unknown types never silently become `blend`.
- Tests cover repeated sums, clamps, unmapped values, self/cycles, duplicates, unknown types, and immutable-snapshot behavior.

## Phase 5 — Make reference-pose and skeleton data procedurally valid

### Primary files

- `extensions/2.0/Khronos/KHR_character_reference_pose/README.md`
- `extensions/2.0/Khronos/KHR_character_reference_pose/schema/animation.KHR_character_reference_pose.schema.json`
- `extensions/2.0/Khronos/KHR_character_skeleton_mapping/README.md`
- `extensions/2.0/Khronos/KHR_character_skeleton_mapping/schema/glTF.KHR_character_skeleton_mapping.schema.json`
- `extensions/2.0/Khronos/KHR_character_skeleton_mapping/schema/KHR_character_skeleton_mapping.jointAssociation.schema.json`

### Reference-pose tasks

- [ ] Require a nonempty tagged animation.
- [ ] Allow only ordinary node translation/rotation/scale (TRS) channels.
- [ ] Require exactly one input value for each referenced sampler and core-valid output cardinality.
- [ ] Exclude `CUBICSPLINE`, which requires at least two keyframes.
- [ ] Resolve omitted components from static values or core defaults.
- [ ] Treat T/A/I/custom labels as author assertions; stored local TRS is authoritative.
- [ ] Do not require `t=0`, `STEP`, a skin, complete joint coverage, unique pose types, or exact anatomy.
- [ ] Repair or remove inverse-bind migration and universal compatibility claims.
- [ ] Normalize line endings.

### Skeleton-mapping tasks

- [ ] Define entries as generic vocabulary-role→node associations for Phase 1; do not require skin-joint membership.
- [ ] Do not promise a retargeting solver unless one is defined.
- [ ] Give rig vocabulary sets version/collision-safe identity or an external-profile boundary.
- [ ] Require nonempty set and role keys.
- [ ] Keep node indices authoritative; optional names are diagnostics.

### Acceptance

- Positive and negative cases are validatable without running a retargeter.
- Multi-key/time-varying or non-TRS animations cannot satisfy the reference-pose validity rules.
- The text makes no unsupported lossless/universal conversion claim.

## Phase 6 — Replace visibility mutation with pure predicates

### Primary files

- `extensions/2.0/Khronos/KHR_node_visibility_hint/README.md`
- `extensions/2.0/Khronos/KHR_node_visibility_hint/schema/node.KHR_node_visibility_hint.schema.json`
- `extensions/2.0/Khronos/KHR_mesh_primitive_visibility_hint/README.md`
- `extensions/2.0/Khronos/KHR_mesh_primitive_visibility_hint/schema/mesh.primitive.KHR_mesh_primitive_visibility_hint.schema.json`

### Normative model

```text
hintRole(n) = nearest role on n or an ancestor; otherwise always
hintVisible(n, C) = standard role predicate for render context C
declaredVisible(n) = AND of KHR_node_visibility.visible (default true) on n and ancestors
renderVisualContent(n, C) = hintVisible(n, C) AND declaredVisible(n)

renderPrimitiveInstance(n, p, C) =
    renderVisualContent(n, C) AND primitiveHintVisible(p, C)
```

### Tasks

- [ ] Define a complete standard-role truth table and unknown-role fallback.
- [ ] Evaluate shared meshes per node instance and simultaneous contexts independently.
- [ ] State that descendant hint metadata overrides inherited hint metadata but not explicit ancestor visibility.
- [ ] Keep `KHR_node_visibility` an optional compositional interaction rather than a mandatory dependency. When present, its declared visibility is ANDed with the hint predicate; declare/require it through ordinary extension rules when an asset depends on both.
- [ ] Require observable equivalence to independent per-view predicates. Internal techniques must not persist cross-view mutations or overwrite authored `KHR_node_visibility` state.
- [ ] Require actual visual suppression for required primitive support; transparent substitution is insufficient.
- [ ] Keep picking and arbitrary queries outside the visual-presentation contract.
- [ ] Leave context selection/transitions to the host.

### Acceptance

- The equations derive exact expected visible sets for descendant override, explicitly hidden ancestors, shared meshes, node-plus-primitive composition, unknown roles, and two supplied contexts.
- The contract prohibits persistent cross-view contamination without constraining internal implementation strategy.
- Required use has MUST-level minimum behavior or causes rejection when unsupported.

## Phase 7 — Tighten passive camera and look-at annotations

### Primary files

- `extensions/2.0/Khronos/KHR_node_camera_hint/README.md`
- `extensions/2.0/Khronos/KHR_node_camera_hint/schema/node.KHR_node_camera_hint.schema.json`
- `extensions/2.0/Khronos/KHR_node_lookat_target/README.md`
- `extensions/2.0/Khronos/KHR_node_lookat_target/schema/node.KHR_node_lookat_target.schema.json`
- `extensions/2.0/Khronos/KHR_node_lookat_target/figures/`

### Camera tasks

- [ ] Prohibit Phase 1 use in `extensionsRequired` while runtimes may ignore/adapt hints.
- [ ] Derive a selected pose using the core-camera global-transform scale convention.
- [ ] For coincident `targetNode`, fall back to the authored rotation.
- [ ] Define `framing_target` as a point of interest unless full framing inputs are added.
- [ ] State that the annotation does not suppress ordinary mesh content on the node.
- [ ] Remove controller/following/transition claims and unfinished contributor markers.

### Look-at tasks

- [ ] Keep the extension a passive marker.
- [ ] Allow required use with one minimum contract: recognize each marker and interpret the node's per-instance global translation as the target point; reject when required support is missing.
- [ ] Leave selection, priority, links, constraints, inverse kinematics, and gaze to the host.
- [ ] Repair the multi-parent example.
- [ ] Remove or clearly label tour, signage, networking, and automatic-behavior narratives as illustrative.

### Acceptance

- Camera hints never imply a controller.
- Look-at markers never orient a consumer by themselves.
- Each feature has one coherent `extensionsRequired` rule.

## Phase 8 — Cross-cutting schema, example, and source-of-truth pass

- [ ] Require nonempty labels, custom identifiers, dictionary keys, and required arrays.
- [ ] Define authoritative indices versus diagnostic names.
- [ ] Align `glTFProperty` inheritance, placement, and defaults.
- [ ] Validate complete examples; label incomplete snippets as partial.
- [ ] Remove dangling indices and repair morph-index, hierarchy, quaternion, and dependency errors.
- [ ] Move every mandatory rule out of Notes, Examples, PR discussion, and external documents.
- [ ] Remove claims of identical final runtime output where composition is host-defined.
- [ ] Limit JSON Schema work to structural constraints it can express.
- [ ] Assign referenced accessor counts, channel target domains, cross-child duplicate classification, cross-object indices, inactive-reference equality, and companion-extension declarations to explicit Validator diagnostics.

### Acceptance

- Every schema passes meta-schema checks.
- Every complete example validates.
- Every procedural rule has planned positive and negative cases.
- Terminology is consistent across all 13 extensions.

## Phase 9 — Proposal candidate freeze and implementation handoff

- [ ] Publish the resolution record, model-specific asset audit, and proposed expected vectors.
- [ ] Review each extension independently and identify which remain Draft during implementation work.
- [ ] Update the PR title/body and remove obsolete questions.
- [ ] Run repository schema, formatting, link, and license checks.
- [ ] Pin the proposal candidate commit and publish a handoff manifest for the corpus, Validator, Unity, and Three streams.
- [ ] Keep advancement status and registry changes pending until the Phase 15 evidence checkpoint.

The handoff to implementation work occurs after the relevant proposal phase is frozen. Expression/mapping/mask implementation blocks on Phases 1–4. Pose/skeleton, visibility, and camera/look-at streams may begin independently after Phases 5, 6, and 7 respectively.

# Part II — Testbed and downstream repository changes

## Phase 10 — Publish one shared informative conformance corpus

Suggested location: `extensions/2.0/Khronos/KHR_character/tests/` or a dedicated public corpus repository. Mark it informative and pin its manifest.

### Required files

- `manifest.json`: proposal commit, selected expression model, time rule, mapping operation, mask fallback, tolerances, and fixture hashes.
- `driver-oracles.json`: supplied input, normalized input, mapped vector, pre-mask vector, and effective post-mask vector.
- `response-oracles.json`: expression index, driver, requested time, target property, and absolute pre-mix sample.
- `pose-oracles.json`: expected local TRS and validator disposition.
- `visibility-oracles.json`: visible node-instance/primitive-instance sets per context.
- `validation-oracles.json`: expected issue code, severity, and JSON path.

### Rules

- [ ] Every consumer verifies fixture hashes and proposal commit.
- [ ] Final colliding property values are not portable oracles while mixing remains host-defined.
- [ ] Include all distinguishing expression, mapping, mask, pose, visibility, required-support, and negative pointer cases from Parts I.
- [ ] Include duplicate-label/index-authority and repeated-animation-reference cases separately.
- [ ] Include an unknown custom mask whose token and companion vendor-extension payload must survive normalized round trips.
- [ ] Include multiple skeleton mapping sets and verify generic associations remain separate from host retarget profiles.

## Phase 11 — Update the character-aware Validator fork

Repository: [`Kjakubzak/glTF-Validator`](https://github.com/Kjakubzak/glTF-Validator)

### Primary areas

- `lib/src/ext/KHR_character/khr_character.dart`
- `lib/src/ext/extensions.dart`
- `lib/src/errors.dart`
- `test/ext/KHR_character/`

### Tasks

- [ ] Align the manual Dart parser, linker, and diagnostics with the frozen schemas; the current fork has no character-schema generation step.
- [ ] Add diagnostics for the selected expression model, one-key validity, inactive-reference equality, channel ownership, typed domains, and required children.
- [ ] Validate mapping direction/domains/references/identifiers and empty dictionaries.
- [ ] Validate custom-mask qualification and companion extension declarations.
- [ ] Validate static TRS-only reference poses and feature-specific required support.
- [ ] Validate skeleton mapping set identity, nonempty keys, association shape, and the selected generic-node association rule.
- [ ] Remove the unconditional XMP dependency assertion if the proposal removes it.
- [ ] Add one red fixture per rule with committed expected report output.
- [ ] Add a recognition sentinel: a deliberately invalid character asset must emit a character-specific issue code.

### Acceptance

- Formatting, static analysis, build, and full tests pass.
- Every statically checkable normative rule has positive and negative coverage.
- Issue codes and JSON paths are stable for the pinned corpus and Validator version.
- Removing character support makes the recognition sentinel fail.

## Phase 12 — Update the Three.js viewer and converter

Repository: [`0b5vr/khr-character-testbed`](https://github.com/0b5vr/khr-character-testbed)

### Viewer expression runtime

Primary files:

- `viewer/src/khr-character/expressions/KHRCharacterExpressionLoaderPlugin.ts`
- `viewer/src/khr-character/expressions/KHRCharacterExpressionManager.ts`
- `viewer/src/khr-character/expressions/KHRCharacterExpression.ts`
- `viewer/src/khr-character/expressions/validateKHRCharacterExpressionChannelTargets.ts`

Create separate pure/runtime layers, for example:

- `KHRCharacterExpressionDriverPipeline.ts`
- `KHRCharacterExpressionEvaluator.ts`
- `ThreeExpressionMixerAdapter.ts`

Tasks:

- [ ] Build immutable response descriptors from glTF data.
- [ ] Implement only the selected mapping/clamp/snapshot/mask pipeline. Add endpoint-command input only if the frozen schema defines a reverse operation; otherwise keep forward mapping for reporting/export and never numerically invert it.
- [ ] Evaluate each expression independently and history-independently.
- [ ] Under global phase, compute `T` from the original animation's channel-referenced sampler inputs before dropping unsupported optional target domains; optional unsupported channels still affect `T`, truly unused samplers do not, and typed children remain classifiers.
- [ ] Do not mutate source clips with `AnimationUtils.makeClipAdditive`.
- [ ] Do not use cached `clipAction(clip, root)` as expression-entry identity.
- [ ] At zero, remove the response source and clear prior influence rather than writing the static sample.
- [ ] Use non-looping endpoint sampling so `e=1` cannot wrap.
- [ ] Keep additive/override behavior in an explicitly host-specific mixer adapter.
- [ ] Keep direct-native input separate from any selected mapping operation; do not silently merge surfaces.
- [ ] Drive expression slots by authoritative array index, or prohibit duplicate labels normatively and validate them. Names must not silently collapse entries.
- [ ] Reject an asset when an unsupported extension is listed in `extensionsRequired`; typed children remain classifiers for optional use, and unsupported optional target extensions omit response records without filtering channels out of the common-duration calculation.
- [ ] Preserve unknown mask tokens and companion vendor-extension payloads through round trips; evaluate unsupported types with the selected fallback, not `blend`.

### Converter

Primary areas:

- `converter/converter/expressions/appendKHRCharacterExpression.ts`
- `converter/converter/expressions/appendMorphtargetAnimation.ts`
- `converter/converter/expressions/appendTextureAnimation.ts`
- `converter/converter/expressions/appendBoneLookExpression.ts`
- `converter/converter/expressions/appendExpressionMasksFromOverride.ts`
- schema-derived TypeScript definitions under `schematypes/`

Tasks:

- [ ] Emit explicit inactive/active keys for the global model using resolved static/default values.
- [ ] Remove assumptions that texture offset/scale always equal core defaults.
- [ ] Migrate mapping bytes only if the frozen proposal changes them; never reinterpret or numerically invert the forward operation. Name native/endpoint sides unambiguously.
- [ ] Treat VRM→multiplicative-mask conversion as lossy unless an exact profile exists; require explicit opt-in or a custom profile.
- [ ] Synchronize public schema-derived types after proposal changes; add a generator before describing them as generated.

### Pose and visibility

- [ ] Add a reference-pose loader exposing every tagged pose by animation index, including duplicate pose labels, after validating exactly one key per referenced sampler.
- [ ] Keep generic skeleton vocabulary-role→node associations separate from optional VRM retarget adapters such as `viewer/src/createKHRCharacterVRMHumanoidRestPose.ts`.
- [ ] Replace persistent `.visible` mutation with a per-view predicate implementing nearest hint metadata AND ancestor `KHR_node_visibility` AND primitive role, per node instance.
- [ ] Suppress primitive draw participation with per-draw filtering, isolated draw objects, scoped all-pass no-draw material/shader substitution, or another route that does not mutate authored state or contaminate other views; generic transparent material is insufficient.

### Tests/CI

- [ ] Add Vitest unit tests; current packages have build scripts but no test script.
- [ ] Add Playwright/headless-render integration tests for visibility.
- [ ] Add required CI separate from Pages deployment.
- [ ] Fail on corpus hash drift and skipped conformance cases.

### Acceptance

- Viewer and converter builds/tests pass.
- Shared deterministic oracles match.
- Two contexts rendered in the same frame produce independent visible sets without cross-contamination.
- Hidden primitive cases affect neither color nor depth.
- Repeated animation references remain independent.

## Phase 13 — Refactor UnityGLTF around a pure wire evaluator

Repository: [`Kjakubzak/UnityGLTF`](https://github.com/Kjakubzak/UnityGLTF)

### Primary areas

- `Runtime/Scripts/KhrCharacter/Contracts/`
- `Runtime/Scripts/KhrCharacter/Evaluation/AdditiveExpressionSemantics.cs`
- `Runtime/Scripts/KhrCharacter/Components/ExpressionController.cs`
- `Runtime/Scripts/KhrCharacter/Import/KhrCharacterBaker.cs`
- `Runtime/Scripts/KhrCharacter/Import/KhrCharacterSkeletonBaker.cs`
- `Runtime/Scripts/KhrCharacter/Export/KhrCharacterExportContext.cs`
- schema/import/export classes

### Architecture

Create distinct responsibilities:

- `ExpressionDriverPipeline`: input validation, selected mapping, one final clamp, immutable snapshot, masks.
- `ExpressionResponseEvaluator`: core interpolation and absolute pre-mix samples.
- `ExpressionController`: Unity-specific Animator/additive/override integration.

### Tasks

- [ ] Under global phase, remove per-sampler normalization and one-key synthesis; typed children remain classifiers and the whole response animation is processed.
- [ ] Derive one duration from the original animation's channel-referenced sampler inputs before dropping unsupported optional targets; optional unsupported channels still affect `T`, truly unused samplers do not.
- [ ] Store ordinary absolute samples.
- [ ] Implement complete `CUBICSPLINE` interpolation.
- [ ] Make the wire evaluator stateless: zero input produces an empty response and a nonzero→zero transition carries no retained wire-space influence.
- [ ] Give the Unity applicator explicit ownership modes: a standalone owner may restore its owned targets, while an integrated/post-Animator adapter must not overwrite live animation-system output merely because an expression input becomes zero.
- [ ] Validate endpoint-command inputs in `[0,1]`; do not silently normalize them. Apply the mapping operation's single specified final clamp before the immutable mask snapshot.
- [ ] Implement endpoint-command input only if the frozen schema contains a reverse mapping operation; otherwise preserve forward mapping for reporting/export and never invert it.
- [ ] Stop implicitly combining direct and mapped values as wire behavior.
- [ ] Address expression entries by authoritative array index, or enforce normative label uniqueness. Names remain diagnostic rather than silently merging driver slots.
- [ ] Treat typed children only as classifiers, never channel filters. Unsupported optional target extensions omit their response records but their referenced samplers still contribute to `T`; reject required extensions unless the implementation supplies their complete behavior.
- [ ] Preserve unknown custom mask types and companion vendor-extension payloads separately from `Blend`; evaluate using the selected fallback and round-trip both.
- [ ] Preserve root/item/classifier/mapping-contribution payloads and requiredness, including unknown child extensions and companion `extensionsUsed` / `extensionsRequired` declarations.
- [ ] Validate initial-state equivalence and morph neutral precedence (`node.weights`, then `mesh.weights`, then zero) without using reference poses as expression neutral values.
- [ ] Decode sparse accessors and validate sampler counts, decoded time ordering, matrix-backed TRS targets, classifier overlap, duplicate property targets, pointer-to-`extras` targets, and output cardinality before evaluation.
- [ ] Represent texture scale, offset, rotation, and other supported material properties as independent response channels with their own samplers; let the Unity adapter fan a material response out to every material use it owns.
- [ ] Preserve ordinary animation entries and make clip autoplay suppression an explicit host option rather than mutating wrap modes or detaching an Animator as conformance behavior.
- [ ] Keep current additive/override logic as host-adapter policy and test it separately.

### Reference pose

- [ ] Enforce nonempty, ordinary TRS-only data with exactly one key per referenced sampler, valid cardinality, and no `CUBICSPLINE`.
- [ ] Resolve omitted components from static/default values.
- [ ] Import and expose every tagged pose by animation index, including duplicate `poseType` labels; do not stop at the first tag.
- [ ] Export canonical `STEP` at zero only as an authoring choice, not normative requirement.

### Skeleton mapping

- [ ] Preserve every generic vocabulary-role→node association and mapping set independently of Unity Humanoid/Mecanim validation.
- [ ] Keep `KhrCharacterSkeletonBaker.ResolveRig` and other retarget adapters as optional host profiles rather than wire interpretation.

### Visibility

Primary areas:

- `Runtime/Scripts/VisibilityHints/Components/ViewContextController.cs`
- `Runtime/Scripts/VisibilityHints/Components/NodeVisibilityHintSet.cs`
- `Runtime/Scripts/VisibilityHints/Components/PrimitiveVisibilityHintSet.cs`

Tasks:

- [ ] Expose a pure `(instance, primitive, context) -> bool` evaluator using nearest hint metadata AND ancestor `KHR_node_visibility` AND the primitive predicate.
- [ ] Apply it transiently through per-camera render hooks, isolated draw objects, scoped material/shader substitution, or another host route that produces the required visible set without mutating glTF-authored data or contaminating other views/instances.
- [ ] Treat generic alpha-zero substitution as insufficient because it does not guarantee absence from color, depth, shadow, picking, or other visual passes; a no-draw material/shader adapter remains valid when it suppresses all applicable passes within an isolated scope.
- [ ] Test Built-in and Universal Render Pipeline color, depth, shadows, two cameras in the same frame, shared meshes, and ancestor visibility without cross-contamination.

### Tests/CI

- [ ] Add dedicated suites under `Tests/Runtime/KhrCharacter/` and `Tests/Runtime/VisibilityHints/` for driver pipeline, response evaluation, pose validity, and per-camera visibility.
- [ ] Retain additive/override tests but label them host-adapter tests.
- [ ] Commit a Unity consumer project harness, or pin another public harness, with `org.khronos.unitygltf` in `testables`; add a workflow that opens it and runs EditMode and PlayMode tests, compilation, corpus hashes, and oracle export.
- [ ] Require named conformance suites rather than only a test-count floor.

### Acceptance

- Pure Unity outputs match the shared corpus before composition.
- Host adapter tests remain green without being treated as portable mixing semantics.
- Zero input does not overwrite a live Animator result.
- No required conformance suite is skipped.

## Phase 14 — Update the Unity sandbox/testbed

Repository: [`Kjakubzak/khr_character_testbed`](https://github.com/Kjakubzak/khr_character_testbed)

Start only after a green, immutable UnityGLTF commit exists.

### Tasks

- [ ] Pin that UnityGLTF commit in `Packages/manifest.json`, `Packages/packages-lock.json`, README, and pin-check tooling.
- [ ] Update `Assets/Editor/GenerateSampleCharacters.cs` and `Assets/Editor/SampleCharacterFactory.cs`; regenerate synthetic fixtures and migrate/re-export FromBlender assets against the frozen wire.
- [ ] Commit proposal/corpus commit IDs and asset hashes in a fixture manifest.
- [ ] Regenerate `Tests/Golden/*.json` only after reviewing semantic diffs.
- [ ] Update sampling tests to enforce the selected model rather than “at least one key.”
- [ ] Compare driver vectors and pre-mix response samples before final rendering.
- [ ] Keep composition tests explicitly Unity-specific.
- [ ] Update neutrality/XMP expectations to match proposal decisions.
- [ ] Add invalid-pose and omitted-component cases.
- [ ] Replace global renderer/material visibility assertions with per-camera draw/depth assertions.
- [ ] Require normalized structural/semantic wire equivalence in round-trip tests, including stable referenced data and preservation of unknown/custom tokens and companion extension payloads; raw JSON/GLB byte identity is not required.

### Eliminate false-green paths

- [ ] Update `.github/workflows/ci.yml` to check out the character-aware Validator fork at a SHA, install Dart, build/run the validator, and export `GLTF_VALIDATOR`; alternatively pin a published character-aware binary.
- [ ] Update `Tools/ci/validate-glb.sh` and `Tools/ci/Validate-Glb.ps1` so `--required` fails when the input directory is absent or contains zero files.
- [ ] Run a recognition-sentinel invalid fixture as a separate expected-error assertion, not in the zero-error valid-asset loop.
- [ ] Make round-trip goldens mandatory; remove “no goldens, skip.”
- [ ] Require zero skipped/inconclusive conformance tests.
- [ ] Keep URP nonblocking only during bring-up; require it before claiming cross-pipeline visibility support.

### Acceptance

- Package pins are reproducible.
- All required Built-in tests pass; URP passes before cross-pipeline claims.
- Goldens and validator reports are committed and reviewed.
- CI cannot pass by ignoring extensions or skipping fixtures.

## Phase 15 — RC-entry evidence and repository checkpoint

- [ ] For executable deterministic features selected to advance, confirm the shared corpus, Validator fork, and at least two independent pure evaluators agree. For passive annotations, require schema/Validator coverage plus independent parse/discovery evidence appropriate to that feature.
- [ ] Review each extension independently; leave extensions without frozen semantics/evidence at Draft.
- [ ] Update `extensions/README.md` and each advancing extension status truthfully.
- [ ] Synchronize once more with target `main` and rerun repository checks.
- [ ] Resolve CLA requirements and required branch checks.
- [ ] Pin proposal, corpus, Validator, Unity, and Three commits in the public review record.
- [ ] Update the PR title/body and perform a fresh final-head review.

Official Validator and Sample Renderer/Viewer integration may proceed during RC, but should normally complete before ratification.

## Phase 16 — Upstream official Validator, Sample Renderer, and Sample Viewer

### [Official glTF Validator](https://github.com/KhronosGroup/glTF-Validator)

- [ ] Rebase the green fork changes onto current official Validator `main`.
- [ ] Submit independently reviewable commits by extension family.
- [ ] Run the shared corpus and official regression suite.
- [ ] Switch testbed CI from fork SHA only after an official release produces equivalent issue codes and passes the recognition sentinel.

### [Official glTF Sample Renderer](https://github.com/KhronosGroup/glTF-Sample-Renderer)

Candidate renderer areas to verify against the target repository head include:

- `source/gltf/gltf.js`
- `source/gltf/animation.js`
- `source/gltf/interpolator.js`
- new character-expression driver/mapping/mask modules
- `source/GltfState/gltf_state.js`
- `source/GltfView/gltf_view.js`
- `source/Renderer/renderer.js`

Tasks:

- [ ] Parse capability and required-extension state.
- [ ] Add an extension to `allowedExtensions` only after the implementation satisfies that extension's minimum support contract; reject unsupported required extensions rather than listing all 13 after parsing alone.
- [ ] Add non-looping stateless global-phase sampling and do not reuse an interpolator that wraps full activation to the first key.
- [ ] Under global phase, derive `T` from original channel-referenced sampler inputs before omitting unsupported optional outputs; optional unsupported channels still affect `T`, truly unused samplers do not.
- [ ] Keep expression evaluation separate from ordinary autoplay.
- [ ] Add per-view node/primitive draw filtering.
- [ ] Add real evaluator unit tests and headless render tests.

### [Official glTF Sample Viewer](https://github.com/KhronosGroup/glTF-Sample-Viewer)

- [ ] Update the Sample Renderer submodule pin only after its evaluator/render tests pass.
- [ ] Optionally add expression sliders, mapping-set selection as host UI, reference-pose inspection, and view-context selection as demonstrations; they are not conformance gates unless the frozen support contract requires them.
- [ ] Keep tracker smoothing, automatic mapping selection, and universal mixing outside conformance behavior.

### Acceptance

- Official implementations consume the same corpus without changing expected results.
- `e=1` does not wrap and `e=0` does not overwrite a live underlay.
- Required-extension capability behavior matches the specification.

## Parallel execution after proposal freeze

- **Stream A, critical path:** expressions → mapping/masks → shared expression fixtures → validator + Three + Unity evaluators.
- **Stream B:** reference pose and skeleton mapping → validator + Three/Unity pose support.
- **Stream C:** visibility → camera/look-at → Three/Unity render adapters.
- **Cross-cutting:** schema generation, examples, links, terminology, and CI may proceed per frozen stream.

The RC transition blocks only on extensions selected to advance, not on extensions explicitly left Draft.

## Final conformance gate

An implementation claiming support for an extension must satisfy every applicable corpus, required-support, round-trip, and dependency gate for that extension. Feature-specific gates apply only where relevant: expression support covers `e=0/e=1` and repeated references; pose support covers pose oracles; visibility support covers per-view visible sets.

Proposal advancement additionally requires two independent consumers to agree on each advancing extension's deterministic primitives, with passive annotations using appropriate parse/discovery evidence rather than an invented numerical evaluator.

Across all claimed/advancing features:

- validator red/green fixtures produce the pinned expected results;
- round trips preserve normalized structural/semantic meaning, stable references, unknown/custom tokens, and companion extension payloads;
- every dependency is pinned;
- no required job, golden file, or applicable conformance fixture is skipped.

## Pre-mortem

- Global phase is selected without an asset audit and breaks one-key/per-sampler content.
- Mapping prose is reversed without renaming or migrating bytes.
- `e=0` writes the static pose and suppresses a live Animator.
- Optional typed-child support changes channel selection or duration.
- Schemas pass while procedural constraints remain untested.
- Only half of a wire rename is applied across schemas, examples, and generated code.
- Advisory features remain both requirable and ignorable.
- Complete-looking examples encode behavior absent from the specification.
- Builds are green because extensions or goldens were skipped.
- Prototype mixer output is mistaken for portable wire conformance.
