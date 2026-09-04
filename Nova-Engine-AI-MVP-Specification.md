# Nova Engine --- AI MVP Technical Specification

## 1. Objective

The AI subsystem provides natural-language control of the game engine.

The MVP is intentionally constrained:

> **Natural language → LLM → structured action → validation →
> whitelisted engine function**

The AI is not an autonomous game developer and does not execute
arbitrary code.

------------------------------------------------------------------------

# 2. AI Architecture

``` mermaid
flowchart TD
    U[User Request] --> API[AI API]
    API --> CTX[Build Scene Context]
    CTX --> LLM[LLM]
    LLM --> ACTION[Structured Action]
    ACTION --> SCHEMA[Schema Validation]
    SCHEMA --> POLICY[Whitelist + Parameter Validation]
    POLICY --> REG[Action Registry]
    REG --> ENGINE[Engine API]
    ENGINE --> RESULT[Execution Result]
    RESULT --> API
    API --> U

    SCHEMA -->|Invalid| CLARIFY[Reject / Clarify]
    POLICY -->|Unsupported| CLARIFY
```

------------------------------------------------------------------------

# 3. Technology Stack

## Required

``` text
Python
FastAPI
LLM API
Pydantic / JSON Schema
```

## Application Components

``` text
LLM Client
Prompt Builder
Scene Context Builder
Action Schema
Validator
Action Registry
Engine API Client
Error Handler
Evaluation Harness
```

No model training is required.

PyTorch and TensorFlow are not required for the MVP.

------------------------------------------------------------------------

# 4. Supported Actions

The MVP supports:

``` text
create_object
delete_object
move_object
rotate_object
scale_object
change_material
add_light
add_camera
duplicate_object
```

These actions form the AI's controlled tool set.

------------------------------------------------------------------------

# 5. Structured Action Schema

Example:

``` json
{
  "action": "create_object",
  "parameters": {
    "type": "cube",
    "name": "Cube_01",
    "position": [0, 0, 0]
  }
}
```

Move:

``` json
{
  "action": "move_object",
  "parameters": {
    "object_id": "Cube_01",
    "position": [2, 0, 5]
  }
}
```

Rotate:

``` json
{
  "action": "rotate_object",
  "parameters": {
    "object_id": "Cube_01",
    "rotation": [0, 45, 0]
  }
}
```

Material:

``` json
{
  "action": "change_material",
  "parameters": {
    "object_id": "Cube_01",
    "color": "#FF0000"
  }
}
```

------------------------------------------------------------------------

# 6. Action Registry

``` python
ACTION_REGISTRY = {
    "create_object": create_object,
    "delete_object": delete_object,
    "move_object": move_object,
    "rotate_object": rotate_object,
    "scale_object": scale_object,
    "change_material": change_material,
    "add_light": add_light,
    "add_camera": add_camera,
    "duplicate_object": duplicate_object,
}
```

Only actions in this registry can reach the engine.

------------------------------------------------------------------------

# 7. Validation Pipeline

``` mermaid
flowchart LR
    JSON[LLM JSON] --> S[Schema]
    S --> P[Parameters]
    P --> W[Whitelist]
    W --> R[Registry]
    R --> E[Engine]
```

Validation must check:

-   Action exists.
-   Required parameters exist.
-   Parameter types are correct.
-   Object IDs are valid.
-   Values are within allowed ranges.
-   The action is whitelisted.

Failure must produce a controlled error.

------------------------------------------------------------------------

# 8. Clarification

The AI must not guess when an instruction is ambiguous.

Example:

> Move the cube.

Scene:

``` text
Cube_01
Cube_02
Cube_03
```

Response:

``` json
{
  "action": "clarification_required",
  "message": "Which cube do you want to move?"
}
```

This is preferable to executing an arbitrary action.

------------------------------------------------------------------------

# 9. Scene Context

The AI receives a compact representation of the current scene.

``` json
{
  "entities": [
    {
      "id": "cube_01",
      "name": "RedCube",
      "type": "cube",
      "position": [0, 0, 0]
    },
    {
      "id": "house_01",
      "name": "House",
      "type": "mesh",
      "position": [10, 0, 5]
    }
  ],
  "selected_object": "cube_01"
}
```

Do not send the entire engine state unnecessarily.

The context builder should expose only information relevant to the
current request.

------------------------------------------------------------------------

# 10. Execution Result

The engine returns a structured result.

Success:

``` json
{
  "success": true,
  "action": "create_object",
  "object_id": "cube_01"
}
```

Failure:

