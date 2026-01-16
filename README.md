# bone-machine-sm7325-kernel
Stock Android Kernel for the Samsung A52s 5G (One UI only), with backported changes from A73 source code, KernelSU-Next root solution, disabled Samsung Knox, debugging and logging features, and switchable SELinux policy.

# How to build

### Clang
https://android.googlesource.com/platform/prebuilts/clang/host/linux-x86/+archive/refs/heads/main/clang-r530567.tar.gz

### GCC
https://developer.arm.com/-/media/Files/downloads/gnu/14.3.rel1/binrel/arm-gnu-toolchain-14.3.rel1-x86_64-aarch64-none-linux-gnu.tar.xz

### Export PATH
`export PATH="/{path-to-your-clang-bin-folder}/clang/bin:/{path-to-your-gcc-bin-folder}/bin:$PATH"`

### Make defconfig
`make O=$(pwd)/out ARCH=arm64 vendor/a52sxq_kor_single_defconfig`

### Make
```
make -j$(nproc) \
  O=$(pwd)/out \
  ARCH=arm64 \
  CC=clang \
  LD=ld.lld \
  NM=llvm-nm \
  OBJCOPY=llvm-objcopy \
  CLANG_TRIPLE=aarch64-none-linux-gnu- \
  LLVM=1 \
  LLVM_IAS=1 \
  CROSS_COMPILE=llvm- \
  CONFIG_SECTION_MISMATCH_WARN_ONLY=y
```

`Clang Shadow Call Stack (SHADOW_CALL_STACK)` y

`Use virtually mapped shadow call stacks (SHADOW_CALL_STACK_VMAP)`  y

`Link-Time Optimization (LTO)` 2

`Use Clang's Control Flow Integrity (CFI) (CFI_CLANG)` y

`Use CFI shadow to speed up cross-module checks (CFI_CLANG_SHADOW)` y

`Use CFI in permissive mode (CFI_PERMISSIVE)` n

`Use RELR relocation packing (RELR)` y

`Use Clang's ThinLTO (EXPERIMENTAL) (THINLTO)` y

Use `modules=0` to skip module compilation or add Image \ ?

### Prepare module files
`mkdir -p modules_for_zip`

`find . -type f -name "*.ko" -exec cp {} modules_for_zip/ \;`

`find modules_for_zip -type f -name "*.ko" -exec llvm-strip --strip-unneeded {} \;`

### Prepare flashable .zip file
[Magiskboot](https://github.com/topjohnwu/Magisk/releases/download/v29.0/Magisk-v29.0.apk)

[avbtool](https://android.googlesource.com/platform/external/avb/+/refs/heads/main/avbtool.py?format=TEXT)

Extract `boot.img` and `vendor_boot.img` from ROM .zip file

Erase footer from both of them with `avbtool erase_footer --image {file-image}.img`

### boot.img
`magiskboot unpack boot.img`

Copy `Image` file, replace with `kernel` file

Repack with `magiskboot repack boot.img` and place in flashable .zip file `images` folder, change its name to `boot.img`

### dtbo.img
Place `dtbo.img` from `out/arch/arm64/boot/` in `images` folder of the flashable .zip file

### vendor_boot.img
`magiskboot unpack -h vendor_boot.img`

Replace `dtb` file with `arch/arm64/boot/dts/vendor/qcom/yupik.dtb`, rename to `dtb`

Open header file and replace first line with `SRPUE26A001` (probably not necessary)

`mkdir ramdisk && cd ramdisk`

`cpio -idm < ../ramdisk.cpio`

`sudo chown -R $(whoami):$(whoami)`

`rm -rf lib/modules/5.4-gki/*`

Place .ko files from `modules_for_zip/` and `modules.alias`, `modules.dep`, `modules.softdep` and `modules.load` in `lib/modules/`, make sure and `modules.dep` entry lines for each module points to `/lib/modules/` and not `/vendor/lib/modules/`

You can find `modules.*` files for the default defconfig in the kernel source tree if you don't have any

`cp -rf {kernel-source-directory}/firmware/tsp_stm/fts5cu56a_a52sxq* lib/firmware/tsp_stm/`

`sudo chown -R root:root .`

`sudo find . -type d -exec chmod 755 '{}' \;`

`sudo find . -type f -exec chmod 644 '{}' \;`

`sudo find .  -print0 | cpio --null -o -H newc --owner root:root > ../ramdisk.cpio`

`cd ..`

`sudo rm -rf /ramdisk`

Repack with `magiskboot repack vendor_boot.img` and place in flashable .zip file `images` folder, change its name to `vendor_boot.img`

### Make flashable .zip file
`zip -r -9 {flashable-kernel-zip-file-name}.zip *` and flash it on your recovery environment

### Start-over
`make ARCH=arm64 mrproper`

`rm -rf out`

## Update Git Submodules
`git submodule update --init --recursive`

### Kernel su repo update example
```
cd KernelSU-Next
git fetch --tags
git checkout v1.0.10
cd ..
git add KernelSU-Next
git commit -m "Update KernelSU-Next submodule to v1.0.10"
```

# Credits
salvogiangri, utkustnr, RisenID, saadelasfur, Simon

# Resources
https://github.com/Mesa-Labs-Archive/android_kernel_samsung_sm7325/

https://github.com/utkustnr/android_kernel_samsung_sm7325/

https://github.com/RisenID/kernel_samsung_ascendia_sm7325

https://github.com/saadelasfur/android_kernel_samsung_sm7325/

https://github.com/ravindu644/Android-Kernel-Tutorials

https://opensource.samsung.com/uploadList?menuItem=mobile (SM-A736B, SM-A528B, SM-A528N)

## Legacy
```
make -j$(nproc) \
  O=$(pwd)/out \
  ARCH=arm64 \
  LLVM=1 \
  LLVM_IAS=1 \
  CONFIG_SECTION_MISMATCH_WARN_ONLY=y
```

`export PATH="/{path-to-your-clang-bin-folder}/clang/bin:$PATH"`

`mkdir -p /tmp/depmod-temp/lib/modules/5.4.254`

`cp {kernel-source-directory}/out/modules_for_zip/*.ko /tmp/depmod-temp/lib/modules/5.4.254/`

`depmod -b /tmp/depmod-temp -r 5.4.254`

`sed -i 's/\(^\| \)\([^: ]*\.ko\)/\1\/vendor\/lib\/modules\/\2/g' /tmp/depmod-temp/lib/modules/5.4.254/modules.dep`

Regex `(:\s*).*`

### GCC
`find modules_for_zip -type f -name "*.ko" -exec aarch64-none-linux-gnu-strip --strip-unneeded {} \;`

```
make -j$(nproc) -C $(pwd) O=$(pwd)/out \
  DTC_EXT=$(pwd)/tools/dtc CONFIG_BUILD_ARM64_DT_OVERLAY=y \
  ARCH=arm64 \
  CC=clang \
  CROSS_COMPILE=aarch64-none-linux-gnu- \
  CLANG_TRIPLE=aarch64-none-linux-gnu- \
  CONFIG_SECTION_MISMATCH_WARN_ONLY=y \
  vendor/a52sxq_kor_single_defconfig
```

```
make -j$(nproc) -C $(pwd) O=$(pwd)/out \
  DTC_EXT=$(pwd)/tools/dtc CONFIG_BUILD_ARM64_DT_OVERLAY=y \
  ARCH=arm64 \
  CC=clang \
  CROSS_COMPILE=aarch64-none-linux-gnu- \
  CLANG_TRIPLE=aarch64-none-linux-gnu- \
  CONFIG_SECTION_MISMATCH_WARN_ONLY=y
```
