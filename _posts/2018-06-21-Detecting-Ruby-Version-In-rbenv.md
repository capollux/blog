---
layout:   post
title:    "Detecting Ruby Version In rbenv"
date:     2018-06-21 15:40:00 +0900
---

rbenv는 아래의 순서대로 루비 버전을 찾는다. `[version]`은 **2.4.4**와 같은 형식을 사용한다.

1. `RBENV_VERSION` 환경 변수에 설정된 ruby version을 사용한다. `rbenv shell [version]` 명령어로 설정할 수 있다.

2. 실행하고자 하는 스크립트 파일이 위치하는 현재 디렉토리와 상위 디렉토리의 계층 구조를 따라 가면서 찾게 되는 `.ruby-version` 파일의 버전을 인식한다. 이 과정은 루트 디렉토리까지 진행된다. `.ruby-version` 파일은 `rbenv local [version]` 명령어로 생성할 수 있다.

3. 전역 version 파일인 `~/.rbenv/version` 파일을 참조하여 ruby version을 인식한다. 이 파일은 `rbenv global [version]` 명령어로 생성할 수 있다.

`rbenv version` 명령어를 터미널에서 입력해보면 아래와 같이 어디서 설정된 ruby 버전인지 확인 가능하다.

```sh
$ rbenv version

# shell
2.4.1 (set by RBENV_VERSION environment variable)

# local
2.4.1 (set by /Users/zzulu/Codes/Rails/rubychatbot/.ruby-version)

# global
2.4.1 (set by /Users/zzulu/.rbenv/version)
```
