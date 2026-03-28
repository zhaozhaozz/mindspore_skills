# Binary .run Package Installation

## Overview

All CANN packages are installed via binary .run files. The recommended way to obtain these files is to extract them from conda packages (no login required, all versions available).

## Obtaining .run Packages

### Method 1: Extract from Conda Package (Recommended)

Conda packages from the Ascend repository contain the original .run binary installers. This method is preferred because:
- No login required
- All versions and architectures are available
- URL pattern is predictable

**Conda repository base URL:** `https://repo.huaweicloud.com/ascend/repos/conda/`

**Architecture directories:**
- `linux-aarch64/` for aarch64
- `linux-64/` for x86_64

**Conda package naming convention:**

| Component | Conda Package Name (>= 8.5.0) | Conda Package Name (< 8.5.0) |
|-----------|-------------------------------|-------------------------------|
| Toolkit | `ascend-cann-toolkit-{ver}-0.conda` | same |
| 910A Operator | `ascend-cann-910-ops-{ver}-0.conda` | `ascend-cann-kernels-910-{ver}-0.conda` |
| 910B Operator | `ascend-cann-910b-ops-{ver}-0.conda` | `ascend-cann-kernels-910b-{ver}-0.conda` |
| 910C Operator | `ascend-cann-a3-ops-{ver}-0.conda` | N/A (910C supported from 8.5.0+) |
| 310P Operator | `ascend-cann-310p-ops-{ver}-0.conda` | `ascend-cann-kernels-310p-{ver}-0.conda` |
| 310B Operator | `ascend-cann-310b-ops-{ver}-0.conda` | `ascend-cann-kernels-310b-{ver}-0.conda` |
| NNAL | `ascend-cann-nnal-{ver}-0.conda` | same |
| NNAE | `ascend-cann-nnae-{ver}-0.conda` | same |
| NNRT | `ascend-cann-nnrt-{ver}-0.conda` | same |

**Browse available versions:**

```bash
curl -s https://repo.huaweicloud.com/ascend/repos/conda/{arch_dir}/ | grep -oP 'href="ascend-cann-[^"]+\.conda"' | sed 's/href="//;s/"//'
```

**Download and extract .run package:**

For each package, use the specific `pkg-` filename (not a glob) to avoid conflicts when extracting multiple packages:

```bash
# Step 1: Download conda package
curl -O https://repo.huaweicloud.com/ascend/repos/conda/{arch_dir}/{conda_package_name}

# Step 2: Extract pkg tar.zst from conda zip
unzip {conda_package_name} 'pkg-{conda_package_name_without_.conda}.tar.zst'

# Step 3: Extract .run file from tar.zst
tar --use-compress-program=unzstd -xf pkg-{conda_package_name_without_.conda}.tar.zst 'Ascend/*.run'

# The .run file is now at Ascend/<name>.run
```

**Complete example (910C, aarch64, CANN 8.5.0):**

```bash
mkdir -p cann_packages && cd cann_packages

# Download and extract toolkit
curl -O https://repo.huaweicloud.com/ascend/repos/conda/linux-aarch64/ascend-cann-toolkit-8.5.0-0.conda
unzip ascend-cann-toolkit-8.5.0-0.conda pkg-ascend-cann-toolkit-8.5.0-0.tar.zst
tar --use-compress-program=unzstd -xf pkg-ascend-cann-toolkit-8.5.0-0.tar.zst 'Ascend/*.run'
# Result: Ascend/Ascend-cann-toolkit_8.5.0_linux-aarch64.run

# Download and extract 910C operator
curl -O https://repo.huaweicloud.com/ascend/repos/conda/linux-aarch64/ascend-cann-a3-ops-8.5.0-0.conda
unzip ascend-cann-a3-ops-8.5.0-0.conda pkg-ascend-cann-a3-ops-8.5.0-0.tar.zst
tar --use-compress-program=unzstd -xf pkg-ascend-cann-a3-ops-8.5.0-0.tar.zst 'Ascend/*.run'
# Result: Ascend/Ascend-cann-A3-ops_8.5.0_linux-aarch64.run

# All .run files are now in Ascend/ directory
ls Ascend/*.run
```

Note: `unzstd` is provided by the `zstd` package. Install with `apt install zstd` or `yum install zstd` if not available.

