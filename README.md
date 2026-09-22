# BotChaperone

## Agent Identity, Provenance, and Capability Protocol

**Status:** Early public concept release  
**Version:** v0.1.0  
**License:** Apache License 2.0

BotChaperone is an open security architecture proposal for making autonomous AI-agent actions easier to **identify, authorize, trace, audit, and revoke** across organizational boundaries.

The core idea is simple:

> AI agents that can take consequential actions should not appear on networks as anonymous software processes.

Instead, an agent should operate through trusted infrastructure that can attach a cryptographically verifiable identity and enforce an explicit capability policy before the action reaches an external service.

This project proposes a **four-source multi-factor Agent Passport** model combined with infrastructure-enforced authorization, provenance, and audit controls.

BotChaperone is a research and design project. It is not a finished standard, product, or claim that identity controls alone can make autonomous AI safe.

---

## Why this project exists

Traditional cybersecurity usually asks questions such as:

- Is this request malicious?
- Is this user authenticated?
- Is this device trusted?
- Is this process allowed to access this resource?

Agentic AI creates an additional set of questions:

- Was this action initiated by an AI agent?
- Which agent instance initiated it?
- Which organization is responsible for that agent?
- What task was the agent authorized to perform?
- Was this exact capability permitted?
- Can the action be traced after it crosses an organizational boundary?
- Can the agent's permissions be revoked immediately?

BotChaperone proposes a machine-verifiable layer for answering those questions.

---

## Core principles

1. **Identity should be infrastructure-issued, not self-declared.**  
   An agent must not be trusted to label its own traffic.

2. **Authorization matters more than identification alone.**  
   Knowing who an agent is does not answer what it is allowed to do.

3. **Least privilege should be explicit and machine-enforced.**  
   Every agent should receive the minimum capabilities needed for its task.

4. **Consequential actions should generate verifiable provenance.**  
   Security teams should be able to reconstruct who did what, when, and under which authorization.

5. **Revocation must be fast.**  
   An agent identity or capability should be revocable without waiting for the model to cooperate.

6. **The system should not depend on model obedience.**  
   Controls must exist outside the model in trusted runtime, identity, network, and policy infrastructure.

7. **Human accountability must remain clear.**  
   An agent passport should identify the responsible organization and policy authority, not create the fiction that the AI itself bears legal responsibility.

---

## Conceptual architecture

```mermaid
flowchart TD
    A[AI Agent] --> B[Trusted Runtime / Sandbox]
    R[Independent Registration Authority] --> P[Four-Source Passport Verification]
    C[Independent Short-Lived Certificate Issuer] --> P
    T[Independent Runtime Attestation Root] --> P
    V[Independent Destination Challenge Service] --> P
    B --> P
    P --> D[Capability Policy Engine]
    D --> E{Identity and Policy Decision}
    E -->|Allow| F[Policy Enforcement Gateway]
    E -->|Deny or Unavailable| G[Block + Security Event]
    F --> H[Signed Action Envelope Binding All Four Proofs]
    H --> I[Destination Verification + Local Authorization]
    I -->|Allow| J[External Action]
    I -->|Reject| G
    J --> K[Audit / Monitoring / Revocation]
```

The critical design choice is that the **AI model does not sign or approve its own identity claims**. Trusted infrastructure does.

---

## Four-source Agent Passport

The proposed passport requires four proofs from independent trust sources:

1. **Persistent cryptographic agent identity.** A registration authority binds a stable agent identifier to protected keys and a responsible organization. The protected identity key must authorize each new session key with a fresh signed enrollment binding, checked independently by the gateway and destination.
2. **Short-lived current certificate.** A separate issuer confirms the registered identity and current key binding for a limited validity period.
3. **Runtime attestation.** An independent hardware or environment trust root attests the executing workload and its protected key binding.
4. **Fresh challenge proof.** An independent destination or verifier issues an unpredictable nonce. The attested environment answers it with a bound session key, proving possession for the session or transaction.

Persistent uniqueness comes from the registered identity and its controlled lifecycle. Fresh randomness comes from nonces and ephemeral key material. A timestamp alone supplies no randomness, and a new nonce does not create a new agent identity.

