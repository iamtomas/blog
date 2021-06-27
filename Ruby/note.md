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
 
 lambda 将会被中断，输出：
 
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
  
  ## 7)  `defined_method` 实现简易的 `attr_accessor`
  
  ``` ruby
  
  class Hi
   def self.set val
     define_method val do
       instance_variable_get "@#{val}"
     end

     define_method "#{val}=" do |value|
       instance_variable_set "@#{val}", value
     end
   end

   set :name
   set :age

   def initialize(name,age)
     @name = name
     @age = age
   end
 end

 h = Hi.new('tomas',24)

 puts h.name
 h.name = "i am tomas!"
 puts h.name
  
 ```
 
 注：`#instance_variable_set` 和 `#instance_variable_get` 相当于实现 setter 和 getter 
 
 ## 8)  继承
 
 ``` ruby
 
 class User
  attr_accessor :name,:age

  def initialize name, age
    @name = name
    @age = age
  end

  def panels
    @panels ||= ['a', 'b']
  end
 end

 class Admin < User

   def panels
     @panels ||= ['a', 'b', 'c']
   end

 end

 user = User.new('tom', '15')
 p user.panels # => ["a", "b"]

 p '-' * 30

 admin = Admin.new('tomas',24)
 p admin.name # => tomas
 p  admin.panels # => ["a", "b", "c"]
 
 ```
 
 ## 8)  `super`
 
 - `#super` 不带括号表示调用父类的同名函数，并将本函数的**所有参数传入父类**的同名函数；

 - `#super()` 带括号则表示调用父类的同名函数，但是**不传入任何参数**；
 
 ``` ruby
 
 class Parent
  def initialize *args
    args.each{ |arg| puts arg }
  end
    
 end

 class Child < Parent
   def initialize a,b,c
     super() # => a b c
     # super # => put nothing
   end
 end

 a, b, c = *%W[a b c]
 Child.new a, b, c 
 
 ```
 
 ## 9)  Modules 作用
 
 - namespace
 - mixin
 - storage (常量、配置等)
 
  `include` 能把 `module` 中的方法注入为**实例方法**，而 `extend` 则是注入为**类方法**
 
 ## 10) Class & Modules 进阶
 
 - Class 进阶
 
 ``` ruby
 
 Array.class # => Class
 Array.superclass # => Object
 Object.superclass # => BasicObject
 BasicObject.superclass #=> nil
 
 ```
 
 - Method Finding 方法查找是自下而上的
 
   eg: 定义 Admin 类和 User 类，他们都有个同名的方法panels，这时查看继承链 `admin.ancestors` ，可以看到 `[ Admin, User, ... ]` 会自下而上先调用自身的方法
 
 - Method Overwrite 方法覆盖

   Class 和 Module 可以重新打开，Method 可以重新定义
 
   使用场景：可以对第三方 lib 或 gem 打补丁，慎用
 
 - include vs prepend

   include 是把当前的 module 注入到当前类的继承链后边
   prepend 则是把当前的 module 注入到当前类的继承链前边
   
   ``` ruby
   
   module Company
    def self.instance_method
      puts "instance"
    end
   end

   class User
     include Company

   end

   class Admin
     prepend Company

   end

   puts User.ancestors
   puts '-' * 30
   puts Admin.ancestors
   
   ```
   
   ## 11) included 方法
   
   当模块被 include 时**会执行**，同时会传递当前作用域的对象 self ；在 ruby 中这个设计方式很常用
   
   ``` ruby
   
   module Management
    def self.included base
      puts "included #{base}"

      base.include InstanceMethods
      base.extend ClassMethods
      base.include Test1
    end

    module InstanceMethods
      def company_notifies
        puts 'instance methods!'
      end
    end

    module ClassMethods
      def progress
        puts 'class methods!'
      end
    end
   end

   module Test1
     def test1
       puts 'test outer module!'
     end
   end

   class User
     include Management
   end

   puts '-' * 30
   user = User.new

   puts user.company_notifies
   puts '-' * 30

   puts User.progress
   puts '-' * 30

   puts user.test1
   
   ```
   
   ## 12) `Class#class_eval`
   
   ``` ruby
   
   # 重新打开类
   class user
   end

   User.class_eval do
     def hello
       puts 'hello'
     end
   end

   user = User.new
   user.hello
   
   ```
   
   思考：这与直接重新定义 class 有什么区别呢？
   
   `class_eval` 不仅是重新打开目标类、注入其他方法，还能将其放在 included 中自动执行
   
   ## 13) `#instance_eval`
   
   是所有类的实例方法，它打开的是当前实例作用域
   
   ``` ruby
   
   a = "hello"
   a.instance_eval do
     def hi
       puts 'hi'
     end
   end

   puts a 
   puts a.hi
   b = "hello"
   b.hi # => error

   
   ```
   
   需要注意一点， `#instance_eval` 也可以给类定义新方法，因为对 ruby 来说，类本质也是个实例
   
   ``` ruby
   
   class User
   end

   User.class_eval do
     def hello
       puts 'hello'
     end
   end

   User.instance_eval do
     def say_hi
       puts 'hi'
     end
   end

   user = User.new
   User.say_hi
   user.hello
   
   ```
   
   ## 14) Gem 的使用
   
   Gems 本身受版本依赖，由 bundle 管理
   另外一些实际运用的场景，比如：RVM 不仅可以提供一个多 Ruby 版本共存的环境，还可以根据项目管理不同的 gemset.
   
   ``` bash
   
   # 建立 gemset
   rvm use 1.8.7
   rvm gemset create rails23
   
   # 切换语言或者 gemset
   rvm use 1.8.7
   rvm use 1.8.7@rails23
   
   # 清空 gemset 中的 Gem
   rvm gemset empty 1.8.7@rails23
   
   # 删除一个 gemset
   rvm gemset delete rails2-3
   
   ```
 

