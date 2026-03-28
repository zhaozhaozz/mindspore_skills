# MindSpore Version Compatibility Reference

## MindSpore-CANN Version Mapping

This table shows the recommended CANN version for each MindSpore release. In practice, adjacent CANN versions are usually compatible — you may see warnings but functionality is unaffected in most cases.

| MindSpore Version | Recommended CANN | Python Versions |
|---|---|---|
| 2.8.0 | CANN 8.5.0 | 3.9, 3.10, 3.11, 3.12 |
| 2.7.2 | CANN 8.5.0 | 3.9, 3.10, 3.11, 3.12 |
| 2.7.1 | CANN 8.3.RC1 | 3.9, 3.10, 3.11 |
| 2.7.0 | CANN 8.2.RC1 | 3.9, 3.10, 3.11 |
| 2.7.0-rc1 | CANN 8.2.RC1 | 3.9, 3.10, 3.11 |
| 2.6.0 | CANN 8.1.RC1 | 3.9, 3.10, 3.11 |
| 2.5.0 | CANN 8.0.RC3 | 3.9, 3.10, 3.11 |
| 2.4.10 | CANN 8.0.RC3 | 3.9, 3.10 |
| 2.4.1 | CANN 8.0.RC2 | 3.9, 3.10 |
| 2.4.0 | CANN 8.0.RC1 | 3.9, 3.10 |
| 2.3.1 | CANN 7.3.0 | 3.8, 3.9, 3.10 |
| 2.3.0 | CANN 7.3.0 | 3.8, 3.9, 3.10 |
| 2.2.14 | CANN 7.0.0 | 3.8, 3.9, 3.10 |
| 2.2.13 | CANN 7.0.0 | 3.8, 3.9, 3.10 |
| 2.2.12 | CANN 7.0.0 | 3.8, 3.9, 3.10 |
| 2.2.11 | CANN 7.0.0 | 3.8, 3.9, 3.10 |
| 2.2.10 | CANN 7.0.0 | 3.8, 3.9, 3.10 |
| 2.2.1 | CANN 7.0.0 | 3.8, 3.9, 3.10 |
| 2.2.0 | CANN 7.0.0 | 3.7, 3.8, 3.9 |
| 2.1.1 | CANN 6.3.RC3 | 3.7, 3.8, 3.9 |
| 2.1.0 | CANN 6.3.RC2 | 3.7, 3.8, 3.9 |
| 2.0.0 | CANN 6.3.RC1 | 3.7, 3.8, 3.9 |

## whl Download URL Pattern

For Linux (Ascend + CPU unified):
```
https://ms-release.obs.cn-north-4.myhuaweicloud.com/{version}/MindSpore/unified/{arch}/mindspore-{version}-cp{pyver}-cp{pyver}-linux_{arch}.whl
```

For CPU-only (Windows):
```
https://ms-release.obs.cn-north-4.myhuaweicloud.com/{version}/MindSpore/cpu/x86_64/mindspore-{version}-cp{pyver}-cp{pyver}-win_amd64.whl
```

For CPU-only (macOS aarch64):
```
https://ms-release.obs.cn-north-4.myhuaweicloud.com/{version}/MindSpore/cpu/aarch64/mindspore-{version}-cp{pyver}-cp{pyver}-macosx_11_0_arm64.whl
```

For CPU-only (macOS x86_64):
```
https://ms-release.obs.cn-north-4.myhuaweicloud.com/{version}/MindSpore/cpu/x86_64/mindspore-{version}-cp{pyver}-cp{pyver}-macosx_10_15_x86_64.whl
```

Variables:
- `{version}`: e.g., `2.8.0`
- `{arch}`: `aarch64` or `x86_64`
- `{pyver}`: Python version digits, e.g., `311` for 3.11

## Full Release List

The authoritative release list with all versions, platforms, and SHA-256 checksums:
https://raw.atomgit.com/mindspore/docs/raw/master/resource/release/release_list_zh_cn.md

## Notes

- Versions 2.7.0+ use `unified` in the URL path (supports both Ascend and CPU)
- Some older versions (pre-2.3) use `ascend` or `cpu` in the URL path instead of `unified`
- Python 3.12 support was added in MindSpore 2.7.2
- Python 3.11 support was added around MindSpore 2.5.0
- This table may become outdated as new versions are released. When in doubt, consult the full release list above.
