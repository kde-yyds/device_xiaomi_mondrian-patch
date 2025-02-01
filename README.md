# A guide on how to build Derpfest AOSP with Lindroid for Redmi K60/POCO F5 Pro (Mondrian)
## Preparation 
### OS
Make sure you have a GNU/Linux environment. Debian and Ubuntu are recommended.  
If you are using Arch Linux, you will encounter errors when building kernel. See the guide below to fix it.
### Hardware
You need a high performance computer. The most important thing is RAM. At least 16GB RAM is required to build smoothly.
Be sure to enable enough swap if you have a small RAM.
### Fetch repositories
Switch to a working directory.
```
mkdir derpfest
cd derpfest
```
#### RisingOS
```
repo init -u https://github.com/DerpFest-AOSP/manifest.git -b 15 --depth=1
repo sync -c --no-clone-bundle --optimized-fetch --prune --force-sync -j$(nproc --all)
```
#### Device trees
```
git clone --depth=1 https://github.com/flakeforever/device_xiaomi_mondrian device/xiaomi/mondrian
git clone --depth=1 https://github.com/flakeforever/device_xiaomi_mondrian-kernel device/xiaomi/mondrian-kernel
git clone --depth=1 https://github.com/flakeforever/kernel_xiaomi_sm8475 kernel/xiaomi/sm8475
git clone --depth=1 https://github.com/flakeforever/vendor_xiaomi_mondrian vendor/xiaomi/mondrian
git clone --depth=1 https://github.com/AOSPA/android_device_qcom_common device/qcom/common --branch=uvite
git clone --depth=1 https://github.com/flakeforever/device_xiaomi_sepolicy device/xiaomi/sepolicy
git clone --depth=1 https://github.com/flakeforever/hardware_xiaomi hardware/xiaomi
```
#### Lindroid
```
git clone --depth=1 https://github.com/Linux-on-droid/vendor_lindroid vendor/lindroid --branch=lindroid-22.1
git clone --depth=1 https://github.com/Linux-on-droid/libhybris libhybris
git clone --depth=1 https://github.com/Linux-on-droid/external_lxc external/lxc
```
#### Patches
```
git clone --depth=1 https://github.com/kde-yyds/device_xiaomi_mondrian-patch/ --branch=derp
```
### Apply patches
```
python3 device_xiaomi_mondrian-patch/apply-patches.py
```
Apply what you need. Maybe not all patches are required.
### Fix errors for Arch Linux
When building android kernel on Arch Linux, `libyaml` cannot be found and configured correctly. Copy the header and lib to `prebuilts/kernel-build-tools` manually to fix it. (Thanks to [@Finish0314](https://github.com/finish0314) for this workaround)
```
cp -r /usr/include/yaml.h prebuilts/kernel-build-tools/linux-x86/include/yaml.h
cp -r /lib64/libyaml-0.so.2.0.9 prebuilts/kernel-build-tools/linux-x86/lib64/libyaml.so
```

### Build Derpfest
Switch to bash if you are not on it.
```
. build/envsetup.sh
lunch derp_$device-userdebug
mka derp
```
Then wait for hours until the build progress complete.
