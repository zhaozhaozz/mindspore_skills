---
name: mindspore-install
description: >-
  Install MindSpore deep learning framework with conda environment management.
  Use this skill whenever the user wants to install MindSpore, create a MindSpore
  development environment, set up MindSpore on Ascend NPU, or manage multiple
  MindSpore versions with conda. Also trigger when user mentions "mindspore
  environment", "install mindspore", or wants to set up a training/inference
  environment on Ascend hardware.
---

# MindSpore Environment Installation

**IMPORTANT**: Communicate with the user in Chinese during execution, even though this skill is written in English.

This skill guides the installation of MindSpore using conda for environment management. MindSpore has specific Python version requirements and must be installed from designated pip sources. On Ascend hardware, CANN must be configured first.

Follow the numbered steps below **in order**. At each step that requires user input, stop and wait for the user's response before proceeding.

## Step 0: Gather Requirements

Ask the user (if not already specified):

1. **Which MindSpore version?** Default to the latest (2.8.0 as of writing). The user knows what version they need — respect their choice.

## Step 1: Check System Environment and Detect Hardware

Detect the system architecture and OS:

```bash
uname -m     # aarch64 or x86_64
uname -s     # Linux, Darwin
```

Check if conda is available:

```bash
conda --version
conda env list
```

If conda is not installed, tell the user and stop. Do NOT install conda yourself.

**Automatically detect Ascend hardware:**

```bash
npu-smi info 2>/dev/null
```

- If the command succeeds and shows NPU devices → **Ascend hardware detected**, proceed with Ascend installation (requires CANN in Step 3)
- If the command fails or shows no devices → **No Ascend hardware detected**, ask user: "未检测到昇腾硬件，是否安装 CPU 版本的 MindSpore？"
  - If user says yes, skip Step 3 (CANN setup)
  - If user says no, stop

**Determine conda environment name:**

Suggest a concise name without dots:
- Format: `ms{version_no_dot}` or `py{pyver}_ms{version_no_dot}`
- Examples: `ms280` for MindSpore 2.8.0, `ms272` for 2.7.2, or `py311_ms272` for Python 3.11 + MindSpore 2.7.2
- Ask user to confirm or provide custom name

## Step 2: Determine Python Version

MindSpore has strict Python version requirements that vary by release. The commonly recommended version is **Python 3.11**. Check `references/version-compatibility.md` for version-specific constraints.

General rules:
- MindSpore 2.7.0+ supports Python 3.9, 3.10, 3.11 (2.7.2+ also supports 3.12)
- MindSpore 2.3.x~2.6.x supports Python 3.8, 3.9, 3.10 (some support 3.11)
- Older versions may only support Python 3.7, 3.8, 3.9

When in doubt, default to Python 3.11 for recent versions (2.5.0+). For older versions, check the reference file.

## Step 3: Set Up CANN (Ascend Only)

If the user is on Ascend hardware, MindSpore requires CANN to be properly configured before it can use the NPU.

### Detect Available CANN Versions

Search for all CANN installations on the system:

```bash
find /usr/local/Ascend ${HOME}/Ascend /home/Ascend -name "set_env.sh" -path "*/cann/set_env.sh" 2>/dev/null
```

For each found set_env.sh, check the CANN version:

```bash
# For each path found, extract version from parent directory or version.cfg
# Example: /usr/local/Ascend/cann/set_env.sh -> check /usr/local/Ascend/ascend-toolkit/latest/version.cfg
# Example: /home/Ascend/8.5.0/cann/set_env.sh -> version is 8.5.0
```

### Present Options to User

List all detected CANN versions and tell the user:

1. **Available CANN versions** on this system
2. **Recommended CANN version** for the target MindSpore version (from `references/version-compatibility.md`)
3. **Compatibility note**: Adjacent versions usually work fine, possibly with warnings

Ask the user which CANN version to use. If only one version is found, confirm with the user before proceeding.

### Source Selected CANN Environment

Once the user selects a version, source its set_env.sh:

```bash
source <selected_cann_path>/set_env.sh
```

Verify CANN is loaded:

```bash
echo $ASCEND_HOME_PATH
```

### If No CANN Found

If no CANN installation is detected, tell the user that CANN needs to be installed first. If the `cann-install` skill is available, suggest using it.

## Step 4: Create Conda Environment

Create a new conda environment with the determined Python version:

```bash
conda create -n {env_name} python={python_version} -y
conda activate {env_name}
```

If an environment with the same name already exists, ask the user whether to:
1. Use the existing environment
2. Remove and recreate it
3. Choose a different name

## Step 5: Install MindSpore

**Make sure the target conda environment is activated before installing.** The pip install must run inside the conda env created in Step 4, not in the base environment.

### Network and Mirror Configuration

Before installing, check network connectivity and configure appropriate pip mirrors.

**Check if in corporate intranet:**

```bash
# Test Huawei internal mirror (only accessible from Huawei intranet)
curl -I --connect-timeout 3 https://mirrors.tools.huawei.com/pypi/simple 2>/dev/null | head -1
```

**Pip index strategy:**

- **For MindSpore itself**: Always use official MindSpore repo for best compatibility
  ```bash
  MINDSPORE_INDEX="-i https://repo.mindspore.cn/pypi/simple --trusted-host repo.mindspore.cn --extra-index-url https://repo.huaweicloud.com/repository/pypi/simple"
  ```

