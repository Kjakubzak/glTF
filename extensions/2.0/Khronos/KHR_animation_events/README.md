# KHR_animation_events

## Contributors

- Ken Jakubzak, Meta

## Status

**Draft** – This extension is not yet ratified by the Khronos Group and is subject to change.

## Dependencies

Written against the glTF 2.0 specification.

## Overview

The `KHR_animation_events` extension defines **timestamped events** and **sync markers** attached to glTF animations. Events are triggered when animation playback crosses specific time points, enabling applications to respond with gameplay logic, audio cues, visual effects, or other behaviors.

Key features:

- **Events**: Named markers at specific times that fire during playback
- **Sync Markers**: Special events that enable animation alignment across layers
- **Payloads**: Optional JSON data attached to events for application-specific parameters

## Extension Schema

This extension is applied to individual animations in the glTF file.

**Schema**: [animation.KHR_animation_events.schema.json](schema/animation.KHR_animation_events.schema.json)

```json
{
  "animations": [
    {
      "name": "Walk",
      "extensions": {
        "KHR_animation_events": {
          "events": [
            {
              "time": 0.2,
              "name": "FootstepLeft"
            },
            {
              "time": 0.7,
              "name": "FootstepRight"
            },
            {
              "time": 0.0,
              "name": "LoopStart",
              "isSync": true
            },
            {
              "time": 1.0,
              "name": "LoopEnd",
              "isSync": true
            }
          ]
        }
      },
      "channels": [...],
      "samplers": [...]
    }
  ]
}
```

### Extension Properties

| Property | Type   | Required | Description |
|----------|--------|----------|-------------|
| `events` | array  | Yes      | Array of event objects attached to this animation. |

### Event Object Properties

| Property  | Type    | Required | Default | Description |
|-----------|---------|----------|---------|-------------|
| `time`    | number  | Yes      |         | Time in seconds when the event fires. Must be ≥ 0 and not NaN or Infinity. |
| `name`    | string  | Yes      |         | Identifier for this event. Used by applications to determine response. |
| `payload` | object  | No       |         | Application-specific JSON data attached to this event. |
| `isSync`  | boolean | No       | `false` | If true, this event is a sync marker for animation alignment. |

## Event Triggering

Events fire when animation playback **crosses** the event time:

- **Forward playback**: Event fires when time advances past `event.time`
- **Backward playback**: Event fires when time retreats past `event.time`
- **Looping**: Events fire each time playback crosses the event time during a loop

### Backward Playback Order

During backward playback, events fire in **reverse chronological order** (highest time first). For example, playing backwards from `t=1.0` to `t=0.0` fires events in order: `t=0.9`, `t=0.5`, `t=0.2`, `t=0.0`.

### Edge Cases

| Scenario | Behavior |
|----------|----------|
| Playback starts exactly at `event.time` | Event fires immediately |
| Playback jumps over `event.time` (seek) | Event fires (events are not skipped on seeks) |
| Playback paused at `event.time` | Event does NOT fire while paused. Fires once when playback resumes and crosses the time. |
| Seeking to `event.time` while paused | Event does NOT fire until playback resumes. |
| Multiple events at same `time` | All events fire in array order |
| Event time exceeds animation duration | Event fires only during non-looping playback that extends beyond the animation's sampler input range. |

### Time Resolution

Event times use double-precision floating-point seconds. Implementations SHOULD fire events with a tolerance of ≤ 1 millisecond from the specified time at 1x playback speed. At higher playback speeds, the tolerance scales proportionally (e.g., ≤ 10ms at 10x speed).

### Numeric Safety

- Event `time` values MUST NOT be NaN or Infinity
- Implementations SHOULD reject events with invalid time values during validation
- Negative `time` values are invalid (schema enforces `minimum: 0`)

## Sync Markers

Sync markers (`isSync: true`) are special events used for **animation alignment** across layers. When transitioning between animations or synchronizing layered animations, sync markers provide natural alignment points.

### Common Sync Marker Patterns

