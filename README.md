[systemtap]: https://sourceware.org/systemtap/
[agentzh]: https://github.com/openresty/nginx-systemtap-toolkit#sample-bt


<h1 align="center">systemtap-toolkit</h1>

<p align="center">
  <img src="http://image.slidesharecdn.com/tracingsummit2014fromdtracetolinux-141013073949-conversion-gate02/95/from-dtrace-to-linux-56-638.jpg"/>
  <br />
</p>


NAME
====
systemtap-toolkit

Description
===========

This is my systemtap toolkit to online analyze the complicated problem on production with heavy load. All tools are based on my favorite linux tracing/probing tool [systemtap].

Any guys which want to know  what the hell it is in the user space and kernel space should be to learn [systemtap] which is awesome tool:)

Table of Contents
=================

* [NAME](#name)
* [Description](#description)
* [tcp-passive-syn-ack-time](#tcp-passive-syn-ack-time)
* [tcp-active-syn-ack-time](#tcp-active-syn-ack-time)
* [tcp-retrans](#tcp-retrans)
* [who-open-file](#who-open-file)
* [syscall-connect](#syscall-connect)
* [sample-bt](#sample-bt)
* [watch-wvar](#watch-var)
* [tcp-trace-packet](#tcp-trace-packet)
* [ngx-req-watch](#ngx-req-watch)
* [stracelike](#stracelike)
* [redis-watch-req](#redis-watch-req)
* [libcurl-watch-req](#libcurl-watch-req)
* [pdomysql-watch-query](#pdomysql-watch-query)

tcp-passive-syn-ack-time
===============
It's used to measure the time of syn packet to ack packet on the server side in the tcp-3-shakehands.

![tcp](https://raw.githubusercontent.com/detailyang/systemtap-toolkit/master/fixtures/tcp.jpg)

````bash
[root@localhost tmp]# ./tcp-passive-syn-ack-timee -p 80 -t 5000
Collecting tcp dport (80)...syn-ack time

interval min:197us, max:858us avg:519us, cnt:3
value |-------------------------------------------------- count
   32 |                                                   0
   64 |                                                   0
  128 |@                                                  1
  256 |@                                                  1
  512 |@                                                  1
 1024 |                                                   0
 2048 |                                                   0
````



tcp-active-syn-ack-time
===============
It's used to measure the time of syn packet to ack packet on the client side in the tcp-3-shakehands.

![tcp](https://raw.githubusercontent.com/detailyang/systemtap-toolkit/master/fixtures/tcp.jpg)

````bash
[root@localhost systemtap-toolkit]# ./tcp-active-syn-ack-time -p 80 -t 5000
Collecting tcp dport (80)...syn-ack time

dport:80 min:417us, max:542us avg:460us, cnt:3
value |-------------------------------------------------- count
   64 |                                                   0
  128 |                                                   0
  256 |@@                                                 2
  512 |@                                                  1
 1024 |                                                   0
 2048 |                                                   0

````

tcp-retrans
===========
It's used to collecting which tcp packet being retransmit

````bash
[root@localhost systemtap-toolkit]# ./tcp-retrans
Printing tcp retransmission

10.0.2.15:49896 -> 172.17.9.41:80 state:TCP_SYN_SENT rto:0 -> 1000 ms
10.0.2.15:49896 -> 172.17.9.41:80 state:TCP_SYN_SENT rto:1000 -> 2000 ms
10.0.2.15:49896 -> 172.17.9.41:80 state:TCP_SYN_SENT rto:2000 -> 4000 ms
````

who-open-file
=============
It's used to find who is opening the specified file

````bash
[root@localhost systemtap-toolkit]# ./who-open-file -f 123 -t 10000
Collecting who is opening filename 123

cat(13740) is opening the filename: "123"
cat(13741) is opening the filename: "123"
````

syscall-connect
==============
It's used to tracing syscall.connect

````bash
telnet(8062) is connecting to AF_INET@192.168.33.10:1800
telnet(8063) is connecting to AF_INET@192.168.33.10:1800
telnet(8064) is connecting to AF_INET@192.168.33.10:1800
telnet(8065) is connecting to AF_INET@192.168.33.10:1800
telnet(8066) is connecting to AF_INET@192.168.33.10:1800
telnet(8067) is connecting to AF_INET@192.168.33.10:1800
telnet(8068) is connecting to AF_INET@192.168.33.10:1800
telnet(8069) is connecting to AF_INET@192.168.33.10:1800
telnet(8070) is connecting to AF_INET@192.168.33.10:1800
````

sample-bt
=========
It's from [agentzh] and be used to sampling the backtrace in the user space and kernel space.

````bash
$ ./sample-bt -p 8736 -t 5 -u > a.bt
WARNING: Tracing 8736 (/opt/nginx/sbin/nginx) in user-space only...
WARNING: Missing unwind data for module, rerun with 'stap -d stap_df60590ce8827444bfebaf5ea938b5a_11577'
WARNING: Time's up. Quitting now...(it may take a while)
WARNING: Number of errors: 0, skipped probes: 24
````

watch-var
=========
It's used to monitor function param changing.

````bash
[root@localhost systemtap-toolkit]# ./watch-var  -f syscall.open -v filename -p 25849
WARNING: Tracing vars syscall.open filename in 25849...
a.out[25849] kernel.function("SyS_open@fs/open.c:1036").call filename: "" => ""./test""
````

tcp-trace-packet
================
Like tcpdump, it's used to tracing tcp packet with more detail include tcp flag.

````bash
[root@localhost systemtap-toolkit]# ./tcp-trace-packet
WARNING: tracking 0 tcp packet
1478067249998698 10.0.2.15:22 => 10.0.2.2:50627 len:92 SYN:0 ACK:1 FIN:0 RST:0 PSH:1 URG:0
1478067249998955 10.0.2.2:50627 <= 10.0.2.15:22 len:40 SYN:0 ACK:1 FIN:0 RST:0 PSH:0 URG:0
1478067250199252 10.0.2.15:22 => 10.0.2.2:50627 len:172 SYN:0 ACK:1 FIN:0 RST:0 PSH:1 URG:0
1478067250199559 10.0.2.2:50627 <= 10.0.2.15:22 len:40 SYN:0 ACK:1 FIN:0 RST:0 PSH:0 URG:0
1478067250399756 10.0.2.15:22 => 10.0.2.2:50627 len:100 SYN:0 ACK:1 FIN:0 RST:0 PSH:1 URG:0
1478067250399963 10.0.2.2:50627 <= 10.0.2.15:22 len:40 SYN:0 ACK:1 FIN:0 RST:0 PSH:0 URG:0
````

ngx-req-watch
===============
It tracing the userland, which can watch and filter by specified condition nginx request in real time

````bash
[root@localhost systemtap-toolkit]# ./ngx-req-watch -p 5614
WARNING: watching /opt/tengine/sbin/nginx(8521 8522 8523 8524) requests
nginx(8523) GET URI:/123?a=123 HOST:127.0.0.1 STATUS:200 FROM 127.0.0.1 FD:16 RT: 0ms
nginx(8523) GET URI:/123?a=123 HOST:127.0.0.1 STATUS:200 FROM 127.0.0.1 FD:16 RT: 0ms
nginx(8523) GET URI:/123?a=123&b=123 HOST:127.0.0.1 STATUS:200 FROM 127.0.0.1 FD:16 RT: 0ms
nginx(8523) GET URI:/123?w HOST:127.0.0.1 STATUS:200 FROM 127.0.0.1 FD:16 RT: 0ms
nginx(8523) GET URI:/123?w HOST:test STATUS:200 FROM 127.0.0.1 FD:16 RT: 0ms
nginx(8523) GET URI:/123?w=a HOST:test STATUS:200 FROM 127.0.0.1 FD:16 RT: 0ms
````

stracelike
==============
Like strace. But it's based on the [systemtap]
````bash
[root@localhost systemtap-toolkit]# ./stracelike -p 4580 -t 20000
WARNING: stracing syscall
Sat Oct 29 12:46:19 2016.094410  epoll_wait(16, 0x1e17b40, 512, 100) = 0 <0.100334>
Sat Oct 29 12:46:19 2016.194756  epoll_wait(16, 0x1e17b40, 512, 100) = 0 <0.100227>
Sat Oct 29 12:46:19 2016.295006  epoll_wait(16, 0x1e17b40, 512, 100) = 0 <0.101086>
````

redis-watch-req
===============
It tracing the userland, which can watch and filter by specified condition redis request in real time

````bash
[root@localhost systemtap-toolkit]# ./redis-watch-req -p 23261
WARNING: watching /usr/bin/redis-server(23261) requests
redis-server(23261) RT:30(us) REQ: id:2 fd:5 ==> get a #-1 RES: #9
redis-server(23261) RT:23(us) REQ: id:2 fd:5 ==> set a #12 RES: #5
redis-server(23261) RT:16(us) REQ: id:2 fd:5 ==> get foo #-1 RES: #5
````

libcurl-watch-req
=================
It traceing the userland, which can watch and filter by specified condition request for softawre which are based on the libcurl like `curl` and `php`.

````bash
[root@localhost systemtap-toolkit]# ./libcurl-watch-req
WARNING: Tracing libcurl (0) ...
curl(23759) URL:http://www.google.com RT:448(ms) RTCODE:0
curl(23767) URL:http://www.facebook.com/asdfasdf RT:596(ms) RTCODE:0
curl(23769) URL:https://www.facebook.com/asdfasdf RT:902(ms) RTCODE:0
````

pdomysql-watch-query
=================
It traceing the userland, which can watch and filter by specified condition request for php's pdo mysql driver.

````bash
[root@localhost systemtap-toolkit]# ./pdomysql-watch-query -l /usr/lib64/php/modules/pdo_mysql.so

Tracing pdo-mysql (0)
php-fpm(12896) 172.17.10.196:3306@root: SELECT * from person RT:0(ms) RTCODE:1
php-fpm(12896) 172.17.10.196:3306@root: SELECT * from person RT:8(ms) RTCODE:1
php-fpm(12896)172.17.10.196:3306@root: SELECT sleep(5) RT:5012(ms) RTCODE:1
````
