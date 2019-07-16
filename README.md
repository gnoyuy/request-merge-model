# request-merge-model
## 请求合并模型设计方案

### 需求背景
用户A请求 接口 /user?name=A 查询用户A信息
用户B请求 接口 /user?name=B 查询用户B信息
后端的资源被查询了两次
有没有一种请求模型，可以让这两个请求，只查询一次资源呢

### 方案
服务端在一个小片段时间内先后 或者 并发 收到这两个请求;
服务端方面，我们把请求按时间线分成连续的n个小片段（例如 10ms- 以ms为最小时间单元)
假如服务端在 第一个小片段1 他的绝对时间是 [0,10) 半开区间 ms这段时间

这时候 用户A和 用户B的请求先后到达，或者同时到达 （例如 A请求在时间点 0ms 点到，b请求在片段的最后一ms 9ms 到达）
服务端不会处理任何请求，用户A先到让它wait; 用户B后到，也wait; 直到片段时间（10ms）时间到期了，也是就是第10ms时，才开始处理
这时候的服务端是已经拿到了两个请求的

服务端不再傻帽似的按原来的计划，转发A请求 /user?name=A， 转 B请求 /user?name=B(这样等待的时间是不是白费了吗，虽然是只有10ms)
服务端可以大有作为啦，
服务端不再请求 /user 接口了 而是去请求 /user/batch?names=A,B 接口 一次查询两个用户信息. 当/user/batch 接口把数据返回给我们的时候
服务端把再 用户A的数据 给A的请求链接， B的数据给B的请求链接（A，B的请求链接还在那里wait）
就这样，所有请求，拿到了自己想要的数据
但是我们只查询了一次后端的资源（就是那次批量的查询）

### todo
`` nginx模块实现以上功能 ``
`` apache模块实现以上功能 ``
