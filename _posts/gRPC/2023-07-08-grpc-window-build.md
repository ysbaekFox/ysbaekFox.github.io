---
published: true
layout: single
title: "[gRPC] Window Build (vcpkg, cmake)"
category: grpc
tags: 
comments: true
sidebar:
  nav: "mainMenu"
---  
* * *

**1) vcpkg를 설치**
```shell
git clone https://github.com/microsoft/vcpkg.git
.\bootstrap-vcpkg.bat
```

<br>

**2) vcpkg를 환경 변수에 추가**

![image](https://github.com/ysbaekFox/ysbaekFox.github.io/assets/54944434/29527a1d-a18d-4649-b538-e12eabeb95a6)

<br>

**3) vcpkg로 gprc 설치**
```shell
vcpkg install grpc
```

<br>

설치 완료 후 **vcpkg install grpc**를 한번 더 입력하면 아래와 같은 메세지가 표시 됩니다.

```shell
The following packages are already installed:
    grpc:x86-windows -> 1.51.1
grpc:x86-windows is already installed
Total install time: 1.62 ms
grpc provides CMake targets:

    # this is heuristically generated, and may not be correct
    find_package(gRPC CONFIG REQUIRED)
    # note: 5 additional targets are not displayed.
    target_link_libraries(main PRIVATE gRPC::gpr gRPC::grpc gRPC::grpc++ gRPC::grpc++_alts)
```

<br>

**4) cmake 및 테스트 코드 작성**


i) main.cpp

```c++
#include <iostream>
#include <grpcpp/grpcpp.h>

int main() 
{
    std::cout << "Hello gRPC(" << grpc::Version() << ")World" << std::endl;
    return 0;
}
```

ii) CMakeLists.txt

```cmake

cmake_minimum_required(VERSION 3.0...3.27)

project(grpc-ysbaekFox-examples)

find_package(gRPC CONFIG REQUIRED)

add_executable(exec main.cpp)
target_link_libraries(exec PRIVATE gRPC::gpr gRPC::grpc gRPC::grpc++ gRPC::grpc++_alts)
```

<br>


**5) cmake 빌드**
```
mkdir build
cd build
cmake .. -DCMAKE_TOOLCHAIN_FILE="{your-vcpkg-home-path}/scripts/buildsystems/vcpkg.cmake"
cmake --build . -j 4
```

<br>

**6) 실행**

![image](https://github.com/ysbaekFox/ysbaekFox.github.io/assets/54944434/c0ffb91b-4954-4955-bc40-2a75ee40e477)
