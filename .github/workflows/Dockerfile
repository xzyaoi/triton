FROM centos:7
ARG llvm_dir=llvm-project
# Add the cache artifacts and the LLVM source tree to the container
ADD "${llvm_dir}" /source/llvm-project

RUN echo -e "[llvmtoolset-build]\nname=LLVM Toolset 13.0 - Build\nbaseurl=https://buildlogs.centos.org/c7-llvm-toolset-13.0.x86_64/\ngpgcheck=0\nenabled=1" > /etc/yum.repos.d/llvmtoolset-build.repo
# Install build dependencies
RUN yum install --assumeyes python3
RUN yum install --assumeyes centos-release-scl
RUN yum install --assumeyes --nogpgcheck llvm-toolset-13.0
SHELL [ "/usr/bin/scl", "enable", "llvm-toolset-13.0"]

RUN python3 -m pip install --upgrade pip
RUN python3 -m pip install --upgrade cmake ninja

# Install MLIR's Python Dependencies
RUN python3 -m pip install -r /source/llvm-project/mlir/python/requirements.txt

# Configure, Build, Test, and Install LLVM
RUN cmake -GNinja -Bbuild \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_C_COMPILER=clang \
  -DCMAKE_CXX_COMPILER=clang++ \
  -DCMAKE_ASM_COMPILER=clang \
  -DCMAKE_CXX_FLAGS="-Wno-everything" \
  -DCMAKE_LINKER=lld \
  -DCMAKE_INSTALL_PREFIX="/install" \
  -DLLVM_BUILD_UTILS=ON \
  -DLLVM_ENABLE_ASSERTIONS=ON \
  -DLLVM_ENABLE_PROJECTS=mlir \
  -DLLVM_INSTALL_UTILS=ON \
  -DLLVM_TARGETS_TO_BUILD="host;NVPTX;AMDGPU" \
  /source/llvm-project/llvm

RUN ninja -C build install
RUN rm -r /source/llvm-project/
