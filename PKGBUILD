# Maintainer: acxz <akashpatel2008 at yahoo dot com>
# Contributor: Sven-Hendrik Haase <svenstaro@archlinux.org>
# Contributor: Konstantin Gizdov (kgizdov) <arch@kge.pw>
# Contributor: Adria Arrufat (archdria) <adria.arrufat+AUR@protonmail.ch>
# Contributor: Thibault Lorrain (fredszaq) <fredszaq@gmail.com>

pkgbase=tensorflow-rocm

# Flags for building without/with cpu optimizations
_build_no_opt=0
_build_opt=1

pkgname=()
[ "$_build_no_opt" -eq 1 ] && pkgname+=(tensorflow-rocm python-tensorflow-rocm)
[ "$_build_opt" -eq 1 ] && pkgname+=(tensorflow-opt-rocm python-tensorflow-opt-rocm)

pkgver=2.15.0
_pkgver=2.15.0
pkgrel=8
pkgdesc="Library for computation using data flow graphs for scalable machine learning"
url="https://www.tensorflow.org/"
license=('APACHE')
arch=('x86_64')
depends=('c-ares' 'pybind11' 'openssl' 'libpng' 'curl' 'giflib' 'icu' 'libjpeg-turbo' 'intel-oneapi-openmp'
         'intel-oneapi-compiler-shared-runtime-libs')
makedepends=('bazel' 'python-numpy' 'rocm-hip-sdk' 'roctracer' 'rccl' 'git' 'miopen' 'python-wheel' 'openmp'
             'python-installer' 'python-setuptools' 'python-h5py' 'python-keras-applications'
             'python-keras-preprocessing' 'cython' 'patchelf' 'python-requests' 'libxcrypt-compat' 'clang'
             'jdk11-openjdk')
optdepends=('tensorboard: Tensorflow visualization toolkit')
source=("$pkgname-$pkgver.tar.gz::https://github.com/tensorflow/tensorflow/archive/v${_pkgver}.tar.gz"
        https://github.com/bazelbuild/bazel/releases/download/6.1.0/bazel_nojdk-6.1.0-linux-x86_64
        fix-c++17-compat.patch)

sha512sums=('51976c7255ffbdb98fe67a28f6ae1c3b9a073e49fe6b44187a53d99654e4af753de53bfa7229cdd1997ac71e8ddecbc15e4759d46c6d24b55eb84c5d31523dfe'
            'b71aed83ae1c3f610df77f7c148703fd3e7aa5901794a2b31c6044c71b3f030831d59f7f3641992105117a422655160fc9b509326b31586c6bca378cbff08762'
            'f682368bb47b2b022a51aa77345dfa30f3b0d7911c56515d428b8326ee3751242f375f4e715a37bb723ef20a86916dad9871c3c81b1b58da85e1ca202bc4901e')

# consolidate common dependencies to prevent mishaps
_common_py_depends=(python-termcolor python-astor python-gast python-numpy python-protobuf
                    absl-py python-h5py python-keras python-keras-applications python-keras-preprocessing
                    python-tensorflow-estimator python-opt_einsum python-astunparse python-pasta
                    python-flatbuffers python-typing_extensions python-ml-dtypes)

get_pyver () {
  python -c 'import sys; print(str(sys.version_info[0]) + "." + str(sys.version_info[1]))'
}

check_dir() {
  # first make sure we do not break parsepkgbuild
  if ! command -v cp &> /dev/null; then
    >&2 echo "'cp' command not found. PKGBUILD is probably being checked by parsepkgbuild."
    if ! command -v install &> /dev/null; then
      >&2 echo "'install' command also not found. PKGBUILD must be getting checked by parsepkgbuild."
      >&2 echo "Cannot check if directory '${1}' exists. Ignoring."
      >&2 echo "If you are not running nacmap or parsepkgbuild, please make sure the PATH is correct and try again."
      >&2 echo "PATH should not be '/dummy': PATH=$PATH"
      return 0
    fi
  fi
  # if we are running normally, check the given path
  if [ -d "${1}" ]; then
    return 0
  else
    >&2 echo Directory "${1}" does not exist or is a file! Exiting...
    exit 1
  fi
}

