1. 缩排使用两个空格
2. 对于没有主体的类，倾向使用单行定义

```ruby

# 差
class FooError < StandardError
end

# 勉强可以
class FooError < StandardError; end

# 好
FooError = Class.new(StandardError)

```

个人更倾向第二种

3. 定义方法时，避免单行写法，除非是空方法

```ruby

# 好
def some_method
  body
end

# 好
def no_op; end

```

4. 操作符前后适当地添加空格

```ruby

sum = 1 + 2
a, b = 1, 2
class FooError < StandardError; end

```

除了指数操作符

```ruby

# 好
e = M * c**2

```

5. (、[ 之后，]、) 之前，不要添加任何空格。在 { 前后，在 } 之前添加空格

```ruby

# 差
some( arg ).other
[ 1, 2, 3 ].each{|e| puts e}

# 好
some(arg).other
[1, 2, 3].each { |e| puts e }

```

对于插值表达式，括号内两端不要添加空格

```ruby

# 差
"From: #{ user.first_name }, #{ user.last_name }"

# 好
"From: #{user.first_name}, #{user.last_name}"

```

6. 把 when 与 case 缩排在同一层级

```ruby

# 差
case
  when song.name == 'Misty'
    puts 'Not again!'
  when song.duration > 120
    puts 'Too long!'
  when Time.now.hour > 21
    puts "It's too late"
  else
    song.play
end

# 好
case
when song.name == 'Misty'
  puts 'Not again!'
when song.duration > 120
  puts 'Too long!'
when Time.now.hour > 21
  puts "It's too late"
else
  song.play
end

```

7. 当将一个条件表达式的结果赋值给一个变量时，保持分支缩排在同一层级

```ruby

# 差 - 非常费解
kind = case year
when 1850..1889 then 'Blues'
when 1890..1909 then 'Ragtime'
when 1910..1929 then 'New Orleans Jazz'
when 1930..1939 then 'Swing'
when 1940..1950 then 'Bebop'
else 'Jazz'
end

result = if some_cond
  calc_something
else
  calc_something_else
end

# 好 - 结构清晰
kind = case year
       when 1850..1889 then 'Blues'
       when 1890..1909 then 'Ragtime'
       when 1910..1929 then 'New Orleans Jazz'
       when 1930..1939 then 'Swing'
       when 1940..1950 then 'Bebop'
       else 'Jazz'
       end

result = if some_cond
           calc_something
         else
           calc_something_else
         end

# 好 - 并且更好地利用行宽
kind =
  case year
  when 1850..1889 then 'Blues'
  when 1890..1909 then 'Ragtime'
  when 1910..1929 then 'New Orleans Jazz'
  when 1930..1939 then 'Swing'
  when 1940..1950 then 'Bebop'
  else 'Jazz'
  end

result =
  if some_cond
    calc_something
  else
    calc_something_else
  end
  
```

个人倾向结构清晰的规范

8. 在各个方法定义之间添加空行，并且将方法分成若干合乎逻辑的段落

```ruby

def some_method
  data = initialize(options)

  data.manipulate!

  data.result
end

def some_method
  result
end

```

9. 在不同缩进的代码之间，不要使用空行分隔

```ruby

# 差
class Foo

  def foo

    begin

      do_something do

        something

      end

    rescue

      something

    end

  end

end

# 好
class Foo
  def foo
    begin
      do_something do
        something
      end
    rescue
      something
    end
  end
end

```

个人感觉是不错的规范

10. 避免在方法调用的最后一个参数之后添加逗号，尤其当参数没有分布在同一行时

```ruby

# 差 - 尽管移动、新增、删除参数颇为方便，但仍不推荐这种写法
some_method(
  size,
  count,
  color,
)

# 差
some_method(size, count, color, )

# 好
some_method(size, count, color)

```



11. 当给方法的参数赋予默认值时，在 = 前后添加空格

```ruby

# 差
def some_method(arg1=:default, arg2=nil, arg3=[])
  # 做一些事情
end

# 好
def some_method(arg1 = :default, arg2 = nil, arg3 = [])
  # 做一些事情
end

```

 一直用着第一种 = =

12. 避免在非必要的情形下使用续行符 \。在实践中，除了字符串拼接，避免在其他任何地方使用续行

```ruby

# 差
result = 1 - \
         2

# 好 - 但仍然丑到爆
result = 1 \
         - 2

long_string = 'First part of the long string' \
              ' and second part of the long string'

```

13. 使用统一的风格进行多行链式方法调用。在 Ruby 社区中存在两种流行的风格：前置 .（风格 A）与后置 .（风格 B）

```ruby

# 风格A
# 差 - 需要查看第一行才能理解第二行在做什么
one.two.three.
  four
  
# 好 - 立刻能够明白第二行在做什么
one.two.three
  .four

# 风格B
# 差 - 需要查看第二行才能知道链式方法调用是否结束
one.two.three
  .four

# 好 - 立刻能够明白第二行还有其他方法调用
one.two.three.
  four

```

个人倾向风格A

14. 当方法调用参数过长时，将它们排列在多行并对齐。若对齐后长度超过行宽限制，将首个参数位置挪到下一行进行缩排也是可以接受的

```ruby

# 初始（行太长了）
def send_mail(source)
  Mailer.deliver(to: 'bob@example.com', from: 'us@example.com', subject: 'Important message', body: source.text)
end

# 差 - 双倍缩排
def send_mail(source)
  Mailer.deliver(
      to: 'bob@example.com',
      from: 'us@example.com',
      subject: 'Important message',
      body: source.text)
end

# 好
def send_mail(source)
  Mailer.deliver(to: 'bob@example.com',
                 from: 'us@example.com',
                 subject: 'Important message',
                 body: source.text)
end

# 好 - 普通缩排
def send_mail(source)
  Mailer.deliver(
    to: 'bob@example.com',
    from: 'us@example.com',
    subject: 'Important message',
    body: source.text
  )
end

```

倾向普通缩排的风格

15. 使用 _ 语法改善大数的数值字面量的可读性

```ruby

# 差 - 有几个零？
num = 1000000

# 好 - 方便人脑理解
num = 1_000_000

```

16. 不要在注释与 def 之间添加空行
17. 将单行长度控制在 80 个字符内
18. 不要使用区块注释。它们不能被空白字符引导，且不如常规注释容易辨认
19. 三元操作符的每个分支只写一个表达式。即不要嵌套三元操作符
20. 远不要使用 and 与 or 关键字。使用 && 与 || 来替代
21. 对于单行主体，倾向使用 if/unless 修饰语法。另一种方法是使用流程控制 && / ||

```ruby

# 差
if some_condition
  do_something
end

# 好
do_something if some_condition

# 好 - 使用流程控制
some_condition && do_something

```

平常用的都是第二种

22. 对于无限循环，使用 Kernel#loop 而不是 while/until

```ruby

# 差
while true
  do_something
end

until false
  do_something
end

# 好
loop do
  do_something
end

```

23. 阅读至 “优先考虑使用显式区块参数”
