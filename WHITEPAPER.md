# Galaxy Orbs: A Whitepaper on Decentralized, Privacy-Preserving Edge AI Inference

## Abstract

Galaxy Orbs constitute a decentralized inference infrastructure engineered for the secure, efficient execution of quantized large language models (LLMs) such as Qwen2.5-0.5B at the network edge. Leveraging ternary quantization on field-programmable gate arrays (FPGAs), Orbs facilitate anonymous, transaction-based inference within a global distributed network, mitigating the privacy vulnerabilities and energy inefficiencies inherent in centralized providers like OpenAI and generic GPU architectures. This whitepaper delineates the hardware-optimized design, cryptographic protocols for end-to-end (E2E) encrypted prompt routing, integration with the gx402 library's Agent framework, and economic mechanisms via the x402 protocol. Orbs operate solely as agnostic compute nodes, processing encrypted XML-formatted inputs without contextual awareness of agent-specific logic. Complementary tools—such as MCP servers for modular field resolution, the Agent Builder for schema synthesis, the x402 Launchpad for agent tokenization, and browser agents for remote web orchestration—extend the ecosystem's composability. Empirical validations from edge FPGA deployments affirm the paradigm's efficacy, achieving substantial model compression and throughput gains for ternary-quantized LLMs. While applicable to proximal applications like Bluetooth-linked micro-robotics and IoT retrofitting, the primary emphasis is on scalable, privacy-centric global inference marketplaces.

## 1. Introduction

### 1.1 Motivation
Centralized LLM inference platforms, exemplified by OpenAI's API, impose inherent risks: all prompts traverse unencrypted channels to proprietary servers, exposing sensitive data to potential surveillance, model fine-tuning without consent, or adversarial exploitation. Concurrently, generic GPU clusters—optimized for broad-spectrum compute—exhibit suboptimal energy profiles for transformer-based architectures, with matrix multiplication-accumulate (MAC) operations dominating latency (>90% in baselines) and power draw, rendering them ill-suited for sustainable, model-specific deployments.

Galaxy Orbs redress these deficiencies through a distributed, E2E-encrypted inference fabric. Orbs function as stateless, anonymous processors: they ingest encrypted XML-serialized prompts via a router gateway, decrypt locally using per-device asymmetric keys, execute the inference workflow on ternary-quantized models, and return encrypted responses—effectuating a pure transaction sans content interpretation. This architecture ensures computational sovereignty, with payments in USDC on Solana atomically settling via the x402 protocol. By programming FPGAs at the gate level with parallel Verilog syntheses tailored to Qwen2.5-0.5B's ternary weights (-1, 0, +1), Orbs attain superior efficiency: operations map natively to lookup tables (LUTs) and multiplexers, obviating floating-point unit (FPU) overheads and yielding compression ratios exceeding 55% (e.g., INT4 analogs reducing Qwen2.5-0.5B from 988 MB to 443.81 MB, with negligible accuracy degradation to 61.97% on WNLI).

As a secondary application, Orbs support proximal Bluetooth Low Energy (BLE) linkages for resource-starved endpoints like sub-centimeter micro-robots or legacy IoT sensors, enabling offloaded inference without network mediation or payments—though the core paradigm targets global, monetized distribution.

### 1.2 Scope and Contributions
- **Hardware Specialization**: Gate-level FPGA acceleration for ternary-quantized Qwen, surpassing generic GPUs in energy and throughput.
- **Cryptographic Routing**: Router-mediated E2E encryption protocols preserving prompt opacity.
- **Workflow Abstraction**: gx402 Agents interfacing agnostic inference endpoints, augmented by MCP for dynamic resolution and browser agents for web tasks.
- **Economic Primitives**: x402 for USDC micropayments to Orbs; Launchpad for speculative agent tokenization.
- **Composability Tools**: Agent Builder for LLM-driven schema generation.

## 2. System Architecture

### 2.1 Hardware Design
Orbs manifest as compact, FPGA-centric nodes provisioned for distributed inference on platforms like the Xilinx Kria KV260, prioritizing low-latency, low-power execution in global networks.

