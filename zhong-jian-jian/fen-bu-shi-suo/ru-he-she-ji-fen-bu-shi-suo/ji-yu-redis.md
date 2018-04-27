# 基于Redis

## 思路

①获取锁：从客户端向redis中`set（lockKey, value, "NX", "PX", expireTime）`，键为`lockKey`如果设置成功，说明其他客户端没有设置过该`lockKey`，获取锁成功；如果失败，说明该`lockKey`已经被其他客户端设置（即锁被其他客户端获取到）；

②解锁：首先获取锁对应的value值，检查是否与requestId相等（保证谁加锁谁解锁），如果相等则删除锁（解锁）。

③死锁的解决：死锁发生的情况是，某客户端获取到了锁，但与redis失去了连接，此时，其他客户端因为之前获取到锁的客户端失去连接而无法解锁，所以就无法获取到锁，造成死锁。解决的办法是，在set的时候，设置超时时间：即第①中的`set（lockKey, value, "NX", "PX", expireTime）`中最后一个参数`expireTime`。

## 参考实现

```text
public class SimpleRedisDistributedLock implements Lock {

    private Jedis jedis;
    private String lockKey;
    private int expireTime;//锁的过期时间，防止客户端挂掉造成死锁，单位：毫秒
    private String value = UUID.randomUUID().toString();
    private static final String OPT_SUCCESS = "OK";

    public SimpleRedisDistributedLock(String localhost, Integer port, String lockKey, int expireTime) {
        //建立redis连接
        this.jedis = new Jedis(localhost, port);
        this.lockKey = lockKey;
        this.expireTime = expireTime;
    }


    @Override
    public void lock() {
        boolean success = false;
        while (!success) {
            String result = jedis.set(lockKey, value, "NX", "PX", expireTime);
            if (OPT_SUCCESS.equals(result)) {
                success = true;
            }
        }
    }

    @Override
    public void lockInterruptibly() throws InterruptedException {

    }

    @Override
    public boolean tryLock() {
        String result = jedis.set(lockKey, value, "NX", "PX", expireTime);
        return OPT_SUCCESS.equals(result);
    }

    @Override
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
        long curTime = System.currentTimeMillis();
        long waitTime = TimeUnit.MILLISECONDS.convert(time, unit);
        while (System.currentTimeMillis() - curTime <= waitTime) {
            String result = jedis.set(lockKey, value, "NX", "PX", expireTime);
            if (OPT_SUCCESS.equals(result)) {
                return true;
            }
        }
        return false;
    }

    @Override
    public void unlock() {
        String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
        jedis.eval(script, Collections.singletonList(lockKey), Collections.singletonList(value));
        jedis.close();
    }

    @Override
    public Condition newCondition() {
        return null;
    }


}
```

