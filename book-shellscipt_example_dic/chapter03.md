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
  - `^.*<title>\(.*)<\/title>.*$` : `<title>` 태그를 검색해 값을 패턴 매칭 한다. 
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


# 031 작업 파일 디렉토리에서 1년 이상 갱신되지 않은 파일 삭제하기 
```bash 
#!/bin/sh 

lgdir="/var/log/my/app"

# 최종 갱신일이 1년 이상된 오래된 파일 삭제 
find $logdir -name "*.log" -mtime +364 -print | xargs rm -fv 
```
  
- `xargs` 명령어는 파일 목록을 인수로 받아서 임의의 명령어를 실행 한다. 
- Tip 
  - `rm` 명령어 사용시 `-f` 옵션을 통해 오류가 나도 에러 발생 안되도록 할 수 있다. 
  - `xargs` 명령어에 전달되는 파라미터에 공백문자가 있으면 에러가 나는데 `-O` 옵션을 사용해 `null` 문자만 처리하도록 할수 있다. 그리고 다른 명령어도 `-print0` 옵션으로 출력되도록 한다. 
- 같이 보기 
  - xargs(1) : https://man7.org/linux/man-pages/man1/xargs.1.html
  - xargs 활용 : https://rsec.kr/?p=91


# 032 로그 파일이 엄청 많은 디렉토리에서 파일들에 명령어를 일괄 실행하기 
```bash 
#!/bin/sh 

logdir="/var/log/myapp"

# 확장자 .log 파일에서 "ERROR" 문자열 검색 
find $logdir -name "*.log" -print | xargs grep "ERROR" /dev/null 
```

- 유닉스에는 명령행 인수 상한값이 `ARG_MAX` 상수로 지정되어 있음 
  - `ARG_MAX`를 읽어오기 위해 `getconf ARG_MAX` 명령어를 사용하면 된다. 
- 파일이 너무 많이 있는 경우 `*`를 사용할 경우 `Argument list to long` 메시지가 나올 수 있다. 이를 `xargs`를 활용하면 처리 된다. 
- Tip
  - 다음과 같이 `grep` 명령어 뒤에 `/dev/null`을 추가 함으로써 여러 파일을 처리하는 것으로 동작하게 하면, 파일명을 출력할 수 있다. 
  ```bash
  find $logdir -name "*.log" -print | xargs grep "ERROR" /dev/null 
  ```

# 033 파일을 백업할 때 파일명에 날짜 넣기 
```bash 
#!/bin/sh

config="myapp.conf"

bak_filename="${config}.$(date '+%Y%m%d')"

# 이미 myapp.conf.20131202 등이 있으면 초까지 넣어서 백업 파일 작성 
if [ -e $bak_filename ]; then 
  bak_filename="${config}.$(date '+%Y%m%d%H%M.%S')"
fi

cp -v "$config" "$bak_filename"
```



# 034 파일들을 다른 디렉토리에 동기화해서 백업처리하기 
```bash
#!/bin/sh 
log_dir="/home/user1/myapp/log"
backup_dir="/backup/myapp"

# /home/user1/myapp/log 안에 있는 로그 파일을 
# /backup/myapp/log 디렉토리에 복사 
rsync -av "$log_dir" "$backup_dir"
```
- `rsync`은 파일을 동기화 하는 명령어로 서버 관리 용도로 널리 쓰임 
  - 원본과 대상의 차이를 기준으로 갱신된 파일만 복사된다. 
  - 파일 타임스탬프, 퍼미션, 소유자 정보 등 파일 속성이 복사된다.
  - ssh를 써서 서버간 파일을 동기화 할 수 있다.
- `-a`는 아카이브 모드로 `-rlptgoD` 옵션을 사용하는 것과 같다. 
  - 하위 디렉토리, 심링크, 퍼미션, 타임스탬프 등 속성 정보를 그대로 복사하는 것이다. 
