    1 slowlog是什么
    redis的slowlog是redis用于记录记录慢查询执行时间的日志系统。由于slowlog只保存在内存中，因此slowlog的效率很高，完全不用担心会影响到redis的性能。Slowlog是Redis从2.2.12版本引入的一条命令。


    2 slowlog设置
    参考 http://redis.readthedocs.org/en/latest/server/slowlog.html 
    slowlog有两种设置方式：


    2.1 redis.conf设置
    在redis.conf中有关于slowlog的设置：

    1
    2
    slowlog-log-slower-than 10000
    slowlog-max-len 128


    其中slowlog-log-slower-than表示slowlog的划定界限，只有query执行时间大于slowlog-log-slower-than的才会定义成慢查询，才会被slowlog进行记录。slowlog-log-slower-than设置的单位是微妙，默认是10000微妙，也就是10ms 
    slowlog-max-len表示慢查询最大的条数，当slowlog超过设定的最大值后，会将最早的slowlog删除，是个FIFO队列


    2.2 使用config方式动态设置slowlog
    如下，可以通过config方式动态设置slowlog

    1
    2
    3
    4
    5
    6
    7
    8
    9
    10
    - 查看当前slowlog-log-slower-than设置
        127.0.0.1:6379> CONFIG GET slowlog-log-slower-than
        1) "slowlog-log-slower-than"
        2) "10000"
    - 设置slowlog-log-slower-than为100ms
        127.0.0.1:6379> CONFIG SET slowlog-log-slower-than 100000
        OK
    - 设置slowlog-max-len为1000
        127.0.0.1:6379> CONFIG SET slowlog-max-len 1000
        OK
    3 slowlog 查看
    3.1 查看slowlog总条数
    1
    2
    127.0.0.1:6379> SLOWLOG LEN
    (integer) 4
    3.2 查看slowlog
    1
    2
    3
    4
    5
    6
    7
    8
    9
    10
    11
    12
    13
    14
    15
    16
    17
    18
    19
    20
    21
    22
    23
    24
    25
    26
    27
    28
    29
    127.0.0.1:6379> SLOWLOG GET
    1) 1) (integer) 25
       2) (integer) 1440057769
       3) (integer) 6
       4) 1) "SLOWLOG"
          2) "LEN"
    2) 1) (integer) 24
       2) (integer) 1440057756
       3) (integer) 36
       4) 1) "CONFIG"
          2) "GET"
          3) "slowlog-log-slower-than"
    3) 1) (integer) 23
       2) (integer) 1440057752
       3) (integer) 11
       4) 1) "CONFIG"
          2) "SET"
          3) "slowlog-log-slower-than"
          4) "1"
    4) 1) (integer) 22
       2) (integer) 1440057493
       3) (integer) 27
       4) 1) "CONFIG"
          2) "GET"
          3) "slowlog-log-slower-than"
    5) 1) (integer) 21
       2) (integer) 1440057133
       3) (integer) 7
       4) 1) "monitor"

    如果要获取指定的条数可以使用SLOWLOG GET N命令
    1
    2
    3
    4
    5
    6
    127.0.0.1:6379> SLOWLOG GET 1
    1) 1) (integer) 26            // slowlog唯一编号id
       2) (integer) 1440057815    // 查询的时间戳
       3) (integer) 47            // 查询的耗时（微妙），如表示本条命令查询耗时47微秒
       4) 1) "SLOWLOG"            // 查询命令，完整命令为 SLOWLOG GET，slowlog最多保存前面的31个key和128字符
          2) "GET"

    slowlog源码解读
    参考：http://blog.sina.com.cn/s/blog_48c95a190101gebh.html 
    写的很详细
