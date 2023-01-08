---
title: "Server 및 Application 확인"
excerpt: "우아한테크코스 배포 인프라 미션 학습 내용입니다."
categories:
  - DevOps
tags:
  - DevOps
date: 2021-06-20
last_modified_at: 2021-06-20
---

## 1. OS 학습의 필요성

* 작성한 코드는 서버 등에 배포되어 빌드되는데, HDD 등의 저장 장치에 있는 프로그램을 실행하면 명령어 등의 데이터가 메모리에 적재되고 스케쥴러에 의해 CPU에서 연산된다.
* 비용이 연산 및 I/O 속도에 비례하기 때문에 평소에는 HDD에 저장하고, 사용자가 실행시 RAM 메모리에 적재하고 이를 CPU의 L1/L2 캐시 및 레지스터로 읽은 후 연산하는 것이 효율적이다.
* OS는 사용자 혹은 응용 프로그램에게 받은 명령을 쉘을 통해 커널에 전달하고, 커널은 프로세스와 메모리 및 파일 시스템과 I/O간 통신을 관장한다.
* 안정적인 서비스 운영을 위해 프로그램이 실행되는 환경을 학습할 필요가 있다.

<br>

## 2. 진단 과정

* 문제가 왜 발생했는지 파악하기 위해, 전후 상황을 파악하고 원인을 분석한다.
  * 서버 운영시 hang이 걸린 경우.
  * 애플리케이션이 응답이 없거나, 예외를 발생하여 정상적인 동작을 하지 않는 경우.
  * 애플리케이션이 점점 느려지는 경우.
  * 특정 시간대 / 특정 사용자 / 특정 기능 등에 문제가 있는 경우 등.
* 플로우는 다음과 같다.
  * Web Server 쪽에서 접속 로그로 느린 응답을 확인한다.
  * WAS 쪽에서 vmstat 등으로 사용률을 파악한다.
  * ps로 스냅샷을 찍어 용의자를 파악한다.

<br>

## 3. stat의 형태

* 요약은 sar, vmstat 등 단위시간 정보의 합계나 평균을 보여준다.
  * 평균은 상황에 따라 잘못된 해석을 유도할 수 있어, 대략적인 상태조사를 위해서만 활용한다.
  * CPU 사용률이 높은지, I/O 평균 응답이 나쁘지 않은지 등.
* 이벤트 기록은 패킷 캡쳐, 시스템 콜 기록 등 순차적으로 이벤트를 기록한다.
  * 데이터가 너무 방대하여 장애 상황에서 원인 파악 용도로 활용하기 보다는, 문제 상황 재현 등 상세한 내용을 조사할 때 활용한다.
* 스냅샷은 ps, top 등 순간의 상태를 기록하여 현재 문제가 발생하고 있지 않은지, 발생한다면 원인 조사를 하는데 활용한다.

<br>

## 4. USE 방법론

