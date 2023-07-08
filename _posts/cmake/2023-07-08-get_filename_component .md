---
published: true
layout: single
title: "[CMake] get_filename_component"
category: cmake
tags:
comments: true
sidebar:
  nav: "mainMenu"
---  
* * *

'get_filename_component' 함수는 CMake에서 파일 경로에서 특정 구성 요소를 추출하는 데 사용되는 CMake 함수입니다. 
이 함수를 사용하여 파일 경로에서 파일 이름, 확장자, 디렉토리 경로 등을 추출할 수 있습니다.

```cmake
get_filename_component(<변수> <경로> <속성>)
```

예를 들어, 다음과 같은 파일 경로를 가정해 봅시다: /path/to/file.txt

```cmake
get_filename_component(FILE_NAME "/path/to/file.txt" NAME)
get_filename_component(BASE_NAME "/path/to/file.txt" NAME_WE)
get_filename_component(EXTENSION "/path/to/file.txt" EXT)
get_filename_component(DIR_PATH "/path/to/file.txt" DIR)

message("File name: ${FILE_NAME}")        # 출력: file.txt
message("Base name: ${BASE_NAME}")        # 출력: file
message("Extension: ${EXTENSION}")        # 출력: .txt
message("Directory path: ${DIR_PATH}")    # 출력: /path/to
```

위의 예제에서는 get_filename_component 함수를 사용하여 파일 경로에서 파일 이름, 기본 이름 (확장자 제외), 
확장자 및 디렉토리 경로를 추출합니다. 추출한 값을 message 함수를 사용하여 출력합니다.

get_filename_component 함수를 사용하면 파일 경로에서 필요한 구성 요소를 추출하여 CMake 스크립트에서 활용할 수 있습니다. 
이를 통해 파일 경로에 대한 작업을 수행하거나 파일 이름, 확장자 등과 관련된 로직을 구현할 수 있습니다.