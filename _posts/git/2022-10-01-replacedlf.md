---
published: true
layout: single
title: "[Git] Git Error - CRLF will be replaced by LF"
category: git
tags:
comments: true
sidebar:
  nav: "mainMenu"
---  
* * *

유닉스 시스템에서는 한 줄의 끝이 LF(Line Feed)로 이루어지는 반면, 
Window에서는 줄 하나가 CR(Carriage Return)와 LF(Line Feed), 즉 CRLF로 이루어지기 때문에 발생.

```
warning: CRLF will be replaced by LF in some/file.file.
The file will have its original line endings in your working directory.
```

아래 명령어 입력 후 해결
```
$ git config --global core.autocrlf true  // window
$ git config --global core.autocrlf input // linux
```