- **Core Accelerator**: A Verilog-synthesized ternary quantization engine embeds Qwen2.5-0.5B weights statically during bitstream configuration, enabling gate-level parallelism. Ternary arithmetic (-1, 0, +1) eschews FPUs, leveraging LUTs for MAC ops and multiplexers for activation routing—directly ameliorating the 91% latency bottleneck in unoptimized baselines. This yields throughput escalations (up to 1.82×, from 2.8 to 5.1 tokens/s in analogous setups) and model footprints amenable to edge constraints (e.g., 55.1% compression via INT4 precursors, extensible to ternary for further ternary-state sparsity). Resource utilization—82% LUTs, 47% flip-flops—facilitates hybrid CPU-FPGA synergies, with 4-channel streaming confining bandwidth to 19.2 GB/s envelopes.
- **Form and Provisioning**: Rugged enclosures (<5 cm³) with USB-C provisioning; no persistent storage, ensuring ephemerality.
- **Network Interface**: Ethernet/Wi-Fi for router federation; optional BLE for local offloads (non-monetized).

### 2.2 Software Stack
The inference pipeline adheres to a minimalist footprint, eschewing high-level runtimes for hardware-direct invocation.

- **Execution Kernel**: Pure Verilog for accelerator orchestration; input decryption via on-device elliptic curve keys (e.g., Ed25519), followed by XML-parsed prompt dispatch to the Qwen bitstream.
- **Cryptographic Primitives**: AES-256-GCM for symmetric envelopes; asymmetric key exchange (ECDH) for session establishment.
- **Agnostic Processing**: Orbs interpret solely the XML workflow (e.g., tokenization, embedding, transformer passes, detokenization), oblivious to upstream agent semantics.

### 2.3 Inference Protocol: Router Gateway and E2E Encryption
Orbs integrate into the Galaxy network via a federated router gateway, which disambiguates load without compromising confidentiality.

- **Session Initiation**: gx402 Agents furnish the router URL and client public key. The router queries availability, returning a target Orb's ephemeral public key (rotated per session).
- **Prompt Encryption**: Clients hybrid-encrypt the XML-serialized prompt: ECDH-derived symmetric key encrypts payload; recipient Orb's public key wraps the symmetric key; client's public key enables response routing.
- **Transmission**: The router proxies ciphertext—never decrypting or logging—verifying signatures and x402 proofs en route. No plaintext traverses the gateway, enforcing E2E opacity.
- **Orb Processing**: Upon receipt, the Orb decrypts with its private key, executes inference (Qwen2.5-0.5B ternary), encrypts response XML symmetrically, and re-wraps for the client public key. Atomic USDC transfer (via Solana SPL) settles concurrently, with failure rollbacks.
- **Anonymity Guarantees**: Ephemeral keys and zero-knowledge attestations (e.g., Groth16 proofs of execution) preclude linkage; Orbs discard state post-response.

This protocol sustains global scalability: routers employ DHT-like routing for O(log N) dispatch, with Orbs registering ephemeral endpoints.

### 2.4 Orbs as the Neural Substrate of Distributed Inference
Orbs embody the decentralized compute substrate, aggregating into a permissionless marketplace where inference emerges from federated, incentivized nodes. Diverging from monolithic providers, this substrate obviates single-point data aggregation, channeling prompts through encrypted conduits to specialized hardware.

- **Federation Dynamics**: Nodes self-register via router directories, advertising capacity (e.g., Qwen ternary throughput). Load balancers—implemented as smart contracts—dispatch based on latency SLAs and stake-weighted reputation.
- **Efficiency Imperative**: Gate-level synthesis for model-specific parallelism (e.g., unrolled attention heads in Verilog) circumvents GPU tensor core inefficiencies, tailoring compute to Qwen's sparse ternary activations. Benchmarks affirm viability: hybrid FPGA designs on Kria KV260 parallelize MACs across LUT fabrics, compressing bandwidth while elevating tokens/s.
- **Emergent Scalability**: Network throughput scales superlinearly with node density, as parallel Orbs shard batched inferences; resilience derives from Byzantine fault tolerance in routing.

For ancillary proximal use cases—e.g., BLE-paired micro-robots offloading path-planning to co-located Orbs—the substrate extends sans payments, but global monetization remains paramount.

## 3. Operational Workflow: gx402 Agent Integration

gx402 Agents abstract inference orchestration, compatible with arbitrary endpoints (e.g., OpenAI APIs) or Galaxy routers. Orbs serve exclusively as backend executors.

### 3.1 Prompt Ingestion and Transformation
- **Invocation**: `Agent.run(prompt)` ingests a schema-conformant object (e.g., `{user_query: string, context: object}`).
- **Validation and Serialization**: Type-checked against inferred schema; unresolved fields flagged for MCP. Serialized to XML for parsimonious tokenization:
  ```xml
  <prompt>
    <user_query>Optimize supply chain</user_query>
    <mcp_fields><inventory>resolve_via_mcp</inventory></mcp_fields>
  </prompt>
  ```