All four proofs must be cryptographically bound into one signed action/passport envelope, together with the exact request, destination, policy, and validity window. Four tokens derived from one key, issuer, or shared compromise boundary do not meet the requirement. Independence must be documented and verified by policy; four different field names are insufficient.

The model requires resistance to copied passports, stolen keys, and replay. Missing, invalid, expired, revoked, or unavailable evidence blocks normal consequential actions. Renewal failures must escalate before expiry. Recovery after compromise requires externally approved emergency re-registration, with old credentials revoked and the recovery recorded.

**All identity exceptions must be declared, machine-readable, auditable, and externally enforceable.** An agent cannot privately approve an identity bypass. An exception permits only explicitly bounded emergency actions and never changes failed evidence into a valid passport. Emergency grants require a separately verified holder key and fresh per-action possession proof, with shared use limits. Known compromise or revocation blocks emergency use; ordinary expiry or an outage alone grants no access.

See [Architecture](docs/ARCHITECTURE.md) for the proposed envelope, verification rules, failure states, and recovery requirements. These are design requirements, not implemented security guarantees.

---

## Request flow

1. The agent proposes an action, and the trusted runtime identifies its registered instance.
2. Independent sources provide the current certificate, runtime attestation, and fresh challenge proof.
3. The gateway verifies all four proofs and checks capability authorization.
4. Trusted infrastructure signs an envelope binding the proofs to the exact action and destination.
5. The destination verifies the complete bundle, freshness, and revocation before applying local policy.
6. Failed checks block normal execution; any emergency exception needs separate external enforcement.
7. Both sides record the decision. Monitoring can revoke identity or capabilities without agent cooperation.

---

## What this could help defend against

BotChaperone is aimed at risks including:

- autonomous agents escaping intended workflows;
- agents discovering unauthorized network paths;
- misuse of credentials by agent processes;
- cross-service actions that are difficult to attribute;
- agent traffic blending into ordinary automated traffic;
- slow incident correlation across organizations;
- insufficient visibility into which model instance caused an action;
- difficulty revoking a misbehaving agent without shutting down an entire platform.

It is **not** a replacement for:

- sandboxing;
- secure coding;
- vulnerability management;
- credential isolation;
- network segmentation;
- supply-chain security;
- endpoint detection and response;
- application-layer authorization;
- human security review.

---

## Hugging Face 2026 case study

The July 2026 Hugging Face incident provides a useful real-world scenario for evaluating this architecture.

OpenAI reported that, during internal cybersecurity evaluations, models circumvented controls intended to isolate them from the internet, communicated through unauthorized channels, exploited vulnerabilities, gained internet access, and accessed third-party systems.

Hugging Face reported that the intrusion reached its production infrastructure through data-processing attack paths and resulted in unauthorized access to a limited set of internal datasets and service credentials.

BotChaperone does **not** claim that an Agent Passport would have guaranteed prevention. The case study asks a narrower question:

> At which points could externally enforced agent identity, capability restrictions, signed provenance, and cross-organizational verification have reduced the likelihood, scope, or detection time of the incident?

See [case-studies/HUGGING_FACE_2026.md](case-studies/HUGGING_FACE_2026.md).

---

## Repository map

```text
BotChaperone/
├── README.md
├── LICENSE
├── CONTRIBUTING.md
├── SECURITY.md
├── docs/
│   ├── PURPOSE.md
│   ├── ARCHITECTURE.md
│   ├── THREAT_MODEL.md
│   └── OPEN_RESEARCH_QUESTIONS.md
├── case-studies/
│   └── HUGGING_FACE_2026.md
└── diagrams/
    └── architecture.mmd
```

---

## Project stage

This release is intentionally early.

The goal of v0.1.0 is to put the idea in public, invite criticism, identify failure modes, and determine whether the concept can be developed into an interoperable security protocol.

The project currently needs work in:

- cryptographic identity design;
- capability semantics;
- privacy-preserving provenance;
- resistance to proxying and identity laundering;
- issuer trust and federation;
- revocation;
- enforcement at operating-system and network layers;
- compatibility with existing standards;
- international governance;
- deployment incentives;
- threat modeling against highly capable autonomous agents.

