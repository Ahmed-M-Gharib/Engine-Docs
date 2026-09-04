# Nova Engine

## Graduation Project --- Final Technical Plan

> **Project identity:** Nova Engine is a browser-native, collaborative
> 3D game development platform built around a C++ engine compiled to
> WebAssembly (WASM). It provides a lightweight browser editor,
> cloud-backed project persistence and real-time collaboration, and an
> AI assistant that converts natural-language requests into validated,
> structured engine actions.

------------------------------------------------------------------------

## 1. Abstract & Vision

Nova Engine is a multidisciplinary graduation project combining systems
programming, browser-based graphics, distributed collaboration, applied
Generative AI, cloud infrastructure, and quality engineering.

The platform consists of:

-   **C++/WASM Engine** --- scene/entity system, rendering, and runtime.
-   **Frontend Editor** --- a lightweight React/TypeScript browser
    editor.
-   **Backend** --- project persistence, asset metadata, and
    server-authoritative collaboration.
-   **AI Command Layer** --- natural language → structured action →
    validation → engine function call.
-   **Cloud/DevOps** --- deployment, CI/CD, infrastructure, and
    observability.
-   **QA/Automation** --- automated testing across the stack.

The project deliberately focuses on a complete, demonstrable MVP rather
than attempting to reproduce a commercial engine such as Unreal Engine.

------------------------------------------------------------------------

# 2. MVP Scope

## 2.1 Mandatory MVP

  -----------------------------------------------------------------------
  Area                                MVP Scope
  ----------------------------------- -----------------------------------
  Engine                              C++ engine core, scene/entity
                                      system, basic 3D rendering,
                                      Emscripten/WASM build, JS ↔ WASM
                                      API

  Frontend                            React + TypeScript editor, viewport
                                      integration, hierarchy, inspector,
                                      basic asset browser, AI panel

  Backend                             REST API, PostgreSQL persistence,
                                      WebSocket server-authoritative
                                      synchronization

  AI                                  LLM integration, structured action
                                      schema, validation,
                                      whitelist/action registry, engine
                                      function execution, clarification
                                      handling

  Cloud/DevOps                        Docker, Terraform, Kubernetes,
                                      CI/CD, basic monitoring

  QA                                  UI, API, WebSocket, AI command, and
                                      engine integration tests
  -----------------------------------------------------------------------

## 2.2 Explicitly Out of Scope for MVP

The following are stretch goals and must not block the mandatory
deliverable:

-   CRDT/OT collaboration
-   Offline conflict-free editing
-   Advanced physics
-   Advanced scripting systems
-   PBR/post-processing/shadow pipelines
-   Sophisticated animation
-   Complex multi-format asset pipelines
-   Autonomous multi-step AI planning
-   Text-to-3D generation
-   AI-generated music/animation
-   Advanced autoscaling

------------------------------------------------------------------------

# 3. High-Level Architecture

``` mermaid
flowchart TB
    U[User] --> FE[React + TypeScript Editor]

    FE -->|REST| BE[Backend]
    FE -->|WebSocket| WS[Collaboration Service]
    FE -->|AI Request| AI[AI Service]
    FE -->|JS API| WASM[WebAssembly Engine]

    BE --> DB[(PostgreSQL)]
    WS --> DB

    AI --> LLM[LLM / Generative AI]
    AI --> VAL[Action Validator]
    VAL --> REG[Whitelisted Action Registry]
    REG -->|Engine API| FE

    WASM --> CPP[C++ Engine]
```

### Architectural principle

The browser executes the game engine locally through WebAssembly. The
backend is responsible for persistence and collaboration; it is **not**
in the per-frame gameplay loop.

The AI service does not directly access engine memory and does not
execute arbitrary code. It produces a structured action that is
validated and then routed through the same engine API used by normal
editor operations.

------------------------------------------------------------------------

# 4. Frontend Architecture

## 4.1 Minimal Technology Stack

  Technology         Responsibility
  ------------------ -------------------------------
  **React**          Editor UI
  **TypeScript**     Type safety and API contracts
  **Vite**           Development/build tooling
  **Zustand**        Local editor state
  **Tailwind CSS**   Styling

