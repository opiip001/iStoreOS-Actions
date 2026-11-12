# Phicomm N1 FTRACE build

This is an N1-only build path. It does not change the existing generic St1/St2 workflows.

## Build order

1. **N1 - Build FTRACE kernel** (`N1_Build-FTRACE-Kernel.yml`)
   - Uses the `ophub/linux-6.12.y` source and the upstream `ophub/kernel` stable 6.12 config as its base.
   - Enables `CONFIG_FUNCTION_TRACER`, `CONFIG_DYNAMIC_FTRACE`, built-in BTF and the module options needed by Honk/Nikki.
   - Verifies the final kernel .config and the packaged kernel modules.
   - Publishes the custom kernel under this repository's `kernel_n1ftrace` release tag, not the global `kernel_stable` tag.

2. **N1 - Build FTRACE Rootfs** (`N1_Build-FTRACE-Rootfs.yml`)
   - Uses the iStoreOS 25.12.6 armsr/armv8 ImageBuilder.
   - Retains the existing package list and adds Honk, its LuCI app/library/Chinese translation, Nikki, its LuCI app/Chinese translation, the built-in-BTF provider, geo databases and their declared kmod dependencies.
   - Adds signed 25.12 feeds and runtime feed/key files for daeuniverse and Nikki.
   - Publishes a rootfs archive only.

3. **N1 - Package FTRACE firmware** (`N1_Build-FTRACE-Firmware.yml`)
   - Uses the rootfs from step 2 and the custom kernel from step 1.
   - Packages the Phicomm N1 / S905D target with the existing ophub Amlogic image packer.
   - Publishes the flashable `.img.gz` image and SHA256 file.

The kernel and rootfs jobs can be run independently or in either order. The firmware workflow must run after both have published their release assets. The network mode selected for the rootfs must match the firmware workflow input.

## Important compatibility notes

- The uploaded running-kernel config is retained as a reference input outside this branch. The workflow currently starts from the upstream ophub stable 6.12 config and applies explicit N1 FTRACE/module settings; it does **not** claim to be a byte-for-byte rebuild of the running 6.12.114 config.
- The kernel workflow follows the latest 6.12.y patch release at run time. It does not pin 6.12.114, because the ophub source branch may have advanced.
- Honk uses the `vmlinux-btf-builtin` provider because this kernel is built with in-kernel BTF. The Honk/Nikki kmod dependencies are resolved by the matching iStoreOS 25.12.6 ImageBuilder repositories; the custom kernel archive supplies the corresponding runtime modules.
- These workflows only build and publish files in GitHub Releases. They do not write to the N1 eMMC, alter its boot partition, or install/flash anything on the device.
- Build completion is not assumed: check each Actions run and its verification step before using the firmware.

## Manual launch

On the `n1-ftrace-build` branch, run the three workflows in Actions:
- N1 - Build FTRACE kernel
- N1 - Build FTRACE Rootfs (choose DHCP or static)
- N1 - Package FTRACE firmware (choose the same network mode)

The firmware image is ready only after the final packaging workflow succeeds.
