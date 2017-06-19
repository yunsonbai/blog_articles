---
title: 使用diamond监控系统或api
date: 2016-06-13 18:25:23
tags:
    - python
    - diamond
    - 系统监控
    - api监控
categories: api或系统监控
---
## 引言
``` 
    如何保证系统或api在出问题的时候及时发现并解决呢？这就需要用到监控系统，监控系统可以是实时监控，也可以是
    非实时的（需要定时去检查系统），这里说一下用diamond实现非实时监控
```
<!--more-->
## 安装
``` bash
     # cd /usr/local/Diamond
     # git clone git@github.com:python-diamond/Diamond.git
```

## 配置
### diamond配置（./conf/diamond.conf）
``` bash
    [server]

    handlers = diamond.handler.graphite.GraphiteHandler, diamond.handler.archive.ArchiveHandler

    # User diamond will run as
    # Leave empty to use the current user
    user =

    # Group diamond will run as
    # Leave empty to use the current group
    group =

    # Pid file
    pid_file = /xx/Diamond/run/diamond.pid

    # Directory to load collector modules from
    collectors_path = /xx/Diamond/share/diamond/collectors/
    collectors_config_path = /xx/Diamond/conf/collectors/
    collectors_reload_interval = 5

    # Directory to load handler configs from
    handlers_config_path = /xx/Diamond/conf/handlers/

    # Directory to load handler modules from
    handlers_path = /xx/Diamond/share/diamond/handlers/

    ################################################################################
    ### Options for handlers
    [handlers]

    # daemon logging handler(s)
    keys = rotated_file

    ### Defaults options for all Handlers
    [[default]]

    [[ArchiveHandler]]

    # File to write archive log files
    log_file = /var/log/diamond/archive.log

    # Number of days to keep archive log files
    days = 7

    [[GraphiteHandler]]
    ### Options for GraphiteHandler

    # Graphite server host(本人认为携程carbon比较合适)
    host = xx.xx.xx.xx

    # Port to send metrics to
    port = 2003

    # Socket timeout (seconds)
    timeout = 15

    # Batch size for metrics
    batch = 1

    [[GraphitePickleHandler]]
    ### Options for GraphitePickleHandler

    # Graphite server host(本人认为携程carbon比较合适)
    host = xx.xx.xx.xx  #写你搭建好的carbon服务器地址

    # Port to send metrics to
    port = 2004

    # Socket timeout (seconds)
    timeout = 15

    # Batch size for pickled metrics
    batch = 256

    [[MySQLHandler]]
    ### Options for MySQLHandler

    # MySQL Connection Info
    hostname    = xx.xx.xx.xx
    port        = xx
    username    = xx
    password    = xx
    database    = diamond
    table       = metrics
    # INT UNSIGNED NOT NULL
    col_time    = timestamp
    # VARCHAR(255) NOT NULL
    col_metric  = metric
    # VARCHAR(255) NOT NULL
    col_value   = value

    [[StatsdHandler]]
    host = 127.0.0.1
    port = 8125

    [[TSDBHandler]]
    host = 127.0.0.1
    port = 4242
    timeout = 15

    [[LibratoHandler]]
    user = user@example.com
    apikey = abcdefghijklmnopqrstuvwxyz0123456789abcdefghijklmnopqrstuvwxyz01

    [[HostedGraphiteHandler]]
    apikey = abcdefghijklmnopqrstuvwxyz0123456789abcdefghijklmnopqrstuvwxyz01
    timeout = 15
    batch = 1

    # And any other config settings from GraphiteHandler are valid here

    [[HttpPostHandler]]

    ### Urp to post the metrics
    url = http://localhost:8888/
    ### Metrics batch size
    batch = 100


    ################################################################################
    ### Options for collectors
    [collectors]
    [[default]]
    ### Defaults options for all Collectors

    # Uncomment and set to hardcode a hostname for the collector path
    # Keep in mind, periods are seperators in graphite
    # hostname = my_custom_hostname

    # If you prefer to just use a different way of calculating the hostname
    # Uncomment and set this to one of these values:

    # smart             = Default. Tries fqdn_short. If that's localhost, uses hostname_short

    # fqdn_short        = Default. Similar to hostname -s
    # fqdn              = hostname output
    # fqdn_rev          = hostname in reverse (com.example.www)

    # uname_short       = Similar to uname -n, but only the first part
    # uname_rev         = uname -r in reverse (com.example.www)

    # hostname_short    = `hostname -s`
    # hostname          = `hostname`
    # hostname_rev      = `hostname` in reverse (com.example.www)

    # shell             = Run the string set in hostname as a shell command and use its
    #                     output(with spaces trimmed off from both ends) as the hostname.

    # hostname_method = smart

    # Path Prefix and Suffix
    # you can use one or both to craft the path where you want to put metrics
    # such as: %(path_prefix)s.$(hostname)s.$(path_suffix)s.$(metric)s
    path_prefix = servers #在graphite显示的最外层名字
    # path_suffix =

    # Path Prefix for Virtual Machines
    # If the host supports virtual machines, collectors may report per
    # VM metrics. Following OpenStack nomenclature, the prefix for
    # reporting per VM metrics is "instances", and metric foo for VM
    # bar will be reported as: instances.bar.foo...
    # instance_prefix = instances

    # Default Poll Interval (seconds)
    interval = 10  #(所有的监控多长时间执行一次，并发送数据给carbon)

    ###############################################
    # Default enabled collectors
    ########################################

    #[[CPUCollector]]
    #enabled = True

    #[[DiskSpaceCollector]]
    #enabled = True

    #[[DiskUsageCollector]]
    #enabled = True

    #[[LoadAverageCollector]]
    #enabled = True

    [[MemoryCollector]]
    enabled = True

    #[[VMStatCollector]]
    #enabled = True

    [loggers]

    keys = root

    # handlers are higher in this config file, in:
    # [handlers]
    # keys = ...

    [formatters]

    keys = default

    [logger_root]

    # to increase verbosity, set DEBUG
    level = INFO
    handlers = rotated_file
    propagate = 1

    [handler_rotated_file]

    class = handlers.TimedRotatingFileHandler
    level = DEBUG
    formatter = default
    # rotate at midnight, each day and keep 7 days
    args = ('/var/log/diamond/diamond.log', 'midnight', 1, 7)

    [formatter_default]

    format = [%(asctime)s] [%(threadName)s] %(message)s
    datefmt =

    #######################################
    ### Options for config merging
    # [configs]
    # path = "/etc/diamond/configs/"
    # extension = ".conf"
    #----------------------------------------------
    # Example:
    # /etc/diamond/configs/net.conf
    # [collectors]
    #
    # [[NetworkCollector]]
              # enabled = True

```