![image](https://user-images.githubusercontent.com/56240505/124613912-af339080-deae-11eb-9209-4136201fcae7.png)

### 4.1. 에러

```bash
ubuntu@ip-192-168-1-31:~$ tail -f /var/log/syslog
Jul  6 13:00:18 ip-192-168-1-31 systemd-timesyncd[560]: Network configuration changed, trying to establish connection.
Jul  6 13:00:19 ip-192-168-1-31 systemd-timesyncd[560]: Synchronized to time server 91.189.89.199:123 (ntp.ubuntu.com).
Jul  6 13:01:01 ip-192-168-1-31 CRON[2538]: (root) CMD (   test -x /etc/cron.daily/popularity-contest && /etc/cron.daily/popularity-contest --crond)
Jul  6 13:17:01 ip-192-168-1-31 CRON[2559]: (root) CMD (   cd / && run-parts --report /etc/cron.hourly)
Jul  6 13:30:19 ip-192-168-1-31 systemd-networkd[670]: eth0: Configured
Jul  6 13:30:19 ip-192-168-1-31 systemd-timesyncd[560]: Network configuration changed, trying to establish connection.
Jul  6 13:30:19 ip-192-168-1-31 systemd-timesyncd[560]: Synchronized to time server 91.189.89.199:123 (ntp.ubuntu.com).
Jul  6 14:00:18 ip-192-168-1-31 systemd-networkd[670]: eth0: Configured
Jul  6 14:00:18 ip-192-168-1-31 systemd-timesyncd[560]: Network configuration changed, trying to establish connection.
Jul  6 14:00:19 ip-192-168-1-31 systemd-timesyncd[560]: Synchronized to time server 91.189.89.199:123 (ntp.ubuntu.com).
```

* 로그는 크게 서버에서 남기는 시스템 로그와 어플리케이션 로그 두 개가 존재한다.
* 시스템 로그는 주로 /var/log/syslog를 확인하며, 그 외에도 cron, boot.log, dmesg 등을 활용한다.
* 일반적으로 로그는 logrotate 설정을 하여 자동으로 압축 및 삭제 등을 한다.

### 4.2. 사용률

* 주로 CPU utilization, Available memory, RX / TX 패킷량, Disk 사용률, IOPS 등을 확인한다.
* 서버의 리소스는 스크립트로 확인하 수 있으며, 우선적으로 Load average를 확인한다.
  * 부하가 클 경우, 영향을 주는 프로세스를 확인한다.
* oom-killer 등 시스템 메시지가 발생한다면 dmesg 혹은 syslog를 통해 확인할 수 있다.
* vmstat를 통해 OS 커널에서 취득할 수 있는 정보를 확인할 수 있다.

> Shell

```bash
ubuntu@ip-192-168-1-31:~$ vmstat 5 5
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 3317720  42020 507504    0    0    14     9    6    9  0  0 100  0  0
```

* si, so : swap-in과 swap-out에 대한 값으로, 0이 아니라면 현재 시스템에 메모리가 부족하다는 뜻이다.
* us, sy, id, wa, st : 각각 user time, 커널에서 사용되는 system time, idle, wait I/O 그리고 stolen time 을 의미한다.
  * stolen time은 hypervisor가 가상 CPU를 서비스 하는 동안 실제 CPU를 차지한 시간을 의미한다.

> Shell

```bash
ubuntu@ip-192-168-1-31:~$ iostat -xt
Linux 5.4.0-1045-aws (ip-192-168-1-31) 	07/06/21 	_x86_64_	(2 CPU)

07/06/21 14:13:42
avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.12    0.00    0.07    0.01    0.06   99.74

Device            r/s     w/s     rkB/s     wkB/s   rrqm/s   wrqm/s  %rrqm  %wrqm r_await w_await aqu-sz rareq-sz wareq-sz  svctm  %util
loop0            1.00    0.00      1.02      0.00     0.00     0.00   0.00   0.00    0.06    0.00   0.00     1.02     0.00   0.10   0.01
loop1            0.00    0.00      0.03      0.00     0.00     0.00   0.00   0.00    0.00    0.00   0.00     6.46     0.00   0.30   0.00
loop2            0.88    0.00      0.90      0.00     0.00     0.00   0.00   0.00    0.06    0.00   0.00     1.02     0.00   0.10   0.01
xvda             0.70    0.33     28.60     21.00     0.14     0.52  16.54  61.03    0.65    2.82   0.00    40.79    62.75   1.58   0.16
```

* iostat을 통해 OS 커널에서 취득한 디스크 사용률을 알 수 있다.
* r/s, w/s rkB/s, wkB/s: read 요청과 write 요청, read kB/s, write kB/s를 의미한다.

```bash
ubuntu@ip-192-168-1-31:~$ free -wh
              total        used        free      shared     buffers       cache   available
Mem:           3.8G        158M        3.1G        772K         44M        537M        3.5G
Swap:            0B          0B          0B
```

* free 명령어로 메모리 사용량을 확인한다.
* available : swapping 없이 새로운 프로세스에서 할당 가능한 메모리의 예상 크기다.

> Shell

```bash
ubuntu@ip-192-168-1-31:~$ top
top - 14:15:16 up  3:45,  1 user,  load average: 0.03, 0.05, 0.02
Tasks: 116 total,   1 running,  56 sleeping,   3 stopped,   0 zombie
%Cpu(s):  0.2 us,  0.0 sy,  0.0 ni, 99.8 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  4028152 total,  3269640 free,   162480 used,   596032 buff/cache
KiB Swap:        0 total,        0 free,        0 used.  3645272 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
    1 root      20   0  159792   9024   6668 S   0.0  0.2   0:03.62 systemd
    2 root      20   0       0      0      0 S   0.0  0.0   0:00.00 kthreadd
    3 root       0 -20       0      0      0 I   0.0  0.0   0:00.00 rcu_gp
    4 root       0 -20       0      0      0 I   0.0  0.0   0:00.00 rcu_par_gp
    6 root       0 -20       0      0      0 I   0.0  0.0   0:00.00 kworker/0:0H-kb
    9 root       0 -20       0      0      0 I   0.0  0.0   0:00.00 mm_percpu_wq
   10 root      20   0       0      0      0 S   0.0  0.0   0:00.16 ksoftirqd/0
   11 root      20   0       0      0      0 I   0.0  0.0   0:00.12 rcu_sched
   12 root      rt   0       0      0      0 S   0.0  0.0   0:00.06 migration/0
   13 root      20   0       0      0      0 S   0.0  0.0   0:00.00 cpuhp/0
   14 root      20   0       0      0      0 S   0.0  0.0   0:00.00 cpuhp/1
   15 root      rt   0       0      0      0 S   0.0  0.0   0:00.34 migration/1
   16 root      20   0       0      0      0 S   0.0  0.0   0:00.05 ksoftirqd/1
   18 root       0 -20       0      0      0 I   0.0  0.0   0:00.00 kworker/1:0H-kb
   19 root      20   0       0      0      0 S   0.0  0.0   0:00.00 kdevtmpfs
   20 root       0 -20       0      0      0 I   0.0  0.0   0:00.00 netns
   21 root      20   0       0      0      0 S   0.0  0.0   0:00.00 rcu_tasks_kthre
   22 root      20   0       0      0      0 S   0.0  0.0   0:00.00 kauditd
   24 root      20   0       0      0      0 S   0.0  0.0   0:00.00 khungtaskd
   25 root      20   0       0      0      0 S   0.0  0.0   0:00.00 oom_reaper
   26 root       0 -20       0      0      0 I   0.0  0.0   0:00.00 writeback
   27 root      20   0       0      0      0 S   0.0  0.0   0:00.00 kcompactd0
   28 root      25   5       0      0      0 S   0.0  0.0   0:00.00 ksmd
   29 root      39  19       0      0      0 S   0.0  0.0   0:00.03 khugepaged
   75 root       0 -20       0      0      0 I   0.0  0.0   0:00.00 kintegrityd
   76 root       0 -20       0      0      0 I   0.0  0.0   0:00.00 kblockd
```

* top 명령어만으로 서버 리소스 사용률을 대부분 파악 가능하다.

### 4.3. 가상 메모리

![image](https://user-images.githubusercontent.com/56240505/124615566-5cf36f00-deb0-11eb-8d02-d2b3b4187e61.png)

* VIRT : 프로세스가 확보한 가상 메모리 영역의 크기다.
* RES : 실제 물리 메모리 영역의 크기다.
* 가상 메모리란 프로그램이 메모리를 사용할 때 물리적인 메모리를 직접 다루지 않고, 가상 메모리라는 소프트웨어적인 메모리를 다루는 구조다.
  * 즉, HDD 등의 2차 기억 장치를 메모리처럼 쓰는 것을 의미한다.
  * 하드웨어에서 제공하는 paging이라는 가상 메모리 구조를 사용해 실현하며, OS가 이 가상 메모리 영역을 관장한다.
* 가상 메모리 구조를 사용하면 본래 물리 메모리 용량 이상의 메모리를 다룰 수 있는 것처럼 프로세스에 보여줄 수 있다.
  * 모든 프로세스들은 자신만의 주소공간을 가지므로(32bit 주소 체계에서 4GB), 가상메모리가 없을 경우 매우 큰 공간이 필요하다.
  * 이에 실제 필요로 하는 부분만 메모리에 올리는 demand-paging 기법을 사용한다.
* 물리 메모리상에서는 뿔뿔히 흩어져 있는 영역을 연속된 하나의 메모리 영역으로 프로세스에게 보일 수 있다.
* 서로 다른 두 개의 프로세스가 참조하는 가상 메모리 영역을 동일한 물리 메모리 영역에 대응시켜 공유 메모리를 구현할 수도 있다.
* 물리 메모리가 부족한 경우는 장시간 사용되지 않은 영역의 **가상 메모리와 물리 메모리 영역 매핑을 해제한다.**
  * 해제된 데이터는 HDD 등에 저장해두고 다시 필요해지면 원래대로 되돌리는데 이를 Swap이라 한다.
  * 따라서 Swap이 발생하는 것은 물리 메모리가 부족하다는 증거이며, RES의 크기가 큰 프로세스가 없는지 확인해야 한다.

### 4.4. 네트워크 상태 확인

> Shell

```bash
ubuntu@ip-192-168-1-31:~$ sar -n DEV,TCP,ETCP 5
Linux 5.4.0-1045-aws (ip-192-168-1-31) 	07/06/21 	_x86_64_	(2 CPU)

14:22:29        IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
14:22:34           lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
14:22:34         eth0      0.20      0.00      0.01      0.00      0.00      0.00      0.00      0.00

14:22:29     active/s passive/s    iseg/s    oseg/s
14:22:34         0.00      0.00      0.20      0.00

14:22:29     atmptf/s  estres/s retrans/s isegerr/s   orsts/s
14:22:34         0.00      0.00      0.00      0.00      0.00

14:22:34        IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
14:22:39           lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
14:22:39         eth0      0.40      1.00      0.02      0.24      0.00      0.00      0.00      0.00

14:22:34     active/s passive/s    iseg/s    oseg/s
14:22:39         0.00      0.00      0.40      0.80

14:22:34     atmptf/s  estres/s retrans/s isegerr/s   orsts/s
14:22:39         0.00      0.00      0.20      0.00      0.00
```

* rxkB/s : Total number of kilobytes received per second
* txkB/s : Total number of kilobytes transmitted per second
* active/s : TCP 연결이 CLOSED 상태에서 SYN-SENT 상태로 직접 전환된 초당 횟수
  * 서버에서 다른 외부 장비로 TCP 연결한 횟수
* passive/s : TCP 연결이 초당 LISTEN 상태에서 SYN-RCVD 상태로 직접 전환한 횟수
  * 서버에 새롭게 접근한 클라이언트 수
* retrans/s : 초당 재전송 된 총 세그먼트 수
  * 재전송은 통신 품질을 판단할 수 있는 기준이다.
    * 네트워크 장비 불량이나 설정에 이상이 있는 경우 재전송이 잦다.
    * 로컬 네트워크인 경우 재전송 비율은 0.01% 이하로 거의 없어야 하고, 원거리 네트워크에서도 0.5% 이하가 일반적이다.

![image](https://user-images.githubusercontent.com/56240505/124616843-63ceb180-deb1-11eb-8c00-095aff19c853.png)

* Time Wait.
* 연결을 해제하는 과정에서, ACK 패킷이 유실되는 경우?
  * FIN에 대한 ACK를 받지 못했기에 LAST-ACK상태이고, 이 후 Client에서 연결을 하기 위해 SYN 요청을 보내더라도 RST를 보내 연결을 끊어버린다.
  * 이에 TIME_WAIT 상태를 두어 이상을 감지하고 한번 더 FIN 패킷을 요청한다.
  * 먼저 연결을 끊는 쪽(Active Closer)에서 TIME_WAIT 소켓이 생성된다.
* TIME_WAIT 상태 자체는 연결을 해제하는 과정에서 자연스럽게 나타나지만, TIME_WAIT 소켓이 많아지면?
  * 로컬의 포트 고갈에 따른 애플리케이션 타임아웃(1분)이 발생한다.
  * 잦은 TCP connection 생성/해제로 인해 서비스의 응답속도가 낮아질 수 있다.
  * TCP는 신뢰성을 보장하기 위해 3 Way Handshaking을 사용하며 연결 / 해제 과정에 많은 비용이 발생한다.
* 따라서 웹 성능 개선을 위해 keepalive, connection pool 등을 통해 연결을 재사용한다.

![image](https://user-images.githubusercontent.com/56240505/124617408-ddff3600-deb1-11eb-966c-f1e9766c6d67.png)

* Close Wait.
* Passive Closer는 FIN 요청을 받으면 CLOSE_WAIT으로 바꾸고 응답 ACK를 전달한다.
  * 그와 동시에 해당 포트에 연결되어 있는 애플리케이션에게 ``close()``를 요청한다.
  * ``close()`` 요청을 받은 애플리케이션은 종료 프로세스를 진행하고 FIN을 클라이언트에 보내 LAST_ACK 상태로 바꾼다.
* 따라서 병목 및 서버 멈춤 현상 등으로 인해 정상적으로 close하지 못할 경우, CLOSE_WAIT 상태로 대기한다.
  * 커널 옵션으로 타임아웃 조절이 가능한 FIN_WAIT이나 재사용이 가능한 TIME_WAIT과 달리, CLOSE_WAIT는 포트를 잡고 있는 프로세스의 종료 또는 네트워크 재시작 외에는 제거 방법이 없다.
  * 따라서 평소에 서비스 부하를 낮은 상태로 유지해야 한다.

<br>

## 5. 포화도

![image](https://user-images.githubusercontent.com/56240505/124617986-5bc34180-deb2-11eb-9ec3-6f8b15a31980.png)

* CPU 사용률 100%는 장애를 의미하지 않지만, 파악하는 이유는?

> Shell

```bash
ubuntu@ip-192-168-1-31:~$ vmstat 5 5
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 0  0      0 3269560  46028 550132    0    0    14    10   10   12  0  0 100  0  0
 0  0      0 3269552  46028 550132    0    0     0     0    9    9  0  0 100  0  0
```

* r : CPU에서 동작중인 프로세스 수를 의미한다.
  * CPU 자원이 포화상태인지 파악할 때 활용한다.
  * r 값이 CPU 값보다 클 경우엔 처리를 하지 못해 대기하는 프로세스가 발생한다.

> Shell

```bash
ubuntu@ip-192-168-1-31:~$ free -wh
              total        used        free      shared     buffers       cache   available
Mem:           3.8G        158M        3.1G        772K         44M        537M        3.5G
Swap:            0B          0B          0B
```

* buffers는 Block I/O의 buffer 캐시 사용량을 의미한다.
* cache는 파일 시스템에서 사용되는 page cache량을 의미한다.
  * 이 값이 0일 경우, Disk I/O가 높다는 뜻이다.

> Shell

```bash
ubuntu@ip-192-168-1-31:~$ iostat -xt
Linux 5.4.0-1045-aws (ip-192-168-1-31) 	07/06/21 	_x86_64_	(2 CPU)

07/06/21 14:34:29
avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.11    0.00    0.07    0.01    0.06   99.76

Device            r/s     w/s     rkB/s     wkB/s   rrqm/s   wrqm/s  %rrqm  %wrqm r_await w_await aqu-sz rareq-sz wareq-sz  svctm  %util
loop0            0.92    0.00      0.94      0.00     0.00     0.00   0.00   0.00    0.06    0.00   0.00     1.02     0.00   0.10   0.01
loop1            0.00    0.00      0.02      0.00     0.00     0.00   0.00   0.00    0.00    0.00   0.00     6.46     0.00   0.30   0.00
loop2            0.81    0.00      0.83      0.00     0.00     0.00   0.00   0.00    0.06    0.00   0.00     1.02     0.00   0.10   0.01
xvda             0.64    0.32     26.17     19.30     0.13     0.48  16.53  60.18    0.65    2.74   0.00    40.78    60.23   1.56   0.15
```

* await : I/O 처리 평균시간을 의미하며, Application이 이 시간동안 대기하게 된다.
  * 보통 하드웨어 상에 문제가 있거나 디스크를 모두 사용하고 있을 경우에 이슈가 발생한다.

<br>

## 6. Thread

* Thread란 프로세스 내에서 실행되는 시간의 흐름 단위로, 프로세스는 최소 하나 이상의 Thread를 갖는다.
* Process vs Thread?
  * 하나의 프로그램에 동시에 병행해서 복수 개의 처리를 수행한다.
  * Multi-Process : 프로세스를 여러 개 생성해서 실행 컨텍스트를 여러 개 확보한다.
  * Multi-Thread : 쓰레드를 여러 개 생성해서 실행 컨택스트를 여러 개 확보한다.
  * 전자는 메모리 공간을 개별적으로 갖지만, 후자는 메모리 공간을 공유한다.
* 메모리 사용 효율은 Multi-Thread가 더 높다.
  * Multi-Process의 경우에도 카피온라이트로 인해 메모리 공간이 공유되므로 현저한 차이는 나지 않는다.
* Multi-Thread의 경우 task 전환시에 메모리 공간의 전환이 발생하지 않는 만큼 전환 비용(Context Switch Overhead)이 낮다.
  * 메모리 공간을 전환하지 않게 되면 CPU 상의 메모리 캐시를 그대로 유지할 수 있다.
* Multi-Process는 기본적으로 프로세스 간 메모리를 직접 공유하지 않고 독립되어 있어 안전하다.
  * Multi-Thread는 메모리 공간 전체를 복수의 쓰레드가 공유하므로 리소스 경합이 발생하기 쉽다.

<br>

## 7. Thread 덤프 분석

* 애플리케이션의 Thread 상의 문제는 대부분 Lock으로 인해 발생한다.
  * BLOCKED 상태인 Thread가 있는지, 한 Task가 너무 오래 Thread를 점유하고 있지는 않은지, CPU 사용률이 너무 높지는 않은지 등을 파악한다.
* Thread 덤프 분석은 다음의 상황에서 활용한다.
  * 사용자 수가 많지도 않은데, CPU 사용량이 떨어지지 않을 때.
  * 특정 애플리케이션을 수행헀는데 응답이 없을 때.

> Shell

```bash
$ ps -ef | pgrep java
$ jstack [pid] > thread.dump
```

* 다음과 같은 정보를 포함하고 있다.
  * 생성 시간
  * JVM
  * DeadLock
  * Thread
* Thread 이름, 식별자, 우선순위(prio), Thread가 점유하는 메모리 주소를 의미하는 Thread ID(tid), OS에서 관리하는 Thread ID (nid), Thread 상태 (NEW, RUNNABLE, BLOCKED, WAITING, TIMED_WAITING, TERMINATED) 등의 정보를 확인할 수 있다.
  * RUNNABLE과 BLOCKED의 경우가 문제가 되기 쉽다.
  * RUNNABLE 상태면서 지속시간이 긴 Thread가 없는지, Lock 처리가 제대로 되지 않아 문제가 발생하고 있지는 않은지 등을 확인한다.

<br>

---

## References

* 우아한테크코스
