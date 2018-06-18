---
layout:   post
title:    "Encrypted Credentials on Rails 5.2"
---

Rails 버전이 5.2로 올라가면서 `config/secrets.yml` 파일이 사라지고 `config/credentials.yml.enc`, `config/master.key` 파일이 새로 생겼다.

기존 Rails에서는 API key와 같이 외부에 노출되면 안되는 민감한 정보들을 주로 `figaro` gem을 이용해서 관리를 했었는데, Rails 5.2부터는 Credentials를 이용하여 gem 없이도 key들을 관리할 수 있게 되었다.

`credentials.yml.enc`에 실제 key들이 암호화 되어 담겨있고, `master.key`를 이용하여 복호화 및 편집이 가능하다. `master.key` 파일은 rails project를 생성할 때 부터 `.gitignore`에 들어가 있어, GitHub 같은 Git 원격 저장소에 key가 올라가는 상황을 막아준다. 의도적으로 `.gitignore`에서 제거하지 않는 이상 원격 저장소에 올라가는 일은 없다.

`master.key`가 노출된다면 암호화된 `credentials.yml.enc`의 내용을 복호화 할 수 있으므로 노출되지 않도록 주의해야한다.

## Editing Credentials

편집을 위한 명령어는 다음과 같다:

```shell
EDITOR=vi bin/rails credentials:edit
```

`vim`이 아닌 다른 에디터도 사용 가능하다. GUI의 경우엔 해당 editor를 실행하는 command가 있어야 하며, **wait** 옵션과 함께 써주어야 한다. **wait** 옵션은 Mac, Linux에서만 사용 가능하다.

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

Credentials가 위와 같은 내용이라면, 다음과 같이 Ruby 코드상에서 사용가능하다.

```ruby
Rails.application.credentials.aws    #=> {:access_key_id=>123, :secret_access_key=>345}
Rails.application.credentials.aws[:access_key_id]       #=> "123"
Rails.application.credentials.aws[:secret_access_key]   #=> "345"
Rails.application.credentials.secret_key_base           #=> "253cb...
```

`Rails.application.credentials.aws` 같은 최상위 요소는 **Hash**의 형태로 되어있다.

## Per-Enviromnent

[DHH](https://github.com/rails/rails/pull/30067)는 Production 환경에서 주로 Credentials 의존성이 발생하며, 환경을 구분하더라도 결국에는 같은 key를 설정하는 경우가 많아 환경 구분이 불필요하다고 이야기한 적이 있다.

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

Rails 환경은 `Rails.env`를 사용하여 가져오며 `.to_sym`을 사용하여 **Symbol**로 만들어 주어야 한다.

```ruby
Rails.application.credentials[Rails.env.to_sym][:aws][:access_key_id]
```

## Deploying Master Key

일반적으로 Deploy는 GitHub을 통해서 이루어지고, GitHub에는 `credentials.yml.enc`만 올라가있기 때문에 Deploy 서버에서 `master.key`를 설정해주어야 한다. 방법은 2가지가 있다.

### Copy & Paste master.key

- 기존의 `master.key` 파일을 그대로 복사하여, Deploy 서버에 업로드한다.
- `capistrano`를 사용하고 있다면 `linked_files`

### Set Environment Variable


환경변수 `RAILS_MASTER_KEY` 
