## 1) `String` 和 `Symbol` 类型区别

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
 
 ## 2) `Proc` 和 `Lambda` 区别
 
 ### 区别一: return

 ``` ruby 

#proc and lambda
def run_a_proc(p)
  puts 'start...'
  p.call
  puts 'end.'
end

#the lambda will be ignore
def run_couple
  run_a_proc proc { puts 'I am a proc'; return }
  run_a_proc lambda { puts 'I am a lambda'; return }
end

run_couple

 ```
 
 lambda 将会被中断掉，输出：
 
 ``` ruby
 
 start...
 I am a proc
 
 ```
 
 将 lambda 和 proc 位置互调后，则输出全部信息，可见在 proc 中 return 可以中断源方法
