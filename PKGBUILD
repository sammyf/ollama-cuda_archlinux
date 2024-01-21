# Maintainer: Alexander F. Rødseth <xyproto@archlinux.org>
# Contributor: Matt Harrison <matt@harrison.us.com>

pkgname=ollama-cuda
pkgdesc='Create, run and share large language models (LLMs) with CUDA'
pkgver=0.1.21
pkgrel=2
arch=(x86_64)
url='https://github.com/jmorganca/ollama'
license=(MIT)
_ollamacommit=ab6be852c77064d7abeffb0b03c096aab90e95fe # tag: v0.1.20
# The llama.cpp git submodule commit hash can be found here:
# https://github.com/jmorganca/ollama/tree/v0.1.20/llm
_llama_cpp_commit=328b83de23b33240e28f4e74900d1d06726f5eb1
makedepends=(cmake cuda git go)
provides=(ollama)
conflicts=(ollama)
source=(git+$url#commit=$_ollamacommit
        llama.cpp::git+https://github.com/ggerganov/llama.cpp#commit=$_llama_cpp_commit
        sysusers.conf
        tmpfiles.d
        ollama.service)
b2sums=('SKIP'
        'SKIP'
        '65d39053cd1dd09562473c2e58f66a447ce0225b32607685f60350596b3288d6568c1cb897393b20236260e632427de1a952e72fe358407020f6cc7820fd4f60'
        '6f0b6886108e8d5f385bf7f9bebc60218797c53d4b88a69cc98564ad02c558cb86633b4f713ddd919d618146c04ac0a9215aa6c32ae192701af9d7850264dd56' 
	'be72a39e823d6631095ce407c92af6aee8650302eeaaa55a970a43592daaa141369d3a5a5eb6a992f6c2f5461370228ac7a84f3318422419fd868575454487d6') 

prepare() {
  cd ${pkgname/-cuda}

  rm -frv llm/llama.cpp
  
  export GIN_MODE=release

  # Copy git submodule files instead of symlinking because the build process is sensitive to symlinks.
  cp -r "$srcdir/llama.cpp" llm/llama.cpp

  # Turn LTO on and set the build type to Release
  sed -i 's,T_CODE=on,T_CODE=on -D LLAMA_LTO=off -D CMAKE_BUILD_TYPE=Release,g' llm/generate/gen_linux.sh

  sed -i 's,var mode string = gin.DebugMode,var mode string = gin.ReleaseMode,g' server/routes.go
  # Let gen_linux.sh find libcudart.so
  sed -i 's,/usr/local/cuda/lib64,/opt/cuda/targets/x86_64-linux/lib,g' llm/generate/gen_linux.sh

  # Let gpu.go find libnvidia-ml.so from the cuda package
  #sed -i 's,/opt/cuda/lib64/libnvidia-ml.so*,/opt/cuda/targets/x86_64-linux/lib/stubs/libnvidia-ml.so*,g' gpu/gpu.go
  sed -i 's,/opt/cuda/lib64/libnvidia-ml.so*,/lib/libnvidia-ml.so*,g' gpu/gpu.go
}

build() {
  cd ${pkgname/-cuda}
  export CGO_CFLAGS="$CFLAGS" CGO_CPPFLAGS="$CPPFLAGS" CGO_CXXFLAGS="$CXXFLAGS" CGO_LDFLAGS="$LDFLAGS"
  go generate ./...
  go build -buildmode=pie -ldflags=-fno-lto -trimpath -mod=readonly -modcacherw -ldflags=-linkmode=external \
    -ldflags=-buildid='' -ldflags="-X=github.com/jmorganca/ollama/version.Version=$pkgver"
}

check() {
  cd ${pkgname/-cuda}
  go test ./api ./format
  ./ollama --version > /dev/null
}

package() {
  install -Dm755 ${pkgname/-cuda}/${pkgname/-cuda} "$pkgdir/usr/bin/${pkgname/-cuda}"
  install -dm700 "$pkgdir/var/lib/ollama"
  install -Dm644 ollama.service "$pkgdir/usr/lib/systemd/system/ollama.service"
  install -Dm644 sysusers.conf "$pkgdir/usr/lib/sysusers.d/ollama.conf"
  install -Dm644 tmpfiles.d "$pkgdir/usr/lib/tmpfiles.d/ollama.conf"
  install -Dm644 ${pkgname/-cuda}/LICENSE "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}