prepare() {
  # Since Tensorflow is currently imcompatible with our version of Bazel, we're going to use
  # their exact version of Bazel to fix that. Stupid problems call for stupid solutions.
  install -Dm755 "${srcdir}"/bazel_nojdk-6.1.0-linux-x86_64 bazel/bazel
  PATH="/usr/lib/jvm/java-11-openjdk/bin:$PATH"
  export PATH="${srcdir}/bazel:$PATH"
  bazel --version

  # Get rid of hardcoded versions. Not like we ever cared about what upstream
  # thinks about which versions should be used anyway. ;) (FS#68772)
  sed -i -E "s/'([0-9a-z_-]+) .= [0-9].+[0-9]'/'\1'/" tensorflow-${_pkgver}/tensorflow/tools/pip_package/setup.py

  #Patch rocm missing function
  >&2 echo "Patching stream_executor"
  patch tensorflow-${_pkgver}/third_party/xla/xla/stream_executor/rocm/rocm_driver.cc < fix-rocm_driver.patch
  patch tensorflow-${_pkgver}/third_party/xla/xla/stream_executor/rocm/rocm_driver_wrapper.h < fix-rocm_driver_wrapper.patch

  >&2 echo "Patching pip script"
  #Patch pip script copy error if partial previous build
  patch tensorflow-${_pkgver}/tensorflow/tools/pip_package/build_pip_package.sh < fix-bazel_copy.patch

  if [ "$_build_no_opt" -eq 1 ]; then
    cp -r tensorflow-${_pkgver} tensorflow-${_pkgver}-rocm
  fi
  if [ "$_build_opt" -eq 1 ]; then
    cp -r tensorflow-${_pkgver} tensorflow-${_pkgver}-opt-rocm
  fi
  rm -rf tensorflow-${_pkgver}

  # These environment variables influence the behavior of the configure call below.
  export TF_NEED_JEMALLOC=1
  export TF_NEED_KAFKA=1
  export TF_NEED_OPENCL_SYCL=0
  export TF_NEED_AWS=1
  export TF_NEED_GCP=1
  export TF_NEED_HDFS=1
  export TF_NEED_S3=1
  export TF_ENABLE_XLA=1
  export TF_NEED_GDR=0
  export TF_NEED_VERBS=0
  export TF_NEED_OPENCL=0
  export TF_NEED_MPI=0
  export TF_NEED_TENSORRT=0
  export TF_NEED_NGRAPH=0
  export TF_NEED_IGNITE=0
  export TF_NEED_ROCM=1
  # Uncomment this when you want to specify specific ROCM_ARCH(s)
  # Otherwise tensorflow will automatically detect your architecture
  # See: https://github.com/tensorflow/tensorflow/commit/c04822a49d669f2d74a566063852243d993e18b1
  # export TF_ROCM_AMDGPU_TARGETS=gfx803,gfx900,gfx906,gfx908,gfx90a,gfx1030,gfx1100,gfx1101,gfx1102
  export TF_NEED_CLANG=1
  export CLANG_COMPILER_PATH=/usr/bin/clang
  # See https://github.com/tensorflow/tensorflow/blob/master/third_party/systemlibs/syslibs_configure.bzl
  export TF_SYSTEM_LIBS="boringssl,curl,cython,gif,icu,libjpeg_turbo,nasm,png,zlib"
  export TF_SET_ANDROID_WORKSPACE=0
  export TF_DOWNLOAD_CLANG=0
  # export TF_NCCL_VERSION=$(pkg-config nccl --modversion | grep -Po '\d+\.\d+')
  export NCCL_INSTALL_PATH=/usr
  # Does tensorflow really need the compiler overridden in 5 places? Yes.
  # https://github.com/tensorflow/tensorflow/issues/60577
  export CC=gcc
  export CXX=g++
  export GCC_HOST_COMPILER_PATH=/usr/bin/gcc
  export HOST_C_COMPILER=/usr/bin/${CC}
  export HOST_CXX_COMPILER=/usr/bin/${CXX}
  export TF_CUDA_CLANG=0  # Clang currently disabled because it's not compatible at the moment.
  export CLANG_CUDA_COMPILER_PATH=/usr/bin/clang
  export TF_CUDA_PATHS=/opt/cuda,/usr/lib,/usr
  export TF_CUDA_VERSION=$(/opt/cuda/bin/nvcc --version | sed -n 's/^.*release \(.*\),.*/\1/p')
  export TF_CUDNN_VERSION=$(sed -n 's/^#define CUDNN_MAJOR\s*\(.*\).*/\1/p' /usr/include/cudnn_version.h)
  # https://github.com/tensorflow/tensorflow/blob/1ba2eb7b313c0c5001ee1683a3ec4fbae01105fd/third_party/gpus/cuda_configure.bzl#L411-L446
  # according to the above, we should be specifying CUDA compute capabilities as 'sm_XX' or 'compute_XX' from now on
  # add latest PTX for future compatibility
  # Valid values can be discovered from nvcc --help
  export TF_CUDA_COMPUTE_CAPABILITIES=sm_52,sm_53,sm_60,sm_61,sm_62,sm_70,sm_72,sm_75,sm_80,sm_86,sm_87,sm_89,sm_90,compute_90
  export TF_PYTHON_VERSION=$(get_pyver)
  export PYTHON_BIN_PATH=/usr/bin/python$(get_pyver)
  export USE_DEFAULT_PYTHON_LIB_PATH=1

  export BAZEL_ARGS="--config=mkl -c opt"
}

