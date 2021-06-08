# 068 ID가 적힌 목록 파일에서 ID에서 추출할 때 ID 끝 문자를 정렬하기 
```bash
#!/bin/sh

# 임시 파일 지정
tmpfile="sort.lst"

# 대상 ID 파일 확인 
if [ ! -f "$1" ]; then 
  echo "ID 목록 파일을 지정하세요." >&2
  exit 1;
fi

# ID 끝 문자 숫자로 목록 정렬 
rev "$1" | sort | rev > $tmpfile

# 정렬한 ID 목록으로 리포트 작성 
./report.sh $tmpfile

# 임시 파일 삭제 
rm -f $tmpfile
```

- `id.lst`
```text
PPX0_2
AN29_9
UIA5_3
BA06_7
QXD3_0
```
- TIP : 파일 끝 정렬을 위해 전체 문자열을 반전하는 `rev`를 사용 후 결과를 구한 후 다시 `rev`를 활용해 원래 값으로 되돌린다.

# 069 텍스트 파일에서 구분자를 지정해서 컬럼 추출하기 
```bash 
#!/bin/sh 

# 미리 설정하지 않으면 에러가 발생하는 환경 변수 정의 
envname="TMPVAR"

# env 명령어로 환경 변수 목록을 표시해서 cut 명령어로 
# * 첫 번째 값을 표시 [ -f 1 ]
# * 분리자는 : [ -d "=" ] 로 표시 
env | cut -f 1 -d "=" > env.lst 

# 확인할 환경 변수명이 env.lst와 같은지로 정의되어 있는지 확인
grep -q "^${envname}$" env.lst

if [ $? -eq 0 ]; then
  # 환경 변수가 설정되어 있으면 start.sh 실행 
  echo "환경 변수 $envname가 설정되어 있습니다."
  ./start.sh
else
  echo "환경 변수 $envname가 설정되어 있지 않습니다."
fi
```
- `cut` : 텍스트 파일에서 일부분을 추출하는 명령어
  - `-d` : 구분자 지정
  - `-f` : 추출할 문자 지정 
- `awk` : `cut`과 비슷하게 활용하며 추가적으로 산술식 등을 처리할 수 있음

# 070 파일 앞머리의 쉬뱅(shebang, #!/bin/sh 등)을 추출해서 스크립트에 따라 확장자 붙이기 
```bash
#!/bin.sh

# 대상 스크립트 파일이 있는지 확인
if [ ! -f "$1" ]; then 
  echo "지정한 파일을 찾지 못했습니다: $1" >&2
  exit 1
fi

# 파일 첫 줄 읽음
headline=$(head -n 1 "$1")

# 파일 첫 줄에 따라 확장자를 판정해서 부여함
case "$headline" in
  */bin/sh|*bash*)
    mv -v "$1" "${1}.sh"
    ;;
  *perl*)
    mv -v "$1" "${1}.pl"
    ;;
  *ruby*)
    mv -v "$1" "${1}.rb"
    ;;
  *)
    echo "Unknown Type: $1"
esac
```
- 관례적으로 스크립트 파일은 해당 언어에 대응하는 확장자를 파일명에 사용하나, 확장자가 없는 경우도 많다. 
- 셔뱅(`shebang`) : 운영체제가 스크립트를 실행할 수 있는 프로그램을 지정하는데 활용하는 코드
  ```bash 
  #!/bin/sh 
  ```
- 파일 실행 관련 필요 권한 
  - Binary : 실행 권한만 필요함
  - Script : 실행 권한과 읽기 권한 필요함. 왜냐하면 실제 `shebang`에 따라 프로그램으로 스크립트를 실행하기 때문이다. 
- `file` : 파일이 무엇인지 판단하는 프로그램
  - `man magic` : `file` 명령어가 파일 종류를 파악하기 위한 매직 파일 

# 071 입력 파일 해시값을 줄마다 추가해서 출력하기 
```bash
#!/bin/sh

# 해시값을 출력할 임시 파일을 초기화 
tmpfile="hash.txt"
: > $tmpfile

# 쉘 구분자를 줄바꿈만 인식하도록 변경
IFS='
'

# 지정한 텍스트 파일에서 한 줄씩 읽어들임
while read -r line
do
  # MD5 해시 취득
  # 명령어에 파일명이 따라오므로 첫 번째 컬럼만 추출
  echo -n "$line" | md5sum | awk '{print $1}' >> $tmpfile
done < $1

# 원본 텍스트 파일과 해시를 출력한 임시 파일을 쉼표로 연결해서 표시
paste -d, "$1" $tmpfile
```
- `paste` : 두 텍스트 파일을 횡방향으로 연결하는 명령어다. 
  - `-d` : 연결할때 사용하는 구분자