```json
{
  "events": [
    { "time": 0.0,  "name": "CycleStart", "isSync": true },
    { "time": 0.5,  "name": "CycleMid",   "isSync": true },
    { "time": 1.0,  "name": "CycleEnd",   "isSync": true }
  ]
}
```

### Sync Alignment Use Cases

- **Locomotion blending**: Align foot plants between walk/run cycles
- **Combat animations**: Sync attack windups across different weapon types
- **Gesture layering**: Align gesture peaks with locomotion cycles

## Payload Data

The optional `payload` property allows application-specific data to be attached to events:

```json
{
  "events": [
    {
      "time": 0.2,
      "name": "FootstepLeft",
      "payload": {
        "foot": "left",
        "surface": "default",
        "intensity": 0.8
      }
    },
    {
      "time": 0.5,
      "name": "PlaySound",
      "payload": {
        "soundId": "sword_whoosh",
        "volume": 0.6,
        "pitch": 1.2
      }
    }
  ]
}
```

Payload schema is not validated by this extension—applications define their own conventions.

## Complete Example

```json
{
  "asset": { "version": "2.0" },
  "extensionsUsed": ["KHR_animation_events"],
  "animations": [
    {
      "name": "WalkCycle",
      "extensions": {
        "KHR_animation_events": {
          "events": [
            {
              "time": 0.0,
              "name": "CycleStart",
              "isSync": true
            },
            {
              "time": 0.25,
              "name": "FootstepLeft",
              "payload": {
                "foot": "left"
              }
            },
            {
              "time": 0.5,
              "name": "CycleMid",
              "isSync": true
            },
            {
              "time": 0.75,
              "name": "FootstepRight",
              "payload": {
                "foot": "right"
              }
            },
            {
              "time": 1.0,
              "name": "CycleEnd",
              "isSync": true
            }
          ]
        }
      },
      "channels": [
        { "sampler": 0, "target": { "node": 0, "path": "translation" } }
      ],
      "samplers": [
        { "input": 0, "output": 1 }
      ]
    }
  ]
}
```

> **Note**: `accessors`, `bufferViews`, and `buffers` omitted for brevity.

## Relationship to KHR_animation_layers

When animations with events are played through `KHR_animation_layers`:

- Events fire based on the **animation's local time**, not the layer's global time
- Layer weight does NOT affect event firing (events fire even at weight 0)
- Masked animations still fire events (masking affects transforms, not events)

Applications that need weight-dependent event behavior should implement custom logic.

## Relationship to KHR_interactivity

This extension is designed to integrate with `KHR_interactivity` when that extension adds animation event support. Implementation details will be defined in a future version of KHR_interactivity.

## Conformance

A conforming implementation of this extension:

- MUST fire events when playback crosses the event time
- MUST fire events in array order when multiple events occur at the same time
- MUST fire events in reverse chronological order during backward playback
- MUST support at least 4096 events per animation
- SHOULD fire events within 1 millisecond tolerance at normal playback speed
- SHOULD store events sorted by time for efficient processing

## Implementation Notes

- **Event ordering**: Events SHOULD be stored sorted by `time` for efficient playback processing.
- **Limits**: Implementations SHOULD support at least 4096 events per animation.
- **Memory**: Event names MAY be interned to reduce memory for repeated names.
- **Validation**: Implementations SHOULD reject negative `time` values and NaN/Infinity.
- **Precision**: Event times should not require more than millisecond precision for typical use cases.

## DCC Tool Mapping

| DCC Tool | Source Feature |
|----------|----------------|
| Blender | NLA Strip Action Markers, Timeline Markers |
| Maya | Time Editor Bookmarks, Animation Markers |
| Unity | Animation Events |
| Unreal | Animation Notifies |

> **Note**: Blender's glTF exporter does not currently support exporting markers. This mapping is aspirational.

## Backwards Compatibility

- This extension SHOULD be listed in `extensionsUsed`, NOT `extensionsRequired`
- Loaders that do not support this extension will play animations normally
- Events are informational and do not affect animation playback correctness

## Known Implementations

(To be added as implementations become available)

## License

This extension specification is licensed under the Khronos Group Extension License.
See: <https://www.khronos.org/registry/gltf/license.html>