### 3.2 MCP Servers: Modular Compute Protocol
MCP decouples dynamic resolution (e.g., API oracles) from core inference.

- **Dispatch**: Schema annotations (`@mcp:external`) trigger partial XML routing to providers.
- **Augmentation**: Providers compute (e.g., inventory query) and return XML fragments; merged non-destructively.
- **x402 Gating**: Initial probe elicits 402 responses; USDC tx resubmits atomically.

### 3.3 Inference Routing and x402 Gateway
- **Endpoint Agnosticism**: Agents specify router URLs for Orbs or direct APIs; encryption auto-negotiates for Galaxy.
- **Gateway Mechanics**: Probes metadata for fees; 402 elicits tx submission (USDC SPL transfer). Proofs (tx sig) authorize retry. Fallbacks cascade to free tiers.
- **Orb Integration**: As per Section 2.3, prompts route encrypted; Orbs execute anonymously, settling in USDC.

### 3.4 Response Parsing and Streaming
- **Output**: XML response (e.g., `<response><plan><step>Reallocate stock</step></plan></response>`).
- **Incremental Deserialization**: Tag-aware SAX parser streams fields to callbacks (`on_update('plan', partial)`), culminating in typed objects.

### 3.5 Browser Agents: Remote Web Orchestration
Browser agents augment gx402 with declarative web automation, decoupled from inference backends like Orbs.

- **Execution Layer**: A Chrome extension interprets Galaxy Query Language (GQL) over SSE streams from agent endpoints. GQL declaratively encodes actions (e.g., `navigate {url: "example.com"}; query {selector: ".data", extract: {text: true, screenshot: true}}`).
- **Integration**: Schema tools (`@browser:scrape`) embed GQL in XML; routed via MCP or direct SSE, with x402 for remote sessions.
- **Return Semantics**: Encrypted JSON/XML payloads (DOM excerpts, visuals) merge into responses; sessions ephemeral, authenticated via x402 tokens.
- **Independence**: Provisioned client-side or via shared pools; payments accrue to instance owners in USDC, orthogonal to Orb inference.

### 3.6 Flow Diagram
```
+----------------+    +----------------------+
| User Prompt    |    | Agent.run()          |
| (Schema Obj)   |--> | (Validate/Serialize) |
+----------------+    +----------+-----------+
                                |
                                v
                       +-------------------+
                       | XML w/ MCP Flags  |
                       +----------+--------+
                                |
                       +--------v--------+  No
                       | MCP Needed?     |--> +-------------------+
                       +----------+------+    | Final XML Prompt  |
                                | Yes          +-------------------+
                                v                     |
                       +-------------------+          v
                       | MCP Provider       |  +-------------------+
                       | (Augment/Return)   |  | Encrypt (Orb PK)  |
                       +----------+------+   |  +-------------------+
                                |             |         |
                       +--------v--------+    |         v
                       | x402 Probe      |    |  +-------------------+
                       | (402? Tx Submit)|    |  | Router Proxy      |
                       +----------------+    |  | (Ciphertext Only) |
                                |             |  +-------------------+
                                |             |         |
                                |             |         v
                                |             |  +-------------------+
                                |             |  | Orb Decrypt/Exec  |
                                |             |  | (Ternary Qwen)    |
                                |             |  +-------------------+
                                |             |         |
                                |             |         v
                                |             |  +-------------------+
                                |             |  | Encrypt Response  |
                                |             |  | (Client PK)       |
                                |             |  +-------------------+
                                |             |         |
                                |             |         v
                                |             |  [Return via Router]
                                |             |
                       +--------v--------+    |
                       | Resubmit MCP    |<---+
                       +----------------+

[Parallel Browser Branch]
                       +-------------------+
                       | GQL in Schema?    |
                       +----------+--------+
                                |
                       +--------v--------+  No
                       | SSE to Extension |--> [Merge to Parse]
                       | (x402 Gated)     |
                       +----------+------+
                                |
                       +--------v--------+
                       | Probe/ Tx Submit |
                       +-----------------+
                                |
                                v
                       +-------------------+
                       | Execute GQL       |
                       | (DOM Actions)     |
                       +-------------------+
                                |
                                v
                       +-------------------+
                       | Encrypted Results |
                       | (JSON/XML)        |
                       +-------------------+
```

## 4. Economic Model: x402 Protocol and Incentives

x402 operationalizes micropayments for distributed compute, standardizing USDC transfers on Solana for universality.

