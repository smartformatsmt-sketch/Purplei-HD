# Purplei‑HD Specification
### High‑Dimensional Context‑Aware Audio Protocol
### Author: Brandon Lee Frost

Purplei‑HD defines a metadata‑driven system for generating **context‑aware audio** in AR, VR, mobile, and spatial computing environments.  
It introduces a new sensory layer that encodes **stance, phase, geometry, emotional color, identity, and spatial orientation** into sound behavior.

This document is the official protocol specification.

---

# 1. Overview

Purplei‑HD is a **contextual audio protocol** that wraps traditional audio with a structured metadata layer.  
This metadata informs a compliant engine how to transform sound in real time based on:

- user state  
- symbolic geometry  
- emotional tone  
- ritual phase  
- identity signature  
- spatial orientation  

Purplei‑HD does not replace audio codecs.  
It defines **how audio should behave** in adaptive environments.

---

# 2. Metadata Schema

Purplei‑HD metadata is represented as a structured JSON object. Implementations SHOULD include root metadata fields to aid validation, provenance, and versioning.

## 2.1 Root Structure

```json
{
  "schema": "https://example.com/schemas/purplei-hd/1.0/metadata.json",
  "version": "1.0.0",
  "timestamp": "2025-12-24T03:46:46Z",
  "source": "device/sensor-suite",
  "confidence": 0.87,
  "stance": { ... },
  "phase": { ... },
  "geometry": { ... },
  "affect": { ... },
  "identity": { ... },
  "space": { ... }
}
```

Notes:
- schema: canonical URI for schema used (machine-discoverable)
- version: semantic version of the metadata payload
- timestamp: ISO 8601 UTC timestamp for when the payload was produced
- source: producer identifier (sensor, model, or engine)
- confidence: optional normalized confidence (0.0–1.0) about the provided metadata

---

# 3. Stance Layer

Represents the user’s **behavioral mode**. The stance layer informs how audio should adapt to the user's current embodied behavior (movement, attention, ritualized action, etc.).

## 3.1 Fields

```json
"stance": {
  "mode": "still | moving | focused | open | ritual",
  "intensity": 0.0,
  "transition": { "duration": 0.25, "easing": "easeInOut" },
  "mode_history": [],
  "engine_hints": {}
}
```

Notes:
- mode: one of the enumerated strings describing behavioral modes. Engines MUST treat unknown modes as "open" when possible.
- intensity: a normalized floating point value in the range 0.0–1.0 indicating the strength of the stance. Default: 0.0.
- transition: optional object describing how to smoothly apply changes (see 3.4)
- mode_history: optional ordered list of recent modes with timestamps (see 3.5)
- engine_hints: optional key-value map to suggest parameter mappings for engines (non-normative)

## 3.2 Mode Descriptions

- still: The user is stationary or behaving with minimal motion. Use subtle, stable audio transforms (e.g., low modulation, steady ambience).
- moving: The user is in transit or actively moving. Use spatialization, doppler-like effects, or increased dynamism in audio behavior.
- focused: The user is concentrating on a task or object. Prioritize clarity, center audio cues, reduce ambient complexity.
- open: The user is receptive, exploratory, or neutral. Apply balanced processing; suitable as a default fallback.
- ritual: The user is engaged in a repeated, symbolic, or ceremonial action. Apply pattern-based, evolving transforms and emphasize temporal structure.

## 3.3 Intensity

- Range: 0.0 (none) to 1.0 (maximum).
- Interpretation: Engines map intensity to effect depth, modulation amount, mix level, or other continuous parameters. Designers SHOULD document the mapping for each engine.
- Default: 0.0 (when omitted). Engines MUST clamp values to [0.0, 1.0].

## 3.4 Transition Metadata

To avoid abrupt audio changes, producers MAY include a transition object describing how to interpolate between previous and current stance values.

```json
"transition": {
  "duration": 0.25,
  "easing": "linear" // one of: linear, easeIn, easeOut, easeInOut
}
```

