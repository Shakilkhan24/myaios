# Building an AI Operating System (AIOS): A Complete Technical Guide

> **Purpose:** This document is a semantic knowledge gist for LLM agents to deeply understand the architecture, design philosophy, folder structure, and implementation patterns of an AI Operating System — enabling the LLM to reason about and reconstruct one from scratch.

---

## Table of Contents

1. [What Is an AI Operating System?](#1-what-is-an-ai-operating-system)
2. [Core Design Philosophy](#2-core-design-philosophy)
3. [System Architecture Overview](#3-system-architecture-overview)
4. [The OS Analogy Mapped to AI](#4-the-os-analogy-mapped-to-ai)
5. [Canonical Folder Structure](#5-canonical-folder-structure)
6. [Layer-by-Layer Deep Dive](#6-layer-by-layer-deep-dive)
   - 6.1 [Kernel Layer — LLM Core](#61-kernel-layer--llm-core)
   - 6.2 [System Call Layer](#62-system-call-layer)
   - 6.3 [Scheduler Layer](#63-scheduler-layer)
   - 6.4 [Memory Subsystem](#64-memory-subsystem)
   - 6.5 [Storage Subsystem](#65-storage-subsystem)
   - 6.6 [Tool Manager](#66-tool-manager)
   - 6.7 [Hook / Reactive Layer](#67-hook--reactive-layer)
   - 6.8 [Context Manager](#68-context-manager)
   - 6.9 [Agent Runtime & Factory](#69-agent-runtime--factory)
   - 6.10 [Terminal / Intent Router](#610-terminal--intent-router)
   - 6.11 [Configuration Manager](#611-configuration-manager)
7. [Data Flow: End-to-End Request Lifecycle](#7-data-flow-end-to-end-request-lifecycle)
8. [Key Design Patterns](#8-key-design-patterns)
9. [Every Critical File Explained](#9-every-critical-file-explained)
10. [Inter-Component Communication](#10-inter-component-communication)
11. [Concurrency Model](#11-concurrency-model)
12. [Extending the OS: Adding a New Subsystem](#12-extending-the-os-adding-a-new-subsystem)
13. [Bootstrap Sequence (How the Kernel Starts)](#13-bootstrap-sequence-how-the-kernel-starts)
14. [API Surface (Runtime REST Endpoints)](#14-api-surface-runtime-rest-endpoints)
15. [Implementation Checklist](#15-implementation-checklist)

---

## 1. What Is an AI Operating System?

A traditional OS manages hardware resources (CPU, RAM, disk, I/O) and presents them to applications through a clean, safe abstraction called **system calls**. Multiple programs can share these resources concurrently through the kernel.

An **AI Operating System (AIOS)** applies this exact pattern to LLM-powered agents:

- The **LLM** is the CPU — the computational resource that executes "reasoning instructions."
- **Memory** (short-term, long-term, semantic) is RAM and disk.
- **Tools** (APIs, code executors, browsers) are peripheral devices.
- **Storage** (file system) is the persistent disk layer.
- **Agents** are user processes — they submit jobs and get results back.
- **The scheduler** governs which agent gets LLM access at what time.
- **System calls** are the typed, safe interface between agents and the kernel.

The key insight: **LLMs are expensive, shared, stateless resources**. Many agents need them simultaneously. The OS must multiplex access fairly, track state, handle preemption, and ensure no agent monopolizes the system.

---

## 2. Core Design Philosophy

| Principle | Description |
|-----------|-------------|
| **Separation of concerns** | Each subsystem (LLM, memory, storage, tool) is isolated behind a typed syscall interface. |
| **Resource multiplexing** | Multiple agents share one LLM via a scheduler, exactly like processes share a CPU. |
| **Pluggable providers** | Memory, LLM backends, and storage are swappable via the Provider/Factory pattern. |
| **Reactive hooks** | Components expose reactive primitives (queues + functions) so the kernel can wire them without tight coupling. |
| **Thread-per-subsystem** | Each resource type (LLM, memory, storage, tool) gets its own processing thread, allowing parallel execution across subsystems. |
| **Typed query/response** | Every operation is expressed as a typed `Query` object (`LLMQuery`, `MemoryQuery`, `StorageQuery`, `ToolQuery`) and returns a typed `Response`. |
| **Timing observability** | Every syscall records `created_time`, `start_time`, `end_time` to compute waiting time and turnaround time (classic OS metrics). |

---

## 3. System Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        USER / AGENT LAYER                       │
│   Agent A        Agent B        Agent C        Terminal CLI     │
│  (cerebrum)     (cerebrum)     (cerebrum)     (intent router)   │
└──────────────────────────┬──────────────────────────────────────┘
                           │  submit typed Query objects
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                     SYSCALL EXECUTOR LAYER                      │
│   SyscallExecutor.execute_request(agent_name, query)            │
│   Routes → LLMSyscall / MemorySyscall / StorageSyscall /        │
│             ToolSyscall                                         │
│   Measures timing: created_time, start_time, end_time           │
└──────┬────────────┬─────────────┬──────────────┬───────────────┘
       │            │             │              │
       ▼            ▼             ▼              ▼
  LLM Queue   Memory Queue   Storage Queue  Tool Queue
  (thread-safe FIFO queues — one per resource type)
       │            │             │              │
       ▼            ▼             ▼              ▼
┌─────────────────────────────────────────────────────────────────┐
│                        SCHEDULER LAYER                          │
│   FIFOScheduler (process_llm_requests thread)                   │
│   RRScheduler   (round-robin with time slicing + context save)  │
│   Each scheduler runs 4 threads: LLM / Memory / Storage / Tool  │
└──────┬────────────┬─────────────┬──────────────┬───────────────┘
       │            │             │              │
       ▼            ▼             ▼              ▼
┌──────────┐ ┌──────────┐ ┌──────────────┐ ┌──────────────┐
│LLMAdapter│ │MemoryMgr │ │ StorageMgr   │ │  ToolManager │
│          │ │          │ │              │ │              │
│litellm / │ │in-house /│ │ LSFS +       │ │ AutoTool +   │
│hflocal / │ │ mem0 /   │ │ vector DB    │ │ MCP server   │
│openai /  │ │ zep      │ │              │ │              │
│ollama    │ │          │ │              │ │              │
└──────────┘ └──────────┘ └──────────────┘ └──────────────┘
                           KERNEL / SUBSYSTEM LAYER
┌─────────────────────────────────────────────────────────────────┐
│                     HOOK / CONFIG LAYER                         │
│   Global queue stores, reactive hook functions, config manager  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. The OS Analogy Mapped to AI

| Traditional OS Concept | AIOS Equivalent | Implementation |
|------------------------|-----------------|----------------|
| CPU | LLM (language model) | `LLMAdapter` wrapping litellm/openai/huggingface |
| Process | Agent | Agent class from `cerebrum` package |
| System Call | `SyscallExecutor.execute_request()` | `aios/syscall/syscall.py` |
| Kernel | AIOS kernel server | `runtime/launch.py` (FastAPI app) |
| Scheduler | `FIFOScheduler` / `RRScheduler` | `aios/scheduler/` |
| RAM | In-process memory manager | `aios/memory/` |
| Disk | Storage manager + filesystem | `aios/storage/` |
| Device Drivers | Tool adapters | `aios/tool/` |
| IPC / Shared Memory | Request queues | `aios/hooks/stores/queue.py` |
| Process Table | `processes` store | `aios/hooks/stores/processes.py` |
| Context Switch | `SimpleContextManager.save_context()` | `aios/context/simple_context.py` |
| Interrupt | Queue event set (`syscall.event.set()`) | Thread event in `Syscall` base class |
| Init / Bootstrap | `runtime/launch.py` startup sequence | FastAPI lifespan / `/initialize/*` endpoints |

---

## 5. Canonical Folder Structure

```
aios/                              ← Root Python package
│
├── __init__.py
│
├── config/                        ← Global configuration manager
│   └── config_manager.py          ← Singleton Config object; reads YAML/env
│
├── syscall/                       ← SYSTEM CALL LAYER (the kernel API surface)
│   ├── __init__.py                ← exports Syscall base class
│   ├── syscall.py                 ← SyscallExecutor: main dispatch & timing
│   ├── llm.py                     ← LLMSyscall(agent_name, LLMQuery)
│   ├── memory.py                  ← MemorySyscall(agent_name, MemoryQuery)
│   ├── storage.py                 ← StorageSyscall + storage_syscalls tool list
│   ├── tool.py                    ← ToolSyscall(agent_name, ToolQuery)
│   ├── factory.py                 ← Syscall factory helpers
│   ├── schema.py                  ← Shared pydantic schemas
│   └── types/
│       ├── __init__.py
│       └── syscall.py             ← Request / Message base pydantic models
│
├── scheduler/                     ← SCHEDULER LAYER
│   ├── __init__.py
│   ├── base.py                    ← BaseScheduler: thread management primitives
│   ├── fifo_scheduler.py          ← FIFOScheduler: batched LLM + sequential others
│   └── rr_scheduler.py            ← RRScheduler: round-robin with time slices
│
├── llm_core/                      ← KERNEL / LLM SUBSYSTEM
│   ├── adapter.py                 ← LLMAdapter: multi-backend LLM router
│   ├── local.py                   ← HfLocalBackend: HuggingFace local inference
│   ├── routing.py                 ← RouterStrategy: Sequential / Smart routing
│   └── utils.py                   ← Message merging, tool call decoding helpers
│
├── memory/                        ← MEMORY SUBSYSTEM
│   ├── manager.py                 ← MemoryManager: high-level memory API
│   ├── base.py                    ← BaseMemory abstract class
│   ├── note.py                    ← MemoryNote dataclass (id, content, keywords, tags)
│   ├── retrievers.py              ← Semantic retrieval utilities
│   ├── context_injector.py        ← Injects relevant memories into LLM prompts
│   ├── conversation_extractor.py  ← Async extraction of conversation memories
│   └── providers/
│       ├── __init__.py
│       ├── base.py                ← MemoryProvider abstract interface
│       ├── factory.py             ← ProviderFactory: create by name
│       ├── in_house.py            ← InHouseProvider: built-in vector + keyword store
│       ├── mem0.py                ← Mem0Provider: mem0.ai integration
│       └── zep.py                 ← ZepProvider: Zep memory integration
│
├── storage/                       ← STORAGE SUBSYSTEM
│   ├── __init__.py
│   ├── storage.py                 ← StorageManager wrapping filesystem
│   └── filesystem/
│       ├── lsfs.py                ← LSFS: LLM-Semantic File System
│       └── vector_db.py           ← VectorDB: semantic file search
│
├── tool/                          ← TOOL SUBSYSTEM (peripheral device drivers)
│   ├── __init__.py
│   ├── manager.py                 ← ToolManager: tool dispatch + MCP server lifecycle
│   ├── mcp_server.py              ← MCP (Model Context Protocol) server
│   └── virtual_env/               ← Sandboxed execution environments
│       ├── controllers/           ← Python interpreter controllers
│       ├── server/                ← Virtual env HTTP server
│       ├── providers/             ← Cloud VM providers (AWS, GCP, Azure, Docker, etc.)
│       ├── accessibility_tree_wrap/
│       └── evaluators/            ← Task evaluation metrics
│
├── hooks/                         ← REACTIVE HOOK / IPC LAYER
│   ├── __init__.py
│   ├── modules/                   ← Hook factories (return queues + managers)
│   │   ├── llm.py                 ← useCore(), useLLMRequestQueue()
│   │   ├── memory.py              ← useMemoryManager(), useMemoryRequestQueue()
│   │   ├── storage.py             ← useStorageManager(), useStorageRequestQueue()
│   │   ├── tool.py                ← useToolManager(), useToolRequestQueue()
│   │   ├── agent.py               ← useFactory() — agent lifecycle
│   │   └── scheduler.py           ← fifo_scheduler_nonblock(), rr_scheduler_nonblock()
│   ├── types/                     ← Type aliases for queue operations
│   │   ├── llm.py                 ← LLMRequestQueue, LLMRequestQueueAddMessage, etc.
│   │   ├── memory.py
│   │   ├── storage.py
│   │   ├── tool.py
│   │   ├── agent.py
│   │   ├── scheduler.py
│   │   └── __init__.py
│   ├── stores/                    ← Singleton global state
│   │   ├── __init__.py
│   │   ├── _global.py             ← Global queue instances (created at import time)
│   │   ├── queue.py               ← Queue store primitives (addMessage, getMessage)
│   │   └── processes.py           ← Process registry (agent process table)
│   └── utils/
│       ├── validate.py            ← Pydantic validation decorator
│       └── utils.py               ← Miscellaneous hook utilities
│
├── context/                       ← CONTEXT MANAGER (for RR preemption)
│   ├── base.py                    ← BaseContextManager abstract class
│   └── simple_context.py          ← SimpleContextManager: save/restore LLM state
│
├── terminal/                      ← TERMINAL / SHELL LAYER
│   ├── __init__.py
│   └── intent_router.py           ← Routes natural language commands to syscalls
│
└── utils/                         ← SHARED UTILITIES
    ├── logger.py                  ← SchedulerLogger and component loggers
    ├── id_generator.py            ← Tool call ID generation
    └── commands/                  ← CLI command helpers

runtime/                           ← KERNEL ENTRY POINTS
├── launch.py                      ← Main FastAPI kernel server
├── launch_kernel.sh               ← Shell bootstrap script
└── run_terminal.py                ← Interactive terminal session

scripts/                           ← OPERATOR TOOLS
├── list_agents.py
├── run_agent.sh
├── launch_vllm.sh
├── launch_sglang.sh
└── run_litellm.py

tests/                             ← TEST SUITE
└── modules/                       ← Per-module integration tests

docs/                              ← DOCUMENTATION
├── CONTRIBUTE.md
├── memory-providers.md
└── assets/

aios-rs/                           ← RUST IMPLEMENTATION (performance-critical paths)
├── Cargo.toml
└── src/

Dockerfile                         ← Container image definition
requirements.txt                   ← Python dependencies
install/install.sh                 ← One-shot environment setup
```

---

## 6. Layer-by-Layer Deep Dive

### 6.1 Kernel Layer — LLM Core

**File:** `aios/llm_core/adapter.py`

The `LLMAdapter` is the heart of the kernel. It is a **multi-backend LLM router** that can load-balance across multiple model endpoints simultaneously.

**Key responsibilities:**
- Accept a list of LLM configurations (OpenAI, Ollama, HuggingFace local, vLLM, SGLang, custom endpoints).
- Route individual `LLMQuery` requests to the appropriate backend using a routing strategy.
- Execute single requests (`execute_llm_syscall`) and batched requests (`execute_llm_syscalls`) for throughput.
- Handle response decoding: text responses, tool calls, JSON-structured outputs.

**Backend types supported:**
| Backend | Use Case |
|---------|----------|
| `openai` | OpenAI API (GPT-4, etc.) |
| `ollama` | Local Ollama server |
| `hflocal` | HuggingFace model loaded directly into GPU memory |
| `vllm` | vLLM server for high-throughput inference |
| `sglang` | SGLang server |
| `litellm` | Universal LLM proxy (default) |

**Routing strategies** (`aios/llm_core/routing.py`):
- `SequentialRouting`: assign requests round-robin across backends.
- `SmartRouting`: assign based on load, latency, or capability.

```python
# Example: Creating an LLMAdapter with two backends
adapter = LLMAdapter(
    llm_configs=[
        {"name": "gpt-4o-mini", "backend": "openai"},
        {"name": "qwen2.5:7b", "backend": "ollama", "hostname": "http://localhost:11434"}
    ],
    log_mode="console",
    use_context_manager=True,   # enables context save/restore for RR scheduler
    routing_strategy=RouterStrategy.Sequential
)
```

**Key methods:**
- `execute_llm_syscall(syscall)` → single LLM call, returns `LLMResponse`
- `execute_llm_syscalls(batch)` → batched LLM calls, one per agent in batch
- `execute_llm_syscall_with_tool(syscall)` → LLM call with tool use
- `execute_llm_syscall_with_json_output(syscall)` → LLM with structured JSON output

---

### 6.2 System Call Layer

**File:** `aios/syscall/syscall.py`

The `SyscallExecutor` is the **single entry point for all agent-to-kernel communication**. It:

1. Wraps the query in a typed `Syscall` object (LLM/Memory/Storage/Tool).
2. Assigns a unique syscall ID.
3. Records `created_time`.
4. Enqueues the syscall to the appropriate global request queue.
5. **Blocks** the calling thread (via `syscall.start(); syscall.join()`) until the scheduler processes it.
6. Returns a dict with `response`, `start_times`, `end_times`, `waiting_times`, `turnaround_times`.

**Syscall types:**
| Syscall Class | File | Handles |
|---------------|------|---------|
| `LLMSyscall` | `syscall/llm.py` | LLMQuery → LLM inference |
| `MemorySyscall` | `syscall/memory.py` | MemoryQuery → store/retrieve/update memories |
| `StorageSyscall` | `syscall/storage.py` | StorageQuery → file read/write/search |
| `ToolSyscall` | `syscall/tool.py` | ToolQuery → external tool/API execution |

**Base Syscall class** (in `aios/syscall/__init__.py`):
- Inherits from `threading.Thread` — it IS a thread that blocks on `event.wait()`.
- Fields: `agent_name`, `query`, `status`, `pid`, `source`, `created_time`, `start_time`, `end_time`, `response`.
- `set_status()` transitions: `"active"` → `"executing"` → `"done"`.
- `event` is a `threading.Event` — set by the scheduler when done, unblocking the agent.

**High-level dispatch logic** (`execute_request`):
```
LLMQuery + action_type="chat"            → inject context → execute_llm_syscall → extract conversation
LLMQuery + action_type="call_tool"       → LLM decides tool → execute_tool_syscall
LLMQuery + action_type="operate_file"    → LLM parses op → execute_storage_syscall
LLMQuery + action_type="chat_with_json_output" → execute_llm_syscall with JSON schema
ToolQuery                                → execute_tool_syscall directly
MemoryQuery + operation="add_agentic_memory" → analyze → retrieve similar → evolve → add
MemoryQuery + other operations           → execute_memory_syscall directly
StorageQuery                             → execute_storage_syscall directly
```

**Creating the executor:**
```python
# The factory function useSysCall() is the conventional entry point
execute_request, SysCallWrapper, syscall_executor = useSysCall()
# execute_request(agent_name, query) is called by agents at runtime
```

---

### 6.3 Scheduler Layer

**Files:** `aios/scheduler/base.py`, `fifo_scheduler.py`, `rr_scheduler.py`

The scheduler is the **OS process scheduler** — it decides which agent's syscall gets processed next.

**BaseScheduler** (`base.py`):
- Holds references to: `llm` (LLMAdapter), `memory_manager`, `storage_manager`, `tool_manager`.
- Holds queue getter functions: `get_llm_syscall`, `get_memory_syscall`, `get_storage_syscall`, `get_tool_syscall`.
- `start_processing_threads(fn_list)` — launches each function as a daemon `Thread`.
- `stop_processing_threads()` — graceful shutdown.

**FIFOScheduler** (`fifo_scheduler.py`):
- Maintains 4 processing threads, one per resource type.
- **LLM thread**: collects requests during a `batch_interval` (default 1s), then sends the batch to `llm.execute_llm_syscalls()` for parallel execution.
- **Memory/Storage/Tool threads**: process one request at a time from each queue via `self.memory_manager.address_request(syscall)`.
- No preemption — a request runs to completion before the next one starts.

**RRScheduler** (`rr_scheduler.py`):
- Adds a `time_slice` per request (default 1 second).
- Uses `SimpleContextManager` to **save LLM generation state** mid-stream when the time slice expires.
- Resumes a paused request in the next round by **restoring the saved context**.
- Provides fairness guarantees when many agents compete for the LLM.

**When to use which:**
- Use `FIFOScheduler` for simple deployments, batch workloads, or when LLM calls are already fast.
- Use `RRScheduler` when long-running agents must not starve short-burst agents.

---

### 6.4 Memory Subsystem

**Files:** `aios/memory/`

The memory subsystem implements **long-term semantic memory** for agents — analogous to a database that "thinks."

**MemoryManager** (`manager.py`):
- High-level API; delegates to a configured `MemoryProvider`.
- Maintains `known_user_ids: Set[str]` — tracks which users have memories, enabling cross-agent shared retrieval.
- Operations: `add_memory`, `retrieve_memory`, `update_memory`, `remove_memory`, `get_memory`.

**Provider hierarchy:**
```
MemoryProvider (abstract, base.py)
├── InHouseProvider (in_house.py)   ← Default: local vector + keyword store
├── Mem0Provider (mem0.py)          ← mem0.ai cloud memory service
└── ZepProvider (zep.py)            ← Zep temporal knowledge graph
```

**MemoryNote** (`note.py`) — the atomic memory unit:
```python
@dataclass
class MemoryNote:
    id: str              # UUID
    content: str         # The raw memory text
    keywords: List[str]  # Extracted keywords (by LLM analysis)
    context: str         # One-sentence context summary
    tags: List[str]      # Classification tags
    created_at: float    # Timestamp
    agent_name: str      # Which agent created this
    user_id: Optional[str]
```

**Agentic memory pipeline** (the most sophisticated operation):
```
User message arrives
        ↓
execute_memory_content_analyze()   ← LLM extracts keywords, context, tags
        ↓
retrieve_memory_raw()              ← Find semantically similar existing memories
        ↓
execute_memory_evolve()            ← LLM decides: should old memories be updated/merged?
        ↓
update affected memories           ← Apply evolutions to existing notes
        ↓
add_memory()                       ← Store the new memory note
```

**ContextInjector** (`context_injector.py`):
- Before each LLM call, retrieves relevant memories for the agent.
- Prepends them to the conversation as a system message: `"Here is what you remember: ..."`
- Resolves `user_id` from memory metadata so the right memories are retrieved.

**ConversationExtractor** (`conversation_extractor.py`):
- After each LLM chat response, asynchronously extracts and stores the conversation turn as a memory.
- Runs in a background thread so it doesn't block the agent.

---

### 6.5 Storage Subsystem

**Files:** `aios/storage/`

**StorageManager** (`storage.py`):
- Wraps a filesystem implementation (currently `LSFS`).
- Normalizes all results to `StorageResponse(response_message: str, finished: bool)`.

**LSFS — LLM Semantic File System** (`storage/filesystem/lsfs.py`):
- A file system where files can be retrieved by **semantic content**, not just name/path.
- Supports: `read`, `write`, `create`, `delete`, `list`, `search` operations.
- Integrates an optional `VectorDB` for embedding-based file search.
- All operations go through `address_request(syscall)` → dispatches by `operation_type`.

**VectorDB** (`storage/filesystem/vector_db.py`):
- Stores embeddings of file content.
- Enables queries like: "find files about machine learning experiments."
- Backed by a local vector store (e.g., FAISS or ChromaDB).

**Storage syscall tools** (`syscall/storage.py`):
- `storage_syscalls` is a list of tool definitions (JSON schema) that the LLM can use to decide which file operation to perform when `action_type="operate_file"`.
- Operations available as tools: `read_file`, `write_file`, `create_file`, `delete_file`, `list_files`, `search_files`.

---

### 6.6 Tool Manager

**Files:** `aios/tool/`

**ToolManager** (`manager.py`):
- Manages the lifecycle of external tools (APIs, code executors, web browsers, etc.).
- On initialization, automatically starts the `mcp_server.py` as a subprocess.
- `address_request(tool_syscall)` → looks up the tool, calls it, returns `ToolResponse`.
- Uses `AutoTool` from `cerebrum` to auto-discover and load tool implementations.
- Tracks `tool_conflict_map` to prevent concurrent conflicting tool calls.

**MCP Server** (`tool/mcp_server.py`):
- Implements the **Model Context Protocol (MCP)** — a standardized protocol for LLM↔tool communication.
- Runs as a separate process, exposes tools over HTTP/SSE.
- Allows tools to be hot-plugged without restarting the kernel.

**Virtual Environments** (`tool/virtual_env/`):
- Sandboxed execution environments for running agent-generated code safely.
- Providers: Docker, AWS, GCP, Azure, VirtualBox, VMware.
- Controllers: Python interpreter sandboxes.
- Evaluators: Metrics for measuring task success inside the sandbox.

**Tool execution flow:**
```
Agent submits ToolQuery(tool_calls=[{name: "calculator", arguments: {...}}])
        ↓
ToolSyscall enqueued to global_tool_req_queue
        ↓
Scheduler calls tool_manager.address_request(syscall)
        ↓
ToolManager loads the tool via AutoTool
        ↓
Tool executes, returns result
        ↓
ToolResponse(response_message, finished=True) returned to agent
```

---

### 6.7 Hook / Reactive Layer

**Files:** `aios/hooks/`

The hook layer is inspired by React hooks — it provides **factory functions** that create and configure subsystem instances along with their request queues.

**Pattern:** Every `use*` function returns a tuple of: `(instance, get_message_fn, add_message_fn, is_empty_fn)`.

```python
# Example: initializing the LLM subsystem
llm = useCore({
    "llm_configs": [...],
    "log_mode": "console"
})
llm_queue, get_llm, add_llm, llm_empty = useLLMRequestQueue()
```

**Hook modules** (`hooks/modules/`):
| Module | Hook Function | Returns |
|--------|---------------|---------|
| `llm.py` | `useCore(params)` | `LLMAdapter` |
| `llm.py` | `useLLMRequestQueue()` | queue + CRUD fns |
| `memory.py` | `useMemoryManager(params)` | `MemoryManager` |
| `memory.py` | `useMemoryRequestQueue()` | queue + CRUD fns |
| `storage.py` | `useStorageManager(params)` | `StorageManager` |
| `storage.py` | `useStorageRequestQueue()` | queue + CRUD fns |
| `tool.py` | `useToolManager(params)` | `ToolManager` |
| `tool.py` | `useToolRequestQueue()` | queue + CRUD fns |
| `agent.py` | `useFactory()` | agent factory |
| `scheduler.py` | `fifo_scheduler_nonblock(...)` | `FIFOScheduler` |
| `scheduler.py` | `rr_scheduler_nonblock(...)` | `RRScheduler` |

**Global stores** (`hooks/stores/_global.py`):
- Instantiates the four global queues at import time.
- These are singletons: `global_llm_req_queue`, `global_memory_req_queue`, etc.
- Every syscall enqueues to these; every scheduler reads from these.
- The global functions `global_llm_req_queue_add_message()` etc. are used by the `SyscallExecutor`.

**Queue store** (`hooks/stores/queue.py`):
- Implements thread-safe queue operations backed by Python's `queue.Queue`.
- `addMessage(queue, item)` → non-blocking put.
- `getMessage(queue)` → blocks with timeout, raises `Empty` on timeout.

---

### 6.8 Context Manager

**Files:** `aios/context/`

Used by the `RRScheduler` to implement **preemptive multitasking**.

**BaseContextManager** (`context/base.py`):
- Abstract interface: `save_context(...)` and `load_context(pid)`.

**SimpleContextManager** (`context/simple_context.py`):
- `save_context(model_name, model, messages, tools, ..., pid, time_limit, ...)`:
  - Starts an LLM call with a `time_limit`.
  - If the call finishes within the time limit → returns `(response, finished=True)`.
  - If the time limit expires mid-generation → pauses, saves partial state keyed by `pid`, returns `(partial_response, finished=False)`.
- `load_context(pid)` → retrieves the saved messages + partial response to resume.
- When `finished=False`, the `SyscallExecutor` re-enqueues the syscall for the next scheduling round.

---

### 6.9 Agent Runtime & Factory

**Files:** `aios/hooks/modules/agent.py`, `cerebrum` package (external)

Agents are defined in the `cerebrum` package (a separate installable library). The AIOS kernel:
1. Uses `useFactory()` to get an `AgentFactory`.
2. The factory can instantiate any agent by name/config.
3. Each agent receives an `execute_request` function bound to the kernel's `SyscallExecutor`.
4. Agents call `execute_request(self.agent_name, query)` to interact with the kernel.

**Agent lifecycle:**
```
POST /submit_agent {agent_id, agent_config}
        ↓
AgentFactory.submit(agent_id, agent_config)
        ↓
Agent thread started
        ↓
Agent runs its task, calling execute_request() for each LLM/tool/memory/storage op
        ↓
Agent thread exits, results available via /get_agent_result
```

---

### 6.10 Terminal / Intent Router

**File:** `aios/terminal/intent_router.py`

A natural language command-line interface to the AIOS kernel:
- Accepts free-text commands like: `"Create a file called notes.txt with my meeting agenda"`
- Routes them to the appropriate syscall type by **intent classification**.
- Uses the LLM itself to parse the intent and extract parameters.
- Useful for interactive terminal sessions (`runtime/run_terminal.py`).

---

### 6.11 Configuration Manager

**File:** `aios/config/config_manager.py`

Singleton `Config` object that:
- Reads from YAML config files and environment variables.
- Provides typed accessors: `get_memory_config()`, `get_storage_config()`, `get_llm_config()`, `get_mcp_server_script_path()`.
- Enables runtime reconfiguration of subsystems without code changes.

---

## 7. Data Flow: End-to-End Request Lifecycle

Here is the complete trace of a single `LLMQuery` from agent submission to response:

```
1. AGENT CODE
   agent.execute_request("my_agent", LLMQuery(messages=[...], action_type="chat"))
                                    │
                                    ▼
2. SYSCALL EXECUTOR (aios/syscall/syscall.py)
   SyscallExecutor.execute_request("my_agent", LLMQuery)
   → action_type="chat", so:
     a. context_injector.inject("my_agent", query)  [optional: prepend memories]
     b. execute_llm_syscall("my_agent", query)
                                    │
                                    ▼
3. SYSCALL CREATION
   syscall = LLMSyscall("my_agent", query)
   syscall.set_status("active")
   syscall.set_created_time(now)
   syscall.set_pid(syscall_id)
                                    │
                                    ▼
4. QUEUE ENQUEUE
   global_llm_req_queue_add_message(syscall)
                                    │
                                    ▼
5. THREAD BLOCKING
   syscall.start()   ← starts Thread, which blocks on event.wait()
   syscall.join()    ← main agent thread blocks here until event is set
                                    │
                                    ▼  (meanwhile, in scheduler thread)
6. SCHEDULER (aios/scheduler/fifo_scheduler.py)
   process_llm_requests() running in background thread:
     time.sleep(batch_interval)
     batch = drain all items from LLM queue
     llm.execute_llm_syscalls(batch)
                                    │
                                    ▼
7. LLM ADAPTER (aios/llm_core/adapter.py)
   Routes each syscall to a backend (openai/ollama/hf/etc.)
   Makes the actual API call (litellm.completion or openai SDK)
   Decodes response: text / tool_calls / JSON
   syscall.set_response(LLMResponse(...))
   syscall.set_status("done")
   syscall.set_end_time(now)
   syscall.event.set()   ← unblocks the agent thread
                                    │
                                    ▼
8. BACK IN SYSCALL EXECUTOR
   completed_response = syscall.get_response()
   computes timing metrics
   returns {"response": LLMResponse, "waiting_times": [...], ...}
                                    │
                                    ▼
9. POST-PROCESSING
   conversation_extractor.extract_async(...)  [async memory storage]
                                    │
                                    ▼
10. AGENT RECEIVES RESPONSE
    response = execute_request(...)["response"]
    # LLMResponse.response_message contains the assistant text
```

---

## 8. Key Design Patterns

### Pattern 1: Typed Query/Response Protocol

Every operation is expressed as a typed query object from the `cerebrum` package:

```python
# LLM inference
LLMQuery(messages=[...], action_type="chat", tools=[...], response_format={...})
LLMResponse(response_message="...", tool_calls=[...], finished=True)

# Memory operations
MemoryQuery(operation_type="add_memory", params={"content": "...", "user_id": "..."})
MemoryResponse(response_message="stored", finished=True)

# Storage operations
StorageQuery(operation_type="write_file", params={"path": "...", "content": "..."})
StorageResponse(response_message="written", finished=True)

# Tool calls
ToolQuery(tool_calls=[{"name": "search", "arguments": {"query": "..."}}])
ToolResponse(response_message="results...", finished=True)
```

### Pattern 2: Thread-per-Subsystem with Event Signaling

```python
# Syscall object IS a thread
class Syscall(threading.Thread):
    def __init__(self, agent_name, query):
        super().__init__()
        self.event = threading.Event()  # signal when done
    
    def run(self):
        self.event.wait()  # blocks until scheduler calls event.set()

# SyscallExecutor enqueues then blocks
queue.put(syscall)
syscall.start()  # starts the blocking thread
syscall.join()   # waits for it to be signaled
response = syscall.get_response()
```

### Pattern 3: Provider Factory for Pluggability

```python
class ProviderFactory:
    PROVIDERS = {
        "in-house": InHouseProvider,
        "mem0": Mem0Provider,
        "zep": ZepProvider,
    }
    
    @classmethod
    def create(cls, provider_type: str, config: Dict) -> MemoryProvider:
        provider_class = cls.PROVIDERS.get(provider_type)
        return provider_class(config)

# At runtime: swap providers without changing consumer code
memory_manager = MemoryManager(provider="mem0")
```

### Pattern 4: Hook Factories (React-inspired)

```python
# Each subsystem exposes a "use" function that wires up all its pieces
def useLLMRequestQueue():
    queue = LLMRequestQueue()
    QueueStore.REQUEST_QUEUE["llm"] = queue
    
    def getMessage(): return QueueStore.getMessage(queue)
    def addMessage(msg): return QueueStore.addMessage(queue, msg)
    def isEmpty(): return QueueStore.isEmpty(queue)
    
    return queue, getMessage, addMessage, isEmpty
```

### Pattern 5: Timing Observability

Every syscall records OS-style performance metrics:
```python
waiting_time = start_time - created_time       # time in queue
turnaround_time = end_time - created_time      # total time from submit to done
execution_time = end_time - start_time         # actual processing time
```

### Pattern 6: LLM-Assisted Operations

The OS uses the LLM itself to parse intent and structure for complex operations:
- **File operations**: LLM parses natural language into specific `read_file`/`write_file` tool calls.
- **Memory analysis**: LLM extracts `keywords`, `context`, `tags` from raw content.
- **Memory evolution**: LLM decides whether new memories should update/merge with existing ones.
- **Intent routing**: Terminal uses LLM to classify user commands into syscall types.

---

## 9. Every Critical File Explained

| File | Role | Key Class/Function |
|------|------|--------------------|
| `runtime/launch.py` | Kernel entry point; FastAPI server | `app`, startup sequence, `/initialize/*` routes |
| `aios/syscall/syscall.py` | Master syscall dispatcher | `SyscallExecutor`, `create_syscall_executor()` |
| `aios/syscall/llm.py` | LLM syscall wrapper | `LLMSyscall` |
| `aios/syscall/memory.py` | Memory syscall wrapper | `MemorySyscall` |
| `aios/syscall/storage.py` | Storage syscall + tool definitions | `StorageSyscall`, `storage_syscalls` |
| `aios/syscall/tool.py` | Tool syscall wrapper | `ToolSyscall` |
| `aios/scheduler/base.py` | Scheduler thread management | `BaseScheduler` |
| `aios/scheduler/fifo_scheduler.py` | FIFO scheduling + batching | `FIFOScheduler` |
| `aios/scheduler/rr_scheduler.py` | Round-robin with preemption | `RRScheduler` |
| `aios/llm_core/adapter.py` | LLM backend router | `LLMAdapter`, `LLMConfig` |
| `aios/llm_core/routing.py` | Load balancing strategy | `RouterStrategy`, `SequentialRouting`, `SmartRouting` |
| `aios/llm_core/local.py` | HuggingFace local inference | `HfLocalBackend` |
| `aios/memory/manager.py` | Memory high-level API | `MemoryManager` |
| `aios/memory/note.py` | Memory atom data structure | `MemoryNote` |
| `aios/memory/providers/factory.py` | Memory provider creation | `ProviderFactory` |
| `aios/memory/providers/in_house.py` | Built-in vector memory | `InHouseProvider` |
| `aios/memory/context_injector.py` | Memory → prompt injection | `ContextInjector` |
| `aios/memory/conversation_extractor.py` | Async conversation saving | `ConversationExtractor` |
| `aios/storage/storage.py` | File system manager | `StorageManager` |
| `aios/storage/filesystem/lsfs.py` | LLM-semantic file system | `LSFS` |
| `aios/storage/filesystem/vector_db.py` | Semantic file search | `VectorDB` |
| `aios/tool/manager.py` | Tool lifecycle manager | `ToolManager` |
| `aios/tool/mcp_server.py` | MCP tool server | MCP endpoints |
| `aios/hooks/stores/_global.py` | Global singleton queues | 4 global queue instances |
| `aios/hooks/stores/queue.py` | Thread-safe queue primitives | `addMessage`, `getMessage` |
| `aios/hooks/stores/processes.py` | Agent process registry | process table |
| `aios/hooks/modules/llm.py` | LLM hook factory | `useCore`, `useLLMRequestQueue` |
| `aios/hooks/modules/scheduler.py` | Scheduler hook factory | `fifo_scheduler_nonblock` |
| `aios/context/simple_context.py` | Context save/restore | `SimpleContextManager` |
| `aios/terminal/intent_router.py` | CLI natural language router | `IntentRouter` |
| `aios/config/config_manager.py` | Global config singleton | `Config`, `config` |

---

## 10. Inter-Component Communication

```
┌──────────────────────────────────────────────────────────────┐
│ How components talk to each other                            │
│                                                              │
│ Agent → Kernel:    execute_request(agent_name, Query)        │
│                    (synchronous call, blocks until done)     │
│                                                              │
│ Kernel → Queues:   global_llm_req_queue_add_message(syscall) │
│                    global_memory_req_queue_add_message(...)  │
│                    global_storage_req_queue_add_message(...) │
│                    global_tool_req_queue_add_message(...)    │
│                                                              │
│ Queues → Scheduler: get_llm_syscall() (blocking pop)         │
│                                                              │
│ Scheduler → LLM:   llm.execute_llm_syscall(syscall)          │
│                    llm.execute_llm_syscalls(batch)           │
│                                                              │
│ Scheduler → Agent: syscall.event.set()  (unblocks thread)    │
│                                                              │
│ LLM → ContextMgr:  save_context(pid, partial_state)         │
│                    load_context(pid) → resumes               │
│                                                              │
│ Memory → LLM:      ContextInjector.inject() adds memories    │
│                    to LLM messages before inference           │
│                                                              │
│ LLM → Memory:      ConversationExtractor.extract_async()     │
│                    saves chat turns as memories              │
│                                                              │
│ External → Kernel: REST API (FastAPI, runtime/launch.py)     │
│                    POST /initialize/llm                      │
│                    POST /initialize/memory                   │
│                    POST /initialize/storage                  │
│                    POST /initialize/tool                     │
│                    POST /initialize/scheduler                │
│                    POST /submit_agent                        │
│                    GET  /get_agent_result                    │
│                    POST /query (direct syscall)              │
└──────────────────────────────────────────────────────────────┘
```

---

## 11. Concurrency Model

The AIOS concurrency model is:

```
Main thread:  FastAPI event loop (uvicorn)
              Handles REST requests, initializes components

Agent threads: One per active agent submission
               Each agent thread blocks in syscall.join() 
               while waiting for the scheduler

Scheduler threads: 4 threads per scheduler instance
               Thread 1: process_llm_requests    (drains LLM queue)
               Thread 2: process_memory_requests (drains memory queue)
               Thread 3: process_storage_requests(drains storage queue)
               Thread 4: process_tool_requests   (drains tool queue)

LLM execution: Can be parallel (batch mode with ThreadPoolExecutor)
               or sequential depending on backend capabilities

Memory extraction: Background daemon thread per extractor instance
                   Runs asynchronously, doesn't block agent

MCP Server: Separate subprocess (not a thread)
            Communicates via HTTP/SSE
```

**Thread safety mechanisms:**
- `threading.Lock` on `tool_conflict_map` in `ToolManager`
- `threading.Lock` on syscall ID counter in `SyscallExecutor`
- `queue.Queue` for all inter-thread communication (thread-safe by design)
- `threading.Event` per syscall for precise unblocking

---

## 12. Extending the OS: Adding a New Subsystem

To add a new resource type (e.g., a **NetworkManager** for HTTP requests):

**Step 1: Define the Query/Response types** (in `cerebrum` or locally):
```python
# network/apis.py
class NetworkQuery:
    url: str
    method: str
    headers: dict
    body: Optional[str]

class NetworkResponse:
    status_code: int
    response_body: str
    finished: bool
```

**Step 2: Create the Syscall class** (`aios/syscall/network.py`):
```python
from aios.syscall import Syscall

class NetworkSyscall(Syscall):
    def __init__(self, agent_name: str, query: NetworkQuery):
        super().__init__(agent_name, query)
        self.target = "network"
```

**Step 3: Create the Manager** (`aios/network/manager.py`):
```python
class NetworkManager:
    def address_request(self, syscall: NetworkSyscall) -> NetworkResponse:
        import requests
        resp = requests.request(
            method=syscall.query.method,
            url=syscall.query.url,
            headers=syscall.query.headers,
            data=syscall.query.body
        )
        return NetworkResponse(status_code=resp.status_code, response_body=resp.text, finished=True)
```

**Step 4: Add hook module** (`aios/hooks/modules/network.py`):
```python
def useNetworkRequestQueue():
    queue = NetworkRequestQueue()
    QueueStore.REQUEST_QUEUE["network"] = queue
    def getMessage(): return QueueStore.getMessage(queue)
    def addMessage(msg): return QueueStore.addMessage(queue, msg)
    def isEmpty(): return QueueStore.isEmpty(queue)
    return queue, getMessage, addMessage, isEmpty
```

**Step 5: Register in global store** (`aios/hooks/stores/_global.py`):
```python
from aios.hooks.modules.network import useNetworkRequestQueue
(global_network_req_queue, ...) = useNetworkRequestQueue()
```

**Step 6: Add processing thread to scheduler** (`aios/scheduler/fifo_scheduler.py`):
```python
def process_network_requests(self):
    while self.active:
        try:
            syscall = self.get_network_syscall()
            self._execute_syscall(syscall, self.network_manager.address_request, "Network")
        except Empty:
            pass

def start(self):
    self.active = True
    self.start_processing_threads([
        self.process_llm_requests,
        self.process_memory_requests,
        self.process_storage_requests,
        self.process_tool_requests,
        self.process_network_requests,  # ← add here
    ])
```

**Step 7: Add dispatch to SyscallExecutor** (`aios/syscall/syscall.py`):
```python
elif isinstance(query, NetworkQuery):
    return self.execute_network_syscall(agent_name, query)
```

**Step 8: Expose initialization endpoint** (`runtime/launch.py`):
```python
@app.post("/initialize/network")
async def initialize_network(config: NetworkConfig):
    network_manager = useNetworkManager(config.dict())
    active_components["network"] = network_manager
```

---

## 13. Bootstrap Sequence (How the Kernel Starts)

When `runtime/launch.py` starts:

```
1. load_dotenv()                          ← Load API keys from .env
2. FastAPI app created
3. CORS middleware added
4. execute_request, SysCallWrapper, syscall_executor = useSysCall()
   └── Creates SyscallExecutor (empty, no subsystems yet)
5. FastAPI server starts (uvicorn)
6. Client sends: POST /initialize/llm     ← Initialize LLM adapter
   └── useCore({llm_configs: [...]})
   └── active_components["llm"] = llm_adapter
7. Client sends: POST /initialize/memory  ← Initialize memory manager
   └── useMemoryManager({...})
   └── active_components["memory"] = memory_manager
8. Client sends: POST /initialize/storage ← Initialize storage manager
   └── useStorageManager({root_dir: "..."})
   └── active_components["storage"] = storage_manager
9. Client sends: POST /initialize/tool    ← Initialize tool manager
   └── useToolManager({...})
   └── active_components["tool"] = tool_manager
   └── MCP server subprocess started
10. Client sends: POST /initialize/scheduler ← Start the scheduler
    └── fifo_scheduler_nonblock(llm, memory, storage, tool, queues)
    └── scheduler.start() → launches 4 processing threads
11. [Optional] Setup ContextInjector and ConversationExtractor
    └── syscall_executor.context_injector = ContextInjector(memory_manager)
    └── syscall_executor.conversation_extractor = ConversationExtractor(memory_manager)
12. System is now operational. Agents can be submitted.
13. Client sends: POST /submit_agent {agent_id, agent_config}
    └── AgentFactory.submit(agent_id, agent_config)
    └── Agent thread starts, calls execute_request() for all operations
```

---

## 14. API Surface (Runtime REST Endpoints)

| Method | Path | Body | Action |
|--------|------|------|--------|
| `POST` | `/initialize/llm` | `LLMConfig` | Start LLM adapter |
| `POST` | `/initialize/memory` | `MemoryConfig` | Start memory manager |
| `POST` | `/initialize/storage` | `StorageConfig` | Start storage manager |
| `POST` | `/initialize/tool` | `ToolManagerConfig` | Start tool manager + MCP server |
| `POST` | `/initialize/scheduler` | `SchedulerConfig` | Start scheduler threads |
| `POST` | `/submit_agent` | `AgentSubmit` | Launch an agent process |
| `GET` | `/get_agent_result` | `?agent_id=...` | Retrieve agent output |
| `POST` | `/query` | `QueryRequest` | Direct syscall (bypass agent) |
| `GET` | `/status` | — | Health check |
| `POST` | `/add_llm` | `LLMConfig` | Hot-add LLM backend |
| `DELETE` | `/remove_llm` | `{llm_name}` | Remove LLM backend |

---

## 15. Implementation Checklist

Use this checklist when building an AIOS from scratch:

### Phase 1: Core Infrastructure
- [ ] Define `Query` / `Response` typed protocol for each resource type (LLM, Memory, Storage, Tool)
- [ ] Implement `Syscall` base class (threading.Thread + threading.Event + timing fields)
- [ ] Implement `SyscallExecutor` with queue routing and timing measurement
- [ ] Create thread-safe queue store with `addMessage` / `getMessage` primitives
- [ ] Implement global singleton queue instances (one per resource type)

### Phase 2: LLM Kernel
- [ ] Implement `LLMAdapter` supporting at least one backend (OpenAI via litellm is simplest)
- [ ] Support `text`, `tool_calls`, and `json` response types
- [ ] Add routing strategy for multi-backend load balancing
- [ ] Implement `LLMSyscall` class

### Phase 3: Scheduler
- [ ] Implement `BaseScheduler` with thread management
- [ ] Implement `FIFOScheduler` with batched LLM processing
- [ ] Implement `RRScheduler` with time-sliced preemption (requires ContextManager)
- [ ] Implement `SimpleContextManager` for state save/restore

### Phase 4: Memory Subsystem
- [ ] Define `MemoryNote` dataclass
- [ ] Implement `MemoryProvider` abstract interface
- [ ] Implement `InHouseProvider` (start with simple dict, add vectors later)
- [ ] Implement `MemoryManager` with pluggable providers
- [ ] Implement `ContextInjector` (memory → prompt)
- [ ] Implement `ConversationExtractor` (async conversation storage)
- [ ] Add agentic memory pipeline: analyze → retrieve → evolve → add

### Phase 5: Storage Subsystem
- [ ] Implement `StorageManager` wrapping a basic file system
- [ ] Implement `LSFS` with read/write/create/delete/list/search
- [ ] Add `VectorDB` for semantic file search
- [ ] Add `storage_syscalls` tool definitions for LLM-parsed file operations

### Phase 6: Tool Subsystem
- [ ] Implement `ToolManager` with tool discovery and execution
- [ ] Implement MCP server for standardized tool communication
- [ ] Add sandboxed execution environment (Docker is simplest)

### Phase 7: Hook Layer
- [ ] Create `use*` hook factories for each subsystem
- [ ] Wire hook outputs into `BaseScheduler` constructor

### Phase 8: Runtime
- [ ] Implement FastAPI kernel server (`runtime/launch.py`)
- [ ] Add initialization endpoints for each subsystem
- [ ] Add agent submission and result retrieval endpoints
- [ ] Add direct query endpoint for testing

### Phase 9: Observability & Configuration
- [ ] Implement `Config` singleton reading from YAML + env
- [ ] Add per-component loggers with configurable log levels
- [ ] Add timing metrics collection and reporting
- [ ] Implement terminal/intent router for CLI access

### Phase 10: Advanced
- [ ] Add external memory providers (Mem0, Zep)
- [ ] Add smart LLM routing (load-aware)
- [ ] Add cloud VM providers for tool sandbox
- [ ] Implement Rust performance-critical paths (queue, scheduler hot loops)
- [ ] Add comprehensive test suite per module

---

## Summary: The Mental Model

An AIOS is built around one core insight: **treat the LLM like a CPU and build the rest of the OS around it.**

1. **Agents** are processes. They have a name and submit typed queries.
2. **System calls** are the only way agents touch resources. They are typed, timed, and queued.
3. **Queues** decouple agents from resources. An agent submits and blocks; the scheduler decides when to execute.
4. **The scheduler** is the heart of fairness — FIFO for simplicity, Round-Robin for preemption.
5. **The LLM** is used not just for agent reasoning, but also internally (memory analysis, intent parsing, file operation parsing).
6. **Memory** is active, not passive — it analyzes, evolves, and injects itself into future LLM calls.
7. **Hooks** wire everything together reactively, without tight coupling.
8. **Config** is a singleton, providers are pluggable, and every subsystem is independently replaceable.

This architecture scales from a single-model local setup to a distributed multi-model cluster by swapping providers and routing strategies without changing agent code.

---

*Reference implementation: [AIOS — github.com/agiresearch/AIOS](https://github.com/agiresearch/AIOS)*
*Language: Python 3.10+ | Key deps: FastAPI, litellm, openai, pydantic, threading, uvicorn*
