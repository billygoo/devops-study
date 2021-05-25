# 055 디폴트 게이트웨이에 ping이 통하는지 확인하기(리눅스)
```bash
#!/bin/sh

# route 명령어 출력에서 디폴트 게이트웨이 얻기
# 첫 번째 컬럼이 "0.0.0.0"인 줄의 두 번째 컬럼을 추출 
gateway=$(route -n | awk '$1 == "0.0.0.0" {print $2}')

# 디폴트 게이트웨이에 ping 
## ** TIP: 모든 아웃풋 날려버리기
ping -c 1 $gateway > /dev/null 2>&1

# ping 종료 스테이터스로 성공, 실패 확인
if [ $? -eq 0 ]; then
  echo "[Success] ping -> $gateway"6
else
	echo "[Failed] ping -> $gateway"
fi
```
- 디폴트 게이트웨이(Default Gateway)는 네트워크 접근 경로에서 외부 네트워크의 출입구가 되는 기기
- `route` : 라우팅 테이블을 표시할 때 사용
  - `-n` : 현재 경로 테이블 내용을 호스트명이 아닌 IP 주소로 표시한다.
- 리눅스 서버 자체에 디폴트 게이트웨이가 설정되어 있지 않으면 다음 가능성이 있다. 
  - 서버 설정 자체에 문제 발생(네트워크 설정 오류)
  - 네트워크 경로에 문제 발생(LAN 케이블이 빠졌을 경우)
  - 디폴트 게이트웨이에 문제 발생(전원이 꺼졌을 가능성 등)


# 056 디폴트 게이트웨이에 ping이 통하는지 확인하기(FreeBSD/Mac)
```bash 
#!/bin/sh

# netstat 명령어 출력에서 디폴트 게이트웨이 얻기 
# 첫 번째 컬럼이 default인 줄의 두 번째 컬럼 추출 
gateway=$(netstat -nr | awk '$1 == "default" {print $2}')

# 디폴트 게이트웨이에 ping 
ping -c 1 $gateway > /dev/null 2>&1

# ping 명령어 종료 스테이터스로 판단
if [ $? -eq 0 ]; then
  echo "[Success] ping -> $gateway"
else
	echo "[Failed] ping -> $gateway"
fi
```
- BSD 계열 `route` 명령어는 경로를 제어하는 명령어로 라우팅 테이블을 표시하는데 사용할 수 없음
- 대체제로 `netstat` 명령어를 사용해야 함
  - `-r` : 현재 경로 테이블을 표시 
  - `-n` : IP 주소만 표시

# 057 ping으로 특정 호스트 응답 평균 시간을 취득하기 
```bash
#!/bin/sh

ipaddr="192.168.2.1"
count=10

echo "Ping to: $ipaddr"
echo "Ping count: $count"
echo "Ping average[ms]:"

# ping 명령어 실행 결과를 임시 파일에 출력 
ping -c $count $ipaddr > ping.$$

# "time=4.32 ms" 부분을 sed로 추출, awk로 평균값 계산 
sed -n "s/^.*time=\(.*\) ms/\1/p" ping.$$ |\
awk '{sum+=$1} END{print sum/NR}'

# 임시 파일 삭제
rm -f ping.$$
```
- `sed` 명령어 `-n` 옵션과 `p` 플래그로 숫자 부분만 추출해서 응답 시간을 구한다. 
  - `-n` : 처리된 라인을 표준 출력으로 표시한다. 
  - `p` 플래그: 표준 출력으로 일치한 패턴을 출력한다.
- `awk` 명령어를 이용해 평균값 계산을 할 수 있다. 
  - `NR`: `awk` 내장 변수로 입력한 레코드 수
  - `END` : 전체 명령 처리 후 마지막 동작


# 058 arp 테이블에서 지정 IP 주소에 대응하는 MAC 주소를 표시하기 
```bash
#!/bin/sh

ipaddr="192.168.2.1"

macaddr=$(arp -ap | awk "/\($ipaddr\)/ {print \$4}")

if [ -n "$macaddr" ]; then 
  echo "$ipaddr -> $macaddr"
else
  echo "$ipaddr가 ARP 캐쉬에 없습니다."
fi 
```
- `arp` : ARP캐쉬를 조작하는 명령어
  - `-a` : 모든 ARP 캐쉬를 표시
  - `-n` : 이름을 해석하지 않음 
