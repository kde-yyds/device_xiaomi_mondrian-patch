# **A guide on how to build RisingOS with Lindroid and KernelSU for Redmi K60/POCO F5 Pro (Mondrian)**
# Build automatically with a script (recommended)
## Preparation 
### OS
Make sure you have a GNU/Linux environment. Debian and Ubuntu are recommended.  
If you are using Arch Linux, you will encounter errors when building kernel. See the guide below to workaround it.
### Hardware
You need a high performance computer. The most important thing is RAM. At least 16GB RAM is required to build smoothly.  
Be sure to enable enough swap if you have a small RAM.  
Reference: AMD Ryzen 7 7700X + 2*8=16GB DDR5 RAM + TiPlus7100 SSD, 8GB Zram and 64GB Swap (Zswap enabled). Around 3 hour for first full build without ccache.
### Working directory
```
mkdir risingos
cd risingos
```
### Get patches and build script
```
git clone --depth=1 https://github.com/kde-yyds/device_xiaomi_mondrian-patch/
```
### Run script
```
bash device_xiaomi_mondrian-patch/build.sh
```
Then wait until it ends.

# Build manually (advanced)
## Preparation 
### OS
Make sure you have a GNU/Linux environment. Debian and Ubuntu are recommended.  
If you are using Arch Linux, you will encounter errors when building kernel. See the guide below to workaround it.
### Hardware
You need a high performance computer. The most important thing is RAM. At least 16GB RAM is required to build smoothly.  
Be sure to enable enough swap if you have a small RAM.  
Reference: AMD Ryzen 7 7700X + 2*8=16GB DDR5 RAM + TiPlus7100 SSD, 8GB Zram and 64GB Swap (Zswap enabled). Around 3 hour for first full build without ccache.
### Fetch repositories
Switch to a working directory.
```
mkdir risingos
cd risingos
```
#### RisingOS
```
repo init -u https://github.com/RisingOS-Revived/android -b fifteen --git-lfs --depth=1
repo sync -c --no-clone-bundle --optimized-fetch --prune --force-sync -j$(nproc --all)
```
#### Device trees (Thanks to [@flakeforever](https://github.com/flakeforever))
```
git clone --depth=1 https://github.com/flakeforever/device_xiaomi_mondrian device/xiaomi/mondrian
git clone --depth=1 https://github.com/flakeforever/device_xiaomi_mondrian-kernel device/xiaomi/mondrian-kernel
git clone --depth=1 https://github.com/flakeforever/kernel_xiaomi_sm8475 kernel/xiaomi/sm8475
git clone --depth=1 https://github.com/flakeforever/vendor_xiaomi_mondrian vendor/xiaomi/mondrian
git clone --depth=1 https://github.com/AOSPA/android_device_qcom_common device/qcom/common --branch=uvite
git clone --depth=1 https://github.com/flakeforever/device_xiaomi_sepolicy device/xiaomi/sepolicy
git clone --depth=1 https://github.com/flakeforever/hardware_xiaomi hardware/xiaomi
```
#### MIUI Camera
Get it from [here](https://github.com/flakeforever/device_xiaomi_mondrian/issues/16)

#### Lindroid
```
git clone --depth=1 https://github.com/Linux-on-droid/vendor_lindroid vendor/lindroid --branch=lindroid-22.1
git clone --depth=1 https://github.com/Linux-on-droid/libhybris libhybris
git clone --depth=1 https://github.com/Linux-on-droid/external_lxc external/lxc
```
#### KernelSU
```
git clone https://github.com/kde-yyds/android_external_kernelsu external/kernelsu
```
#### Patches
```
git clone --depth=1 https://github.com/kde-yyds/device_xiaomi_mondrian-patch/
```
### Fix errors for Arch Linux (Thanks to [@Finish0314](https://github.com/finish0314) for this workaround)
When building android kernel on Arch Linux, `libyaml` cannot be found and configured correctly. Copy the header and lib to `prebuilts/kernel-build-tools` manually to fix it.
```
cp -r /usr/include/yaml.h prebuilts/kernel-build-tools/linux-x86/include/yaml.h
cp -r /lib64/libyaml-0.so.2.0.9 prebuilts/kernel-build-tools/linux-x86/lib64/libyaml.so
```
### SM8450 audio
```
git clone --depth=1 https://github.com/LineageOS/android_vendor_qcom_opensource_audioreach-graphservices hardware/qcom-caf/sm8450/audio/graphservices --branch=lineage-22.1-caf-sm8450
```
### Apply patches
```
python3 device_xiaomi_mondrian-patch/apply-patches.py
```
#### Patch Descriptions

| Patch Name | Description | Use |
|------------|-------------|-----|
| bionic/0001-early-return-in-cfi_slowpath_common.patch | Early return in CFI slowpath common | Fix Lindroid libhybris hwcomposer segmentation fault |
| vendor,lineage/0001-build-Disable-build-kernel-module.patch | Disable build kernel module in vendor lineage | Use prebuilt kernel module |
| device,xiaomi,mondrian/0001-Add-flags-for-risingos.patch | Add flags for RisingOS | Enable GMS and camera for risingOS |
| device,xiaomi,mondrian/0001-Add-Chipset-Maintainer-properties.patch | Add Chipset Maintainer properties | Add maintainer and chipest name |
| device,xiaomi,mondrian/0001-fix-AodBrightnessService.java-compilation-error.patch | Fix AodBrightnessService.java compilation error | Fixes compilation errors in AodBrightnessService.java on risingOS |
| device,xiaomi,mondrian/0001-Disable-default-frame-rate-limit-for-games.patch | Disable default frame rate limit for games | Unlock 120hz for games by default |
| device,xiaomi,mondrian/0001-inherit-lindroid.patch | Inherit From Lindroid | Add Lindroid to system |
| vendor,rising/0001-remove-RISING_RELEASE_TYPE.patch | Remove Rising release type | Fix mtdoops kernel module load failure due to a too-long fingerprint |
| system,core/0001-disable-selinux.patch | Disable SELinux | Disables SELinux on boot. |
| frameworks,base/0001-SystemUI-Blur-the-background-of-SysUI-dialogs.patch | Blur the background of SystemUI dialogs | Blur the background of SystemUI dialogs |
| frameworks,base/0001-Battery-Fix-battery-time-for-xiaomi-device.patch | Fix battery time for Xiaomi device | Fix battery current capicity (xxx mAh) display error |
| frameworks,base/0001-Increase-blur-radius-to-60px.patch | Increase blur radius to 60px | Increases the blur radius for QS |
| frameworks,base/0001-Ignore-uevent-s-with-null-name-for-Extcon-WiredAcces.patch | Ignore uevents with null name for Extcon Wired Access | Fix android ueventd crash when starting lxc container |
| external,wpa_supplicant_8/0001-fix-hostapd-compilation-error.patch | Fix hostapd compilation error | Resolves compilation errors in hostapd |
| hardware,qcom-caf,sm8450,audio,pal/0001-Revert-Reapply-pal-validate-stream-handle-in-pal-ser.patch | Revert and reapply pal validate stream handle in pal service | Fixes audio service |
| kernel,xiaomi,sm8475/0009-Enable-lxc-configs-for-mondrian.patch | Enable LXC configs for Mondrian | Add LXC support to kernel |
| kernel,xiaomi,sm8475/0003-kernel-module.c-Ignore-symbols-crc-check.patch | Ignore symbols CRC check in kernel module | Add LXC support to kernel |
| kernel,xiaomi,sm8475/0007-overlayfs-dont-make-DCACHE_OP_-HASH-COMPARE-weird.patch | Prevent DCACHE_OP_HASH_COMPARE issues in overlayfs | Add LXC support to kernel |
| kernel,xiaomi,sm8475/0005-gki_defcinfig-add-lxc-config.patch | Add LXC config in GKI defconfig | Add LXC support to kernel |
| kernel,xiaomi,sm8475/0008-enable-KernelSU.patch | Enable KernelSU | Enables prebuilt KernelSU |
| kernel,xiaomi,sm8475/0001-halium-GKI-use-Android-ABI-padding-for-SYSVIPC-task_.patch | Use Android ABI padding for SYSVIPC task in Halium GKI | Add LXC support to kernel, fix bootloop |
| kernel,xiaomi,sm8475/0004-kernel-Use-the-stock-config-for-proc-config.gz.patch | Use the stock config for proc config.gz | Add LXC support to kernel |
| kernel,xiaomi,sm8475/0002-GKI-use-Android-ABI-padding-for-POSIX_MQUEUE-user_st.patch | Use Android ABI padding for POSIX MQUEUE user in GKI | Add LXC support to kernel |
| kernel,xiaomi,sm8475/0006-cgroup-fix-cgroup-prefix.patch | Fix cgroup prefix | Add LXC support to kernel |
| frameworks,native/0001-inputflinger-allow-disabling-input-devices-via-idc.patch | Allow disabling input devices via IDC in inputflinger | Patch for Lindroid |
| vendor,lindroid/0001-revert-refresh-rate-6.patch | Revert refresh rate 6 in Lindroid vendor | Enable native refresh rate for Lindroid |
| vendor,lindroid/0001-disable-hw-overlay-when-starting-perspectived.patch | Disable hardware overlay when starting perspectived in Lindroid vendor | Disables hardware overlay to fix graphical glitches in Lindroid |
| vendor,lindroid/0001-do-not-mount-ion-and-mali0-mount-dma_heap.patch | Do not mount ion and mali0, mount dma heap in Lindroid vendor | Make freedreno work in LXC container |
### Build RisingOS
Switch to bash if you are not on other shell.
```
. build/envsetup.sh
riseup mondrian userdebug
rise b
```
Then wait for hours until the build progress complete.
# Success! So… what’s next?
You’ve done it! Welcome to the elite club of self-builders. You’ve built your operating system from scratch, from the ground up. You are the master/mistress of your domain… and hopefully you’ve learned a bit on the way and had some fun too.  
# Do you need some testing before flash?
## DSU
Select `odm` `product` `system` `system_ext` `vendor` `product` images and compress them into a zip (select Store).  
Push to your mondrian.  
Install with DSU Sideloader, reboot and test.

It works well? Congratulations!  
Then flash it to your device and then share your image with others (if possible).
