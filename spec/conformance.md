# Conformance Rules (v0.0.3 candidate)

Structural validity against `procedure-manifest.schema.json` is **necessary but
not sufficient**. A manifest is **arbitrable** only if it satisfies the rules in
scope for that manifest. A manifest failing a normative rule is *refused at
formation time* — the refusal boundary, executable before money moves.

Rules are stated as MUST requirements. Rule ids are stable and citable. C1–C15
apply to every manifest, subject to the conditional clauses written in the
individual rules. C16–C21 apply to every manifest containing one or more
`llm_judge` requirements. Measurement and attestation requirements acquire
through their declared sources; they do not inherit C16–C21 merely by appearing
beside an LLM-judged requirement.

The reference validator currently enforces C1–C15. C16–C21 are normative
v0.0.3 candidates and are **not yet reference-enforced**; that marker is removed
only when the eligibility checker is integrated. Their authoritative text is
[`spec/acquisition.md`](./acquisition.md).

For every manifest containing an `llm_judge`, formation MUST also refuse any
declared `provenance_profile` whose registry entry does not establish all four
eligibility terms for that judge:
`authorized_execution`, `exact_request_binding`,
`unique_terminal_execution`, and `sufficient_scope`. Registration alone
does not confer sufficiency. This is the static profile-sufficiency check that
precedes the C16–C21 runtime/replay obligations.

| Id | Rule | Status | Enforcement tier |
|----|------|--------|------------------|
| **C1** | Exactly one party MUST have role `payer` and exactly one MUST have role `payee`. | reference-enforced | formation/static |
| **C2** | Every requirement with `evaluation.method = "measurement"` MUST reference, via `measurement_source_id`, a source declared in `measurement_sources`. | reference-enforced | formation/static |
| **C3** | Every requirement with `evaluation.method = "llm_judge"` MUST reference, via `judge_prompt_id`, a prompt declared in `judge.prompts` with `role = "evaluation"`. | reference-enforced | formation/static |
| **C4** | Every panel member's `runs` MUST be odd. *(Revised in v0.0.2: the odd-panel-under-majority clause is withdrawn — across-panel aggregation is unanimous-only, so panel size is unconstrained.)* | reference-enforced | formation/static |
| **C5** | *(Revised in v0.0.2 — now universal.)* `judge.fallback_ladder` MUST be present and non-empty for **every** manifest, regardless of hosting: pinned hashes pin a name, not continued availability, so every judge needs a liveness policy agreed at formation. Every ladder entry with `action = "substitute"` MUST include a pinned `substitute` (with `version`; with `weights_hash` if self-hosted). | reference-enforced | formation/static |
| **C6** | If `remedy.structure = "challenge_window_escrow"`, `challenge_window_seconds` MUST be present. If `"streaming"`, `stream` MUST be present. If `"bonded_finality"`, `bond` MUST be present. | reference-enforced | formation/static |
| **C7** | If `adjudication.default_rule.outcome = "split"`, `split_ratio_payer_bps` MUST be present. If `adjudication.outcome_rule.rule = "weighted_threshold"`, `weights` MUST cover every requirement id exactly, and `pass_threshold_bps` MUST be present. | reference-enforced | formation/static |
| **C8** | Every evidence item's `submitter_role` MUST correspond to at least one declared party with that role. Every requirement with `evaluation.method = "attestation"` MUST name, via `attestor_role_name`, a declared attestor party's `name`. | reference-enforced | formation/static |
| **C9** | `judge.sampling.temperature` MUST be `0`. | reference-enforced | formation/static |
| **C10** | If any requirement uses `llm_judge`, `adjudication.injection_screening.enabled` MUST be `true`. | reference-enforced | formation/static |
| **C11** | Every requirement MUST be supported by at least one evidence item (`supports` containing its id) **or** be a `measurement`/`attestation` requirement. Every evidence item of `type = "url"` supporting an `llm_judge` requirement MUST have `transformation = "render_screenshot"`. | reference-enforced | formation/static |
| **C12** | Every panel model (and every self-hosted substitute) with `hosting = "self_hosted"` MUST include `weights_hash`. No model `version` may be `"latest"` or empty. | reference-enforced | formation/static |
| **C13** | *(New in v0.0.2.)* `adjudication.policy_on_unresolved.policy` MUST be `"resolve_against_burden"`. The remaining enum values (`count_as_pass`, `count_as_fail`, `escalate_to_default_outcome`) are reserved: schema-known but **refused**, because an unimplemented policy is not an executable procedure. Implementations of reserved policies are welcome as contributions (see `spec/adjudication.md` §3). | reference-enforced | formation/static |
| **C14** | *(New in v0.0.2.)* Aggregation MUST be fully parameterized: `within_judge.rule = "majority_with_dissent_cap"` with `max_dissents` present and `max_dissents < runs / 2` for **every** panel judge (so a tolerated majority is always a strict majority); `across_panel.rule = "unanimous"`. | reference-enforced | formation/static |
| **C15** | *(New in v0.0.2.)* Every evidence item with `transformation != "none"` MUST include `transformation_pin` (pinned tool+version or content hash). An outcome-relevant transformation is part of the committed procedure; a deterministic judge over an unpinned, lossy, or adversarial transformation is not a closed procedure. | reference-enforced | formation/static |
| **C16** | *(New in v0.0.3.)* Every LLM-judge run MUST have a deterministic `run_id` derivable from committed pre-result state. The committed provenance profile MUST identify an authoritative predecessor-state/commitment source and an independently verifiable ordering mechanism establishing that predecessor before execution acquired authority. A bare timestamp, `committed_at`, executor assertion, or local-clock comparison MUST NOT establish this relation by itself. If predecessor authority/order cannot be established, the run is `UNRESOLVED`. | candidate — not yet reference-enforced | formation/static surface + runtime evidence |
| **C17** | *(New in v0.0.3.)* Every claimed execution MUST bind a `request_hash` covering every outcome-relevant execution input committed by the manifest. | candidate — not yet reference-enforced | formation/static surface + runtime evidence |
| **C18** | *(New in v0.0.3.)* Hosted execution MUST carry provider/provenance admission attestation; requester submission alone MUST NOT prove admission. `deterministic_reexecution` MAY instead establish admission from committed lineage plus deterministic recomputation. For each authorized `attempt_id`, the selected profile MUST establish at most one authoritative claim or make multiple authentic claims detectable as equivocation with a committed fail-closed consequence. | candidate — not yet reference-enforced | arbitrator runtime |
| **C19** | *(New in v0.0.3.)* The provenance profile MUST establish a unique terminal state for each claimed attempt, or make terminal equivocation detectable with a committed fail-closed consequence. Byte-identical duplicate terminal records are redundant copies of the same record and are not equivocation; distinct conflicting canonical terminal-state records are. | candidate — not yet reference-enforced | arbitrator runtime |
| **C20** | *(New in v0.0.3.)* Retry MUST be authorized only by a manifest-committed, independently verifiable terminal absence/failure state. Caller-local timeout or executor assertion MUST NOT authorize another attempt. | candidate — not yet reference-enforced | arbitrator runtime |
| **C21** | *(New in v0.0.3.)* Outcome-relevant scope MUST be machine-evaluable. In v0.0.3 `required_scope` and `observed_scope` are flat sets of enumerated tokens and satisfaction is exactly `required_scope ⊆ observed_scope`; richer scope forms/predicates are reserved and refused at formation. | candidate — not yet reference-enforced | formation/static surface + runtime evidence |

