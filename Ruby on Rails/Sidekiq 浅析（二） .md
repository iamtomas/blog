## 前言

继第一篇 [Sidekiq 浅析（一）](https://ruby-china.org/topics/41582) ，准备展开 `Poller` 、 `Manager` 及 `Processor` 的分析，在版本上源码并没发现太多不同，这篇更像是记录学习的过程。Sidekiq 的核心部分 ， Martin 大佬讲得会更清楚易懂，感兴趣得可跳转至 [Sidekiq 任务调度流程分析](https://ruby-china.org/topics/31470) 

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

> 习惯性放上有意思的代码

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

主要分为两步：初始化等待、检查重试以及定时任务队列并按照执行时间 score 排序推送到 Redis

初始化等待：值得注意的细节之一，通过加入随机时间错开  Redis IO 操作

检查并推入 redis ：采取的策略，遍历获取前100个任务（retry 重试 跟 schedule 延时 的集合）推入到相关的队列等待拉取 

---

第三步，执行 `Sidekiq::Manager#start` ， Manager 充当的是负责按照 concurrency 数量配置 worker ，并加以管理的角色

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

生成了多个 `Sidekiq::Processor` 实例，并执行了 `#run` ，Processor 

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
 
 可以看出先通过 `@strategy.retrieve_work` 获取任务，若没获取到或 worker 停止了，则将任务重新放到队列
 
 到这，因为调用层特别多，我拾取了几个我感兴趣的点，其他的感兴趣可以继续阅读源码~
 
 1. 在处理 `Sidekiq::Processor#process_one` 的 `process(@job)` 时，跟踪源码到最后是 `#perform` ，也就是我们最后定义任务时传进来的代码块
 2. 针对错误和异常，最好都阅读阅读 https://github.com/mperham/sidekiq/wiki/Error-Handling ，看完有所心得，比如在 `通过 Gems（比如 Sentry） 附加到 Sidekiq 的全局错误处理程序的方式，以便在 Sidekiq 内部出现错误时随时通知他们`
 3. 发现了两处不同，其一是重试机制 （咳咳，阅读到这，发觉对 `server_middleware` 的理解有误，目前暂未真正理解它真正的作用，对前篇已做修正！）
 
 ```ruby
 # v4.2.3
 # https://github.com/mperham/sidekiq/blob/5ebd857e3020d55f5c701037c2d7bedf9a18e897/lib/sidekiq/processor.rb
 
 def process(work)
    # ..
  
    stats(worker, job, queue) do
      Sidekiq.server_middleware.invoke(worker, job, queue) do # 在 `server_middleware` 进行了重试处理，具体的分析可以查看 Martin 大佬的
        # ..
        execute_job(worker, cloned(job['args'.freeze]))
      end
    end
  
    # ..
  end
  
  
  # v6.0.2
  # https://github.com/mperham/sidekiq/blob/13e2b564c8ab9275de910a5b60cf12412062d4e5/lib/sidekiq/processor.rb
  
  def process(work)
    # ..
    
    dispatch(job_hash, queue, jobstr) do |worker| # 重试处理放在了 `#dispatch` 中
      Sidekiq.server_middleware.invoke(worker, job_hash, queue) do
        execute_job(worker, job_hash["args"])
      end
    end
    
    # ..
  end
 
  def dispatch(job_hash, queue, jobstr)
    # since middleware can mutate the job hash
    # we need to clone it to report the original
    # job structure to the Web UI
    # or to push back to redis when retrying.
    # To avoid costly and, most of the time, useless cloning here,
    # we pass original String of JSON to respected methods
    # to re-parse it there if we need access to the original, untouched job

    @job_logger.prepare(job_hash) do
      @retrier.global(jobstr, queue) do
        @job_logger.call(job_hash, queue) do
          stats(jobstr, queue) do
            # Rails 5 requires a Reloader to wrap code execution.  In order to
            # constantize the worker and instantiate an instance, we have to call
            # the Reloader.  It handles code loading, db connection management, etc.
            # Effectively this block denotes a "unit of work" to Rails.
            @reloader.call do
              klass = constantize(job_hash["class"])
              worker = klass.new
              worker.jid = job_hash["jid"]
              @retrier.local(worker, jobstr, queue) do
                yield worker
              end
            end
          end
        end
      end
    end
  end
 
 
 ```
 
 可以看到官方解释：因为中间件可以修改 Job Hash ，所以采用 clone 的方式将 Job 上报给 Web UI 或者重试时推回 Redis 
 
 所以到这里，假设 server_middleware 的作用如 Martin 理解的那出（通过中间件，及时捕捉失败的任务，针对允许再次重试的任务，按失败次数计算新的重试时间），那么新版本其实就是针对这个 bug （中间件可以修改 Job Hash） ，做了替换的方式
 
 4. 其二是下次重试时间间隔的计算调整

```ruby
# v4.2.3
# https://github.com/mperham/sidekiq/blob/5ebd857e3020d55f5c701037c2d7bedf9a18e897/lib/sidekiq/middleware/server/retry_jobs.rb#L177-L179

def seconds_to_delay(count)
  (count ** 4) + 15 + (rand(30)*(count+1))
end


# v6.0.2
# https://github.com/mperham/sidekiq/blob/13e2b564c8ab9275de910a5b60cf12412062d4e5/lib/sidekiq/job_retry.rb#L225

def delay_for(worker, count, exception)
  jitter = rand(10) * (count + 1)
  if worker&.sidekiq_retry_in_block
    custom_retry_in = retry_in(worker, count, exception).to_i
    return custom_retry_in + jitter if custom_retry_in > 0
  end
  (count**4) + 15 + jitter
end

```
 
 最后想总结的是一个值得去实践摸索的尝试，先拿出 Martin 的实践：
 
 > 缺省情况下，一个首次失败任务下次重回队列（不是执行）的理论最大时间间隔大概是 67.5 秒！（固定的 15 秒 + 最大随机时间 30 秒 + 最大理论检查时间 22.5 秒）。所以，如果你的任务很重要，又需要尽快重试，就需要对几部分时间的相关配置参数进行调优了哦！在我自己的工作中，我针对某个队列任务设置的 sidekiq_retry_in 公式为线性时间，即 1s、2s、...50s，然后在重试检查那里设置了 :poll_interval_average 为 5 秒，新的下次执行时间理论最大时间间隔就是 8.5 秒！不过这些配置需要慎重调整，综合考虑业务以及业务量，既要尽可能保证任务尽早处理完，又得保证 Redis 没被 IO 压垮。

注：最大时间间隔最好再重新计算！

在对时间的相关配置参数调优的过程，最好结合 Sentry 等（Sidekiq 内部出现错误时随时通知）错误服务，避免出现诡异报错导致任务丢失~
 
 
 
 