Optional additions should only be introduced when a concrete requirement
appears.

### Not required for MVP

-   Next.js
-   Redux
-   Three.js as the main renderer
-   Babylon.js
-   GraphQL
-   Socket.IO
-   Electron

The 3D viewport remains driven by the C++/WASM engine rather than
introducing a second JavaScript 3D engine.

## 4.2 Editor Components

``` mermaid
flowchart LR
    APP[React Editor] --> TB[Toolbar]
    APP --> SH[Scene Hierarchy]
    APP --> IN[Inspector]
    APP --> AB[Asset Browser]
    APP --> VP[WASM Viewport]
    APP --> AIUI[AI Panel]
    APP --> ST[Console / Status]
```

### Mandatory UI

1.  Toolbar
2.  Scene hierarchy
3.  Inspector
4.  Viewport
5.  Basic asset browser
6.  AI command panel
7.  Save/play/undo/redo controls
8.  Basic status/error feedback

The frontend should expose the engine without duplicating engine
responsibilities.

------------------------------------------------------------------------

# 5. Engine ↔ Frontend Contract

The engine exposes a small, stable, versioned API.

``` mermaid
flowchart LR
    UI[React UI] --> API[Engine API]
    AI[AI Action] --> API
    API --> BRIDGE[JS ↔ WASM Bridge]
    BRIDGE --> CPP[C++ Engine]
```

Example frontend calls:

``` ts
engine.createObject(...)
engine.deleteObject(...)
engine.setTransform(...)
engine.setMaterial(...)
engine.addLight(...)
engine.addCamera(...)
```

The frontend should not implement rendering, physics, or scene internals
itself.

------------------------------------------------------------------------

# 6. AI MVP --- Core Generative AI System

## 6.1 Main Concept

The AI MVP is intentionally a **constrained command interpreter**, not
an autonomous game developer.

``` mermaid
flowchart TD
    USER[User Natural Language] --> LLM[LLM]
    LLM --> ACTION[Structured Action]
    ACTION --> SCHEMA[Schema Validation]
    SCHEMA --> WHITELIST[Whitelist Validation]
    WHITELIST --> REGISTRY[Action Registry]
    REGISTRY --> ENGINE[Engine API]
    ENGINE --> CPP[C++ / WASM Engine]

    SCHEMA -->|Invalid| ERROR[Reject / Ask for Clarification]
    WHITELIST -->|Unsupported| ERROR
```

### Core rule

> **LLM output is data, not executable code.**

The LLM never executes C++, touches engine memory, or calls arbitrary
server functions.

------------------------------------------------------------------------

# 7. AI Action Model

The AI communicates with the engine through a fixed structured action
format.

Example:

``` json
{
  "action": "create_object",
  "parameters": {
    "type": "cube",
    "name": "RedCube",
    "position": [0, 0, 0]
  }
}
```

Another example:

``` json
{
  "action": "move_object",
  "parameters": {
    "object_id": "cube_01",
    "position": [2, 0, 5]
  }
}
```

The action schema becomes the contract between the AI team, frontend
team, backend team, and engine team.

------------------------------------------------------------------------

# 8. Whitelisted MVP Actions

The mandatory AI MVP supports exactly these nine operations:

1.  `create_object`
2.  `delete_object`
3.  `move_object`
4.  `rotate_object`
5.  `scale_object`
6.  `change_material`
7.  `add_light`
8.  `add_camera`
9.  `duplicate_object`

Additional actions require an explicit scope decision.

------------------------------------------------------------------------

# 9. Action Registry

Every supported action maps to one controlled engine function.

``` mermaid
flowchart TD
    A[Structured AI Action] --> V[Validator]
    V --> R[Action Registry]

    R --> C[create_object()]
    R --> D[delete_object()]
    R --> M[move_object()]
    R --> RT[rotate_object()]
    R --> S[scale_object()]
    R --> MAT[change_material()]
    R --> L[add_light()]
    R --> CAM[add_camera()]
    R --> DUP[duplicate_object()]

    C --> E[Engine API]
    D --> E
    M --> E
    RT --> E
    S --> E
    MAT --> E
    L --> E
    CAM --> E
    DUP --> E
```

Conceptually:

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

