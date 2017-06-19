---
title: 对redis命令incr的并发探索 
date: 2016-05-20 14:57:56
tags:
    - python
    - redis
    - incr
    - 并发
    - 业务锁
categories: redis
---

## 引言
``` bash
    某个项目想利用redis实现业务锁,比如抽奖只允许被抽中一次, 那么在并发的情况下一定是要锁的,我们用django模
    拟每次去redis查询yunson对应的num完后，给num加1
```
<!--more-->
### 服务器配置
``` bash
双核，三个端口，nginx做代理（每个权重都为一样）
```
### 使用hset
#### redis数据结构
``` bash
    yunson---{'num':100}（json格式）
```
#### 代码
``` bash
    class MR(APIView):
        def get(self, request):
            REDIS_CONF = {
                'host': '127.0.0.1',
                'port': 6379,
                'db': 10,
                'password': '',
            }
            r = redis.Redis(connection_pool=redis.ConnectionPool(**REDIS_CONF))
            yunson = json.loads(r.hget('test', 'yunson'))
            num = yunson['num']
            yunson['num'] += 1
            yunson = json.dumps(yunson)
            r.hset('test', 'yunson', yunson)
            logger = Logger(str('JS_DJANGO'))
            logger.info(str(num))
            return Response(REDIS_CONF)
```
#### 测试
```bash
    ab -n 1200 -c 400 http://127.0.0.1/mr
```
#### 结果
```bash
    This is ApacheBench, Version 2.3 <$Revision: 655654 $>
    Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
    Licensed to The Apache Software Foundation, http://www.apache.org/

    Benchmarking 127.0.0.1 (be patient)
    Completed 120 requests
    Completed 240 requests
    Completed 360 requests
    Completed 480 requests
    Completed 600 requests
    Completed 720 requests
    Completed 840 requests
    Completed 960 requests
    Completed 1080 requests
    Completed 1200 requests
    Finished 1200 requests


    Document Path:          /mr
    Document Length:        54 bytes

    Concurrency Level:      400
    Time taken for tests:   17.807 seconds
    Complete requests:      1200
    Failed requests:        182
       (Connect: 0, Receive: 0, Length: 182, Exceptions: 0)
    Write errors:           0
    Non-2xx responses:      182
    Total transferred:      324848 bytes
    HTML transferred:       86458 bytes
    Requests per second:    67.39 [#/sec] (mean)
    Time per request:       5935.669 [ms] (mean)
    Time per request:       14.839 [ms] (mean, across all concurrent requests)
    Transfer rate:          17.82 [Kbytes/sec] received


    redis结果：
        {"num": 616}  差出：1200-182-616 = 402   
        打日志时：根据日志分析发现除服率相当高上大几十（当然没看全，一看这样头就大
        (你咋算的：
            Complete requests:      1200
            Failed requests:        182)
```
### 优化使用incr
#### redis数据结构
``` bash
    yunson---0
```
#### 代码
``` bash
    class MR(APIView):
        def get(self, request):
            REDIS_CONF = {
                'host': '127.0.0.1',
                'port': 6379,
                'db': 10,
                'password': '',
            }
            r = redis.Redis(connection_pool=redis.ConnectionPool(**REDIS_CONF))
            try:
                logger = Logger(str('JS_DJANGO'))
                if not r.get('yunson'):
                    yunson = r.incr('yunson')
                    r.set('yunson ', yunson , ex=60)
                else:
                    yunson= r.incr('baisong')
                logger.info(str(yunson))
            except Exception, e:
                print Exception, e
            return Response(REDIS_CONF)
```
#### 测试
``` bash
    ab -n 1200 -c 400 http://127.0.0.1/mr
```
#### 结果
``` bash
    This is ApacheBench, Version 2.3 <$Revision: 655654 $>
    Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
    Licensed to The Apache Software Foundation, http://www.apache.org/

    Benchmarking 127.0.0.1 (be patient)
    Completed 120 requests
    Completed 240 requests
    Completed 360 requests
    Completed 480 requests
    Completed 600 requests
    Completed 720 requests
    Completed 840 requests
    Completed 960 requests
    Completed 1080 requests
    Completed 1200 requests
    Finished 1200 requests


    Server Software:        nginx/1.1.19
    Server Hostname:        127.0.0.1
    Server Port:            80

    Document Path:          /mr
    Document Length:        54 bytes

    Concurrency Level:      400
    Time taken for tests:   17.311 seconds
    Complete requests:      1200
    Failed requests:        0
    Write errors:           0
    Total transferred:      313200 bytes
    HTML transferred:       64800 bytes
    Requests per second:    69.32 [#/sec] (mean)
    Time per request:       5770.450 [ms] (mean)
    Time per request:       14.426 [ms] (mean, across all concurrent requests)
    Transfer rate:          17.67 [Kbytes/sec] received
    
    redis结果：
        yunson---1200
        分毫不差
```
### 总结
``` bash
    在业务逻辑控制锁的时候，尤其是在并发的时候可以考虑利用redis的这个机制，当然用数据库
    的锁也可以（同时数据库的锁更为灵活），但是如果只是类似于这种计数机制，可以尝试用redis
    毕竟redis是操作内存，速度上要快些
```