---
layout:   post
title:    "Encrypted Credentials on Rails 5.2"
date:     2018-06-19 00:03:00 +0900
---

Rails 버전이 5.2로 올라가면서 `config/secrets.yml` 파일이 사라지고 `config/credentials.yml.enc`, `config/master.key` 파일이 새로 생겼다.

`credentials.yml.enc`에 실제 key들이 암호화 되어 담겨있고, `master.key`를 이용하여 복호화 및 편집이 가능하다. 

Secrets는 key들이 담긴 평문으로된 `secrets.yml` 파일을 관리하는 형태라서 필요에 따라 파일을 통째로 deploy 서버나 다른 로컬로 복사해야하는 상황이 생기는데 이와 달리, Credentials는 열쇠와 같은 `master.key` 파일만 관리해주면 되기 때문에 간결하다.

`master.key` 파일은 Rails 어플리케이션을 생성할 때 부터 `.gitignore`에 들어가 있어, GitHub 같은 Git 원격 저장소에 key가 올라가는 것을 막아준다. 의도적으로 `.gitignore`에서 제거하지 않는 이상 원격 저장소에 올라가는 일은 없다.

`master.key`가 노출된다면 암호화된 `credentials.yml.enc`의 내용을 복호화 할 수 있으므로 노출되지 않도록 주의해야한다.

## Editing Credentials

편집을 위한 명령어는 다음과 같다:

```shell
EDITOR=vi bin/rails credentials:edit
```

`vim`이 아닌 다른 에디터도 사용 가능하다. GUI의 경우엔 해당 editor를 실행하는 command가 있어야 하며, `--wait` 옵션과 함께 써주어야 한다. `--wait` 옵션은 Mac, Linux에서만 사용 가능하다.

```shell
EDITOR="sublime --wait" bin/rails credentials:edit
EDITOR="atom --wait" bin/rails credentials:edit
```

## Using Credentials

편집을 위한 명령어를 입력하면 아래와 같이 yml 형태의 내용을 확인할 수 있다.

```yaml	
aws:
  access_key_id: 123
  secret_access_key: 345
secret_key_base: 253cb0babaf8a8cfd0b948b89f47a6
```

위와 같은 내용이 작성되어 있다면, 다음과 같이 Ruby 코드상에서 사용가능하다.

```ruby
Rails.application.credentials.aws    #=> {:access_key_id=>123, :secret_access_key=>345}
Rails.application.credentials.aws[:access_key_id]       #=> "123"
Rails.application.credentials.aws[:secret_access_key]   #=> "345"
Rails.application.credentials.secret_key_base           #=> "253cb...
```

`Rails.application.credentials.aws` 같은 최상위 요소는 `Hash`의 형태로 되어있다.

## Per-Enviromnent

Credentials은 Production 환경과 연관된 경우가 많고, 환경을 구분해서 설정하더라도 결국에는 같은 key를 사용하는 경우가 많아 환경 구분이 불필요할 것이라고 [DHH](https://github.com/rails/rails/pull/30067)는 이야기하였다.

그럼에도 환경에 따라 다른 key를 설정하고자 하는 경우에는 다음과 같이 작성하고 사용할 수 있다:

```yaml
development:
  aws:
    access_key_id: 123
    secret_access_key: 345
  secret_key_base: 253cb0babaf8a8cfd0b948b89f47a6

production:
  aws:
    access_key_id: 123
    secret_access_key: 345
  secret_key_base: c7924c08cbbe622c24f06bc4e1bdb3
```

Rails 환경은 `Rails.env`를 사용하여 가져오며 `.to_sym`을 사용하여 `Symbol`로 만들어 주어야 한다.

```ruby
Rails.application.credentials[Rails.env.to_sym][:aws][:access_key_id]
```

## Deploying Master Key

주로 deploy는 GitHub을 통해서 이루어지고, GitHub에는 `credentials.yml.enc`만 올라가있기 때문에 Deploy 서버에서 `master.key`를 설정해주어야 한다. 방법은 2가지가 있다.

### Copy & Paste master.key

기존의 `master.key` 파일을 그대로, deploy 서버에 업로드한다. 만약 `capistrano`를 사용하고 있다면, deploy 서버의 `shared/config` 폴더에 `master.key` 파일을 업로드 한 후, 아래의 코드를 `config/deploy.rb` 파일에 추가한다.

```ruby
# Capistrano config/deploy.rb
append :linked_files, "config/master.key"
```

### Set Environment Variable

`RAILS_MASTER_KEY` 환경변수에 `master.key`의 값을 설정한다. Rails는 `RAILS_MASTER_KEY` 환경변수에 설정된 값을 자동으로 master key로 사용을 한다.

```shell
# e.g. ubuntu
export RAILS_MASTER_KEY=c7924c08cbbe622c24f06bc4e1bdb3

# e.g. heroku
heroku config:set RAILS_MASTER_KEY=c7924c08cbbe622c24f06bc4e1bdb3
```

## Upgrading From Previous Rails

[DHH](https://github.com/rails/rails/pull/30067)는 모든 새로운 어플리케이션들은 Credentials를 사용하겠지만, 기존의 어플리케이션들은 억지로 Credentials를 사용할 필요는 없으니 Secrets를 계속 사용하면 된다고 했다.

그럼에도 Credentials를 사용하고자 하는 Rails 5.2 이전 버전으로 생성된 프로젝트는 Rails 버전을 5.2로 업그레이드 한 후, 아래의 코드를 `config/environments/production.rb` 파일에 추가하여 `ENV['RAILS_MASTER_KEY']`와 `config/master.key`를 사용할 수 있다:

```ruby
# config/environments/production.rb
config.require_master_key = true
```

## Conclusion

예전의 Secrets의 방식은 조금 불친절했다. key들을 관리할 목적으로 존재하는 파일이었지만 평문이었고, 어플리케이션을 생성할 때 자동으로 `.gitignore`에 등록이 되어있는 것도 아니어서 실수로 GitHub에 올라가는 경우가 발생하곤 했다. 그러나 Rails 5.2부터 생긴 Credentials는 key들이 암호화가 되어있고, 이를 암호화/복호화 하는 `master.key` 파일이 `.gitignore`에 처음부터 등록이 되어있어 실수로 key들이 GitHub 등을 통하여 노출되는 것을 방지할 수 있게 되었다. 이러한 간결하면서도 확실한 환경 제공이 Rails가 점점 더 친절해지고 있다고 느끼게 하는 부분이 아닌가 싶다.
