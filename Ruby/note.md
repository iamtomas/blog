1) `String` 和 `Symbol` 类型区别

``` ruby

# String
a = "123" # => 123
a.object_id # => 280
a = "123" # => 123
a.object_id # => 300

# Symbol
a = :tomas # => :tomas
a.object_id # => 2181148
a = :tomas # => :tomas
a.object_id # => 2181148

```

 `#object_id` 即内存中指针指向的地址; `String` 每次赋值都会生成一个新的内存空间， `Symbol` 则与 `String` 不同，它效率高、耗存少且不可变，所以一般不频繁修改的话优先使用 `Symbol` , 占用更少资源、性能更优
 
 2) 