# 072 CSV 파일에서 지정한 특정 레코드의 컬럼값 얻기 
- `data.csv`
```txt
0001,kim,45
0002,lee,312
0003,park,102
0004,Kang,3
0005,Seo,92
```

```bash
#!/bin/sh 

# CSV 파일 지정 
csvfile="data.csv"

# ID가 지정되지 않으면 종료
if [ -z "$1" ]; then 
  echo "ID를 지정하세요." >&2
  exit 1
fi

# CSV 파일이 존재하지 않으면 종료 
if [ ! -f "$csvfile" ]; then
  echo "CSV 파일이 존재하지 않습니다: $csvfile" >&2
  exit 1
fi

while read line
do
  # cut으로 컬럼 추출
  id=$(echo $line | cut -f 1 -d ',')
  name=$(echo $line | cut -f 2 -d ',')
  score=$(echo $line | cut -f 3 -d ',')

  # ID 컬럼이 인수로 지정한 ID와 일치하면 표시
  if [ "$1" = "$id" ]; then
    echo "$name"
  fi
done < $csvfile
```
- `cut` : 파일의 각 라인의 텍스트 추출 명령어

# 073 CSV 파일에 ID 목록을 입력해서 대응하는 ID 컬럼값 얻기 
```bash 
#!/bin/sh

filecheck()
{
  if [ ! -f "$1" ]; then
    echo "ERROR: File $1 does not exist." >&2
    exit 1;
  fi
}

# CSV 파일명과 ID 목록 파일명을 지정해서 파일 존재 확인 
csvfile="data.csv"
idlistfile="$1"
filecheck "$csvfile"

filecheck "$idlistfile"

while IFS=, read id name score
do
  grep -xq "$id" "$idlistfile"
  if [ $? -eq 0 ]; then
    echo $name
  fi
done < "$csvfile"
```
- 프로그램 실행 전 환경변수를 선언후 프로그램 실행시 임시 환경변수를 사용할 수 있다.
  ```bash
  # [환경변수 선언] [프로그램 명령어] [프로그램 파라미터]
  TMPDIR=/tmp ./my_script.sh

  IFS=, read id name score
  ```
- `grep -x` : 한 줄 전체가 패턴과 완전히 일치할 때만 표시되도록 하는 것

# 074 숫자로 된 CSV 파일에서 평균값 계산하기 
```bash
#!/bin/sh

# CSV 파일이 존재하지 않으면 종료
if [ ! -f "$1" ]; then
  echo "대상 CSV 파일이 존재하지 않습니다: $1" >&2
  exit 1
fi

# 확장자를 제외한 파일명 취득
filename=${1%.*}

awk -F, '{sum += $3} END{print sum / NR}' "$1" > ${filename}.avg
```

# 075 숫자값(CSV 파일)에서 "*"를 써서 간단한 텍스트 그래프 출력하기 
```bash
#!/bin/sh

csvfile="data.csv"
GRAPH_WIDTH=50

markprint() {
  local i=0
  while [ $i -lt $1 ]
  do
    echo -n "*"
    i=$(expr $i + 1)
  done
}

# 자료에서 최대값 얻음. 역순 정렬해서 첫 줄 얻음
max=$(awk -F, '{print $3}' "$csvfile" | sort -nr | head -n 1)

# 자료가 모두 0이면 최대값을 1로 지정
if [ $max -eq 0 ]; then
  max=1
fi

# CSV 파일을 읽어서 값마다 그래프 출력 
while IFS=, read id name score
do
  markprint $(expr $GRAPH_WIDTH \* $score / $max)
  echo " [$name]"
done < $csvfile
```

# 076 로그 파일 컬럼 위치를 바꿔서 출력하고 보기 쉽게 바꾸기
```bash
#!/bin/sh

# 로그 파일이 존재하지 않으면 종료 
if [ ! -f "$1" ]; then 
  echo "대상 로그 파일이 존재하지 않습니다: $1" >&2
  exit 1
fi 

# 리퀘스트 시작과 원격 호스트를 외부 파일에 출력 
awk '{print $4,$5,$1}' "$1" > "${1}.lst"
```
- TIP: awk를 활용해서 문자열을 재조합해 원하는 대로 출력한다. 
- awk 명령어에서 print 할 때는 쉼표(`,`)가 스페이스가 된다. 


