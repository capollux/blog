---
layout:   post
title:    "How rbenv Works"
date:     2018-06-27 15:30:00 +0900
---

rbenv는 설치된 모든 버전의 ruby를 `~/.rbenv/versions` 디렉토리 아래에 둔다.

## rbenv init

`rbenv init -` 명령어를 실행하면 rbenv의 초기화 스크립트가 출력된다.

```sh
$ rbenv init -
export PATH="/Users/zzulu/.rbenv/shims:${PATH}"
export RBENV_SHELL=bash
source '/usr/local/Cellar/rbenv/1.1.1/libexec/../completions/rbenv.bash'
command rbenv rehash 2>/dev/null
rbenv() {
  local command
  command="$1"
  if [ "$#" -gt 0 ]; then
    shift
  fi

  case "$command" in
  rehash|shell)
    eval "$(rbenv "sh-$command" "$@")";;
  *)
    command rbenv "$command" "$@";;
  esac
}
```

따라서 `eval "$(rbenv init –)"` 명령어를 통해서 rbenv의 초기화 스크립트를 실행한다. rbenv를 사용하기 위해서는 초기화가 항상 필요하므로, `~/.bash_profile`에 아래의 내용을 작성하여 터미널이 실행될 때 마다 rbenv의 초기화가 진행되도록 한다.

```sh
if which rbenv > /dev/null; then eval "$(rbenv init -)"; fi
```

## shims

shim에 대한 위키피디아의 정의는 다음과 같다.

> In computer programming, a shim is a small library that transparently intercepts API calls and changes the arguments passed, handles the operation itself, or redirects the operation elsewhere.

rbenv는 `~/.rbenv/shims` 디렉토리가 존재하며, 환경변수 `PATH`의 제일 앞에 이 경로를 추가함으로써 터미널에서 실행되는 명령어가 rbenv의 관리하에 있는 ruby 명령어일 경우 rbenv가 intercept 할 수 있도록 한다.
따라서 `ruby -v` 명령어를 실행하게 되면, `~/.rbenv/shims` 디렉토리에서 shim script인 `ruby`를 찾고, 실행하게 된다.

`~/.rbenv/shims` 디렉토리에 있는 실행 파일들의 내용을 보면 모두가 아래와 같은 동일한 쉘 스크립트 코드를 가지고 있다. `~/.rbenv/shims/rails`든 `~/.rbenv/shims/bundler`든 모두 같은 코드를 가지고 있다.

```sh
#!/usr/bin/env bash
set -e
[ -n "$RBENV_DEBUG" ] && set -x

program="${0##*/}"
if [ "$program" = "ruby" ]; then
  for arg; do
    case "$arg" in
    -e* | -- ) break ;;
    */* )
      if [ -f "$arg" ]; then
        export RBENV_DIR="${arg%/*}"
        break
      fi  
      ;;  
    esac
  done
fi

export RBENV_ROOT="/Users/zzulu/.rbenv"
exec "/usr/local/Cellar/rbenv/1.1.1/libexec/rbenv" exec "$program" "$@"
```

shim 파일 자체로는 많은 일을 하지 않는다. 환경 변수 `RBENV_DIR`와 `RBENV_ROOT`를 설정하고, `/usr/local/Cellar/rbenv/1.0.0/libexec/rbenv exec [original-command] [original-args]` 명령어를 실행한다. 
따라서 `rails s`를 명령어를 입력하면 `~/.rbenv/shims/rails`가 실행이 되는 것이고, 실제로는 `rbenv exec rails s`가 실행되는 것이다.


## rbenv exec

`rbenv exec`는 현재 사용중인 ruby 버전의 bin 디렉토리가 제일 앞에 오도록 환경 변수 `PATH`를 설정하여 script 파일을 실행하도록 한다.

`rbenv exec rails s`를 실행하게 되면 다음과 같은 과정을 거쳐 실행된다.