build() {
  if [ "$_build_no_opt" -eq 1 ]; then
    echo "Building with rocm and without non-x86-64 optimizations"
    cd "${srcdir}"/tensorflow-${_pkgver}-rocm
    export CC_OPT_FLAGS="-march=x86-64"
    export TF_NEED_CUDA=0
    export TF_NEED_ROCM=1
    ./configure
    bazel \
      build \
        ${BAZEL_ARGS[@]} \
        //tensorflow:libtensorflow.so \
        //tensorflow:libtensorflow_cc.so \
        //tensorflow:install_headers \
        //tensorflow/tools/pip_package:build_pip_package
    bazel-bin/tensorflow/tools/pip_package/build_pip_package --src "${srcdir}"/tmprocm-src --dst "${srcdir}"/tmprocm --rocm
  fi


  if [ "$_build_opt" -eq 1 ]; then
    echo "Building with rocm and with non-x86-64 optimizations"
    cd "${srcdir}"/tensorflow-${_pkgver}-opt-rocm
    export CC_OPT_FLAGS="-march=native -O3"
    export TF_NEED_CUDA=0
    export TF_NEED_ROCM=1
    ./configure
    bazel \
      build --config=avx_linux \
        ${BAZEL_ARGS[@]} \
        //tensorflow:libtensorflow.so \
        //tensorflow:libtensorflow_cc.so \
        //tensorflow:install_headers \
        //tensorflow/tools/pip_package:build_pip_package
    bazel-bin/tensorflow/tools/pip_package/build_pip_package --src "${srcdir}"/tmpoptrocm-src --dst "${srcdir}"/tmpoptrocm --rocm
  fi
}