------------------------------------------------------------------------

# 10. AI Execution Example

User:

> Create a red cube.

``` mermaid
sequenceDiagram
    participant U as User
    participant F as Frontend
    participant A as AI Service
    participant L as LLM
    participant V as Validator
    participant E as Engine

    U->>F: "Create a red cube"
    F->>A: AI request + scene context
    A->>L: Prompt + available actions
    L-->>A: Structured action
    A->>V: Validate action
    V-->>A: Valid
    A->>E: create_object(...)
    E-->>F: Entity created
    F-->>U: Updated scene
```

Possible action:

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

A second action can change its material:

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

# 11. Validation and Safety

The AI pipeline must always be:

``` text
Natural Language
      ↓
LLM
      ↓
Structured Action
      ↓
Schema Validation
      ↓
Whitelist Validation
      ↓
Parameter Validation
      ↓
Engine Function
```

Invalid requests are rejected.

Examples:

``` text
Unsupported action
Invalid object ID
Invalid numeric range
Malformed parameters
Ambiguous object reference
```

The AI should ask for clarification instead of guessing when necessary.

Example:

> Move the cube.

If multiple cubes exist:

> Which cube do you want to move?

------------------------------------------------------------------------

# 12. Scene Context

The AI needs enough information to identify objects, but the complete
scene should not be sent blindly on every request.

Example context:

``` json
{
  "scene": {
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
    ]
  },
  "selected_object": "cube_01"
}
```

This allows requests such as:

> Move the selected object to the left.

or:

> Move the red cube next to the house.

------------------------------------------------------------------------

# 13. AI Error Handling

The engine should return structured errors.

``` json
{
  "success": false,
  "error": {
    "code": "ENTITY_NOT_FOUND",
    "message": "Entity cube_01 does not exist."
  }
}
```

The AI service can then produce a user-friendly response.

The AI must not silently invent a successful result.

------------------------------------------------------------------------

# 14. AI Technology Stack

### Mandatory

``` text
Python
FastAPI
LLM API
Pydantic / JSON Schema
Action Registry
Engine API client
```

### Optional later

``` text
PostgreSQL-backed conversation persistence
RAG
Embeddings
Vector search
Advanced agent frameworks
```

No model training is required for the MVP.

No PyTorch/TensorFlow requirement exists for the MVP.

------------------------------------------------------------------------

# 15. AI Team Responsibilities

## AI Engineer 1 --- LLM & Command System

-   LLM API integration
-   System prompt
-   Action/tool definitions
-   Structured output schema
-   Scene context construction
-   Clarification handling
-   Conversation context

## AI Engineer 2 --- Execution & Reliability

-   Action validation
-   Whitelist enforcement
-   Action registry
-   Engine API integration
-   Error handling
-   Logging
-   AI evaluation tests
-   Failure/rejection testing

Both engineers jointly maintain the AI-engine contract.

------------------------------------------------------------------------

# 16. AI Evaluation

The AI should be evaluated as an engineering subsystem rather than
demonstrated only through screenshots.

Create a test dataset of representative commands:

``` text
Create a cube.
Move the cube to (2,0,5).
Rotate the selected object.
Make the cube red.
Duplicate the house.
Add a point light.
Delete object X.
Move the red cube next to the house.
```

Measure:

-   Valid action rate
-   Correct action selection
-   Parameter correctness
-   Engine execution success
-   Unsupported-action rejection
-   Ambiguous-request handling
-   Invalid-parameter rejection

Example target metrics can be defined after the first prototype rather
than inventing fixed accuracy claims in advance.

------------------------------------------------------------------------

# 17. Collaboration Model

The MVP remains server-authoritative:

``` mermaid
sequenceDiagram
    participant C1 as Client A
    participant S as Server
    participant DB as PostgreSQL
    participant C2 as Client B

    C1->>S: Scene operation
    S->>S: Validate / sequence
    S->>DB: Persist
    S-->>C1: Confirm operation
    S-->>C2: Broadcast operation
```

AI-issued operations should enter the same operation pathway as manual
editor operations wherever practical.

This keeps AI actions consistent with normal editing and makes them
easier to test.

------------------------------------------------------------------------

# 18. Vertical Slice

