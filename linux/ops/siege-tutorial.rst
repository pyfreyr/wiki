.. _seige-tutorial:


=====================
Siege 教程
=====================


Siege 是一个 http 负载测试和基准测试工具。可以根据配置对一个WEB站点进行多用户的并发访问，记录每个用户所有请求过程的相应时间，并在一定数量的并发访问下重复进行。可以根据配置对一个WEB站点进行多用户的并发访问，记录每个用户所有请求过程的相应时间，并在一定数量的并发访问下重复进行。Siege 可以从您选择的预置列表中请求随机的 URL,然后进行压力测试。


安装
==========

进入 http://download.joedog.org/siege/ 选择最新版本：

::

    $ curl http://download.joedog.org/siege/siege-latest.tar.gz -O

解压：

::

    $ tar -zxvf siege-latest.tar.gz

编译，安装：

::

    $ cd siege-4.0.4
    $ ./configure
    $ make && make install

安装路径为 ``/usr/local/bin``，查看版本：

::

    $ siege --version
    SIEGE 4.0.4


参数说明
============

使用 ``siege -h`` 查看帮助信息：

::

    $ siege -h
    SIEGE 4.0.2
    Usage: siege [options]
           siege [options] URL
           siege -g URL
    Options:
      -V, --version             VERSION, prints the version number.
      -h, --help                HELP, prints this section.
      -C, --config              CONFIGURATION, show the current config.
      -v, --verbose             VERBOSE, prints notification to screen.
      -q, --quiet               QUIET turns verbose off and suppresses output.
      -g, --get                 GET, pull down HTTP headers and display the
                                transaction. Great for application debugging.
      -c, --concurrent=NUM      CONCURRENT users, default is 10
      -r, --reps=NUM            REPS, number of times to run the test.
      -t, --time=NUMm           TIMED testing where "m" is modifier S, M, or H
                                ex: --time=1H, one hour test.
      -d, --delay=NUM           Time DELAY, random delay before each requst
      -b, --benchmark           BENCHMARK: no delays between requests.
      -i, --internet            INTERNET user simulation, hits URLs randomly.
      -f, --file=FILE           FILE, select a specific URLS FILE.
      -R, --rc=FILE             RC, specify an siegerc file
      -l, --log[=FILE]          LOG to FILE. If FILE is not specified, the
                                default is used: PREFIX/var/siege.log
      -m, --mark="text"         MARK, mark the log file with a string.
                                between .001 and NUM. (NOT COUNTED IN STATS)
      -H, --header="text"       Add a header to request (can be many)
      -A, --user-agent="text"   Sets User-Agent in request
      -T, --content-type="text" Sets Content-Type in request



使用示例
=============

请求 url：

::

    $ siege -c 20 -r 10 http://www.baidu.com

从 urls.txt 中选择 url 请求：

::

    $ siege -c 5 -r 10 -f urls.txt

POST 请求：

::

    $ siege -c 10 -r 5 http://www.xxxx.com/index.php POST UserId=XXX&Sign=cff6wyt505wyt4c

设置请求间隔：

::

    $ siege -c 10 -r 5 -d 3 http://www.baidu.com


结果说明
=============

::

    Transactions:                  50 hits  （处理次数，本次处理 50 条请求）
    Availability:              100.00 %  （成功次数百分比）
    Elapsed time:                0.28 secs  （运行总时间）
    Data transferred:            0.16 MB  （数据传输量）
    Response time:               0.03 secs  （平均响应时间）
    Transaction rate:          178.57 trans/sec  （请求频率，每秒处理 178.57 次请求）
    Throughput:                  0.56 MB/sec  （吞吐量，传输速度）
    Concurrency:                 4.86  （实际最高并发连接数）
    Successful transactions:       50  （成功传输次数）
    Failed transactions:            0  （失败传输次数）
    Longest transaction:         0.04  （最长传输时间）
    Shortest transaction:        0.01  （最短传输时间）





