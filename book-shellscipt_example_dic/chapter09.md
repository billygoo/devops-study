> 09 서버 관리 

# 104 서버 네트워크 인터페이스와 IP 주소 목록 얻기 
```bash
#!/bin/sh

# ifconfig 명령어로 유효한 인터페이스 표시
# awk 명령어로 인터페이스 명과 IP 주소 추출
# LANG=C /sbin/ifconfig | \
# awk '/^[a-z]/ {print "[" $1 "]"} 
#     /inet / {split($2,arr,":"); print arr[2]}'

# ifconfig 아웃풋 포맷이 변경됨, 추가로 inet6도 표시
LANG=C /sbin/ifconfig | \
awk '/^[a-z]/ {print "[" $1 "]"} 
  /inet[6]? / {print $2}'
```
- `ifconfig` 명령어 출력 결과를 `awk`로 가공하기
- `LANG` 환경변수 : 
- `awk` 
  - split 함수 : 변수를 구분자로 나누어 저장함
    - `split([나눌 변수], [결과 저장 변수], 구분자)`

# 105 서버 에 작성된 사용자 계정 목록 얻기 
```bash
#!/bin/sh

# 사용자 계정 정보 파일
filename="/etc/passwd"

# 줄 첫 글자가 #인 주석은 제외 하고 cut 명령어로 첫번째 값을 표시
grep -v "^#" "$filename" | cut -f 1 -d ":"
```
- `/etc/passwd` : 사용자 목록 파일로 다음 순서로 값이 저장되어 있다
  - 사용자명 -> 암호 -> UID -> GID -> Comment -> home path -> login shell
  - `/etc/default/useradd` : 사용자가 추가 될때 기본 정보를 수정할 수 있다.
- `grep -v "[제외할 패턴]"` : 필요 없는 문자열을 제외할 때 `-v` 옵션을 사용한다.

# 106 허가된 사용자만 스크립트 실행 가능하게 하기
```bash
#!/bin/sh 

# 스크립트 실행을 허용할 사용자 정의 
script_user="batch1"

# id 명령어로 현재 사용자를 취득, 정의와 일치하는지 확인
if [ $(id -nu) = "$script_user" ]; then 
  ./batch_program
else
  echo "[ERROR] $script_user 사용자로 실행하세요." >&2
  exit 1
fi
``` 
- `id`: 사용자 식별 정보를 반환
  - `-u` : user id를 표시 한다. 
  - `-n` : id 번호 대신 이름을 표시, `-u`나 `-g`와 같이 사용


# 107 시스템 셧다운하기 
```bash
#!/bin/sh

# 자기 이외의 사용자가 로그인하지 않았는지 who 명령어 출력으로 확인 
other_user=$(who | wc -l)
if [ "$other_user" -ge 2 ]; then
  echo "[ERROR] who 명령어 출력이 2줄 이상 : 작업 중인 사용자가 있습니다." >&2
  exit 1 
fi 

# 미리 정지해야 할 프로세스가 아직 남아 있는지 확인
commname="/usr/libexec/mysqld"
ps ax -o command | grep -q "^$commname"
if [ $? -eq 0 ]; then
  echo "[ERROR] 셧다운 중리 : 프로세스 $commname 실행 중" >&2
  exit 2
fi

# 셧다운 실행.
shutdown -h now
```
- `ps` : 
  - `a` : 다른 사용자의 프로세스도 표시
  - `x` : 터미널을 가지지 않은 프로세스를 보여준다. 
  - `-o [fmt]`: 콤마(`,`) 또는 스페이스로 구분된 키워드와 연관된 값을 표시함
- Shutdown 명령어는 os 마다 조금씩 차이가 있다.
  - 또한 실행시 sudo 권한이 필요하다.
