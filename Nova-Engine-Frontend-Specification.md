# Nova Engine --- Frontend Technical Specification

## 1. Goal

Build a lightweight browser-based editor that exposes the C++/WASM
engine without duplicating engine responsibilities.

The frontend is an **editor shell**, not a second game engine.

------------------------------------------------------------------------

## 2. MVP Technology Stack

``` text
React
TypeScript
Vite
Zustand
Tailwind CSS
```

### Optional libraries only when needed

``` text
Lucide React       → icons
dnd-kit             → asset drag/drop
Monaco Editor       → scripting editor, if scripting enters MVP
```

Avoid adding frameworks simply because they are popular.

------------------------------------------------------------------------

# 3. Frontend Architecture

``` mermaid
flowchart TB
    APP[React Application]

    APP --> TOOLBAR[Toolbar]
    APP --> HIER[Scene Hierarchy]
    APP --> INSPECT[Inspector]
    APP --> ASSETS[Asset Browser]
    APP --> AI[AI Panel]
    APP --> VIEW[WASM Viewport]

    APP --> STORE[Zustand State]
    APP --> API[REST / WebSocket]
    VIEW --> WASM[JS ↔ WASM Bridge]
    WASM --> ENGINE[C++ Engine]
```

------------------------------------------------------------------------

# 4. Required UI

## Toolbar

``` text
Select | Move | Rotate | Scale

Create | Delete | Duplicate

Undo | Redo | Save

             Play
```

## Scene Hierarchy

``` text
Scene
├── Main Camera
├── Directional Light
├── Player
├── Cube
└── House
```

## Inspector

``` text
Selected Object: Cube

Transform
  Position
  Rotation
  Scale

Material
  Color
```

## Asset Browser

``` text
Assets
├── Models
├── Textures
├── Materials
└── Audio
```

## AI Panel

``` text
┌─────────────────────────────┐
│ Nova AI                     │
├─────────────────────────────┤
│ Create a red cube.          │
│                             │
│ ✓ Cube_01 created           │
├─────────────────────────────┤
│ Ask AI...             Send  │
└─────────────────────────────┘
```

------------------------------------------------------------------------

# 5. State Management

Use Zustand for local editor state.

Suggested stores:

``` text
editorStore
sceneStore
selectionStore
aiStore
```

Server state can initially be handled through the existing
REST/WebSocket layer without introducing a large state-management
framework.

------------------------------------------------------------------------

# 6. WASM Integration

The frontend communicates with the engine through a small typed API.

``` mermaid
flowchart LR
    R[React Component] --> API[Typed Engine API]
    API --> WASM[WASM Bridge]
    WASM --> CPP[C++ Engine]
    CPP --> WASM
    WASM --> API
    API --> R
```

Example:

``` ts
engine.createObject(...)
engine.deleteObject(...)
engine.setTransform(...)
engine.setMaterial(...)
```

The exact binding mechanism is owned by the Engine/WASM team.

------------------------------------------------------------------------

# 7. AI Integration

The AI panel sends natural-language requests to the AI service.

``` mermaid
sequenceDiagram
    participant U as User
    participant F as Frontend
    participant A as AI Service
    participant E as Engine

    U->>F: Create a red cube
    F->>A: Request + scene context
    A->>A: LLM + validation
    A->>F: Validated action/result
    F->>E: Execute engine operation
    E-->>F: Scene event
    F-->>U: Updated viewport
```

The frontend should never execute arbitrary LLM output.

------------------------------------------------------------------------

# 8. Frontend ↔ Backend

### REST

Use for:

-   Authentication
-   Projects
-   Project metadata
-   Scene persistence
-   Asset metadata

### WebSocket

Use for:

-   Real-time scene operations
-   Collaboration events
-   AI/engine status events where appropriate

Use the browser's native WebSocket API unless a concrete requirement
justifies another library.

------------------------------------------------------------------------

# 9. Frontend Scope Rule

The frontend owns:

``` text
UI
Selection
Editor state
User input
API communication
Viewport integration
AI interface
```

The frontend does NOT own:

``` text
Rendering algorithms
ECS internals
Physics
Game loop
Engine memory
AI reasoning
Database
Server synchronization rules
```

------------------------------------------------------------------------

# 10. Definition of Done

The frontend MVP is complete when a user can:

1.  Open a project.
2.  See the WASM-rendered scene.
3.  Select an object.
4.  Inspect/edit basic properties.
5.  Create/delete/duplicate objects.
6.  Save/load a scene.
7.  See collaborative updates.
8.  Open the AI panel.
9.  Submit a natural-language command.
10. See the resulting engine operation reflected in the viewport.
