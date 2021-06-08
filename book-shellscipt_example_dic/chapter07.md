> 쉘 기능을 자유자재로 다루기 

# 086 함수나 if 문 같은 히어 도큐먼트를 쓸 때 탭 정렬로 보기 좋게 만들기
```bash
#!/bin/sh

# 명령어 인수 확인 
if [ -z "$1" ]; then 
  echo "title을 인수로 지정하세요." >&2 
  exit 1
else
  # 명령행 인수 $1 문자열을 title 요소에 넣어서 표시
  # 히어 도큐먼트에 -(하이픈)을 지정해서 
  # 앞부분 탭을 무시하고 들여쓰기
  cat <<-EOT
  <html>
  <head>
      <title>$1</title>
  </head>
  <body>
      <p>Auto HTML sample.</p>
  </body>
  </html>
  EOT
  # 히어 도큐먼트 부분에 있는 들여쓰기는 스페이스가 아니라 탭 
fi 
```
- 히어 도큐먼트에서 `<<-`를 사용할 경우 각 줄 앞에 있는 **탭(tab)** 을 무시하게 된다.
- 단, 스페이스와 탭을 혼용해 들여쓰기를 할 수 있다. 

# 087 스크립트 실행할 때 시그널을 받아서 현재 실행 상태 출력하기
```bash
#!/bin/sh

count=0                     # 실행 횟수
server="192.168.2.105"      # 통신 대상 서버

# 시그널 USR1 트랩 설정. 현재 $count 표시
trap 'echo "Try Count: $count"' USR1

# nc 명령어로 연속 통신 확인 반복
while [ "$count" -le 1000 ]
do
  # 카운트 1 늘리고 nc 명령어 실행 마지막에 1초 대기 
  count=$(expr $count + 1)
  nc -zv "$server" 80
  sleep 1
done 
```
- `trap` : 시그널 처리를 하기 위한 명령어
  - `USR1` 시그널 : 애플리케이션 마다 원하는 대로 기능을 지정하는 시그널


# 088 HUP 시그널을 받아서 실행 중에 설정 파일을 다시 읽어들이기 
```bash
#!/bin/sh

# 환경 초기화 쉘 함수. 로그 출력할 곳을 설정한 setting.conf 읽음 
loadconf() {
  . ./setting.conf
}

# HUP 시그널로 설정을 다시 읽도록 정의 
trap 'loadconf' HUP 

loadconf        # 첫 초기화 
while :
do
  uptime >> "${UPTIME_FILENAME}"
  sleep 1
done
```

- `setting.conf`
```bash
UPTIME_FILENAME="/var/tmp/uptime.log"
```

- `trap`의 `HUP` 시그널 : 설정 파일을 다시 읽는 동작에 사용

# 089 이상 종료해도 흔적을 남기지 않도록 종료 시 작업 파일을 삭제하는 뒷처리 하기 
```bash
#!/bin/sh

# 임시 파일을 정의, 빈 파일로 초기화 
tmpfile="calctmp.$$"
: > "$tmpfile"

# 트랩 설정, 종료할 때 임시 파일 삭제 
trap 'rm -f "$tmpfile"' EXIT

# 오래 걸리는 계산을 하는 외부 스크립트 설정
./calcA.sh >> "$tmpfile"
./calcB.sh >> "$tmpfile"

# 계산 결과를 더해서 최종 합계를 계산
awk '{sum += 1} END{print sum}' "$tmpfile"
```
- trap 
  - `EXIT` 시그널 : bash에서 사용 가능한 `유사 시그널`로 정상 종료(스크립트 마지막 실행 or exit 명령어로 명시적 종료)나 강제 종료(`ctrl+c` or kill 명령어) 모두에서 발생함.
  - 유사 시그널 : 보통 시그널은 OS가 발생시켜 관리하지만, 유사 시그널은 bash가 바생시키고 받는것도 bash 프로세스 뿐이다. 
    - EXIT : 스크립트 종료시
    - ERR  : 명령어 종료 스테이터스가 0이 아닐 때
    - DEBUG : 명령어가 실행 될 대
    - RETURN : .(닷 명령어) 또는 source 명령어로 외부 스크립트 호출 시