- `-n`(`--dry-run`)을 사용하면 실제 동작하지 않고 복사될 파일만 출력한다.
- 같이보기 
  - `rsync` : https://man7.org/linux/man-pages/man1/rsync.1.html

# 035 로컬 디렉토리에 파일을 만들지 않고 직접 원격 호스트에 아카이브하기 
```bash 
#!/bin/sh 

username="user1"
server="192.168.1.5"

tar cvf - myapp/log | ssh ${username}@${server} "cat > /backup/myspplog.tar"
```
- `tar` 는 파일을 하나로 묶을 뿐 압축 처리하지 않는다. 대신 압축하려면 `-z` 옵션을 활용하면 된다.
- `tar` command example
  ```bash
  tar -cvf [압축 파일명] [압축할 파일들 또는 디렉토리]
  tar -cvf temp.tar /etc 
  # 압축 파일명을 `-`으로 지정하면 표준 출력으로 전달한다.
  tar -cvf - /etc
  ```

# 036 중요한 파일을 암호 걸어서 zip으로 아카이브하기 
```bash
#!/bin/sh

logdir="/home/user1/myapp"

cd "$logdir"

# /home/user1/myapp/log 디렉토리에 있는 로그 파일을 
# 암호 걸린 zip 으로 아카이브 
zip -e -r log.zip log 
```
- `tar+gz`은 아카이브 할때 암호를 지정할 수 없기 때문에 `zip -e` 옵션을 활용할 수 있다. 

# 037 gzip 명령어로 압축률 높이기 
```bash 
#!/bin/sh 

tar cf archive.tar log 

# -9 옵션으로 압축률을 최대로 함
gzip -9 archive.tar 
```
- `gzip` 명령어는 `-숫자` 옵션을 써서 압축률을 조정할 수 있다.
- `GZIP` 환경 변수에 `-숫자` 옵션을 지정하면 기본값 옵션을 지정할 수 있다. 

# 038 tar 아카이브할 때 일부 파일이나 디렉토리 제외하기
```bash
#!/bin/sh

tar cvf archive.tar --exclude ".svn" myapp
```
- `tar` 명령어 사용시 `--exclude` 옵션 활용시 필요 없는 파일/디렉토리를 제외할 수 있다. 
- `tar` 명령어 `-X` 옵션 사용하면 저장된 파일에서 정의된 파일/디렉토리를 제외할 수 있다. 

# 039 tar 아카이브에 파일 추가하기 
```bash
#!/bin/sh

# 년월로 아카이브 파일 지정(예: 201312.tar)
archivefile="$(date +'%Y%m').tar"
# 오늘 날짜로 로그 파일 지정(예: 20131205.log)
logfile="$(date +'%Y%m%d').log"
# 월별 아카이브에 오늘 로그 추가 
tar rvf $archivefile log/$logfile
```
- 'tar'의 'r' 옵션을 지정하면 아카이브 파일 끝에 파일을 추가할 수 있다.


# 040 파일 퍼미션과 타임 스탬프 등 원래 파일 속성을 유지한 채 파일 복사하기 
```bash
#!/bin/sh

backup_dir="home/user1/backup"

# myapp 디렉토리를 $backup_dir 밑에 백업 복사
while getooopts "a" option
do
  case $option in 
    a)
      cp -a myapp "$backup_dir"
      exit
      ;;
  esac
done

cp -R myapp "$backup_dir"
```
- `cp` 명령어로 파일을 복사하면 `umask`, `timestamp`가 변경된다. 
- `cp` 명령어로 백업 용도로 복사하려면 `-a` 옵션을 사용해야 한다. 

# 041 HTMl 파일인 .htm과 .html 확장자를 txt로 일괄 변경하기 
```bash
#!/bin/sh

for filename in *
do 
  case "$filename" in
    *.htm | *.html)
      #파일명 앞 부분을 취득(index)
      headname=${filename%.*}

      # 파일명을 .txt로 변환 
      mv "$filename" "${headname}.txt"
    ;;
  esac
done
```
- 파라미터 확장(Parameter Expansion)을 사용해 문자열들 연산을 할 수 있다. 

