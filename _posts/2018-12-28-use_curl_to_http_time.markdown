---
layout: post
title: 使用CURL跟踪一个http请求的各段执行时间
date: 2018-12-25 23:18:42 +0800
categories:
  - 系统运维
---
- 目录
{:toc #markdown-toc}

很多场景我们需要获取站点或者某个接口的各类响应时间，在不适用其他软件工具的情况下，系统命令CURL可以完美的帮我们打印出各段的响应时间。

## 命令演示
```shell
# curl -o /dev/null -s -w %{time_namelookup}::%{time_connect}::%{time_appconnect}::%{time_pretransfer}::%{time_starttransfer}::%{time_total}"\n"  "http://www.baidu.com"
```
我简单的向www.bai.com 发送了一个get请求。
其中`-o` 选项把curl返回内容数据的写到`/dev/null`文件中去。`-s`为静默模式，把所有进度相关信息从stdout中屏蔽掉。
`-w`选项指定了请求完成后输出什么信息出来。

 `-w FORMAT`
 格式为一个字符串，可以包含任意数量变量字符串混合的纯文本，也支持使用一个已`@filename`命名的文件来对其结果做格式化。其支持的变量参数可以查看 [curl帮助文档](https://curl.haxx.se/docs/manpage.html)

## 各类响应时间

 演示命令的各参数含义如下,文中未做定义的变量含义请查阅[curl帮助文档](https://curl.haxx.se/docs/manpage.html)：
 ```
time_namelookup：DNS 解析域名耗时
time_connect：client和server端建立TCP连接耗时
time_appconnect: SSL建连耗时
time_pretransfer:从client发出请求；到web的server到文件传输即将开始的耗时
time_starttransfer：从client发出请求；到web的server响应第一个字节的时间，其中包括time_pretransfer的耗时
time_total：client发出请求；到web的server发送会所有的相应数据的时间
speed_download：下载速度  单位 byte/s

 ```

## 格式化输出

```python
$ vim  format.txt 
\n
            time_namelookup:  %{time_namelookup}\n
               time_connect:  %{time_connect}\n
            time_appconnect:  %{time_appconnect}\n
           time_pretransfer:  %{time_pretransfer}\n
              time_redirect:  %{time_redirect}\n
         time_starttransfer:  %{time_starttransfer}\n
                            ----------\n
                 time_total:  %{time_total}\n
\n

$ curl -w "@format.txt" -o /dev/null -s  https://www.baidu.com

            time_namelookup:  1.511
               time_connect:  1.724
            time_appconnect:  2.281
           time_pretransfer:  2.281
              time_redirect:  0.000
         time_starttransfer:  2.510
                            ----------
                 time_total:  2.724

```

 [curl-help]:https://curl.haxx.se/docs/manpage.html