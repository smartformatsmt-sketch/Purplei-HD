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

Purplei‑HD metadata is represented as a structured JSON object.

## 2.1 Root Structure

```json
{
  "stance": { ... },
  "phase": { ... },
  "geometry": { ... },
  "affect": { ... },
  "identity": { ... },
  "space": { ... }
}
```

---

# 3. Stance Layer

Represents the user’s **behavioral mode**. The stance layer informs how audio should adapt to the user's current embodied behavior (movement, attention, ritualized action, etc.).

## 3.1 Fields

```json
"stance": {
  "mode": "still | moving | focused | open | ritual",
  "intensity": 0.0
}
```

Notes:
- mode: one of the enumerated strings describing behavioral modes. Engines MUST treat unknown modes as "open" when possible.
- intensity: a normalized floating point value in the range 0.0–1.0 indicating the strength of the stance. Default: 0.0.

## 3.2 Mode Descriptions

- still: The user is stationary or behaving with minimal motion. Use subtle, stable audio transforms (e.g., low modulation, steady ambience).
- moving: The user is in transit or actively moving. Use spatialization, doppler-like effects, or increased dynamism in audio behavior.
- focused: The user is concentrating on a task or object. Prioritize clarity, center audio cues, reduce ambient complexity.
- open: The user is receptive, exploratory, or neutral. Apply balanced processing; suitable as a default fallback.
- ritual: The user is engaged in a repeated, symbolic, or ceremonial action. Apply pattern-based, evolving transforms and emphasize temporal structure.

## 3.3 Intensity

- Range: 0.0 (none) to 1.0 (maximum).
- Interpretation: Engines map intensity to effect depth, modulation amount, mix level, or other continuous parameters. Designers SHOULD document the mapping for each engine.
- Default: 0.0 (when omitted).

## 3.4 JSON Schema (example)

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Purplei-HD Stance",
  "type": "object",
  "properties": {
    "stance": {
      "type": "object",
      "properties": {
        "mode": {
          "type": "string",
          "enum": ["still", "moving", "focused", "open", "ritual"],
          "description": "Behavioral mode"
        },
        "intensity": {
          "type": "number",
          "minimum": 0.0,
          "maximum": 1.0,
          "default": 0.0,
          "description": "Normalized intensity of the stance"
        }
      },
      "required": ["mode"],
      "additionalProperties": false
    }
  },
  "required": ["stance"],
  "additionalProperties": true
}
```

## 3.5 Examples

Example 1 — stationary, low-intensity:

```json
{
  "stance": {
    "mode": "still",
    "intensity": 0.1
  }
}
```

Example 2 — moving, high-intensity:

```json
{
  "stance": {
    "mode": "moving",
    "intensity": 0.85
  }
}
```

Example 3 — focused with moderate intensity in a full metadata payload:

```json
{
  "stance": { "mode": "focused", "intensity": 0.6 },
  "phase": { ... },
  "geometry": { ... },
  "affect": { ... },
  "identity": { ... },
  "space": { ... }
}
```

## 3.6 Implementation Guidance

- Mapping: Engines SHOULD document how intensity maps to audio parameters (e.g., reverb wetness, filter cutoff, modulation depth).
- Transitions: For smooth UX, interpolate intensity and mode transitions over a short time window rather than applying abrupt changes.
- Fallbacks: If mode is missing or unrecognized, treat as "open" and use intensity if present.

---

(End of insertion)
