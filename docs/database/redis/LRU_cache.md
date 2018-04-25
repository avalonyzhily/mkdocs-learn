### 使用redis实现缓存

- LRU(最长时间未使用)是redis唯一支持的缓存回收算法(3.0及以下)
  - redis针对LRU有几种回收策略
    - noeviction 缓存达到上限直接返回错误,将处理完全交给客户端
    - allkeys-lru/volatile-lru 针对所有键/过期键的lru回收
    - allkeys-random/volatile-random 针对所有键/过期键的随机回收
    - volatile-ttl 针对过期键中存活时间(time to live -> TTL)最短的键进行回收
  - 以上针对过期键的策略,可能会找不到满足条件的key,届时将于noeviction情况相同
- Redis4.0开始将支持LFU(最近最不常用)算法的缓存回收策略
  - allkeys-lfu/volatile-lfu 针对所有键/过期键的lfu回收