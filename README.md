# Nova Engine

> A browser-native collaborative 3D game development platform powered by a C++ engine compiled to WebAssembly.

Nova Engine is a graduation project that combines **C++ game-engine development, WebAssembly, web technologies, real-time collaboration, cloud infrastructure, and Generative AI** into one integrated platform.

## ✨ Overview

Nova Engine allows users to create and edit simple 3D game scenes directly in the browser.

The platform consists of:

* 🎮 **C++ / WASM Engine** — core scene, entity, and rendering systems
* 🖥️ **Web Editor** — lightweight React + TypeScript editor
* ☁️ **Backend** — projects, persistence, and real-time collaboration
* 🤖 **AI Assistant** — natural language converted into validated engine actions
* 🚀 **Cloud & DevOps** — deployment, CI/CD, and infrastructure
* 🧪 **QA** — automated testing across the platform

## 🏗️ Architecture

```text
                    ┌─────────────────┐
                    │   React Editor  │
                    │                 │
                    │ Hierarchy       │
                    │ Inspector       │
                    │ Viewport        │
                    │ AI Panel        │
                    └────────┬────────┘
                             │
                 ┌───────────┼───────────┐
                 │           │           │
                REST      WebSocket      AI
                 │           │           │
                 ▼           ▼           ▼
             ┌─────────────────────────────┐
             │           Backend           │
             │     Persistence / Sync      │
             └─────────────────────────────┘
                             │
                             │
                    ┌────────▼────────┐
                    │  C++ / WASM     │
                    │     Engine      │
                    └─────────────────┘
```

## 🤖 AI

The MVP uses a constrained Generative AI architecture:

```text
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
C++ / WASM Engine
```

The AI can perform a controlled set of scene operations such as:

* Create object
* Delete object
* Move object
* Rotate object
* Scale object
* Change material/color
* Add light
* Add camera
* Duplicate object

The LLM **does not execute arbitrary code or directly access engine memory**.

## 🛠️ Main Technologies

### Engine

* C++
* WebAssembly
* Emscripten

### Frontend

* React
* TypeScript
* Vite
* Zustand
* Tailwind CSS

### Backend

* REST API
* WebSocket
* PostgreSQL

### AI

* Python
* FastAPI
* LLM API
* Pydantic / JSON Schema
* Structured actions and tool/function calling

### Infrastructure

* Docker
* Terraform
* Kubernetes
* CI/CD
* Monitoring

## 📁 Repository Documentation

Detailed project documentation is available in:

* [`Nova-Engine-Final-Technical-Plan.md`](./Nova-Engine-Final-Technical-Plan.md)
* [`Nova-Engine-Frontend-Specification.md`](./Nova-Engine-Frontend-Specification.md)
* [`Nova-Engine-AI-MVP-Specification.md`](./Nova-Engine-AI-MVP-Specification.md)

## 🎯 Project Scope

The project focuses on delivering a complete, demonstrable **MVP** within the academic year.

Advanced capabilities such as autonomous multi-step AI planning, CRDT collaboration, text-to-3D generation, and advanced rendering are treated as **stretch goals**.

## 👥 Team

Nova Engine is developed by a team of **8 engineers** across:

* Engine / WASM
* Frontend / Graphics
* AI
* Backend
* DevOps
* QA

## 📌 Status

🚧 **Under Development**

This repository contains the ongoing development of the Nova Engine graduation project.