- duration: seconds to interpolate over (non-negative number).
- easing: easing algorithm for the interpolation; engines MAY use a mapping to their internal smoothing functions.

## 3.5 Mode History

Providing temporal context can improve detection of patterns such as rituals or oscillations. Mode history is optional and SHOULD be bounded in length.

```json
"mode_history": [
  { "mode": "still", "since": "2025-12-24T03:40:00Z" },
  { "mode": "moving", "since": "2025-12-24T03:42:10Z" }
]
```

## 3.6 Engine Hints (non-normative)

engine_hints is an optional map that suggests how to map stance fields to engine parameters. Hints are advisory only and MUST NOT be treated as authoritative.

```json
"engine_hints": {
  "reverb_wet": "intensity * 0.6",
  "filter_cutoff": "2000 + (intensity * 4000)",
  "lfo_depth": "intensity"
}
```

## 3.7 Implementation Guidance

- Mapping: Engines SHOULD document how intensity maps to audio parameters (e.g., reverb wetness, filter cutoff, modulation depth).
- Transitions: For smooth UX, interpolate intensity and mode transitions over a short time window rather than applying abrupt changes.
- Fallbacks: If mode is missing or unrecognized, treat as "open" and use intensity if present.
- Validation: Engines SHOULD validate incoming metadata against the JSON Schema and clamp or default missing fields where appropriate.

---

# 4. Enhancements and Extensions

This section contains optional extensions and operational guidance to improve interoperability and adoption.

## 4.1 JSON-LD and Canonical Schema

To support discoverability and linked-data tooling, producers MAY publish a JSON-LD @context and use a canonical schema URI in the root "schema" property. Example @context (non-normative):

```json
"@context": {
  "Purplei": "https://example.com/ns/purplei#",
  "stance": "Purplei:stance",
  "mode": "Purplei:mode",
  "intensity": "Purplei:intensity"
}
```

## 4.2 Privacy and Security Guidance

Stance may be derived from sensitive sensor data (pose, camera, biometric). Implementers MUST consider privacy and follow these guidelines:

- Data minimization: transmit only the metadata required for audio behavior, not raw sensor streams.
- Anonymization: avoid embedding persistent identifiers in metadata unless necessary; prefer ephemeral sources.
- Consent: ensure user consent covers the use of inferred behavioral metadata.
- Local processing: when possible, compute stance on-device and transmit only the resulting small metadata payload.
- Logging: avoid long-term storage of fine-grained modality timestamps (mode_history) unless explicitly allowed.

## 4.3 Conformance Tests (suggested)

Provide example test vectors and expected behaviors for engines. Minimal test cases:

- Test A: Missing mode — payload { "stance": { "intensity": 0.5 } } should be treated as mode="open" and intensity=0.5.
- Test B: Intensity clamping — payload with intensity=1.5 should be clamped to 1.0.
- Test C: Transition smoothing — sudden mode change with transition.duration=0.5 should ramp parameters over ~0.5s.
- Test D: Mode history usage — engine that detects ritual should identify repeating mode patterns in mode_history.
- Test E: Phase progress clamping — payload with phase.progress=1.2 should be clamped to 1.0.
- Test F: Missing phase state — payload { "phase": { "progress": 0.5 } } should default state to "integration" and use progress.

## 4.4 Reference Implementations and Snippets

TypeScript interfaces (example):

```ts
export type PhaseState = 'entry' | 'ascent' | 'peak' | 'descent' | 'integration';

export interface Transition {
  duration: number;
  easing?: 'linear'|'easeIn'|'easeOut'|'easeInOut';
}

export interface Stance {
  mode: 'still' | 'moving' | 'focused' | 'open' | 'ritual';
  intensity?: number; // 0.0 - 1.0
  transition?: Transition;
  mode_history?: { mode: string; since: string }[];
  engine_hints?: Record<string, any>;
}

export interface Phase {
  state: PhaseState;
  progress?: number; // 0.0 - 1.0
  transition?: Transition;
  engine_hints?: Record<string, any>;
}

export interface PurpleiHDMetadata {
  schema: string;
  version: string;
  timestamp: string;
  source?: string;
  confidence?: number;
  stance: Stance;
  phase?: Phase;
  [key: string]: any;
}
```

