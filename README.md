# SovereignContext (sovereign-context)

> **The local-first, zero-knowledge context firewall and reversible pseudonymization engine for autonomous web agents.**

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![Architecture: Local-First](https://img.shields.io/badge/Architecture-100%25%20On--Device%20(WebGPU%2FWasm)-brightgreen.svg)](#architecture)
[![Target: Mozilla Builders / Incubator](https://img.shields.io/badge/Mozilla-Builders%20%26%20Incubator%20Aligned-FF7139.svg)](#why-mozilla-foundation)
[![Status: Concept & Prototype Specification](https://img.shields.io/badge/Status-Alpha%20%2F%20RFC-orange.svg)](#roadmap)

---

## 📌 Executive Summary

As artificial intelligence shifts from passive text chatbots to **autonomous web agents** (browser operators, autonomous coding agents, desktop automations), a critical privacy crisis has emerged: **unbounded context exfiltration**. 

Modern web agents ingest raw DOM snapshots, clipboard buffers, local storage keys, session tokens, and personal form inputs, transmitting complete unredacted payloads to centralized cloud model providers (OpenAI, Anthropic, Google). 

**SovereignContext** is an open-source, client-side firewall that sits between web agents and cloud LLM APIs. Executing entirely on-device via WebGPU and lightweight Small Language Models (SLMs), it provides **bidirectional, reversible pseudonymization**: it intercepts raw agent context, synthetically masks sensitive identities and secrets with context-preserving proxies, sends only anonymized structures to frontier models, and seamlessly decodes the agent's instructions back into real values locally before any browser or DOM execution occurs.

---

## 🔍 The Competitive Landscape: What Exists vs. What We Solve

A common question is: *“Doesn’t Microsoft Presidio, Ollama, or an AI Gateway already do this?”*

**No.** Existing tools solve adjacent problems for servers or local chats, but completely fail to address the specific vulnerability of **autonomous browser agents**:

| Feature / Capability | Enterprise Redactors (e.g., MS Presidio) | Offline LLM Runners (e.g., Ollama / Jan) | Cloud AI Gateways (e.g., Portkey / LiteLLM) | Privacy Browser Exts (e.g., Privacy Badger) | **SovereignContext** |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Execution Location** | Backend / Server | Local Heavy Desktop | Centralized Cloud Server | Browser Extension | **100% Client-Side (WebGPU/Wasm)** |
| **Agent / DOM Context Aware** | ❌ No | ❌ No | ❌ No | ⚠️ Regex / Ad Heuristics | **✅ Yes (DOM & Tool Call Parsing)** |
| **Bidirectional Reversible Mapping** | ❌ No (Hard Redacts `[PII]`) | ❌ N/A | ⚠️ Rare (Proprietary / Cloud) | ❌ No | **✅ Yes (Local Keyed Synthetic Vault)** |
| **Preserves LLM Reasoning Context** | ❌ Breaks downstream reasoning | ❌ N/A | ❌ Cloud-dependent | ❌ N/A | **✅ Yes (Synthetically Coherent Tokens)** |
| **No Third-Party Cloud Dependency** | ❌ Requires hosting | ✅ Completely offline | ❌ Another cloud intermediary | ✅ Client-side | **✅ Zero-Server / Air-Gapped Local** |
| **Agent Capability Permissions** | ❌ No | ❌ No | ⚠️ Server Rate Limits | ❌ Tracker blocking only | **✅ Granular Agent Access Control** |

### Why Existing Solutions Fail for Autonomous Agents:
1. **Hard Redaction (`[REDACTED]`) Destroys Agent Utility**: Replacing `user@company.com` or `Account #49382` with `[EMAIL]` breaks an agent's ability to reason, click specific elements, or complete multi-step workflows. SovereignContext uses *synthetic contextual substitution* (e.g., mapping to `test_user_a@demo.org`), preserving syntactic validity and reasoning capability.
2. **Local-Only LLMs Lack Agent Reasoning Power**: Running an entire agent workflow on a 4-bit local model (e.g., 7B parameters) frequently fails at complex web tasks. SovereignContext lets users leverage state-of-the-art cloud models (Claude 3.5 Sonnet, GPT-4o) **without ever exposing real personal identity, financial figures, or passwords to cloud vendors.**
3. **Cloud Gateways Trade One Risk for Another**: Routing your agent traffic through another hosted "privacy proxy" simply shifts your vulnerability to that proxy operator. SovereignContext runs **zero cloud servers**.

---

## 🏛️ Architecture

```
                                      [ THE BROWSER / OS ]
                                                │
                          ┌─────────────────────┴─────────────────────┐
                          │                                           │
                   [Web Application]                          [Autonomous Agent]
             (DOM, Form Inputs, Session Data)              (Browser Use / Playwright)
                          │                                           │
                          └─────────────────────┬─────────────────────┘
                                                │
                                    [ Raw Prompt / DOM Payload ]
                                                ▼
 ╔══════════════════════════════════════════════════════════════════════════════════════════╗
 ║                                SOVEREIGNCONTEXT (LOCAL ENGINE)                           ║
 ║                                                                                          ║
 ║   1. Semantic Detection Engine (Wasm regex + local WebGPU SLM: SmolLM2 / Qwen2.5)        ║
 ║      ├── Detects PII, Credentials, API Keys, Private Context                             ║
 ║      └── Enforces Granular Agent Permission Policies                                    ║
 ║                                                                                          ║
 ║   2. Reversible Token Vault (Local In-Memory SQLite / AES-GCM Encrypted)                 ║
 ║      └── Maps: "Jane Doe, Card 4012..." ──▶ "Synthetic User Alpha, Card 0001..."         ║
 ║                                                                                          ║
 ║   3. Local Cryptographic Audit Ledger                                                    ║
 ║      └── Appends SHA-256 Merkle root of intercepted & masked payloads for user audit     ║
 ╚══════════════════════════════════════════════════════════════════════════════════════════╝
                                                │
                              [ Anonymized / Synthetic Payload ]
                                                ▼
                                   [ Cloud Frontier LLM ]
                               (OpenAI / Anthropic / Google)
                                                │
                               [ Synthetic Agent Response ]
                               (e.g., "Click #input-card-0001")
                                                ▼
 ╔══════════════════════════════════════════════════════════════════════════════════════════╗
 ║                     REVERSE DE-ANONYMIZATION (LOCAL CLIENT ONLY)                         ║
 ║   - Swaps "0001" back to real "4012" in-flight within secure local sandbox               ║
 ║   - Dispatches safe, real instruction to browser DOM                                     ║
 ╚══════════════════════════════════════════════════════════════════════════════════════════╝
```

---

## 🚀 Key Features

* **⚡ Zero-Server WebGPU SLM Guard**: Runs entirely within the browser sandbox or local OS daemon using WebGPU hardware acceleration. Zero telemetry, zero external calls.
* **🔄 Bidirectional Reversible Pseudonymization**: Generates structurally valid synthetic data so cloud agents retain 100% reasoning accuracy, while real identity values never leave your physical device.
* **🛡️ Granular Agent Permission Manifests**: Similar to Android/iOS runtime permissions, SovereignContext prompts users before an agent is permitted to read specific DOM fields, clipboard buffers, or cookies.
* **📜 Local Merkle-Tree Audit Trail**: Provides an immutable, human-readable inspection log showing every token sanitized, every query dispatched, and every outbound destination.
* **🔌 Drop-In Compatibility**: Supports OpenAI-compatible endpoints, Anthropic API formats, Playwright/Puppeteer browser agents, and standard WebExtension APIs.

---

## 🎯 Why This Aligns with the Mozilla Foundation

The Mozilla Foundation’s grantmaking and incubation mandates prioritize:
1. **Countering Monopolistic Surveillance**: SovereignContext prevents cloud AI giants from using individual browsing behaviors to build behavioral telemetry profiles.
2. **Open Source Digital Public Goods**: Fully permissive licensing (Apache 2.0), transparent codebases, and zero proprietary lock-in.
3. **Local & Human-Scale AI**: Embracing on-device SLMs to empower individual end users rather than relying on hyper-scale data centers.
4. **Internet Health & Digital Agency**: Providing everyday users with practical, understandable defenses against opaque automated systems.

---

## 📂 Repository Structure

```
sovereign-context/
├── core/                   # Core interception & reversible pseudonymization engine
│   ├── src/
│   │   ├── slm/            # WebGPU / Wasm Small Language Model pipeline
│   │   ├── vault/          # In-memory encrypted mapping table
│   │   ├── parser/         # DOM, JSON, and raw text stream tokenizer
│   │   └── audit/          # Merkle tree local logging engine
├── extension/              # WebExtension (Firefox & Chromium compatible)
│   ├── src/
│   │   ├── background/     # Request interceptor & proxy layer
│   │   ├── content/        # DOM agent interaction gatekeeper
│   │   └── ui/             # Permission modals & live privacy dashboard
├── adapters/               # Integration adapters (Playwright, Browser-Use, LiteLLM)
├── specs/                  # RFCs, Threat Model, and Architecture Specs
├── LICENSE                 # Apache 2.0
└── README.md
```

---

## 🛠️ Getting Started (Local Development)

### Prerequisites
* **Node.js**: v20.x or higher
* **npm** or **pnpm**
* A modern browser with **WebGPU** support (Firefox Nightly, Chrome, Edge)

### Installation
```bash
# Clone the repository
git clone https://github.com/your-username/sovereign-context.git
cd sovereign-context

# Install dependencies
npm install

# Build core and extension
npm run build

# Run unit and integration tests
npm test
```

---

## 🗺️ Roadmap

- [x] **Phase 0: Architecture & Threat Model RFC** (Current)
- [ ] **Phase 1: Heuristic & Regex Interceptor Engine** (Client-side regex + rule-based PII vault)
- [ ] **Phase 2: On-Device SLM Integration** (WebGPU-powered SmolLM2 / Qwen2.5 for semantic PII detection)
- [ ] **Phase 3: Firefox / Chromium WebExtension MVP** (Real-time agent context visualization and consent popups)
- [ ] **Phase 4: Multi-Agent Adapter Framework** (Direct drop-in for `browser-use`, `playwright`, and `LangChain`)

---

## 📄 License

Distributed under the **Apache 2.0 License**. See [`LICENSE`](LICENSE) for details.