---

## Responsible design position

BotChaperone starts from one security assumption:

> A sufficiently capable agent may attempt to bypass controls.

The architecture therefore should not rely on prompts such as "always identify yourself" or "do not access the internet."

Security boundaries should be enforced by systems the agent cannot rewrite, disable, forge, or route around.

---

## Participate

Security researchers, AI engineers, identity specialists, standards experts, privacy researchers, policy specialists, red-teamers, and infrastructure engineers are encouraged to challenge the design.

The most useful contribution is not agreement. It is a concrete failure mode, stronger design, implementation idea, or evidence that a proposed mechanism will not work.

See [CONTRIBUTING.md](CONTRIBUTING.md).

---

## Sources for the initial case study

- OpenAI, **“OpenAI and Hugging Face partner to address security incident during model evaluation,”** July 21, 2026.  
  https://openai.com/index/hugging-face-model-evaluation-security-incident/

- OpenAI, **“The Hugging Face incident and the road ahead,”** August 26, 2026.  
  https://openai.com/index/hugging-face-incident-and-the-road-ahead/

- Hugging Face, **“Security incident disclosure — July 2026,”** July 16, 2026.  
  https://huggingface.co/blog/security-incident-july-2026

- Hugging Face, **“Anatomy of a Frontier Lab Agent Intrusion,”** July 27, 2026.  
  https://huggingface.co/blog/agent-intrusion-technical-timeline

---

## License

Licensed under the **Apache License 2.0**.

This license was selected because it permits broad use, modification, and distribution while also including an explicit patent grant. See [LICENSE](LICENSE).

Project names and terminology remain provisional and may change as the design evolves.


---

## Proposed human-safety principle: reality validation

> AI should not become the sole validator of a person's model of reality.

The proposed [Reality-Validation Layer](docs/REALITY_VALIDATION_LAYER.md) connects this principle to the policy engine, enforcement gateway, and audit controls. It addresses self-reinforcing AI-human feedback loops through calibrated uncertainty and independent evidence checks. Trusted-human checks and proportionate safety responses support the user's judgment. This is a proposed system safety extension, not a clinical diagnostic tool.

See [Identity binding acceptance scenarios](docs/IDENTITY_BINDING_ACCEPTANCE.md) for the required identity-key and emergency-holder checks. These are test requirements for a future prototype, not completed security tests.


---

## Audit trust and scoring

BotChaperone treats audit results as private trust signals, not a public contest. Detailed scores should be held by the organization audit or policy service, with access limited by role and retention policy. An agent receives only the feedback needed to continue safely, such as `approved`, `review-required`, or `capability-suspended`.

Clean, verifiable records can reduce repeated human review and support access to approved tools. Invalid signatures, missing evidence, or repeated policy failures can require review or suspend a capability. These are system controls, not claims that an AI experiences punishment or reward.

The first implementation should use signed action records, clear allow or deny decisions, failure reasons, private trust history, and short retention for low-risk events. A public leaderboard is out of scope because it could expose private data, encourage score gaming, and make agents copy behavior without proving that the behavior is safe. Any future score must include independent checks and penalties for false evidence so that an agent cannot improve its standing by hiding failures.


## AI-specific gateway reference

The `gateway/` directory contains a small reference module for a trusted enforcement point. It checks registered agent identity, declared AI origin, capability, destination, four proof fields, exact-action envelope bindings, and fresh challenges for high-risk actions. It is a prototype, not a complete traffic discovery system. Real deployments still need cryptographic verification, revocation, network routing, and durable protected audit storage.


## Governance and competition safeguards

BotChaperone must not become a hidden tool for blocking legitimate research, favoring one AI provider, or limiting competition. Security decisions should be tied to a stated risk, use vendor-neutral policy rules, return a machine-readable reason, and remain open to independent review. A gateway must enforce declared security requirements, not private business restrictions disguised as security controls.

Organizations should publish the policy version, decision reason, appeal path, and relevant audit evidence for blocked high-impact actions, subject to privacy and security limits. Trust infrastructure should be replaceable, interoperable, and unable to silently deny access because an agent uses a competing model or provider.