- **For other dependencies**: Use faster mirrors based on network environment
  - Huawei internal (if accessible): `-i https://mirrors.tools.huawei.com/pypi/simple`
  - Huawei Cloud public: `--trusted-host mirrors.huaweicloud.com -i https://mirrors.huaweicloud.com/repository/pypi/simple`
  - Default PyPI: `-i https://pypi.org/simple`

**IMPORTANT - CANN-provided packages:**

Do NOT attempt to install these packages via pip, they are provided by CANN environment:
- `te` (Tensor Engine)
- `tbe` (Tensor Boost Engine)
- Other CANN operator packages

These packages become available after sourcing CANN's `set_env.sh` in Step 3.

**If download fails or is very slow**, ask the user:
- "下载遇到问题，是否需要配置代理？请提供代理地址（格式：http://proxy_host:port 或 http://user:pass@proxy_host:port）"
- If user provides proxy:
  ```bash
  export http_proxy="http://proxy_host:port"
  export https_proxy="http://proxy_host:port"
  ```

### Method A: pip Install (Recent Versions)

Recent versions can be installed directly via pip from the official MindSpore repo:

```bash
conda activate {env_name}
pip install mindspore=={version} \
    -i https://repo.mindspore.cn/pypi/simple \
    --trusted-host repo.mindspore.cn \
    --extra-index-url https://repo.huaweicloud.com/repository/pypi/simple
```

**Try this method first.** If pip cannot find the specified version, fall back to Method B.

### Method B: whl Package Install (Historical Versions)

For older versions not available on the pip index, download the whl package directly.

The whl URL follows this pattern:
```
https://ms-release.obs.cn-north-4.myhuaweicloud.com/{version}/MindSpore/unified/{arch}/mindspore-{version}-cp{pyver}-cp{pyver}-linux_{arch_full}.whl
```

Where:
- `{version}` = MindSpore version (e.g., `2.5.0`)
- `{arch}` = `aarch64` or `x86_64`
- `{pyver}` = Python version without dot (e.g., `311` for Python 3.11)
- `{arch_full}` = `aarch64` or `x86_64`

Example for MindSpore 2.5.0 on x86_64 with Python 3.11:
```bash
pip install https://ms-release.obs.cn-north-4.myhuaweicloud.com/2.5.0/MindSpore/unified/x86_64/mindspore-2.5.0-cp311-cp311-linux_x86_64.whl
```

For CPU-only on non-Linux platforms, the URL uses `cpu` instead of `unified`:
- Windows: `https://ms-release.obs.cn-north-4.myhuaweicloud.com/{version}/MindSpore/cpu/x86_64/mindspore-{version}-cp{pyver}-cp{pyver}-win_amd64.whl`
- macOS aarch64: `https://ms-release.obs.cn-north-4.myhuaweicloud.com/{version}/MindSpore/cpu/aarch64/mindspore-{version}-cp{pyver}-cp{pyver}-macosx_11_0_arm64.whl`

The full release list with all versions and download links is at:
https://raw.atomgit.com/mindspore/docs/raw/master/resource/release/release_list_zh_cn.md

If the download fails, check network connectivity and suggest the user try with a proxy or download manually from the release list.

## Step 6: Verify Installation

### Basic Verification (All Platforms)

```bash
python -c "import mindspore; print(mindspore.__version__)"
```

### Ascend Verification

```bash
python -c "import mindspore; mindspore.set_device('Ascend'); mindspore.run_check()"
```

**Interpreting the output:**
- If it prints version info and passes the check — installation is successful
- If there's a CANN version warning — this is usually fine, most versions are compatible. Tell the user but don't treat it as an error
- If it raises an error about missing Ascend drivers or CANN — CANN environment is not properly set up, go back to Step 3

### CPU-Only Verification

If the user is not on Ascend hardware:

```bash
python -c "import mindspore; mindspore.set_device('CPU'); mindspore.run_check()"
```

## Step 7: Report Results

Tell the user:

1. The conda environment name and how to activate it: `conda activate {env_name}`
2. The installed MindSpore version
3. If on Ascend: remind them to source CANN's `set_env.sh` before using MindSpore in new shell sessions
4. Suggest adding both `conda activate` and `source set_env.sh` to their shell rc file for convenience

## Important Rules

- **Never change the user's requested MindSpore version** without explicit permission. Even if there's a CANN version mismatch, install what the user asked for and inform them of the compatibility situation.
- **Try pip install first**, fall back to whl only if pip can't find the version.
- **Do NOT modify the user's conda base environment.** Always create or use a dedicated environment.
- **Do NOT modify system-level CANN installations.** Only source the environment, never modify it.
- **If something fails, report the error clearly** and wait for user instructions. Don't guess at fixes.
- **Multiple environments are fine.** Users often maintain multiple MindSpore versions for different projects. Don't warn against this — it's expected workflow.

## Troubleshooting

### pip can't find the version
The MindSpore pip index only hosts recent versions. Try Method B (whl download).

### Import fails with "No module named 'mindspore'"
Check that the correct conda environment is activated. Check that pip installed to the right environment: `which pip` should point to the conda env.

### Ascend device not found
- Ensure CANN environment is sourced: `echo $ASCEND_HOME_PATH`
- Check if NPU is visible: `npu-smi info`
- The Ascend driver may not be loaded. This requires system admin intervention.

### CANN version warning
This is informational, not an error. Most MindSpore versions work with a range of CANN versions. See `references/version-compatibility.md` for recommended pairings.
