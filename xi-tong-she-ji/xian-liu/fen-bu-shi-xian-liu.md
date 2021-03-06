# 分布式限流

分布式限流最关键的是要将限流服务做成原子化，而解决方案可以使用Redis+Lua或者Nginx+Lua技术来实现。

## 限制某个接口的时间窗请求数

### Redis+Lua实现

我们需要编写一个Lua脚本：

```lua
-- 限流脚本
local key = KEY[1] -- 限流KEY（1秒1个）
local limite = tonumber(ARGV[1]) -- 限流大小
local current = tonumber(redis.call("INCRBY",key, "1")) -- 请求数加1
if current > limite then 
    return 0
elseif current == 1 then
    redis.call("EXPIRE", key, "2")
end
return 1
```

> 脚本说明（Lua小白必备）
>
> ①在Lua脚本中执行Redis命令：redis.call\(\)
>
> ②`tonumber`是Lua的一个函数，这个函数会尝试将它的参数转换为数字
>
> ③ARGV\[1\]表示传递给脚本的第一个参数的值

因为如上操作是在一个Lua脚本中，且Redis是单线程模型，因此是线程安全的。 但是上面的脚本有一个问题：当达到限流大小后还是会递增，可以进行如下改造：

```java
-- 限流脚本

local key = KEY[1] -- 限流KEY（1秒1个）
local limite = tonumber(ARGV[1]) -- 限流大小
local current = tonumber(redis.call("GET",key) or "0") -- 之前请求数
if current + 1 > limite then
    return 0
else -- 请求数加1，并设置2秒过期
    redis.call("INCRBY", key, "1")
    redis.call("EXPIRE", key, "2")
    return 1
end
```

 写好lua脚本之后，在Java代码中判断是否需要限流：

```java
public boolean permit() throws IOException {
    String luaScript = Files.toString(new File("limit_v2.lua"), Charset.defaultCharset());
    String key = "ip:" + System.currentTimeMillis() / 1000;
    String limit = "3";
    Long res = (Long) new Jedis().eval(luaScript, Lists.newArrayList(key), Lists.newArrayList(limit));
    return res == 1;
}
```

> 从 Redis 2.6.0 版本开始，通过内置的 Lua 解释器，可以使用 [EVAL](http://redisdoc.com/script/eval.html#eval) 命令对 Lua 脚本进行求值。

因为Redis的限制，不能在Redis Lua中使用TIME获取时间戳，因此只能通过应用传入。在某些情况下（机器时钟不准），限流会存在一些问题。

仔细阅读上述方案，我们不难发现，其实这就是应用级限流中“限制某个接口的时间窗请求数”的翻版，这里将之前的代码贴出来：

```text
public class PeriodCounter {

    /**
     * 每秒限制的请求数
     */
    private final long limit;

    /**
     * 键：当前时间，秒
     * 值：该秒内的累计请求量
     */
    LoadingCache<Long, AtomicLong> secondCounter = CacheBuilder.newBuilder()
            //写入2秒后删除
            .expireAfterWrite(2, TimeUnit.MINUTES)
            .build(new CacheLoader<Long, AtomicLong>() {
                @Override
                public AtomicLong load(Long aLong) throws Exception {
                    //重新获取初始值为0
                    return new AtomicLong(0);
                }
            });

    public PeriodCounter(long limit){
        this.limit = limit;
    }

    public boolean permit() throws ExecutionException {
        while (true){
            //获取当前秒
            long currentSecond = System.currentTimeMillis() / 1000;
            System.out.println(currentSecond);
            if(secondCounter.get(currentSecond).incrementAndGet() > limit){
                return false;
            }else {
                return true;
            }
        }
    }
}
```

 Redis+Lua方案中，Redis其实充当的是缓存角色，Lua脚本只是为了保证判断流程的原子性，与应用内的“限制某个接口的时间窗请求数”的思路是一致的。

### Nginx+Lua实现

```lua
-- Lua限流脚本:::Nginx版本
local locks = require "resty.lock"
local function permit()
    local lock = locks:new("locks")
    local elapsed, err = lock:lock("limit_key") -- 互斥锁
    local limit_counter = ngx.shared.limit_counter -- 计数器
    local key = "ip:" .. os.time()
    local limit = 5 -- 限流大小
    local current = limit_counter:get(key)

    if current == nil then
        limit_counter:set(key, 1, 1) -- 允许访问，第一次需要设置过期时间
        lock:unlock()
        return 1
    elseif current + 1 > limit then -- 超出限流大小
        lock:unlock()
        return 0
    else -- 允许访问
        limit_counter:incr(key, 1)
        lock:unlock()
        return 1
    end
end
ngx.print(permit())
```

 在上面的脚本中，我们使用了`lua-resty-lock`互斥锁模块来解决原子问题，并使用`ngx.shared.DICT`共享字典来实现计数器，所以需要在`Nginx`的配置中先定义两个共享字典（分别用来存放锁和计数器）：

```text
http{
    ... ...
    lua_shared_dict locks 10m;
    lua_shared_dict limit_counter 10m;
}
```

## 总结

有人会纠结：如果应用并发量非常大，Redis或者Nginx是否能扛得住？这个问题要从多方面来考虑：流量是不是真的有这么大，是不是当并发量太大时降级为应用级限流。京东目前的抢购业务就是使用Redis+Lua来限流的，详见[京东抢购服务高并发实践](https://mp.weixin.qq.com/s/40GHwueY8T3ji3DZ8yoxhQ)。

对于分布式限流，一般都是业务场景需要这种形式的限流；而流量入口的限流则应该在接入层来完成。

## 内容来源：

《亿级流量网站架构核心技术》：限流详解



