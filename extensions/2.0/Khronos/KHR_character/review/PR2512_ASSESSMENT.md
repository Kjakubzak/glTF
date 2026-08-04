# KHR Character Phase 1: pre-RC assessment (Informative)

Review snapshot: 2026-08-04
Proposal head: [`bf38201bc5f4ed5de815c79ad43f4fc281bee83f`](https://github.com/KhronosGroup/glTF/commit/bf38201bc5f4ed5de815c79ad43f4fc281bee83f)

> Non-normative reviewer assessment; not an adopted Khronos Working Group decision.

## Executive assessment

At the pinned snapshot, [PR #2512](https://github.com/KhronosGroup/glTF/pull/2512) is not ready for a synchronized 13-extension Release Candidate. The [original issue inventory](https://github.com/0b5vr/khr-character-testbed/tree/10329983d6eb160d28d985b56078ba785e744252/provisional/20260803_current_proposal_issues) correctly reaches a no-go conclusion, but it is not a correct flat list of 226 blockers.

This review's disposition is:

- **113 Valid:** an actionable specification, schema, example, support-contract, or process concern remains.
- **79 Valid but narrowed:** a factual kernel remains, but the original scope, severity, rationale, or proposed remedy was too broad.
- **34 Invalid or withdrawn:** remove these as standalone findings because they demand host policy, duplicate inherited glTF rules, or contradict the declared feature scope.

These are issue-inventory classifications, not blocker counts. The pre-RC work reduces to six semantic gates plus specification and repository stabilization:

1. Freeze one stateless expression driver-to-response contract.
2. Define expression mapping as an explicitly named stored-data operation and its boundary with masks.
3. Define fallback and capability behavior for unknown custom masks.
4. Make reference-pose storage statically and procedurally testable.
5. Replace visibility state mutation with a pure per-render-view predicate.
6. Resolve feature-specific minimum support and `extensionsRequired` behavior.

## Review boundary

Unless core glTF or an applicable extension says otherwise, glTF leaves application orchestration—source acquisition, whether and when to play an animation, optional-choice selection, transitions, and final cross-animation mixing—to the host. Stored validity, defaults, standard property/token meanings, deterministic evaluation functions, fallbacks, and required-extension behavior remain part of the asset contract.

This boundary has two consequences:

- “glTF generally avoids runtime behavior” does not excuse an incomplete stored-data function.
- A format review should not manufacture a universal animation system, retargeter, camera controller, or gaze solver when the feature does not claim one.

## Normative and non-normative sources

Under the [glTF document conventions](https://github.com/KhronosGroup/glTF/blob/2b29723d025a995971726f2989697cdc49b1222a/specification/2.0/Specification.adoc#L110-L137), substantive specification sections are normative unless their title is suffixed `(Informative)`. Notes, Implementation Notes, and Examples are always informative.

For this proposal:

- **Normative sources:** unmarked substantive sections in the advancing extension READMEs, their schemas, and inherited core or required-extension rules.
- **Non-normative evidence:** the PR body and discussion, issue inventories, supporting review documents, external working documents, prototypes, implementation code, test results, and observed sample behavior.
- **Always informative:** Notes, Implementation Notes, and Examples. A requirement written only there does not replace coherent normative text.
- **Needs explicit labeling:** background, known-implementation, and known-limitation sections intended to be advisory should be marked `(Informative)`.

Every advancing README should include common normative/informative boilerplate. Required behavior found only in discussion, an external working document, or an implementation must move into the normative README/schema.

## Repository snapshot

### Proposal

| Item | State at snapshot |
|---|---|
| PR | Open and Draft, titled `[PROPOSAL] Khronos Avatar Extensions - Phase 1` |
| Head | `bf38201bc5f4ed5de815c79ad43f4fc281bee83f` |
| Last PR update | 2026-07-30 06:56 UTC |
| Diff | 71 commits, 31 files, 4305 additions |
| PR base | `90139f8f126c222a4b684219f226f85004ad45fc` |
| Upstream `main` | `2b29723d025a995971726f2989697cdc49b1222a` |
| Base gap | 29 upstream commits |
| Generic repository test | Passed |
| `license/cla` | Pending |
| Extension status | All 13 READMEs remain Draft; no advancing registry entries |
| Proposal fixtures | No checked-in `.gltf` or `.glb` assets in the PR |

The commit status remained pending because `license/cla` was pending. Independently of that machine state, the semantic and stabilization work below remained unresolved.

### Public implementations and evidence

| Repository | Reviewed state | Evidence status |
|---|---|---|
| [`0b5vr/khr-character-testbed`](https://github.com/0b5vr/khr-character-testbed) | `9d9907caccc7ac5eb7a1af74083ce4fd35d4a4aa` | Viewer/converter builds pass at this commit, but the packages have no expression test script or shared expected-result fixture. |
| [`Kjakubzak/khr_character_testbed`](https://github.com/Kjakubzak/khr_character_testbed) | `d437472f3e8cad9a5c0ca63a6018cf8051863747` | Public nightly Built-in and Universal Render Pipeline (URP) jobs fail at the snapshot; package-pin preflight succeeds; round-trip golden tests are skipped at this commit. |
| [`Kjakubzak/UnityGLTF`](https://github.com/Kjakubzak/UnityGLTF) branch `catsg_khr_character_testbed` | `943dd12a98212a8dac8fb035e60b110836b14000` | No public check result establishes cross-runtime conformance. |
| [`Kjakubzak/glTF-Validator`](https://github.com/Kjakubzak/glTF-Validator) | `ff5f5d5e137575620d7292c225a8c4256ff5d43e` | Character-aware validation exists in the fork, but not as shared semantic proof for the unresolved model. |

Passing builds show implementation health, not agreement on the wire contract. Failing external workflows do not by themselves prove a specification defect; they show that the public evidence is not a clean, pinned conformance result.

## Original top-level gate disposition

| Original gate | Disposition | Current reading |
|---|---|---|
| `RC-01` Stable normative behavior | **Valid but narrowed** | Expression sampling, mapping/masks, pose validity, visibility, and support contracts need stable text. Universal final animation mixing, view-context activation, and host selection do not. |
| `RC-02` Stable scope | **Valid but narrowed** | One character per glTF 2.0 asset is a permissible explicit limitation. Multi-character composition and glTF 2.1 are not required here. |
| `RC-03` Stable vocabularies | **Valid but narrowed** | Standard tokens and claimed mappings need collision/version-safe identity or an explicit external-profile boundary. A universal humanoid or expression vocabulary is not mandatory. |
| `RC-04` Schema completeness | **Valid** | Structural schemas and procedural rules must match the selected semantics. |
| `RC-05` Validator support | **Timing overstated** | Official Validator integration may occur during RC. The rules and validation plan must be stable before the wire freeze. |
| `RC-06` Sample Viewer support | **Timing overstated** | Official viewer integration may occur during RC and should normally complete before ratification. |
| `RC-07` Official samples | **Valid but narrowed** | Pinned distinguishing evidence is strongly recommended; one universal official corpus is not automatically an RC-entry gate. |
| `RC-08` Independent interoperability | **Valid but narrowed** | Require agreement on deterministic primitives, not identical output for host-defined mixing, retargeting, camera adaptation, or target selection. |
| `RC-09` Registry and status | **Valid** | Advancing extensions need truthful independent statuses and registry entries. |
| `RC-10` Legal/process checks | **Valid** | The Contributor License Agreement (CLA) was pending at the snapshot. |
| `RC-11` Stable naming | **Valid** | Settle wire names and token spelling now if they will change. |
| `RC-12` Self-contained specification | **Valid** | Required behavior cannot live only in the PR body, external working documents, or implementations. |

## 2026-08-04 issue-inventory tidy-up

The public [`20260804_tidyup.md`](https://github.com/0b5vr/khr-character-testbed/blob/9d9907caccc7ac5eb7a1af74083ce4fd35d4a4aa/provisional/20260803_current_proposal_issues/20260804_tidyup.md) is useful as a discussion inventory. It is non-normative and did not itself change the proposal or implementations.

It improves the earlier inventory by reducing 226 findings to 188, separating release-process topics from its chosen scope, retaining important expression/pose/visibility/schema kernels, and rewriting `CHAR-03` into an acceptable request for an explicit one-character scope statement.

However, 29 withdrawn claims remain materially overbroad:

`GEN-14`; `CHAR-04`; `EXP-12/13/15/18`; `JNT-06`; `MOR-05/06/09`; `TEX-04/05/08`; `MASK-01/09/10`; `POSE-04/07`; `NVIS-13`; `CAM-01/03/05/09`; `LOOK-01/02/04/05/12`; `DOC-16`.

The remaining overreach includes universal expression mixing, unnecessary skin/scene ownership, retargeting, runtime state machines, context activation, camera selection/following, look-at linkage/selection/inverse kinematics (IK), and picking/ray-query behavior outside the declared visibility scope.

The tidy-up also removes or consolidates 38 original IDs without a proposal change. Four were correctly withdrawn (`CHAR-05`, `RIG-02`, `PVIS-07`, `LOOK-03`); the other 34 retain valid or narrowed material, including expression sampling (`EXP-05/06`), mapping identity (`MAP-01`), pose validation (`POSE-09/14`), rig identity/versioning (`RIG-01/08`), schemas (`SCH-02/12`), fixtures (`TEX-09`), and release/documentation work.

Removal or consolidation in a review inventory is not evidence that a proposal issue is fixed.

## Updated disposition matrix for all 226 original findings

| Group | Valid | Valid but narrowed | Invalid / withdraw |
|---|---|---|---|
| `GEN` | 01,02,04,09,10,19 | 03,05,06,07,08,11,12,13,15,16,17,18,20 | 14 |
| `CHAR` | 06,07,08,09 | 01,02 | 03,04,05 |
| `EXP` | 02,05,06,07,08,09,10,16,17,20 | 01,03,04,11,14,19 | 12,13,15,18 |
| `JNT` | 01,03,04,08 | 02,05,07 | 06 |
| `MOR` | 01,07,10 | 02,03,04,08 | 05,06,09 |
| `TEX` | 01,02,03,07,09 | 06 | 04,05,08 |
| `MAP` | 01–10 | 11,12,13,14 | — |
| `MASK` | 02,03,04,06,07,08,13,14 | 05,11,12,15 | 01,09,10 |
| `POSE` | 01,02,08,09,11,12,13,14 | 03,05,06,10 | 04,07 |
| `RIG` | 01,04,05,07,08,10,11 | 03,06,09,12,13 | 02 |
| `NVIS` | 01,02,03,05,06,10,11,12 | 04,07,08,09 | 13 |
| `PVIS` | 02,03,04,05,08 | 01,06,09 | 07 |
| `CAM` | 02,06,12,13,16,17 | 04,07,08,10,11,14,15 | 01,03,05,09 |
| `LOOK` | 07,08,09,10,11,13 | 06 | 01,02,03,04,05,12 |
| `SCH` | 02,03,04,06,07,09,12,14 | 01,05,08,10,11,13 | — |
| `NAME` | 04,05,06,07,09 | 01,02,03,08,10,11 | — |
| `DOC` | 01,02,03,04,05,08,09,11,13,14 | 06,07,10,12,15,17 | 16 |

Totals: **113 Valid, 79 Valid but narrowed, 34 Invalid/withdrawn**.

`EXP-09` is Valid rather than Narrow because the one-key fork is both a real stored-data issue and a concrete Unity/Three incompatibility.

## Complete invalid/withdrawn list

- `GEN-14`
- `CHAR-03`, `CHAR-04`, `CHAR-05`
- `EXP-12`, `EXP-13`, `EXP-15`, `EXP-18`
- `JNT-06`
- `MOR-05`, `MOR-06`, `MOR-09`
- `TEX-04`, `TEX-05`, `TEX-08`
- `MASK-01`, `MASK-09`, `MASK-10`
- `POSE-04`, `POSE-07`
- `RIG-02`
- `NVIS-13`
- `PVIS-07`
- `CAM-01`, `CAM-03`, `CAM-05`, `CAM-09`
- `LOOK-01`, `LOOK-02`, `LOOK-03`, `LOOK-04`, `LOOK-05`, `LOOK-12`
- `DOC-16`

They are withdrawn for one or more of these reasons:

- they demand host orchestration such as universal mixing, selection, or controller behavior;
- core glTF, the AOM, or a required extension already supplies the claimed missing rule;
- they impose ownership, skin, scene, or vocabulary models beyond the declared scope;
- they treat advisory camera/look-at metadata as deterministic controllers;
- they overlook already-computable standard mask behavior;
- they attribute absent claims or demand nonessential normative work.

## Corrections from expression-driver analysis

| Claim | Correct disposition |
|---|---|
| Expressions behave like independently playing clips | Incorrect framing. Each entry should be a stateless response to a current host-supplied scalar. |
| Driver acquisition must be standardized | Incorrect. Tracking, lip sync, animation graphs, user data, smoothing, prediction, and source arbitration remain host policy. |
| Three uses ordinary core one-key behavior | Incorrect. Additive conversion makes a one-key track zero/identity and inert; Unity synthesizes base-to-target behavior. |
| An absolute neutral sample makes `e=0` fully inactive | Insufficient. Holding mapping/mask results fixed, effective `e=0` must submit no property response and clear prior response influence. Outgoing masks still read the pre-mask vector. |
| Time-scrubbing options exhaust the design space | Incorrect. Fixed-target weighting is a real alternative that must be selected explicitly or deferred. |
| Global phase is already the proposal meaning | Incorrect. It is a preferred common-animation-timeline design, not settled current semantics. A complete per-sampler model is also RC-capable if chosen exclusively and migration-tested. |
| Typed children obviously classify or filter | Unsettled. A whole-animation package can make them classifiers; the recorded PR intent is closer to filtering, with required-extension and parent-only-example consequences. |
| Another glTF animation can portably drive expression values | Not currently. The schema has no mutable expression-driver Asset Object Model (AOM) property. |
| Mask behavior is VRM-compatible | Too broad. The proposal multiplies factors; VRM sums and saturates override weights and defines binary interactions. |

## Remaining concerns and required fixes

### 1. Expression driver-to-response contract

Define a conceptual externally supplied driver per expression entry: default, finite-value handling, clamping, and history-independent evaluation. Choose exactly one coefficient role, time rule, sample interpretation, and channel-ownership rule.

Viable packages include:

- one animation-wide phase with ordinary absolute core samples and explicit inactive/active keys;
- fully specified per-sampler transfer curves with exact typed filtering and one-key rules;
- a fixed-target coefficient-as-weight model with immutable base/target and domain-specific contribution math.

Global phase is a design preference, not a core requirement. Whichever model is selected must define `e=0`, inactive reference, one-key validity, typed/untyped behavior, required children, repeated-reference identity, ordinary playback independence, and whether portable in-file driver wiring is out of scope or backed by an AOM/order/cycle contract.

### 2. Mapping and mask boundary

The current mapping text describes native source→endpoint target contribution. Unity and the playback-oriented recommendation use endpoint command→native driver distribution. These are neither identical nor inverses.

Preserve and name the forward operation, deliberately migrate the existing meaning, or add a separately named reverse input-mapping operation. Define domains, repeated sums, unmapped values, final clamp, direct-versus-mapped boundary, mapping before the immutable pre-mask snapshot, and versioned/collision-safe mapping-set identity or an external-profile boundary.

### 3. Unknown custom masks

Standard masks, self-masks, duplicates, and cycles are already computable from the immutable snapshot and product equations. For unknown custom types:

- preserve the token;
- define a deterministic evaluator fallback;
- do not silently reinterpret it as `blend`;
- use vendor-qualified tokens and a companion vendor extension on the same mask object for essential portable behavior;
- state that numeric identity fallback is deterministic but may not preserve author intent;
- narrow the VRM-compatibility claim or publish an exact conversion profile.

### 4. Reference poses

Require a nonempty tagged animation with ordinary node translation/rotation/scale (TRS) channels only, exactly one input key for each referenced sampler, valid core cardinality and all other core animation rules, and static asset/default values for omitted components. Core `CUBICSPLINE` requires at least two keys and is incompatible with the one-key pose contract.

Treat T/A/I labels as author assertions; do not require a skin, exact anatomy, complete coverage, `t=0`, `STEP`, unique pose types, or a retargeter. Remove or repair incomplete inverse-bind migration and compatibility claims.

### 5. Visibility

Use a pure per-render-view predicate rather than mutating `KHR_node_visibility` or engine-global state:

```text
hintRole(n) = nearest hint role on n or an ancestor; otherwise always
hintVisible(n, C) = standard role predicate for supplied context C
declaredVisible(n) = AND of KHR_node_visibility.visible (default true) on n and ancestors
renderVisualContent(n, C) = hintVisible(n, C) AND declaredVisible(n)

renderPrimitiveInstance(n, p, C) =
    renderVisualContent(n, C) AND primitiveHintVisible(p, C)
```

A descendant hint overrides inherited hint metadata but cannot defeat an explicitly hidden `KHR_node_visibility` ancestor. Evaluate shared meshes per node instance. Define role truth tables and unknown fallback. Context choice/transitions remain host policy. Transparent-material substitution is best-effort, not conforming suppression in every visual pass.

### 6. Required support

- **Typed expressions:** filtered children essential to acceptable output must be required; optional children explicitly permit partial/inert output. Classifier-only children do not alter parent execution.
- **Visibility:** may be required only with MUST-level standard-role behavior and real visual suppression for required primitive support.
- **Camera hints:** keep advisory and prohibit use in `extensionsRequired` unless a stronger profile is added.
- **Look-at markers:** required use is coherent only if implementations MUST recognize markers and interpret each annotated node's per-instance global translation as the target point. Selection, API shape, priority, constraints, IK, and gaze remain host policy.

### Additional camera clarifications

- Derive a selected hint pose from the node global transform using core-camera scale conventions.
- If `targetNode` is coincident, fall back to authored rotation or make the target direction unusable.
- Treat `framing_target` as a point of interest unless bounds, margins, aspect, and framing inputs are added.

Selection, following, adaptation, roll policy, and transitions remain host policy while camera hints are advisory.

## Other freeze-time work

- State one-character and `rootNode` scope truthfully.
- Choose generic skeleton vocabulary-role→node association or enforce skin-joint membership; do not imply an unspecified retargeter.
- Define version/collision-safe vocabulary identity or an external-profile boundary.
- Remove the unconditional XMP dependency when no XMP data exists, or justify it.
- Require nonempty labels/custom identifiers and make indices authoritative if labels duplicate.
- Settle wire names and standard token spelling.
- Align schemas with the selected semantics and add procedural diagnostics.
- Repair malformed examples or mark incomplete snippets as partial.
- Remove examples and claims that infer tours, network behavior, framing algorithms, VRM gaze/IK, or identical runtime output not encoded by the data.
- Synchronize with target `main` at execution time, update statuses/registry, refresh the PR body, and resolve the CLA.

## Before RC, during RC, and out of scope

### Before entering RC

- Freeze validity, standard meanings, fallbacks, and support contracts for each advancing extension.
- Select the expression and mapping models and run a pinned migration audit before final wire freeze.
- Resolve mask fallback, pose validity, pure visibility, and required support.
- Align normative text, schemas, examples, names, identity, and dependencies.
- Synchronize with target `main`, rerun checks, assign independent statuses, update registry entries, and resolve the CLA.
- Pin reviewed assets/implementations and publish distinguishing evidence for the wire choices.

### May occur during RC

- official Khronos Sample Viewer integration;
- official glTF Validator integration;
- public final review;
- broader shared fixtures and two-runtime coverage;
- final non-wire-destabilizing corrections.

### Remain host policy unless separately standardized

- tracking/audio/user-data acquisition, smoothing, prediction, update cadence, and source arbitration;
- final nonzero cross-animation composition;
- mapping-set selection and direct/mapped source arbitration;
- retargeting, exact anatomy, full joint coverage, and universal skin ownership;
- view-context selection/transitions;
- camera selection/following/adaptation/transitions;
- look-at selection, priority, links, constraints, IK, and gaze;
- pixel-identical advisory output;
- portable in-file expression-driver animation if Phase 1 explicitly excludes it.

## Confidence and proof

Confidence is high on the six semantic gates, normative/source-of-truth defects, schema/example work, and the 34 withdrawals. Confidence is medium on the chosen expression model, mapping operation/migration, vocabulary mechanism, custom-mask fallback, and asset migration cost.

Agreement among reviewers or green self-tests does not prove the selected remedies. Proof requires revised normative text, red/green validator cases, hand-authored distinguishing fixtures, pinned migration results, and independent implementations agreeing on deterministic pre-mix operations.

## Public sources

- [PR #2512](https://github.com/KhronosGroup/glTF/pull/2512)
- [Original 226-item inventory](https://github.com/0b5vr/khr-character-testbed/tree/10329983d6eb160d28d985b56078ba785e744252/provisional/20260803_current_proposal_issues)
- [188-item tidy-up](https://github.com/0b5vr/khr-character-testbed/blob/9d9907caccc7ac5eb7a1af74083ce4fd35d4a4aa/provisional/20260803_current_proposal_issues/20260804_tidyup.md)
- [Core animation policy](https://github.com/KhronosGroup/glTF/blob/2b29723d025a995971726f2989697cdc49b1222a/specification/2.0/Specification.adoc#L2410-L2419) and [sampling](https://github.com/KhronosGroup/glTF/blob/2b29723d025a995971726f2989697cdc49b1222a/specification/2.0/Specification.adoc#L2565-L2589)
- [`KHR_animation_pointer`](https://github.com/KhronosGroup/glTF/blob/2b29723d025a995971726f2989697cdc49b1222a/extensions/2.0/Khronos/KHR_animation_pointer/README.md#L29-L43)
- [`KHR_interactivity` animation common-time precedent](https://github.com/KhronosGroup/glTF/blob/2b29723d025a995971726f2989697cdc49b1222a/extensions/2.0/Khronos/KHR_interactivity/Specification.adoc#L4284-L4306)
- [`KHR_node_visibility`](https://github.com/KhronosGroup/glTF/blob/2b29723d025a995971726f2989697cdc49b1222a/extensions/2.0/Khronos/KHR_node_visibility/README.md)
- [VRM expressions](https://github.com/vrm-c/vrm-specification/blob/70ae16e93abd6da727fdf641b67aa41010c6d933/specification/VRMC_vrm-1.0/expressions.md#L131-L280)
- [Three implementation](https://github.com/0b5vr/khr-character-testbed/commit/10329983d6eb160d28d985b56078ba785e744252)
- [UnityGLTF implementation](https://github.com/Kjakubzak/UnityGLTF/commit/943dd12a98212a8dac8fb035e60b110836b14000)
- [Unity testbed](https://github.com/Kjakubzak/khr_character_testbed/commit/d437472f3e8cad9a5c0ca63a6018cf8051863747)
- [Validator fork](https://github.com/Kjakubzak/glTF-Validator/commit/ff5f5d5e137575620d7292c225a8c4256ff5d43e)

This assessment proposes no normative or implementation changes by itself. The [companion execution plan](./PR2512_EXECUTION_PLAN.md) defines the work sequence.
