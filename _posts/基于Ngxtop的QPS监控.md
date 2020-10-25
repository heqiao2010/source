---
layout:     post
title:      "基于Ngxtop的QPS监控"
category:   nginx
date:       2018-07-14 18:00:00
author:     "HQ"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 服务端
---

![](https://img.hacpai.com/bing/20180309.jpg?imageView2/1/w/960/h/520/interlace/1/q/100)

>之前参与一个公有云项目的开发，系统入口是公有云平台提供的LB。云平台的LB再将请求转发到后方的多台Nginx，Nginx上再做反向代理到后方的服务器。为了获取系统的QPS，我们在Nginx服务器上写了个定时任务脚本，定期采集并发量，然后汇总。

### 并发量采集脚本
1. 先安装ngxtop
依次运行：
```
sudo yum -y install epel-release
sudo yum -y install python-pip #安装python-pip
sudo yum clean all #清除缓存
sudo pip install ngxtop #安装Ngxtop
```
**注意：由于ngxtop是通过监控access.log文件来获取并发量的，因此nginx.conf中的access log一定要打开。**

安装好了之后就能得到当前这台nginx服务器上此时的并发量了。当然也可以通过一些参数对请求做一些过滤比如`ngxtop -i 'status >= 400' print request status http_referer`;我们获取的并发数是整个系统的，可以不过滤。
```
running for 10 seconds, 1035 records processed: 102.47 req/sec

Summary:
|   count |   avg_bytes_sent |   2xx |   3xx |   4xx |   5xx |
|---------+------------------+-------+-------+-------+-------|
|    1035 |         4793.348 |   611 |   414 |    10 |     0 |

Detailed:
| request_path                                            |   count |   avg_bytes_sent |   2xx |   3xx |   4xx |   5xx |
|---------------------------------------------------------+---------+------------------+-------+-------+-------+-------|
| /portal/protocol                                        |     351 |            4.427 |    14 |   337 |     0 |     0 |
| /portal/protocol/onlines                                |	142 |           29.585 |   142 |     0 |     0 |     0 |
| /fs/group1/M00/38/1A/wKgBjVr-enmAUwbrAAA4icErDmQ64.html |      93 |         7019.968 |    93 |     0 |     0 |     0 |
| /fs/group1/M00/20/55/wKgBK1rX-yGAGUzxAAA2uqGptGw07.html |	 54 |         4009.000 |    54 |     0 |     0 |     0 |
| /portal/getBindStatus                                   |	 48 |           92.000 |    48 |     0 |     0 |     0 |
| /portal/portalError.jsp                                 |	 44 |         6481.818 |    44 |     0 |     0 |     0 |
| /fs/group1/M00/3A/C2/wKgBjVsHZ2yADQyKAAA2BaQs3gQ79.html |	 42 |        12323.667 |    41 |     1 |     0 |     0 |
| /fs/group1/M00/39/D5/wKgBjVsDzz-AEZ_BAAA3KhsbE4Q76.html |	 24 |         8877.833 |    23 |     1 |     0 |     0 |
| /fs/group1/M00/3A/C2/wKgBjVsHZpKAcXlwAAA3pIzc9NQ54.html |	 14 |         4051.000 |    14 |     0 |     0 |     0 |
| /fs/group1/M00/3A/C2/wKgBjVsHZ4mAemZDAAA2Ifroljg77.html |	 12 |         9739.083 |    12 |     0 |     0 |     0 |
```

由于ngxtop命令，需要运行一段时间，才能计算出QPS值，因此我们不能获取这个命令的实时输出，而是需要延时后，取结果。通过如下的shell函数可以做到，延时输出。

```
timeout() {

time=$1

# start the command in a subshell to avoid problem with pipes
# (spawn accepts one command)

command="/bin/sh -c \"$2\""

expect -c "set echo \"-noecho\"; set timeout $time; spawn -noecho $command; expect timeout { exit 1 } eof { exit 0 }"

if [ $?  = 1 ] ;  then

echo  "Timeout after ${time} seconds"

fi

}

timeout 5s /usr/bin/ngxtop >  $work_dir/out.txt   #运行ngxtop命令5s，然后将输出重定向到text文件中。
```
剩下的工作就是将qps值从命令中截取出来即可：
```
req_per_sec=`/usr/bin/cat $work_dir/out.txt|grep records |/usr/bin/awk -F 'sec' '{print $1}'|grep -oP -m 1 "record.+"|/usr/bin/awk -F 'req' '{print $1}'|/usr/bin/awk -F':' '{print $2}'`
```


### 保证采集脚本在数台nginx服务器上并发执行
如何保证采集脚本在多台nginx上几乎同时执行呢——用`python-fab`实现。
```
# -*- coding: utf-8 -*-
# !/usr/bin/python

import os
import paramiko
import root_path

from fabric.api import *
from fabric.context_managers import settings
from fabric.contrib.files import exists
from fabric.decorators import hosts

from utillib import CommonUtil
from utillib import MySqlUtil
from ngxtop import NgxtopDao

paramiko.util.log_to_file("filename.log")

logger = CommonUtil.getLogger(__name__)
env.user = 'root'
env.key_filename = '/root/.ssh/id_rsa.pub'
env.hosts=CommonUtil.get_value_list('ngxtop') # 取nginx机器列表

@task
@parallel # 保证并发执行
def collect_result():
    if not exists('/home/azureuser/auth-webserver/ngxtop_statistics/concurrent_statistics.sh'):
        run('mkdir -p /home/azureuser/auth-webserver/ngxtop_statistics')
        put('/home/azureuser/auth-webserver/ngxtop/concurrent_statistics.sh','/home/azureuser/auth-webserver/ngxtop_statistics') # 拷贝脚本

    result=run('sh /home/azureuser/auth-webserver/ngxtop_statistics/concurrent_statistics.sh') # 执行脚本
    #logger.info(result)
    return result

@task
@hosts(CommonUtil.get_value_list('ngxtop'))
def delete_script():
    run('rm -rf /home/azureuser/auth-webserver/ngxtop_statistics/concurrent_statistics.sh')


@task
@runs_once # 入库保证只执行一次
def save_to_database():
    collected_output = execute(collect_result)
    create_time=None
    wrong_data_type=False
    create_time=CommonUtil.get_local_time()
    insert_table_name = 'tbl_uam_ngxtop'
    for host, info in collected_output.iteritems():
        logger.info("On host {0} last user was {1}".format(host, info))
        data=info.split(' ')
        ngx_data={}
        ngx_data['create_time'] = create_time
        if(len(data) == 2 ):
            # 有些时候，取出的qps值中间可能会缺失一位，这里补5.
            ngx_data['request_per_second'] = str(data[0]) + '5' + str(data[1])
        else:
            ngx_data['request_per_second'] = str(info)
        ngx_data['ip'] = str(host)
        NgxtopDao.save(insert_table_name,ngx_data)
    logger.info(ngx_data)


if __name__ == '__main__':
    os.system('fab -f NgxtopCollector.py save_to_database')
```
那么这个python脚本，通过crontab每5min触发一次，就能得到相应的qps数据了。

### 数据展示
用E-charts做了个表格，这样几个系统的qps值就可以采集到了。其实通过这个可以做一个简单的告警，比如当qps值到达4000时，发送一个告警邮件，或者告警微信消息。
![qps](https://raw.githubusercontent.com/heqiao2010/heqiao2010.github.io/master/img/qps.png "qps")

![qps](https://raw.githubusercontent.com/heqiao2010/heqiao2010.github.io/master/img/qps2.png "qps")


感谢[YXS](http://yxs1112003.github.io/)编码实现。