# 077 웹 서버 로그 파일에서 특정 상태값만 취득하기 
```bash
#!/bin/sh

logfile="access_log"

# 로그 파일이 존재하지 않으면 종료 
if [ ! -f "$logfile" ]; then
  echo "대상 로그 파일이 존재하지 않습니다: $logfile" >&2
  exit 1
fi

# HTTP Status를 외부 파일에 출력
awk '$(NF-1)==404 {print $7}' "$logfile" > "${logfile}.404"
```
- TIP : awk로 원하는 문자만 검사해서 조건이 허용되는 케이스 처리하기 
- `NF` : awk 명령어로 처리하는 줄의 컬럼(필드) 수


# 078 시스템 로그에서 IP 주소마다 횟수 집계하기 
```bash
#!/bin/sh

# sshd 로그 파일 
securelog="/var/log/secure"

# IP 주소를 추출하기 위한 패턴으로 변수에 저장 
pattern="^.*sshd\[.*\].*Failed password for.* from \(.*\) port .*"

# 암호 인증 실패 로그에서 IP 주소를 추출, 카운트해서 표시
sed -n "s/$pattern/\1/p" "$securelog" | sort | uniq -c | sort -nr
```
- `sed`로 원하는 문자열만 출력해 정렬한 후 `uniq`로 동일한 줄의 횟수를 세어서 정렬해 출력한다. 
  - 동일한 줄 확인할 때 `sort | uniq -c | sort -nr` 명령어를 자주 쓴다.
- `uniq` : 파일의 반복되는 줄을 리포트 하거나 필
  - `-c` : 동일한 줄의 출현 횟수 세기
- `sort`
  - `-n` : 숫자 정렬
  - `-r` : 역순 정렬

# 079 웹 접속 로그에서 파일별 접속 횟수 집계하기 
```bash
#!/bin/sh

logfile="access_log"

# 로그 파일이 존재하지 않으면 종료
if [ ! -f "$logfile" ]; then 
  echo "대상 로그 파일이 존재하지 않습니다: $logfile" >&2
  exit 1
fi

# 로그 파일에서 GET메소드로 취득한 파일 접속 횟수 집계
# awk 명령어로 파일을 추출해서 sort와 uniq로 카운트해서 역순 정렬
awk '$6=="\"GET" {print $7}' "$logfile" | sort | uniq -c | sort -nr
```
- awk로 로그를 필터링 해서 필요한 문자만 출력해 집계할 수 있는 방법 
- TIP : [AWState](https://awstats.sourceforge.io/)를 이용해 웹 서버 아파치 로그 분석할 수 있다. 

# 080 sed로 HTML 파일 속성을 바꿀 때 슬래시 이스케이프 피하기 
```bash 
#!/bin/sh

# 출력 디렉토리 정의
outdir="newdir"

# 출력 디렉토리 존재 확인, 없으면 에러
if [ ! -d "$outdir" ]; then 
  echo "출력 디렉토리가 존재하지 않습니다: $outdir" >&2
  exit 1
fi 

# 현재 디렉토리의 HTML 파일 처리 
for htmlfile in *.html
do
  # 파일 내용에서 /img/를 /images/로 변환
  sed "s%/img/%/images/%g" "$htmlfile" > "${outdir}/${htmlfile}"
done
```
- `sed` 명령어로 치환할 때 보통 **s 명령어** 뒤에 `/`(슬래시)로 패턴을 표시하고, `/`(슬래시) 자체가 문자열에 포함되어 있을 경우 `\`(역슬래시) 기호로 이스케이프 한다. 
- 하지만 가독성이 떨어서 패턴을 지정하는 구분자를 변경해 사용하면 편하게 활용 가능하다. 
- **s 명령어** 뒤에 있는 문자를 구분자로 인식하며, 보통 `%`를 많이 사용한다. 
```bash
sed "s/\/img\//\/images\//g" test.html

sed "s%/img/%/images/%g" test.html
```

# 081 오른쪽 정렬로 숫자를 표시하는 텍스트 표 만들기 
```bash
#!/bin/sh 

# 검색할 문자열 정의 
 search_text="ERROR 19:"

 # 현재 디렉토리에서 확장자가 .log인 파일을 순서대로 처리 
 for filename in *.log 
 do
  # 일치하는 줄 수를 -c 옵션으로 취득
  count=$(grep -c "$search_text" "$filename")
  # printf 명령어로 오른쪽 정렬 6칸으로 변형해서 출력
  printf "%6s (%s)\n" "$count" "$filename"
done
```
- `grep -c` : 패턴이 일치하는 줄 수를 표시
- `printf` : 문자열을 다양한 포맷으로 출력
  - Usage : `printf [포맷] [아규먼트]` 

# 082 정해진 자리수 숫자에 하이픈 넣기(우편번호 등)
```bash
#!/bin/sh

# 하이픈을 삭제 여부 플래그, 1이면 삭제
d_flg=0

# getopts 명령어로 삭제 옵션(-d) 판별
while getopts "d" option
do
  case $option in 
    d)
      d_flag=1
      ;;
    \?)
      exit 1
      ;;
  esac
