# COLORIS

根据[COLORIS: A Dynamic Cache Partitioning System Using Page Coloring](http://cs-people.bu.edu/yingy/docs/coloris.pdf)提供的源码进行初步试验。

## Environment

```
CPU model            : Intel(R) Core(TM) i5-6500 CPU @ 3.20GHz
Number of cores      : 4
L1d cache            : 32K
L1i cache:           : 32K
L2 cache:            : 256K
L3 cache:            : 6144K
OS                   : Ubuntu precise (12.04.5)
Arch                 : i686 (32 Bit)
Kernel               : 3.8.8
```

* interferences
	- `sysbench --test=cpu --cpu-max-prime=250000 run` on core 0
	- [`cachebench -b -x1 -m30 -d10 -e1`](http://icl.cs.utk.edu/llcbench/index.htm) on core 1
* target program
	- `python -m SimpleHTTPServer 8000` on core 2
* load generator
	- [`wrk -t4 -c800 -d300s --latency http://127.0.0.1:8000/index.html`](https://github.com/wg/wrk) on core 3

Using COLORIS with `insmod alloc.ko apps=sysbench,cachebench,python,wrk qos_pair=100,80,100,80,60,0,70,30` according to README of COLORIS:
```
Module usage:
insmod alloc.ko apps=... qos_pair=...
For apps parameter, pass in applications' process names (as in 
task_struct->comm);
For qos_pair, pass in application-specific QoS requirement (high thred, 
low thred), which should be integers no larger than 100; set to any 
negative integer if want to use system-wide thresholds;
e.g.
insmod alloc.ko apps=gobmk,hmmer qos_pair=70,30,-1,-1
```

## Result

### No COLORIS, No Interference

```
Running 5m test @ http://127.0.0.1:8000/index.html
  4 threads and 800 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     2.92ms   30.95ms   1.92s    99.85%
    Req/Sec     0.94k   640.97     3.36k    72.60%
  Latency Distribution
     50%    2.08ms
     75%    2.09ms
     90%    2.12ms
     99%    2.16ms
  319830 requests in 5.00m, 71.25GB read
  Socket errors: connect 0, read 4248, write 535, timeout 391
Requests/sec:   1065.82
Transfer/sec:    243.13MB
```

### No COLORIS, With Interference

```
Running 5m test @ http://127.0.0.1:8000/index.html
  4 threads and 800 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     3.01ms   29.97ms   1.65s    99.83%
    Req/Sec   829.55    542.54     3.23k    72.89%
  Latency Distribution
     50%    2.18ms
     75%    2.20ms
     90%    2.21ms
     99%    2.24ms
  309563 requests in 5.00m, 68.96GB read
  Socket errors: connect 0, read 3454, write 931, timeout 399
Requests/sec:   1031.53
Transfer/sec:    235.31MB
```

### With COLORIS, No Interference

```
Running 5m test @ http://127.0.0.1:8000/index.html
  4 threads and 800 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     3.03ms   31.56ms   1.65s    99.84%
    Req/Sec     0.93k   549.43     3.40k    76.18%
  Latency Distribution
     50%    2.05ms
     75%    2.08ms
     90%    2.10ms
     99%    2.13ms
  496509 requests in 5.00m, 110.61GB read
  Socket errors: connect 0, read 3558, write 1025, timeout 361
Requests/sec:   1654.50
Transfer/sec:    377.42MB
```

### With COLORIS, With Interference

```
Running 5m test @ http://127.0.0.1:8000/index.html
  4 threads and 800 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     3.02ms   29.61ms   1.61s    99.83%
    Req/Sec   840.95    570.51     3.22k    72.71%
  Latency Distribution
     50%    2.17ms
     75%    2.19ms
     90%    2.21ms
     99%    2.25ms
  322817 requests in 5.00m, 71.91GB read
  Socket errors: connect 0, read 5486, write 455, timeout 423
Requests/sec:   1075.70
Transfer/sec:    245.39MB
```
