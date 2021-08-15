
## 前言

参考的 [Sidekiq 的源码分析](https://ruby-china.org/topics/31470) 对应版本是 4.2.3，此篇笔记主要是与当前版本 6.0.2 比较不同以及补充

## 

我所认为的不同之处：

1、 启动 `bin/sidekiq` ，执行了 `#integrate_with_systemd` 

在该方法中引入了 `sidekiq/sd_notify` ，通过文档注释，我们可以暂时了解到该模块主要是用于通知 systemd 状态变化，进程间的通信是通过 socket 传递，感兴趣的同学可以阅读下  [源码](https://github.com/mperham/sidekiq/blob/13e2b564c8ab9275de910a5b60cf12412062d4e5/lib/sidekiq/sd_notify.rb#L39) (纯 ruby 实现)

``` ruby

begin
  cli = Sidekiq::CLI.instance
  cli.parse # 解析 sidekiq 的配置参数，其中包括队列的配置、worker 数量的配置等

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