done

# 명령어 인수로 지정한 우편번호 파일을 쉘 변수 filename에 대입
shift $(expr $OPTIND - 1)
filename="$1"

# 지정한 우편번호 파일 확인
if [ ! -f "$filename" ]; then
  echo "대상 파일이 존재하지 않습니다: $filename" >&2
  exit 1
fi 

# d_flag가 지정되면 하이픈 삭제, 아니면 추가 
if [ "$d_flag" -eq 1 ]; then
  # *하이픈 삭제
  # awk로 앞뒤 공백 제거 -> 포맷 확인 -> 하이픈 삭제
  awk '{print $1}' "$filename" | grep '^[0-9]\{3\}-[0-9]\{4\}$' | sed "s/-//"
else
  # *하이픈 추가
  # awk로 앞뒤 공백 제거 -> 포맷 확인 -> 하이픈 추가 
  awk '{print #1}' "$filename" | grep '^[0-9]\{7\}$' | sed "s/\(...\)/\1-/"
fi
```


# 083 파일 크기를 줄이기 위해 자바스크립트 파일에서 빈 줄 제거하기 
```bash
#!/bin/sh

# 변환 파일 출력용 디렉토리명
outdir="newdir"

# 파일 출력용 디렉토리 확인
if [ ! -d "$outdir" ]; then 
  echo "Not a directory: $outdir"
  exit 1
fi

for filename in *.js
do
  # 빈 줄 또는 스페이스나 탭 문자만 있는 줄을 sed 명령어 d로 삭제
  sed '/^[[:blank:]]*$/d' "$filename" > "${outdir}/${filename}"
done
```
- `sed`의 `d` 플래그를 통해 일치하는 패턴을 삭제 

# 084 텍스트 파일에서 HTML 파일 만들기 
```bash
#!/bin sh

# HTML에서 이스케이프가 필요한 기호를 문자 참조로 치환 
# 마지막에 줄 끝을 <br> 태그로 치환
sed -e 's/&/\&amp;/g' \
    -e 's/</\&lt;/g' \
    -e 's/>/\&gt;/g' \
    -e "s/'/\&#39;/g" \
    -e 's/"/\&quot;/g' \
    -e 's/$/<br>/' \
    "$1"
```
- `sed`의 `g`플래그 이용해 일치하는 패턴을 치환
  - `-e` : 여러개의 패턴을 지정할 수 있음

# 085 HTML 파일 문자 코드를 자동으로 판별해서 UTF-8로 인코딩 된 파일로 바꾸기
```bash
#!/bin/sh

# 변환한 파일을 출력할 디렉토리
outdir="newdir"

# 파일 출력용 디렉토리 확인 
if [ ! -d "$outdir" ]; then 
  echo "Not a directory: $outdir"
  exit 1 
fi

# 현재 디렉토리의 .html 파일이 대상 
for filename in *.html
do
  # grep 명령어로 meta 태그 Content-Type을 선택해서 sed 명령어로 charset= 지정 부분 추출
  charset=$(grep -i '<meta ' "$filename" | \
              grep -i 'http-equiv="Content-Type"' | \
              sed -n 's/.*charset=\([-_a-zA-Z0-9]*\)".*/\1/p')
  # charset를 얻지 못하면 iconv 명령어를 실행하지 않고 건너뛰기
  if [ -z "$charset" ]; then 
    echo "charset not found: $filename" >&2
    continue
  fi

  # meta 태그에서 추출한 문자 코드를 UTF-8로 변환 후 $outdir 디렉토리에 출력
  iconv -c -f "$charset" -t UTF-8 "$filename" > "${outdir}/${filename}"
done
```
- `sed -n` : silent 모드로 실행 
- `sed`의 `p` 플래그 : 일치하는 부분만 출력
- `grep -i` : 대소문자 구분 없이 실행
- `iconv` : 문자 코드를 변경하기 위해 사용하는 유틸리티
  -  Usage : `iconv -f [입력 문자 코드] -t [출력 문자 코드] [파일명]`