| Category | Syntax | Description | Example |
|----|----|----|----|
| 길이 | `${#PARAM}` | 문자열 길이 | | 
|  | `${#ARRAY[@]}` or `${#ARRAY[*]}` | 배열의 길이는 @ 또는 *을 써야한다. | | 
| 매칭되는 문자열 삭제 | `${PARAM#패턴}` | 앞에서 부터 검색해 매칭되는 문자열 삭제 | |
| | `${PARAM##패턴}` | 앞에서 부터 검색해 마지막으로 매칭되는 문자열 삭제 | |
| | `${PARAM%패턴}` | 뒤에서 부터 검색해 매칭되는 문자열 삭제 | |
| | `${PARAM%%패턴}` | 뒤에서 부터 검색해 마지막으로 매칭되는 문자열 삭제 | |
- 활용 예시 : 확장자 분리, 디렉토리와 파일명 분리, 라인 찾아 지우기 등
- 같이 보기 
  - https://www.gnu.org/software/bash/manual/html_node/Shell-Parameter-Expansion.html

# 042 처리 시작 전에 실행 권한을 확인해서 정상 동작이 가능한지 확인 후 실행하기
```bash
#!/bin/sh 

start_command="./start.sh"

if [ -x "$start_command" ]; then
  $start_command
else
  echo "ERROR: -x $start_command failed." >&2
  exit 1
fi
```
- `test` 명령어는 조건 판단을 해서 그 결과가 참이면 종료 스테이터스로 `0`을 돌려준다. 

# 043 두 파일을 비교해서 오래된 파일 삭제하기 
```bash
#!/bin/sh

# 비교 대상 파일
log1="log1.log"
log2="log2.log"

# 인수 파일이 존재하는지 확인해서 존재하지 않으면 종료
filecheck()
{
  if [ ! -e "$1" ]; then
    echo "ERROR: File $1 does not exist." >&2
    exit 1;
  fi
}

filecheck "$log1"
filecheck "$log2"

# 두 파일을 비교해서 오래된 쪽 삭제
if [ "$log1" -nt "$log2" ]; then
  echo "[$log1]->newer, [$log2]->older"
  rm $log2
else
  echo "[$log2]->newer, [$log1]->older"
  rm $log1
fi
```
- `-nt`, `-ot` 조건식으로 타임스탬프를 비교할 수 있다.

# 044 두 디렉토리를 비교해서 한쪽에만 있는 파일 표시하기 
```bash
#!/bin/sh

# 비교할 디렉토리명
dirA="dir1"
dirB="dir2"

# dir1/과 dir2/ 파일 목록 차이를 조사하기 
( cd dir1; find . -maxdepth 1 -type f -print | sort ) > dir1-file.lst
( cd dir2; find . -maxdepth 1 -type f -print | sort ) > dir2-file.lst 

comm dir1-file.lst dir2-file.lst
```
- `comm` 명령어로 두 파일의 차이를 출력할 수 있다. 
- 단 `comm` 명령어로 비교 하기 위해선 두 파일이 정렬 되어 있어야 한다. 
- `-1`, `-2`, `-3` 옵션을 이용해 원하는 출력을 조정할 수 있다. 
- 같이보기
  - comm - compare two sorted files line by line
  - https://man7.org/linux/man-pages/man1/comm.1.html


# 045 디렉토리에 있는 서브디렉토리들의 디스크 사용량 조사하기 
```bash
#!/bin/sh

data_dir="/home/user1/myapp/data"

# $data_dir 디렉토리의 서브디렉토리 용량 표시
du -sk ${data_dir}/*/ | sort -rn
```
- du 명령어로 표시되는 값은 파일 크기가 아니라 파일이 디스크에서 사용하는 블록 크기 이다. 


