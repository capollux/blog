---
layout:   post
title:    "Find Hash Key From Value In Ruby"
date:     2017-12-12 09:33:00 +0900
---

`Hash#key` returns the key for a given value

```ruby
@hash = {"1"=>"one", "2"=>"two", "3"=>"three"}
@hash.key("one") #=> "1"
```

or `nil` if no such value exists:

```ruby
@hash.key("four") #=> nil
```

To get either the value for a key, or the key for a value, you could use:

```ruby
def finder(str)
  @hash[str] || @hash.key(str)
end

finder("1")   #=> "one"
finder("one") #=> "1"
```
