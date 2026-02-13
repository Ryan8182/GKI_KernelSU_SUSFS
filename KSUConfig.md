# KernelSU Configuration Documentation

## Current Configuration Status

This repository is configured to use the official KernelSU repository from [tiann/KernelSU](https://github.com/tiann/KernelSU).

### Branch Configuration

Located in `.github/workflows/gki-kernel.yml` (lines 220-263):

**For "Stable(标准)" builds:**
- Uses `-s builtin` flag
- Checks out the latest tagged version (stable release)

**For "Dev(开发)" builds:**
- **Official variant**: Uses `-s main` flag → Targets the `main` branch (active development)
- **MKSU variant**: Uses `-s main` flag → Targets the `main` branch
- **Next variant**: Uses `-s next-susfs-dev` flag → Targets KernelSU-Next repository
- **SukiSU variant**: Uses `-s builtin` flag → Uses latest tag

### Setup Command

The Official KernelSU variant uses:
```bash
curl -LSs "https://raw.githubusercontent.com/tiann/KernelSU/main/kernel/setup.sh" | bash -s main
```

This command:
1. Downloads the setup script from the `main` branch (where the setup script is located)
2. Executes it with `-s main` parameter to checkout the `main` branch of KernelSU

## Branch Analysis

### Main Branch (Currently Used for Dev Builds)
- **Status**: Active development branch
- **Latest Commit**: 2026-02-11
- **Commit SHA**: `c44adc14c7fdaf9e7d3de8a32575bf15e1e35206`

**Recent Important Updates:**
- `kernel: use correct errno when add_try_umount failed` (#3212)
- `manager: Allow patching for devices with other different KMI versions` (#3185)
- `kernel: Fix setup_selinux using __task_cred directly` (#3189)
- `kernel: Fix potential memory leaks` (#3170)
- `kernel: improve Git repository detection for KernelSU versioning` (#3108)

### Dev Branch
- **Status**: Experimental features
- **Latest Commit**: 2026-01-27
- **Commit SHA**: `1bc89429a83af92a36ed1467d4e65e2f61865455`

**Notable Feature:**
- `ksud: boot-patch: replace magiskboot with android-bootimg (experimental)`

## Current Feature Set

### Integrated Components
1. **KernelSU**: Root solution for Android GKI devices (Official/MKSU/Next/SukiSU variants)
2. **SUSFS**: Addon root hiding kernel patches (v1.5.x+)
3. **BBR**: TCP congestion control algorithm set as default
4. **Wireguard**: VPN protocol support
5. **LZ4KD**: ZRAM compression algorithm from HUAWEI source
6. **Additional ZRAM algorithms**: LZ4K, LZ4HC, deflate, 842, lz4k_oplus

### Configuration Options
- **ZRAM Support**: Enabled by default with multiple compression algorithms
- **KPM Support**: Enabled for SukiSU variant
- **BBR**: Set as default TCP congestion algorithm
- **SUSFS Features**:
  - SUS_MAP
  - SUS_PATH (disabled for 6.6 kernels)
  - SUS_MOUNT
  - SUS_KSTAT
  - TRY_UMOUNT
  - SPOOF_UNAME
  - HIDE_KSU_SUSFS_SYMBOLS
  - SPOOF_CMDLINE_OR_BOOTCONFIG
  - OPEN_REDIRECT

## Optimization Notes

### Already Implemented Optimizations
1. ✅ Using latest KernelSU main branch for development builds
2. ✅ SUSFS patches properly integrated for all supported Android versions
3. ✅ BBR congestion algorithm enabled and set as default
4. ✅ Advanced ZRAM algorithms (LZ4KD, etc.) integrated
5. ✅ Kernel signing with AVB configured
6. ✅ ccache configured for faster builds
7. ✅ Build space optimization (removing unnecessary packages)

### Possible Future Optimizations

1. **KernelSU Dev Branch Experimental Features**
   - The `dev` branch contains experimental boot-patch features that replace magiskboot with android-bootimg
   - **Recommendation**: Monitor this feature for stability before switching from `main` to `dev`

2. **Bazel Build Optimizations**
   - Current: `--lto=thin` (Thin Link Time Optimization)
   - Consider: Testing with full LTO for potentially better performance (at cost of build time)

3. **Build Cache**
   - Already using ccache with 2GB cache size
   - Consider: Increasing cache size to 4GB for frequently built variants

4. **Kernel Modules**
   - Currently building with `BUILD_SYSTEM_DLKM=0`
   - Most ZRAM/compression modules built-in rather than as loadable modules

5. **OnePlus 8 Elite Support**
   - Special `hmbird_patch.c` added for OnePlus 8 Elite platform support
   - Currently activated via `supp_op` flag

6. **Baseband-guard Support**
   - LSM-based protection against illegal writes to critical partitions
   - Optional feature (disabled by default)
   - Can be enabled via `BBG` input parameter

## Supported Android/Kernel Versions

| Android Version | Kernel Version | Status |
|----------------|---------------|---------|
| Android 12 | 5.10 | ✅ Fully Supported (sub_level: 236, 240) |
| Android 13 | 5.10 | ✅ Fully Supported |
| Android 13 | 5.15 | ✅ Fully Supported |
| Android 14 | 5.15 | ✅ Fully Supported |
| Android 14 | 6.1 | ✅ Fully Supported |
| Android 15 | 6.6 | ⚠️ Supported (known auto-restart bug in LTS) |

## Build Workflow Structure

Main workflows:
- `gki-kernel.yml`: Core build logic (reusable workflow)
- `kernel-a12-5.10.yml`: Android 12 5.10 kernel matrix
- `kernel-a13-5.10.yml`: Android 13 5.10 kernel matrix
- `kernel-a13-5.15.yml`: Android 13 5.15 kernel matrix
- `kernel-a14-5.15.yml`: Android 14 5.15 kernel matrix
- `kernel-a14-6.1.yml`: Android 14 6.1 kernel matrix
- `kernel-a15-6.6.yml`: Android 15 6.6 kernel matrix

Dispatcher workflows:
- `build-kernel-a12-5-10.yml`: Manual dispatch for Android 12
- `build-kernel-a13-5-10.yml`: Manual dispatch for Android 13
- `build-kernel-a14-6-1.yml`: Manual dispatch for Android 14
- (Additional versions follow same pattern)

## Recommendations

### For Production Use
- ✅ **Use "Stable(标准)" builds** for most users (tagged releases)
- ✅ **Use "Dev(开发)" builds** for testing latest features
- ⚠️ **Monitor KernelSU releases** for security updates

### For Development
- ✅ Current configuration using `main` branch is optimal for active development
- ✅ All recent bug fixes and improvements are automatically included
- ℹ️ Consider periodically testing `dev` branch features in a separate workflow

### For Custom Modifications
1. Fork the repository
2. Modify `.github/workflows/gki-kernel.yml` for custom patches
3. Test thoroughly before deploying to devices
4. Keep track of upstream KernelSU changes

## Changelog

### 2026-02-13
- ✅ Verified configuration targets official KernelSU repository
- ✅ Confirmed "Dev(开发)" builds use `main` branch (latest development)
- ✅ Documented current optimization status
- ✅ Added notes on possible future optimizations
- ℹ️ No changes needed - current configuration is optimal

## References

- [KernelSU Official Repository](https://github.com/tiann/KernelSU)
- [KernelSU Documentation](https://kernelsu.org)
- [SUSFS Repository](https://gitlab.com/simonpunk/susfs4ksu)
- [Kernel Patches Repository](https://github.com/Tools-cx-app/kernel_patches)
- [SukiSU Patches](https://github.com/ShirkNeko/SukiSU_patch)