Python dataclasses (example):

```py
from dataclasses import dataclass
from typing import Optional, List, Dict

@dataclass
class ModeHistoryEntry:
    mode: str
    since: str

@dataclass
class Transition:
    duration: float
    easing: Optional[str] = 'linear'

@dataclass
class Stance:
    mode: str
    intensity: float = 0.0
    transition: Optional[Transition] = None
    mode_history: Optional[List[ModeHistoryEntry]] = None
    engine_hints: Optional[Dict[str, object]] = None

@dataclass
class Phase:
    state: str
    progress: float = 0.0
    transition: Optional[Transition] = None
    engine_hints: Optional[Dict[str, object]] = None

@dataclass
class PurpleiHDMetadata:
    schema: str
    version: str
    timestamp: str
    source: Optional[str] = None
    confidence: Optional[float] = None
    stance: Stance = None
    phase: Optional[Phase] = None
    extra: Optional[Dict[str, object]] = None
```

## 4.5 Expanded JSON Schema (root + stance + phase)

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Purplei-HD Metadata",
  "type": "object",
  "properties": {
    "schema": { "type": "string", "format": "uri" },
    "version": { "type": "string" },
    "timestamp": { "type": "string", "format": "date-time" },
    "source": { "type": "string" },
    "confidence": { "type": "number", "minimum": 0, "maximum": 1 },
    "stance": {
      "type": "object",
      "properties": {
        "mode": { "type": "string", "enum": ["still", "moving", "focused", "open", "ritual"] },
        "intensity": { "type": "number", "minimum": 0, "maximum": 1, "default": 0 },
        "transition": {
          "type": "object",
          "properties": {
            "duration": { "type": "number", "minimum": 0 },
            "easing": { "type": "string", "enum": ["linear","easeIn","easeOut","easeInOut"] }
          },
          "additionalProperties": false
        },
        "mode_history": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "mode": { "type": "string" },
              "since": { "type": "string", "format": "date-time" }
            },
            "required": ["mode","since"],
            "additionalProperties": false
          }
        },
        "engine_hints": { "type": "object", "additionalProperties": true }
      },
      "required": ["mode"],
      "additionalProperties": false
    },
    "phase": {
      "type": "object",
      "properties": {
        "state": { "type": "string", "enum": ["entry","ascent","peak","descent","integration"] },
        "progress": { "type": "number", "minimum": 0, "maximum": 1, "default": 0 },
        "transition": {
          "type": "object",
          "properties": {
            "duration": { "type": "number", "minimum": 0 },
            "easing": { "type": "string", "enum": ["linear","easeIn","easeOut","easeInOut"] }
          },
          "additionalProperties": false
        },
        "engine_hints": { "type": "object", "additionalProperties": true }
      },
      "required": ["state"],
      "additionalProperties": false
    }
  },
  "required": ["stance"],
  "additionalProperties": true
}
```

## 4.6 Example Full Payload

```json
{
  "schema": "https://example.com/schemas/purplei-hd/1.0/metadata.json",
  "version": "1.0.0",
  "timestamp": "2025-12-24T03:46:46Z",
  "source": "device/pose-v2",
  "confidence": 0.92,
  "stance": {
    "mode": "moving",
    "intensity": 0.85,
    "transition": { "duration": 0.3, "easing": "easeInOut" },
    "mode_history": [
      { "mode": "still", "since": "2025-12-24T03:40:00Z" },
      { "mode": "moving", "since": "2025-12-24T03:42:10Z" }
    ],
    "engine_hints": { "reverb_wet": "intensity * 0.6" }
  },
  "phase": {
    "state": "ascent",
    "progress": 0.42,
    "transition": { "duration": 0.4, "easing": "easeInOut" },
    "engine_hints": {
      "spectral_tilt": "-6 + (progress * -12)",
      "reverb_size": "0.4 + (progress * 0.6)"
    }
  }
}
```

---

# 5. Conformance and Attribution

- Engines and producers that implement Purplei‑HD SHOULD declare which schema version they support using the root "schema" and "version" fields.
- When extending the spec, use a vendor prefix and publish the extension schema URI.

---

# 6. Phase Layer

The Phase layer represents the user’s **experiential or ritual phase** within a temporal sequence.  
Phases describe *where* the user is in a symbolic, emotional, or procedural arc.  
Engines use this to shape transitions, harmonic evolution, and temporal structure.

## 6.1 Fields

```json
"phase": {
  "state": "entry | ascent | peak | descent | integration",
  "progress": 0.0,
  "transition": { "duration": 0.4, "easing": "easeInOut" },
  "engine_hints": {}
}
```

- state: one of the enumerated strings describing the phase within an arc. Engines SHOULD treat unknown states as "integration" when possible.
- progress: normalized value (0.0–1.0) indicating how far along the current phase is; engines map this to temporal envelopes, morph targets, or modulation indices.
- transition: optional smoothing metadata to drive crossfades or parameter ramps between phases.
- engine_hints: optional advisory mappings for engine-specific parameters (non-normative).

## 6.2 Engine Hints and Examples (non-normative)

The Phase layer benefits from explicit engine hint examples to guide implementers. Below are a handful of illustrative mappings showing how "progress" or "state" could be mapped to audio parameters. These are advisory only.

Example engine_hints for Phase:

```json
"engine_hints": {
  "spectral_tilt": "-6 + (progress * -12)",    // dB per octave, more negative as phase progresses
  "reverb_size": "0.4 + (progress * 0.6)",      // mix between 0.4 -> 1.0
  "mod_index": "progress * 5",                 // modulation depth 0 -> 5
  "voice_brightness": "(state == 'peak') ? 1.0 : (progress)"
}
```

Interpretation guidance:
- spectral_tilt: engines can map this to filter/equalizer settings to darken or brighten timbre over the phase.
- reverb_size: maps to perceived environmental size or wet/dry mix.
- mod_index: maps to LFO or FM modulation depth for evolving textures.
- voice_brightness: conditional hint to sharply increase brightness at the peak state.

## 6.3 Implementation Guidance

- Progress mapping: Engines SHOULD clamp progress to [0.0, 1.0] and document how progress affects parameters.
- State fallbacks: For missing or unknown phase.state, default to "integration" and use progress if present.
- Transitions: When both stance.transition and phase.transition are present, engines MAY combine or prioritize according to local policy (e.g., phase transitions govern long-form structure, stance transitions govern immediate timbral changes).
- Validation: Include phase in schema validation and ensure numerical fields are clamped.

## 6.4 Example Phase Usage Patterns

- entry -> ascent: gradually open a filter and increase reverb_size, introduce harmonic layers.
- peak: emphasize brightness and transient clarity; increase modulation intensity for highlight effects.
- descent -> integration: reduce modulation, narrow bandwidth, and reduce reverb to return to ambient state.

---

# 7. Privacy, Attribution, and Versioning (continued)

See Section 4.2 for privacy guidance and Section 5 for conformance rules.

---

# Appendix: Additional Phase Conformance Tests

- Phase Test 1: Missing state — { "phase": { "progress": 0.3 } } -> default state="integration", progress=0.3
- Phase Test 2: Progress clamping — { "phase": { "state": "ascent", "progress": -0.5 } } -> progress clamped to 0.0
- Phase Test 3: Engine hints parsing — ensure engine_hints strings are treated as advisory, not executed as code (engines must parse safely).