- **Orb Remuneration**: Inferences settle in USDC (e.g., 0.001-0.01 per query, dynamic via oracle); Orbs query tx proofs pre-execution, with slashing for non-delivery.
- **Network Alignment**: Fees fractionally buyback $GXY (ecosystem token), funding gx402 maintenance via trading royalties.
- **Privacy Preservation**: Zero-knowledge succinct arguments verify payments without revealing prompt metadata.

### 4.1 x402 Launchpad: Agent Tokenization
The Launchpad empowers gx402 users to tokenize Agents for public dissemination, fostering speculative markets orthogonal to inference payments.

- **Synthesis**: Via Agent Builder, users describe Agents (e.g., "E-commerce optimizer with web scraping"); LLM infers schema, tools (MCP/browser), and deployment artifacts.
- **Token Genesis**: Launchpad mints SPL tokens (e.g., fixed supply, governance-enabled); metadata embeds Agent spec for on-chain verification.
- **Monetization Flywheel**: Public invocations route via x402, accepting USDC—which the facilitator swaps to native tokens (AMM-mediated), partially burning supply (e.g., 20%) for deflationary pressure. Treasury accrues for iterations.
- **Speculative Utility**: Tokens signal Agent efficacy (e.g., adoption metrics as oracles); Orbs remain USDC-agnostic, decoupling backend from frontend economics.
- **Use Case**: A tokenized "SupplyChainAgent" ingests USDC for Orb-routed inferences; creators stake tokens for priority routing.

## 5. Agent Builder: Automated Schema Synthesis
The Builder leverages lightweight LLMs to transpile natural language into executable schemas.

- **Input Parsing**: Descriptive prompts (e.g., "Resolve inventory via MCP, infer paths with Qwen, scrape vendors via browser").
- **Inference**: Generates typed structures (TypeScript/Python) with annotations (`@mcp:oracle`, `@x402:gate`, `@browser:gql`), including validation stubs.
- **Export**: Instantiable Agents, with simulated traces for refinement.

## 6. Advantages and Comparative Analysis

| Attribute          | Centralized APIs (OpenAI) | Generic Edge GPUs       | Galaxy Orbs (FPGA-Ternary) |
|--------------------|---------------------------|-------------------------|----------------------------|
| **Privacy**        | Prompt Logging/Exfil     | Local but Verbose       | E2E Encrypted/Agnostic    |
| **Efficiency**     | Network + FPU Overhead   | Broad Compute Waste     | Gate-Level Model-Specific |
| **Scalability**    | Vendor-Locked            | Device-Bounded          | Federated Marketplace     |
| **Cost Model**     | Opaque Subscriptions     | CapEx Intensive         | USDC Micropayments        |
| **Customization**  | API-Constrained          | Firmware-Heavy          | Schema/Quantization Agnostic |

Ternary FPGA deployments validate superiority: 55% compression, 1.82× throughput, LUT-efficient parallelism.

## 7. Security and Privacy Considerations
- **E2E Ciphertexts**: Router blindness via hybrid encryption; Orbs employ hardware security modules (HSMs) for key isolation.
- **Anonymity Primitives**: Ephemeral IDs, ZK-SNARKs for execution proofs sans input revelation.
- **Adversarial Resilience**: Signed challenges thwart replays; bandwidth caps mitigate DoS.

## 8. Implementation and Roadmap
- **MVP**: Qwen2.5-0.5B ternary on Kria KV260; gx402 v1.0 with router SDK.
- **Q1 2026**: Multi-model bitstreams; Launchpad governance.
- **Horizon**: Cross-chain USDC bridges; Orb hardware commoditization.

## 9. Conclusion
Galaxy Orbs pioneer a privacy-sovereign, efficient inference substrate, supplanting centralized opacity with distributed, specialized compute. Through x402 economics, E2E protocols, and gx402 abstractions, this framework equips Agents for global deployment, from tokenized marketplaces to edge adjuncts.

## References
- gx402 Repository: https://github.com/galaxydo/gx402
- $GXY Token: Pump.fun CA `PKikg1HNZinFvMgqk76aBDY4fF1fgGYQ3tv9kKypump`
- Li, Y., et al. (2025). "On-Device Qwen2.5: Efficient LLM Inference with Model Compression and Hardware Acceleration." arXiv:2504.17376v1.

---

*Published: November 07, 2025*  
*Authors: GalaxyDO Team*  
*Contact: [@galaxydoxyz on X](https://x.com/galaxydoxyz)*
