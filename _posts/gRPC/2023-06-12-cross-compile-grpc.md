---
published: true
layout: single
title: "[gRPC] How to cross compile gRPC, using CMake"
category: grpc
tags: 
comments: true
sidebar:
  nav: "mainMenu"
---  
* * *

## 1. clone gRPC (v1.55.1)
grpc 1.55.1을 사용하겠습니다.

```
$ git clone --recurse-submodules -b v1.55.1 --depth 1 --shallow-submodules https://github.com/grpc/grpc
```

<br>

## 2.  clone된 protobuf 삭제 및 22.0 버전으로 다시 clone
1번에서 자동으로 clone된 protobuf를 삭제하고, specific version으로 다시 clone해 줄 것 입니다.

```shell
$ cd {grpc clone }/grpc/third_party
$ rm -rf protobuf
$ git clone --recurse-submodules -b v22.0 --depth 1 --shallow-submodules https://github.com/protocolbuffers/protobuf.git
```

<br>

## 3. host 환경에서 gRPC Build 및 설치
gRPC를 cross compile 하기 위해서는 host 환경에서 먼저 build가 필요합니다. 왜냐하면 cross compile 시에 host 환경의 grpc plugin 및 protoc generator 등이 필요하기 때문 입니다.

그리고 gRPC는 한번 설치하고 나면 삭제하는 것이 쉽지 않습니다. CMake의 CMAKE_INSTALL_PREFIX을 사용하여 특정 경로에 설치하도록 하겠습니다.

1) 먼저 Host 환경으로 컴파일한 gRPC를 설치할 경로를 생성 합니다.  
```shell
$ mkdir {grpc clone }/host_installed
```

<br>

2) 그리고 나서 Host 환경으로 컴파일한 gRPC를 설치할 경로를 생성 합니다.

```shell
mkdir -p {grpc clone }/cmake/host_build
$ cd {grpc clone }/cmake/host_build
$ cmake ../.. -DgRPC_INSTALL=ON \
 -DCMAKE_INSTALL_PREFIX={grpc clone }/host_installed \
 -DCMAKE_BUILD_TYPE=Release \
 -DgRPC_ABSL_PROVIDER=module \
 -DgRPC_CARES_PROVIDER=module \
 -DgRPC_PROTOBUF_PROVIDER=module \
 -DgRPC_RE2_PROVIDER=module \
 -DgRPC_SSL_PROVIDER=module \
 -DgRPC_ZLIB_PROVIDER=module
```
- gRPC_<submodulename>_PROVIDER=pakcage → package에서 찾아서 사용
- gRPC_<submodulename>_PROVIDER=module →  직접 빌드
- CMAKE_INSTALL_PREFIX=/path → make install 시에 설치할 경로, 지정해주지 않으면 시스템 root에 설치 된다

<br>

3) host_build 경로에서 build 수행
```shell
$ make -j 8
```

<br>

4) host_build 경로에서 install (install 시 host_installed 경로에 설치 된다.)
```shell
$ make install
```

<br>

## 4. toolchain 사용하여 cross compile
참고로 제가 사용한 toolchain에는 cares, ssl, zlib이 이미 설치 되어 있어서 package 옵션으로 설정하여 build 하였습니다.

```shell
$ mkdir -p {grpc clone }/cmake/sdk_build
$ cd {grpc clone }/cmake/sdk_build
$ cmake ../.. -DgRPC_INSTALL=ON \
 -DCMAKE_INSTALL_PREFIX={install-path}} \
 -DCMAKE_BUILD_TYPE=RelWithDebInfo \
 -DgRPC_BUILD_TESTS=OFF \
 -DgRPC_ABSL_PROVIDER=module \
 -DgRPC_CARES_PROVIDER=package \
 -DgRPC_PROTOBUF_PROVIDER=module \
 -DgRPC_RE2_PROVIDER=module \
 -DgRPC_SSL_PROVIDER=package \
 -DgRPC_ZLIB_PROVIDER=package \
 -DgRPC_BUILD_GRPC_CPP_PLUGIN=ON \
 -D_gRPC_PROTOBUF_PROTOC_EXECUTABLE={grpc clone }/host_installed/bin/protoc \
 -D_gRPC_CPP_PLUGIN={grpc clone }/host_installed/bin/grpc_cpp_plugin \
 -DCMAKE_TOOLCHAIN_FILE={toolchain-path}
$ sudo make install
```
- _gRPC_PROTOBUF_PROTOC_EXECUTABLE → 빌드시에 protobuf 정의 파일 generate할 때 사용할 protoc 파일 경로 (host 환경에서 사용해
야하므로 host 환경에 설치된 protoc 경로를 사용해야한다.)
- _gRPC_CPP_PLUGIN → 빌드시에 grpc 정의 파일 generate할 때 사용할 plugin 파일 경로 (host 환경에서 사용해야하므로 host 환경에 설치된 
plugin 경로를 사용해야한다.)

## 5. build 확인.
{install-path} 경로에 bin / include / lib 폴더가 생성된 것을 확인합니다. static library로 build 됩니다. 필요한 모든 library를 linking 하지 않으면 linking error가 발생합니다.