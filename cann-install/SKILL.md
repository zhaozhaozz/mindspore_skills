---
name: cann-install
description: Install Ascend CANN (Compute Architecture for Neural Networks) toolkit and operator packages on Linux systems. Use this skill when users explicitly request to install CANN environment, set up Ascend NPU development environment, or install CANN packages for Ascend 910A/910B/910C chips.
---

# Ascend CANN Installation

**IMPORTANT**: Communicate with the user in Chinese during execution, even though this skill is written in English.

This skill contains everything needed to install CANN. Do NOT search the web or guess download URLs. All packages are obtained by extracting .run binaries from the Ascend conda repository.

Follow the numbered steps below **in order**. Do not skip steps. At each step that requires user input, stop and wait for the user's response before proceeding.

## CANN Package Types

### Required (must install both)

| Package | Description | Varies by Chip |
|---------|-------------|---------------|
| **toolkit** | Main development toolkit with compiler, debugger, model converter | No (architecture only) |
| **ops** (>=8.5.0) / **kernels** (<8.5.0) | Operator package with chip-specific compute operators | Yes |

### Optional (present to user after determining version)

| Package | Description | Varies by Chip | Available Versions |
|---------|-------------|---------------|-------------------|
| **nnal** | Neural Network Acceleration Library with ATB engine for Transformer/LLM optimization | No | 8.1.RC1+ |
| **nnae** | Neural Network Accelerate Engine for training and high-performance inference | No | 8.1.RC1 ~ 8.2.RC1 |
| **nnrt** | Neural Network Runtime, lightweight for edge/device inference | No | 8.0.0 ~ 8.3.RC1 |

## Installation Workflow

### Step 0: Determine Installation Target

First clarify: is this a **local** or **remote** (SSH) installation?

- If the user provides SSH connection info (host, port, user), it's remote installation
- All detection commands (Steps 2-4) must run on the **target machine**
- If remote server has no internet, download locally and `scp` to the server

### Step 1: Confirm Target Version

If the user already specified a version (e.g. "install CANN 8.5.0"), confirm it. Otherwise ask.

Then query the conda repo to verify the version exists and list available packages:

```bash
curl -s https://repo.huaweicloud.com/ascend/repos/conda/{arch_dir}/ | grep -oP 'href="ascend-cann-[^"]+\.conda"' | sed 's/href="//;s/"//' | grep '{version}'
```

`{arch_dir}` = `linux-aarch64` for aarch64, `linux-64` for x86_64.

### Step 2: Detect System Architecture

Run on the **target machine**:

```bash
uname -m
```

### Step 3: Check Existing Installation and User Identity

Run on the **target machine**:

```bash
whoami
ls -l /usr/local/Ascend/ascend-toolkit/ 2>/dev/null
cat /usr/local/Ascend/ascend-toolkit/latest/version.cfg 2>/dev/null
ls -l ${HOME}/Ascend/ascend-toolkit/ 2>/dev/null
cat ${HOME}/Ascend/ascend-toolkit/latest/version.cfg 2>/dev/null
```

The `ls -l` output shows symlinks. The actual version is in `latest/version.cfg`. For example, if you see `8.3 -> 8.3.RC1`, the installed version is 8.3.RC1.

If a previous CANN version is found at the system path (`/usr/local/Ascend`), **tell the user** what version exists. Unless the user explicitly asks to upgrade the system installation, default to installing to a new path under the user's home directory. Do not overwrite the system-level installation - it may be shared or depended on by other users/services.

Use `whoami` to determine default install path:
- If no existing installation: root -> `/home/Ascend/{version}`, non-root -> `${HOME}/Ascend/{version}`
- If system path already has CANN: default to `/home/Ascend/{version}` for root or `${HOME}/Ascend/{version}` for non-root, unless user explicitly says to upgrade in place

**IMPORTANT**: The install path and all parent directories must have 755 permissions. Avoid paths under `/root` as it typically has 750 permissions and the installer will fail.

### Step 4: Identify Chip Type

Run on the **target machine**. This step is critical - do not skip it.

`npu-smi info` only shows "Ascend910" without distinguishing 910A/B/C. The reliable method is the PCI device ID:

```bash
lspci -n -D | grep -o '19e5:d[0-9a-f]\{3\}' | head -n1 | cut -d: -f2
```

**PCI Device ID -> Chip mapping:**

| Device ID | Chip | Operator Package Suffix |
|-----------|------|------------------------|
| `d500` | Ascend 310P | `310p` |
| `d801` | Ascend 910A | `910` |
| `d802` | Ascend 910B (A2) | `910b` |
| `d803` | Ascend 910C (A3) | `a3` |

Do NOT rely on OPP config directories (like `ascend910b` under toolkit) - those are multi-chip config files bundled with the toolkit, not indicators of the current hardware.

If `lspci` is unavailable, ask the user which chip type they have.

### Step 5: Present Installation Plan and Get Confirmation

Before downloading anything, present a clear summary to the user and **wait for confirmation**:

1. **Target**: local or remote (host info)
2. **Architecture**: aarch64 / x86_64
3. **Chip type**: detected result with chip name
4. **CANN version**: user-confirmed version
5. **Required packages**:
   - toolkit
   - ops/kernels (with chip-specific name)
6. **Available optional packages**: list any nnal/nnae/nnrt that exist for this version in the conda repo
7. **Installation path**: ask user, show default based on root/non-root
8. Ask: which optional packages to install (if any)?

Only proceed after the user confirms.

### Step 6: Download and Extract

**CRITICAL**: .conda files are ZIP archives containing tar.zst files. Follow these exact steps.

Download all packages in one command:

