# KHR_interactivity_animation

## Contributors

- Ken Jakubzak, Meta

## Status

Draft

## Dependencies

Written against glTF 2.0 specification.

**Required Extensions:**

- KHR_interactivity

**Optional Extensions:**

- KHR_animation_layers
- KHR_animation_events
- KHR_animation_type

## Overview

This extension defines animation control nodes as a **companion extension to KHR_interactivity**, following the pattern established by KHR_node_visibility. The extension enables runtime animation control (start, stop, crossfade, layer manipulation) through behavior graphs.

### Design Principles

1. **Complement, Don't Duplicate**: Uses existing `animation/start`, `animation/stop`, and `animation/stopAt` nodes from KHR_interactivity core
2. **playbackId Handle Pattern**: Animation instances are managed through integer handles returned by start operations
3. **Explicit Targeting**: All nodes use explicit parameter names (`gltfAnimationIndex`, `layerIndex`) to avoid ambiguity
4. **Wall-Clock Crossfade**: Crossfade durations are in real seconds, not animation time
5. **No 1-Frame Lag**: Weight changes take effect before layer evaluation in the same frame

## Extension Declaration

```json
{
  "extensionsUsed": ["KHR_interactivity", "KHR_interactivity_animation"],
  "extensionsRequired": ["KHR_interactivity"]
}
```

## Node Categories

This extension defines **15 new nodes** across 5 categories:

| Category | Count | Nodes |
|----------|-------|-------|
| Playback Control | 4 | `animation/pause`, `animation/resume`, `animation/setSpeed`, `animation/setTime` |
| Crossfade | 1 | `animation/crossfade` |
| Query | 3 | `animation/getDuration`, `animation/getTime`, `animation/isPlaying` |
| Layer Control | 4 | `layer/setWeight`, `layer/setAnimationWeight`, `layer/setMask`, `layer/getWeight` |
| Event Listeners | 3 | `event/onAnimationComplete`, `event/onAnimationEvent`, `event/onSyncMarker` |

**Note:** `animation/start`, `animation/stop`, and `animation/stopAt` already exist in KHR_interactivity core (Section 4.14.3).

## Playback Control Nodes

### animation/pause

Pauses a running animation, preserving current time.

| Property | Type | Description |
|----------|------|-------------|
| **Input Flows** | | |
| `in` | flow | Trigger to pause |
| **Output Flows** | | |
| `out` | flow | Fires on success |
| `err` | flow | Fires if playbackId is invalid or already paused |
| **Input Values** | | |
| `playbackId` | int | Handle from animation/start |

**Behavior:**

- Paused animations retain their position in the animation state array
- Takes effect at frame boundary, not mid-evaluation

### animation/resume

Resumes a paused animation from its current time.

| Property | Type | Description |
|----------|------|-------------|
| **Input Flows** | | |
| `in` | flow | Trigger to resume |
| **Output Flows** | | |
| `out` | flow | Fires on success |
| `err` | flow | Fires if playbackId is invalid or not paused |
| **Input Values** | | |
| `playbackId` | int | Handle from animation/start |

### animation/setSpeed

Changes playback speed of a running animation.

| Property | Type | Description |
|----------|------|-------------|
| **Input Flows** | | |
| `in` | flow | Trigger to set speed |
| **Output Flows** | | |
| `out` | flow | Fires on success |
| `err` | flow | Fires if playbackId is invalid |
| **Input Values** | | |
| `playbackId` | int | Handle from animation/start |
| `speed` | float | Speed multiplier (negative reverses playback) |

**Behavior with KHR_animation_events:**

- Speed sign change triggers event ordering reversal per KHR_animation_events backward playback rules
- Events do NOT re-fire when crossing the same time twice during speed change

### animation/setTime

Seeks animation to specific time (scrubbing).

| Property | Type | Description |
|----------|------|-------------|
| **Input Flows** | | |
| `in` | flow | Trigger to seek |
| **Output Flows** | | |
| `out` | flow | Fires on success |
| `err` | flow | Fires if playbackId is invalid |
| **Input Values** | | |
| `playbackId` | int | Handle from animation/start |
| `time` | float | Time in seconds |

