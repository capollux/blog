---
layout:   post
title:    "yield and block"
date:     2018-07-16 09:53:00 +0900
---

## block

multi-line은 `do`와 `end`, inline은 `{`와 `}` 사이가 block.

```ruby
# multi-line
(1..3).to_a.each do |n|
  puts n
end

# Inline
(1..3).to_a.each {|n| puts n}
```

여기서 `|n|`은 block parameter.


## yield (w/ Implicit block)

```ruby
def hello
  name = 'zzulu'
  puts "reached the top"
  yield(name) if block_given?
  puts "reached the bottom"
end

hello do |name|
  puts "Hello, #{name}"
end
```

```
# result
reached the top
Hello, zzulu
reached the bottom
```

`yield`가 호출되면 `block`을 실행하고, 나머지 아래 부분을 실행함.

`yield`가 호출될 때, `block`이 주어지지 않으면 `no block given (yield) (LocalJumpError)` 에러가 난다. 에러가 raise 하는걸 막기 위해 `if block_given?`을 추가해 줄 수 있다.

위 예시의 `yield(name)`처럼 parameter로 넘겨주고, `block`에서 `|name|`으로 parameter를 받을 수 있다. 물론 2개 이상도 넘길 수 있음.

ruby는 어떠한 object라도 method로 넘겨줄 수 있고, method는 이 object를 block으로 사용하려고 한다. method의 마지막 parameter 앞에 `&`를 붙이면, 해당 parameter는 method의 block으로 간주된다.

`&`가 붙은 parameter가 이미 Proc object라면, 그냥 method의 block으로 사용된다.

```ruby
def hello
  yield if block_given?
end
 
hello = Proc.new { puts "hello" }
 
hello(&hello)
```

```
hello
```

그러나 만약에 parameter가 Proc object가 아니라면, `#to_proc`을 호출해서 Proc으로 변환하고 method의 block으로 사용한다.

```ruby
def hello
  yield if block_given?
end
 
class Hello
  def to_proc
    Proc.new {puts 'converted hello'}
  end
end
 
hello(&Hello.new)
```

```
converted hello
```


## &block (ampersand parameter, w/ Explicit block)

method에서 `&block`을 parameter로 받으면 method가 local variable 대신 block을 참조한다. (variable 이름이 block인거지 다른 걸 사용해도 상관 없다.)

```ruby
def hello(&block)
  puts block
  block.call
end

hello { puts "Hello!" }
```

```
#<Proc:0x007fc28d8ef600@example.rb:6>
Hello!
```

이런 식으로 `block` 자체를 method가 넘겨 받아서 `#call` method로 `block`을 실행할 수도 있다. `yield`를 사용하는 거랑 같은데, 가독성때문에 `#call` 방식을 선호하는 사람들도 있다.


## Return value

`yield`의 return 값은 `block`안에서 마지막으로 평가된 표현식(last evaluated expression)이다.

```ruby
def hello
  name = yield
  puts "Hello, #{name}"
end

hello do
  'zzulu'
end
```

```
Hello, zzulu
```

`block`안에서 `return`을 사용하면 method 안이 아니기 때문에 `unexpected return (LocalJumpError)` 에러가 난다.

`break`와 `next`를 사용하면 값을 return하고 호출을 종료한다. `break`는 `yield`나 `#call`을 호출한 method나 iterator도 함께 종료한다. `next`는 `yield`나 `#call`을 호출한 methed의 line으로 돌아가 값을 return하고 method를 마저 실행한다.


## to_proc

object 앞에 `&`가 붙으면 `#to_proc`이 호출되어 Proc object로 변환되고 block으로 사용된다고 했었다. 아래 예시의 `#map`과 `#inject`의 결과는 같다.

```ruby
# Without Symbol#to_proc
[1, 2, 3].map { |n| n.to_s }
[3, 4, 5].inject { |product, n| product * n }

# With Symbol#to_proc
[1, 2, 3].map(&:to_s)
[3, 4, 5].inject(&:*)
```

symbol class에 `#to_proc`를 아래와 같이 직접 구현할 수 있다. (이미 구현되어 있다.)

```ruby
class Symbol
  def to_proc
    Proc.new { |obj, *args| obj.send(self, *args) } # 실제로 이 코드가 작성되어 있는건 아닐거다.
  end
end
```

아래 예시처럼 재정의해서 사용할 수도 있다.

```ruby
class Integer
  def to_proc
    Proc.new do |obj, *args|
      obj % self == 0
    end
  end
end

numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10].select(&3)
puts numbers
```

```
3
6
9
```
