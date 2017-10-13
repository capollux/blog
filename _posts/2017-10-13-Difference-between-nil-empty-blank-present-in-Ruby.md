---
layout:   post
title:    "Difference between .nil?, .empty?, .blank?, .present? in Ruby"
date:     2017-10-13 22:46:00 +0900
---

### nil?

This can be used with any object. It will return `true` when **the object is not nil** and `false` if **the object has something in it**.

```ruby
nil.nil? # => true.
[].nil? # => false.
"".nil? # => false.
"  ".nil? # => false.
```


### empty?

This can not be used with any object, this is valid for using with object of class **Array**, **Hash** and **String**. This will return `true` if **the object does not have any value at all**. If it is used with nil object, it will throw an error "NoMethodError".

```ruby
nil.empty? # => Undefined method.
[].empty? # => true.
"".empty? # => true.
"  ".empty? # => false.
empty? # => false.
```

### blank?

This can be used with any object. It will return `true` if **the object has a value of nil, false, empty, or a white space string**. 

```ruby
nil.blank? # => true.
[].blank? # => true.
"".blank? # => true.
"    ".blank? # => true.
false.blank? # => true.
```

### present?

This can be used with any object. This behave opposite of ***blank?*** exactly.

```ruby
nil.present? # => false.
[].present? # => false.
"".present? # => false.
"  ".present? # => false.
false.present? # => false.
```

Note : ***.blank?*** and ***.present?*** are rails methods.
