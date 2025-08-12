# TypeScript Playground Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                             TypeScript Website (Gatsby)                             │
│                          packages/typescriptlang-org                                │
└─────────────────────────────────────────────────────────────────────────────────────┘
                                          │
                                          │ hosts
                                          ▼
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              Playground UI Layer                                    │
│                           packages/playground                                       │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────────────────┐  │
│  │   Navigation    │  │     Editor      │  │         Sidebar                     │  │
│  │   Examples      │  │   (Monaco +     │  │      Plugin System                  │  │
│  │   Handbook      │  │   TypeScript)   │  │   ┌─────────────────────────────┐   │  │
│  │   Settings      │  │                 │  │   │ • JavaScript Output         │   │  │
│  └─────────────────┘  │   ┌─────────────┐ │  │   │ • Type Definitions         │   │  │
│           │            │   │   Sandbox   │ │  │   │ • Errors & Warnings        │   │  │
│           │            │   │    Core     │ │  │   │ • AST Viewer              │   │  │
│           │            │   └─────────────┘ │  │   │ • Custom Plugins          │   │  │
│           │            └─────────────────┘  │   └─────────────────────────────┘   │  │
│           └─────────────────────┬─────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────────────┘
                                  │
                                  │ uses
                                  ▼
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              Sandbox Core                                          │
│                           packages/sandbox                                         │
│  ┌─────────────────────────────────────────────────────────────────────────────┐   │
│  │                         Monaco Editor                                      │   │
│  │  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────────────────┐  │   │
│  │  │  Code Editor    │ │   Language      │ │    Virtual File System     │  │   │
│  │  │   - Syntax      │ │    Service      │ │  - Multi-file Support     │  │   │
│  │  │   - Completion  │ │   - Error Check │ │  - Import Resolution       │  │   │
│  │  │   - Formatting  │ │   - Type Info   │ │  - @filename Support       │  │   │
│  │  └─────────────────┘ └─────────────────┘ └─────────────────────────────┘  │   │
│  └─────────────────────────────────────────────────────────────────────────────┘   │
│                                    │                                               │
│                                    │ communicates with                             │
│                                    ▼                                               │
│  ┌─────────────────────────────────────────────────────────────────────────────┐   │
│  │                    TypeScript Compiler API                                 │   │
│  │  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────────────────┐  │   │
│  │  │   Parser        │ │   Type Checker  │ │      Code Generator        │  │   │
│  │  │  - AST Build    │ │  - Type Analysis│ │   - JS/DTS Emission        │  │   │
│  │  │  - Syntax Check │ │  - Error Report │ │   - Source Maps            │  │   │
│  │  └─────────────────┘ └─────────────────┘ └─────────────────────────────┘  │   │
│  └─────────────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────────────┘
                                  │
                                  │ offloads to
                                  ▼
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                            Playground Worker                                       │
│                         packages/playground-worker                                 │
│  ┌─────────────────────────────────────────────────────────────────────────────┐   │
│  │                         Web Worker                                         │   │
│  │  ┌─────────────────────────────────────────────────────────────────────┐   │   │
│  │  │                 TypeScript Language Service                        │   │   │
│  │  │              (Heavy computation off main thread)                   │   │   │
│  │  │  • Type checking    • IntelliSense    • Error reporting           │   │   │
│  │  └─────────────────────────────────────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────────────┘

Supporting Packages:
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│ Playground      │ │ Playground      │ │  Create Plugin  │ │ TypeScript VFS  │
│   Examples      │ │   Handbook      │ │    Template     │ │    Library      │
│ (Code samples)  │ │ (Documentation) │ │   (Scaffolding) │ │ (File System)   │
└─────────────────┘ └─────────────────┘ └─────────────────┘ └─────────────────┘

Data Flow:
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│ User Types  │───▶│   Monaco    │───▶│   Sandbox   │───▶│  Compiler   │
│    Code     │    │   Editor    │    │    Core     │    │    API      │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
                                                                  │
┌─────────────┐    ┌─────────────┐    ┌─────────────┐             │
│   Plugin    │◀───│ Playground  │◀───│   Results   │◀────────────┘
│  Updates    │    │     UI      │    │  (JS/DTS)   │
└─────────────┘    └─────────────┘    └─────────────┘
```

## Key Components Interaction

1. **User Input** → Monaco Editor captures keystrokes
2. **Monaco Editor** → Sends to Sandbox for TypeScript processing  
3. **Sandbox** → Coordinates with TypeScript Compiler via Worker
4. **Worker** → Performs heavy computation off main thread
5. **Compiler** → Returns results (JS, DTS, Errors)
6. **Playground UI** → Updates plugins with new results
7. **Plugins** → Display processed information to user

## Plugin System Architecture

```
Plugin Registration:
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Plugin    │───▶│ Playground  │───▶│ Tab System  │
│  Factory    │    │ Register    │    │ & Container │
└─────────────┘    └─────────────┘    └─────────────┘

Plugin Lifecycle:
willMount → didMount → modelChanged → modelChangedDebounce → willUnmount → didUnmount
```

This architecture provides a solid foundation for understanding how the TypeScript Playground works and how to extend it.