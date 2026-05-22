# redroid-script

[English](README.md)

## 关于这个 Fork

本仓库基于 [ayasa520/redroid-script](https://github.com/ayasa520/redroid-script) 维护。

原项目看起来已经较长时间没有维护，部分上游下载地址也已经失效。本仓库主要修复这些失效的下载地址，并尽量保持原脚本行为不变。

## 预构建镜像

如果你不想自己构建镜像，可以直接使用下面的预构建镜像。

### ARM64 镜像

测试环境：Oracle Cloud，Ubuntu 20.04.6 LTS，Linux `5.4.0-167-generic aarch64`，4 核 CPU，24 GB 内存。

先安装并加载所需内核模块：

```bash
apt install linux-modules-extra-$(uname -r)
modprobe binder_linux devices="binder,hwbinder,vndbinder"
modprobe ashmem_linux
```

启动容器：

```bash
docker run -itd --restart=always --privileged \
  --name a11_1 \
  -v ~/redroid/redroid01/data:/data \
  -p 11101:5555 \
  abing7k/redroid:a11_magisk_arm \
  androidboot.redroid_fps=30 \
  androidboot.redroid_gpu_mode=guest
```

### AMD64 镜像

AMD64 镜像包含 ARM 转译支持，但兼容性有限，部分应用可能无法正常运行。

先安装并加载所需内核模块：

```bash
apt install linux-modules-extra-$(uname -r)
modprobe binder_linux devices="binder,hwbinder,vndbinder"
modprobe ashmem_linux
```

在部分 AMD64 系统上，重启后这些内核模块可能不会自动加载。如果遇到这种情况，可以把上面的命令加入开机启动脚本。

启动容器：

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

可用镜像标签：

1. `abing7k/redroid:a11_magisk_arm`
2. `abing7k/redroid:a11_gapps_arm`
3. `abing7k/redroid:a11_gapps_magisk_arm`
4. `abing7k/redroid:a11_arm`
5. `abing7k/redroid:a11_magisk_ndk_amd`
6. `abing7k/redroid:a11_gapps_magisk_ndk_amd`
7. `abing7k/redroid:a11_gapps_ndk_amd`
8. `abing7k/redroid:a11_ndk_amd`

## 使用 scrcpy-web 连接

可以使用 `scrcpy-web` 连接到 Android 容器：

```bash
docker run -itd --privileged --name scrcpy-web -p 8000:8000/tcp emptysuns/scrcpy-web:v0.1
docker exec -it scrcpy-web adb connect your_server_ip:11101
```

在浏览器中打开 `http://your_server_ip:8000`，然后点击 **H264 Converter**。

![scrcpy-web H264 converter](assets/202312151943304.png)

从屏幕底部向上滑动。

![scrcpy-web screen 1](assets/202312151950429.png)

![scrcpy-web screen 2](assets/202312151952545.png)

## 构建自己的镜像

### Remote-Android 脚本

这个脚本可以在不重新编译完整镜像的情况下，为 ReDroid 镜像加入 OpenGApps、Magisk、libndk translation 和 Widevine L3。

如果脚本无法正常工作，请提交 issue。

### 依赖

- Docker 或 Podman
- `lzip`
- `requirements.txt` 中的 Python 依赖

安装 Python 依赖：

```bash
python3 -m pip install -r requirements.txt
```

### 容器运行时

默认容器运行时是 Docker。可以使用 `-c` 或 `--container` 选择 Docker 或 Podman：

```text
-c {docker,podman}, --container {docker,podman}
```

### Android 版本

使用 `-a` 或 `--android-version` 选择基础 ReDroid Android 版本。支持的值包括 `8.1.0`、`9.0.0`、`10.0.0`、`11.0.0`、`12.0.0`、`12.0.0_64only` 和 `13.0.0`。默认值是 `11.0.0`。

```bash
python3 redroid.py -a 11.0.0
```

### 添加 OpenGApps

![OpenGApps](assets/3.png)

```bash
python3 redroid.py -g
```

### 添加 libndk translation

![libndk translation](assets/2.png)

`libndk_translation` 来自 guybrush 固件镜像。在 AMD64 系统上，它的性能可能比 `libhoudini` 更好。

脚本只会在 x86 或 x86_64 主机上安装 libndk translation，并且只支持 Android `11.0.0`、`12.0.0` 和 `12.0.0_64only`。

```bash
python3 redroid.py -n
```

### 添加 Magisk

![Magisk](assets/1.png)

Zygisk 以及 LSPosed 之类的模块应该可以正常工作。

```bash
python3 redroid.py -m
```

### 添加 Widevine DRM L3

![Widevine DRM L3](assets/4.png)

```bash
python3 redroid.py -w
```

### 示例

下面的命令会在同一个 ReDroid 镜像中加入 OpenGApps、Magisk、libndk translation 和 Widevine L3：

```bash
python3 redroid.py -a 11.0.0 -gmnw
```

在 x86_64 主机上，生成的镜像标签是：

```text
redroid/redroid:11.0.0_gapps_ndk_magisk_widevine
```

然后使用生成的镜像启动容器。如果脚本输出了不同的标签，请以脚本输出为准。

```bash
docker run -itd --restart=always --privileged \
  --name a11_1 \
  -v ~/redroid/redroid01/data:/data \
  -p 11101:5555 \
  redroid/redroid:11.0.0_gapps_ndk_magisk_widevine \
  androidboot.redroid_gpu_mode=guest
```

## 常见问题

### Magisk installed: N/A

根据部分 WayDroid 用户反馈，更换内核可能解决这个问题：

https://t.me/WayDroid/126202

### 设备未通过 Play Protect 认证

在宿主机上执行下面的命令：

```bash
adb root
adb shell settings get secure android_id
```

![Android ID](assets/202401162356635.png)

复制 Android ID，并在下面的网站注册：

https://www.google.com/android/uncertified/

### libndk 无法工作

目前这个 fork 只在 `redroid/redroid:11.0.0` 上验证过。开启 Zygisk 可能会导致 32 位应用的 libndk 失效，但 ARM64 应用仍可能正常运行。

### libhoudini 无法工作

我目前没有成功让任何版本的 `libhoudini` 在 ReDroid 上正常工作。

### 安装 APK

可以使用 `adb install` 安装 APK：

```bash
adb install app.apk
```

## 致谢

1. [remote-android](https://github.com/remote-android)
2. [waydroid_script](https://github.com/casualsnek/waydroid_script)
3. [Magisk Delta](https://huskydg.github.io/magisk-files/)
4. [vendor_intel_proprietary_houdini](https://github.com/supremegamers/vendor_intel_proprietary_houdini)
