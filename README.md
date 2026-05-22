# redroid-script

[简体中文](README.zh-CN.md)

## About this fork

This repository is a maintained fork of [ayasa520/redroid-script](https://github.com/ayasa520/redroid-script).

The original project appears to be inactive, and some upstream download URLs had stopped working. This fork mainly updates those download URLs while keeping the original script behavior as close to upstream as possible.

## Prebuilt images

You can use the prebuilt images below if you do not want to build your own image.

### ARM64 image

Tested environment: Oracle Cloud, Ubuntu 20.04.6 LTS, Linux `5.4.0-167-generic aarch64`, 4 CPU cores, and 24 GB memory.

Install the required kernel modules first:

```bash
apt install linux-modules-extra-$(uname -r)
modprobe binder_linux devices="binder,hwbinder,vndbinder"
modprobe ashmem_linux
```

Run the container:

```bash
docker run -itd --restart=always --privileged \
  --name a11_1 \
  -v ~/redroid/redroid01/data:/data \
  -p 11101:5555 \
  abing7k/redroid:a11_magisk_arm \
  androidboot.redroid_fps=30 \
  androidboot.redroid_gpu_mode=guest
```

### AMD64 image

The AMD64 image includes ARM translation support, but compatibility is limited. Some apps may not run correctly.

Install the required kernel modules first:

```bash
apt install linux-modules-extra-$(uname -r)
modprobe binder_linux devices="binder,hwbinder,vndbinder"
modprobe ashmem_linux
```

On some AMD64 systems, these kernel modules may not stay loaded after a reboot. If that happens, add the commands above to a startup script.

Run the container:

```bash
docker run -itd --restart=always --privileged \
  --name a11_01 \
  -v ~/redroid/redroid01/data:/data \
  -p 11101:5555 \
  abing7k/redroid:a11_magisk_ndk_amd \
  androidboot.redroid_gpu_mode=auto \
  ro.product.cpu.abilist0=x86_64,arm64-v8a,x86,armeabi-v7a,armeabi \
  ro.product.cpu.abilist64=x86_64,arm64-v8a \
  ro.product.cpu.abilist32=x86,armeabi-v7a,armeabi \
  ro.dalvik.vm.isa.arm=x86 \
  ro.dalvik.vm.isa.arm64=x86_64 \
  ro.enable.native.bridge.exec=1 \
  ro.dalvik.vm.native.bridge=libndk_translation.so \
  ro.ndk_translation.version=0.2.2
```

Available image tags:

1. `abing7k/redroid:a11_magisk_arm`
2. `abing7k/redroid:a11_gapps_arm`
3. `abing7k/redroid:a11_gapps_magisk_arm`
4. `abing7k/redroid:a11_arm`
5. `abing7k/redroid:a11_magisk_ndk_amd`
6. `abing7k/redroid:a11_gapps_magisk_ndk_amd`
7. `abing7k/redroid:a11_gapps_ndk_amd`
8. `abing7k/redroid:a11_ndk_amd`

## Connect with scrcpy-web

You can use `scrcpy-web` to connect to the Android container:

```bash
docker run -itd --privileged --name scrcpy-web -p 8000:8000/tcp emptysuns/scrcpy-web:v0.1
docker exec -it scrcpy-web adb connect your_server_ip:11101
```

Open `http://your_server_ip:8000` in your browser, then click **H264 Converter**.

![scrcpy-web H264 converter](assets/202312151943304.png)

Swipe up from the bottom of the screen.

![scrcpy-web screen 1](assets/202312151950429.png)

![scrcpy-web screen 2](assets/202312151952545.png)

## Build your own image

### Remote-Android script

This script adds OpenGApps, Magisk, libndk translation, and Widevine L3 to a ReDroid image without recompiling the entire image.

If the script does not work, please open an issue.

### Requirements

- Docker or Podman
- `lzip`
- Python dependencies from `requirements.txt`

Install the Python dependencies:

```bash
python3 -m pip install -r requirements.txt
```

### Container runtime

The default container runtime is Docker. Use `-c` or `--container` to choose Docker or Podman:

```text
-c {docker,podman}, --container {docker,podman}
```

### Android version

Use `-a` or `--android-version` to select the base ReDroid Android version. Supported values are `8.1.0`, `9.0.0`, `10.0.0`, `11.0.0`, `12.0.0`, `12.0.0_64only`, and `13.0.0`. The default value is `11.0.0`.

```bash
python3 redroid.py -a 11.0.0
```

### Add OpenGApps

![OpenGApps](assets/3.png)

```bash
python3 redroid.py -g
```

### Add libndk translation

![libndk translation](assets/2.png)

`libndk_translation` comes from the guybrush firmware image. It may perform better than `libhoudini` on AMD64 systems.

The script only installs libndk translation on x86 or x86_64 hosts, and only for Android `11.0.0`, `12.0.0`, and `12.0.0_64only`.

```bash
python3 redroid.py -n
```

### Add Magisk

![Magisk](assets/1.png)

Zygisk and modules such as LSPosed should work.

```bash
python3 redroid.py -m
```

### Add Widevine DRM L3

![Widevine DRM L3](assets/4.png)

```bash
python3 redroid.py -w
```

### Example

This command adds OpenGApps, Magisk, libndk translation, and Widevine L3 to the same ReDroid image:

```bash
python3 redroid.py -a 11.0.0 -gmnw
```

On an x86_64 host, the generated image tag is:

```text
redroid/redroid:11.0.0_gapps_ndk_magisk_widevine
```

Then start a container from the generated image. If the script prints a different tag, use the tag printed by the script.

```bash
docker run -itd --restart=always --privileged \
  --name a11_1 \
  -v ~/redroid/redroid01/data:/data \
  -p 11101:5555 \
  redroid/redroid:11.0.0_gapps_ndk_magisk_widevine \
  androidboot.redroid_gpu_mode=guest
```

## Troubleshooting

### Magisk installed: N/A

Based on feedback from some WayDroid users, changing the kernel may solve this issue:

https://t.me/WayDroid/126202

### The device is not Play Protect certified

Run the following commands on the host:

```bash
adb root
adb shell settings get secure android_id
```

![Android ID](assets/202401162356635.png)

Copy the Android ID and register it here:

https://www.google.com/android/uncertified/

### libndk does not work

This fork has only been verified with `redroid/redroid:11.0.0`. Enabling Zygisk may break libndk for 32-bit apps, but ARM64 apps can still work.

### libhoudini does not work

I have not been able to make any version of `libhoudini` work on ReDroid.

### Install APK files

Use `adb install` to install APK files:

```bash
adb install app.apk
```

## Credits

1. [remote-android](https://github.com/remote-android)
2. [waydroid_script](https://github.com/casualsnek/waydroid_script)
3. [Magisk Delta](https://huskydg.github.io/magisk-files/)
4. [vendor_intel_proprietary_houdini](https://github.com/supremegamers/vendor_intel_proprietary_houdini)