### Method 2: Direct Download URLs (Fallback)

Direct URLs are available for some versions but the URL pattern is not always predictable and may require login.

**Known URLs for CANN 8.5.0:**

| Chip | Architecture | Component | URL |
|------|-------------|-----------|-----|
| All | aarch64 | Toolkit | `https://ascend-repo.obs.cn-east-2.myhuaweicloud.com/CANN/CANN%208.5.T63/Ascend-cann_8.5.0_linux-aarch64.run` |
| All | x86_64 | Toolkit | `https://ascend-repo.obs.cn-east-2.myhuaweicloud.com/CANN/CANN%208.5.T63/Ascend-cann_8.5.0_linux-x86_64.run` |
| 910B | aarch64 | Operator | `https://ascend-repo.obs.cn-east-2.myhuaweicloud.com/CANN/CANN%208.5.T63/Ascend-cann-910b-ops_8.5.0_linux-aarch64.run` |
| 910C | aarch64 | Operator | `https://ascend-repo.obs.cn-east-2.myhuaweicloud.com/CANN/CANN%208.5.T63/Ascend-cann-A3-ops_8.5.0_linux-aarch64.run` |

### Method 3: Manual Download (Last Resort)

If both methods above fail, direct the user to download manually:

https://www.hiascend.com/developer/download/community/result

## .run File Naming Convention

### Toolkit (same for all chips)

- Extracted from conda: `Ascend-cann-toolkit_{version}_linux-{arch}.run`
- Direct download from obs: `Ascend-cann_{version}_linux-{arch}.run`

These are the same package with different file names. When using the conda extraction method, the file name contains `toolkit`.

### Operator Packages (chip-specific)

| Chip | .run Name (>= 8.5.0) | .run Name (< 8.5.0) |
|------|----------------------|---------------------|
| 910A | `Ascend-cann-910-ops_{ver}_linux-{arch}.run` | `Ascend-cann-910-kernels_{ver}_linux-{arch}.run` |
| 910B (A2) | `Ascend-cann-910b-ops_{ver}_linux-{arch}.run` | `Ascend-cann-910b-kernels_{ver}_linux-{arch}.run` |
| 910C (A3) | `Ascend-cann-A3-ops_{ver}_linux-{arch}.run` | N/A |
| 310P | `Ascend-cann-310p-ops_{ver}_linux-{arch}.run` | `Ascend-cann-310p-kernels_{ver}_linux-{arch}.run` |
| 310B | `Ascend-cann-310b-ops_{ver}_linux-{arch}.run` | `Ascend-cann-310b-kernels_{ver}_linux-{arch}.run` |

### Optional Packages (architecture only, no chip distinction)

| Package | .run Name |
|---------|----------|
| NNAL | `Ascend-cann-nnal_{ver}_linux-{arch}.run` |
| NNAE | `Ascend-cann-nnae_{ver}_linux-{arch}.run` |
| NNRT | `Ascend-cann-nnrt_{ver}_linux-{arch}.run` |

## Installation Steps

### 1. Install Toolkit (First)

```bash
bash ./Ascend-cann-toolkit_<version>_linux-<arch>.run --quiet --install --install-path=<install_path>
```

- `--install-path`: user-specified installation path
- Default if omitted: root 用户 `/usr/local/Ascend`，非 root 用户 `${HOME}/Ascend`
- `--quiet` suppresses interactive prompts

### 2. Install Operator Package (Second)

```bash
bash ./Ascend-cann-<chip>-ops_<version>_linux-<arch>.run --quiet --install --install-path=<install_path>
```

Use the same `--install-path` as the toolkit.

### 3. Install Optional Packages (If Selected)

```bash
bash ./Ascend-cann-nnal_<version>_linux-<arch>.run --quiet --install --install-path=<install_path>
```

Same pattern for nnae/nnrt. Use the same `--install-path`.

### 4. Initialize Environment

```bash
source <install_path>/cann/set_env.sh
```

The install script also prints this command at the end of installation.

## Network Troubleshooting

If downloads fail, check if a proxy is needed:

```bash
curl -x http://<proxy>:<port> -O <url>
```

Ask the user about proxy configuration if direct download fails.