The first integration milestone should be extremely small:

``` mermaid
flowchart LR
    E[C++ Engine] --> W[WASM]
    W --> F[React Editor]
    F --> B[Backend]
    F --> A[AI Service]
    A --> L[LLM]
    L --> A
    A --> F
```

### Demonstration

1.  Open project.
2.  See a WASM-rendered scene.
3.  Create/select an object.
4.  Type: **"Create a red cube."**
5.  AI produces a structured action.
6.  Action is validated.
7.  Engine creates the cube.
8.  Frontend updates the hierarchy and viewport.
9.  Save the project.
10. Reload and verify persistence.

If this works, the team has a defensible end-to-end foundation.

------------------------------------------------------------------------

# 19. Development Plan

## Stage 1 --- Foundation

### Engine

-   Scene/entity model
-   Minimal renderer
-   WASM build
-   Initial JS/WASM API

### Frontend

-   React/TypeScript/Vite setup
-   Editor shell
-   Viewport integration
-   Hierarchy
-   Inspector

### Backend

-   Project model
-   PostgreSQL schema
-   REST foundation
-   WebSocket foundation

### AI

-   Python/FastAPI service
-   LLM connection
-   First action schema
-   Mock engine action registry

### DevOps

-   Docker
-   CI pipeline
-   Development deployment

### QA

-   Test strategy
-   Unit test foundations
-   API contract tests

------------------------------------------------------------------------

## Stage 2 --- Core Development

### Frontend

-   Scene editing UI
-   Transform controls
-   Save/load
-   AI panel

### AI

-   All nine actions
-   Schema validation
-   Whitelist enforcement
-   Scene context
-   Clarification handling
-   Engine API integration

### Backend

-   Persistence
-   WebSocket sequencing
-   Project/scene synchronization

------------------------------------------------------------------------

## Stage 3 --- Integration

-   Connect real C++/WASM API to frontend.
-   Connect AI action registry to real engine functions.
-   Route AI operations through the normal scene-operation pathway.
-   Add end-to-end tests.
-   Validate collaboration with AI-issued operations.

------------------------------------------------------------------------

## Stage 4 --- Finalization

-   UI polish
-   Reliability testing
-   Performance testing
-   AI evaluation
-   Deployment
-   Monitoring
-   Final demonstration game
-   Documentation and defense preparation

------------------------------------------------------------------------

# 20. Stretch Roadmap

Only after the MVP is stable:

### AI

``` text
MVP
  ↓
Scene-aware commands
  ↓
Multi-step planning
  ↓
RAG / Engine documentation
  ↓
AI debugging
  ↓
AI level generation
  ↓
Generative assets
  ↓
Text-to-3D
```

### Collaboration

``` text
Server-authoritative MVP
        ↓
Conflict handling
        ↓
CRDT / OT
        ↓
Offline editing
```

### Engine

``` text
Basic renderer
        ↓
Materials
        ↓
Lighting
        ↓
Physics
        ↓
Animation
        ↓
Advanced rendering
```

------------------------------------------------------------------------

# 21. Team Distribution

  Unit                      People Main Ownership
  ----------------------- -------- --------------------------------------------------
  Backend + Engine/WASM          2 C++ engine, WASM, persistence, WebSocket
  Frontend + Graphics            2 React editor, viewport integration, UI
  AI                             2 LLM, actions, validation, AI-engine integration
  QA                             1 Automated testing and system verification
  DevOps                         1 Docker, Terraform, Kubernetes, CI/CD, monitoring

------------------------------------------------------------------------

# 22. Final Project Boundary

The project is considered complete when the team can demonstrate:

``` text
Browser
   ↓
React Editor
   ↓
WASM C++ Engine
   ↓
3D Scene

+

Cloud Project Persistence
+
Real-time Collaboration
+
AI Natural Language Control
```

with the core AI path:

``` text
User Request
     ↓
LLM
     ↓
Structured Action
     ↓
Schema Validation
     ↓
Whitelist Validation
     ↓
Engine Function
     ↓
C++ Engine
     ↓
Updated Scene
```

This is the **mandatory AI contribution**.

Everything beyond this is an extension, not a dependency for project
completion.