_package() {
  # install headers first
  install -d "${pkgdir}"/usr/include/tensorflow
  cp -r bazel-bin/tensorflow/include/* "${pkgdir}"/usr/include/tensorflow/

  # install python-version to get all extra headers
  WHEEL_PACKAGE=$(find "${srcdir}"/$1 -name "tensor*.whl")
  python -m installer --destdir="$pkgdir" $WHEEL_PACKAGE

  # move extra headers to correct location
  local _srch_path="${pkgdir}/usr/lib/python$(get_pyver)"/site-packages/tensorflow/include
  check_dir "${_srch_path}"  # we need to quit on broken search paths
  cp -rf "${_srch_path}"/* "${pkgdir}"/usr/include/tensorflow/

  # clean up unneeded files
  rm -rf "${pkgdir}"/usr/bin
  rm -rf "${pkgdir}"/usr/lib
  rm -rf "${pkgdir}"/usr/share

  # make sure no lib objects are outside valid paths
  local _so_srch_path="${pkgdir}/usr/include"
  check_dir "${_so_srch_path}"  # we need to quit on broken search paths
  find "${_so_srch_path}" -type f,l \( -iname "*.so" -or -iname "*.so.*" \) -print0 | while read -rd $'\0' _so_file; do
    # check if file is a dynamic executable
    ldd "${_so_file}" &>/dev/null && rm -rf "${_so_file}"
  done

  # install the rest of tensorflow
  tensorflow/c/generate-pc.sh --prefix=/usr --version=${pkgver}
  sed -e 's@/include$@/include/tensorflow@' -i tensorflow.pc -i tensorflow_cc.pc
  install -Dm644 tensorflow.pc "${pkgdir}"/usr/lib/pkgconfig/tensorflow.pc
  install -Dm644 tensorflow_cc.pc "${pkgdir}"/usr/lib/pkgconfig/tensorflow_cc.pc
  install -Dm755 bazel-bin/tensorflow/libtensorflow.so "${pkgdir}"/usr/lib/libtensorflow.so.${pkgver}
  ln -s libtensorflow.so.${pkgver} "${pkgdir}"/usr/lib/libtensorflow.so.${pkgver:0:1}
  ln -s libtensorflow.so.${pkgver:0:1} "${pkgdir}"/usr/lib/libtensorflow.so
  install -Dm755 bazel-bin/tensorflow/libtensorflow_cc.so "${pkgdir}"/usr/lib/libtensorflow_cc.so.${pkgver}
  ln -s libtensorflow_cc.so.${pkgver} "${pkgdir}"/usr/lib/libtensorflow_cc.so.${pkgver:0:1}
  ln -s libtensorflow_cc.so.${pkgver:0:1} "${pkgdir}"/usr/lib/libtensorflow_cc.so
  install -Dm755 bazel-bin/tensorflow/libtensorflow_framework.so "${pkgdir}"/usr/lib/libtensorflow_framework.so.${pkgver}
  ln -s libtensorflow_framework.so.${pkgver} "${pkgdir}"/usr/lib/libtensorflow_framework.so.${pkgver:0:1}
  ln -s libtensorflow_framework.so.${pkgver:0:1} "${pkgdir}"/usr/lib/libtensorflow_framework.so
  install -Dm644 tensorflow/c/c_api.h "${pkgdir}"/usr/include/tensorflow/tensorflow/c/c_api.h
  install -Dm644 LICENSE "${pkgdir}"/usr/share/licenses/${pkgname}/LICENSE

  # Fix interoperability of C++14 and C++17. See https://bugs.archlinux.org/task/65953
  patch -Np0 -i "${srcdir}"/fix-c++17-compat.patch -d "${pkgdir}"/usr/include/tensorflow/absl/base
}

_python_package() {
  WHEEL_PACKAGE=$(find "${srcdir}"/$1 -name "tensor*.whl")
  python -m installer --destdir="$pkgdir" $WHEEL_PACKAGE

  # create symlinks to headers
  local _srch_path="${pkgdir}/usr/lib/python$(get_pyver)"/site-packages/tensorflow/include/
  check_dir "${_srch_path}"  # we need to quit on broken search paths
  find "${_srch_path}" -maxdepth 1 -mindepth 1 -type d -print0 | while read -rd $'\0' _folder; do
    rm -rf "${_folder}"
    _smlink="$(basename "${_folder}")"
    ln -s /usr/include/tensorflow/"${_smlink}" "${_srch_path}"
  done

  # tensorboard has been separated from upstream but they still install it with
  # tensorflow. I don't know what kind of sense that makes but we have to clean
  # it out from this pacakge.
  rm -rf "${pkgdir}"/usr/bin/tensorboard

  install -Dm644 LICENSE "${pkgdir}"/usr/share/licenses/${pkgname}/LICENSE
}

package_tensorflow-rocm() {
  pkgdesc="Library for computation using data flow graphs for scalable machine learning (with ROCM)"
  depends+=(rocm-hip-sdk miopen rccl)
  conflicts=(tensorflow)
  provides=(tensorflow)

  cd "${srcdir}"/tensorflow-${_pkgver}-rocm
  _package tmprocm
}

package_tensorflow-opt-rocm() {
  pkgdesc="Library for computation using data flow graphs for scalable machine learning (with ROCM and AVX CPU optimizations)"
  depends+=(rocm-hip-sdk miopen rccl)
  conflicts=(tensorflow)
  provides=(tensorflow tensorflow-rocm)

  cd "${srcdir}"/tensorflow-${_pkgver}-opt-rocm
  _package tmpoptrocm
}

package_python-tensorflow-rocm() {
  pkgdesc="Library for computation using data flow graphs for scalable machine learning (with ROCM)"
  depends+=(tensorflow-rocm rocm-hip-sdk miopen rccl "${_common_py_depends[@]}")
  conflicts=(python-tensorflow)
  provides=(python-tensorflow)

  cd "${srcdir}"/tensorflow-${_pkgver}-rocm
  _python_package tmprocm
}

package_python-tensorflow-opt-rocm() {
  pkgdesc="Library for computation using data flow graphs for scalable machine learning (with ROCM and AVX CPU optimizations)"
  depends+=(tensorflow-opt-rocm rocm-hip-sdk miopen rccl "${_common_py_depends[@]}")
  conflicts=(python-tensorflow)
  provides=(python-tensorflow python-tensorflow-rocm)

  cd "${srcdir}"/tensorflow-${_pkgver}-opt-rocm
  _python_package tmpoptrocm
}

# vim:set ts=2 sw=2 et:
