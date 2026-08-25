# Language Implementation Kit

> [!WARNING]
> This directory is non-normative. The specification, API companions,
> protobuf IDL, JSON Schema, and conformance fixture documentation are
> authoritative. If this kit disagrees with an authoritative source, follow
> the authoritative source.

This source-only kit helps start independent Go, Python, and TypeScript
implementations of `YamlSigil.v1alpha1`. It provides generation templates,
a reusable implementation prompt, and a machine-readable fixture index. It
does not define another API, wire format, schema, or conformance-runner
protocol.

## Contents

| File | Purpose |
| --- | --- |
| [`implementation-prompt.md`](./implementation-prompt.md) | Reusable prompt for a language implementation. |
| [`buf.gen.go.yaml`](./buf.gen.go.yaml) | Go protobuf message generation. |
| [`buf.gen.python.yaml`](./buf.gen.python.yaml) | Python protobuf message and type-stub generation. |
| [`buf.gen.typescript.yaml`](./buf.gen.typescript.yaml) | TypeScript protobuf message generation. |
| [`conformance-manifest.json`](./conformance-manifest.json) | Non-normative fixture discovery and expectation index. |

## Pin every input

Record exact full Git commit IDs for all three repositories before starting an
implementation:

- The authoritative
  [`yaml-sigil-spec`](https://github.com/NVIDIA/yaml-sigil-spec)
  specification.
- The [`yaml-sigil-traits`](https://github.com/NVIDIA/yaml-sigil-traits)
  vocabulary and interface companion.
- The [`yaml-sigil-rs`](https://github.com/NVIDIA/yaml-sigil-rs) reference
  implementation.

Do not use `main`, another floating branch, or an unresolved tag as an
implementation input. The traits and Rust repositories are implementation
aids. The specification commit remains authoritative if the repositories
disagree.

## Generate protobuf messages

Run generation from the repository root. Always direct output to a disposable
or downstream-owned directory. For example:

```shell
kit_output_dir="$(mktemp -d)"
buf generate --template implementation-kit/buf.gen.go.yaml \
  --output "$kit_output_dir"
```

Use `buf.gen.python.yaml` or `buf.gen.typescript.yaml` in the same command for
the other languages. Do not add generated output to this specification
repository.

The templates pin these public Buf plugins and revision `1`:

| Language | Plugins |
| --- | --- |
| Go | [`protocolbuffers/go:v1.36.12`](https://github.com/bufbuild/plugins/tree/main/plugins/protocolbuffers/go/v1.36.12). |
| Python | [`protocolbuffers/python:v36.0`](https://github.com/bufbuild/plugins/tree/main/plugins/protocolbuffers/python/v36.0) and [`protocolbuffers/pyi:v36.0`](https://github.com/bufbuild/plugins/tree/main/plugins/protocolbuffers/pyi/v36.0). |
| TypeScript | [`bufbuild/es:v2.14.0`](https://github.com/bufbuild/plugins/tree/main/plugins/bufbuild/es/v2.14.0). |

The Go template uses `paths=source_relative`. Before generating downstream
code, replace the managed-mode `go_package_prefix` value
`example.invalid/replace-with-your-module` with the implementation's real Go
module path. The sentinel is not a publishable import path.

Each template excludes `SigningService`, `TranscriptionService`, and
`VerificationService` by fully qualified type name. The request and response
messages remain generated, including `VerifyFromPreVerifyRequest` and
`VerifyFromPreVerifyResponse`. Treat them as in-process data types, not as
transport bindings. Generated output must contain no service descriptors,
gRPC stubs, or Connect stubs.

> [!IMPORTANT]
> Do not structure `VerifyFromPreVerify` as an RPC. Keep it in-process on the
> same verifier instance and pass only opaque state produced by that instance.
> Do not register it over gRPC, Connect, HTTP, IPC, or another public
> transport.

## Substitute local generators

Remote generation sends the protobuf image to the selected public Buf plugin.
When local generation is required, keep the output, options, and
`exclude_types` entries unchanged, remove `revision`, and replace the plugin
selector:

| Language | Local selector |
| --- | --- |
| Go | Replace `remote` with `local: protoc-gen-go`. |
| Python | Replace the two `remote` entries with `protoc_builtin: python` and `protoc_builtin: pyi`, and set `protoc_path` on each entry. |
| TypeScript | Replace `remote` with `local: protoc-gen-es`. |

Pin the local generator versions through the downstream project's toolchain.
The local versions must implement the options used by the corresponding
template.

## Add generated-code runtimes

Generated source needs a language runtime that the generator itself does not
vendor:

| Language | Downstream runtime dependency |
| --- | --- |
| Go | `google.golang.org/protobuf`. |
| Python | The `protobuf` distribution that satisfies the generated runtime-version check. |
| TypeScript | `@bufbuild/protobuf`. |

Pin and review those dependencies under the downstream project's dependency
policy. The message-only templates do not require a gRPC or Connect runtime.

Generated protobuf types implement serialization, field access, and descriptor
metadata. They do not implement YamlSigil semantics. In particular, they do
not perform Artifact Decomposition, YAML signature-carrier safety checks,
conformance-profile handling, base64 validation, algorithm validation,
cryptographic verification, verified-payload isolation, or reader-side trust
policy.

## Use the conformance manifest

[`conformance-manifest.json`](./conformance-manifest.json) groups all fixture
assets into stable cases. Paths are relative to the repository root. Each case
names its operation, form and profile context, and typed expectations or
property assertions.

The manifest is an exhaustive discovery index for the specification commit
that contains it. It is not a new universal runner protocol. Follow the linked
suite `README.md` files and text sidecars for invocation inputs, exact
expectations, and provenance. Those sources win if the manifest disagrees.

## Separate compatibility from security assurance

Passing the functional conformance fixtures establishes compatibility only.
After the implementation passes conformance, commission an independent
language- and ecosystem-specific security assessment before production use or
any production-readiness claim. The assessment must include static analysis,
dependency review, fuzzing, parser and resource-bound review, cryptographic API
review, and transport exposure review. Resolve its findings under the
downstream project's security policy.