``` json
{
  "success": false,
  "error": {
    "code": "ENTITY_NOT_FOUND",
    "message": "Entity cube_01 does not exist."
  }
}
```

The AI layer can turn the result into a user-facing message.

------------------------------------------------------------------------

# 11. Example End-to-End Requests

### Simple

> Create a cube.

→ `create_object`

### Transformation

> Move the selected cube to the right.

→ `move_object`

### Material

> Make the cube red.

→ `change_material`

### Duplication

> Duplicate this object.

→ `duplicate_object`

### Ambiguous

> Delete the cube.

If several cubes exist:

→ `clarification_required`

------------------------------------------------------------------------

# 12. AI Service Structure

``` text
ai-service/
│
├── api/
│   └── routes.py
│
├── agent/
│   ├── llm.py
│   ├── prompts.py
│   └── context.py
│
├── actions/
│   ├── schemas.py
│   ├── validator.py
│   └── registry.py
│
├── engine/
│   └── client.py
│
├── evaluation/
│   ├── dataset.json
│   └── evaluator.py
│
└── main.py
```

------------------------------------------------------------------------

# 13. AI Development Plan

## Phase 1 --- Prototype

-   Python environment
-   FastAPI endpoint
-   LLM connection
-   One action: `create_object`
-   Mock engine function

Goal:

``` text
"Create a cube"
        ↓
Structured action
        ↓
create_object()
```

------------------------------------------------------------------------

## Phase 2 --- Action System

Implement all nine actions.

Add:

-   Pydantic schemas
-   Validation
-   Action registry
-   Error handling
-   Whitelist enforcement

Goal:

``` text
Natural Language
        ↓
Any supported action
        ↓
Validated engine function
```

------------------------------------------------------------------------

## Phase 3 --- Scene Awareness

Add:

-   Current scene context
-   Selected object
-   Entity identification
-   Ambiguity detection

Goal:

> Move the red cube next to the house.

works without hard-coded object IDs in the user request.

------------------------------------------------------------------------

## Phase 4 --- Frontend Integration

Connect:

``` text
React AI Panel
        ↓
AI Service
        ↓
LLM
        ↓
Action
        ↓
Validation
        ↓
Engine API
        ↓
WASM
```

------------------------------------------------------------------------

## Phase 5 --- Evaluation

Build a command dataset and measure:

``` text
Action Selection Accuracy
Parameter Accuracy
Execution Success Rate
Invalid Request Rejection
Ambiguity Handling
```

The evaluation dataset should be representative of the supported engine
operations.

------------------------------------------------------------------------

# 14. Explicitly Not in AI MVP

Do not make these dependencies for the graduation MVP:

``` text
Autonomous multi-step planning
Text-to-3D
Image generation
Texture generation
AI animation
AI music
LLM fine-tuning
Training a model from scratch
Arbitrary C++ generation/execution
Complex RAG system
```

These can become future extensions after the command system is stable.

------------------------------------------------------------------------

# 15. Stretch AI Roadmap

``` mermaid
flowchart LR
    MVP[Structured Commands]
    MVP --> CONTEXT[Advanced Scene Context]
    CONTEXT --> PLAN[Multi-step Planning]
    PLAN --> RAG[Engine Documentation RAG]
    RAG --> DEBUG[AI Debugging]
    DEBUG --> LEVEL[AI Level Generation]
    LEVEL --> ASSET[Generative Assets]
    ASSET --> T3D[Text-to-3D]

    style MVP fill:#d9ead3
```

The MVP command architecture should remain the foundation for all future
AI features.

------------------------------------------------------------------------

# 16. AI Definition of Done

The AI MVP is complete when:

1.  A user can enter natural-language scene instructions.
2.  The LLM returns only the defined structured action format.
3.  Every action is schema-validated.
4.  Unsupported actions are rejected.
5.  Invalid parameters are rejected.
6.  Ambiguous requests trigger clarification.
7.  Valid actions are mapped to whitelisted engine functions.
8.  The engine executes the action.
9.  The result is returned to the frontend.
10. Automated tests measure AI command reliability.

------------------------------------------------------------------------

# 17. Core Design Principle

``` text
             LLM
              │
              │ produces data
              ▼
      Structured Action
              │
              ▼
         Validation
              │
              ▼
       Action Registry
              │
              │ calls only approved functions
              ▼
          Engine API
              │
              ▼
         C++ / WASM
```

> **The LLM is a decision-making interface, not an execution
> environment.**

This keeps the AI subsystem deterministic enough to test, safe enough to
integrate, and small enough to complete within the academic-year MVP.
