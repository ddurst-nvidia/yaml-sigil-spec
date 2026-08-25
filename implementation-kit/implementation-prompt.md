# Reusable Language Implementation Prompt

Copy this prompt into an implementation repository and replace every angle-
bracketed placeholder before work begins.

---

Implement `YamlSigil.v1alpha1` in `<TARGET_LANGUAGE>` for
`<TARGET_ECOSYSTEM>`. Produce source, tests, and developer documentation only.
Do not publish compiled executables, executable WebAssembly, installers,
containers, or generated release artifacts as part of this task.

Use these repositories at these exact full 40-character Git commit IDs:

- Authoritative specification:
  [`yaml-sigil-spec`](https://github.com/NVIDIA/yaml-sigil-spec) at
  `<YAML_SIGIL_SPEC_COMMIT>`.
- Interface and vocabulary companion:
  [`yaml-sigil-traits`](https://github.com/NVIDIA/yaml-sigil-traits) at
  `<YAML_SIGIL_TRAITS_COMMIT>`.
- Reference implementation:
  [`yaml-sigil-rs`](https://github.com/NVIDIA/yaml-sigil-rs) at
  `<YAML_SIGIL_RS_COMMIT>`.

Stop if any placeholder remains or if any ref resolves through `main`, another
floating branch, or an unresolved tag. Record the three resolved commit IDs in
the implementation's developer documentation and test metadata. Do not copy
repository-local branches, private review context, credentials, internal URLs,
or untracked files.

Treat the specification commit as authoritative. Use the traits repository to
understand portable interface vocabulary and use the Rust repository as a
behavioral reference. Neither companion overrides the specification. Report a
conflict instead of silently following a companion.

Generate protobuf message and enum types from the specification's `proto/`
tree with the `<GO|PYTHON|TYPESCRIPT>` message-only Buf v2 template under
`implementation-kit/`. Pin the template's remote plugin version and revision,
or substitute an equivalently pinned local generator as documented by the kit.
For Go, replace the placeholder `go_package_prefix` with this implementation's
real module path. Add and pin the required generated-code runtime dependency.
Do not generate or hand-write gRPC, Connect, HTTP, IPC, or other transport
stubs for `SigningService`, `TranscriptionService`, or `VerificationService`.

> [!CAUTION]
> Do not structure `VerifyFromPreVerify` as an RPC. It must remain an
> in-process, same-verifier-instance operation over opaque successful
> pre-verification state. Do not register it over gRPC, Connect, HTTP, IPC, or
> another public transport. Generated `VerifyFromPreVerifyRequest` and
> `VerifyFromPreVerifyResponse` messages are data types only.

Implement the Signing API, Transcription API, Verification API, Artifact
Decomposition, YAML and protobuf forms, transcoding rules, conformance
profiles, base64 profile, `keyid` constraints, algorithm profiles, parser
resource bounds, and verified-payload isolation exactly as defined by the
pinned specification. Generated protobuf types provide serialization only.
They do not implement any YamlSigil semantic or security rule.

Use the pinned specification commit's
`implementation-kit/conformance-manifest.json` to discover fixture cases.
Follow every linked suite `README.md` and supporting sidecar for invocation
material, precise expected behavior, and provenance. The normative fixture
documentation and fixture bytes override the non-normative manifest if they
disagree. Do not reinterpret the manifest as a new universal test-runner
protocol.

Build a functional conformance harness appropriate to `<TARGET_LANGUAGE>`.
Exercise every manifest case in each stated form and profile, including
operation outcomes, invocation errors, verifier states, returned-byte rules,
and property assertions. Add language-specific negative tests for parser
depth, constructed-node, alias-expansion, memory, input-size, and time bounds
where the portable fixtures intentionally cannot prescribe a library-specific
counter.

Keep compatibility evidence separate from security assurance. Passing
conformance establishes functional compatibility, not production readiness.
After the initial functional implementation passes conformance, require an
independent language- and ecosystem-specific security assessment before
production use or any production-readiness claim. At minimum, the assessment
must cover:

- Static analysis.
- Direct and transitive dependency review.
- Coverage-guided fuzzing of parsing, decomposition, transcoding, and wire
  decoding boundaries.
- YAML parser behavior and explicit resource-bound enforcement.
- Cryptographic library selection, key validation, API use, error mapping,
  and secret handling.
- Transport and registration exposure, including proof that
  `VerifyFromPreVerify` is not remotely callable.

Resolve assessment findings and document the residual risks before making a
production-readiness claim. Do not describe conformance success alone as a
security audit, certification, or approval.

Finish by reporting the exact three input commits, generated-code tool and
runtime versions, test commands, fixture coverage, known deviations, and
security-assessment status. Do not claim completion while a normative behavior
or fixture case remains unimplemented.

---
