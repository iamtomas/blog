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
 
 ### 区别二：参数约束不同
 
 ``` ruby
 
 # lambda must be passed matched parameters
 hello_proc = proc do |a, b|
   puts 'hello proc'
 end
 # hello_proc.call 1 # no error
 hello_proc.call
 
 hello_lambda = lambda do |a, b|
  puts 'hello lambda'
 end

 # hello_lambda.call # occure exception
 hello_lambda.call 1, 2
 
 ```
 
 ## 3) 了解 `Block` 
 
 - 作为参数
 - 匿名函数
 - Callback
 - 使用 do / end 或者 {} 来定义

 ``` ruby
 
 # before ruby 2.0
 
 x = 1
 1..2.each {|x| puts x} # => x = 3
 
 ```
 
 注：ruby 2.0 之前的版本，块代码会污染变量
 
 通过 `yield` 和 `call` 调用块代码 
 
 ## 4) 了解 `Boolean` 表达式
 
 - &&,||,!
 - and,or,not

 注：表达式的优先级
 
 ## 5)  `String` 常用方法
 
 1. `#sub` 和 `#gsub` 区别
  `#sub` 匹配第一个字符并替换
  `#gsub` 匹配所有字符并替换
  
  ``` ruby
  
  a = "hello world"
  a.sub('o','a') # => "hella world"
  a.gsub('o','a') # => "hella warld"
  
  ```
  
  2. `#dup` 作用及与 `#clone` 区别

   ``` ruby
   
   # `#dup` 作用
   a = "hello"
   b = a
   a.object_id == b.object_id # => true
   b = a.dup
   a.object_id == b.object_id # => false
   
   # `#dup` 与 `#clone` 区别
   ## 在 ruby-doc 文档中解释到：`#dup` 不会复制类扩展到模块，例如：
   
   class Klass
     attr_accessor :str
   end

   module Foo
     def foo; 'foo'; end
   end

   s1 = Klass.new #=> #<Klass:0x401b3a38>
   s1.extend(Foo) #=> #<Klass:0x401b3a38>
   s1.foo #=> "foo"

   s2 = s1.clone #=> #<Klass:0x401b3a38>
   s2.foo #=> "foo"

   s3 = s1.dup #=> #<Klass:0x401b3a38>
   s3.foo #=> NoMethodError: undefined method `foo' for #<Klass:0x401b3a38>
   
   ```
  
  ## 6)  内置方法 `send` 和 `respond_to?`
  
  ``` ruby
  
  # 通过 `send` 调用方法
  
  def hi
   puts 'hello'
  end
  
  send(:hi) # => hello
  
  # `respond_to?` 判断是否有这个内置方法
  
  a = 'hello'
  a.respond_to?(:length)
  
  ```
  
  ## 7)  Ruby 的动态特性 `defined_method`
  

