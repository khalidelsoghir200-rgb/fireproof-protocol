# FireProof Constitution

Status: governing document for future work. Accepted by the project owner on
2026-07-17 and refined by ADR-0021. This document changes future direction; it
does not rewrite historical evidence or completed verification records.

## Founding judgment

FireProof is not an AI platform, a dashboard, an observability product, or a
generic SaaS application. It is a **Receiver-Owned Acceptance Runtime** for
consequential changes initiated by AI in systems of record.

Its durable product object is a portable evidence bundle that lets an
independent party answer a narrow but consequential question:

> Why did the receiving system accept or reject this AI-initiated change, and
> what state transition did it durably commit?

An external authority or governance system may issue a signed decision. It
does not own acceptance merely by doing so. The receiver owns the record, the
locally admitted trust relation, the live-state check, the atomic write, and
the signature over its result.

We do not define the company by the limits of a current vendor, model, cloud,
or agent framework. We define it by an invariant that becomes more important as
autonomous systems gain authority: a consequential system-of-record mutation
must be locally accepted and independently provable after the surrounding
systems change or disappear.

The vision is expansive. The proof is deliberately narrow. A complete MVP is
the smallest end-to-end demonstration that makes this invariant hard to
dispute; it is not a small vision.

## Article 1 — Product identity

An Action Evidence Runtime supplies the evidence architecture. The first
product runtime is receiver-owned: it validates a portable Authority artifact
against its own trust configuration, checks that exact request against current
state, atomically accepts or rejects the mutation, and creates a receiver
Fulfillment only for a durable terminal outcome.

At minimum, an admitted evidence chain binds:

1. an intended operation or privacy-preserving commitment to it;
2. a signed Authority artifact and the receiver's admitted signer relation;
3. the exact target and expected state version;
4. the receiver's terminal acceptance or rejection outcome;
5. before/after commitments for a committed state transition, where relevant;
6. identities, time, version, and verification material; and
7. a receiver signature for any claimed succeeded Fulfillment.

A receipt is not an assertion of absolute truth. It proves the signed,
versioned statements and any explicitly scoped receiver observation or
attestation. A missing, contradictory, expired, tampered, stale, or replayed
link must remain visible as such.

## Article 2 — Non-negotiable principles

1. **Evidence is not a log.** Mutable logs, dashboards, and vendor event
   histories may support operations; they are not the FireProof product.
2. **The receiver is sovereign.** The owner of the system of record controls
   the acceptance predicate, trust root, current state, replay fence, write
   transaction, and Fulfillment key. An agent never obtains a direct write
   path by presenting its own claim.
3. **No proof, no change.** A consequential AI-originated write requires
   locally verified and receiver-admitted Authority evidence.
4. **No current-state match, no acceptance.** Authority for one record version
   cannot silently mutate another version.
5. **No receiver signature, no succeeded fulfillment.** An issuer,
   intermediary, or agent cannot claim that a system of record committed a
   change.
6. **Verification is independent.** A third party must be able to verify an
   admitted receipt without an account on FireProof or access to the original
   agent platform.
7. **The format is neutral.** Replacing an agent, model, cloud, integrator, or
   system of record must not invalidate prior evidence or require a proprietary
   verifier.
8. **The chain is explicit.** Request commitments, authority, receiver trust
   relation, current-state expectation, terminal outcome, signer identity, and
   version linkage are first-class inputs, not inferred from prose or a UI.
9. **Disclosure is minimal.** Evidence proves what is necessary while avoiding
   gratuitous customer data. Commitments, hashes, references, and selective
   disclosure are preferred to raw sensitive payloads.
10. **Ambiguity fails honestly.** Failed, expired, invalid, and indeterminate
    outcomes are never presented as success. A claimed succeeded fulfillment
    is never released before the durable receiver transaction commits.
11. **Longevity is part of the product.** Canonicalization, stable schemas,
    versioning, trust material, and offline verification matter because a
    receipt must outlive a deployment.
12. **The verifier cannot depend on our survival.** No FireProof service,
    database, or dashboard may become the sole truth source for a receipt.
13. **Novelty needs proof.** A new technique is admitted only when it
    strengthens an invariant, has a bounded test, and does not turn the
    evidence format into vendor lock-in or cryptographic theater.

## Article 3 — Strategic posture

FireProof is built as a strategic trust primitive, not as a self-service trial
product. We do not measure the first phase by signups, screens, free trials,
or generic pilot volume.

The strategic test is whether independently verifiable receiver acceptance can
be useful across organizations and systems. The project may later be
integrated, licensed, deployed, or acquired; none of those outcomes justifies
overstating an unproven capability today.

## Article 4 — Public verification and private operating value

Independent verification requires an open interoperability boundary. The
eventual public boundary is the evidence specification, canonicalization and
signing rules, safe fixtures, verifier, conformance suite, and reference
profile. It is intended for a clean Apache-2.0 release only after the existing
licensing and release controls are satisfied.

The private operating value is the receiver runtime and its semantic
integrations: local trust admission, state comparison, transactional writes,
replay control, Fulfillment signing, key and trust lifecycle management,
evidence lifecycle controls, and support. It may be self-hosted,
partner-operated, or enterprise-deployed. It is not a hosted SaaS requirement.
Customer credentials, customer data, private receipts, and operational keys
never enter the public verification core.

## Article 5 — Scope discipline

The first complete runtime is an owner-controlled ERP Supplier Master sandbox
with synthetic data. It permits one operation:
`supplier.payment-destination.replace`. The receiving ERP must reject a bare,
tampered, replayed, untrusted, expired, mismatched, or stale request; it may
write only after local verification and an exact current-state match. On
success it must commit the state change and receiver-signed Fulfillment in one
durable transaction, then permit offline verification of the exported bundle.

The first runtime does not need a bank connection, real supplier record, SAP,
Siemens, or Accenture integration. It must be sufficiently neutral that those
environments would not require a new evidence model.

The following are outside the first runtime unless a new accepted decision
admits them:

- a generic agent builder, workflow engine, orchestration platform, or tool
  executor;
- a central FireProof policy engine, identity authority, credential broker,
  MCP/API gateway, or direct agent write path;
- a dashboard as the primary value proposition;
- a tracing, SIEM, IAM, compliance, or legal-liability replacement;
- a free-trial SaaS motion, public pricing, or a 14-day pilot offer;
- a vendor-specific evidence format;
- blockchain or additional cryptography without a measurable evidence need;
- broad multi-tenant hosting, billing, customer-data collection, or public
  deployment.

## Article 6 — Historical record and donor discipline

Completed Gate 1-3 evidence and the closed HubSpot demonstration are historical
records. They remain inspectable, but they are not current operating
instructions and do not authorize reuse of a deleted account, credential, or
pilot process.

The repository `fireproof-ship-core` is a research donor, never an import
authority. No donor implementation, fixture, or protocol assertion enters this
repository without path-level provenance, license review, specification
comparison, clean VM reproducibility, and an explicit decision record.

## Article 7 — Operating discipline

All code, build, test, security, package, and release work runs on the project
VM. Local work is review and synchronization only. A claim is verified only
for its exact VM command, commit, and scope.

Every material change has one owner, one bounded objective, one source of
truth, a stated public/private boundary, and a definition of done. A
technology experiment records its hypothesis, counterfactual, failure signal,
and removal path before it becomes a dependency. R2 contract changes must be
accepted before R2 source changes; R1 source and fixtures must remain
compatible.

## Precedence

1. Project-owner decisions and safety constraints.
2. This constitution.
3. A newer accepted architecture or product decision record.
4. Stage-specific acceptance criteria.
5. Implementation convenience.
