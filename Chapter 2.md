
> back-of-the-envelope calculations are estimates you create using a combination of thought experiments and common performance numbers to get a good feel for which designs will meet your requirements.

> [!NOTE]
> "Back-of-the-envelope estimation" 是一种快速、非正式的估算方法，通常在没有详细数据和资源的情况下进行。它的目的是在短时间内得到一个大概的数值或者结论，以便快速评估某个问题或决策的可行性。这种方法通常通过简化假设和基本的数学运算来进行。


# Power of two

| Power | Approximate value | Full name  | Short name |
| :---: | :---------------: | :--------: | :--------: |
|  10   |    1000(1024)     | 1 Kilobyte |    1 KB    |
|  20   |     1 Million     | 1 Megabyte |    1 MB    |
|  30   |     1 Billion     | 1 Gigabyte |    1 GB    |
|  40   |    1 Trillion     | 1 Terabyte |    1 TB    |
|  50   |  1 Quadraillion   | 1 Petabyte |    1 PB    |


# Latency numbers every programmer should know


> Dr. Dean from Google reveals the length of typical computer operations in 2010 [1]. Some numbers are outdated as computers become faster and more powerful. However, those numbers should still be able to give us an idea of the fastness and slowness of different computer operations.


|               Operation time                |       Time        |
| :-----------------------------------------: | :---------------: |
|             L1 CACHE reference              |      0.5 ns       |
|              Branch mispredict              |       5 ns        |
|             L2 cache reference              |       7 ns        |
|              Mutex lock/unlock              |      100 ns       |
|            Main memory refernce             |      100 ns       |
|        Compress 1K bytes with Zippy         | 10,000 ns = 10 µs |
|      Send 2K bytes over 1 Gpbs network      | 20,000 ns = 20 µs |
|     Read 1 MB sequentially from memory      |      250 µs       |
|    Round trip within the same datacenter    |      500 µs       |
|                  Disk seek                  |       10 ms       |
|   Read 1 MB sequentially from the network   |       10 ms       |
|      Read 1 MB sequentially from disk       |       30 ms       |
| Send packet California -> Netherlands -> CA |      150 ms       |
|                                             |                   |

![[Pasted image 20240717173542.png]]