**Behavior:**

- Values outside [0, duration] are clamped
- Events between old and new time do NOT fire (seek, not playback)
- Sync markers are NOT triggered by setTime

## Crossfade Node

### animation/crossfade

Blends from one animation to another over time.

| Property | Type | Description |
|----------|------|-------------|
| **Configuration** | | |
| `fromAnimation` | int | glTF animation index to fade FROM |
| `toAnimation` | int | glTF animation index to fade TO |
| **Input Flows** | | |
| `in` | flow | Trigger crossfade |
| **Output Flows** | | |
| `out` | flow | Fires immediately when crossfade starts |
| `done` | flow | Fires when crossfade completes |
| `err` | flow | Fires on error (e.g., incompatible masks) |
| **Input Values** | | |
| `duration` | float | Crossfade duration in WALL-CLOCK seconds (default: 0.3) |
| `toStartTime` | float | Start time for target animation (default: 0.0) |
| `toSpeed` | float | Speed for target animation (default: 1.0) |
| `toLoop` | bool | Whether target animation loops (default: false) |
| **Output Values** | | |
| `playbackId` | int | Handle for the TO animation |

**Critical Design Decisions:**

1. **Uses animation indices, NOT playbackIds for source**
   - Rationale: playbackIds can become invalid; animation indices are stable
   - `fromAnimation` refers to the glTF `animations[]` index
   - The engine finds the active playback of that animation to fade from

2. **Duration is wall-clock time, NOT animation time**
   - 0.5s crossfade = 0.5 real seconds regardless of animation speed
   - Prevents instant crossfades when speed=0

3. **Crossfade Interruption Policy:**
   - Starting a new crossfade during an active crossfade:
     - Captures current blended state as new source
     - Cancels previous crossfade
     - Previous `done` flow does NOT fire

4. **Mask Handling:**
   - If fromAnimation and toAnimation have different masks:
     - Masks are interpolated during crossfade
     - Unmapped bones use bind pose
   - If masks are incompatible, `err` fires

## Query Nodes

### animation/getDuration

Gets the duration of an animation in seconds.

| Property | Type | Description |
|----------|------|-------------|
| **Configuration** | | |
| `animation` | int | glTF animation index |
| **Input Flows** | | |
| `in` | flow | Trigger query |
| **Output Flows** | | |
| `out` | flow | Fires with result |
| **Output Values** | | |
| `duration` | float | Duration in seconds |

### animation/getTime

Gets the current playback time of an animation instance.

| Property | Type | Description |
|----------|------|-------------|
| **Input Flows** | | |
| `in` | flow | Trigger query |
| **Output Flows** | | |
| `out` | flow | Fires with result |
| `err` | flow | Fires if playbackId is invalid |
| **Input Values** | | |
| `playbackId` | int | Handle from animation/start |
| **Output Values** | | |
| `time` | float | Current time in seconds |
| `normalizedTime` | float | Current time as [0,1] phase |

### animation/isPlaying

Queries whether an animation instance is currently playing.

| Property | Type | Description |
|----------|------|-------------|
| **Input Flows** | | |
| `in` | flow | Trigger query |
| **Output Flows** | | |
| `out` | flow | Fires with result |
| **Input Values** | | |
| `playbackId` | int | Handle from animation/start |
| **Output Values** | | |
| `playing` | bool | True if animation is playing (not stopped) |
| `paused` | bool | True if animation is paused |

## Layer Control Nodes

These nodes require the **KHR_animation_layers** extension.

### layer/setWeight

Sets the blend weight of an animation layer.

| Property | Type | Description |
|----------|------|-------------|
| **Configuration** | | |
| `layer` | int | Index into KHR_animation_layers/layers[] |
| **Input Flows** | | |
| `in` | flow | Trigger weight change |
| **Output Flows** | | |
| `out` | flow | Fires on success |
| `err` | flow | Fires if layer index is invalid |
| **Input Values** | | |
| `weight` | float | Blend weight [0,1] |

**Evaluation Order:** Weight changes take effect BEFORE layer evaluation in the same frame.

### layer/setAnimationWeight

Sets weight of a specific animation reference within a layer.

