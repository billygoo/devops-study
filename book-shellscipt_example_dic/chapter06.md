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