```bash
mkdir -p /tmp/cann_install && cd /tmp/cann_install
curl -O https://repo.huaweicloud.com/ascend/repos/conda/{arch_dir}/ascend-cann-toolkit-{version}-0.conda \
     -O https://repo.huaweicloud.com/ascend/repos/conda/{arch_dir}/ascend-cann-{chip}-ops-{version}-0.conda
# Add -O for each optional package if user selected them
```

Extract all .run files in one command block:

```bash
for pkg in *.conda; do
  unzip -o "$pkg" 'pkg-*.tar.zst'
done
for tarfile in pkg-*.tar.zst; do
  tar --use-compress-program=unzstd -xf "$tarfile" 'Ascend/*.run'
done
```

All .run files will be in `Ascend/` directory. See `references/binary-install.md` for package naming details.

**For remote installation**, use this network fallback strategy:

1. **First attempt**: Try downloading on the remote server (run the download commands via SSH)
2. **If download fails on remote**: Download and extract on the local machine, then transfer:
   ```bash
   # On local machine
   mkdir -p /tmp/cann_install && cd /tmp/cann_install
   curl -O https://repo.huaweicloud.com/ascend/repos/conda/{arch_dir}/ascend-cann-toolkit-{version}-0.conda
   # ... extract as shown above ...

   # Transfer to remote
   scp -P {port} ./Ascend/*.run {user}@{host}:/tmp/cann_install/Ascend/
   ```
3. **If local download also fails**: Inform the user that network access to the conda repo is blocked. Suggest:
   - Configure proxy settings (see Network Issues section)
   - Or manually download from https://www.hiascend.com/developer/download/community/result

### Step 7: Install Packages

Install packages **one at a time**. Before each install command, tell the user what you're about to install.

**Install toolkit first:**

Tell user: "Installing CANN toolkit {version} to {install_path}"

```bash
bash ./Ascend/Ascend-cann-toolkit_{version}_linux-{arch}.run --quiet --install --install-path={install_path}
```

**Then install ops:**

Tell user: "Installing {chip} operator package {version} to {install_path}"

```bash
bash ./Ascend/Ascend-cann-{chip}-ops_{version}_linux-{arch}.run --quiet --install --install-path={install_path}
```

**Then install optional packages (if selected):**

**IMPORTANT**: Before installing optional packages (nnal/nnae/nnrt), you MUST source the environment variables first. The toolkit installation output will print the exact path to `set_env.sh` (e.g. `source /home/Ascend/8.5.0/cann-8.5.0/set_env.sh`). Use that exact path:

```bash
source {exact_path_from_toolkit_install_output}
```

Then for each optional package, tell user what you're installing before running the command:

```bash
bash ./Ascend/Ascend-cann-nnal_{version}_linux-{arch}.run --quiet --install --install-path={install_path}
```

**If any install command fails:**
1. Check the log file: `/var/log/ascend_seclog/ascend_toolkit_install.log` or `/var/log/ascend_seclog/ascend_install.log`
2. Report the error to the user with the relevant log excerpt
3. **Do NOT** try alternative install paths or modify permissions without user approval
4. Wait for user instructions on how to proceed

### Step 8: Report Results

After installation completes successfully, tell the user:

1. The installation path
2. The environment initialization command (based on actual install path):
   ```bash
   source <install_path>/cann/set_env.sh
   ```
   Example for root default: `source /usr/local/Ascend/cann/set_env.sh`
   Example for non-root default: `source ${HOME}/Ascend/cann/set_env.sh`
3. This command only takes effect in the current shell session. Suggest adding it to `~/.bashrc` for persistence.

## Important Rules

- Follow the steps in order. Do not skip the chip detection step.
- Do NOT search the web for download URLs. All packages come from the conda repo extraction method.
- Do NOT use conda to install directly. Conda is only used as a source to extract .run binary files.
- Do NOT modify conda configuration on the target machine.
- Always get user confirmation before running install commands.
- Install toolkit before ops/kernels, ops/kernels before optional packages.
- Version 8.5.0+ uses "ops" naming; earlier versions use "kernels".
- If install fails, report the error and wait for user instructions. Do NOT try alternative paths or modify permissions without approval.
- If all download methods fail, direct user to: https://www.hiascend.com/developer/download/community/result

## Network Issues and Proxy

Network problems are common when downloading large CANN packages (often 1-2 GB each).

1. **If a download fails, immediately tell the user.** Do not try random proxy configurations or alternative URLs yourself. Ask the user how to proceed.

2. **For remote installation with no internet**: download and extract .run files on the local machine, then transfer via scp:
   ```bash
   scp -P <port> ./Ascend/*.run <user>@<host>:/tmp/cann_install/Ascend/
   ```

3. **If the user provides proxy settings**: the proxy may require authentication. Ask the user for proxy address and credentials. Set via environment variables with proper URL encoding for special characters:
   ```bash
   export http_proxy="http://<user>:<password>@<proxy_host>:<port>"
   export https_proxy="http://<user>:<password>@<proxy_host>:<port>"
   ```
   **Security**: NEVER save proxy credentials to any file, log, or memory. Use only as transient environment variables. Clear after download:
   ```bash
   unset http_proxy https_proxy
   ```

## Troubleshooting

**Permission Issues**: The install path and all parent directories must have 755 permissions. If installing to `/root/Ascend`, the installer will fail because `/root` typically has 750 permissions. Use `/usr/local/Ascend` or a path under `/home` instead.

**Version Compatibility**: All packages must use the same version.

**Missing zstd**: The extraction step requires `unzstd`. Install with `apt install zstd` or `yum install zstd`.
