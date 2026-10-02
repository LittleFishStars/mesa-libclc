# mesa-libclc

Arch Linux 的 [mesa-libclc](https://gitlab.freedesktop.org/karolherbst/mesa-libclc) 打包（AUR 格式），
用于消除 Mesa Rusticl 的这条提示：

> === Rusticl warning: Patched Mesa libclc not detected. Upstream libclc may contain
> known bugs or breaking changes and isn't guaranteed to work reliably.
> Please visit https://gitlab.freedesktop.org/karolherbst/mesa-libclc for more information. ===

## 背景

Mesa 的 OpenCL 实现 Rusticl 在加载时会检查 `libclc` 是否带有 Mesa 自己的补丁——具体是查找符号
`__clc_mesa_libclc_version`。而 Arch 官方仓库的 `libclc` 是上游 LLVM 版本，没有这个符号，
因此任何使用 OpenCL 的程序（`clinfo`、`fastfetch`、Blender Cycles、darktable 等）都会打印上述提示。

Mesa 官方为此维护了 libclc 的 fork，Debian、Yocto 等发行版已经在使用它。Arch 官方尚未采纳，
本仓库填补这个空缺。

## 安装

```bash
git clone https://github.com/LittleFishStars/mesa-libclc
cd mesa-libclc
makepkg -si
```

本包 `provides` / `conflicts` 官方 `libclc`，`makepkg -si` 安装时会提示替换官方包；
`opencl-mesa` 对 `libclc` 的依赖由本包满足。

安装完成后验证：

```bash
fastfetch          # 不再出现 Rusticl 警告
clinfo | head -5   # Platform Name: rusticl
```

## 说明

- 只构建 **SPIR-V 目标**（`spirv-mesa3d-.spv`、`spirv64-mesa3d-.spv`），这正是 Mesa 的 Rusticl
  实际读取的文件（Mesa 26.x 中这两个路径是硬编码的）；官方 `libclc` 里的 `amdgcn` / `nvptx64`
  目标不包含在此包内。
- 构建使用系统 `clang` / `llvm`（当前 23.x），约几分钟；不需要编译整套 LLVM。
- 与 AUR 中 `libclc-minimal-git` 的区别：后者从 `llvm-project.git` 构建并手动打 8 个补丁，
  其 makedepends 依赖 `llvm-minimal-git`、`clang-minimal-git`，代价极高；本包直接使用 fork 的
  发布 tag，构建轻量、版本可复现。

## 回滚到官方 libclc

```bash
sudo pacman -Rdd mesa-libclc
sudo pacman -S libclc
```

## 版本

跟随上游 fork 的发布 tag，当前为 `22.1.8.3`。

## 许可

打包脚本与产物遵循上游 `Apache-2.0 WITH LLVM-exception`。
