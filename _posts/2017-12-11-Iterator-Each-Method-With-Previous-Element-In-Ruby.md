---
layout:   post
title:    "Iterator Method With Previous Element In Ruby"
date:     2017-12-11 14:14:00 +0900
---
 
Sometimes we need previous element when iterate **Array** or **Hash**.
Method `.each_cons(N)` can do this.

```ruby
# N == 2
[1,2,3,4].each_cons(2) {|a, b| puts "#{a}, #{b}" }
1, 2
2, 3
3, 4
=> nil

# N == 3
[1,2,3,4].each_cons(3) {|a, b, c| puts "#{a}, #{b}, #{c}" }
1, 2, 3
2, 3, 4
=> nil
```

Or you can use like this below:

```ruby
# N == 2
[1,2,3,4].each_cons(2) {|arr| puts "#{arr}" }
[1, 2]
[2, 3]
[3, 4]
=> nil

# N == 3
[1,2,3,4].each_cons(3) {|arr| puts "#{arr}" }
[1, 2, 3]
[2, 3, 4]
=> nil
```
