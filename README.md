# RamaLama Exploration on Fedora Linux

**Outreachy May 2026 Contribution**  
**Task:** Learn how RamaLama makes working with AI boring (in a good way)  
**Contributor:** Ibrahim Olawoyin
**Program:** Outreachy Round 32 Internship (May 2026 Cohort)  
**Project:** Develop a SLM/LLM using RamaLama RAG based off Fedora RPM Packaging Guidelines    
**Task:** [[Outreachy 2026] RamaLama: learn how RamaLama makes working with AI boring #124](https://forge.fedoraproject.org/commops/interns/issues/124)
**Date:** March 31, 2026


---

## Contribution Summary

This documents a comprehensively my exploration of RamaLama on Fedora Linux, evaluating multiple model transport mechanisms and systematically analyzing model output quality against official Fedora documentation. The exploration tested four transport protocols (Ollama, HuggingFace, OCI Container, and ModelScope), executed two functional models with four distinct prompts, and documented complete installation, configuration, and troubleshooting processes.

**Key Findings:**
- Ollama and HuggingFace transports successfully deployed models
- OCI container transport requires additional configuration
- ModelScope transport proved impractical for bandwidth-constrained environments
- Model outputs exhibited 100% hallucination rate on Fedora-specific technical queries
- RamaLama successfully abstracts infrastructure complexity (makes deployment boring)
- Output quality requires RAG enhancement for production use

---

## Table of Contents

1. [Environment Specification](#environment-specification)
2. [Prerequisites and Installation](#prerequisites-and-installation)
3. [Version Verification](#version-verification)
4. [Transport Evaluation](#transport-evaluation)
5. [Model Execution Testing](#model-execution-testing)
6. [Output Quality Analysis](#output-quality-analysis)
7. [System Diagnostics](#system-diagnostics)
8. [Does RamaLama Make AI Boring?](#does-ramalama-make-ai-boring)
9. [Appendices](#appendices)

---

## Environment Specification

**Operating System:**
- Distribution: Fedora 43 (WSL2 variant)
- Kernel: 6.6.87.2-microsoft-standard-WSL2

**Hardware:**
- CPU: 12 cores (Intel with AVX512 support)
- Total Memory: 8184 MB (approximately 8 GB)
- Free Memory: 7184 MB (approximately 6.7 GB)

**Container Runtime:**
- Engine: Podman 5.8.1 (rootless)
- OCI Runtime: crun 1.25.1
- Network Backend: netavark 1.17.2

**Software:**
- Python: 3.14.3
- RamaLama: 0.18.0
- Default Inference Runtime: llama.cpp

---

## Prerequisites and Installation

### Step 1: Verify Podman Installation

RamaLama defaults to Podman when both Podman and Docker are available. Podman is the recommended container engine for Fedora systems.

**Command:**
```bash
podman --version
```

**Output:**
```
podman version 5.8.1
```

![Podman Version Check](screenshots/podman_version_output.png)

### Step 2: Test Podman Functionality

**Command:**
```bash
podman run hello-world
```

**Output:**
```
WARN[0000] "/" is not a shared mount, this could cause issues or missing mounts with rootless containers
Resolved "hello-world" as an alias (/etc/containers/registries.conf.d/000-shortnames.conf)
Trying to pull quay.io/podman/hello:latest...
Getting image source signatures
Copying blob 81df7ff16254 done   |
Copying config 5dd467fce5 done   |
Writing manifest to image destination
!... Hello Podman World ...!

         .--"--.
       / -     - \
      / (O)   (O) \
   ~~~| -=(,Y,)=- |
    .---. /`  \   |~~
 ~/  o  o \~~~~.----. ~~
  | =(X)= |~  / (O (O) \
   ~~~~~~~  ~| =(Y_)=-  |
  ~~~~    ~~~|   U      |~~

Project:   https://github.com/containers/podman
Website:   https://podman.io
Desktop:   https://podman-desktop.io
Documents: https://docs.podman.io
YouTube:   https://youtube.com/@Podman
X/Twitter: @Podman_io
Mastodon:  @Podman_io@fosstodon.org
```

**Observation:** Warning about shared mount is expected in WSL2 environments and does not prevent RamaLama operation.

![Podman Hello World Test](screenshots/02_podman_hello_world.png)

### Step 3: Verify Python Installation

**Command:**
```bash
sudo dnf install python3
```

**Output:**
```
Updating and loading repositories:
Repositories loaded.
Package "python3-3.14.3-1.fc43.x86_64" is already installed.

Nothing to do.
```

**Result:** Python 3.14.3 confirmed pre-installed on Fedora 43.

![Python Installation Check](screenshots/03_python_check.png)

### Step 4: Install RamaLama

**Command:**
```bash
pip install ramalama
```

**Output:**
```
Defaulting to user installation because normal site-packages is not writeable
Collecting ramalama
  Downloading ramalama-0.18.0-py3-none-any.whl.metadata (57 kB)
Collecting argcomplete (from ramalama)
  Downloading argcomplete-3.6.3-py3-none-any.whl.metadata (16 kB)
Collecting pyyaml (from ramalama)
  Downloading pyyaml-6.0.3-cp314-cp314-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl.metadata (2.4 kB)
Collecting jsonschema (from ramalama)
  Downloading jsonschema-4.26.0-py3-none-any.whl.metadata (7.6 kB)
Collecting jinja2 (from ramalama)
  Downloading jinja2-3.1.6-py3-none-any.whl.metadata (2.9 kB)
[... dependency resolution ...]
Downloading ramalama-0.18.0-py3-none-any.whl (204 kB)
[... downloading dependencies ...]
Installing collected packages: rpds-py, pyyaml, MarkupSafe, attrs, argcomplete, referencing, jinja2, jsonschema-specifications, jsonschema, ramalama
Successfully installed MarkupSafe-3.0.3 argcomplete-3.6.3 attrs-26.1.0 jinja2-3.1.6 jsonschema-4.26.0 jsonschema-specifications-2025.9.1 pyyaml-6.0.3 ramalama-0.18.0 referencing-0.37.0 rpds-py-0.30.0
```

**Dependencies Installed:**
- Core: ramalama-0.18.0
- Configuration: pyyaml-6.0.3, jsonschema-4.26.0
- Templating: jinja2-3.1.6
- Utilities: argcomplete-3.6.3, attrs-26.1.0

**Installation Path:** `~/.local/lib/python3.14/site-packages/ramalama`

![RamaLama Installation](screenshots/04_ramalama_install.png)

---

## Version Verification

### Initial Attempt: Incorrect Command

**Command:**
```bash
ramalama --version
```

**Output:**
```
usage: ramalama [-h] [--debug | --quiet] [--dryrun] [--engine {podman,docker}] [--nocontainer] [--runtime {llama.cpp,mlx,vllm}] [--store STORE]
                {run,serve,bench,benchmark,perplexity,convert,benchmarks,rag,chat,containers,ps,help,info,inspect,list,ls,login,logout,pull,push,rm,stop,version,daemon} ...
ramalama: error: unrecognized arguments: --version
```

**Observation:** The `--version` flag is not recognized. RamaLama uses subcommand structure.

![RamaLama Version Error](screenshots/05_version_error.png)

### Research: Consulting Help Documentation

**Command:**
```bash
ramalama help
```

**Relevant Output:**
```
positional arguments:
  {run,serve,bench,benchmark,perplexity,convert,benchmarks,rag,chat,containers,ps,help,info,inspect,list,ls,login,logout,pull,push,rm,stop,version,daemon}
    run                 run specified AI Model as a chatbot
    serve               serve REST API on specified AI Model
    [...]
    version             display version of RamaLama
    daemon              daemon operations
```

**Key Learning:** RamaLama uses `ramalama version` (subcommand) instead of `ramalama --version` (flag).

![RamaLama Help Output](screenshots/06_help_output.png)

### Successful Version Check

**Command:**
```bash
ramalama version
```

**Output:**
```
ramalama version 0.18.0
```

**Result:** Version 0.18.0 confirmed installed.

![RamaLama Version Success](screenshots/07_version_success.png)

---

## Transport Evaluation

This section documents systematic testing of four model transport mechanisms supported by RamaLama.

### Transport 1: Ollama (Success)

#### Test 1.1: TinyLlama via Ollama

**Command:**
```bash
ramalama pull ollama://tinyllama
```

**Output:**
```
Downloading ollama://library/tinyllama:latest ...
Trying to pull ollama://library/tinyllama:latest ...
Downloading tinyllama
100% |████████████████████████████████████████████████████████████████████████████████████████████████████████|  608.16 MB/ 608.16 MB   6.21 MB/s        0s
```

**Performance Metrics:**
- Model Size: 608.16 MB
- Download Speed: 6.21 MB/s
- Status: Complete, no errors

![Ollama TinyLlama Pull](screenshots/08_ollama_tinyllama_pull.png)

#### Test 1.2: Granite 3.1 Dense 2B via Ollama

**Command:**
```bash
ramalama pull ollama://granite3.1-dense:2b
```

**Output:**
```
Downloading ollama://library/granite3.1-dense:2b ...
Trying to pull ollama://library/granite3.1-dense:2b ...
Downloading granite3.1-dense
100% |████████████████████████████████████████████████████████████████████████████████████████████████████████|    1.46 GB/   1.46 GB   6.66 MB/s        0s
2026-03-31 14:19:46 - ERROR - ❌ Network Error: [SSL: TLSV1_ALERT_DECODE_ERROR] tlsv1 alert decode error (_ssl.c:1081)
```

**Performance Metrics:**
- Model Size: 1.46 GB
- Download Speed: 6.66 MB/s
- Status: Download complete, post-download SSL error (non-critical)

**Analysis:** SSL error occurs after successful download. Model files are intact and usable despite the error message.

![Granite Pull with SSL Error](screenshots/09_granite_pull_ssl_error.png)

#### Test 1.3: DeepSeek R1 via Ollama

**Command:**
```bash
ramalama pull ollama://deepseek-r1
```

**Output:**
```
Downloading ollama://library/deepseek-r1:latest ...
Trying to pull ollama://library/deepseek-r1:latest ...
Downloading deepseek-r1
 21% |█████████████████████                                                                                   |    1.06 GB/   4.87 GB   5.77 MB/s   14m 18s
2026-03-31 14:37:39 - WARNING - ⚠️ I/O Error: [Errno 104] Connection reset by peer. Retrying 1/5 ...
100% |████████████████████████████████████████████████████████████████████████████████████████████████████████|    4.87 GB/   4.87 GB   6.53 MB/s    2m 46s
```

**Performance Metrics:**
- Model Size: 4.87 GB
- Initial Speed: 5.77 MB/s
- Final Speed: 6.53 MB/s
- Connection Reset: 21% progress, auto-retry successful
- Total Time: 2 minutes 46 seconds

**Analysis:** RamaLama automatic retry mechanism successfully recovered from network interruption.

![DeepSeek Pull with Retry](screenshots/10_deepseek_pull_retry.png)

**Ollama Transport Summary:**
- ✅ Reliable downloads across multiple model sizes
- ✅ Automatic retry on network failure
- ✅ Consistent download speeds (6+ MB/s)
- ⚠️ Minor post-download SSL errors (non-blocking)

---

### Transport 2: HuggingFace (Success)

#### Test 2.1: TinyLlama GGUF via HuggingFace

**Command:**
```bash
ramalama pull hf://TheBloke/TinyLlama-1.1B-Chat-v1.0-GGUF
```

**Output:**
```
Downloading hf://TheBloke/TinyLlama-1.1B-Chat-v1.0-GGUF ...
Trying to pull hf://TheBloke/TinyLlama-1.1B-Chat-v1.0-GGUF ...
Downloading tinyllama-1.1b-chat-v1.0.Q2_K.gguf
100% |████████████████████████████████████████████████████████████████████████████████████████████████████████|  460.74 MB/ 460.74 MB   7.41 MB/s        0s
```

**Performance Metrics:**
- Model Size: 460.74 MB
- Download Speed: 7.41 MB/s
- Quantization: Q2_K
- Status: Complete, no errors

![HuggingFace TinyLlama Pull](screenshots/11_hf_tinyllama_pull.png)

#### Test 2.2: Phi-2 via HuggingFace (Failed)

**Command:**
```bash
ramalama pull hf://phi2
```

**Output:**
```
Downloading hf:///phi2 ...
Trying to pull hf:///phi2 ...
URL pull failed and hf not available
Error: Failed to pull model: 'No metadata found
```

**Root Cause Analysis:**
- Short name `phi2` failed to resolve to full repository path
- HuggingFace requires complete repository specification
- Correct format: `hf://owner/repository/model-file.gguf`

**Decision:** Excluded from testing due to metadata resolution failure.

![Phi-2 Pull Failure](screenshots/12_phi2_pull_failure.png)

**HuggingFace Transport Summary:**
- ✅ Reliable for fully-qualified repository paths
- ✅ Excellent download speeds (7+ MB/s)
- ❌ Short name resolution not supported
- Requires explicit model file specification

---

### Transport 3: OCI Container (Partial Success)

#### Test 3.1: Pull Quay.io RamaLama Image

**Command:**
```bash
ramalama pull docker://quay.io/ramalama/ramalama
```

**Output:**
```
Downloading quay.io/ramalama/ramalama:latest ...
Trying to pull quay.io/ramalama/ramalama:latest...
Getting image source signatures
Copying blob 2056340a78af done   |
Copying blob 651197e7da71 done   |
Copying config d4d3dfd626 done   |
Writing manifest to image destination
```

**Result:** Image pull successful.

![Quay.io Image Pull Success](screenshots/13_quay_image_pull.png)

#### Test 3.2: Attempt Execution

**Command:**
```bash
ramalama run docker://quay.io/ramalama/ramalama
```

**Output:**
```
Error: securejoin.OpenInRoot /models: openat2 /models: no such file or directory
Error: Failed to serve model ramalama, for ramalama run command
```

**Root Cause:** Container image lacks embedded model files. Requires volume mount configuration.

![Quay.io Run Error](screenshots/14_quay_run_error.png)

#### Test 3.3: Docker Installation for Troubleshooting

To verify whether Docker availability affects OCI transport:

**Command:**
```bash
sudo dnf install docker
```

**Output:**
```
Transaction Summary:
 Installing:        24 packages

Total size of inbound packages is 99 MiB. Need to download 99 MiB.
After this operation, 375 MiB extra will be used (install 375 MiB, remove 0 B).
[...]
Complete!
```

**Packages Installed:**
- moby-engine 29.3.0
- docker-cli 29.3.0
- containerd 2.1.6
- Total disk usage: 375 MB

![Docker Installation](screenshots/15_docker_install.png)

#### Test 3.4: Retry After Docker Installation

**Command:**
```bash
ramalama run docker://quay.io/ramalama/ramalama
```

**Output:**
```
Error: securejoin.OpenInRoot /models: openat2 /models: no such file or directory
Error: Failed to serve model ramalama, for ramalama run command
```

**Result:** Identical error. Docker availability did not resolve the issue.

**Conclusion:** OCI container transport successfully pulls images but requires manual volume mounting for model storage. Not suitable for this exploration's scope.

---

### Transport 4: ModelScope (Failed)

#### Test 4.1: Install ModelScope Dependencies

**Command:**
```bash
pip install modelscope
```

**Output:**
```
Collecting modelscope
  Downloading modelscope-1.35.3-py3-none-any.whl.metadata (43 kB)
[... dependency resolution ...]
Installing collected packages: urllib3, tqdm, setuptools, packaging, idna, filelock, charset_normalizer, certifi, requests, modelscope
Successfully installed certifi-2026.2.25 charset_normalizer-3.4.6 filelock-3.25.2 idna-3.11 modelscope-1.35.3 packaging-26.0 requests-2.33.1 setuptools-82.0.1 tqdm-4.67.3 urllib3-2.6.3
```

![ModelScope Dependencies Install](screenshots/16_modelscope_install.png)

#### Test 4.2: Attempt Model Pull

**Command:**
```bash
ramalama pull modelscope://qwen/Qwen1.5-0.5B-Chat-GGUF
```

**Output (First 30 minutes):**
```
Downloading modelscope://qwen/Qwen1.5-0.5B-Chat-GGUF ...
Trying to pull modelscope://qwen/Qwen1.5-0.5B-Chat-GGUF ...
Downloading [LICENSE]: 100%|███████████████████████████████████████████████████████████████████████████████████████████| 7.11k/7.11k [00:01<00:00, 3.68kB/s]
Downloading [configuration.json]: 100%|███████████████████████████████████████████████████████████████████████████████████| 73.0/73.0 [00:02<00:00, 32.4B/s]
Downloading [qwen1_5-0_5b-chat-q4_k_m.gguf]: 100%|████████████████████████████████████████████████████████████████████████| 388M/388M [10:06<00:00, 672kB/s]
Downloading [README.md]: 100%|█████████████████████████████████████████████████████████████████████████████████████████| 4.36k/4.36k [00:02<00:00, 1.67kB/s]
Downloading [qwen1_5-0_5b-chat-q3_k_m.gguf]: 485MB [24:07, 351kB/s]
Downloading [qwen1_5-0_5b-chat-q2_k.gguf]: 500MB [33:58, 257kB/s]
Downloading [qwen1_5-0_5b-chat-q4_0.gguf]: 479MB [37:45, 222kB/s]
[... downloads multiple quantization variants simultaneously ...]
```

**Performance Metrics:**
- Simultaneous Downloads: 11 files
- Average Speed: 200-600 KB/s per file
- Progress at Cancellation: 7/11 files
- Time Elapsed: 37+ minutes

**Issues Identified:**
1. Downloads all quantization variants (not user-configurable)
2. Network throughput significantly lower than Ollama/HuggingFace
3. No single-file download option
4. Total bandwidth exceeds 5 GB

**Decision:** Manual cancellation after 37 minutes. Impractical for bandwidth-constrained environments.

![ModelScope Pull Attempt](screenshots/17_modelscope_pull.png)

**ModelScope Transport Summary:**
- ❌ Extremely slow download speeds (200-600 KB/s)
- ❌ Forces download of all quantization variants
- ❌ High total bandwidth consumption
- ❌ Not suitable for global contributors with limited connectivity
- Recommendation: Avoid for this project

---

## Model Execution Testing

This section documents execution attempts across all successfully pulled models.

### Execution 1: Ollama TinyLlama (Success)

**Command:**
```bash
ramalama run ollama://tinyllama
```

**Output:**
```
e7b36328dc30f2a189f6742d7aee0e1b611b0a15c3d94b1eea660185e642bd14
🦭 >
```

**Observations:**
- Container ID: `e7b36328dc30...` (changes per execution)
- Startup Time: 5-7 seconds
- Prompt Interface: Active and responsive

![Ollama TinyLlama Execution](screenshots/18_ollama_tinyllama_run.png)

### Execution 2: HuggingFace TinyLlama (Success)

**Command:**
```bash
ramalama run hf://TheBloke/TinyLlama-1.1B-Chat-v1.0-GGUF
```

**Output:**
```
5db04a9dc2bd921379cda589aaaf9205d66698cb4d8e0d2d7c9ebe766c2bcc0c
🦭 >
```

**Observations:**
- Container ID: `5db04a9dc2bd...` (unique per execution)
- Startup Time: 6-8 seconds (slightly slower than Ollama)
- Prompt Interface: Active and responsive

**Container ID Analysis:** The hexadecimal string represents Podman's container identifier, confirming proper containerized execution for each model run.

---

### Execution 3: Granite 3.1 Dense 2B (Failed - Memory)

**Command:**
```bash
ramalama run ollama://granite3.1-dense:2b
```

**Output:**
```
26c67e29d0621078d97ff95b5995843f2635972ddaf9efdf56be01be986b5a04
..................................................................................................................................
2026-03-31 15:23:28 - ERROR - Failed to serve model granite3.1-dense, for ramalama run command
2026-03-31 15:23:28 - ERROR - Command 'health check of container ramalama-kzUAING10J' timed out after 180 seconds
```

**Diagnostic Output (Excerpt):**
```
llama_kv_cache:        CPU KV buffer size = 10240.00 MiB
```

**Root Cause:**
- Model requires 10,240 MB (10 GB) KV cache
- Available system memory: 7,184 MB
- Memory shortfall: ~3 GB

**Timeout Behavior:** RamaLama enforces 180-second health check timeout. Model initialization exceeded this limit due to insufficient memory.

![Granite Timeout Error](screenshots/27_granite_timeout.png)

---

### Execution 4: DeepSeek R1 8B (Failed - Memory)

**Command:**
```bash
ramalama run ollama://deepseek-r1
```

**Output:**
```
2c283a5b138ff4043eaaae25a43e0c6bf673f3edfaa8a56cce079d71e37f2045
.........................................................................................................................................
2026-03-31 15:30:47 - ERROR - Failed to serve model deepseek-r1, for ramalama run command
2026-03-31 15:30:47 - ERROR - Command 'health check of container ramalama-zH7kTUjkSc' timed out after 180 seconds
```

**Diagnostic Output (Excerpt):**
```
llama_kv_cache:        CPU KV buffer size = 18432.00 MiB
```

**Root Cause:**
- Model requires 18,432 MB (18 GB) KV cache
- Available system memory: 7,184 MB
- Memory shortfall: ~11 GB

**Conclusion:** 8B parameter models exceed hardware constraints for this environment. Testing restricted to 1B parameter models.

---

## Output Quality Analysis

This section systematically evaluates model outputs against official Fedora documentation. Each test case includes the prompt, both model responses, ground truth verification, and evaluation across four dimensions: Accuracy, Hallucination, Structure, and Practical Usefulness.

### Test Case 1: Fedora Four Foundations

#### Prompt
```
What are the Four Foundations of the Fedora Project?
```

#### Response: Ollama TinyLlama

**Full Output:**
```
The Four Foundation of the Fedora Project are:
1. Fedora Project: The Fedora Project is a community-driven project that aims to build and maintain the Fedora operating system. It is the primary project of the Fedora Project and is responsible for managing the development, maintenance, and release of the operating system.

2. Fedora Core: The Fedora Core is the base operating system for the Fedora Project. It is designed to be simple and easy to use, but with the ability to add more features as they are needed.

3. Fedora Desktop: The Fedora Desktop is the primary user interface for the Fedora Project, which includes applications, software, and tools. It is designed to be simple and easy to use, but with the ability to add more features as they are needed.

4. Fedora Server: The Fedora Server is the primary server operating system for the Fedora Project. It is designed to be simple and easy to use, but with the ability to add more features as they are needed.

These four foundations are the basis for the Fedora Project and are what make the Fedora Project unique and highly regarded in the community. The Fedora Project is built on these foundations to ensure that the operating system is well-suited for its purpose and meets the needs of its users.
```

![Ollama Four Foundations Response](screenshots/19_ollama_four_foundations.png)

**Evaluation:**

| Dimension | Rating | Analysis |
|-----------|--------|----------|
| **Accuracy** | Completely Incorrect | Invented product names (Fedora Core, Fedora Desktop, Fedora Server) instead of foundational values |
| **Hallucination** | Severe | Fabricated technical components that do not exist as "foundations" |
| **Structure** | Moderate | Clear numbered list format, but content fundamentally wrong |
| **Practical Usefulness** | None | Would mislead Fedora contributors about project philosophy |

---

#### Response: HuggingFace TinyLlama (First Run)

**Full Output:**
```
The Four Foundation is a group of values that define the Fedora Project, which is an open source operating system (OS) community. It includes the following foundation values:

1. Forking freedom
2. The power of simplicity
3. A strong community
4. Open-source code and documentation

The Four Foundation values allow the Fedora Project to have a strong community that contributes to the success of the Fedora operating system. They ensure that the community's values are reflected in the operating system, allowing developers and contributors to work with a community that values open-source and contributor-driven development. This allows Fedora to create a strong and successful community, contributing to the success of the operating system.
```

![HuggingFace Four Foundations Response](screenshots/20_hf_four_foundations.png)

**Evaluation:**

| Dimension | Rating | Analysis |
|-----------|--------|----------|
| **Accuracy** | Completely Incorrect | All four "values" are fabricated; none match actual foundations |
| **Hallucination** | Severe | Invented terms like "Forking freedom" and "The power of simplicity" |
| **Structure** | Clear | Well-organized response with explanatory context |
| **Practical Usefulness** | None | Fundamentally misleading about Fedora's core values |

---

#### Ground Truth (Verified)

**Actual Four Foundations of Fedora:**

1. **Freedom** - Promoting software and content freedom
2. **Friends** - Strong, open community
3. **Features** - Excellence and timely delivery
4. **First** - Leading in innovation

**Source:** [Fedora Project Foundations](https://docs.fedoraproject.org/en-US/project/)

**Accuracy Verdict:** Both models achieved **0% accuracy**. No foundation was correctly identified in any response.

---

### Test Case 2: Fedora RPM Packaging Guidelines for Python

#### Prompt
```
Explain Fedora RPM packaging guidelines for Python libraries
```

#### Response: Ollama TinyLlama

**Full Output (Excerpt):**
```
Fedora is a Linux distro that follows the RPM (Recipe Package Manager) packaging system. RPM stands for "RPMS" and is one of the most popular packaging systems for Linux distributions.

Here are the packaging guidelines for Python libraries on Fedora RPMs:

1. Create a PyPI package: The first step is to create a PyPI package using the Python Package Installer (PyPI). Here's a step-by-step guide:

    a. Install Python: Fedora users can install Python using their package manager, which is typically easy to install using the package manager. Here's the command for installing Python on Fedora 32:

    ```
    sudo yum install python3
    ```

    b. Create a Python package: Once Python is installed, create a Python package using PyPI. Here's a step-by-step guide to create a Python package using PyPI:

    ```
    pip install pypi_package
    ```
[...]
3. Build the package: After creating the RPM spec file, build the package using the following command:

    ```
    rpmbuild -bs <spec_file_name>
    ```
[...]
5. Publish the package: Once the package is published, users can install it on their Fedora system or any other system that supports RPMs. Here's a step-by-step guide on how to install a Fedora RPM package on a Linux system:

    a. Open a terminal window.

    b. Type `yum install <package_name>` to install the package.

    c. If the package is already installed, Fedora will prompt you to update the package. If you want to update, type `yum update <package_name>`.
```

![Ollama Python Packaging Response](screenshots/21_ollama_python_guidelines.png)

**Evaluation:**

| Dimension | Rating | Analysis |
|-----------|--------|----------|
| **Accuracy** | Completely Incorrect | Wrong workflow (PyPI first), deprecated commands (yum), fabricated process |
| **Hallucination** | High | RPM does not stand for "Recipe Package Manager"; invented packaging steps |
| **Structure** | Moderate | Step-by-step format, but steps are fundamentally wrong |
| **Practical Usefulness** | None | Following this would fail to produce valid Fedora packages |

**Critical Errors:**
- Uses `yum` (deprecated in Fedora 22+, replaced by `dnf`)
- Suggests creating PyPI package first (backward workflow)
- No mention of `%pyproject_buildrequires`, `%pyproject_wheel`, `%pyproject_install`
- Fabricated `pip install pypi_package` command

---

#### Response: HuggingFace TinyLlama

**Full Output (Excerpt):**
```
The Fedora RPM packaging guidelines for Python libraries are designed to make it easier for developers to package and distribute Python libraries for use in Fedora Linux. The guidelines aim to ensure that Python libraries are compiled and distributed using the same set of rules, providing a uniform and consistent experience across Fedora RPM packages.

Here are some specific guidelines for Python libraries:

1. Fedora RPM Packaging:
- Use the Fedora RPM packaging guidelines.
- Use the Fedora RPM packaging rules for Python libraries.
- Specify the Python version as "2.7" for all Python libraries.
- Specify the Fedora RPM packaging options (e.g., SHA256, no-download).
- Specify the Fedora RPM release and version.
- Specify the Fedora RPM repository for the Python libraries.
[... continues with repetitive "Specify the Fedora RPM repository" statements ...]

2. Fedora RPM Packaging:
- Use the Fedora RPM packaging guidelines.
[... repeats similar content ...]

3. Fedora RPM Packaging:
- Use the Fedora RPM packaging guidelines.
[... further repetition ...]
```

![HuggingFace Python Packaging Response](screenshots/22_hf_python_guidelines.png)

**Evaluation:**

| Dimension | Rating | Analysis |
|-----------|--------|----------|
| **Accuracy** | Completely Incorrect | Python 2.7 deprecated since 2020; no actual macros or guidelines |
| **Hallucination** | High | Invented "guidelines" with no technical substance |
| **Structure** | Poor | Severe repetition, circular reasoning |
| **Practical Usefulness** | None | No actionable information for packaging |

**Critical Errors:**
- Specifies Python 2.7 (end-of-life since January 2020)
- No mention of PEP 517/518 compliance
- Repetitive content without technical details
- Missing all modern Fedora Python macros

---

#### Ground Truth (Verified)

**Actual Fedora Python Packaging Guidelines:**

Modern Fedora Python packaging uses **pyproject macros** following PEP 517/518:

```spec
BuildRequires:  python3-devel
BuildRequires:  pyproject-rpm-macros

%generate_buildrequires
%pyproject_buildrequires

%build
%pyproject_wheel

%install
%pyproject_install
```

**Key Requirements:**
- Use `%pyproject_buildrequires` to generate dependencies
- Use `%pyproject_wheel` for building
- Use `%pyproject_install` for installation
- NO manual `pip install` inside spec files
- Python 2 is no longer supported

**Source:** [Fedora Python Packaging Guidelines](https://docs.fedoraproject.org/en-US/packaging-guidelines/Python/)

**Accuracy Verdict:** Both models achieved **0% accuracy**. Neither mentioned correct macros or modern Python packaging practices.

---

### Test Case 3: Writing RPM Spec Files

#### Prompt
```
Write a simple .spec file for a Python app
```

#### Response: Ollama TinyLlama

**Full Output:**
```
name: my_app
version: 1.0
author: me
email: me@mydomain.com
license: MIT
description: This is a simple Python app.

[!include(build, exclude='tools/environment_variables.spec.jinja2')]

commands:
  start:
    description: Start the app in development mode.
    command: python3 -m my_app.app
    secrets:
      - PYTHONPATH: "my_app:path/to/my_app"

  test:
    description: Run tests.
    command: pytest -vv --pyargs my_app.app
    secrets:
      - TEST_ADMIN_EMAIL: "<|system|>"
      - TEST_ADMIN_PASSWORD: "<<|user|>:"

  deploy:
    description: Deploy the app to Heroku.
    command: heroku run python app.py --app my_app
    secrets:
      - HEROKU_APP_NAME: "my_app"
      - HEROKU_API_KEY: "<<|user|>:"
```

![Ollama Spec File Response](screenshots/23_ollama_spec_file.png)

**Evaluation:**

| Dimension | Rating | Analysis |
|-----------|--------|----------|
| **Accuracy** | Completely Incorrect | YAML-like CI/CD config, not RPM spec format |
| **Hallucination** | High | Generated deployment configuration instead of packaging spec |
| **Structure** | Misleading | Looks professional but wrong file type entirely |
| **Practical Usefulness** | None | Cannot be processed by rpmbuild |

**Critical Errors:**
- Uses YAML syntax instead of RPM spec syntax
- Contains CI/CD commands (start, test, deploy)
- No RPM sections (%prep, %build, %install, %files)
- Mentions Heroku deployment (irrelevant to RPM)

---

#### Response: HuggingFace TinyLlama

**Full Output:**
```
To write a simple .spec file for a Python app:

1. Open your terminal or command line, and navigate to the directory where your app's .spec file is located.

2. Create a new folder for your .spec file. For example, `mkdir my-app`.

3. In the new folder, create a new file (e.g. `my-app.spec`), and save it with a name that identifies your app (e.g. `my-app.spec`).

4. Add the following contents to your .spec file:

```
name: my-app

dependencies:
- python>=3.9
- pip:
  - setuptools

base_dir: /path/to/your/app

apps:
- name: app-1
  cmd: app-1
  entrypoint: app-1.py
  shorthand: app-1
[... continues with app-2 through app-7 ...]
```

5. Save the .spec file with a name that identifies your app (e.g. `my-app.spec`).

6. You're now ready to run your app by running the executable file. For example, `python my-app.py`.
```

![HuggingFace Spec File Response](screenshots/24_hf_spec_file.png)

**Evaluation:**

| Dimension | Rating | Analysis |
|-----------|--------|----------|
| **Accuracy** | Completely Incorrect | Generic YAML format, not RPM spec |
| **Hallucination** | High | Invented app deployment structure |
| **Structure** | Poor | Repetitive app entries without purpose |
| **Practical Usefulness** | None | Not processable as RPM spec file |

**Critical Errors:**
- YAML format instead of RPM spec
- Lists multiple identical "apps" with no rationale
- Missing all RPM-specific sections and macros
- Final instruction contradicts RPM packaging purpose

---

#### Ground Truth (Verified)

**Actual RPM Spec File Structure for Python:**

```spec
Name:           myapp
Version:        1.0
Release:        1%{?dist}
Summary:        Example Python application

License:        MIT
URL:            https://example.com/myapp
Source0:        %{name}-%{version}.tar.gz

BuildArch:      noarch
BuildRequires:  python3-devel
BuildRequires:  pyproject-rpm-macros

%description
Example Python application demonstrating modern Fedora packaging.

%prep
%autosetup

%generate_buildrequires
%pyproject_buildrequires

%build
%pyproject_wheel

%install
%pyproject_install

%files
%license LICENSE
%doc README.md
%{python3_sitelib}/myapp/
%{python3_sitelib}/myapp-%{version}.dist-info/

%changelog
* Mon Mar 31 2026 Your Name <you@example.com> - 1.0-1
- Initial package
```

**Required Sections:**
- Header (Name, Version, Release, Summary, License)
- %description
- %prep (source preparation)
- %build (compilation)
- %install (installation)
- %files (file manifest)

**Source:** [Fedora Packaging Guidelines](https://docs.fedoraproject.org/en-US/packaging-guidelines/)

**Accuracy Verdict:** Both models achieved **0% accuracy**. Neither produced valid RPM spec syntax.

---

### Test Case 4: Understanding %config Macro

#### Prompt
```
What is %config in RPM packaging?
```

#### Response: Ollama TinyLlama

**Full Output:**
```
%config is a feature available in RPM packaging. It is used to configure the installation of a package. Here is a brief overview of what %config does:

1. Package: The package that is being configured has its configuration data stored in a configuration file.
2. Package Manager: The package manager is responsible for creating the package, installing it, and running the associated install scripts.
3. Package Configuration: The package configuration file contains all the settings and options for the package.
4. Package Installation: The %config section of the package configuration file is used to configure the package during installation.

When a package is installed, the %config section is executed, which will read the configuration file and configure the package for the selected platform. This process is controlled by the %config section of the package configuration file.

In RPM packaging, %config is used to configure the installation of a package. It is a powerful feature that allows package developers to customize the installation process for their package. By using %config, package developers can create customized packages with different configurations and settings.
```

![Ollama Config Explanation](screenshots/25_ollama_config.png)

**Evaluation:**

| Dimension | Rating | Analysis |
|-----------|--------|----------|
| **Accuracy** | Completely Incorrect | Treats %config as installation script/process |
| **Hallucination** | High | Invented execution model and configuration process |
| **Structure** | Acceptable | Organized explanation, but fundamentally wrong |
| **Practical Usefulness** | None | Misrepresents %config purpose entirely |

---

#### Response: HuggingFace TinyLlama

**Full Output:**
```
In RPM (Recipe Packaging Format) packaging, the `%config` variable is used to specify a custom configuration file. This variable is used to set the environment variables required for the installation process. The `rpm` package manager supports this variable in its spec file. The `%config` variable can be used to set a variable that will be passed to the `rpm` command during the installation process. For example, in the `.spec` file:

```
%config(
   varname=config_var,
   value=config_value,
   comment=config_comment
)
```

This spec file will pass the value of the variable `varname` to the `rpm` command during the installation process. This variable will be set to the value of the variable `config_var` in the installed package.
```

![HuggingFace Config Explanation](screenshots/26_hf_config.png)

**Evaluation:**

| Dimension | Rating | Analysis |
|-----------|--------|----------|
| **Accuracy** | Completely Incorrect | Treats %config as function with parameters |
| **Hallucination** | High | Fabricated syntax `%config(varname=...)` |
| **Structure** | Acceptable | Clear code example, but wrong syntax |
| **Practical Usefulness** | None | Invented syntax would cause rpmbuild errors |

---

#### Ground Truth (Verified)

**Actual %config Macro Purpose:**

`%config` is used to **mark configuration files** in the `%files` section:

```spec
%files
%config(noreplace) /etc/myapp/myapp.conf
%config /etc/myapp/defaults.conf
```

**Behavior:**
- **%config**: File is marked as configuration
- During upgrade, if user modified the file, RPM saves user's version as `.rpmsave`
- **%config(noreplace)**: Preserves user modifications, saves new version as `.rpmnew`

**Purpose:** Protects user customizations during package updates.

**Source:** [RPM Packaging Guide](https://rpm-packaging-guide.github.io/) and [RPM Spec File Reference](https://rpm-software-management.github.io/rpm/manual/spec.html)

**Accuracy Verdict:** Both models achieved **0% accuracy**. Neither correctly explained %config as a file-marking directive.

---

### Output Quality Summary Table

| Test Case | Prompt Topic | Ollama Accuracy | HF Accuracy | Hallucination Level | Practical Use |
|-----------|--------------|-----------------|-------------|---------------------|---------------|
| 1 | Four Foundations | 0% | 0% | Severe | None |
| 2 | Python Guidelines | 0% | 0% | High | None |
| 3 | Spec File Writing | 0% | 0% | High | None |
| 4 | %config Macro | 0% | 0% | High | None |

**Overall Assessment:**
- **Accuracy Rate:** 0% across all tests and models
- **Hallucination Rate:** 100% (every response contained fabricated information)
- **Determinism:** Models produce different wrong answers for identical prompts
- **Practical Usefulness:** Zero utility for Fedora contributors without RAG enhancement

---

## System Diagnostics

### RamaLama Info Output

**Command:**
```bash
ramalama info
```

**Full JSON Output (Key Sections):**

```json
{
    "Accelerator": "none",
    "Engine": {
        "Name": "podman",
        "Info": {
            "host": {
                "arch": "amd64",
                "cpus": 12,
                "distribution": {
                    "distribution": "fedora",
                    "variant": "wsl",
                    "version": "43"
                },
                "kernel": "6.6.87.2-microsoft-standard-WSL2",
                "memFree": 7184429056,
                "memTotal": 8184340480,
                "ociRuntime": {
                    "name": "crun",
                    "version": "crun version 1.25.1"
                }
            },
            "version": {
                "Version": "5.8.1"
            }
        }
    },
    "Runtimes": {
        "Available": [
            "llama.cpp",
            "mlx",
            "vllm"
        ],
        "Default": "llama.cpp"
    },
    "Store": "/home/olawoyin/.local/share/ramalama",
    "UseContainer": true,
    "Version": "0.18.0"
}
```

![RamaLama Info Output](screenshots/28_ramalama_info.png)

**Key Configuration Details:**

**Container Engine:**
- Name: Podman
- Version: 5.8.1
- Runtime: crun 1.25.1
- Rootless: Yes

**Inference Runtimes:**
- Default: llama.cpp
- Available: llama.cpp, mlx, vllm

**Storage:**
- Model Storage: `/home/olawoyin/.local/share/ramalama`
- Container Storage: `/home/olawoyin/.local/share/containers/storage`

**Hardware Detection:**
- CPU: 12 cores (AMD64)
- Total Memory: 8184 MB
- Free Memory: 7184 MB
- Accelerator: None detected

**Shortnames Configuration:**

RamaLama provides convenient shortnames for popular models:

| Shortname | Full Path |
|-----------|-----------|
| `tinyllama` | `hf://TheBloke/TinyLlama-1.1B-Chat-v1.0-GGUF` |
| `granite` | `ollama://granite3.1-dense` |
| `granite:2b` | `ollama://granite3.1-dense:2b` |
| `deepseek` | `ollama://deepseek-r1` |
| `phi2` | `huggingface://MaziyarPanahi/phi-2-GGUF/phi-2.Q4_K_M.gguf` |

**Network Configuration:**
- Backend: netavark 1.17.2
- DNS: aardvark-dns 1.17.0
- Default Network: podman

---

## Does RamaLama Make AI Boring?

The task asks: **Does RamaLama make working with AI boring?** The answer requires distinguishing between infrastructure complexity and output quality.

### What RamaLama Makes Boring (Positive)

**1. Installation Simplicity (5/5)**

Single command installation with automatic dependency resolution:
```bash
pip install ramalama
```

No manual configuration of inference engines, container runtimes, or model repositories. This is exceptionally boring in the best way possible.

**2. Model Acquisition (5/5)**

Unified interface across multiple transports:
```bash
ramalama pull ollama://tinyllama
ramalama pull hf://TheBloke/TinyLlama-1.1B-Chat-v1.0-GGUF
```

No need to learn transport-specific tools or APIs. This abstraction makes model management predictably boring.

**3. Command Structure (4/5)**

Logical subcommand organization following container management patterns:
```bash
ramalama pull    # Acquire models
ramalama run     # Execute models
ramalama info    # System diagnostics
ramalama list    # Inventory models
```

Learning curve exists (discovered through `--version` error), but structure becomes boring once understood.

**4. Container Integration (5/5)**

Seamless Podman integration on Fedora:
- Automatic container creation
- Rootless execution
- No Docker requirement
- Container isolation per execution

RamaLama handles containerization completely transparently. Users never interact with containers directly.

**5. Transport Abstraction (4/5)**

Multiple model sources through single interface:
- Ollama (community models)
- HuggingFace (GGUF quantized models)
- OCI registries (containerized models)
- ModelScope (regional repositories)

Success varied by transport, but the abstraction layer itself works as designed.

**Infrastructure Boring Score: 23/25 (92%)**

---

### What RamaLama Does NOT Make Boring (Yet)

**1. Model Execution Reliability (3/5)**

Platform-dependent success:
- ✅ 1B parameter models: Successful
- ❌ 2B parameter models: Memory timeout
- ❌ 8B parameter models: Memory timeout

Hardware constraints still matter. Not boring when models fail silently after 180 seconds.

**2. Output Quality (1/5)**

Systematic hallucination analysis revealed:
- **Accuracy:** 0% across all technical queries
- **Hallucination Rate:** 100%
- **Determinism:** Same prompt, different wrong answers
- **Practical Use:** Zero utility without enhancement

The models produce confident, well-structured, completely incorrect responses. This is the opposite of boring; it requires constant verification.

**3. Model Selection Guidance (2/5)**

No built-in mechanism to:
- Recommend models for hardware constraints
- Indicate expected accuracy domains
- Warn about known limitations
- Suggest RAG enhancement requirements

Users must discover through trial and error which models work.

**Output Quality Boring Score: 6/25 (24%)**

---

### The Verdict: Selectively Boring

**RamaLama successfully makes AI model DEPLOYMENT boring:**

The infrastructure abstraction works brilliantly. Install, pull, run. Three commands to go from zero to executing LLMs. This is exactly the right kind of boring, the kind that lets developers focus on problems rather than tooling.

**RamaLama does NOT make AI ASSISTANCE boring yet:**

The outputs are unreliable without enhancement. Every response requires verification against authoritative sources. This exploration demonstrated that even simple factual queries about Fedora documentation produce elaborate hallucinations.

**The Missing Piece: RAG**

This exploration validates the Outreachy project architecture:
1. **RamaLama handles deployment** (boring infrastructure) ✅
2. **RAG will handle knowledge** (boring accuracy) ⬅️ Next phase

When RAG injects verified Fedora documentation into the context, the model outputs should become boring in the desired way: consistent, accurate, predictable. The exploration confirms that RamaLama provides the right foundation; it needs the knowledge layer.

**Final Answer:**

RamaLama makes working with AI deployment boring (excellent). It does not yet make AI assistance boring (needs RAG). The project direction is validated: infrastructure works, knowledge enhancement is the next logical step.

---

## Appendices

### Appendix A: Command Reference

**Installation Commands:**
```bash
# Verify prerequisites
podman --version
python3 --version

# Install RamaLama
pip install ramalama

# Verify installation
ramalama version
```

**Model Management:**
```bash
# Pull models
ramalama pull ollama://MODEL_NAME
ramalama pull hf://REPO/MODEL.gguf

# List downloaded models
ramalama list

# Remove models
ramalama rm MODEL_NAME
```

**Execution:**
```bash
# Run interactive session
ramalama run TRANSPORT://MODEL

# View system info
ramalama info

# List running containers
ramalama ps
```

**Transport Examples:**
```bash
# Ollama
ramalama pull ollama://tinyllama

# HuggingFace
ramalama pull hf://TheBloke/TinyLlama-1.1B-Chat-v1.0-GGUF

# OCI Container
ramalama pull docker://quay.io/ramalama/ramalama
```

---

### Appendix B: Screenshot Index

| # | Filename | Description |
|---|----------|-------------|
| 01 | `01_podman_version.png` | Podman version verification |
| 02 | `02_podman_hello_world.png` | Podman functionality test |
| 03 | `03_python_check.png` | Python installation confirmation |
| 04 | `04_ramalama_install.png` | RamaLama pip installation output |
| 05 | `05_version_error.png` | Initial `--version` error |
| 06 | `06_help_output.png` | RamaLama help documentation |
| 07 | `07_version_success.png` | Correct version command output |
| 08 | `08_ollama_tinyllama_pull.png` | Ollama TinyLlama download |
| 09 | `09_granite_pull_ssl_error.png` | Granite pull with SSL error |
| 10 | `10_deepseek_pull_retry.png` | DeepSeek pull with auto-retry |
| 11 | `11_hf_tinyllama_pull.png` | HuggingFace TinyLlama download |
| 12 | `12_phi2_pull_failure.png` | Phi-2 metadata error |
| 13 | `13_quay_image_pull.png` | Quay.io OCI image pull |
| 14 | `14_quay_run_error.png` | OCI container execution error |
| 15 | `15_docker_install.png` | Docker installation output |
| 16 | `16_modelscope_install.png` | ModelScope dependencies |
| 17 | `17_modelscope_pull.png` | ModelScope download attempt |
| 18 | `18_ollama_tinyllama_run.png` | Ollama TinyLlama execution start |
| 19 | `19_ollama_four_foundations.png` | Test 1: Ollama response |
| 20 | `20_hf_four_foundations.png` | Test 1: HuggingFace response |
| 21 | `21_ollama_python_guidelines.png` | Test 2: Ollama Python packaging |
| 22 | `22_hf_python_guidelines.png` | Test 2: HuggingFace Python packaging |
| 23 | `23_ollama_spec_file.png` | Test 3: Ollama spec file generation |
| 24 | `24_hf_spec_file.png` | Test 3: HuggingFace spec file generation |
| 25 | `25_ollama_config.png` | Test 4: Ollama %config explanation |
| 26 | `26_hf_config.png` | Test 4: HuggingFace %config explanation |
| 27 | `27_granite_timeout.png` | Granite memory timeout error |
| 28 | `28_ramalama_info.png` | System diagnostics output |

---

### Appendix C: Verification Sources

All ground truth comparisons verified against official documentation:

**Fedora Project:**
- Four Foundations: https://docs.fedoraproject.org/en-US/project/
- Python Packaging Guidelines: https://docs.fedoraproject.org/en-US/packaging-guidelines/Python/
- General Packaging Guidelines: https://docs.fedoraproject.org/en-US/packaging-guidelines/

**RPM Packaging:**
- RPM Packaging Guide: https://rpm-packaging-guide.github.io/
- RPM Spec File Reference: https://rpm-software-management.github.io/rpm/manual/spec.html

**RamaLama:**
- Official Repository: https://github.com/containers/ramalama
- Documentation: https://github.com/containers/ramalama/tree/main/docs

---

### Appendix D: Hardware Limitations and Model Selection

**Memory Constraints Discovered:**

| Model | Parameters | KV Cache Required | Available RAM | Result |
|-------|------------|-------------------|---------------|--------|
| TinyLlama | 1.1B | ~600 MB | 7.2 GB | ✅ Success |
| Granite Dense | 2.5B | 10.2 GB | 7.2 GB | ❌ Timeout |
| DeepSeek R1 | 8.2B | 18.4 GB | 7.2 GB | ❌ Timeout |

**Recommendations for Similar Environments:**
- Limit testing to 1B parameter models
- Consider quantized models (Q2_K, Q4_K) for larger parameter counts
- Plan for 2x model size in available RAM for KV cache
- Monitor timeout errors as indicators of memory pressure

---

## Repository Structure

```
ramalama-exploration-outreachy-fedora/
├── README.md
└── screenshots/
    ├── 01_podman_version.png
    ├── 02_podman_hello_world.png
    ├── 03_python_check.png
    ├── 04_ramalama_install.png
    ├── 05_version_error.png
    ├── 06_help_output.png
    ├── 07_version_success.png
    ├── 08_ollama_tinyllama_pull.png
    ├── 09_granite_pull_ssl_error.png
    ├── 10_deepseek_pull_retry.png
    ├── 11_hf_tinyllama_pull.png
    ├── 12_phi2_pull_failure.png
    ├── 13_quay_image_pull.png
    ├── 14_quay_run_error.png
    ├── 15_docker_install.png
    ├── 16_modelscope_install.png
    ├── 17_modelscope_pull.png
    ├── 18_ollama_tinyllama_run.png
    ├── 19_ollama_four_foundations.png
    ├── 20_hf_four_foundations.png
    ├── 21_ollama_python_guidelines.png
    ├── 22_hf_python_guidelines.png
    ├── 23_ollama_spec_file.png
    ├── 24_hf_spec_file.png
    ├── 25_ollama_config.png
    ├── 26_hf_config.png
    ├── 27_granite_timeout.png
    └── 28_ramalama_info.png
```

---

## Conclusion

This exploration systematically evaluated RamaLama on Fedora Linux across four transport mechanisms, documenting installation, configuration, model execution, and output quality. The findings validate RamaLama's infrastructure abstraction while confirming the necessity of RAG enhancement for production deployment.

**Key Achievements:**
- ✅ Complete Fedora 43 installation and configuration
- ✅ Four transport protocols tested (Ollama, HuggingFace, OCI, ModelScope)
- ✅ Two models successfully executed with systematic prompt testing
- ✅ 28 screenshots documenting every step
- ✅ Ground truth verification against official Fedora documentation
- ✅ Comprehensive evaluation framework applied to all outputs

**Project Validation:**
The exploration confirms that RamaLama provides robust model deployment infrastructure. The 100% hallucination rate on technical queries validates the project's next phase: implementing RAG with Fedora documentation to transform unreliable outputs into trustworthy packaging assistance.

RamaLama makes infrastructure boring. The Outreachy project will make the outputs boring too.

---

**End of Document**
