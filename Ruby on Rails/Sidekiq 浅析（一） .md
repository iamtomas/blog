本着提高阅读源码能力和欣赏大佬的代码，打算在此分享处女篇 `Sidekiq 浅析（一）` ，希望大佬能指出不足之处


## 前言

参考的 Martin 大佬的文章 [Sidekiq 的源码分析](https://ruby-china.org/topics/31470) ，其对应版本是 4.2.3，此篇笔记主要是与分析当前版本 6.0.2 并比较不同之处及疑问

通过时序图理清关系的习惯，get ✅  （后续分析完附上自己理解的时序图）

![image](https://user-images.githubusercontent.com/83901620/129534361-12ae23c2-8c5b-46cd-81a3-3cb83bf5c622.png)


## 不同之处及疑问

1、 启动 `bin/sidekiq` ，另外执行了 `#integrate_with_systemd` 

在该方法中引入了 `sidekiq/sd_notify` ，通过文档注释，我们可以暂时了解到该模块主要是用于通知 systemd 状态变化，其中进程间的通信是通过 socket 传递，感兴趣的同学可以阅读下  [源码](https://github.com/mperham/sidekiq/blob/13e2b564c8ab9275de910a5b60cf12412062d4e5/lib/sidekiq/sd_notify.rb#L39) (纯 ruby 实现)

``` ruby
# bin/sidekiq


begin
  cli = Sidekiq::CLI.instance
  cli.parse # 解析 sidekiq 的配置参数，其中包括 :concurrency、:queue 等、初始化日志及校验参数

  integrate_with_systemd

  cli.run # 继续下一步
rescue => e

# ..

# integrate_with_systemd

Sidekiq.configure_server do |config|
  Sidekiq.logger.info "Enabling systemd notification integration"
  require "sidekiq/sd_notify"
  config.on(:startup) do
    Sidekiq::SdNotify.ready
  end
  config.on(:shutdown) do
    Sidekiq::SdNotify.stopping
  end
  Sidekiq.start_watchdog if Sidekiq::SdNotify.watchdog?
end

```

> 岔个题外话，感觉 Sidekiq 的作者是个挺有性格的人，包括他的代码 0.0 ，大家也可以分享下平时写了啥骚代码

``` ruby

  # lib/sidekiq/cli.rb
  

  def self.banner
    %{
    #{w}         m,
    #{w}         `$b
    #{w}    .ss,  $$:         .,d$
    #{w}    `$$P,d$P'    .,md$P"'
    #{w}     ,$$$$$b#{b}/#{w}md$$$P^'
    #{w}   .d$$$$$$#{b}/#{w}$$$P'
    #{w}   $$^' `"#{b}/#{w}$$$'       #{r}____  _     _      _    _
    #{w}   $:     ,$$:      #{r} / ___|(_) __| | ___| | _(_) __ _
    #{w}   `b     :$$       #{r} \\___ \\| |/ _` |/ _ \\ |/ / |/ _` |
    #{w}          $$:        #{r} ___) | | (_| |  __/   <| | (_| |
    #{w}          $$         #{r}|____/|_|\\__,_|\\___|_|\\_\\_|\\__, |
    #{w}        .d$$          #{r}                             |_|
    #{reset}}
  end
  
  alias_method :☠, :exit
  
  # ..

```

2、 创建 pipe 并启动信号的捕获

疑问一：Sidekiq 的信号量机制是怎么发送、捕获，到底是通过 pipe 还是 socket（前边 systemd 状态变化是通过 socket ） 来实现进程的通信（原谅我操作系统菜 = =） ，这一过程有没大佬能简单明了得描述下？

疑问二：Redis 线程池数量 < concurrency + 2 性能会显著降低，那么 concurrency 在各个环境下应该设置为多少？

疑问三：在哪、又是如何控制进程的？

```ruby

# lib/sidekiq/cli.rb


def run

# ..

# 疑问一！！
self_read, self_write = IO.pipe
sigs = %w[INT TERM TTIN TSTP]
# USR1 and USR2 don't work on the JVM
sigs << "USR2" if Sidekiq.pro? && !jruby?
sigs.each do |sig|
  trap sig do
    self_write.puts(sig)
  end
rescue ArgumentError
  puts "Signal #{sig} not supported"
end

# ..

# Redis 版本检查，需要 v4.0.0 或更高版本

# 内存策略 检查是否为 “noeviction”，若不是，则 Redis 可能会清除 Sidekiq 的数据，具体可以查看：https://github.com/mperham/sidekiq/wiki/Using-Redis#memory

# 疑问二！！
cursize = Sidekiq.redis_pool.size
needed = Sidekiq.options[:concurrency] + 2
raise "Your pool of #{cursize} Redis connections is too small, please increase the size to at least #{needed}" if cursize < needed

# 用于及时捕捉失败的任务，针对允许再次重试的任务，按失败次数计算新的重试时间
Sidekiq.server_middleware

# 疑问三！！
# 在这之前，进程仅使用主线程进行初始化
# 在这之后，将有多个进程执行
fire_event(:startup, reverse: false, reraise: true)

launch(self_read) # 继续下一步

end

def launch(self_read)
  # ..
  
  begin
    launcher.run 

    # 进程接收到的信号处理以及退出处理
    while (readable_io = IO.select([self_read])) 
      signal = readable_io.first[0].gets.strip
      handle_signal(signalsignal)
    end
  rescue Interrupt
    logger.info "Shutting down"
    launcher.stop
    logger.info "Bye!"
    # ..
    exit(0)
  end
end


```

> 又到了骚代码环节

```ruby

def self.❨╯°□°❩╯︵┻━┻
  puts "Calm down, yo."
end

```

到这为止讲述了真正启动前的准备动作，后面继续分析 `Poller` 、 `Manager` 及 `Processor`

--- 















