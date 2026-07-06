# Guide: Animating a Full Character with Multiple Animations

**Status:** Draft companion guide (non-normative) for the KHR animation & character extension set.

This guide shows how the KHR animation and character extensions **compose** to animate a complete
character — locomotion, additive motion, masked gestures, facial expressions, runtime transitions, and event
sync — at the same time. Each extension is a building block; this document is the "how they fit together"
overview that no single extension README provides.

It is **non-normative**: every rule lives in the individual extension specs, which are linked throughout.

## The two pipelines

A character has two animation domains that composite independently and then combine on the final model:

1. **Body / skeletal animation** — locomotion, gestures, additive motion on the rig's joints.
   Composited by **`KHR_animation_layers`** using per-animation blend semantics from **`KHR_animation_type`**.
2. **Facial expressions** — `blink`, `smile`, `jawOpen`, visemes, etc.
   Described by **`KHR_character_expression`** (+ friends) and composited by the expression runtime, with
   conflicts resolved by **`KHR_character_expression_mask`**.

Runtime control (start/stop/crossfade/weights) for both is driven by **`KHR_interactivity_animation`**, and
timing is aligned with **`KHR_animation_events`** (sync markers + event callbacks).

```text
                    ┌─────────────────────────── final character pose ───────────────────────────┐
                    │                                                                              │
  BODY:   layer stack (KHR_animation_layers)                 FACE: expressions (KHR_character_expression)
          ├─ layer 0  base locomotion  (override)                  ├─ blink   (weight 0..1)
          ├─ layer 1  upper-body gesture (masked)                  ├─ smile   (weight 0..1)
          └─ layer 2  additive breathing (additive)               └─ …  conflicts → KHR_character_expression_mask
                    │                                                                              │
          per-animation type = KHR_animation_type                 root marker = KHR_character
                    │                                                                              │
          RUNTIME (KHR_interactivity_animation): crossfade, layer/setWeight, setAnimationWeight, setMask
          TIMING   (KHR_animation_events): sync markers align footfalls; event callbacks fire on cues
```

## Building blocks

| Extension | Role in a multi-animation character |
|---|---|
| `KHR_character` | Root marker: "this asset is a character" (references the character root node). Signals consumers to run character pipelines. |
| `KHR_animation_layers` | The ordered **layer stack** — how multiple body animations blend, prioritize, and mask onto the skeleton. |
| `KHR_animation_type` | Per-animation **blend semantics**: `override` (replace) vs `additive` (delta on top). Makes layer blending predictable. |
| `KHR_character_expression` | Descriptive mapping of facial **expressions → animations** (`blink`, `smile`, `jawOpen`). Does not store data itself. |
| `KHR_character_expression_mask` | How one expression **blends/blocks** another (conflict resolution among expressions). |
| `KHR_interactivity_animation` | **Runtime control** via behavior graphs: play/pause, crossfade, and layer/animation weight + mask changes. |
| `KHR_animation_events` | **Sync markers** to align animations (e.g. footfalls) and **event callbacks** on animation cues. |

## How a frame composites

Per frame, the runtime evaluates **body** and **face** independently, then applies both to the model:

1. **Runtime step (interactivity).** Apply this frame's weight/mask/crossfade changes *before* evaluation —
   `KHR_interactivity_animation` guarantees "no 1-frame lag," so `layer/setWeight` etc. take effect this frame.
2. **Body: evaluate the layer stack** (`KHR_animation_layers`), base layer (index 0) → top:
   - Within each layer, blend its animations by their `KHR_animation_type` (override animations replace;
     additive animations add deltas).
   - Apply the layer's **skeletal mask** (per-joint weights) and the **layer weight**, then composite onto the
     accumulated skeletal pose.
3. **Face: evaluate expressions** (`KHR_character_expression`). Each active expression drives its target(s) by a
   0→1 weight; overlapping expressions resolve via `KHR_character_expression_mask`.
4. **Combine.** Body drives skeletal joints; expressions drive face morphs/targets. They occupy different
   channels, so they coexist — *except* where an expression also drives a body joint (e.g. a jaw bone), which is
   exactly what skeletal masks and expression masks are for.

> **Scope rule.** Keep body/locomotion in the animation-layer stack and facial expressions in the expression
> system. They are parallel systems that combine on the final character — not a single flat list of clips.

## Worked example

A character that **walks**, **breathes** (additive), **waves** with only its upper body, **smiles/blinks**, and
at runtime **crossfades walk → run** while firing **footstep events**.

### 1. Classify each body animation (`KHR_animation_type`)

Tag additive clips; everything else defaults to `override`.

```json
{
  "animations": [
    { "name": "Idle", "channels": [], "samplers": [] },
    { "name": "Walk", "channels": [], "samplers": [] },
    { "name": "Run",  "channels": [], "samplers": [] },
    { "name": "WaveHand", "channels": [], "samplers": [] },
    {
      "name": "AdditiveBreathing",
      "extensions": { "KHR_animation_type": { "type": "additive" } },
      "channels": [], "samplers": []
    }
  ]
}
```

