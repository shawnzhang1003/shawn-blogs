# redis基础

## redis客户端初始化
```go
rdb := redis.NewClient(&redis.Options{
	Addr: "localhost:6379",
	DB:   0,
})
```

## set操作，设置kv键值对
1. 基本键值对插入
   
   rdb.Set(ctx, "user:1000", "John Doe", 0) //这里的0指不设置超时删除时间
  
2. 带过期时间的键值存储
   
   rdb.Set(ctx, "session:12345", "session_data", 60*time.Second)

3. 仅当键不存在时设置值

   rdb.SetNX(ctx, "lock:user:1000", "1", 10*time.Second)

4. 仅当键已存在时设置值

   rdb.SetXX(ctx, "config:version", "v2.0", 0)

5. 原子性的获取并设置值

   rdb.GetSet(ctx, "counter", "0")
## 电商场景下的redis应用
在电商场景中，Redis 被广泛应用于多种场景，主要利用其高性能和丰富的数据结构特性来提高系统的响应速度和并发处理能力。以下是一些常见的 Redis 应用场景：

### 1. **缓存**

- **商品详情页缓存**：将频繁访问的商品详情数据存入 Redis，以减少对数据库的直接读操作，加快页面加载速度。
- **用户信息缓存**：将用户的登录信息、购物车内容等存储在 Redis 中，以便迅速读取和修改。

```go
func getProductDetails(productID string) (Product, error) {
    // 尝试从 Redis 缓存中获取商品详情
    productJSON, err := rdb.Get(ctx, "product:"+productID).Result()
    if err == nil {
        var product Product
        json.Unmarshal([]byte(productJSON), &product)
        return product, nil
    }

    // 如果缓存未命中，则从数据库获取商品详情
    product, err := getProductFromDB(productID)
    if err != nil {
        return Product{}, err
    }

    // 将商品详情存入 Redis 缓存
    productJSON, _ = json.Marshal(product)
    rdb.Set(ctx, "product:"+productID, productJSON, 10*time.Minute)

    return product, nil
}
```

### 2. **会话管理**

使用 Redis 存储用户会话（Session），实现分布式会话管理，使得用户可以在负载均衡的多个服务器之间无缝切换。

```go
func saveSession(sessionID string, userInfo User, ttl time.Duration) error {
    userJSON, err := json.Marshal(userInfo)
    if err != nil {
        return err
    }
    err = rdb.Set(ctx, "session:"+sessionID, userJSON, ttl).Err()
    return err
}

func getSession(sessionID string) (User, error) {
    userJSON, err := rdb.Get(ctx, "session:"+sessionID).Result()
    if err != nil {
        return User{}, err
    }
    var user User
    json.Unmarshal([]byte(userJSON), &user)
    return user, nil
}
```

### 3. **购物车**

将用户的购物车内容存储在 Redis 中，可以提供快速读写操作，并且可以轻松实现购物车过期机制。

```go
func addItemToCart(userID string, item CartItem) error {
    cartKey := "cart:" + userID
    itemJSON, _ := json.Marshal(item)
    err := rdb.HSet(ctx, cartKey, item.ProductID, itemJSON).Err()
    if err == nil {
        rdb.Expire(ctx, cartKey, 24*time.Hour) // 设置购物车过期时间
    }
    return err
}

func getCart(userID string) (map[string]CartItem, error) {
    cartKey := "cart:" + userID
    itemsMap, err := rdb.HGetAll(ctx, cartKey).Result()
    if err != nil {
        return nil, err
    }
    cart := make(map[string]CartItem)
    for productID, itemJSON := range itemsMap {
        var item CartItem
        json.Unmarshal([]byte(itemJSON), &item)
        cart[productID] = item
    }
    return cart, nil
}
```

### 4. **限流与防刷**

使用 Redis 实现接口请求限流和防止恶意刷单，例如通过计数器和滑动窗口算法进行请求频率控制。

```go
func rateLimiter(userID string, limit int, window time.Duration) bool {
    key := "rate_limiter:" + userID
    currentCount, err := rdb.Get(ctx, key).Int()
    if err == redis.Nil {
        rdb.Set(ctx, key, 1, window).Err() // 第一次请求，设置过期时间
        return true
    }
    if currentCount < limit {
        rdb.Incr(ctx, key).Err()
        return true
    }
    return false
}
```

### 5. **库存管理**

通过 Redis 的原子操作和 Lua 脚本来避免超卖问题，实现高效的库存扣减。

```go
func decrementStock(productID string, quantity int) (bool, error) {
    luaScript := `
        local stock = redis.call("GET", KEYS[1])
        if not stock then
            return 0
        end
        stock = tonumber(stock)
        if stock < tonumber(ARGV[1]) then
            return 0
        end
        redis.call("DECRBY", KEYS[1], ARGV[1])
        return 1
    `
    res, err := rdb.Eval(ctx, luaScript, []string{"stock:" + productID}, quantity).Int()
    if err != nil {
        return false, err
    }
    return res == 1, nil
}
```

### 6. **排行榜**

使用 Redis 的有序集合（Sorted Set）来实现实时更新的排行榜功能，例如销售排行榜、浏览量排行榜等。

```go
func incrementProductSales(productID string, increment float64) error {
    return rdb.ZIncrBy(ctx, "sales_rank", increment, productID).Err()
}

func getTopProducts(n int) ([]string, error) {
    return rdb.ZRevRange(ctx, "sales_rank", 0, int64(n-1)).Result()
}
```

### 7. **消息队列**

使用 Redis 的 List 数据结构或 Pub/Sub 功能来实现简单的消息队列，用于订单处理、通知推送等场景。

```go
func enqueueOrder(order Order) error {
    orderJSON, _ := json.Marshal(order)
    return rdb.LPush(ctx, "order_queue", orderJSON).Err()
}

func dequeueOrder() (Order, error) {
    orderJSON, err := rdb.RPop(ctx, "order_queue").Result()
    if err != nil {
        return Order{}, err
    }
    var order Order
    json.Unmarshal([]byte(orderJSON), &order)
    return order, nil
}
```

### 8. **推荐系统**

利用 Redis 存储用户行为数据，如浏览历史、购买记录等，通过实时计算或定期离线计算生成个性化推荐。

```go
func recordUserAction(userID, productID string) error {
    actionKey := "user_action:" + userID
    return rdb.ZAdd(ctx, actionKey, &redis.Z{
        Score:  float64(time.Now().Unix()),
        Member: productID,
    }).Err()
}

func getUserRecommendations(userID string, n int) ([]string, error) {
    actionKey := "user_action:" + userID
    return rdb.ZRevRange(ctx, actionKey, 0, int64(n-1)).Result()
}
```

### 总结

以上只是 Redis 在电商场景中的部分应用实例。由于 Redis 具备高性能、支持多种数据结构、持久化等特性，它在电商系统中还有更多潜在应用场景，例如分布式锁、地理位置查询等。根据具体业务需求，合理设计和使用 Redis，可以大幅提升系统的性能和用户体验。