- ARP Cache : ARP 캐시는 IP 주소가 MAC 주소로 확인 될 때 ​​생성되는 주소 확인 프로토콜 항목의 모음입니다. ARP 캐시는 해커와 사이버 공격자가 잠재적으로 사용한다는 단점이 있습니다. 공격자는 ARP 캐시를 통해 가짜 IP 주소를 숨길 수 있다. 
(by [wiki](https://en.wikipedia.org/wiki/ARP_cache))


# 059 호스트명으로 IP주소 취득하기 
```bash
#!/bin/sh

# IP 주소를 얻고 싶은 호스트명 정의 
fqdn="www.google.com"

echo "Address if $fqdn"
echo "============"

# host 명령어로 IP 주소 얻기, awk 가공해서 출력
host $fqdn | \
awk '/has address/ {print $NF, "IPv4"} \
/has IPv6 address/ {print $NF, "IPv6"}'
```
- `host` : DNS 서버에 문의해서 이름을 해석하는 명령어 
  - 대체 가능 명령어 : `nslookup`, `dig`
  - `bind-utils` 패키지에 포함
- `awk`
  - `NF` : 마지막 컬럼을 표시하는 변수

# 060 IP 주소로 호스트명을 얻기 
```bash
#!/bin/sh

while read ipaddr
do
  # host 명령어로 IP 주소 변환
  revlookup=$(host "$ipaddr")

  # host 명령어가 성공했는지
  if [ $? -eq 0 ]; then 
    echo -n "$ipaddr,"
    # host 명령어 출력을 awk로 호스트명만 표시 
    echo "$revlookup" | awk '{print $NF}' | sed 's/\.$//'
  else
    echo "$ipaddr,"
  fi

  # DNS 서버 부하 경감을 위해 1초간 대기
  sleep 1
done < $1
```

# 061 서버의 특정 포트가 여려 있는지 확인하는 스크립트 작성하기
```bash
#!/bin/sh

ipaddr="192.168.2.52"
faillog="fail-port.log"

# 확인할 포트는 80, 2222, 8080
for port in 80 2222 8080
do 
  nc -w 5 -z $ipaddr $port 

  if [ $? -ne 0 ]; then
    echo "Failed at port: $port" >> $ "$faillog"
  fi
done
```
- `nc` : Netcat이라고 부르며 TCP/UDP 패킷으로 다양한 네트워크 확인이 가능함 
  ```txt
  nc [옵션] [대상호스트] [포트 번호]
  ```
  - `-z` : TCP 3WAY 핸드 쉐이크만 하고 실제 데이터 통신은 일어나지 않음
  - `-w` : 타임아웃 설정

# 062 간이 TCP 서버 띄우기 
```bash 
#!/bin/sh

port=8080
nc -v -k -l $port
```
- `nc` 명령어 활용 
  - `-v` : verbose 모드
  - `-k` : 접속 지속하는 옵션 
  - `-l` : 듣기 모드 실행

# 063 ftp로 자동 내려받기, 자동 올리기 
```bash 
#!/bin/sh

# FTP 접속 설정
server="192.168.2.5"
user="user1"
password="xxxxxx"
dir="/home/user1/myapp/log"
filename="app.log"

ftp -n "$server" << __EOT__
user "$user" "$password"
binary
cd "$dir"
get "$filename"
__EOT__
```
- `ftp -n` : 실행때 아무것도 표시하지 않음 
- here doc을 활용하면 실행 명령을 자동화 할 수 있음 

# 064 쉘 스크립트로 CGI 실행하기 
```bash
#!/bin/sh

# CGI 헤더 출력
echo "Content-Type: text/plain"
echo 

# 명령어를 실행해서 브라우저에 표시 
echo "Test CGI: uptime"
uptime
```
- 일반 텍스트로 HTTP를 처리할 때 프로토콜에 맞추어 HTTP 헤더 부분과 본문 부분을 빈 줄로 나눠야 한다. 

# 065 지정한 크기의 파일을 만들어서 전송 속도를 측정하기 
```bash 
#!/bin/sh

# 전송 속도를 측정할 임시 파일 크기 지정, 단위는 킬로바이트(KB)
filesize=1024
# 전송 속도를 측정할 임시 파일명
tmpdata="tmpdata.tmp"
timefile="timecount.tmp"

# 전송에 사용할 임시 파일 작성 
dd if=/dev/zero of="$tmpdata" count=$filesize bs=1024 2> /dev/null

# FTP 전송해서 파일을 PUT
server="192.168.2.5"
user="user1"
password="xxxxxxx"

echo "Filesize: $filesize (KB)"
echo "FTP Server: $server"

(time -p ftp -n "$server") << __EOT__ 2> "$timefile"
user "$user" "$password"
binary
put "$tmpdata"
__EOT__

# time 명령어 출력 결과에서 실제 시간을 얻은 후 나눠서 속도 계산 
realtime=$(awk '/^real / {print $2}' "$timefile")
speed=$(echo "${filesize}/${realtime}" | bc)
echo "Transfer Speed: $speed (KB/sec)"

# 임시 파일 삭제 
rm -f "$tmpdata" "$timefile"
```
- /dev/urandom : 랜덤 바이트 파일을 만들때 사용한 데이터 소스이다
- `TIP` : 명령어 전체를 ()로 둘러싼 서브쉘을 사용할 경우 명령어 전체를 통째로 리다이렉트 처리 할 수 있다. 
  ```bash
  (time -p ftp -n "$server") << __EOT__ 2> "$timefile"
  user "$user" "$password"
  binary
  put "$tmpdata"
  __EOT__
  ```
  - 만약, 서브쉘로 실행 하지 않으면 `time` 명령어만 전달된다. 
- `TIP` : `ftp`는 보안상 문제가 있지만 `scp` 같은 `ssh` 기반 통신은 암호화에 의한 CPU 오버헤드가 커서 순수 파일 측정하는데에는 `ftp`를 사용하는게 효과적임

# 066 IP 주소에 따른 처리 분기를 case문으로 작성하기 
```bash
#!/bin/sh

# 대상 IP 주소를 명령행 인수로 지정하지 않으면 에러 표시 후 종료 
if [ -z "$1" ]; then 
  echo "IP 주소를 지정하세요." >&2
  exit 1
fi

# 대상 네트워크라면 ping 명령어 실행 
case "$1" in
  192.168.2.*|192.168.10.*)
    ping -c 1 "$1" > /dev/null 2>&1

    if [ $? -eq 0 ]; then
      echo "Ping to $1 : [OK]"
    else
      echo "Ping to $1 : [NG]"
    fi
    ;;
  *)
    echo "$1 테스트 대상이 아닙니다." >&2
    exit 2
    ;;
esac
```
- `case`문으로 패턴을 비교하는데 `*`와 같은 와일드카드를 사용할 수 있다.

# 067 로컬 쉘 스크립트 파일을 원격 호스트에서 그대로 실행하기
```bash
#!/bin/sh

username="user1"
script="check.sh"

cat $script | ssh ${username}@192.168.2.4 "sh"
cat $script | ssh ${username}@192.168.2.5 "sh"
cat $script | ssh ${username}@192.168.2.6 "sh"
```

- `check.sh`
```bash
#!/bin/sh

# 개통 확인할 대상 서버
checkserver="192.168.2.35"

# 스크립트를 실행한 호스트명 표시 
hostname

# 서버 개통을 ping 명령어로 확인 
ping -c 1 "$checkserver" > /dev/null 2>&1

if [ $? -eq 0 ]; then
  echo "Ping to $checkserver : [OK]"
else
  echo "Ping to $checkserver : [NG]"
fi
```
- 인터렉티브 vs 비 인터렉티브
  - 인터렉티브 쉘은 입출력을 위해 키보드와 터미널이 접속되는 경우 
  - 비 인터렉티브 쉘은 키보드나 터미널 접속이 안되는 경우