1. 어떤 버전의 ruby를 사용할 지 찾는다. `rbenv version-name` 명령어를 입력하면 현재 위치에서 사용하게 되는 ruby 버전을 보여주는데 이 버전을 사용한다. ([rbenv가 ruby 버전을 찾는 순서](/2018/06/21/Detecting-Ruby-Version-In-rbenv.html))
```
RBENV_VERSION=2.4.1
```

2. 어떤 명령어를 실행할지 exec script의 첫 argument에서 찾는다. `rbenv exec rails s`를 실행했으므로 지금 상황에서는 `rails` 명령어를 나타낸다.
```
RBENV_COMMAND=rails
```

3. `rbenv which` 명령어 실행하여 실행될 명령어가 있는 path를 찾는다. `rbenv which rails` 명령어를 입력하면 path를 보여주는데 이 path를 사용한다.
```sh
$ rbenv which rails
/Users/zzulu/.rbenv/versions/2.4.1/bin/rails
```
```
RBENV_COMMAND_PATH=/Users/zzulu/.rbenv/versions/2.4.1/bin/rails
```

4. `RBENV_COMMAND_PATH`로부터 `RBENV_BIN_PATH`를 생성하여 환경 변수 `PATH`의 앞에 붙인다.
```
RBENV_BIN_PATH=/Users/shot/.rbenv/versions/2.4.1/bin
export PATH="${RBENV_BIN_PATH}:${PATH}"
```

5. 최종적으로 원본 명령어가 실행된다. 시스템은 shim이 아닌 원래의 binary를 찾아 실행한다.
```
rails s
```

요약하자면 `rbenv exec rails s` 명령어를 실행하면, rbenv exec가 명령어를 `PATH="~/.rbenv/versions/2.4.1/bin:$PATH" rails s`로 변경하여 실행하게 된다.


## rbenv rehash

`rbenv rehash`는 `~/.rbenv/shims` 디렉토리에 shim script를 만들어주는 명령어이다.

이 명령어의 실행결과로 `~/.rbenv/versions/[ruby-version]/bin` 디렉토리의 각 파일명과 동일한 쉘 스크립트 파일이 `~/.rbenv/shims` 디렉토리에 생성된다. 

새로운 젬을 추가(`gem install rails`) 또는 삭제(`gem uninstall rails`)한 후에는 `rbenv rehash` 명령어가 실행되어 현재의 ruby 버전의 실행 파일들(`~/.rbenv/versions/[current-version]/bin/` 디렉토리 안의 스크립트들)의 이름과 동일한 shim script 파일들을 `~/.rbenv/shims` 디렉토리에 생성한다. (오래된 버전의 rbenv는 자동으로 이 작업을 해주지 않아 따로 plugin이 존재했었다. 그러나 지금은 자동으로 해준다.)

shim들이 생성되는 상세한 과정은 다음과 같다.

1.  `~/.rbenv/shims` 디렉토리가 존재하는지 확인한다. 만약에 디렉토리가 없다면 생성한다.

2. shim prototype 파일인 `~/.rbenv/shims/.rbenv-shim`이 존재하는지 확인한다. 만약에 존재한다면 다른 곳에서 rehash가 진행되고 있다는 의미이므로 현재 진행중인 rehash를 종료한다.

3. `.rbenv-shim` 파일이 없다면 생성하여, 다른 rehash 작업이 방해하지 못하도록 한다. 그리고는 위에서 보았던 shim script의 내용을 prototype 파일에 작성한다.

4. 설치된 ruby 버전의 모든 bin 디렉토리를 탐색하여 해당 디렉토리에 존재하는 스크립트의 이름과 동일한 이름의 shim script를 `~/.rbenv/shims` 디렉토리에 생성하고, prototype 파일에 작성된 내용을 shim script에 복사한다. 모든 shim script는 같은 내용으로 작성된다.

5. 마지막으로 prototype 파일을 삭제한다.