### 2. Build the body layer stack (`KHR_animation_layers`)

A base locomotion layer (Idle↔Walk blend), an **upper-body-masked** gesture layer, and an **additive** breathing
layer. The mask uses the define-then-reference pattern (`nodes` + per-joint `weights` + `includeDescendants`).

```json
{
  "extensionsUsed": ["KHR_animation_layers", "KHR_animation_type"],
  "extensions": {
    "KHR_animation_layers": {
      "masks": [
        { "name": "UpperBody", "nodes": [1, 2, 3], "weights": [0.5, 1.0, 1.0], "includeDescendants": true }
      ],
      "layers": [
        {
          "name": "BaseLocomotion",
          "weight": 1.0,
          "animations": [
            { "animation": 0, "weight": 0.3 },
            { "animation": 1, "weight": 0.7 }
          ]
        },
        {
          "name": "UpperBodyGesture",
          "weight": 1.0,
          "mask": 0,
          "animations": [ { "animation": 3 } ]
        },
        {
          "name": "Breathing",
          "weight": 0.4,
          "animations": [ { "animation": 4 } ]
        }
      ]
    }
  }
}
```

Result: the legs/hips walk (base), the upper body waves (masked so it doesn't disturb the legs), and breathing
is layered additively on top at 40% — three body animations at once.

### 3. Add facial expressions (`KHR_character` + `KHR_character_expression`)

Mark the asset as a character and map expression semantics onto animations. Expression compositing (and any
conflicts, e.g. `smile` vs `jawOpen`) is handled by the expression system / `KHR_character_expression_mask`,
independently of the body layer stack.

```json
{
  "extensionsUsed": ["KHR_character", "KHR_character_expression"],
  "extensions": {
    "KHR_character": { "node": 0 }
  }
}
```

### 4. Drive it at runtime (`KHR_interactivity_animation`)

Crossfade walk → run over 0.3 s of wall-clock time, and blend the gesture layer's weight from a variable:

```json
{
  "nodes": [
    { "type": "event/onCustom", "configuration": { "id": "startRunning" } },
    {
      "type": "animation/crossfade",
      "configuration": { "fromAnimation": 1, "toAnimation": 2 },
      "inputValues": { "duration": { "value": 0.3 }, "toLoop": { "value": true } }
    },
    {
      "type": "layer/setWeight",
      "configuration": { "layer": 1 },
      "inputValues": { "weight": { "node": 3, "socket": "result" } }
    }
  ]
}
```

Use `layer/setAnimationWeight` and `layer/setMask` for finer control, and `animation/pause` / `resume` /
`setSpeed` / `setTime` for playback. Expression weights are driven the same way (via the expression runtime).

### 5. Keep everything in sync (`KHR_animation_events`)

Add sync markers so blended locomotion clips (Walk/Run) align footfalls, and event callbacks for gameplay cues:

- **Sync markers** — align `footL` / `footR` across Walk and Run so a crossfade doesn't slide the feet.
- **`event/onAnimationEvent`** / **`event/onSyncMarker`** / **`event/onAnimationComplete`** — fire footstep
  SFX, spawn effects, or chain the next animation.

## Authoring checklist

- [ ] Tag additive clips with `KHR_animation_type: { "type": "additive" }`; leave replace-clips as default `override`.
- [ ] Order layers base → top; the base layer should be a full-body `override` locomotion layer.
- [ ] Use skeletal **masks** for partial-body layers (gestures, aim) so they don't fight locomotion.
- [ ] Keep **facial expressions** in `KHR_character_expression`, **not** as extra body layers; resolve expression
      overlaps with `KHR_character_expression_mask`.
- [ ] Add **sync markers** to any clips you'll crossfade/blend (locomotion especially).
- [ ] Drive weights/crossfades/masks through `KHR_interactivity_animation` (weight changes are same-frame).
- [ ] Respect each extension's **Limits / Numeric Safety / Shortest-Path Slerp** (see `KHR_animation_layers` →
      "Safety & Conformance").

## Reference

- Layer stack, masks, blending, evaluation pipeline → [`KHR_animation_layers`](./README.md)
- Additive vs override math → [`../KHR_animation_type/README.md`](../KHR_animation_type/README.md)
- Runtime control nodes → [`../KHR_interactivity_animation/README.md`](../KHR_interactivity_animation/README.md)
- Sync markers & event callbacks → [`../KHR_animation_events/README.md`](../KHR_animation_events/README.md)
- Facial expressions → [`../KHR_character_expression/README.md`](../KHR_character_expression/README.md),
  [`../KHR_character_expression_mask/README.md`](../KHR_character_expression_mask/README.md)
- Character root marker → [`../KHR_character/README.md`](../KHR_character/README.md)

## Contributors

- Ken Jakubzak, Meta

## License

Licensed under the Khronos [specification license](https://www.khronos.org/registry/gltf/specs/2.0/specification-license.txt).
