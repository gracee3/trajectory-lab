# Environment lineage and reuse plan

Status: design and source audit, 2026-09-06. No images were built, pulled or tested for this review. Registry availability, installed inventories, disk sizes and hardware compatibility remain unverified.

This plan covers the eight projects in [Project Scenarios](PROJECT_SCENARIOS.md). Keep the existing five areas per project; environment planning does not expand that backlog. The immediate objective is a small, reusable environment with a measurable configuration result, then incremental extensions for these projects.

## Decision: one run contract, several image families

A Codex base image is useful for new environments. It should not require rebuilding every existing project on the same distribution.

Use two entry paths:

- **New environments:** a pinned Ubuntu 26.04 image, pinned Codex CLI and common command-line tools, extended with selected language and project dependencies.
- **Existing project environments:** preserve the project's original parent and dependency stack, then add the same versioned Codex support payload and run contract.

The support payload means a compatible CLI artifact, launch configuration and smoke checks. It is not a promise that one binary works on every architecture or libc. Verify compatibility for each family. Credentials belong to runtime injection, never an image layer.

Image inheritance has one final parent per stage. “Modular” means reusable build stages, recipes and dependency manifests; it does not mean merging arbitrary Rust, CUDA and Python images. Keep services and reference evaluators separate where they already have useful boundaries.

Ubuntu 26.04 is the preferred new-family target. Its exact image digest and compatibility are a selection gate, not an established baseline. Existing Ubuntu 22.04/24.04 and Debian images remain legitimate branches.

## Common run contract

Every scenario declares:

- Source repository and immutable commit; image digest and target architecture.
- Codex artifact/version, launch mode and model/configuration identifiers.
- Workspace location, writable home, user, resource limits and network policy.
- Available tools, installer sets, data/model assets and their checksums.
- Setup recipe, observable starting condition, prompt and termination policy.
- Separate acceptance checks, outcome artifacts and hardware requirements.
- Capture destination and known event gaps.

The initial capture candidate remains `codex exec --json`, subject to the installed-version validation in [Research Report](RESEARCH_REPORT.md). Collect its raw stream, stderr and process status outside the candidate workspace. Label runner timestamps and metadata separately from native events. A common container image does not itself guarantee event coverage.

The agent needs access to its model endpoint even when package installation is offline. Treat model access, package access and project-service access as separate policies.

Start the common image with shell, Git, ripgrep, certificates and the small tools required by the run contract. Add compilers, browser engines, Torch and CUDA only in relevant profiles. A utility Python interpreter, if needed by scripts, must not silently determine a project's Python environment.

## Existing Dockerfiles and extent of reuse

The inspected main snapshots contain **13 Dockerfiles across five repositories**, plus Supermicro's Compose definition. Three projects need new image recipes for the inspected source. “Reusable” below is a source-based assessment, not a successful container build.

### WhisperX-batch