# 090 늘 지정한 환경 변수를 설정해서 명령어를 실행하도록 래퍼 스크립트 작성하기 
```bash
#!/bin/sh

# TMPDIR 환경 변수 설정 
TMPDIR="/disk1/tmp"
export TMPDIR

# exec 명령어로 myappd 실행하고 명령어 인수를 "$0"로 넘김
exec ./myapp "$0"
```
- 래퍼(Wapper) 스크립트 : 환경 변수 등을 자체적으로 설정하고 다른 프로그램을 실행하는 프로그램
    - 데몬 프로그램이나 상주형 애플리케이션 실행 시 자주 사용
- exec : 프로세스를 새로 생성하지 않고 기존 프로세스를 덮어 씌우고 실행한다. 
- export : 변수를 환경변수로 저장하게 해준다. 
- 같이 읽어보기
  - [`fork`와 `exec`](https://blackinkgj.github.io/fork_and_exec/)
  - [`export`와 변수](http://keepcalmswag.blogspot.com/2018/06/linux-unix-export-echo_49.html)
  - [실행 중인 프로세스의 환경변수 확인하기](https://coolengineer.com/entry/%EB%A6%AC%EB%88%85%EC%8A%A4-%EC%8B%A4%ED%96%89-%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4%EC%9D%98-%ED%99%98%EA%B2%BD%EB%B3%80%EC%88%98-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0)
    ```bash
    # cat /proc/[프로세스 번호]/environ
    cat /proc/1000/environ
    ```

# 091 scp로 파일 전송할 때 CPU 이용률 계산해서 압축 처리를 할 것인지 판단하기
```bash
#1/bin/sh

# 테스트 전송 파일명, 전송할 곳 등 정의
username="user1"
filename="transfer.dat"
hostname="192.168.2.10"
path="/var/tmp"
tmpfile="timetmp.$$"

# scp 명령어로 파일 전송
# time 명령어로 시간을 측정, 임시 파일에 출력 
(time -p scp -C "$filename" ${username}@${hostname}:"${path}" ) 2> "$tmpfile"

# time 명령어 출력 임시 파일에서 각 time 추출
realtime=$(awk '/^real / {print $2}' "$tmpfile")
usertime=$(awk '/^user / {print $2}' "$tmpfile")
systime=$(awk '/^sys / {print $2}' "$tmpfile")

# CPU 사용 시간에서 CPU 사용률 계산
cpu_percentage=$(echo "scale=2; 100 * ($usertime + $systime) / $realtime" | bc )
echo "scp 전송 CPU 사용률 : $cpu_percentage (%)"

# 임시 파일 삭제 
rm -f "$tmpfile"
```
- 단순 `scp` 전송 속도를 측정해서 CPU 사용률에 따라 압축 여부를 검토할 수 있다. 
- `scp -C` : 전송할 때 압축 해서 파일을 전송하는 옵션이지만 CPU 부하가 늘어난다.
- time : 지정한 명령어 실행 시간과 CPU 사용 시간을 측정
  - real, user, sys 시간을 출력한다.
  - real : 명령어 시작부터 종료까지 경과한 시간 
  - user : 사용자 CPU 처리 시간
  - sys : 시스템 CPU 처리 시간
  - user+sys : CPU 처리 시간
  - real-(user+sys): I/O 대기 시간 
- `bc` : 소스 계산을 못하는 expr 명령어 대신 사용하는 명령어로 표준 입력으로 받은 값을 계산한다.
  - `scale` : 처리할 소수점 이하 지정

# 092 이식성을 고려한 외부 명령어 이용하기
```bash
#!/bin/sh

# echo 명령어 경로를 환경에 따라 바꿔서 쉘 변수 ECHO에 대입 
case $(uname -s) in
  # mac 이면 쉘 내장이 아니라 외부 명령어 /bin/echo 사용
  Darwin)
    ECHO="/bin/echo"
    ;;
  *)
    ECHO="echo"
    ;;
esac

$ECHO -n "이것은 줄이 이어진 "
$ECHO "메시지 입니다."
```
- `uname -s` : OS 명칭을 표시


# 093 리다이렉트가 번잡하지 않도록 그룹핑해서 보기 좋게 만들기 
```bash 
#!/bin/sh

# 중괄호로 그룹핑해서 리다이렉트를 하나로 합치기 
{
  echo "[Script start]"
  date
  ls
  echo "[Script end]"
} > output.log
```
- 중괄호(`{}`), 소괄호(`()`)를 사용해 명령어 그룹핑을 할 수 있다. 
- 소괄호(`()`) 실행시 그룹핑 된 명령어는 서브쉘로 실행된다. 

# 094 명령어가 실패한 시점에 종료해 스크립트 오작동 방지하기 
```bash
#!/bin/sh

# 명령어 종료 스테이터스가 0이 아니라면 스크립트를 바로 종료하기 
set -e

# 삭제 파일이 있는 디렉토리(일부로 오타)
deldir="/var/log/myapp-"

# 디렉토리 $deldir로 이동해서 확장자가 .log인 파일 삭제 
# set -e 때문에 디렉토리 이동이 실패하면 rm 명령어가 실행되지 않음
cd "$deldir"
rm -f *.log
```
- `set -e` : 스크립트 실행 중 명령어 상태가 0이 아닌 경우 프로그램을 종료
  - `set +e`를 지정하면 효과가 무효화됨
  - 마지막 명령어 실행 상태만 검사하므로, 파이프라인으로 여러 명령어를 쓸때 마지막에 `true`나 `:`(널 명령어)를 사용하면 언제나 명령어가 성공적으로 실행된다.
  - `diff` 명령어와 같이 종료 값이 값 비교 결과 값으로 사용되는 경우 의도하지 않은 결과를 만들 수 있다.

# 095 여러 URL 파일을 동시에 병렬로 내려받기 
```bash
#!/bin/sh

# 병렬로 여러 사이트에서 내려 받기
# 각각 백그라운드에서 처리 
curl -sO http://www.example.org/download/bigfile.dat &
curl -sO http://www.example.com/files/sample.pdf &
curl -sO http://jp.example.net/images/large.jpb &
```
- 명령어 마지막에 `&`를 붙이면 백그라운드로 명령어가 실행된다.

# 096 여러 호스트에 병렬로 ping을 날려서 대기 시간 줄이기 
```bash
#!/bin/sh

# 호스트 세 개를 병렬로 ping 6번 반복해서 실행
ping -c 6 192.168.2.1 > host1.log &
ping -c 6 192.168.2.2 > host2.log &
ping -c 6 192.168.2.3 > host3.log &

# 모든 ping 명령어가 종료할 때까지 대기, 동기화
wait

# ping 명령어 결과 출력
cat host1.log host2.log host3.log 
```
- `wait` : 스크립트 자체의 모든 명령어가 종료될때까지 기다린다. 

# 097 쉘 스크립트 일부에 Perl이나 Ruby사용하기 
```bash
#!/bin/sh

# 테스트 통신할 서버 정의 
ipaddr="192.168.2.1"
port=80

# 1에서 10까지 정수값 난수를 펄 한 줄 명령어로 생성
waittime=$(perl -e 'print 1 + int(rand(10))')

# 테스트 명령어를 대기 시간을 두고 2번 실행 
nc -w 5 -zv $ipaddr $port
echo "Wait: $waittime sec."
sleep $waittime
nc -w 5 -zv $ipaddr $port
```

