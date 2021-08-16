## 前言

继第一篇 [Sidekiq 浅析（一）](https://ruby-china.org/topics/41582) ，准备展开 `Poller` 、 `Manager` 及 `Processor` 的分析，这篇更像是我的学习过程。Sidekiq 的核心部分 ， Martin 大佬讲得会更清楚易懂，感兴趣得可跳转至 [Sidekiq 任务调度流程分析](https://ruby-china.org/topics/31470) 

为了方便理解，挂上时序图

![image](https://user-images.githubusercontent.com/83901620/129534361-12ae23c2-8c5b-46cd-81a3-3cb83bf5c622.png)

## 继续分析 `Sidekiq::Launcher#run`

```ruby
# lib/sidekiq/launcher.rb


def run
  @thread = safe_thread("heartbeat", &method(:start_heartbeat)) # 监测 Sidekiq 心跳
  @poller.start
  @manager.start
end

```

第一步，执行 `start_heartbeat` 方法，专门创建了线程用于监测 Sidekiq 心跳，截取了部分代码，感兴趣得可以来看看~

```ruby
# lib/sidekiq/launcher.rb


def start_heartbeat
  loop do
    heartbeat
    sleep 5
  end
  Sidekiq.logger.info("Heartbeat stopping...")
end

```

> 习惯性放上片段有意思的代码

```ruby

def heartbeat
  $0 = PROCTITLES.map { |proc| proc.call(self, to_data) }.compact.join(" ")

  ❤
end

def ❤
# ..
end


```

---

第二步，执行 `Sidekiq::Scheduled::Poller#start` , Poller 类充当的更像是轮询器，定时得从 Redis 中检查是否有 重复(retry) 或 延时(scheduled) 的任务

```ruby
# lib/sidekiq/scheduled.rb


def start
  @thread ||= safe_thread("scheduler") {
    initial_wait # 避免同时 IO 操作

    until @done
      enqueue
      wait
    end
    Sidekiq.logger.info("Scheduler exiting...")
  }
end

def initial_wait
  total = 0
  total += INITIAL_WAIT unless Sidekiq.options[:poll_interval_average]
  total += (5 * rand)

  @sleeper.pop(total)
rescue Timeout::Error
end

def enqueue
  @enq.enqueue_jobs
  # ..
end

class Enq
  def enqueue_jobs(now = Time.now.to_f.to_s, sorted_sets = SETS)
    Sidekiq.redis do |conn|
      sorted_sets.each do |sorted_set
        until (jobs = conn.zrangebyscore(sorted_set, "-inf", now, limit: [0, 100])).empty?
          jobs.each do |job|
            if conn.zrem(sorted_set, job) # 集合中移除
              Sidekiq::Client.push(Sidekiq.load_json(job)) # 用于将作业推送到 Redis
              Sidekiq.logger.debug { "enqueued #{sorted_set}: #{job}" }
            end
          end
        end
      end
    end
  end
end
```

主要分为两步：初始化等待、检查重试以及定时任务队列并按照执行时间 score 排序

初始化等待：值得注意的细节之一，通过加入随机时间错开  Redis IO 操作

检查并推入 redis ：采取的策略，遍历获取前100个任务（retry 重试 跟 schedule 延时 的集合）推入到相关的队列等待拉取 

---

第三步，执行 `Sidekiq::Manager#start` ， Manager 负责按照 concurrency 数量配置 worker ，并加以管理

```ruby
# lib/sidekiq/manager.rb

require "sidekiq/processor"

def initialize(options = {})
  logger.debug { options.inspect }
  @options = options
  @count = options[:concurrency] || 10
  raise ArgumentError, "Concurrency of #{@count} is not supported" if @count < 1

  @done = false
  @workers = Set.new
  @count.times do
    @workers << Processor.new(self, options) 
  end
  @plock = Mutex.new
end

def start
  @workers.each do |x|
    x.start
  end
end

```

生成了多个 `Sidekiq::Processor` 实例，并执行了 `#run`

```ruby
# lib/sidekiq/processor.rb


def run
  process_one until @done
  
  # ..
end

def process_one
  @job = fetch # 获取任务
  process(@job) if @job # 处理任务
  @job = nil
end

```

 `#fetch` 的作用是获取任务，再将其作为参数执行 `#process` 完成对任务的处理，否则就重新获取新的任务
 
 先来看看 `#fetch`
 
 ```ruby
 # lib/sidekiq/processor.rb
 
 
 def get_one
  work = @strategy.retrieve_work
  # ..
  
  work
  
  #..
end

def fetch
  j = get_one
  if j && @done
    j.requeue
    nil
  else
    j
  end
end
 
 ```
 
 