### 数据库支持（mysql）
``` bash
    create database diamond
```

### 增加MemoryCollector（当然按照上边的配置还能加其他的）
``` bash
    在目录/xx/Diamond/share/diamond/collectors/中添加一个py文件
    该文件diamond已经写好，在src目录下寻找一下
    （src/collectors/memory/memory.py）

```

### 增加自己的Collector
#### 添加ExampleCollector.conf
``` bash
    在/xx/Diamond/conf/collectors/添加一个ExampleCollector.conf
    （你也可以是其他的名字，xxCollector.conf ）
```

#### 添加ExampleCollector.py
``` bash
    在/xx/Diamond/share/diamond/collectors/中添加ExampleCollector.py
    # coding=utf-8
    import diamond.collector


    def test():
        # dosomething
        pass   

    class ExampleCollector(diamond.collector.Collector):

        def collect(self):
            metric_name = "my.example.metric"
            st = time.time()
            test()
            et = time.time()
            metric_value = (et - st) * 1000
            self.publish(metric_name, metric_value)

    if __name__ == '__main__':
        pass

```

### 启动
``` bash
     # cd /usr/local/Diamond
     # python ./bin/diamond -lf -c ./conf/diamond.conf

```

### 查看结果
``` bash
    可以到已经搭建好的graphite（待我整理完会补充，也可以到网上搜一下）
    后台查看应该会有 /Metrics/servers .....
```

More info: [Diamond](http://diamond.readthedocs.io/en/latest/)