Evidence: [Dockerfile.whisperx-torch280-cu128](https://github.com/gracee3/whisperX-batch/blob/b019546fbae413711615577c387906ea06af198f/Dockerfile.whisperx-torch280-cu128).

An existing multistage recipe builds on CUDA 12.8.1/cuDNN devel and runtime images with Ubuntu 22.04. It installs Python 3.11, Torch and torchaudio 2.8.0+cu128, CTranslate2 4.7.1, faster-whisper 1.2.1, WhisperX 3.8.2 and pyannote.audio 4.0.4. The runtime receives the prepared virtual environment and FFmpeg.

- **Reuse extent:** a strong starting parent for the project's full GPU environment. Preserve Ubuntu 22.04 initially and add the Codex payload, development utilities and an appropriate launcher.
- **Development distinction:** the slim final stage is not the builder. Native dependency repair may require compiler/header additions; Python configuration work may only need a writable virtual environment and installers.
- **Reproducibility gaps:** CUDA parents use tags rather than digests; some Python requirements remain ranges; Python provisioning can use a PPA. Resolve and archive the complete chosen dependency set.
- **Offline needs:** Python 3.11 runtime/install source, all matching wheels, apt dependency closure, FFmpeg, and separately identified model assets. Existing no-cache pip options mean this Dockerfile is not already an installer warehouse.
- **Lighter option:** isolate scheduling, argument handling or metadata issues in the Python family when their acceptance checks do not require real inference.

Its import/version smoke checks do not establish successful GPU inference. Reusing this stack avoids making an Ubuntu upgrade part of every WhisperX scenario.

### native-asr

Evidence: [Sherpa-ONNX](https://github.com/gracee3/native-asr/blob/b5b08cb5ee69998d349ba217048005ea53a5fa4e/docker/sherpa-onnx/Dockerfile), [NeMo-Speech.cpp](https://github.com/gracee3/native-asr/blob/b5b08cb5ee69998d349ba217048005ea53a5fa4e/docker/nemo-speech/Dockerfile), [whisper.cpp](https://github.com/gracee3/native-asr/blob/b5b08cb5ee69998d349ba217048005ea53a5fa4e/docker/whisper-cpp/Dockerfile), and [Moonshine](https://github.com/gracee3/native-asr/blob/b5b08cb5ee69998d349ba217048005ea53a5fa4e/docker/moonshine/Dockerfile).

All four recipes use a digest-pinned Ubuntu 24.04 base and separate native builds from small runtime images. They already provide useful engine boundaries:

- Sherpa-ONNX 1.13.2 with ONNX Runtime 1.24.4.
- NeMo-Speech.cpp 1.0.0 with pinned ggml source and explicit CPU build settings.
- whisper.cpp 1.9.2, built without CUDA.
- Moonshine 0.1.1 from a checksum-verified SDK archive and a C++ wrapper.

**Reuse extent:** retain these as engine services for orchestration scenarios, or extend an individual runtime for executable/configuration diagnosis. For C++ fixes, derive a development profile from the corresponding builder and retain source, headers and compilers. The runtime images use UID 65532 and recognizer entrypoints; Codex requires an explicit launcher and writable home/workspace.

These native engines do not need a Python Torch stack. Offline builds must also capture nested upstream dependencies, such as ONNX Runtime and ggml, rather than just the top-level repository.

The TUI separately pins Rust 1.97.1 in [tui/rust-toolchain.toml](https://github.com/gracee3/native-asr/blob/b5b08cb5ee69998d349ba217048005ea53a5fa4e/tui/rust-toolchain.toml); it can share a Rust toolchain profile while retaining its own system dependencies. Engine model assets remain separate, versioned inputs.

### Qwen38 INT8 Lab

Evidence: [quant](https://github.com/gracee3/qwen38-int8-lab/blob/44949714ff6dde6db866532f8600129af689361e/docker/quant/Dockerfile), [vLLM](https://github.com/gracee3/qwen38-int8-lab/blob/44949714ff6dde6db866532f8600129af689361e/docker/vllm/Dockerfile), [evaluation](https://github.com/gracee3/qwen38-int8-lab/blob/44949714ff6dde6db866532f8600129af689361e/docker/eval/Dockerfile), [SM86 FP8](https://github.com/gracee3/qwen38-int8-lab/blob/44949714ff6dde6db866532f8600129af689361e/docker/vllm-fp8-sm86/Dockerfile), and [llama.cpp](https://github.com/gracee3/qwen38-int8-lab/blob/44949714ff6dde6db866532f8600129af689361e/docker/llama/Dockerfile).

This project already has much of the proposed lineage:

- Quantization and vLLM share a digest-pinned PyTorch 2.13.0 / CUDA 13.0 / cuDNN 9 parent, then install separate dependency locks.
- The quant image checks compressed-tensors 0.18.0; vLLM checks 0.17.0. Sharing a parent does not make their installed environments interchangeable.
- Evaluation extends a configurable local vLLM image and adds lm-eval dependencies and data.
- The SM86 FP8 extension expects a host CUDA toolkit mount for compilation.
- llama.cpp uses separately digest-pinned CUDA 13.3.0 Ubuntu 24.04 build/runtime images and a pinned source commit, with an SM86 target.

**Reuse extent:** extend the existing quant and serving images with the common Codex support payload. Keep them separate. Treat evaluation as an independent service when its checks are withheld. Retain llama.cpp as a distinct native family.

The evaluation and FP8 recipes default to a mutable local vLLM tag; record the actual parent digest. A digest in a label is not a pin on `FROM`. Evaluation also downloads NLTK data during build, so its offline inventory includes data, not only wheels.

The PyTorch parent's actual Python/OS inventory must be inspected before assigning a Python compatibility label; the Dockerfile tag alone does not specify it. FP8 toolkit mounts need exact toolkit/compiler provenance and per-run writable compilation caches. A host toolkit dependency is not yet a self-contained image.

Do not infer registry availability from repository-local image names.

### gpt-oss-rs

Evidence: [server Dockerfile](https://github.com/gracee3/gpt-oss-rs/blob/8b3ff46e25c213104219db8e9d390bc05dacf8bf/Dockerfile), [CPU oracle Dockerfile](https://github.com/gracee3/gpt-oss-rs/blob/8b3ff46e25c213104219db8e9d390bc05dacf8bf/oracle/Dockerfile.cpu), and [oracle dependency lock](https://github.com/gracee3/gpt-oss-rs/blob/8b3ff46e25c213104219db8e9d390bc05dacf8bf/oracle/requirements.cpu.lock).

The root recipe is CUDA-oriented: CUDA 13.0.1 on Ubuntu 24.04, a Rust build with the CUDA feature, and a small server runtime. Its image tags and rustup installation are not fully pinned, and its Cargo build does not use `--locked`.

**Reuse extent:** useful for CUDA server work after pinning and checking the intended feature combination. It does not establish a ready CPU/Tiger Lake/HET development environment. Source-editing scenarios need a builder-derived profile or a new Rust CPU profile; the final server image lacks Cargo and source.

The CPU oracle is a stronger reproducibility reference: digest-pinned Python 3.12.12 slim Bookworm, hash-locked Torch 2.12.1+cpu dependencies and a checksum-verified official gpt-oss source archive. It explicitly excludes several unrelated Python packages.

**Keep the oracle separate** when it supplies authoritative answers. Its read-only files are not hidden from an agent sharing the image. Reuse it as an evaluator, with its own offline wheels and source archive.

CPU instruction sets, memory capacity and GPU support remain host requirements. A CPU image cannot supply AVX extensions missing from the host. The repository's Rust manifests do not establish an exact shared toolchain pin for this project; select and validate one before claiming compatibility.

### Supermicro observability

Evidence: [gpu-exporter/Dockerfile](https://github.com/gracee3/supermicro-observability/blob/a1d239f53b1e2eece262ab611ea4f46bf5f9e1c0/gpu-exporter/Dockerfile), [gpu-exporter/Cargo.toml](https://github.com/gracee3/supermicro-observability/blob/a1d239f53b1e2eece262ab611ea4f46bf5f9e1c0/gpu-exporter/Cargo.toml), and [compose.yaml](https://github.com/gracee3/supermicro-observability/blob/a1d239f53b1e2eece262ab611ea4f46bf5f9e1c0/compose.yaml).

The custom GPU exporter image uses digest-pinned Debian Bookworm slim and copies an already-built musl binary. It is a packaging recipe, not a Rust builder. The Compose file already integrates pinned upstream monitoring services, including Prometheus, Grafana, node-exporter, NVML export, smartctl export and cAdvisor.

- **Reuse extent:** reuse upstream service images and selected configuration/dashboard assets as scenario services. Reuse the custom exporter runtime after supplying a verified binary.
- **New work needed:** a Rust/musl development profile for exporter changes; the manifest uses edition 2024 but does not pin an exact Rust toolchain.
- **Hardware distinction:** real GPU export needs appropriate NVIDIA utility exposure; it does not require installing Torch.
- **Isolation distinction:** the production Compose file uses host networking, host paths, devices and privileged capabilities for legitimate monitoring. Create a scenario-specific Compose definition with only the resources its checks require.
- **Lighter option:** configuration, parsing and metric-contract checks may use controlled inputs without running the whole monitoring installation.

Do not give the agent the production Docker socket. A read-only filesystem mount of that socket does not make Docker API operations read-only. Preserve production deployment configuration as evidence, rather than treating it as the general sandbox policy.

### Digital Liquid Light Lab

Evidence: [research branch snapshot](https://github.com/gracee3/digital-liquid-light-lab/tree/c5300491883993a66234f15e848ec5506adcdead), [rust-toolchain.toml](https://github.com/gracee3/digital-liquid-light-lab/blob/c5300491883993a66234f15e848ec5506adcdead/rust-toolchain.toml) and [docs/environment.md](https://github.com/gracee3/digital-liquid-light-lab/blob/c5300491883993a66234f15e848ec5506adcdead/docs/environment.md).

Main contains only the README at the inspected snapshot. The research and interactive source snapshots inspected have no Dockerfiles or Compose definitions.

**Reuse extent:** reuse Rust 1.97.1 and project build/diagnostic conventions; a new image recipe is required. Start with the shared Rust family for CPU numerical and simulation checks. Add a separate graphics profile for wgpu, Vulkan and desktop integration.

GPU/display acceptance must record the actual backend, adapter and driver. The project explicitly distinguishes automatic CPU-adapter fallback from intentional software rendering. A headless CPU test result cannot stand in for desktop/GPU acceptance.

Pin the source-bearing branch commit in a scenario; using current main would not provide the code under study.

### Mirabile

Evidence: [Cargo.toml](https://github.com/gracee3/mirabile/blob/a407a828a3cd685bfe9f6cdfbfac1a05f0d6eee0/Cargo.toml) and [scripts/check.sh](https://github.com/gracee3/mirabile/blob/a407a828a3cd685bfe9f6cdfbfac1a05f0d6eee0/scripts/check.sh); also the [product branch snapshot](https://github.com/gracee3/mirabile/tree/b8893e5574eb59234d3a02b2c055664ce5352b95).

Neither the inspected main nor product snapshot contains a Dockerfile or Compose definition. The workspace declares Rust 1.88 as a minimum, not an exact toolchain pin. Fast checks require Cargo, Git and Python 3.

**Reuse extent:** reuse the project's checks and dependency locks in a new Rust profile. Rust 1.97.1 is a candidate for sharing with the explicitly pinned projects, subject to validation here.

Add WASM/Trunk and a pinned browser only for web scenarios. Cache the matching browser executable and system libraries as well as Rust dependencies. Core semantic checks can avoid that larger environment. Independent reference-calculation dependencies, if used, should live with their evaluator rather than every agent image.

### Magnolia

Evidence: [rust-toolchain.toml](https://github.com/gracee3/magnolia/blob/b42316fa3b1f5fd30a387cd98a982ca61bc5ec74/rust-toolchain.toml), [Cargo.toml](https://github.com/gracee3/magnolia/blob/b42316fa3b1f5fd30a387cd98a982ca61bc5ec74/Cargo.toml) and [tests/e2e/package.json](https://github.com/gracee3/magnolia/blob/b42316fa3b1f5fd30a387cd98a982ca61bc5ec74/tests/e2e/package.json); also the [native-ASR branch snapshot](https://github.com/gracee3/magnolia/tree/7da122c2ac9c56557112862f96b2193742f09862).

Neither inspected snapshot contains a Dockerfile or Compose definition. Rust 1.97.1 is pinned. The browser test package pins Playwright 1.62.1 and declares Node >=24; choose an exact Node version for an image.

**Reuse extent:** share the Rust toolchain with native-asr's TUI and Liquid Light Lab, then add project-specific dependencies. Use separate additions for WASM/browser tests and native PipeWire/audio builds. Native compilation can need audio development libraries even when a selected check does not require microphone access.

Browser scenarios need the matching Playwright browser and OS libraries preloaded. Live audio/device behavior requires a declared integration environment; do not infer that from container-only unit tests.

The native-ASR branch is a separate dependency lane. Do not assume it can consume native-asr's Sherpa image unchanged merely because both use Sherpa; engine versions, interfaces and model provenance must match.

## Proposed families

These are proposed recipes, not existing image tags:

- **Common Ubuntu 26.04:** Codex and basic inspection tools; small configuration and prompt/data scenarios.
- **Rust 1.97.1:** directly matches three inspected toolchain pins. Add per-project Cargo dependency sets; validate before adopting it for other Rust projects.
- **Rust web:** Rust plus WASM/Trunk and browser dependencies. Magnolia additionally needs its selected Node/Playwright stack; Mirabile may use its own browser harness.
- **Rust/native audio:** Rust plus the C/C++ and audio libraries needed by the selected application. Reuse native-asr engine containers alongside it when appropriate.
- **Rust numerics/graphics:** CPU checks first; graphics and device integration are explicit extensions.
- **Python repair:** a selected Python runtime and complete offline installer sets. No Torch unless required.
- **WhisperX GPU:** preserve the existing Ubuntu 22.04 / Python 3.11 / Torch 2.8.0 / CUDA 12.8 family.
- **Qwen GPU:** preserve the existing PyTorch/CUDA parent with separate quant, serving and evaluation environments.
- **Native CUDA:** project-specific gpt-oss-rs or llama.cpp build/runtime families, with explicit architecture targets.
- **Services and evaluators:** monitoring services, native ASR engines and the CPU oracle retain independent images.

Do not build every family before the first scenario. The map identifies extension points; the first implementation should prove the common contract and one selected environment.

## Offline installer collection

Store one immutable copy of each unique artifact, keyed by checksum, and let multiple scenarios reference it through read-only mounts. The runner selects mounts before launch; Codex does not need Docker control to obtain them.

For Python/Torch, collect wheels rather than treating Torch as a Debian package. Each installer set must include the complete dependency closure for the selected Python ABI, architecture and CPU/CUDA build. An offline install can use `pip --no-index --find-links` against that set. A Torch wheel alone is insufficient.

Keep two useful modes:

- **Prepared environment:** dependencies already installed; focus on project behavior.
- **Dependency-repair environment:** the starting environment contains the intended mismatch or omission, and compatible installers are available for a genuine offline repair.

Select a small number of complete version combinations based on actual scenarios. Do not prebuild every Python × Torch × CUDA combination. Multiple versions are feasible, but archive size, installed environments and GPU libraries must be measured before choosing the retention budget.

The collection may include:

- Python runtimes, wheels and any required native build dependencies.
- Rust toolchains, target components, Cargo registry crates and pinned Git dependencies.
- Apt packages and their transitive dependencies, separated by distribution/release/architecture.
- CUDA runtime or development components, explicitly distinguishing execution from compilation.
- Browser binaries, Node packages, FFmpeg and other native libraries.
- Model weights, tokenizers, small data fixtures and evaluation data, each with provenance.

A cache is not proof of offline completeness. Cargo build scripts, CMake dependency downloads, browser installers and data downloads can still reach the network. Prove the chosen recipe with external package access disabled.

Do not mix Ubuntu 22.04/24.04/26.04 Debian-package sets. Do not deduplicate installed virtual environments by assuming wheels or native libraries are interchangeable. Reuse identical OCI layers and installer blobs where possible, while accounting for per-run writable storage.

## Healthy state and scenario injection

Version these separately:

- A known-good source snapshot and environment.
- A setup recipe that introduces the intended fault or starting state.
- The agent-visible prompt, files and permitted resources.
- A protected evaluator and expected acceptance behavior.

Deriving a broken state from a known-good project snapshot is a good default. Pin the commit and image digest; do not use moving main as the definition of a scenario. Some historical problems will be better represented by an earlier snapshot or a small extraction, so fault injection is not mandatory.

Run private setup outside the agent container, then expose only its resulting state. Removing a secret script in a later Docker layer does not remove it from earlier layers. Keep answer-bearing history, patches, evaluator files and setup records out of the candidate workspace and image.

When history would reveal the repair, materialize only the intended source snapshot and, if useful, initialize a fresh Git repository for candidate diffs. Provide the agent enough observable evidence to diagnose the problem without providing the setup recipe.

Use a fresh container/workspace per attempt. Share immutable images and installers, not mutable databases, virtual environments or compiler caches across attempts. Per-run writable caches may be seeded from an approved baseline.

Git stores recipes, manifests, pins, prompts and scenario definitions. An image registry stores built images; an artifact store holds large installers and model/data assets. The run bundle links their exact identities.

## First measurable gates

Before claiming a reusable baseline:

1. **Launch and capture:** the selected Codex artifact starts in the image, executes a small tool task, and produces the expected exposed events plus separately captured stderr/process outcome.
2. **Healthy project check:** one pinned source snapshot passes its declared acceptance check in that environment.
3. **Offline provisioning:** the selected dependency set installs or restores without external package downloads; record missing artifacts and elapsed time.
4. **Scenario validity:** later, the injected starting state fails the intended check and the known reference repair passes it. Record outcome separately from capture fidelity.
5. **Repeatability:** a fresh attempt starts from the same declared state without consuming prior candidate changes.

Measure image size, installer-store size, cold setup time, warm launch time and repair/check time. For hardware tasks also record the host CPU, driver, GPU and relevant capabilities. Behavioral results can be deterministic without claiming identical performance across machines.

## Remaining decisions and scope limits

- Select the exact Ubuntu 26.04 digest, Codex release/artifact and target architecture.
- Inspect existing local or registry images before deciding whether to import them or rebuild recipes.
- Establish actual Python/OS inventories for inherited GPU parents.
- Choose one initial project/environment and its measurable setup check.
- Resolve complete artifact sets and measure storage before keeping several stack versions.
- Specify the available CPU/GPU/audio/display hosts and which checks genuinely need them.

The branch audit is targeted, not an exhaustive claim about every historical commit. Dockerfile contents match inspected main for native-asr's Tromso snapshot `f2edf64`, Qwen snapshots `c082748` and `55125dc`, and gpt-oss-rs's HET archive `7bb4593`. No Dockerfiles were found in the inspected Liquid Light Lab research/interactive snapshots, Mirabile product snapshot or Magnolia native-ASR snapshot.

This document is the environment-planning handoff. It authorizes no claim that an image builds, works offline or passes a hardware check; those require the measured gates above. Existing scenario evidence remains in [Project Scenarios](PROJECT_SCENARIOS.md).
