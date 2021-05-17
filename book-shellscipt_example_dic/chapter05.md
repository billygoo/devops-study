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