# 46 작업 파일을 만들 때 내용을 읽지 못하도록 보안 대책 세우기
```bash
#!/bin/sh

umask 077

# echo 명령어 출력을 권한 600인 임시 파일로 작성 
echo "ID: abcd123456" > idinfo.tmp 
```

- umask : 마스크 값을 세 자리 8진수로 지정합니다. 이때 2진수로 각 권한이 마스킹 된 곳을 제거하고 권한을 할당합니다.


# 47 바이너리 파일에 포함된 문자열 얻기 
```bash
#!/bin/sh

# 검색할 에러 메시지 
message="Unknown Error"

strings -f /home/user1/myapp/bin/* | grep "$message"
```
- 실행 파일은 컴파일된 바이너리 파일이므로 단순한 텍스트 검색이 불가능하기 때문에 `strings`, `od`, `hexdump` 명령어를 이용해 검색할 수 있다.
  - `strings`
    - `-f` : 문자열 표시 때 파일명도 같이 표시
    - *mac에서는 `-f` 옵션이 없음*
  - `od` : 바이너리 파일 내용을 직접 살펴볼 때 파일을 8진수로 덤프하는 명령어
    - `-c` : ASCII 문자열 출력
  - `hexdump` : 16진수로 덤프하는 명령어
    - `-c` : ASCII 문자열 출력

# 48 .svn 등 숨은 파일과 디렉토리만 나열하기 
```bash
#!/bin/sh

# IFS에 줄바꿈 설정 
IFS='
'

# 현재 디렉토리 아래에 있는 파일을 $filename으로 순서대로 처리 
for filename in $(ls -AF)
do
  case "$filename" in
    .*/)
      echo "dot directory: $filename"
    ;;
    .*)
      echo "dot file: $filename"
    ;;
  esac
done
```
- 유닉스에서는 파일명 처음이 닷(.)이면 닷 파일이라고 부르며 숨김 파일로 다룬다. 
- 'IFS' : Internal Field Separator의 약어로 쉘이 사용하는 구분자를 기록하는 특수 변수
  - 기본값 : 줄바꿈(`\r`), 탭(`\t`), 스페이스
  - 스크립트에서 변경 후 원복하지 않으면 의도하지 않은 결과가 나올 수 있으니 항상 사용 후 복귀한다.
    ```bash
    IFS_BACKUP=$IFS
    IFS='
    '

    ## handle script

    IFS=$IFS_BACKUP
    ```
- `ls`
  - `-A` : 닷 파일을 포함한 현재 디렉토리의 모든 파일을 표시 
  - `-F` : 디렉토리 마지막에 `/`를 붙여준다.

# 49 이중 실행이 가능한 임시 파일 작성하기 
```bash
#!/bin/sh

tmpfile="tmp.$$"

date > $tmpfile
sleep 10

cat $tmpfile
rm -f $tmpfile
```
- 임시 파일명 지정시 프로세스 ID(`$$`)를 활용 가능
- `mktemp`명령어로 고유한 임시 파일명을 생성할 수 있음 


# 050 sed로 파일 치환 시 심볼릭 링크를 실제 파일로 바꾸지 않게 하기 
```bash
#!/bin/sh

filename="target.txt"

if [ ! -e "$filename" ]; then
  # 대상 파일이 존재하지 않으면 에러 종료
  echo "ERROR: File not exists." >&2
  exit 1
elif [ -h "$filename" ]; then
  # 대상 파일이 심볼릭 링크면 readlink 명령어로 
  # 실제 파일을 대상으로 처리하기 
  sed -i.bak "s/Hello/Hi/g" "$(readlink "$filename")"
else
  sed -i.bak "s/Hello/Hi/g" "$filename"
fi
```
- `sed` 
  - `-i` : 파일 내용을 변경할 때 옵션을 사용해 원래 파일을 백업할 수 있음.
    - linux에서는 default가 자기자신이나, mac에서는 반드시 값을 입력해야 한다. 아니면 "" 비어있는 문자열을 명시해야함
- `readlink` : 실제 파일 경로를 얻어온다.