| Property | Type | Description |
|----------|------|-------------|
| **Configuration** | | |
| `layer` | int | Index into KHR_animation_layers/layers[] |
| `animationRef` | int | Index into layer.animations[] |
| **Input Flows** | | |
| `in` | flow | Trigger weight change |
| **Output Flows** | | |
| `out` | flow | Fires on success |
| `err` | flow | Fires if indices are invalid |
| **Input Values** | | |
| `weight` | float | Animation reference weight |

### layer/setMask

Dynamically changes the mask applied to a layer.

| Property | Type | Description |
|----------|------|-------------|
| **Configuration** | | |
| `layer` | int | Index into KHR_animation_layers/layers[] |
| **Input Flows** | | |
| `in` | flow | Trigger mask change |
| **Output Flows** | | |
| `out` | flow | Fires on success |
| `err` | flow | Fires if indices are invalid |
| **Input Values** | | |
| `mask` | int | Index into KHR_animation_layers/masks[], or -1 for no mask |

**Behavior:**

- Mask changes take effect on next evaluation frame
- No blend-out of previous mask (instant switch)

### layer/getWeight

Gets the current blend weight of an animation layer.

| Property | Type | Description |
|----------|------|-------------|
| **Configuration** | | |
| `layer` | int | Index into KHR_animation_layers/layers[] |
| **Input Flows** | | |
| `in` | flow | Trigger query |
| **Output Flows** | | |
| `out` | flow | Fires with result |
| `err` | flow | Fires if layer index is invalid |
| **Output Values** | | |
| `weight` | float | Current layer weight |

## Event Listener Nodes

These nodes integrate with **KHR_animation_events** when available.

### event/onAnimationComplete

Fires when an animation reaches its end (non-looping) or completes a loop iteration.

| Property | Type | Description |
|----------|------|-------------|
| **Configuration** | | |
| `animation` | int | glTF animation index to monitor |
| **Output Flows** | | |
| `out` | flow | Fires on completion |
| **Output Values** | | |
| `playbackId` | int | Handle of the completed animation |
| `loopCount` | int | Number of completed loops (0 for non-looping) |

**Timing:** Fires AFTER final frame evaluation, BEFORE loop resets time.

### event/onAnimationEvent

Fires when a KHR_animation_events event is triggered.

| Property | Type | Description |
|----------|------|-------------|
| **Configuration** | | |
| `animation` | int | glTF animation index to monitor |
| `eventName` | string | Filter by event name, or empty for all events |
| **Output Flows** | | |
| `out` | flow | Fires when event triggers |
| **Output Values** | | |
| `name` | string | Event name |
| `time` | float | Animation local time |
| `globalTime` | float | Scene time (accounts for start offset, speed) |
| `payload` | any | Event payload data |

**Time Domains:**

- `time` = animation local time (as defined in KHR_animation_events)
- `globalTime` = scene time

### event/onSyncMarker

Fires when a sync marker is reached (KHR_animation_events with `isSync: true`).

| Property | Type | Description |
|----------|------|-------------|
| **Configuration** | | |
| `animation` | int | glTF animation index to monitor |
| **Output Flows** | | |
| `out` | flow | Fires when sync marker is reached |
| **Output Values** | | |
| `name` | string | Sync marker name |
| `phase` | float | Normalized [0,1] position in animation |

**Note:** This is syntactic sugar for `event/onAnimationEvent` filtered to `isSync: true` events.

## Evaluation Pipeline

The execution order within a single frame MUST be:

```text
1. Interactivity graph executes
   └── Weight-setting nodes update layer weights
   └── Animation control nodes update playback state
2. Animation sampling pass
   └── KHR_animation_layers evaluates with NEW weights
   └── Animations sampled at current time
3. Event firing pass
   └── KHR_animation_events checks for crossed events
   └── Event nodes fire their output flows
4. Render
```

**Critical Guarantee:** Layer weight changes from step 1 MUST be visible in step 2. No 1-frame lag.

## Conformance Levels

### Level 1: Basic Playback (MUST)

Implementations MUST support:

