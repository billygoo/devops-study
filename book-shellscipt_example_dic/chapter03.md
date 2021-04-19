# 023 절대 경로, 상대 경로 관계없이 같은 동작하기 
```bash
#!/bin/sh

cd "$(dirname "$0")"

./start.sh
./end.sh
```

- `$0`은 실행된 쉘의 명령어 자체를 뜻한다. 
- 디렉토리 경로에 스페이스가 포함될 수 있으니 큰 따옴표(`"`)로 감싸서 사용한다.


# 024 명령어 사용법을 표시할 때 현재 파일명 표시하기 
```bash
#!/bin/sh

prog=$(basename "$0")

# 인수가 하나가 아니면 도움말을 표시하고 종료
if [ $# -ne 1 ]; then 
    echo "Usage: $prog <string>" 1>&2
    exit 1
fi

# 명령행 인수 $1 표시 
echo "Start: $prog ..."
echo " Input Argument: $1"
echo "Stop: $prog ..."
```

- `basename` : 현재 파일명(or 경로명) 읽어오기
- 특수 매개변수(Special Parameters) 

| 종류  | 변수   | 설명  |
|---|---|---|
| 위치 매개변수  | $0  | 실행된 스크립트 이름  |
|   | $1  | $1, $2, ..., ${10} 순서로 번호 부여, 단 10번째 부터는 중괄호(`{}`)로 감싸야 함  |
|   | $*  | 전체 인자 값  |
|   | $@  | 전체 인자 값과 동일하지만 큰 따옴표(`"`)로 감싸면 다른 값이 나온다  |
|   | $#  | 매개변수의 총 개수  |
| 특수 매개변수  | $$  | 현재 스크립트의 PID  |
|   | $?  | 최근 실행된 함수의 실행 결과  |
|   | $!  | 최근 실행한 백그라운드 명령의 PID  |
|   | $-  | 현재 옵션 플래그  |
|   | $_  | 지난 명령어의 마지막 인자로 설정된 특수 변수  |


# 025 디렉토리 이동한 다음 간단히 원래 장소로 돌아가기
```bash
#!/bin/sh

# 괄호 안은 서브쉘이 되므로 디렉토리 이동은 이 안에서만 유효
(
    echo "Archive: /var/tmp/archive.tar"
    cd /var/tmp
    tar cvf archive.tar *.txt
)

# 스크립트 실행은 현재 디렉토리에서 처리 
echo "Start: command.sh"
./command.sh
```

- 괄호로 둘러싼 부분은 서브쉘(subshell)로 실행된다. 즉, 새로운 프로세스가 생성되어 실행된다.
- `( )`, `$( )`, \` \`, `|`, `&`를 이용하여 명령어를 실행할 때 서브 쉘이 실행된다. 
- 주의 사항 
  - 서브쉘에서 실행된 것은 부모 프로세스에 영향을 미치지 않는다. 


# 026 디렉토리에 잇는 파일과 디렉토리 수 조사하기 
```bash 
#!/bin/sh

targetdir="/home/user1/myapp/work"

filecount=$(find "$targetdir" -maxdepth 1 -type f -print | wc -l)
dircount=$(find "$targetdir" -maxdepth 1 -type d -print | wc -l)

dircount=$(expr $dircount - 1)

echo "대상 디렉토리: $targetdir"
echo "파일 수: $filecount"
echo "디렉토리 수: $dircount"
```

- `find` 명령어 사용시 `-type` 옵션을 사용하면 원하는 타입의 파일만 찾을 수 있다. 


# 027 파일 내용을 삭제해서 빈 파일로 만들기 
```bash
#!/bin/sh

# uptime 명령어 기록 파일 정의
uptimelog="uptime.log"

# 널 명령어로 빈 파일로 초기화 
: > $uptimelog

# 10초마타 6번, uptime 명령어 실행
for i in 1 2 3 4 5 6
do
    uptime >> $uptimelog
    sleep 10
done
```

- `:`(null command) 는 빌트인 명령어로 항상 성공을 반환한다. 
- 같이 보기
  - `:`(null command) : https://www.shell-tips.com/bash/null-command/
    - 활용 사례 : 무한 루푸 만들 때, 빈 파일 만들때, 블록 Comment를 만들때 


# 028 신규 파일을 만들지 않고 이미 잇는 파일만 파일 갱신일을 바꾸기 
```bash
#!/bin/sh

# [YYYYMMDDhhmm.SS]로 [년월일시분.초] 지정
timestamp=201311190123.45

# 파일 타임스탬프 갱신
# -c 옵션이 있으므로 lock 파일은 신규 파일을 만들지 않음 
touch -t $timestamp app1.log
touch -c -t $timestamp lock.tmp 
```

- `stat` 명령어로 타임스탬프를 확인할 수 있다.
- `touch` 명령어를 이용해 타임스탬프를 변경할 수 있다. 
- `find` 명령어의 `-mtime`옵션을 사용하면 타임스탬프 기준으로 검색할 수 있다.
- 유닉스 파일 타임스탬프

| Kind | Description |
| --- | --- |
| atime | 최종 접근 시각(access time) | 
| mtime | 최종 수정 시각(modify time) | 
| ctime | 최종 상태변경 시각(change time) | 


# 029 여러 HTML 파일에서 title 태그만 추출해서 각각 다른 파일로 출력하기 
```bash
#!/bin/sh

# 현재 디렉토리에 있는 .html 파일이 대상
for htmlfile in *.html
do
    # 파일명에서 확장자를 뺀 문자열 취득 
    fname=$(basename $htmlfile .html)

    # <title> 태그 내용을 후방참조로 추출, 파일 출력 
    sed -n "s/^.*<title>\(.*)<\/title>.*$/\1/p" $htmlfile > output/${fname}.txt
done
```

- `basename` 명령어에서 두 번째 인자로 suffix를 붙이면 반환값에서 해당 값이 지워진 채 반환된다.
  - 파일 확장자를 지울 때 활용 가능
- `sed -n "s/^.*<title>\(.*)<\/title>.*$/\1/p" $htmlfile`
  - `$htemlfile`을 검색한다.
  - `-n` : 패턴 스페이스를 출력하지 않는다.
  - `^.*<title>\(.*)<\/title>.*$` : <title> 태그를 검색해 값을 패턴 매칭 한다. 
  - `\1` : 후방 참조(또는 역참조)로 tag 내용을 표시한다. 
  - `p` 플래그: 치환이 발생했을 때만 출력한다.  
- 같이 보기
  - Linux sed 사용법 : https://www.lesstif.com/lpt/linux-sed-6979751.html


# 030 특정 디렉토리에서 n일 전부터 m일 전까지 변경된 파일 목록 얻기 
```bash
#!/bin/sh

logdir="/var/log/myapp"

# 4일 전부터 2일 전까지 갱신된 파일 목록을 표시 
find $logdir -name "*.log" -mtime -4 -mtime +1 -print
```

- `find` 명령어의 `-mtime`을 이용해 기간을 지정할 수 있다. 
  - `-n`: n일 전 이후 새로운 
  - `+n`: n일 전까지 