## Semantic conformance

The strengthened §11 property is the **replay-checkable** enforcement tier:
given a frozen acquisition transcript, independent implementations must agree on
which observations acquire authority before the existing v0.0.2 replay begins.

v0.0.2 established the downstream semantic conformance property:

> Given the same committed manifest and the same complete run observations,
> two independent conforming implementations MUST derive the same requirement
> resolutions, the same contract outcome, and the same authorized remedy.

v0.0.3 strengthens that property one boundary earlier:

> **Given the same committed manifest and the same acquisition/transcript
> evidence, two independent conforming implementations MUST admit the same
> observations as authoritative and MUST derive the same requirement
> resolutions, contract outcome, and authorized remedy.**

The required acquisition regression set is F1, F1b, F2–F6, and F6b in
`spec/acquisition.md` §12. In particular, F1b requires claim-level equivocation
to fail closed, and F6b prevents a bare numeric timestamp comparison from
manufacturing predecessor authority/order.

The reference validator ships a deterministic replay harness
(`validator/src/replay.ts`) implementing the normative state machine in
`spec/adjudication.md`, plus regression fixtures under `validator/fixtures/`.
Fixture #1 is the observation set that falsified v0.0.1's closure claim in the
RFC thread (an unspecified confidence-reduction step allowed mean vs. median to
flip the remedy). Under v0.0.2 the same observations produce a single
deterministic outcome, and the harness asserts the result is invariant under
permutation of the (now evidence-only) confidence values.

Independent implementations SHOULD run their pipelines against these fixtures;
divergence from the expected outcomes means an outcome-relevant transformation
remains outside the manifest, which is a bug in the implementation or a defect
report against this spec — both are welcome.

## Notes

- **C4/C9** exist because reproducibility in v0.0.2 is achieved by odd-run
  aggregation at temperature 0 with published transcripts — auditability, not
  bit-exactness. The spec explicitly disclaims byte-identical reruns; see
  `spec/adjudication.md` §4.
- **C5/C12** encode the pinning-vs-capability tradeoff and (as of v0.0.2) the
  model-retirement reality: liveness policies are universal.
- **C10/C11/C15** are the minimum injection-and-provenance posture: screening
  on, rendered (not raw) representation for rich remote content reaching an
  LLM judge, and pinned transformations with originals preserved.
- **Refusal semantics.** Formation-time refusal means the validator rejects the
  manifest before signing. If a defect is discovered post-signing, the
  arbitrator returns ERC-792 ruling 0 (`RefusedToArbitrate`) and funds follow
  `remedy.ruling_map.refused`.

## Anti-goals

Conformance does not certify that a rubric is *wise*, that prompts are
well-crafted, or that a burden allocation is *fair*. It certifies that the
procedure is complete, closed, and executable — that nothing essential to
rendering a decision is left undefined. **Conforming → procedurally closed and
executable; never → fair or correct.** Quality of clauses is the concern of a
reviewed clause library, which this spec enables but does not contain.
