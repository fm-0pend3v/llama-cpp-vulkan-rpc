# llama-cpp-vulkan-rpc

[![Build and Upload to COPR](https://github.com/fm-0pend3v/llama-cpp-vulkan-rpc/actions/workflows/build.yml/badge.svg)](https://github.com/fm-0pend3v/llama-cpp-vulkan-rpc/actions/workflows/build.yml)

Automated RPM builds of [llama.cpp](https://github.com/ggml-org/llama.cpp) for **Fedora**, compiled with **Vulkan GPU acceleration** (`-DGGML_VULKAN=ON`) and **distributed RPC server support** (`-DGGML_RPC=ON`).

This repository automatically syncs with upstream stable releases and builds portable RPM packages distributed via **Fedora Copr**.

---

## Features & Build Configuration

* **Vulkan Backend (`-DGGML_VULKAN=ON`):** Enables hardware-accelerated LLM inference on any Vulkan-capable GPU (AMD, Intel, NVIDIA) without requiring proprietary CUDA or ROCm toolkits.
* **Distributed RPC Support (`-DGGML_RPC=ON`):** Includes `ggml-rpc-server` and RPC client capabilities, allowing you to offload model layers across multiple machines over the network.
* **Dynamic CPU Backends (`-DGGML_BACKEND_DL=ON` & `-DGGML_CPU_ALL_VARIANTS=ON`):** Ships portable binaries that dynamically load the optimal CPU instruction set (AVX2, AVX-512, etc.) at runtime.
* **Private Shared Libraries (`llama-cpp-vulkan-rpc-libs`):** Installs `libggml*.so` and `libllama*.so` into a dedicated private directory (`%{_libdir}/llama-cpp-vulkan-rpc`) registered via `ld.so.conf.d`, preventing library conflicts with other `ggml`-based packages on Fedora (such as `whisper.cpp`).
* **Bundled Web UI:** Pre-stages the official `llama-server` web interface at build time for reliable offline compilation.

---

## Included Binaries

Installing `llama-cpp-vulkan-rpc` provides the standard suite of `llama.cpp` tools along with the RPC worker daemon:

* **`ggml-rpc-server`** — Distributed RPC worker server to expose local GPU/CPU memory and compute to a primary `llama.cpp` instance.
* **`llama-server`** — OpenAI-compatible HTTP API server with built-in web chat UI.
* **`llama-cli`** — Interactive command-line interface for running models locally or across RPC nodes.
* **`llama-quantize`** — Tool to quantize GGUF models to smaller precisions.
* **`llama-gguf-split`** — Utility to split or merge large GGUF model files.
* **`llama-bench`** — Performance benchmarking tool for prompt processing and token generation.
* **`llama-perplexity`** — Model quality and perplexity evaluation tool.
* **`llama-embedding`** — Text embedding extraction utility.
* **`llama-tts`** — Text-to-speech inference tool.
* *...and all other standard `llama-*` utilities.*

---

## Installation (Fedora)

Enable the Copr repository and install the package using `dnf`:

```bash
sudo dnf copr enable -y fm-0p3n-d3v/llama-cpp-vulkan-rpc
sudo dnf install -y llama-cpp-vulkan-rpc
```

To verify the installed build number and enabled backends:

```bash
llama-cli --version
```

---

## Quick Usage: Distributed RPC Inference

### 1. Start the RPC Worker Node (Remote Machine)
On the machine sharing its GPU/RAM resources, start `ggml-rpc-server`:

```bash
ggml-rpc-server -H 0.0.0.0 -p 50052
```

### 2. Connect from the Main Node
On your primary machine, pass the `--rpc` flag to `llama-cli` or `llama-server` pointing to one or more worker nodes:

```bash
llama-server -m /path/to/model.gguf --rpc 192.168.1.50:50052 -ngl 99
```

---

## How Automation Works

1. **Monthly & On-Demand Checks:** A GitHub Actions workflow (`.github/workflows/build.yml`) checks `ggml-org/llama.cpp` for the latest stable release (`vX.Y.Z`) and resolves its pinned build tag (`bXXXX`).
2. **Upstream Spec Sync:** It fetches the base `llama-cpp.spec` from [Unrest6585/fedora-llamacpp](https://github.com/Unrest6585/fedora-llamacpp), applies the RPC build flags, updates package metadata, and increments the RPM `Release` counter automatically.
3. **SRPM & Copr Build:** It bundles the source tarball and prebuilt UI assets into a `.src.rpm` and triggers a clean build on **Fedora Copr**.
4. **RPM Distribution:** Once the build succeeds, the resulting RPM packages are automatically published and ready to download or install via `dnf` from the [fm-0p3n-d3v/llama-cpp-vulkan-rpc Copr repository](https://copr.fedorainfracloud.org/coprs/fm-0p3n-d3v/llama-cpp-vulkan-rpc/).

---

## Credits & License

* **Upstream Project:** [ggml-org/llama.cpp](https://github.com/ggml-org/llama.cpp) (MIT License).
* **Packaging Base:** Based on the RPM spec file and build architecture originally created by [Unrest6585](https://github.com/Unrest6585/fedora-llamacpp) (also distributed via [sneed/llama-cpp-vulkan on Copr](https://copr.fedorainfracloud.org/coprs/sneed/llama-cpp-vulkan)), modified to enable `GGML_RPC=ON` and package `ggml-rpc-server`.