- 함께보기 
  - [특별한 의미를 갖는 종료 코드](https://wiki.kldp.org/HOWTO/html/Adv-Bash-Scr-HOWTO/exitcodes.html)

# 108 파일명으로 설치된 RPM 패키지명을 확인하기 
```bash
#!bin/sh 

# 파일을 지정하는 명령행 인수를 확인
if [ ! -f "$1" ]; then
  echo "파일이 없습니다: $1" >&2
  exit 2
fi

# 파일명에서 속한 RPM 패키지명 취득
pkgname=$(rpm -qf "$1")

# rpm -qf 명령어 결과로 패키지명 표시
if [ $? -eq 0 ]; then
  echo "$1 -> $pkgname"
else
  echo "$1은 패키지에 포함되지 않습니다." >&2
  exit 1
fi
```
- `rpm` : Redhat 패키지 관리 시스템으로, 패키지를 설치 및 조작하는 명령어
  - `-q` : query 요청
  - `-qf` : 파일이 어떤 패키지에 속하는지 검사 
  - `-ql` : 패키지에 속한 파일 목록 출력


# 109 RPM 패캐지며이 적힌 목록 파일에서 각각의 패키지가 설치, 갱신된 날짜를 확인하기
```bash
#!/bin/sh

# 지정한 목록 파일 존재 확인
if [ ! -f "$1" ]; then 
  echo "대상 패키지 목록 파일이 존재하지 않습니다: $1" >&2
  exit 1
fi

# 인수로 지정한 파일($1)에서 패키지 목록 얻기 
pkglist=#(cat "$1")

# 설치된 rpm 갱신일자 출력
rpm -q $pkglist --queryformat '%{INSTALLTIME:date} : %{NAME}\n'
```
- rpm query 명령어
  ```
  Query options (with -q or --query):
    -c, --configfiles                list all configuration files
    -d, --docfiles                   list all documentation files
    -L, --licensefiles               list all license files
    --dump                           dump basic file information
    -l, --list                       list files in package
    --queryformat=QUERYFORMAT        use the following query format
    -s, --state                      display the states of the listed files
  ```


# 110 서버 구축 패키지 목록을 쉘 스크립트 형태로 관리하기 
```bash
#!/bin/sh

# 설치할 패키지명 정의
pkglist="httpd zsh xz git"

# 패키지 목록에서 순서대로 한 줄씩 읽기
for pkg in $pkglist
do
  # yum 명령어로 패키지 설치 
  yum -y install $pkg
done
```


# 111 특정 프로세스가 정지했는지 감시하기 
```bash
#!/bin/sh

# 감시할 프로세스 명령어
commname="/usr/libexec/mysqld"

# 대상 명령어 프로세스 수를 카운트
count=$(ps ax -o command | grep "$commname" | grep -v "^grep" | wc -l)

# grep 명령어 출력 결과가 0이면 프로세스가 존재하지 않으므로 알림 처리하기
if [ "$count" -eq 0 ]; then
  echo "[ERROR] 프로세스 $commname 찾지 못했습니다." >&2
  /home/user1/bin/alert.sh
fi
```
- `ps` : 현재 프로세스 저오를 출력해 주는 유틸리티
  - `-ax` : 모든 프로세스 표시(BSD 스타일)
  - `-o` : 출력할 표시 항목 지정할 수 있다. 
    - 항목엔 `pid,format,state,tname,time,command`이 있음
  - dash(`-`)를 사용하지 않는 것은 여러 스타일의 옵션을 지원하기 때문이다. (`man ps`)
    ```
    This version of ps accepts several kinds of options:

       1   UNIX options, which may be grouped and must be preceded by a dash.
       2   BSD options, which may be grouped and must not be used with a dash.
       3   GNU long options, which are preceded by two dashes.
    ```

# 112 특정 프로세스 실행 개수가 제한값을 넘었는지 확인하기
```bash
#!/bin/sh

# 감시할 프로세스 명령어와 프로세스 허용 수
commname="/home/user1/bin/calc"
threshold=3

# 프로세스 개수 카운트
count=$(ps ax -o command | grep "$commname" | grep -v "^grep" | wc -l)

# 프로세스 수가 허용값 이상이면 경고 처리
if [ "$count" -ge "$threshold" ]; then
  echo "[ERROR] 프로세스 $commname 다중 실행($count)" >&2
  /home/user1/bin/alert.sh
fi
```


# 113 프로세스를 감시해서 다운 시 자동으로 재실행하기
```bash
#!/bin/sh

# 감시할 프로세스 명령어
commname="/usr/sbin/httpd"

# 감시 프로세스 실행 명령어
start="service httpd start"

# 감시 대상 명령어 프로세스 수 카운트
count=$(ps ax -o command | grep "$commname" | grep -v "^grep" | wc -l) 

# grep 명령어 출력 결과가 0이면 프로세스가 존재하지 않거나
# 이상 상황이라고 보고 프로세스 재실행
if [ "$count" -eq 0 ]; then
  # 로그에 시각 표시
  date_str=$(date '+%Y/%m/%d %H:%M:%S')
  echo "[$date_str] 프로세스 $commname 찾지 못했습니다." >&2
  echo "[$date_str] 프로세스 $commname 실행" >&2

  # 감시 프로세스 실행
  $start
fi
```
- `TIP`: `cron`에 스크립트를 이용해 자동화 할 수 있다. 

# 114 서버 ping 감시하기 
```bash
#!/bin/sh

# ping 실행 결과 스테이터스, 0이면 성공이므로 1로 초기화
result=1

# 대상 서버가 명령행 인수로 지정되지 않으면 에러 종료 
if [ -z "$1" ]; then
  echo "대상 호스트를 지정하세요." >&2
  exit 1 
fi 

# ping 명령어 3회 실행, 성공하면 result를 0으로
i=0
while [ $i -lt 3 ] 
do
  # ping 명령어 실행. 종료 상태만 필요하므로 /dev/null에 리다이렉트
  ping -c 1 "$1" > /dev/null

  # ping 명령어 종료 상태 판별, 성공하면 result=0으로 반복문 탈출
  # 실패하면 3초 대기 후 재실행
  if [ $? -eq 0 ]; then
    result=0
    break
  else
    sleep 3
    i=$(expr $i + 1)
  fi
done

# 현재 시각을 [2013/02/01 13:15:44] 형태로 조합
date_str=$(date '+%Y/%m/%d %H:%M:%S')

# ping 실행 결과를 $result로 판별해서 표시 
if [ $result -eq 0 ]; then 
  echo "[$date_str] Ping OK: $1"
else
  echo "[$date_str] Ping NG: $1"
fi
```
- `TIP` : ping 사용시 사전에 테스트 해봐야 한다. 보안상으로 ICMP 패킷을 허용하지 않는 경우가 있기 때문이다. 


# 115 웹 접근 감시하기
```bash
#!/bin/sh

# 감시 대상 URL 지정
url="http://www.example.org/webapps/check"

# 현재 시각을 [2013/02/01 13:15:44] 형태로 조합
date_str=$(date '+%Y/%m/%d %H:%M:%S')

# 감시 URL에 curl 명령어로 접속해서 종료 상탤를 변수 curlresult에 대입 
httpstatus=$(curl -s "$url" -o /dev/null -w "%{http_code}")
curlresutl=$?

# curl 명령어에 실패하면 HTTP 접속 자체에 문제가 있다고 판단
if [ "$curlresult" -ne 0 ]; then
  echo "[$date_str] HTTP 접속 이상: curl exit status[$curlresult]"
  /home/user1/bin/alert.sh
# 400 번대, 500 번대 HTTP 상태 코드라면 에러로 보고 경고 
elif [ "$httpstatus" -ge 400 ]; then
  echo "[$date_str] HTTP 상태 이상: HTTP status[$httpstatus]"
  /home/user1/bin/alert.s.h
fi
```
- `TIP` : 웹 서버 동작 확인을 위해서 웹 서버 상태 코드를 확인할 수 있다. 
- `curl -w [포맷]` : 명령어 완료 후 출력할 항목 지정


# 116 디스크 용량 감시하기
```bash
#!/bin/sh

# 감시할 디스크 사용률의 허용값 %
used_limit=90

# df 명령어 출력 결과 임시 파일명
tmpfile="df.tmp.$$"

# df 명령어로 디스크 사용량 표시. 첫 줄은 헤더이므로 제거
df -P | awk 'NR >= 2 {print $5,$6}' > "$tmpfile"

# df 명령어로 출력 임시 파일에서 사용률 확인
while read percent mountpoint
do
  # "31%"을 "31"로 % 기호 삭제 
  percent_val=${percent%\%}

  # 디스크 사용량이 허용값 이상이면 경고
  if [ "$percent_val" -ge "$used_limit" ]; then
    # 현재시각을 [2015/02/01 13:15:12] 형식으로 조합
    date_str=$(date '+%Y/%m/%d %H:%M:%S')

    echo "[$date_str] Disk Capacity Alert: $mountpoint ($percent used)"
    /home/user1/bin/alert.sh
  fi
done < "$tmpfile"
```
- `df` : 디스크 사용량을 표시하는 명령어
  - `-P` : POSIX 출력 형식 사용하며, 파일 시스템명이 길어도 한줄로 표시함
  - `-h` : 사람이 읽기 쉬운 형식으로 용량 표시
  - `-k` : 1024바이트 단위로 용량 표시
  - `-i` : inode 정보 표시
  - `-l` : 로컬 파일시스템만 표시

# 117 메모리 스왑 감시하기
```bash
#!/bin/sh

# 감시할 스왑 발생 횟수, 이 숫자를 넘기면 경고
swapcount_limit=10

# vmstat 명령어 출력에서 스왑인, 스왑아웃 값 취득 
swapcount=$(vmstat 1 6 | awk 'NR >= 4 {sum += $7 + $8} END{print sum}')

# 스왑 횟수가 허용값을 넘기면 경고
if [ "$swapcount" -ge "$swapcount_limit" ]; then
  # 현재시각을 [2015/02/01 13:15:12] 형식으로 조합
  date_str=$(date '+%Y/%m/%d %H:%M:%S')

  # 스왑 발생 경고 출력
  echo "[$date_str] Swap Alert: $swapcount (si+so)"
  /home/user1/bin/alert.sh
fi
```
- Swap-in(보조장치 -> 메모리), Swap-out(메모리 -> 보조장치)이 자주 발생할 경우 메모리가 부족하다고 판단할 수 있다.
- `vmstat` : 서버의 현재 가상 메모리 상태를 표시 
  - `vmstat [options] [delay [count]]` : 측정할 횟수와 지연시간을 지정해 표시한다. 
  - linux : si, so로 표시됨
  - FreeBSD : pi, po로 표시됨 
  - Mac : `vmstat` 명령어 대신 `vm_stat` 명령어 사용 
- 같이보기 : `vmstat` 에서 표시되는 정보
  ```
   Procs
       r: The number of runnable processes (running or waiting for run time).
       b: Number of processes blocked waiting for I/O to complete.

   Memory
       swpd: the amount of virtual memory used.
       free: the amount of idle memory.
       buff: the amount of memory used as buffers.
       cache: the amount of memory used as cache.
       inact: the amount of inactive memory.  (-a option)
       active: the amount of active memory.  (-a option)

   Swap
       si: Amount of memory swapped in from disk (/s).
       so: Amount of memory swapped to disk (/s).

   IO
       bi: Blocks received from a block device (blocks/s).
       bo: Blocks sent to a block device (blocks/s).

   System
       in: The number of interrupts per second, including the clock.
       cs: The number of context switches per second.

   CPU
       These are percentages of total CPU time.
       us: Time spent running non-kernel code.  (user time, including nice time)
       sy: Time spent running kernel code.  (system time)
       id: Time spent idle.  Prior to Linux 2.5.41, this includes IO-wait time.
       wa: Time spent waiting for IO.  Prior to Linux 2.5.41, included in idle.
       st: Time stolen from a virtual machine.  Prior to Linux 2.6.11, unknown.
  ```


# 118 CPU 사용률 감시하기 
```bash
#!/bin/sh

# 감시할 CPU %idle 허용값
idle_limit=10.0

# CPU %idel을 mpstat 명령어로 취득, 마지막 줄의 평균값을 추출 
cpu_idle=$(mpstat 1 5 | tail -n 1 | awk '{print $NF}')

# 현재 %idle과 허용값을 bc 명령어로 비교 
is_alert=$(echo "$cpu_idle < $idle_limit" | bc)

# 경고할 것인지 판별
if [ "$is_alert" -eq 1 ]; then 
  # 현재시각을 [2015/02/01 13:15:12] 형식으로 조합
  date_str=$(date '+%Y/%m/%d %H:%M:%S')

  # CPU %idle 저하를 경고로 출력
  echo "[$date_str] CPU %idle Alert: $cpu_idle (%)"
  /home/user1/bin/alert.sh
fi
```
- `mpstat` : 프로세스와 관련된 통계 정보를 출력
  - `mpstat [ options ] [ <interval> [ <count> ] ]` : vmstat와 비슷하게 측정 횟수 및 지연 시간을 지정해 사용
  - Mac : `iostat` 명령어를 대신 사용함 
  - linux : 패키지가 설치 되어 있지 않을 경우 `sysstat` 패키지를 설치
- 같이보기 : `mpstat`에서 표시되는 정보
  ```
  CPU
      Processor number. The keyword all indicates that statistics are calculated as averages among all processors.

  %usr
      Show the percentage of CPU utilization that occurred while executing at the user level (application).

  %nice
      Show the percentage of CPU utilization that occurred while executing at the user level with nice priority.

  %sys
      Show the percentage of CPU utilization that occurred while executing at the system level (kernel). Note that this does not include time spent servicing hardware and software interrupts.

  %iowait
      Show the percentage of time that the CPU or CPUs were idle during which the system had an outstanding disk I/O request.

  %irq
      Show the percentage of time spent by the CPU or CPUs to service hardware interrupts.

  %soft
      Show the percentage of time spent by the CPU or CPUs to service software interrupts.

  %steal
      Show the percentage of time spent in involuntary wait by the virtual CPU or CPUs while the hypervisor was servicing another virtual processor.

  %guest
      Show the percentage of time spent by the CPU or CPUs to run a virtual processor.

  %gnice
      Show the percentage of time spent by the CPU or CPUs to run a niced guest.

  %idle
      Show the percentage of time that the CPU or CPUs were idle and the system did not have an outstanding disk I/O request.
  ```
# 119 웹 페이지 변경 감시하기 
```bash
#!/bin/sh

# 감시 대상 URL
url="http://www.example.org/update.html"

# 내려 받기 파일명 정의
newfile="new.dat"
oldfile="old.dat"

# 파일 내려받기 
curl -so "$newfile" "$url"

# 이전에 내려받은 파일과 curl로 내려받은 파일 비교 
cmp -s "$newfile" "$oldfile"

# cmp 명령어 종료 스테이터스가 0이 아니면 차이가 존재
if [ $? -ne 0 ]; then 
  # 현재시각을 [2015/02/01 13:15:12] 형식으로 조합
  date_str=$(date '+%Y/%m/%d %H:%M:%S')

  # 파일 변경 알림 
  echo "[$date_str] 파일이 변경 되었습니다."
  echo "대상 URL: $url"
fi

mv -f "$newfile" "$oldfile"
```
- `TIP` : 웹 페이지 변경을 감시하기 위해서 매번 파일을 내려받이 이전 파일과 비교를 통해 확인할 수 있음

# 120 MySQL 데이터베이스 백업하기 
```bash
#!/bin/sh

# 데이터베이스 접속 설정 
DBHOST="192.168.11.5"
DBUSER="backup"
DBPASS="password"
DBNAME="billy"

# 데이터베이스 백업 설정 
BACKUP_DIR="/home/user1/backup"
BACKUP_ROTATE=3
MYSQLDUMP="/usr/bin/mysqldump"

# 백업 출력할 디렉토리 확인
if [ ! -d "$BACKUP_DIR" ]; then
  echo "백업용 디렉토리가 존재하지 않습니다: $BACKUP_DIR" >&2
  exit 1
fi

today=$(date '+%Y%m%d')

# mysqldump 명령어로 데이터베이스 백업을 실행 
$MYSQLDUMP -h "${DBHOST}" -u "${DBUSER}" -p"${DBPASS}" "${DBNAME}" > "${BACKUP_DIR}/${DBNAME}-${today}.dump 

# mysqldump 명령어 종료 상태 $?로 확인 
if [ $? -eq 0 ]; then
  gzip "${BAcKUP_DIR}/${DBNAME}-${today}.dump"

  # 오래된 백업 파일 삭제
  find "${BACKUP_DIR}" -name "${DBNAME}-*.dump.gz" -mtime +${BACKUP_ROTATE} | xargs rm -f 
else
  echo "백업 작성 실패:${BACKUP_DIR}/${DBNAME}-${today}.dump
  exit 2
fi
```
- `TIP`: 실행할 명령어를 변수에 할당해서 실행함