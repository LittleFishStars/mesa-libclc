# Maintainer: 月落星沉 <ylxc_email@yeah.net>
#
# Mesa 官方的 libclc fork：https://gitlab.freedesktop.org/karolherbst/mesa-libclc
#
# 背景：Arch 官方 libclc 是上游 LLVM 版本，不含 Mesa 的补丁，因此每次加载
# Rusticl（Mesa 的 OpenCL 实现）都会打印提示：
#   === Rusticl warning: Patched Mesa libclc not detected. ...
# 本包提供打过补丁的 libclc，并通过 provides/conflicts 取代官方 libclc。
#
# 说明：该 fork 只构建 SPIR-V 目标（spirv-mesa3d- / spirv64-mesa3d-），
# 这正是 Mesa 的 Rusticl 实际使用的部分；官方 libclc 中的 amdgcn/nvptx64
# 目标不包含在此包内。

pkgname=mesa-libclc
pkgver=22.1.8.3
pkgrel=1
pkgdesc="Mesa's patched fork of libclc (SPIR-V), required by Rusticl"
arch=('any')
url="https://gitlab.freedesktop.org/karolherbst/mesa-libclc"
license=('Apache-2.0 WITH LLVM-exception')
makedepends=('cmake' 'ninja' 'clang' 'llvm' 'python' 'spirv-llvm-translator')
provides=('libclc')
conflicts=('libclc')
options=('!lto' '!debug')
source=("$url/-/archive/$pkgver/$pkgname-$pkgver.tar.gz")
sha256sums=('d6fc17db2c4ae20ecfe04060af0572b2ab3701bb13822dab93efcaca0675205e')

prepare() {
  cd "$pkgname-$pkgver"

  # fork 把安装目录改成了 share/mesa-clc，但 Mesa 26.x 仍然只查找
  # /usr/share/clc/{spirv,spirv64}-mesa3d-.spv，因此改回 share/clc
  sed -i 's|\${CMAKE_INSTALL_DATADIR}/mesa-clc|${CMAKE_INSTALL_DATADIR}/clc|' CMakeLists.txt
}

build() {
  cd "$pkgname-$pkgver"

  # 只构建 Mesa 使用的两个 SPIR-V 目标
  cmake -S . -B build -G Ninja \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/usr \
    -DCMAKE_C_COMPILER=clang \
    -DCMAKE_CXX_COMPILER=clang++ \
    -DLLVM_DIR=/usr/lib/cmake/llvm \
    -DLIBCLC_TARGETS_TO_BUILD='spirv-mesa3d-;spirv64-mesa3d-'
  cmake --build build
}

package() {
  cd "$pkgname-$pkgver"

  DESTDIR="$pkgdir" cmake --install build

  # fork 生成的是 mesa-libclc.pc，且路径指向 share/mesa-clc；
  # 这里换成与官方 libclc 包同名的 libclc.pc，指向实际安装位置
  rm -f "$pkgdir/usr/share/pkgconfig/mesa-libclc.pc"
  install -Dm644 /dev/stdin "$pkgdir/usr/share/pkgconfig/libclc.pc" <<END
prefix=/usr
datarootdir=/usr/share
datadir=/usr/share
pkgdatadir=\${datadir}/clc
libexecdir=\${pkgdatadir}

Name: libclc
Description: Library requirements of the OpenCL C programming language
Version: $pkgver
END

  install -Dm644 LICENSE.TXT "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}
