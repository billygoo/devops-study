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
- awk 
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
- ps : 
  - a : 다른 사용자의 프로세스도 표시
  - x : 터미널을 가지지 않은 프로세스를 보여준다. 
  - -o [fmt]: 콤마(`,`) 또는 스페이스로 구분된 키워드와 연관된 값을 표시함
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


# 116 디스크 용량 감시하기
```bash
#!/bin/sh
