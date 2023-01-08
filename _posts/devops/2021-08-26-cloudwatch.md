---
title: "AWS CloudWatch Logs Dashboard 구성"
excerpt: "EC2 및 어플리케이션의 Log를 수집해보자."
categories:
  - DevOps
tags:
  - DevOps
date: 2021-08-26
last_modified_at: 2021-08-26
---

## 1. AWS CloudWatch

다음과 같은 기능들을 제공한다.

* 애플리케이션 모니터링
* 시스템 전반의 성능 변경 사항에 대응
* 리소스 사용률 최적화
* 운영 상태에 대한 통합된 보기를 확보하는데 필요한 데이터 제공
* 로그, 지표 및 이벤트 형태로 모니터링 및 운영 데이터를 수집
* AWS 리소스, 어플리케이션 및 서비스에 대한 통합된 뷰 제공

<br>

## 2. Dashboard 생성

![image](https://user-images.githubusercontent.com/56240505/130912012-983d5a9f-1e89-4cc0-8978-537ccf203a76.png)

* 좌측 대시보드에서 대시보드 생성하기를 선택한다.

<br>

## 3. Dashboard 지표 위젯 추가

![image](https://user-images.githubusercontent.com/56240505/130912097-e34d237a-070f-4b7b-b35f-6d6bb14ffa88.png)

![image](https://user-images.githubusercontent.com/56240505/130912214-7bbdbeb5-34dc-4722-931d-16169e81f299.png)

* CPUUtilization, NetworkIn, NetworkOut 등 지표를 추가한다.
  * 내 대시보드에서 ``위젯 추가 - 행 - 지표``를 선택한다.
* 지표 그래프 추가 탭에서 검색하는 부분이 있는데, 이를 통해 내가 검사하고 싶은 인스턴스 이름과 지표 등을 잘 보고 선택한다.

### 3.1. MEM_USED, DISK_USED 지표 위젯 추가

* CPUUtilization, NetworkIn, NetworkOut 등의 지표는 EC2 > 인스턴스별 지표에 해당한다.
  * EC2 인스턴스별 지표는 인스턴스 이름 태그, ID, 지표 이름을 보여준다.

![image](https://user-images.githubusercontent.com/56240505/130912545-aa8a5c40-ccdd-4c58-9a6c-7627b3ecd5d4.png)

* 반면, mem_used나 disk_used 등의 지표는 CWAgent > host 에 해당한다.
  * host ip와 지표 이름만을 보여준다.
* 따라서 내가 선택하고 싶은 EC2의 IP를 확인하면서 선택해야하는 번거로움이 존재한다.

그런데 실제로 보면 내가 모니터링하고 싶은 인스턴스의 Host IP가 없다. 별도의 IAM Role & Metrics 설정을 추가해야 한다.

### 3.2. IAM Role & Metrics 설정

이전까지는 단순히 AWS가 제공하는 단순한 EC2 인스턴스 관련 지표를 대시보드에 추가하는 작업이었다. 이번에는 EC2 인스턴스에서 수집한 정보(지표 / 로그)를 CloudWatch로 푸시하는 스텝이다.

자세한 내용은 [AWS Docs](https://aws.amazon.com/ko/premiumsupport/knowledge-center/cloudwatch-push-metrics-unified-agent/) 문서를 참고하자.

![image](https://user-images.githubusercontent.com/56240505/130912785-e6bca682-8313-4d93-a003-6609a6c94d73.png)

* 해당 EC2 SSH에 접속하여 아래와 같은 설정을 추가한다.

> Shell

```bash
$ wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb
$ sudo dpkg -i -E ./amazon-cloudwatch-agent.deb
```

> Shell

```bash
# sudo vi /opt/aws/amazon-cloudwatch-agent/bin/config.json
{
        "agent": {
                "metrics_collection_interval": 60,
                "run_as_user": "root"
        },
        "metrics": {
                "metrics_collected": {
                        "disk": {
                                "measurement": [
                                        "used_percent",
                                        "used",
                                        "total"
                                ],
                                "metrics_collection_interval": 60,
                                "resources": [
                                        "*"
                                ]
                        },
                        "mem": {
                                "measurement": [
                                        "mem_used_percent",
                                        "mem_total",
                                        "mem_used"
                                ],
                                "metrics_collection_interval": 60
                        }
                }
        }
}
```

> Shell

```bash
$ sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -s -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json
$ sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a status
{
  "status": "running",
  "starttime": "2021-03-20T15:12:07+00:00",
  "configstatus": "configured",
  "cwoc_status": "stopped",
  "cwoc_starttime": "",
  "cwoc_configstatus": "not configured",
  "version": "1.247347.5b250583"
}
```

* 이후 대시보드에서 mem_used, disk_used 지표로 해당 인스턴스 IP를 조회해보면 나온다.

<br>

## 4. Logs 수집

EC2 인스턴스의 Metrics(자원 사용 정도 등)을 수집했지만, 조금 더 다양한 로그들을 수집해본다.

> Shell

```bash
$ curl https://s3.amazonaws.com/aws-cloudwatch/downloads/latest/awslogs-agent-setup.py -O
$ sudo apt-get install python
$ sudo python ./awslogs-agent-setup.py --region  ap-northeast-2
```

> Shell

```bash
$ sudo vi /var/awslogs/etc/awslogs.conf

[/var/log/syslog]
datetime_format = %b %d %H:%M:%S
file = /var/log/syslog
buffer_duration = 5000
log_stream_name = {instance_id}
initial_position = start_of_file
log_group_name = [로그그룹 이름]
```

* awslogs-agent 설치할 때 별도로 설정을 건들이지 않았다면 해당 파일의 하단에 위와 같이 syslog가 존재한다.
  * syslog는 말 그대로 해당 인스턴스에 대한 시스템 로그들이다.
* 로그 그룹 이름은 로그를 식별할 때 대단히 중요하니 꼭 명시적으로 작성한다.

> Shell

```bash
[/var/log/syslog]
datetime_format = %b %d %H:%M:%S
file = /var/log/syslog
buffer_duration = 5000
log_stream_name = {instance_id}
initial_position = start_of_file
log_group_name = pickgit-dev-syslog

[/home/ubuntu/logs/access/access.log]
datetime_format = %d/%b/%Y:%H:%M:%S %z
file = /home/ubuntu/logs/access/access.log
buffer_duration = 5000
log_stream_name = access.log
initial_position = end_of_file
log_group_name = pickgit-dev-access-log

[/home/ubuntu/logs/db/db.log]
datetime_format = %d/%b/%Y:%H:%M:%S %z
file = /home/ubuntu/logs/db/db.log
buffer_duration = 5000
log_stream_name = db.log
initial_position = end_of_file
log_group_name = pickgit-dev-db-log

[/home/ubuntu/logs/error/error.log]
datetime_format = %d/%b/%Y:%H:%M:%S %z
file = /home/ubuntu/logs/error/error.log
buffer_duration = 5000
log_stream_name = error.log
initial_position = end_of_file
log_group_name = pickgit-dev-error-log

[/home/ubuntu/logs/warn/warn.log]
datetime_format = %d/%b/%Y:%H:%M:%S %z
file = /home/ubuntu/logs/warn/warn.log
buffer_duration = 5000
log_stream_name = warn.log
initial_position = end_of_file
log_group_name = pickgit-dev-warn-log

[/home/ubuntu/logs/info/info.log]
datetime_format = %d/%b/%Y:%H:%M:%S %z
file = /home/ubuntu/logs/info/info.log
buffer_duration = 5000
log_stream_name = info.log
initial_position = end_of_file
log_group_name = pickgit-dev-info-log
```

우리가 원하는건 syslog 이외의 어플리케이션 관련 로그들이니 조금 더 커스터마이징해본다. 자세한 내용은 [AWS Dcos](https://docs.aws.amazon.com/ko_kr/AmazonCloudWatch/latest/logs/AgentReference.html) 문서를 참고하자.

* file에 들어가는 value에 * 등 Pattern Matcher 사용이 가능하다.

> Shell

```bash
$ sudo service awslogs restart
```

* 설정을 마무리하면 awslogs를 재시작한다.

<br>

## 5. Logs 위젯 추가

로그가 잘 수집이 되었다면 이제 CloudWatch 콘솔에서도 로그 그룹이 검색된다.

![image](https://user-images.githubusercontent.com/56240505/130913819-d994eb4e-7b17-49ce-a54b-95cf129e0a8f.png)
![image](https://user-images.githubusercontent.com/56240505/130913832-c6f8c73e-32dc-4272-ad16-91cce9cee245.png)
![image](https://user-images.githubusercontent.com/56240505/130913851-640abeb3-ea1b-4351-9577-322d694ffa75.png)
![image](https://user-images.githubusercontent.com/56240505/130913898-b99888eb-0a0d-42b3-85f2-ffe2388a0e9d.png)

* 정상적으로 설정했음에도 로그 그룹 검색이 안 되는 경우가 있다.
* 로그 그룹에 정상적으로 추가되었어도, EC2에 쌓인 로그가 웹에 반영되는데 어느 정도 시간이 걸린다.

![image](https://user-images.githubusercontent.com/56240505/130914183-bda42927-1a7c-425c-bbc6-ec8efe874b55.png)
![image](https://user-images.githubusercontent.com/56240505/130914198-bef0ca51-39f4-413c-bebf-6bdf116a8484.png)

* 대시보드에서 로그 테이블 추가를 누르고, 원하는 로그 그룹을 선택한 다음 위젯을 추가하면 된다.
* 쿼리를 통해 대시보드에 보여줄 로그의 양, 시간, 필터링 등을 세밀하게 조정할 수 있다.

<br>

## 6. Nginx 로그 수집

> Shell

```bash
[/var/log/syslog]
datetime_format = %b %d %H:%M:%S
file = /var/log/syslog
buffer_duration = 5000
log_stream_name = {instance_id}
initial_position = start_of_file
log_group_name = pickgit-reverse-proxy-syslog

[/var/log/nginx/access.log]
datetime_format = %d/%b/%Y:%H:%M:%S %z
file = /var/log/nginx/access.log
buffer_duration = 5000
log_stream_name = access.log
initial_position = end_of_file
log_group_name = pickgit-reverse-proxy-nginx-access

[/var/log/nginx/error.log]
datetime_format = %Y/%m/%d %H:%M:%S
file = /var/log/nginx/error.log
buffer_duration = 5000
log_stream_name = error.log
initial_position = end_of_file
log_group_name = pickgit-reverse-proxy-nginx-error
```

리버스 프록시가 존재하는 인스턴스에 똑같이 awslogs를 설치해주고 로그 그룹 수집을 설정한다.

* EC2 인스턴스에 직접 Nginx를 설치했다면 로그는 /var/log/nginx에 쌓인다.
* 만약 도커를 사용한다면 nginx 로그가 도커 컨테이너 내부에 쌓이게 된다.

> Shell

```bash
docker run -d -p 80:80 -p 443:443 -v /var/log/nginx:/var/log/nginx nginx/latest
```

도커를 사용하는 경우 위와 같이 볼륨 마운트 옵션을 추가해 nginx를 실행한다. 볼륨이란 컨테이너의 생명 주기와 관계없이 데이터를 영속적으로 저장할 수 있는 방법이다. 컨테이너가 볼륨을 사용하기 위해서는 볼륨을 컨테이너에 마운트(mount)해야 한다.

* -v 옵션은 :의 좌측(host의 file system)이 우측(container의 file system)과 연결된다.
  * 좌측은 마운트할 볼륨명을, 우측에는 컨테이너 내부의 경로를 명시한다.
  * 컨테이너 내부의 /var/log/nginx 폴더가 호스트의 /var/log/nginx 폴더와 동일해진다.
* 호스트쪽 파일 시스템에 마운트 한다는 것은 컨테이너가 파기되어도 컨테이너에서 작성되거나 수정된 파일이 호스트 파일 시스템에 남아있음을 의미한다.

![image](https://user-images.githubusercontent.com/56240505/130914853-48fcc604-fb46-464c-9db6-28d53fc7e57e.png)

* 리버스 프록시 서버 또한 로그가 잘 수집됨을 볼 수 있다.

<br>

---

## References

* [AWS Dcos](https://docs.aws.amazon.com/ko_kr/AmazonCloudWatch/latest/logs/AgentReference.html)
* 우아한테크코스
