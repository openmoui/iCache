# ICache
分布式缓存系统

From https://geektutu.com/post/geecache-day1.html

### 简介

1. 采用LRU 缓存淘汰算法。
2. 支持高防缓存击穿
3. 设置ttl和惰性删除
4. 增加了grpc进行通信
5. 使用etcd做服务注册和服务发现

创建一个缓存Group命名为 `MyCache`, 缓存大小2字节, 如果缓存不存在, 在`geecache.GetterFunc()`中定义如何获取待缓存的数据，这里就直接在字典中获取
``` go
func createGroup() *geecache.Group {
	return geecache.NewGroup("scores", 2<<10, "lru", geecache.GetterFunc( //lru算法做测试
		func(key string) ([]byte, error) {
			log.Println("[SlowDB] Search key", key)
			if v, ok := db[key]; ok {
				return []byte(v), nil
			}
			return nil, fmt.Errorf("%s not exist", key)
		}))
}
```

缓存结果获取
``` bash
# http://localhost:9999/api?key=Tom
```
如果缓存中有就直接返回,如果缓存中没有。从slow db中取，如果slow db没有就返回空。

