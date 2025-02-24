<div align = center>

<img src="./.assets/DogDayAndroid.png" width="200" height="175" alt="banner">

<h1>Build Your Own Kernel with Github Action</h1>

English | [中文](./README_cn.md)

![License](https://img.shields.io/static/v1?label=License&message=BY-NC-SA&logo=creativecommons&color=green)
![Language](https://img.shields.io/github/languages/top/DogDayAndroid/Android-Kernel-Builder)
![Issues](https://img.shields.io/github/issues/DogDayAndroid/Android-Kernel-Builder)
![Pull Requests](https://img.shields.io/github/issues-pr/DogDayAndroid/Android-Kernel-Builder)
<br>

This Github Action helps you build kernels. It reads multiple kernel sources from a configuration file and builds them using different toolchains. Additionally, it supports patching the kernel with KernelSU and uploading the built kernel image.
<br>

---

**[<kbd> <br>  Configure  <br> </kbd>](#configuration-file-syntax)** 
**[<kbd> <br>  Quick Start  <br> </kbd>](#how-to-use)** 
**[<kbd> <br>  Local testing  <br> </kbd>](#local-testing)**

---
</div>

# Github Action

This action contains two jobs: `Set-repos` and `Build-Kernel`.

The `Set-repos` job reads the kernel sources from the configuration file and outputs them to the `Build-Kernel` job. The `Build-Kernel` job uses the outputted kernel sources to build the kernels and upload the built kernel images.

## Trigger

| Event name        | Description  |
| ----------------- | ------------ |
| workflow_dispatch | Manually run |

## Workflow

| Build Step                 | Description                                    |
| -------------------------- | ---------------------------------------------- |
| Checkout                   | Check out code                                 |
| Generate Matrix            | Generate kernel source matrix from config file |
| Create working dir         | Create working directory                       |
| Install prerequisites      | Install necessary dependencies for build       |
| Clone kernel source        | Clone kernel source code                       |
| Get toolchains             | Get toolchains                                 |
| Set args                   | Set build arguments                            |
| Update KernelSU (optional) | Patch kernel with KernelSU                     |
| Make defconfig             | Generate kernel configuration file             |
| Build kernel               | Build kernel                                   |
| Upload Image               | Upload kernel image                            |
| Upload Image.gz            | Upload compressed kernel image                 |
| Upload dtb                 | Upload device tree blob file                   |
| Upload dtbo.img            | Upload device tree overlay file                |

## Configuration File Syntax

Here is an example configuration file (`repos.json`):

```json
[
  {
    "device": "gki",
    "defconfig": "gki_defconfig",
    "anykernel": {
      "repo": "https://github.com/osm0sis/AnyKernel3",
      "branch": "main",
      "configs": {
        "do.devicecheck": true,
        "do.modules": false,
        "do.systemless": true,
        "do.cleanup": true,
        "do.cleanuponabort": true,
        "device.name1": "topaz",
        "device.name2": "tapas",
        "supported.versions": "13 - 15",
        "supported.patchlevels": ""
      }
    },
    "toolchains": [
      {
        "name": "gas",
        "repo": "https://android.googlesource.com/platform/prebuilts/gas/linux-x86",
        "branch": "master"
      },
      {
        "name": "clang",
        "url": "https://github.com/ZyCromerZ/Clang/releases/download/21.0.0git-20250221-release/Clang-21.0.0git-20250221.tar.gz"
      }
    ],
    "params": {
      "ARCH": "arm64",
      "CROSS_COMPILE": "aarch64-linux-gnu-",
      "CROSS_COMPILE_ARM32": "arm-linux-gnueabi-",
      "CROSS_COMPILE_COMPAT": "arm-linux-gnueabi-",
      "CLANG_TRIPLE": "aarch64-linux-gnu-",
      "AR": "",
      "CC": "clang"
    }
  }
]
```

This JSON code describes a build configuration that includes the following:

| Field Name  | Description                                                                                                                         |
| ----------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| device      | The device codename used for kernel directory naming and configuration.                                                              |
| defconfig   | The name of the kernel configuration file to use (e.g., gki_defconfig).                                                             |
| anykernel   | Configuration for AnyKernel3 packaging, including repository URL, branch, and device-specific settings.                             |
| toolchains  | An array containing information about the required toolchains, including the repository URL/download link, branch, and name.        |
| params      | An object containing information about the build parameters, including architecture type, cross-compiler, and compiler information. |

Here's a table of the parameters in the `params` object:

| Parameter Name       | Description                                                                                    |
| -------------------- | ---------------------------------------------------------------------------------------------- |
| ARCH                 | The architecture type, in this case `arm64`.                                                   |
| CROSS_COMPILE        | The cross-compiler prefix, in this case `aarch64-linux-gnu-`.                                  |
| CROSS_COMPILE_ARM32  | The cross-compiler prefix for 32-bit compatibility support, in this case `arm-linux-gnueabi-`. |
| CROSS_COMPILE_COMPAT | The cross-compiler prefix for 32-bit compatibility support, in this case `arm-linux-gnueabi-`. |
| CLANG_TRIPLE         | The clang compiler's triple, in this case `aarch64-linux-gnu-`.                                |
| AR                   | The archiving program for static libraries, which is empty in this case.                       |
| CC                   | The C compiler, which is `clang` in this case.                                                 |

These configuration details will be used in the build process to automate the creation of specific kernel image files.

# How to use

This project's basic usage is as follows:

1. Fork this project on GitHub.

2. Modify the `repos.json` file through the Github website or pull it to your local machine and commit the changes.

3. Go to the `Action` page on Github and find `Build kernels`, then `Run workflow`.

4. Wait for the compilation to finish, then download the compiled product from the corresponding page.

5. Use your preferred packaging software to package the kernel ([AnyKernel3](https://github.com/osm0sis/AnyKernel3), [Android-Image-Kitchen](https://github.com/osm0sis/Android-Image-Kitchen), [MagiskBoot](https://github.com/topjohnwu/Magisk/releases), etc.)

![Artifacts](./.assets/artifacts.png)

# Local testing

If you don't want to run the action on `Github`, you can use [nektos/act](https://github.com/nektos/act) to test this workflow locally and output the files.

## Normal local build (kernel source code is fetched using Git)

This is the recommended local testing process. Simply install [nektos/act](https://github.com/nektos/act) and run the following command:

```sh
# Collect artifacts to /tmp/artifacts folder:
act --artifact-server-path /tmp/artifacts
```

If you want to store the artifacts in a different location, change `/tmp/artifacts` to your preferred directory.

If there are errors, use the `-v` flag to generate an error report and submit an issue. Here's the command:

```sh
# Collect artifacts to /tmp/artifacts folder:
act --artifact-server-path /tmp/artifacts -v
```

## Full local build (kernel source code is stored locally)

If you need to perform a completely local build, consider building as follows:

1. Set up a local `Gitea` or `Gitlab` Git service and modify the configuration file address to point to the local service address.

2. Use `git daemon` to create a secondary image locally.

This is just a suggestion, and we do not provide a specific guide.

# Acknowledgments

- [weishu](https://github.com/tiann) : Developer of KernelSU
- [rifsxd](https://github.com/rifsxd) : Developer of KernelSU-Next
- [rsuntk](https://github.com/rsuntk) : Developer of RKSU
- [AKR Android Developer Community](https://www.akr-developers.com/) ： Provides build tutorials
- [DogDayAndroid/KSU_Thyme_BuildBot](https://github.com/DogDayAndroid/KSU_Thyme_BuildBot) : Predecessor of this project
- [xiaoleGun/KernelSU_Action](https://github.com/xiaoleGun/KernelSU_Action) ： Drawing on some Github Actions
- [UtsavBalar1231/Drone-scripts](https://github.com/UtsavBalar1231/Drone-scripts) ： Drawing on some Github Actions

# Contributor

<a href="https://github.com/DogDayAndroid/Android-Kernel-Builder/graphs/contributors">
  <img src="https://contrib.rocks/image?repo=DogDayAndroid/Android-Kernel-Builder" alt="contributors"/>
</a>

# Star history

[![Star History](https://starchart.cc/DogDayAndroid/Android-Kernel-Builder.svg)](https://starchart.cc/DogDayAndroid/Android-Kernel-Builder)

# License

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>.