- `animation/pause`
- `animation/resume`
- `animation/setSpeed`
- `animation/getDuration`
- `animation/getTime`
- `animation/isPlaying`

### Level 2: Full Control (SHOULD)

Implementations SHOULD support all Level 1 nodes plus:

- `animation/setTime`
- `animation/crossfade`
- `layer/setWeight`
- `layer/getWeight`

### Level 3: Complete (MAY)

Implementations MAY support all Level 2 nodes plus:

- `layer/setAnimationWeight`
- `layer/setMask`
- `event/onAnimationComplete`
- `event/onAnimationEvent`
- `event/onSyncMarker`

## Object Model Integration

This extension relies on the following Object Model pointers (to be added to ObjectModel.adoc):

### Animation Runtime State

| Pointer | Type | Description |
|---------|------|-------------|
| `/animations/{}/time` | float | Current playback time (seconds) |
| `/animations/{}/speed` | float | Playback speed multiplier |
| `/animations/{}/playing` | bool | Is animation currently playing |

### KHR_animation_layers Pointers

| Pointer | Type | Description |
|---------|------|-------------|
| `/extensions/KHR_animation_layers/layers/{}/weight` | float | Layer blend weight |
| `/extensions/KHR_animation_layers/layers/{}/animations/{}/weight` | float | Animation reference weight |
| `/extensions/KHR_animation_layers/layers/{}/mask` | int | Mask index |
| `/extensions/KHR_animation_layers/layers.length` | int | Number of layers (read-only) |
| `/extensions/KHR_animation_layers/masks.length` | int | Number of masks (read-only) |

## Example Usage

### Basic Play/Pause Toggle

```json
{
  "nodes": [
    {
      "type": "event/onSelect",
      "configuration": { "node": 0 }
    },
    {
      "type": "flow/branch",
      "inputValues": {
        "condition": { "node": 3, "socket": "playing" }
      }
    },
    {
      "type": "animation/pause",
      "inputValues": { "playbackId": { "node": 4, "socket": "playbackId" } }
    },
    {
      "type": "animation/isPlaying",
      "inputValues": { "playbackId": { "node": 4, "socket": "playbackId" } }
    },
    {
      "type": "animation/start",
      "configuration": { "animation": 0 }
    }
  ]
}
```

### Crossfade Between Walk and Run

```json
{
  "nodes": [
    {
      "type": "event/onCustom",
      "configuration": { "id": "startRunning" }
    },
    {
      "type": "animation/crossfade",
      "configuration": {
        "fromAnimation": 0,
        "toAnimation": 1
      },
      "inputValues": {
        "duration": { "value": 0.3 },
        "toLoop": { "value": true }
      }
    }
  ]
}
```

### Layer Weight Blending

```json
{
  "nodes": [
    {
      "type": "math/lerp",
      "inputValues": {
        "a": { "value": 0.0 },
        "b": { "value": 1.0 },
        "t": { "node": 2, "socket": "value" }
      }
    },
    {
      "type": "layer/setWeight",
      "configuration": { "layer": 1 },
      "inputValues": {
        "weight": { "node": 0, "socket": "result" }
      }
    },
    {
      "type": "variable/get",
      "configuration": { "variable": "upperBodyBlend" }
    }
  ]
}
```

## Schema

See the `schema/` directory for JSON schemas defining each node type.

## Known Limitations

1. **Unity Mecanim**: Runtime mask changes may not be supported due to compiled state machine architecture
2. **Crossfade with Different Masks**: Requires mask interpolation which may not be available in all engines
3. **Event Timing**: Sub-frame event precision depends on engine tick rate

## Future Extensions (Not in V1)

The following features are deferred to future versions:

- Blend spaces / 2D blend trees
- State machines
- Sync groups
- Root motion extraction
- Inverse kinematics integration
- Animation retargeting at runtime

## Reference

- [KHR_interactivity](../KHR_interactivity/README.md) - Core behavior graph specification
- [KHR_animation_layers](../KHR_animation_layers/README.md) - Animation layering system
- [KHR_animation_events](../KHR_animation_events/README.md) - Animation events and sync markers
- [KHR_animation_type](../KHR_animation_type/README.md) - Override/additive animation types
